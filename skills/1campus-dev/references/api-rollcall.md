# Rollcall API（課堂點名 API）規格

提供與校務系統整合的出缺席管理功能，支援課堂點名的完整作業流程。

# 內容
## 課堂點名API (rollcall)
課堂點名 API 提供應用程式進行出缺席管理所需的功能，包含點名設定檔查詢、學生資料取得與缺曠紀錄寫入。

**重要設計
- **「出席」表示方式**：學生出席時，absenceName 使用空字串 (`""`) 表示
- **單日單節設計**：每次 API 呼叫處理單一日期的單一節次，簡化邏輯並提高安全性

**點名資料表特色
- **一日一筆記錄**：每個學生每天在資料表中只有一筆記錄
- **XML 包含所有節次**：學生當天所有有假別的節次都記錄在同一筆資料的 XML 中
- **全出席的表示**：學生當日全部出席時，資料表中查無該學生記錄（非空值或空 XML）

**重要寫入邏輯
點名系統與請假系統整合時的重要概念：

**基本原則**：
- **正式請假優先**：學生透過正式流程請的假（如病假、事假）具有較高保護，通常無法被課堂點名覆蓋
- **點名假別可調整**：教師在課堂上記錄的狀態（如曠課、遲到、早退）可以互相調整
- **出席確認限制**：教師確認學生出席時，只能覆蓋部分類型的缺席記錄

**實際使用情境**：
- ✅ 曠課 → 遲到：課堂狀態可以調整
- ✅ 遲到 → 出席：課堂狀態可以修正為出席
- ❌ 病假 → 曠課：正式請假無法改為課堂狀態
- ❌ 事假 → 出席：正式請假無法確認為出席

**API 使用建議**：
- 寫入前先透過 studentAbsence API 查看學生當前狀態
- 若寫入失敗，檢查錯誤訊息了解是否為保護性假別
- 空字串表示確認出席，請謹慎使用

Base URL：`https://devapi.1campus.net`

完整的互動式 API 規格請參考 [Swagger 文件](https://devapi.1campus.net/doc/)。

---

## GET /api/rollcall/{dsns}/config — 取得點名設定檔

取得該校的點名設定，包含節次定義與缺曠類別設定

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| period | array of object | 節次設定清單 |

#### period 陣列元素 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| name | string | 節次名稱 |
| schedulePeriod | integer | 對應課表節次編號（來源：節次對照表的 CoursePeriod 屬性） |
| absence | array of object | 該節次可用的缺曠類別 |

### 錯誤回應

- `400`：參數錯誤
- `401`：未授權
- `403`：權限不足
- `500`：伺服器錯誤

---

## GET /api/rollcall/{dsns}/studentAbsence — 取得點名學生資料

取得指定日期的學生清單及預設缺曠狀態。period 為非必填，未提供時回傳所有節次資料，按節次分組。需擇一提供 classID 或 courseID。點名權限由實作服務商自行處理。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |
| occurDate | query | string | 是 | 點名日期 (yyyy/MM/dd)（例如 `2023/05/27`） |
| period | query | string | 否 | 節次識別值（非必填，未提供時回傳所有節次）（例如 `一`） |
| classID | query | integer | 否 | 班級編號（與 courseID 擇一必填）（例如 `8`） |
| courseID | query | integer | 否 | 課程編號（與 classID 擇一必填） |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| period | array of object | 節次清單，包含每節的學生缺曠資料 |

#### period 陣列元素 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| name | string | 節次名稱 |
| schedulePeriod | integer | 對應課表的節次序號，無對應時為 null |
| student | array of object | 該節次的學生清單 |

### 錯誤回應

- `400`：參數錯誤
- `401`：未授權
- `403`：權限不足
- `500`：伺服器錯誤

---

## POST /api/rollcall/{dsns}/pushAbsence — 寫入缺曠紀錄

批次寫入學生缺曠紀錄，遵守假別保護機制，確認假別後寫入 attendance 出缺席記錄表及 log 記錄表

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| rollCall | object | 否 | 點名資料 |

#### rollCall 物件欄位

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| occurDate | string | 否 | 點名日期 (yyyy/MM/dd) |
| period | string | 否 | 節次識別值 |
| student | array of object | 否 | 學生缺曠資料 |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| result | string | 處理結果 |
| processed | object | 處理統計 |
| summary | object | 處理摘要 |
| errors | array of object | 錯誤記錄（如有） |

#### processed 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| total | integer | 總記錄數 |
| success | integer | 成功記錄數 |
| failed | integer | 失敗記錄數 |

#### summary 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| dates | array of string | 處理的日期清單 |
| periods | array of string | 處理的節次清單 |
| students | array of integer | 處理的學生ID清單 |

#### errors 陣列元素 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| occurDate | string | 發生錯誤的日期 |
| period | string | 發生錯誤的節次 |
| studentID | integer | 發生錯誤的學生ID |
| error | string | 錯誤訊息 |

### 錯誤回應

- `400`：參數錯誤
- `401`：未授權
- `403`：權限不足
- `409`：資料衝突
- `500`：伺服器錯誤
