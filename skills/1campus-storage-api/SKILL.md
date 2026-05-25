---
name: 1campus-storage-api
description: 1Campus Storage Service 對外 API 參考文件。涵蓋檔案儲存服務的前置準備、認證流程、Token API（建立/查詢/更新/刪除 upload token）、File API（上傳/下載/metadata/刪除檔案）。當開發者詢問 storage 檔案上傳、下載、token 管理、或 1campus 儲存服務串接時觸發此 Skill。
license: CC-BY-ND-4.0
metadata:
  author: 1campus
  organization: iSchool
  version: '1.0.0'
---

# 1Campus Storage API

1Campus Storage Service 提供檔案儲存與管理功能，讓第三方應用系統能上傳、下載、管理檔案。檔案儲存於 Google Cloud Storage，透過 Token 機制控制存取權限。

**Base URL**: `https://storage.1campus.net`

---

## Getting Started（前置準備）

### 1. 取得 OAuth Client

如果您還沒有 `client_id` 和 `client_secret`，請至以下網址註冊：

> https://auth.ischool.com.tw/1campus/manage/

**注意**: URL 結尾的 `/` 是必要的，少了會無法正常存取。

### 2. 申請 Storage Scope

Storage API 需要 `storage` scope，此 scope 需要額外申請，請聯繫 1Campus 相關人員開通。

### 3. 取得 Access Token

使用 **Client Credentials** 流程取得 access_token：

```bash
curl -X POST 'https://auth.ischool.com.tw/oauth/token.php' -H 'Content-Type: application/x-www-form-urlencoded' -d 'grant_type=client_credentials&client_id={YOUR_CLIENT_ID}&client_secret={YOUR_CLIENT_SECRET}&scope=storage'
```

回應：
```json
{
  "access_token": "a1b2c3d4e5f6...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### 4. 串接流程總覽

```
取得 access_token（OAuth Client Credentials）
    ↓
POST /token（建立 upload_token，指定學校 dsns 與限制條件）
    ↓
POST /file（用 upload_token 上傳檔案，取得 fileid 與 modify_token）
    ↓
