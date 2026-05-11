# RoleClaim API（身分綁定 API）規格

透過資料驗證，使帳號與已建立資料產生關連
...

Base URL：`https://devapi.1campus.net`

完整的互動式 API 規格請參考 [Swagger 文件](https://devapi.1campus.net/doc/)。

---

## GET /api/roleClaim/teacher/{schoolDsns}/list — 總攬，列出全校教師帳號及教師代碼

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `h.demo.1campus.net`） |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## GET /api/roleClaim/student/{schoolDsns}/list — 總攬，查詢學生的帳號、學生代碼、以及學生綁定的家長

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `h.demo.1campus.net`） |
| studentID | query | integer | 否 | 學生系統編號 |
| studentSourceIndex | query | string | 否 | 學生SourceIndex |
| classID | query | string | 否 | 所屬班級系統編號 |
| classSourceIndex | query | string | 否 | 所屬班級SourceIndex |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## GET /api/roleClaim/teacher/{schoolDsns}/find — 使用教師代碼查詢教師

使用教師代碼查詢教師，驗證教師代碼是否正確，並回傳教師資料以供確認

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `h.demo.1campus.net`） |
| teacherCode | query | string | 是 | 教師代碼 |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| teacherID | integer (int64) | 教師系統編號，如果此攔為null時，表示代碼錯誤，沒有對應學生。 |
| teacherSourceIndex | string | 教師SourceIndex |
| teacherName | string | 教師姓名 |
| teacherCode | string | 教師代碼 |
| teacherAcc | string | 目前教師帳號 |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## GET /api/roleClaim/student/{schoolDsns}/find — 使用學生代碼查詢學生

使用學生代碼查詢學生，驗證學生代碼是否正確，並回傳學生資料以供確認

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `h.demo.1campus.net`） |
| studentCode | query | string | 是 | 學生代碼 |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| classID | integer (int64) | 班級系統編號 |
| classSourceIndex | string | 班級SourceIndex |
| className | string | 學生所屬班級的班級名稱 |
| studentID | integer (int64) | 學生系統編號，如果此攔為null時，表示代碼錯誤，沒有對應學生。 |
| seatNo | integer (int32) | 學生的座號 |
| studentNumber | string | 學生的學號 |
| studentName | string | 學生姓名 |
| studentAcc | string | 目前學生帳號 |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## GET /api/roleClaim/parent/{schoolDsns}/find — 使用家長代碼查詢子女

使用家長代碼查詢子女，驗證家長代碼是否正確，並回傳子女資料以供確認

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `h.demo.1campus.net`） |
| parentCode | query | string | 是 | 家長代碼 |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| classID | integer (int64) | 班級系統編號 |
| classSourceIndex | string | 班級SourceIndex |
| className | string | 學生所屬班級的班級名稱 |
| studentID | integer (int64) | 學生系統編號，如果此攔為null時，表示代碼錯誤，沒有對應學生。 |
| seatNo | integer (int32) | 學生的座號 |
| studentNumber | string | 學生的學號 |
| studentName | string | 學生姓名 |
| parent | array of object | 目前綁定家長清單 |

#### parent 陣列元素 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| parentAcc | string | 家長帳號 |
| parentName | string | 家長姓名 |
| relationship | string | 家長稱謂 |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## POST /api/roleClaim/teacher/{schoolDsns}/setCode — 設定教師代碼

修改指定教師的教師代碼

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `h.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## POST /api/roleClaim/teacher/{schoolDsns}/bind — 綁定教師帳號

將教師帳號設為指定帳號

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `h.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| teacherID | integer (int64) | 否 | 教師系統編號 |
| teacherAcc | string | 否 | 連結的教師帳號，將此帳號設定為teacherCode對應教師的teacherAcc |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## POST /api/roleClaim/student/{schoolDsns}/setCode — 設定學生代碼

修改指定學生的學生代碼

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `h.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## POST /api/roleClaim/student/{schoolDsns}/setParentCode — 設定學生的家長代碼

修改指定學生的家長代碼

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `h.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## POST /api/roleClaim/student/{schoolDsns}/bind — 綁定學生帳號

將學生帳號修改為指定帳號

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `h.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| studentID | integer (int64) | 否 | 學生系統編號 |
| studentAcc | string | 否 | 連結的學生帳號，將此帳號設定為studentCode對應學生的studentAcc |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## POST /api/roleClaim/student/{schoolDsns}/bindParent — 綁定學生家長

綁定學生家長帳號，將指定家長帳號新增至學生家長清單中

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `h.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| studentID | integer (int32) | 否 | 學生系統編號 |
| parentAcc | string | 否 | 家長帳號 |
| parentName | string | 否 | 家長姓名 |
| relationship | string | 否 | 家長稱謂 |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## POST /api/roleClaim/student/{schoolDsns}/unbindParent — 解除綁定學生家長

解除綁定家長帳號，將指定家長帳號從學生家長清單中移除

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `h.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| studentID | integer (int32) | 否 | 學生系統編號 |
| parentAcc | string | 否 | 家長帳號 |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限
