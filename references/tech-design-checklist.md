# 技術設計品質檢查清單

在生成或審閱技術設計文件時，逐項確認以下清單。

---

## 系統架構

- [ ] 每個技術選型都有明確的「選擇這個而非其他的理由」
- [ ] 架構符合使用者的成本預算（免費方案不建議付費服務）
- [ ] 架構符合部署環境（本機 / 內部 / 對外公開）
- [ ] 沒有過度工程化（不需要微服務就別用微服務）
- [ ] 技術棧清單完整（前端、後端、資料庫、部署、Email 等）

**免費方案可選技術棧參考：**

| 類型 | 免費選項 |
|------|---------|
| 後端 | Node.js/Express、Python/FastAPI、Go |
| 前端 | React (Vite)、Vue、Svelte |
| 資料庫 | SQLite（本機）、Supabase（500MB）、PlanetScale（5GB） |
| 部署-前端 | Vercel、Netlify、GitHub Pages |
| 部署-後端 | Railway（500h/mo）、Render（750h/mo）、Fly.io |
| Email | Resend（3,000 封/mo）、Brevo（300 封/天） |
| 認證 | express-session、Lucia auth、Better Auth |

---

## 資料流程

- [ ] 描述了至少一個正常流程（happy path）
- [ ] 描述了至少一個異常流程（error case）
- [ ] 步驟是有序的（不是概念描述，而是 1→2→3 的序列）
- [ ] 明確指出資料驗證發生的位置（前端 / 後端 / DB）
- [ ] 說明了外部服務呼叫的時機（Email、Payment 等）

---

## 資料模型

- [ ] 每個主要實體都有對應的表/集合
- [ ] 關鍵欄位有型別與用途說明
- [ ] 有說明實體之間的關聯（一對多、多對多）
- [ ] 有說明哪些欄位需要索引（高頻查詢的欄位）
- [ ] 敏感欄位（密碼、token）有說明儲存方式（hash，不存明文）

---

## 錯誤處理策略

- [ ] 前後端分層描述（不是只說「顯示錯誤」）
- [ ] HTTP 狀態碼使用有規範（400/401/403/404/422/429/500）
- [ ] 錯誤回應格式統一（例如 `{ error: { code, message } }`）
- [ ] 不向使用者暴露系統內部細節（stack trace、SQL 錯誤）
- [ ] 有說明哪些操作可以重試（冪等性考量）

**HTTP 狀態碼速查：**

| 碼 | 語意 | 使用時機 |
|----|------|---------|
| 400 | Bad Request | 輸入格式錯誤、缺少必要欄位 |
| 401 | Unauthorized | 未登入、Token 無效/過期 |
| 403 | Forbidden | 已登入但無權限 |
| 404 | Not Found | 資源不存在 |
| 409 | Conflict | 資源衝突（如 Email 已存在） |
| 422 | Unprocessable | 格式正確但業務邏輯不允許 |
| 429 | Too Many Requests | 超過限流 |
| 500 | Server Error | 未預期的伺服器錯誤 |

---

## 資訊安全

- [ ] 認證機制明確（Session / JWT / OAuth），並說明選擇理由
- [ ] 密碼雜湊演算法明確（bcrypt cost ≥ 10，或 argon2id）
- [ ] 輸入驗證在前端 + 後端都有（不只在前端）
- [ ] SQL 使用 parameterized queries（防 injection）
- [ ] 敏感資料傳輸加密（HTTPS）
- [ ] Session/Cookie 設定：httpOnly、secure、sameSite
- [ ] 有說明至少 3 個 OWASP Top 10 對應措施

**OWASP Top 10 快查（2021）：**

| 編號 | 風險 | 常見對應措施 |
|------|------|------------|
| A01 | 存取控制失效 | 每個 API 都驗證權限，不只靠前端隱藏 |
| A02 | 加密失效 | HTTPS、bcrypt、敏感欄位加密 |
| A03 | 注入攻擊 | Parameterized queries、輸入驗證 |
| A04 | 不安全設計 | 威脅建模、最小權限原則 |
| A05 | 安全設定錯誤 | 關閉預設帳號、設定安全 headers |
| A06 | 過時元件 | 定期更新依賴、使用 npm audit |
| A07 | 認證失敗 | 帳號鎖定、強密碼政策、MFA |
| A08 | 軟體完整性失效 | 驗證第三方套件來源 |
| A09 | 日誌/監控不足 | 記錄認證事件、異常告警 |
| A10 | SSRF | 驗證外部 URL、限制內網存取 |

---

## QA 策略

- [ ] 有分層（單元 / 整合 / E2E），不是全部都說「寫測試」
- [ ] 單元測試：指出具體要測的函式/模組
- [ ] 整合測試：列出要測的 API 端點與情境
- [ ] E2E 測試：對應到 User Stories 的 Scenarios
- [ ] 推薦的測試工具與技術棧一致
- [ ] CI 整合方式明確（PR 時跑測試、門檻設定）

**測試工具選擇參考：**

| 技術棧 | 單元測試 | 整合測試 | E2E |
|--------|---------|---------|-----|
| Node.js | Vitest / Jest | Supertest + Vitest | Playwright |
| Python | pytest | pytest + httpx | Playwright |
| React | Vitest + Testing Library | MSW + Vitest | Playwright |

---

## 維運考量

- [ ] Log 欄位具體（不是「記錄 Log」）
- [ ] Log 不包含敏感資訊（密碼、token、信用卡號）
- [ ] 監控指標具體（回應時間 P95、錯誤率、業務指標）
- [ ] 告警閾值有數字（不是「過高時告警」）
- [ ] 資料庫備份策略明確（頻率、保留期）
- [ ] 版本管理方式明確（Schema migration、rollback 步驟）

**Log 必要欄位：**
```json
{
  "timestamp": "ISO 8601",
  "level": "info|warn|error",
  "requestId": "trace ID",
  "userId": "已登入時",
  "ip": "客戶端 IP",
  "method": "GET",
  "path": "/api/...",
  "statusCode": 200,
  "duration_ms": 45
}
```

**禁止進 Log 的欄位：**
- password, password_hash
- token, access_token, refresh_token
- credit_card, cvv
- secret, api_key
