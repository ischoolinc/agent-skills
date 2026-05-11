# Jasmine API（班群資料 API）規格

使教育領域中各個優秀的系統，能夠更快速地將服務提供給使用者。
1Campus 平台提供完整的教學場域資訊 API，透過這些 API 讓資料整合不再是阻礙。

# 內容
## 身分認證API (identityCode)
當您的系統上架至 1Campus 平台後，使用者可以透過 1Campus 快速登入您的系統。身分認證 API 提供用戶的帳號、操作身分與基本資料，讓您的系統能夠以統一帳號提供服務，實現無縫登入。

### 模擬測試
* 使用展示專用帳號進行測試：
  * 帳號：`dev.teacher01@1campus.net`
  * 密碼：`1234`
* [操作影片](https://youtu.be/_5oqxe03x78?si=_niYX6u-6IGlH3JH)

1. 登入 [1Campus.net](https://1Campus.net)
2. 鍵盤按住 `Alt + Shift`，滑鼠點擊右上角帳號選單，選擇「開發者設定」。
3. 進入「開發者設定」後，您將看到此帳號在這台電腦(瀏覽器)上的開發者階段小工具。
4. 點擊「+」新增設定：
  * title：顯示的小工具名稱
  * guid：唯一識別碼
  * role：使用身分（應選擇 `student`、`teacher`、`parent` 或 `admin` 其中之一。如有多個身分，請為每個身分建立不同設定）
  * icon：小工具圖標（200x200，去背景）
  * url：小工具開啟網址（可使用開發主機的 `localhost` 地址，例如 `http://127.0.0.1:1234/xxxx`）
5. 新增設定後，切換身分回主頁面，您將看到紅色標註的小工具，即可開始模擬使用情境。
  * 備註：開發者設定中的小工具只會在這台電腦上看到，開發完成後可將同樣JSON設定傳給1Campus平台申請上架
6. 若多人共同開發或測試，您可以分享開發者階段小工具的 JSON 設定。
7. 測試學校清單
  * 1Campus展示高中，DSNS : `h.demo.1campus.net`
  * 1Campus展示國中，DSNS : `j.demo.1campus.net`
  * 1Campus展示國小，DSNS : `p.demo.1campus.net`
8. 測試帳號清單（密碼皆為 1234）：
  * `dev.teacher01@1campus.net`：1Campus展示國中-班導師/授課教師/家長、1Campus展示國小-家長
  * `dev.teacher02@1campus.net`：1Campus展示國小-班導師/授課教師/家長
  * `dev.teacher03@1campus.net`：1Campus展示國中-授課教師、1Campus展示國小-授課教師
  * `dev.j.s20101@1campus.net`：1Campus展示國中-201班01號學生
  * `dev.j.s20102@1campus.net`：1Campus展示國中-201班02號學生
  * `dev.p.s50101@1campus.net`：1Campus展示國小-501班01號學生
  * `dev.p.s50102@1campus.net`：1Campus展示國小-501班02號學生
  * 更多學生帳號可通過 `getClassStudent` 查詢。

**操作身分
操作身分是指使用者在特定情境下的身份，與帳號不同。
一個帳號可以擁有多個操作身分。
舉例來說，一位教師可能在多所學校授課；一位家長可能有多個子女在不同學校就讀，甚至在同一學校有多個子女；一位教師可能同時也是家長。這時候，僅識別帳號無法完全呈現使用者的情境，因此需要進一步區分操作身分。
假如一位家長想查看二兒子的學習成果，透過操作身分，系統可以精確地呈現二兒子的資料，而不需用戶進行繁瑣的切換與查詢。

## 班群資料API (jasmine)
班群資料 API 解決了複雜的成班流程問題，讓您一鍵完成班群資料的建立。
### 模擬測試
* 使用展示專用服務進行測試：
  * client_id：`edf96e2da7a4f156f1df52f07ab3490f`
  * client_secret：`84aff4ae1841be51b6dcdd41546e3ad1eaf9b2d05e8240050753ac5fdddf3940`
* [操作影片1](https://www.youtube.com/watch?v=KcM4zlmJJWE)
* [操作影片2](https://www.youtube.com/watch?v=pHtdEMvZ_h8)

1. 為您的系統註冊 clientID [開啟](https://auth.ischool.com.tw/1campus/manage/)
  * 備註：如果您尚未開始開發，可以先跳過此步驟。
2. 使用 OAuth Client 取得 `access_token`。
  * 備註：跳過步驟1者，可使用展示專用服務的 `client_id` 和 `client_secret` 進行測試。
3. 點擊 Service 旁的鎖頭圖示（開鎖狀態），會彈出輸入視窗。
4. 輸入 `access_token` 後，點擊「Authorize」儲存，再點擊「Close」關閉。
5. 此時 Service 旁的鎖頭圖示會變為關閉狀態，表示已輸入授權。
6. 填入參數，並呼叫 Service。

### 資料授權
班群資料 API 使用 OAuth 認證機制。應用系統需先註冊並由 1Campus 平台開通使用權限，然後透過 client_credentials 流程獲得 access_token 來進行資料呼叫。
資料的存取範圍取決於 1Campus 平台授權的學校清單，這需要業務端與學校進行授權協商。如果尚未獲得學校授權，可使用展示資料庫中的假資料進行開發。

* 註冊 OAuth Client
* 開通使用權限

**API授權與目前使用者帳號
`jasmine` API 的授權方式為學校直接授權給應用系統，存取資料時使用的是應用系統的身分，而非當前使用者帳號。當前使用者帳號可作為查詢條件之一。

**可存取學校
學校授權清單是由 1Campus 平台管理的，只有獲得授權的應用系統才能查詢對應的學校資料。不同的應用系統可能會取得不同的學校清單，具體取決於學校授權。

**識別欄位
* `teacherAcc`、`studentAcc`
* `teacherID`、`studentID`

**班級(class) VS 課程(course)
* 班級和課程是兩個不同的概念。
* 班級通常指的是學校的學年度內由一群學生組成的班級，而課程則是教師所授課程。
* 學期轉換時，班級和課程的資料應根據實際情況進行更新。

Base URL：`https://devapi.1campus.net`

完整的互動式 API 規格請參考 [Swagger 文件](https://devapi.1campus.net/doc/)。

---

## GET /{schoolDsns}/identity/{code} — 取得目前登入帳號、操作身分、及基本資料

取得目前使用者帳號進行登入，如果此帳號有多個身分，應該開啟傳入的指定身分進行使用

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |
| code | path | string | 是 | 1Campus平台產生的一次性使用代碼，30秒後失效（例如 `8d7a5ce1-1d53-47b6-9ea8-46dc8c3ce000`） |

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
| teacherSourceIndex | string | 教師sourceIndex |
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

## GET /api/jasmine/getUserRole — 取得指定帳號在學校的身分

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| account | query | string | 否 | 要查詢的使用者帳號 |
| idNumberHash | query | string | 否 | 要查詢的使用者身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號))) |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| school | array of object |  |

#### teacherRole 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| teacherID | integer (int64) | 教師的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| teacherSourceIndex | string | 教師SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| teacherName | string | 教師的姓名 |
| teacherNo | string | 教師編號 |
| gender | string | 教師性別，scope需要包含 jasmine.profile 才會顯現 |
| idNumberHash | string | 教師身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，scope需要包含 jasmine.idNumberHash 才會顯現 |
| tag | array of string | 教師的類別，scope需要包含 jasmine.teacherTag 才會顯現 |
| position | array of object | 教師擔任的職務，scope需要包含 jasmine.teacherPosition 才會顯現 |
| systemPremission | array of string | 系統權限，僅列出與此系統相關的權限 |

#### studentRole 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| classID | integer (int64) | 學生所屬班級的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| classSourceIndex | string | 班級SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| gradeYear | integer (int32) | 學生所屬班級的年級 |
| className | string | 學生所屬班級的班級名稱 |
| classNo | string | 學生所屬班級的班級編號 |
| studentID | integer (int64) | 學生的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| studentSourceIndex | string | 學生SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| studentNumber | string | 學生的學號 |
| studentName | string | 學生的姓名 |
| seatNo | integer (int32) | 學生的座號 |
| gender | string | 學生性別，scope需要包含 jasmine.profile 才會顯現 |
| birthdate | string | 學生生日，scope需要包含 jasmine.profile 才會顯現 |
| idNumberHash | string | 學生身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，scope需要包含 jasmine.idNumberHash 才會顯現 |
| status | string | 學生狀態，scope需要包含 jasmine.profile 才會顯現 |

#### school 陣列元素 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| schoolDsns | string | 學校主機唯一識別 |
| schoolOfficialName | string | 學校正式名稱 |
| schoolName | string | 學校名稱 |
| schoolCode | string | 學校代碼 |
| schoolType | string | 學校類型(國小、國中、高中) |
| schoolYear | integer (int32) | 學校目前學年度 |
| semester | integer (int32) | 學校目前學期 |
| isTeacher | boolean | 是否具教師身分 |
| isStudent | boolean | 是否具學生身分 |
| isParent | boolean | 是否具家長身分 |
| teacherRole | object | 教師身分資料 |
| studentRole | object | 學生身分資料 |
| parentRole | array of object | 家長身分資料 |

---

## GET /api/jasmine/getSchool — 取得學校資料

查詢應用系統被授權的學校清單與基本資料，包含學校名稱、學校代碼、學校類型、以及該校目前的學年度（schoolYear）與學期（semester）。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | query | string | 否 | 要查詢的Dsns |
| schoolCode | query | string | 否 | 要查詢的學校代碼 |
| schoolType | query | string | 否 | 要查詢的學校類型(國小、國中、高中) |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| school | array of object | 查詢學校基本資料 |

#### school 陣列元素 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| schoolDsns | string | 學校主機唯一識別 |
| schoolOfficialName | string | 學校正式名稱 |
| schoolName | string | 學校名稱 |
| schoolCode | string | 學校代碼 |
| schoolType | string | 學校類型(國小、國中、高中) |
| schoolYear | integer (int32) | 學校目前學年度 |
| semester | integer (int32) | 學校目前學期 |

---

## GET /api/jasmine/{schoolDsns}/getTeacher — 取得教師資料

可傳入參數用老師帳號、教師ID直接查詢，或者不傳參數取回全校所有教師

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |
| teacherAcc | query | string | 否 | 使用指定教師帳號，查詢此教師 |
| teacherID | query | int64 | 否 | 使用指定教師ID，查詢此教師 |
| idNumberHash | query | string | 否 | 要查詢的教師身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號))) |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| teacher | array of object |  |

