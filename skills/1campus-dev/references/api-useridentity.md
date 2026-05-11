# UserIdentity API（帳號識別資訊 API）規格

UserIdentity API 提供帳號識別資訊相關功能。所有需授權的端點皆需透過 OAuth Bearer Token 進行驗證。

Base URL：`https://devapi.1campus.net`

完整的互動式 API 規格請參考 [Swagger 文件](https://devapi.1campus.net/doc/)。

---

## POST /api/userIdentity/lookup — 查詢帳號識別資訊

上傳指定帳號清單，查詢設定的識別資訊

### Request Body

| 欄位名稱 | 類型 | 必填 | 說明 |
|----------|------|------|------|
| type | string | 是 | 識別資訊類型 |
| account | array of string | 是 | 帳號清單 |

### 範例

```json
{
  "type": "twOIDC"
}
```

### 錯誤回應

- `400`：傳入參數錯誤
- `401`：認證失敗或無權限
