# Schedule API（日課表 API）規格

提供日課表查詢服務，支援依日期區間與過濾條件查詢課表資訊。

# 內容
## 日課表API (schedule)
日課表 API 提供課堂點名所需的課表資訊，包含課程、教師、學生、場地等相關資料。

日課表查詢可用於課堂點名等需要以日為單位的課程表，指定查詢的對象（教師、學生或課程），系統會根據操作身分的權限範圍回傳對應的課表資料。

Base URL：`https://devapi.1campus.net`

完整的互動式 API 規格請參考 [Swagger 文件](https://devapi.1campus.net/doc/)。

---

## GET /api/schedule/{dsns}/daily — 取得日課表

依日期區間與過濾條件回傳日課表清單。teacherID 與 studentID 擇一；所有過濾參數皆為選填，可查詢該日期區間的所有課表。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |
| schoolYear | query | integer | 否 | 學年度，未提供時以目前學年期為預設值（例如 `113`） |
| semester | query | integer | 否 | 學期 (1 或 2)，未提供時以目前學年期為預設值（例如 `1`） |
| beginDate | query | string | 否 | 起始日期 (yyyy/MM/dd)。未提供時將以學年期為範圍查詢（例如 `2024/06/01`） |
| endDate | query | string | 否 | 結束日期 (yyyy/MM/dd)。僅當 beginDate 提供時生效；若缺漏則等於 beginDate（例如 `2024/07/31`） |
| teacherID | query | integer | 否 | 教師編號；與 studentID 擇一 |
| studentID | query | integer | 否 | 學生編號；與 teacherID 擇一 |
| courseID | query | integer | 否 | 課程編號 |

### 錯誤回應

- `400`：參數錯誤
- `401`：未授權
- `403`：權限不足
- `404`：查無資料
- `500`：伺服器錯誤
- `503`：無法取得目前學年期設定

---

## GET /api/schedule/{dsns}/courseDate — 取得學校上課日清單

依學年度和學期回傳該學期的上課日清單，用於協助前端進行日期選擇與查詢引導。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |
| schoolYear | query | integer | 否 | 學年度，未提供時以目前學年期為預設值（例如 `113`） |
| semester | query | integer | 否 | 學期 (1 或 2)，未提供時以目前學年期為預設值（例如 `1`） |

### 錯誤回應

- `400`：參數錯誤
- `401`：未授權
- `403`：權限不足
- `404`：查無資料
- `500`：伺服器錯誤
- `503`：無法取得目前學年期設定