#### teacher 陣列元素 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| schoolDsns | string | 學校主機唯一識別 |
| schoolOfficialName | string | 學校正式名稱 |
| schoolName | string | 學校名稱 |
| schoolCode | string | 學校代碼 |
| schoolType | string | 學校類型(國小、國中、高中) |
| teacherID | integer (int64) | 教師的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| teacherSourceIndex | string | 教師SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| teacherNo | string | 教師編號 |
| teacherName | string | 教師的姓名 |
| gender | string | 教師性別，scope需要包含 jasmine.profile 才會顯現 |
| idNumberHash | string | 教師身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，scope需要包含 jasmine.idNumberHash 才會顯現 |
| teacherAcc | string | 教師帳號 |
| tag | array of string | 教師的類別，scope需要包含 jasmine.teacherTag 才會顯現 |
| position | array of object | 教師擔任的職務，scope需要包含 jasmine.teacherPosition 才會顯現 |
| systemPremission | array of string | 系統權限，僅列出與此系統相關的權限 |

---

## GET /api/jasmine/{schoolDsns}/getClass — 取得班級及班導師

可傳入參數用老師帳號、學生帳號、或班級ID直接查詢，或者不傳參數取回全校所有班級

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |
| teacherAcc | query | string | 否 | 使用指定教師帳號，查詢此教師為班導師或副班導師的班級 |
| teacherID | query | int64 | 否 | 使用指定教師ID，查詢此教師為班導師或副班導師的班級 |
| teacherIDNumberHash | query | string | 否 | 使用指定教師身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，查詢此教師為班導師或副班導師的班級 |
| studentAcc | query | string | 否 | 使用指定學生帳號，查詢此學生為所屬的班級。 |
| studentID | query | int64 | 否 | 使用指定學生ID，查詢此學生為所屬的班級。 |
| studentIDNumberHash | query | string | 否 | 使用指定學生身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，查詢此學生所屬的班級。 |
| classID | query | int64 | 否 | 使用classID查詢指定班級 |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| class | array of object | 查詢班級及學生資料 |