GET /file/:fileid（用 fileid 下載檔案）
PUT /file/:fileid（用 modify_token 換檔，fileid 不變）
DELETE /file/:fileid（用 modify_token 刪除檔案）
```

---

## 認證機制

Storage API 使用兩層 token：

| Token | 用途 | 取得方式 |
|-------|------|---------|
| `access_token` | 管理 upload token（建立/查詢/更新/刪除） | OAuth Client Credentials 流程 |
| `upload_token` | 上傳檔案 | 透過 `POST /token` 建立 |
| `modify_token` | 換檔（PUT）/ 刪除檔案（DELETE） | 上傳成功時回傳 |

**access_token 傳遞方式**（擇一）：
- Header: `Authorization: Bearer {access_token}`
- Query: `?access_token={access_token}`

**upload_token 傳遞方式**（擇一）：
- Header: `upload_token: {token}`
- Query: `?upload_token={token}`

**modify_token 傳遞方式**（擇一）：
- Header: `modify_token: {token}`
- Query: `?modify_token={token}`

---

## Token API

管理 upload token 的生命週期。所有 Token API 都需要 `access_token` 認證。

### POST /token — 建立 upload token

建立一個有時效性的上傳令牌，用於後續檔案上傳。

**認證**: 需要 `access_token`

**Request Body** (JSON):

| 參數 | 類型 | 必填 | 預設值 | 說明 |
|------|------|:---:|--------|------|
| `dsns` | string \| null | 否 | null | 目標學校 DSNS，null 表示存放到 client 自身空間 |
| `remaining_uploads` | number | 否 | 100 | 允許上傳的檔案數量上限 |
| `max_size` | number | 否 | 5242880 (5MB) | 單檔大小上限（bytes） |
| `expiry_hours` | number | 否 | 見下方規則 | 上傳檔案的過期時間（小時） |

> ⚠️ **預設值規則（重要，與直覺不同）**
>
> - **只有 request body 是完全空物件 `{}`** 時，server 才會套用整組預設值（含 `expiry_hours: 168`）
> - body 帶任一欄位（例如 `{"dsns":"xxx"}`）但**省略 `expiry_hours`** → 視為未設定 → **上傳的檔案永不過期**（`valid_until = NULL`）
> - `expiry_hours: 0` 或其他 falsy 值 → 同樣是**永不過期**
> - **`upload_token` 本身的有效期（回應中的 `valid_until`）永遠是固定 24 小時**，由 DB 預設值決定，**不受 `expiry_hours` 影響**。`expiry_hours` 只控制「上傳檔案的存活時間」

**注意**: 若指定 `dsns`，系統會自動檢查該學校是否授權此應用使用儲存服務。若學校不存在於系統中，會自動從 1Campus 平台同步。

```bash
curl -X POST 'https://storage.1campus.net/token' -H 'Authorization: Bearer {access_token}' -H 'Content-Type: application/json' -d '{"dsns":"j.demo.1campus.net","remaining_uploads":10,"max_size":10485760,"expiry_hours":48}'
```

**回應** (200):
```json
{
  "id": 123,
  "token": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "restrictions": {
    "dsns": "j.demo.1campus.net",
    "remaining_uploads": 10,
    "max_size": 10485760,
    "expiry_hours": 48,
    "client_id": "edf96e2da7a4f156f1df52f07ab3490f"
  },
  "create_at": "2026-04-02T10:00:00+08:00",
  "update_at": "2026-04-02T10:00:00+08:00",
  "valid_until": "2026-04-04T10:00:00+08:00",
  "ref_client_id": 1
}
```

**錯誤**:
| 狀態碼 | 說明 |
|--------|------|
| 401 | access_token 無效、過期或未提供 |
| 404 | 指定的學校不存在於 1Campus 平台 |

### GET /token/:upload_token — 查詢 token 資訊

取得指定 upload token 的詳細資訊。若 token 已過期則視為不存在。

**認證**: 需要 `access_token`

```bash
curl 'https://storage.1campus.net/token/{upload_token}' -H 'Authorization: Bearer {access_token}'
```

**回應** (200): 同建立 token 的回應格式。

**Token 不存在或已過期**:
```json
{
  "msg": "not found"
}
```

### PUT /token/:upload_token — 更新 token 限制

更新 upload token 的限制條件（部分更新，僅覆蓋指定欄位）。`client_id` 不可修改。

**認證**: 需要 `access_token`

**Request Body** (JSON): 與建立 token 相同的參數，僅需傳入要修改的欄位。

```bash
curl -X PUT 'https://storage.1campus.net/token/{upload_token}' -H 'Authorization: Bearer {access_token}' -H 'Content-Type: application/json' -d '{"remaining_uploads":50}'
```

**回應** (200): 更新後的完整 token 記錄。

**錯誤**:
| 狀態碼 | 說明 |
|--------|------|
| 404 | Token 不存在 |

### DELETE /token/:upload_token — 刪除 token

刪除指定的 upload token。

**認證**: 需要 `access_token`

```bash
curl -X DELETE 'https://storage.1campus.net/token/{upload_token}' -H 'Authorization: Bearer {access_token}'
```

**回應** (200):
```json
{
  "status": "ok"
}
```

---

## File API

檔案的上傳、下載、查詢與刪除。支援跨域存取（CORS 全開）。

**CORS 設定**: 允許所有 origin，支援 GET、POST、DELETE、PUT methods。

### POST /file — 上傳檔案

使用 multipart/form-data 上傳一或多個檔案。

**認證**: 需要 `upload_token`

**Request**: `Content-Type: multipart/form-data`，每個 file field 為一個檔案。

```bash
curl -X POST 'https://storage.1campus.net/file?upload_token={upload_token}' -F 'file=@/path/to/document.pdf'
```

**回應** (200):
```json
[
  {
    "form_field": "file",
    "filename": "document.pdf",
    "fileid": "f1a2b3c4-d5e6-7890-abcd-ef1234567890",
    "modify_token": "m1a2b3c4-d5e6-7890-abcd-ef1234567890",
    "expiry": "2026-04-09T10:00:00+08:00",
    "mime_type": "application/pdf"
  }
]
```

**重要欄位**:
- `fileid` — 檔案唯一識別碼，用於下載和查詢 metadata
- `modify_token` — 檔案修改令牌，用於換檔（PUT）/ 刪除檔案（DELETE）（請妥善保存）
- `expiry` — 檔案過期時間，由 upload token 的 `expiry_hours` 決定

**錯誤**:
| 狀態碼 | 說明 |
|--------|------|
| 401 | upload_token 無效、過期或未提供 |
| 500 | 上傳過程發生錯誤（檔案超過 max_size 等） |

**注意**: 若多檔上傳中任一檔案失敗，所有已上傳的檔案都會被回滾刪除。

### GET /file/:fileid/:filename? — 下載檔案

取得檔案內容。實際上會 redirect 到 Google Cloud Storage 的 signed URL。

**認證**: 無需認證（fileid 本身即為存取憑證）

**路徑參數**:
| 參數 | 說明 |
|------|------|
| `fileid` | 檔案唯一識別碼（必填） |
| `filename` | 指定下載時的檔案名稱（選填） |

**Query 參數**:
| 參數 | 說明 |
|------|------|
| `dl` | 設為 `true` 使用原始檔名下載；設為自訂字串則用該字串作為檔名 |

```bash
# 直接存取（瀏覽器會 redirect 到檔案）
curl -L 'https://storage.1campus.net/file/{fileid}'

