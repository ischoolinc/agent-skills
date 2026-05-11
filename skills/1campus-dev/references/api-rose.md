# Rose API（排調代課系統 API）規格

使排課、調代課系統可與校務系統整合。
...

Base URL：`https://devapi.1campus.net`

完整的互動式 API 規格請參考 [Swagger 文件](https://devapi.1campus.net/doc/)。

---

## GET /api/rose/{dsns}/getTeacher — 取得教師資料

取回全校所有在職教師

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `h.demo.1campus.net`） |

---

## GET /api/rose/{dsns}/getClassroom — 取得上課場地資料

取回全校所有上課場地

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `h.demo.1campus.net`） |

---

## GET /api/rose/{dsns}/getPreClass — 取得預開課班級資料

取回指定學期的預開課班級資料

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `h.demo.1campus.net`） |
| schoolYear | query | integer | 是 | 學年度（例如 `111`） |
| semester | query | integer | 是 | 學期（例如 `2`） |

---

## GET /api/rose/{dsns}/getPreCourse — 取得預開課課程資料

取回指定學期的預開課課程資料

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `h.demo.1campus.net`） |
| schoolYear | query | integer | 是 | 學年度（例如 `111`） |
| semester | query | integer | 是 | 學期（例如 `2`） |
| courseType | query | string | 否 | 篩選回傳開課類別，原班 \| 跨班，如果是選課系統使用時，可以篩選跨班課程作為選課課程清單，不需要將所有原班課程也列出來。 |
| source | query | string | 否 | 篩選系統名稱，排課 \| 選課 |

---

## GET /api/rose/{dsns}/getPreCourseStudent — 取得預開課課程及修課學生資料

取回指定學期的預開課課程資料，以及指定的修課學生，僅有跨班課程會有來自選課系統寫入的修課學生資料，原班課程直接使用班級學生，不會額外指定修課學生資料。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `h.demo.1campus.net`） |
| schoolYear | query | integer | 是 | 學年度（例如 `111`） |
| semester | query | integer | 是 | 學期（例如 `2`） |
| courseType | query | string | 否 | 篩選回傳開課類別，原班 \| 跨班，如果是選課系統使用時，可以篩選跨班課程作為選課課程清單，不需要將所有原班課程也列出來。 |
| source | query | string | 否 | 篩選系統名稱，排課 \| 選課 |

---

## GET /api/rose/{dsns}/getPreCourseWeeklySchedule — 取得預開課課程排課週課表資料

取回指定學期的預開課課程資料，以及排課週課表資料。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `h.demo.1campus.net`） |
| schoolYear | query | integer | 是 | 學年度（例如 `111`） |
| semester | query | integer | 是 | 學期（例如 `2`） |
| courseType | query | string | 否 | 篩選回傳開課類別，原班 \| 跨班，如果是選課系統使用時，可以篩選跨班課程作為選課課程清單，不需要將所有原班課程也列出來。 |
| source | query | string | 否 | 篩選系統名稱，排課 \| 選課 |

---

## GET /api/rose/{dsns}/getCourseDailySchedule — 取得開課課程實際課表資料

取回指定學期的開課課程資料，以及實際課表資料。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `h.demo.1campus.net`） |
| schoolYear | query | integer | 是 | 學年度（例如 `111`） |
| semester | query | integer | 是 | 學期（例如 `2`） |
| courseType | query | string | 否 | 篩選回傳開課類別，原班 \| 跨班，如果是選課系統使用時，可以篩選跨班課程作為選課課程清單，不需要將所有原班課程也列出來。 |
| source | query | string | 否 | 篩選系統名稱，排課 \| 選課 |

---

## POST /api/rose/{dsns}/setClassroom — 寫入場地資料

自動依classroomNo新增或更新場地清單

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `h.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|

### 錯誤回應

- `401`：認證失敗或無權限

---

## POST /api/rose/{dsns}/setPreCourse — 寫入預開課課程資訊

自動依schoolYear、semester、classNo、subjectName判斷新增或更新預開課課程資訊

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `h.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|

### 錯誤回應

- `401`：認證失敗或無權限

---

## POST /api/rose/{dsns}/setPreCourseStudent — 寫入預開課修課學生資訊

自動依schoolYear、semester、classNo、subjectName判斷預開課課程並將修課學生指定成傳入學生清單

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `h.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|

### 錯誤回應

- `401`：認證失敗或無權限

---

## POST /api/rose/{dsns}/setPreCourseWeeklySchedule — 寫入預開課排課週課表資訊

自動依schoolYear、semester，重新建立完整的預開課程週課表資訊，呼叫此API時，需同時傳入整學期的週課表

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `h.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|

### 錯誤回應

- `401`：認證失敗或無權限

---

## POST /api/rose/{dsns}/setCourseDailySchedule — 寫入實際課程日課表資訊

自動依schoolYear、semester、date同步指定學年度、學期、日期的日課表，呼叫時需傳入該日期整個課表資料，系統會自動判斷新增、更新日課表資料，並自動刪除該日期中沒有傳入的日課表資訊。成功比對到課程的資料會寫入daily_schedule資料表，無法比對的資料會寫入mismatch_daily_schedule資料表供管理者後續處理。

### 參數

| 名稱 | 位置 | 類型 | 必填 | 說明 |
|------|------|------|------|------|
| dsns | path | string | 是 | 指定學校的主機名稱（例如 `h.demo.1campus.net`） |

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|

### 錯誤回應

- `401`：認證失敗或無權限