#### department 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| departmentID | integer (int64) | 科別的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| departmentName | string | 科別的科別名稱 |
| departmentCode | string | 科別的科別代碼 |

#### teacher 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| teacherID | integer (int64) | 班導師的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| teacherSourceIndex | string | 班導師SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| teacherNo | string | 班導師編號 |
| teacherName | string | 班導師姓名 |
| gender | string | 班導師性別，scope需要包含 jasmine.profile 才會顯現 |
| idNumberHash | string | 班導師身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，scope需要包含 jasmine.idNumberHash 才會顯現 |
| teacherAcc | string | 班導師帳號 |

#### secondaryTeacher 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| teacherID | integer (int64) | 副班導師的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| teacherSourceIndex | string | 副班導師SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| teacherNo | string | 副班導師編號 |
| teacherName | string | 副班導師姓名 |
| gender | string | 副班導師性別，scope需要包含 jasmine.profile 才會顯現 |
| idNumberHash | string | 副班導師身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，scope需要包含 jasmine.idNumberHash 才會顯現 |
| teacherAcc | string | 副班導師帳號 |

#### class 陣列元素 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| schoolDsns | string | 學校主機唯一識別 |
| schoolOfficialName | string | 學校正式名稱 |
| schoolName | string | 學校名稱 |
| schoolCode | string | 學校代碼 |
| schoolType | string | 學校類型(國小、國中、高中) |
| schoolYear | integer (int32) | 學校目前學年度 |
| semester | integer (int32) | 學校目前學期 |
| classID | integer (int64) | 班級的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| classSourceIndex | string | 班級SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| gradeYear | integer (int32) | 班級的年級 |
| className | string | 班級的班級名稱 |
| classNo | string | 班級的班級編號 |
| department | object | 班級的科別資料，scope需要包含 jasmine.department 才會顯現 |
| teacher | object |  |
| secondaryTeacher | object | 副班導師資料 |

