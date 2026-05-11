# MifareID API（NFC 晶片卡認證 API）規格

提供晶片卡身分認證服務
...

Base URL：`https://devapi.1campus.net`

完整的互動式 API 規格請參考 [Swagger 文件](https://devapi.1campus.net/doc/)。

---

## GET /api/mifareID/{schoolDsns}/cardNumber/{cardNumber} — 取得目前登入帳號、操作身分、及基本資料

取得目前使用者帳號進行登入，如果此帳號有多個身分，應該開啟傳入的指定身分進行使用

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 指定學校的主機名稱（例如 `test.hc.edu.tw`） |
| cardNumber | path | string | 是 | mifare晶片卡的值 |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| status | string | 查詢結果： * success 表示成功 * not found 表示查不到這個代碼(可能已失效或錯誤) |
| account | string | 使用者帳號 |
| schoolDsns | string | 學校主機唯一識別 |
| schoolName | string | 學校名稱 |
| schoolYear | integer (int32) | 學校目前學年度 |
| semester | integer (int32) | 學校目前學期 |
| language | string | 使用者語系 |
| roleType | string | 操作身分的類型( teacher、student )，對應的 teacher、student 欄位中會有此操作身分的基本資料 |
| teacher | object | 當 roleType = teacher 時才會有值，否則為null |
| student | object | 當 roleType = student 時才會有值，否則為null |

#### teacher 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| teacherID | integer (int64) | 教師的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| teacherSourceIndex | string |  |
| teacherNo | string | 教師編號 |
| teacherName | string | 教師的姓名 |
| teacherStatus | string | 狀態(在職、已離職) |

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
| studentStatus | string | 學籍狀態(在學、不在校) |