# 指定下載檔名（透過路徑）
curl -L 'https://storage.1campus.net/file/{fileid}/my-report.pdf'

# 指定下載檔名（透過 query）
curl -L 'https://storage.1campus.net/file/{fileid}?dl=my-report.pdf'

# 使用原始檔名下載
curl -L 'https://storage.1campus.net/file/{fileid}?dl=true'
```

**回應**: HTTP 302 Redirect 到 GCS signed URL

**Cache 行為**:
- 302 回應帶有 `Cache-Control: public, max-age=3000`（50 分鐘），瀏覽器與中間 CDN 會快取此 redirect
- GCS object 本身的 `cacheControl` metadata 為 `public, max-age=3600`（1 小時），瀏覽器會快取實際檔案內容
- **換檔（PUT）後最多 1 小時內，使用者可能仍看到舊內容**；如需即時更新，請 POST 新檔開新 fileid

**錯誤**:
| 狀態碼 | 說明 |
|--------|------|
| 404 | fileid 未提供、檔案不存在、檔案已過期、或 signed URL 產生失敗 |

### GET /file/:fileid/metadata — 查詢檔案 metadata

取得檔案的基本資訊（不含檔案內容）。

**認證**: 無需認證

```bash
curl 'https://storage.1campus.net/file/{fileid}/metadata'
```

**回應** (200):
```json
{
  "fileid": "f1a2b3c4-d5e6-7890-abcd-ef1234567890",
  "size": 1048576,
  "valid_until": "2026-04-09T10:00:00+08:00",
  "origin_name": "document.pdf",
  "content_type": "application/pdf",
  "create_at": "2026-04-02T10:00:00+08:00",
  "update_at": "2026-04-02T10:00:00+08:00"
}
```

**回應欄位**:
| 欄位 | 說明 |
|------|------|
| `fileid` | 檔案唯一識別碼 |
| `size` | 檔案大小（bytes） |
| `valid_until` | 過期時間，null 表示永不過期 |
| `origin_name` | 原始檔案名稱 |
| `content_type` | MIME type |
| `create_at` | 建立時間 |
| `update_at` | 最後更新時間 |

**錯誤**:
| 狀態碼 | 說明 |
|--------|------|
| 404 | fileid 未提供或檔案不存在 |

### PUT /file/:fileid — 換檔（更新檔案內容）

使用 `modify_token` 替換檔案內容，**fileid、URL、modify_token 全部保留不變**。
適用於 logo、background 等「URL 不可變但內容會更新」的場景，避免「先刪後傳」導致的孤兒檔。

**認證**: 需要 `modify_token`

**路徑參數**:
| 參數 | 必填 | 說明 |
|------|:---:|------|
| `fileid` | 是 | 必須與 modify_token 對應的檔案相符 |

**Request**: `Content-Type: multipart/form-data`，**僅處理第一個檔案欄位**（任意欄位名；多餘檔案會被忽略並 log 警告）

```bash
curl -X PUT 'https://storage.1campus.net/file/{fileid}' -H 'modify_token: {modify_token}' -F 'file=@/path/to/new-logo.png'
```

**回應** (200):
```json
{
  "fileid": "f1a2b3c4-d5e6-7890-abcd-ef1234567890",
  "filename": "new-logo.png",
  "size": 24680,
  "mime_type": "image/png",
  "modify_token": "m1a2b3c4-d5e6-7890-abcd-ef1234567890",
  "expiry": "2026-04-09T10:00:00+08:00",
  "update_at": "2026-04-16T11:00:00+08:00"
}
```

> `update_at` 是換檔當下的 server 時間（與 DB 寫入可能差 < 1 秒）；若需 DB 權威值請查 `GET /file/{fileid}/metadata`。

**保留欄位**: `fileid`、`modify_token`、`expiry`（valid_until）、所屬 client/school 都不變
**更新欄位**: `filename`（dl_name）、`size`、`mime_type`（content_type）、`update_at`

**Cache 注意事項**:
換檔後最多 1 小時內，使用者瀏覽器與 CDN 快取仍可能回傳舊內容。
如需立即生效，請改用 `POST /file` 上傳新檔（產生新 fileid 與 URL）。

**錯誤**:
| 狀態碼 | 說明 |
|--------|------|
| 400 | multipart 解析失敗、未提供檔案 |
| 401 | modify_token 未提供 |
| 403 | URL 的 fileid 與 modify_token 對應的檔案不符 |
| 404 | modify_token 對應的檔案不存在 |
| 409 | 原檔尚未完成上傳（success=false），不允許換檔 |
| 410 | 原檔已過期，不允許換檔（請改用 POST 新上傳） |
| 500 | GCS 上傳或 DB 更新失敗 |

### DELETE /file/:fileid — 刪除檔案

使用 modify_token 刪除檔案。同時刪除 GCS 上的實體檔案與資料庫記錄。

**認證**: 需要 `modify_token`

**路徑參數**:
| 參數 | 必填 | 說明 |
|------|:---:|------|
| `fileid` | 否 | 若提供，必須與 modify_token 對應的檔案相符；不提供則僅以 modify_token 為準 |

```bash
# 推薦：URL 帶 fileid（雙重確認）
curl -X DELETE 'https://storage.1campus.net/file/{fileid}?modify_token={modify_token}'