---

## GET /api/jasmine/{schoolDsns}/getClassStudent — 取得班級及學生資料

可傳入參數用老師帳號、學生帳號、或班級ID直接查詢，或者不傳參數取回全校所有班級及學生

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |
| teacherAcc | query | string | 否 | 使用指定教師帳號，查詢此教師為班導師或副班導師的班級 |
| teacherID | query | int64 | 否 | 使用指定教師ID，查詢此教師為班導師或副班導師的班級 |
| teacherIDNumberHash | query | string | 否 | 使用指定教師身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，查詢此教師為班導師或副班導師的班級 |
| studentAcc | query | string | 否 | 使用指定學生帳號，查詢此學生為所屬的班級。當指定學生時，只會回傳此學生資料而非全班學生。 |
| studentID | query | int64 | 否 | 使用指定學生ID，查詢此學生為所屬的班級。當指定學生時，只會回傳此學生資料而非全班學生。 |
| studentIDNumberHash | query | string | 否 | 使用指定學生身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，查詢此學生所屬的班級。當指定學生時，只會回傳此學生資料而非全班學生。 |
| classID | query | int64 | 否 | 使用classID查詢指定班級 |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| class | array of object | 查詢班級及學生資料 |

#### department 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| departmentID | integer (int64) | 科別的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| departmentName | string | 科別的科別名稱 |
| departmentCode | string | 科別的科別代碼 |

#### teacher 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| teacherID | integer (int64) | 班導師的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| teacherSourceIndex | string | 班導師SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| teacherNo | string | 班導師編號 |
| teacherName | string | 班導師姓名 |
| gender | string | 班導師性別，scope需要包含 jasmine.profile 才會顯現 |
| idNumberHash | string | 班導師身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，scope需要包含 jasmine.idNumberHash 才會顯現 |
| teacherAcc | string | 班導師帳號 |

#### secondaryTeacher 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| teacherID | integer (int64) | 副班導師的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| teacherSourceIndex | string | 副班導師SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| teacherNo | string | 副班導師編號 |
| teacherName | string | 副班導師姓名 |
| gender | string | 副班導師性別，scope需要包含 jasmine.profile 才會顯現 |
| idNumberHash | string | 副班導師身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，scope需要包含 jasmine.idNumberHash 才會顯現 |
| teacherAcc | string | 副班導師帳號 |

#### class 陣列元素 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| schoolDsns | string | 學校主機唯一識別 |
| schoolOfficialName | string | 學校正式名稱 |
| schoolName | string | 學校名稱 |
| schoolCode | string | 學校代碼 |
| schoolType | string | 學校類型(國小、國中、高中) |
| schoolYear | integer (int32) | 學校目前學年度 |
| semester | integer (int32) | 學校目前學期 |
| classID | integer (int64) | 班級的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| classSourceIndex | string | 班級SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| gradeYear | integer (int32) | 班級的年級 |
| className | string | 班級的班級名稱 |
| classNo | string | 班級的班級編號 |
| department | object | 班級的科別資料，scope需要包含 jasmine.department 才會顯現 |
| teacher | object |  |
| secondaryTeacher | object | 副班導師資料 |
| student | array of object | 班級學生資料 |

---

## GET /api/jasmine/{schoolDsns}/getCourse — 取得課程資料

