# agent-skills

iSchool 內部 Agent Skills 合集。收錄 1Campus 平台等產品的 API、SDK 與內部工具的 Skill 文件，可供各種支援 Skill 格式的 AI 工具（Claude Code、Cursor、Codex 等）載入使用，協助同仁與合作夥伴串接公司服務。

本合集遵循 [Agent Skills](https://agentskills.io/) 格式，由 [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) 推廣的開放規範。

---

## Skill 清單

### 1campus-storage-api

1Campus Storage Service 檔案儲存 API 參考。涵蓋 OAuth 認證、Upload Token 管理、檔案上傳/下載/換檔/刪除完整流程。

**Use when:**
- 串接 1Campus 儲存服務上傳/下載檔案
- 管理 upload token 生命週期（建立、更新、刪除）
- 處理檔案換檔（PUT）、刪除（DELETE）
- 排查 GCS signed URL 快取行為

### 1campus-dev

1Campus 平台 API 整合開發助手。涵蓋身分認證（Identity Code / OAuth SSO）、班群資料（Jasmine）、課表（Schedule）、點名（Rollcall）等 11 個 API 模組的串接指引，附測試帳號與 sandbox credentials。

**⭐ 重點功能：訊息推播（Dandelion）** — 透過 API 即可發送通知到 1Campus Next APP，支援針對學生／家長／教師、整班／整校的批次推播，並提供 `${學生姓名}`、`${家長稱謂}` 等個人化關鍵字自動替換，是教育服務最常用的整合功能。

**Use when:**
- 發送推播通知到 1Campus Next APP（作業提醒、活動通知、緊急公告等）
- 串接 1Campus 校園 API（取得學校、班級、學生、課表等）
- 整合 1Campus 身分認證（Identity Code 或 OAuth SSO）
- 申請 Scope 權限、評估資料機敏程度
- 服務上架到 1Campus 平台、規劃整合架構

### 1campus-gpt-doc

1Campus GPT V4 Client API 第三方開發者文件。涵蓋單階段／二階段 API、SSE 串流事件、Client Function Call、Context 模板、Skirk 嵌入整合等完整 Client 端使用方式。

**Use when:**
- 串接 1Campus GPT V4 API（單階段或二階段）
- 實作 SSE 事件處理（text/tool/reasoning/search 等）
- 開發 Client-side Function Call（AI 控制前端 UI）
- 處理 Context 模板與動態變數注入
- 嵌入 Skirk AI 助理到自家應用（iframe、入口 URL、LINE Bot）
- 設定 Skirk 認證模式（anonymous / identity_code / passthrough / code_exchange）

---

## 安裝方式

### 方式一：使用 `skills` CLI（推薦）

[`skills` CLI](https://www.npmjs.com/package/skills) 是 Vercel Labs 出的通用 skill 管理工具，支援多種 AI agent（Claude Code、Cursor 等）。

```bash
# 安裝整個合集
npx skills add ischoolinc/agent-skills

# 只安裝指定 skill
npx skills add ischoolinc/agent-skills --skill 1campus-storage-api

# 指定 agent（預設會自動偵測本機已安裝的 agent）
npx skills add ischoolinc/agent-skills --agent claude-code

# 全域安裝（user-level，所有專案共用）
npx skills add ischoolinc/agent-skills -g

# 查看 repo 內可用的 skill
npx skills add ischoolinc/agent-skills --list

# 列出已安裝的 skill
npx skills list

# 更新到最新版
npx skills update
```

### 方式二：手動安裝（不使用 CLI）

如果不想用 `skills` CLI，也可以直接 clone 後複製到對應 AI 工具的 skills 目錄。以 Claude Code 為例：

```bash
git clone https://github.com/ischoolinc/agent-skills.git
cp -r agent-skills/skills/1campus-storage-api ~/.claude/skills/
```

或用 symlink 方便日後 `git pull` 同步：

```bash
git clone https://github.com/ischoolinc/agent-skills.git ~/src/agent-skills
ln -s ~/src/agent-skills/skills/1campus-storage-api ~/.claude/skills/1campus-storage-api
```

> 其他 AI 工具請依其文件指定的 skills 目錄路徑放置；只要該工具支援 `SKILL.md` 規格，本合集的 skill 即可直接使用。

安裝完成後重啟 AI 工具，輸入相關關鍵字即會自動觸發對應 skill。

---

## 貢獻指南

### 新增 Skill

1. 在 `skills/` 下建立新目錄：`skills/<product>-<skill-name>/`
   - 命名慣例：以**產品品牌**為前綴（如 `1campus-`），跨產品的通用工具用 `ischool-` 前綴
2. 撰寫 `SKILL.md`，frontmatter 範例：

   ```markdown
   ---
   name: 1campus-your-skill
   description: |
     一句話描述用途與觸發時機（這段會決定 AI 何時載入此 skill）
   license: CC-BY-ND-4.0
   metadata:
     author: 1campus
     organization: iSchool
     version: '1.0.0'
   ---

   # Skill 標題
   ...內容
   ```

3. （選用）新增 `metadata.json`：含 abstract、references 等補充資料
4. 更新本 README 的 Skill 清單
5. 開 PR 並請 reviewer 確認

### 從內部 Skill 複製過來時的檢查清單

許多 skill 原本是公司內部維護的版本（例如同名 repo 或本機 `~/.claude/skills/` 下的版本），複製到本合集前**必須**逐項檢查並清掃以下內容，避免內部資訊外流：

#### 🚨 一定要移除

- [ ] **內部 Skill 名稱引用**：如 `gpt-v4-arch`、`gpt-thread-inspector`、`gpt-knowledge-pack`、`gpt-schema` 等只有公司內部能用的 skill 名稱
- [ ] **私有 repo 連結**：指向 `ischoolinc/` 下 private repo 的 URL（除非該 repo 已公開）
- [ ] **內部資料庫 schema / 表名 / 欄位**：除非該結構已對外公開
- [ ] **正式環境 Credential**：production 環境的 API key、token、密碼、OAuth secret 絕對不能出現（即使是「只是範例」）。dev/sandbox 環境的測試 credential 若是平台**刻意公開供開發者試用**（如沙箱用 OAuth client、測試帳號密碼），可保留並明確標註為「測試用」
- [ ] **內部 IP、VPN 路徑、私有網址**：如 `10.x.x.x`、`*.internal`、未公開的服務 URL
- [ ] **本機路徑**：`/Users/xxx/...`、`~/Downloads/...` 這類個人開發機路徑
- [ ] **員工姓名、私密 email**：除非該人員同意公開掛名

#### 🤔 需要評估

- [ ] **內部工具指令**：只有公司同仁能用的 CLI 指令，要不要保留？若 skill 主要給外部用戶，這類指令應移除或改寫
- [ ] **內部規範引用**：如「依照 CLAUDE.md 規則」、「按照公司開發習慣」這類只有內部人員看得懂的話
- [ ] **過於詳細的內部架構**：若 skill 是 API 文件型，避免暴露後端實作細節（例如哪個 microservice 處理哪個請求）

#### ✅ 應保留

- ✅ API endpoint、參數、錯誤碼（這是公開資訊）
- ✅ curl 範例（用 `{YOUR_TOKEN}` 占位符代替實際 token）
- ✅ Skill 主體內容（除非觸及上述地雷）

### Skill 撰寫原則

- **保持工具中立**：避免硬綁特定 AI 工具的指令或路徑，讓 skill 能跨平台使用
- **description 寫清楚觸發情境**：列出關鍵字、使用場景，這影響 AI 自動載入的準確度
- **內容以實用為主**：API 文件型 skill 應提供完整的 endpoint、參數、範例、錯誤碼
- **範例用單行 curl**：避免反斜線換行，方便複製
- **大量補充資料分檔**：可在 skill 目錄下新增 `references/` 子目錄存放

---

## License

本合集採用 [Creative Commons Attribution-NoDerivatives 4.0 International (CC BY-ND 4.0)](./LICENSE) 授權。

- ✅ 可以散布、複製、商業使用
- ✅ 需保留版權聲明與授權連結
- ❌ 不可修改後再散布

> 注意：本合集授權的是「文件」本身；實際使用 API 仍需向 iSchool Inc. 申請對應的 OAuth 憑證與授權。
