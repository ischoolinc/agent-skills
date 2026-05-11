# Code API（代碼 API）規格

Code API 提供代碼相關功能。所有需授權的端點皆需透過 OAuth Bearer Token 進行驗證。

Base URL：`https://devapi.1campus.net`

完整的互動式 API 規格請參考 [Swagger 文件](https://devapi.1campus.net/doc/)。

---

## GET /api/code/qrcode/img — 取得QRCode圖片

取得QRCode圖片，直接取代 https://chart.googleapis.com/chart?... ，傳入參數應可直接使用。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| chl | query | string | 是 | 要編碼的資料（例如 `https://www.ischool.com.tw`） |
| chs | query | string | 否 | 圖片大小（例如 `300x300`） |
| choe | query | string | 否 | 如何將資料編碼到 QR code。預設值UTF-8（例如 `UTF-8`） |
| chld | query | string | 否 | error_correction_level可接受(L、M、Q、H)，預設L（例如 `L`） |

### Response

- `200`：回傳 `image/png` 格式的資料

### 錯誤回應

- `400`：傳入參數錯誤

---

## POST /api/code/temp/push — 建立代碼

將資料上傳，建理或指定代碼，以供後續使用

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| code | string | 是 | 代碼，如果自動產生，傳入"generate"；如果指定，傳入指定的代碼，代碼重複時會互相覆蓋。 |
| body | object | 是 | 資料內容，不限格式，可自行定義，傳陣列就會get到陣列，傳物件就會get到物件，傳字串就get到字串哦，get時的content-type會跟著變化。 |
| expire | string | 是 | 代碼有效期限，如果是字串，格式為 yyyy/MM/dd HH:mm:ss，例如 2020/01/01 12:00:00。 |
| public | boolean | 否 | 是否公開，如果是true，則任何人都可以使用此代碼；如果是false，則只有建立此代碼的人可以取得(必須透過 /api/code/temp/:code/private )。預設值為true。 |

### 範例

```json
{
  "code": "generate",
  "body": {
    "name": "John",
    "age": 30,
    "city": "New York"
  },
  "expire": "2020/01/01 12:00:00"
}
```

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## GET /api/code/temp/{code}/private — 取得私有代碼資料

取得私有代碼資料，必須傳入access_token進行驗證

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| code | path | string | 是 | 代碼 |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## GET /api/code/temp/{code} — 取得代碼資料

取得代碼資料，不需驗證，如果是javascript前端呼叫，會自動回傳Access-Control-Allow-Origin，避免CORS問題。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| code | path | string | 是 | 代碼 |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限