可傳入參數用老師帳號、或課程ID直接查詢，或者不傳參數取回全校所有課程

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |
| teacherAcc | query | string | 否 | 使用指定教師帳號，查詢此教師教授的課程 |
| teacherID | query | int64 | 否 | 使用指定教師ID，查詢此教師教授的課程 |
| teacherIDNumberHash | query | string | 否 | 使用指定教師身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，查詢此教師教授的課程 |
| studentAcc | query | string | 否 | 使用指定學生帳號，查詢此學生修習的課程 |
| studentID | query | int64 | 否 | 使用指定學生ID，查詢此學生修習的課程 |
| studentIDNumberHash | query | string | 否 | 使用指定學生身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，查詢此學生修習的課程 |
| courseID | query | int64 | 否 | 使用courseID查詢指定課程 |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| course | array of object |  |

#### class 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| classID | integer (int64) | 班級的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| classSourceIndex | string | 班級SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| gradeYear | integer (int32) | 班級的年級 |
| className | string | 班級的班級名稱 |
| classNo | string | 班級的班級編號 |
| teacher | object | 班導師資料 |
| secondaryTeacher | object | 副班導師資料 |

#### course 陣列元素 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| schoolDsns | string | 學校主機唯一識別 |
| schoolOfficialName | string | 學校正式名稱 |
| schoolName | string | 學校名稱 |
| schoolCode | string | 學校代碼 |
| schoolType | string | 學校類型(國小、國中、高中) |
| schoolYear | integer (int32) | 學校目前學年度 |
| semester | integer (int32) | 學校目前學期 |
| courseID | integer (int64) | 課程的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| courseSourceIndex | string | 課程SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| courseName | string | 課程名稱 |
| subject | string | 課程科目 |
| subjectLevel | string | 課程科目級別 |
| class | object | 開課班級 |
| teacher | array of object | 授課教師資料 |

---

## GET /api/jasmine/{schoolDsns}/getCourseStudent — 取得課程資料及修課學生

可傳入參數用老師帳號、或課程ID直接查詢，或者不傳參數取回全校所有課程及學生

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |
| teacherAcc | query | string | 否 | 使用指定教師帳號，查詢此教師教授的課程 |
| teacherID | query | int64 | 否 | 使用指定教師ID，查詢此教師教授的課程 |
| teacherIDNumberHash | query | string | 否 | 使用指定教師身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，查詢此教師教授的課程 |
| studentAcc | query | string | 否 | 使用指定學生帳號，查詢此學生修習的課程 |
| studentID | query | int64 | 否 | 使用指定學生ID，查詢此學生修習的課程 |
| studentIDNumberHash | query | string | 否 | 使用指定學生身分證號 SHA256 雜湊值，計算方式為 Sha256(Upper(Trim(身分證號)))，查詢此學生修習的課程 |
| courseID | query | int64 | 否 | 使用courseID查詢指定課程 |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| course | array of object |  |

#### class 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| classID | integer (int64) | 班級的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| classSourceIndex | string | 班級SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| gradeYear | integer (int32) | 班級的年級 |
| className | string | 班級的班級名稱 |
| classNo | string | 班級的班級編號 |
| teacher | object | 班導師資料 |
| secondaryTeacher | object | 副班導師資料 |

#### course 陣列元素 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| schoolDsns | string | 學校主機唯一識別 |
| schoolOfficialName | string | 學校正式名稱 |
| schoolName | string | 學校名稱 |
| schoolCode | string | 學校代碼 |
| schoolType | string | 學校類型(國小、國中、高中) |
| schoolYear | integer (int32) | 學校目前學年度 |
| semester | integer (int32) | 學校目前學期 |
| courseID | integer (int64) | 課程的系統編號，此系統編號在同一學校主機(schoolDsns)中唯一 |
| courseSourceIndex | string | 課程SourceIndex，scope需要包含 jasmine.sourceIndex 才會顯現 |
| courseName | string | 課程名稱 |
| subject | string | 課程科目 |
| class | object | 開課班級 |
| teacher | array of object | 授課教師資料 |
| student | array of object | 修課學生資料 |

---

## POST /api/jasmine/{schoolDsns}/getTeacherDeparted — 清查已離職教師

傳入已知的teacherID、或sourceIndex，回傳已離職教師清單

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| teacherID | array of integer | 否 | 教師系統編號 |
| teacherSourceIndex | array of string | 否 | 教師sourceIndex |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## POST /api/jasmine/{schoolDsns}/getStudentDeparted — 清查非在校學生

傳入已知的studentID、sourceIndex，回傳非在校學生清單(休學中的學生亦會視為非在校，復學後會再加回來並使用同studentID)

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 指定學校的主機名稱（例如 `j.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| studentID | array of integer | 否 | 學生系統編號 |
| studentSourceIndex | array of string | 否 | 學生sourceIndex |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限