# 也可省略 fileid（僅靠 modify_token 識別）
curl -X DELETE 'https://storage.1campus.net/file?modify_token={modify_token}'
```

**回應** (200):
```json
{
  "count": 1
}
```

**檔案不存在時**:
```json
{
  "count": 0
}
```

**錯誤**:
| 狀態碼 | 說明 |
|--------|------|
| 401 | modify_token 未提供 |
| 403 | URL 的 fileid 與 modify_token 對應的檔案不符 |

---

## 完整串接範例

### 上傳檔案到指定學校

```bash
# 1. 取得 access_token
ACCESS_TOKEN=$(curl -s -X POST 'https://auth.ischool.com.tw/oauth/token.php' -d 'grant_type=client_credentials&client_id={CLIENT_ID}&client_secret={CLIENT_SECRET}&scope=storage' | jq -r '.access_token')

# 2. 建立 upload token（指定學校、限制 10 個檔案、每個最大 10MB、48 小時過期）
UPLOAD_TOKEN=$(curl -s -X POST 'https://storage.1campus.net/token' -H "Authorization: Bearer $ACCESS_TOKEN" -H 'Content-Type: application/json' -d '{"dsns":"j.demo.1campus.net","remaining_uploads":10,"max_size":10485760,"expiry_hours":48}' | jq -r '.token')

# 3. 上傳檔案
curl -X POST "https://storage.1campus.net/file?upload_token=$UPLOAD_TOKEN" -F 'file=@./document.pdf'

# 回應中的 fileid 用於下載，modify_token 用於刪除
```

### 上傳檔案到 client 自身空間

```bash
# dsns 設為 null 或不帶，檔案存放到 client 自身空間
UPLOAD_TOKEN=$(curl -s -X POST 'https://storage.1campus.net/token' -H "Authorization: Bearer $ACCESS_TOKEN" -H 'Content-Type: application/json' -d '{"remaining_uploads":5,"max_size":5242880}' | jq -r '.token')

curl -X POST "https://storage.1campus.net/file?upload_token=$UPLOAD_TOKEN" -F 'file=@./image.png'
```
