# IdentityCode API（身分認證碼 API）規格

管理identityCode，提供身分認證服務
...

Base URL：`https://devapi.1campus.net`

完整的互動式 API 規格請參考 [Swagger 文件](https://devapi.1campus.net/doc/)。

---

## GET /{dsns}/identity/{code} — 取得目前登入帳號、操作身分、及基本資料

取得目前使用者帳號進行登入，如果此帳號有多個身分，應該開啟傳入的指定身分進行使用

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |
| code | path | string | 是 | 1Campus平台產生的一次性使用代碼，300秒後失效 |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| schoolDsns | string | 學校主機唯一識別 |
| schoolName | string | 學校名稱 |
| schoolYear | integer (int32) | 學校目前學年度 |
| semester | integer (int32) | 學校目前學期 |
| account | string | 使用者帳號 |
| language | string | 使用者語系 |
| roleType | string | 操作身分的類型( teacher、student、parent )，對應的 teacher、student、parent 欄位中會有此操作身分的基本資料 |
| teacher | object | 當 roleType = teacher 時才會有值，否則為null |
| student | object | 當 roleType = student 時才會有值，否則為null |
| parent | object | 當 roleType = parent 時才會有值，否則為null |

#### teacher 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| teacherID | integer (int64) | 教師的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| teacherSourceIndex | string |  |
| teacherNo | string | 教師編號 |
| teacherName | string | 教師的姓名 |

#### student 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| classID | integer (int64) | 學生所屬班級的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| gradeYear | integer (int32) | 學生所屬班級的年級 |
| className | string | 學生所屬班級的班級名稱 |
| classNo | string | 學生所屬班級的班級序號 |
| seatNo | integer (int32) | 學生的座號 |
| studentID | integer (int64) | 學生的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| studentSourceIndex | string |  |
| studentNumber | string | 學生的學號 |
| studentName | string | 學生的姓名 |

#### parent 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| classID | integer (int64) | 子女所屬班級的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| gradeYear | integer (int32) | 子女所屬班級的年級 |
| className | string | 子女所屬班級的班級名稱 |
| classNo | string | 子女所屬班級的班級序號 |
| seatNo | integer (int32) | 子女的座號 |
| studentID | integer (int64) | 子女的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| studentSourceIndex | string |  |
| studentNumber | string | 子女的學號 |
| studentName | string | 子女的姓名 |
| relationship | string | 家長的稱謂 |

### 錯誤回應

- `404`：指定的 identity code 不存在或已被使用
- `410`：指定的 identity code 已過期
- `500`：系統在處理身份資訊時發生非預期錯誤

---

## POST /api/identityCode/{schoolDsns}/teacher/create — 產生教師身分的identityCode

teacherID 或 teacherSourceIndex 必須擇一傳入

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `j.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| teacherID | integer (int64) | 否 | 要產生的教師系統編號 |
| teacherSourceIndex | string | 否 | 要產生的教師sourceIndex |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## POST /api/identityCode/{schoolDsns}/student/create — 產生學生身分的identityCode

studentID 或 studentSourceIndex 必須擇一傳入

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `j.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| studentID | integer (int64) | 否 | 要產生的學生系統編號 |
| studentSourceIndex | string | 否 | 要產生的學生sourceIndex |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限
