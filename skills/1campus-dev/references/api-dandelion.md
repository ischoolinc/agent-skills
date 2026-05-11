# Dandelion API（訊息推播 API）規格

發送針對特定老師、學生和家長的推播訊息。

**訊息內容
在APP上呈現的訊息包括以下欄位：categoryName、title、body。categoryName用來分類訊息，而title和body則分別為訊息的標題和內容。

**訊息內容關鍵字自動替換
在title和body欄位中，可以使用以下關鍵字，自動替換為收件者的相關資訊：${學生年級}、${學生班級}、${學生座號}、${學生學號}、${學生姓名}、${家長姓名}、${家長稱謂}。

**收件者
收件者（receiver）可依teacherID、studentID、classID、courseID推播給指定對象，receiver為陣列，表示訊息可同時指定傳給多個收件者。在大量的通知情境中，可以搭配classID以及關鍵字自動替換，一次發給全校所有班級的學生、家長。
當發送給家長時，是以學生為單位，因此receiver中的studentID為必填欄位，並且toStudent、toParent至少要有一個為true，表示要發送給學生本人或是家長。此時系統只需管理學生，不需管理家長帳號。
假如發送給家長時，這個學生沒有任何家長綁定，系統會發送一則空白訊息。在管理端可以看到這則推播要給這學生的家長，但因為沒有家長綁定的關係，所以沒有任何人收到。

**排程發送
排程時間（scheduleTime）情境說明：某些訊息可能需要仰賴資料匯算，需要行政端的操作和確認。此時，可以使用排程發送功能，在特定時間發送訊息，解決行政端操作時間和期望發送時間不一致的情況。

以上是推播訊息服務的基本概念，希望這份文件能夠幫助你更清楚地使用我們的API服務。如果有任何問題，請隨時聯絡我們。

Base URL：`https://devapi.1campus.net`

完整的互動式 API 規格請參考 [Swagger 文件](https://devapi.1campus.net/doc/)。

---

## POST /api/dandelion/{schoolDsns}/push — 發送訊息

將訊息發送給指定的老師學生、班級、或是全校

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `j.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| categoryName | string | 是 | 訊息的群組名稱，APP將依此名稱分類訊息。 |
| title | string | 是 | 訊息標題，可利用以下關鍵字，自動替換帶入收件者資訊：${學生年級}、${學生班級}、${學生座號}、${學生學號}、{學生姓名}、${家長姓名}、${家長稱謂} |
| body | string | 是 | 訊息內容，可利用以下關鍵字，自動替換帶入收件者資訊：${學生年級}、${學生班級}、${學生座號}、${學生學號}、{學生姓名}、${家長姓名}、${家長稱謂} |
| scheduleTime | string | 否 | 排程發送時間，格式為 yyyy/MM/dd HH:mm:ss，例如 2020/01/01 12:00:00。如果不傳入此欄位，則會立即發送。 |
| receiver | array of object | 否 |  |

#### receiver 陣列元素 物件欄位

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| type | string | 是 | 收件者類型，可選擇 teacher、student、class、course 四種 teacher：指定teacherID清單 student：指定studentID清單、toStudent、toParent class：指定classID、toTeacher、toStudent、toParent course：指定courseID、toStudent、toParent |
| teacherID | array of integer | 否 | type=teacher時，傳入收件者教師的系統編號 |
| teacherSourceIndex | array of string | 否 | type=teacher時，傳入收件者教師的SourceIndex |
| studentID | array of integer | 否 | type=student時，傳入收件者學生的系統編號 |
| studentSourceIndex | array of string | 否 | type=student時，傳入收件者學生的SourceIndex |
| idNumberHash | array of string | 否 | type=student時，傳入收件者學生的身分證字號的Hash值type=teacher時，傳入收件者教師的身分證字號的Hash值。身分證號Hash值計算方式為Sha256(Upper(Trim(身分證號))) |
| classID | integer (int64) | 否 | type=class時，傳入收件者班級的系統編號 |
| courseID | integer (int64) | 否 | type=course時，傳入收件者課程的系統編號 |
| toTeacher | boolean | 否 | type=class、type=student時，指定是否發送給班級的班導師 |
| toStudent | boolean | 否 | type=student、class、course 時，指定是否發送給對應的學生本人 |
| toParent | boolean | 否 | type=student、class、course 時，指定是否發送給對應的學生家長 |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限

---

## GET /api/dandelion/{schoolDsns}/history/list — 查看發送紀錄

查看發送紀錄，可以查看發送的訊息接收統計等資訊。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `j.demo.1campus.net`） |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限
- `500`：伺服器錯誤

---

## GET /api/dandelion/{schoolDsns}/history/detail/{uuid} — 查看發送紀錄

查看發送紀錄，可以查看發送的訊息接收統計等資訊。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `j.demo.1campus.net`） |
| uuid | path | string | 是 | 訊息唯一識別（例如 `12345678-1234-1234-1234-123456789012`） |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| uuid | string | 訊息唯一識別碼 |
| sender | object |  |
| categoryName | string | 訊息的群組名稱 |
| title | string | 訊息標題 |
| body | string | 訊息內容 |
| scheduleTime | string | 排程發送時間 |
| receiverCount | integer | 收件者數量 |
| sendCount | integer | 發送數量 |
| readCount | integer | 已讀數量 |
| lastUpdateTime | string | 訊息內容的最後更新時間 |
| receiverDetail | array of object |  |

#### sender 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| clientID | string | 發送者的clientID |
| clientName | string | 發送者的名稱 |

#### receiverDetail 陣列元素 物件欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| receiverAccount | string | 收件者帳號 |
| receiverUUID | string | 收件者唯一識別碼 |
| sendTime | string | 推播發送時間 |
| readTime | string | 收件者讀取時間 |
| schoolDsns | string | 學校主機唯一識別 |
| schoolCode | string | 學校代碼 |
| schoolName | string | 學校名稱 |
| schoolType | string | 學校類型 |
| roleType | string | 角色類型 |
| refStudentID | integer (int64) | 收件者在學校的學生系統編號 |
| refTeacherID | integer (int64) | 收件者在學校的教師系統編號 |
| refParentID | integer (int64) | 收件者在學校的家長系統編號 |
| gradeYear | integer | 收件者的年級 |
| className | string | 收件者的班級 |
| seatNo | integer | 收件者的座號 |
| studentNumber | string | 收件者的學號 |
| studentName | string | 學生姓名 |
| teacherName | string | 教師姓名 |
| parentName | string | 家長姓名 |
| parentRelationship | string | 家長稱謂 |
| customizedTitle | string | 實際發送給此收件者的標題 |
| customizedBody | string | 實際發送給此收件者的內容 |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限
- `500`：伺服器錯誤

---

## GET /api/dandelion/admin/{schoolDsns}/init — 初始化管理者工具

初始化管理者工具，使用此API可以取得一個網址，透過此網址可以進入管理者工具，此管理者工具可以查看您的clientID發送的訊息，查看的結果亦同步顯示在學校的管理者工具中。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| schoolDsns | path | string | 是 | 學校主機唯一識別（例如 `j.demo.1campus.net`） |

### Response 欄位

| 欄位名稱 | 類型 | 說明 |
|----------|------|------|
| redirectUrl | string | 啟動管理者工具的網址，此網址會導向到管理者工具的首頁，此管理者工具僅會顯示您的clientID發送的訊息，不會顯示來自其他clientID發送的訊息。 |

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限
- `500`：伺服器錯誤
