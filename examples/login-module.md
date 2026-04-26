# 完整範例：登入模組規劃

**輸入需求**：
> 使用者可以用 Email + 密碼登入，連續失敗 5 次後鎖定 15 分鐘。
> 忘記密碼時可透過 Email 取得重設連結，連結 1 小時後失效。
> Admin 可以在後台查看登入記錄並停用帳號。

**技術偏好**：完全免費 / 個人本機 + 對外公開 / 長期維護

---

## 輸出一：User Stories

**Story S1：使用者密碼登入**
作為已註冊使用者，我想要用 Email + 密碼登入，以便存取個人帳號與功能。

Scenario：正常登入成功
  Given 使用者已擁有有效帳號，且帳號未被停用或鎖定
  When  使用者輸入正確的 Email 與密碼並點擊「登入」
  Then  系統建立已認證 Session，跳轉至首頁並顯示使用者名稱

Scenario：密碼錯誤累計達上限後鎖定
  Given 使用者已連續登入失敗 4 次（本次為第 5 次）
  When  使用者再次輸入錯誤密碼
  Then  系統鎖定帳號 15 分鐘並顯示「登入失敗次數過多，請於 15 分鐘後再試」

Scenario：帳號已被 Admin 停用
  Given 使用者帳號已被 Admin 停用
  When  使用者嘗試登入（無論密碼是否正確）
  Then  系統顯示「此帳號已停用，請聯繫管理員」，不建立 Session

---

**Story S2：忘記密碼重設**
作為忘記密碼的使用者，我想要透過 Email 重設密碼，以便在不洩漏舊密碼的前提下恢復存取。

Scenario：成功申請重設連結
  Given 使用者帳號存在且 Email 已驗證
  When  使用者在忘記密碼頁提交 Email 表單
  Then  系統寄出含唯一 token 的重設連結（有效期 1 小時），頁面顯示「請查收 Email」

Scenario：使用有效連結重設密碼
  Given 使用者收到重設 Email 且連結在 1 小時內
  When  使用者點擊連結，輸入新密碼並確認
  Then  系統更新密碼雜湊，使舊 token 失效，跳轉至登入頁顯示「密碼已重設」

Scenario：使用已過期連結
  Given 重設連結已超過 1 小時
  When  使用者點擊連結
  Then  系統顯示「連結已失效，請重新申請」，導向忘記密碼頁面

---

**Story S3：Admin 帳號管理**
作為 Admin，我想要查看登入記錄並停用帳號，以便處理異常登入行為。

Scenario：查看指定使用者的登入記錄
  Given Admin 已登入後台管理介面
  When  Admin 搜尋特定使用者並點擊「查看登入記錄」
  Then  系統顯示該使用者最近 30 筆記錄（時間、IP、結果）

Scenario：停用使用者帳號
  Given Admin 在使用者管理頁面查看某帳號
  When  Admin 點擊「停用帳號」並確認操作
  Then  系統將帳號標記為停用，該使用者下次嘗試登入時收到停用提示

---

## 輸出二：技術設計

### 系統架構
採用 Node.js + Express 後端、React 前端，PostgreSQL 作為主要資料庫（Supabase Free Tier，500MB 免費）。選擇 PostgreSQL 而非 SQLite，因為需要多使用者並發且計畫對外公開。前端部署至 Vercel（免費方案），後端可部署至 Railway（免費方案含 500 小時/月）。Email 發送使用 Resend（免費方案含 3,000 封/月）。身份驗證使用 express-session + connect-pg-simple（Session 存於 PostgreSQL），避免引入 JWT 的複雜性。

### 資料流程
登入流程：前端提交 email/password → POST /api/auth/login → 後端查詢 users 表確認帳號存在且未停用 → 查詢 login_attempts 判斷是否已鎖定（最近 15 分鐘失敗次數 ≥ 5）→ bcrypt.compare 驗證密碼 → 成功：建立 Session；失敗：新增 login_attempts 記錄並回傳錯誤。密碼重設流程：提交 email → 產生 crypto.randomBytes(32) token → 存入 password_reset_tokens 表（含過期時間）→ Resend API 發送 email → 使用者點擊連結 → 驗證 token 有效性 → bcrypt.hash 新密碼 → 更新 users 表 → 刪除 token。

### 資料模型
**users**：email(unique, not null), password_hash(varchar), is_active(boolean, default true), role(enum: user/admin), last_login_at(timestamp)
**login_attempts**：user_id(FK→users), ip_address(varchar), success(boolean), attempted_at(timestamp)，不含 user_id 的記錄用於記錄不存在帳號的嘗試
**password_reset_tokens**：user_id(FK→users), token_hash(varchar, unique), expires_at(timestamp), used_at(timestamp nullable)，儲存 token 的 hash 而非明文

### 錯誤處理策略
前端：所有 API 錯誤統一以 toast 通知顯示，表單驗證錯誤顯示於欄位下方。錯誤訊息用語統一為繁體中文，不向使用者顯示技術細節（如 SQL 錯誤）。後端：使用統一的 errorHandler middleware，回傳格式 `{ error: { code, message } }`。HTTP 狀態碼：400 輸入錯誤、401 認證失敗、403 無權限、429 請求過多、500 伺服器錯誤。重試策略：僅對冪等操作（GET）自動重試，非冪等操作（POST login）不自動重試，由使用者手動操作。

### 資訊安全
認證：express-session 搭配 httpOnly + secure + sameSite=Strict Cookie，Session secret 存於環境變數。密碼雜湊：bcrypt，cost factor 12。輸入驗證：前端 Zod schema 驗證（即時回饋）+ 後端重複驗證（信任邊界），SQL 查詢全部使用 parameterized queries 防 injection。密碼重設 token 儲存 SHA-256 hash，不儲存明文；token 使用後立即失效。OWASP 對應：A01 存取控制（角色檢查 middleware）、A02 加密（bcrypt + HTTPS）、A03 注入（parameterized queries）、A07 認證失敗（帳號鎖定機制）。

### QA 策略
單元測試（Vitest）：密碼雜湊/比對函式、token 產生/驗證邏輯、帳號鎖定判斷函式，覆蓋正常與邊界案例。整合測試（Supertest）：POST /auth/login（成功、密碼錯、鎖定、帳號停用）、POST /auth/forgot-password（存在/不存在 email）、POST /auth/reset-password（有效/過期 token）。E2E 測試（Playwright）：完整登入流程、忘記密碼重設流程，對應 S1-S2 的 Scenarios。CI：GitHub Actions 在 PR 時自動執行全部測試，覆蓋率門檻 80%。

### 維運考量
Log 使用 pino（結構化 JSON），必要欄位：timestamp、level、userId（已登入時）、ip、method、path、statusCode、duration_ms。敏感欄位（password、token）絕對不進 log。監控指標：登入失敗率（異常閾值 > 20%）、密碼重設請求量（異常閾值 > 正常 10 倍）、API 回應時間（P95 > 500ms 告警）。資料庫：Supabase 提供自動備份，同時設定每週 pg_dump 存至 Backblaze B2（免費方案）。版本管理：資料庫 Schema 使用 db-migrate 管理，rollback 步驟記錄於 RUNBOOK.md。

---

## 輸出三：任務規劃

## 第1階段：資料層與認證基礎
> 建立資料庫 Schema、基本認證 API，其他功能均依賴此基礎

### T1.1 初始化專案與資料庫 Schema
- **目標**：建立可執行的專案骨架與資料庫結構
- **工作項目**：
  - 初始化 Node.js 專案：`npm init`，安裝 express、pg、bcrypt、express-session、connect-pg-simple、pino、zod
  - 建立 `db/migrations/001_initial_schema.sql`，定義 users、login_attempts、password_reset_tokens 三個表
  - 建立 `db/seed.sql`，新增一筆 admin 測試帳號（密碼用 bcrypt hash）
  - 建立 `.env.example`，列出所有必要環境變數
  - 設定 `src/db.js`，建立 pg Pool 連線
- **完成條件**：執行 migration 後三個表存在；執行 seed 後 users 表有一筆記錄

### T1.2 實作登入 API（含帳號鎖定）
- **目標**：POST /api/auth/login 正確驗證帳號密碼、處理鎖定邏輯，並建立 Session
- **工作項目**：
  - 建立 `src/routes/auth.js`，掛載 POST /login 路由
  - 輸入驗證（Zod）：email 格式、password 非空
  - 查詢 users 表，帳號不存在或 is_active=false 均回傳 401（不透露帳號是否存在）
  - 查詢 login_attempts 判斷最近 15 分鐘失敗次數，≥5 次回傳 429
  - bcrypt.compare 驗證密碼
  - 成功：建立 express-session，記錄 userId 與 role
  - 失敗：INSERT INTO login_attempts (success=false)
  - 撰寫整合測試：成功、密碼錯、帳號鎖定、帳號停用四個 case
- **完成條件**：整合測試全部通過；curl 測試四種情境回傳正確狀態碼

### T1.3 實作忘記密碼 API（申請 + 重設）
- **目標**：完整的密碼重設流程，含 token 產生、Email 發送、token 驗證、密碼更新
- **工作項目**：
  - POST /api/auth/forgot-password：產生 32 bytes random token，儲存 SHA-256 hash，使用 Resend API 發送重設 Email（無論帳號是否存在，均回傳相同訊息）
  - POST /api/auth/reset-password：驗證 token hash、確認未過期且未使用、bcrypt.hash 新密碼、UPDATE users、UPDATE token used_at
  - 建立 `src/emails/reset-password.html` 模板
  - 撰寫整合測試：申請成功、申請不存在帳號、重設成功、過期 token、已使用 token
- **完成條件**：整合測試全部通過；開發環境可收到測試 Email

---

## 第2階段：前端介面與 Admin 功能
> 建立使用者可見的登入介面與 Admin 後台

### T2.1 建立登入與忘記密碼表單
- **目標**：可用的 React 登入介面，含正確的錯誤狀態顯示
- **工作項目**：
  - 建立 React 專案（Vite），安裝 react-router-dom、axios、zod、react-hook-form
  - `src/pages/Login.jsx`：email/password 表單，client-side Zod 驗證，串接 POST /api/auth/login，loading/error 狀態
  - `src/pages/ForgotPassword.jsx`：email 表單，串接 API，成功顯示「請查收 Email」
  - `src/pages/ResetPassword.jsx`：從 URL 取 token，新密碼/確認密碼表單，串接 API
  - Protected Route：未登入自動導向 /login
- **完成條件**：可完整走過登入、忘記密碼、重設密碼三個流程；錯誤訊息正確顯示

### T2.2 實作 Admin 後台登入記錄與停用功能
- **目標**：Admin 可查看登入記錄並停用帳號
- **工作項目**：
  - 後端：GET /api/admin/users（列出所有使用者）、GET /api/admin/users/:id/login-history（最近 30 筆）、PATCH /api/admin/users/:id（更新 is_active），所有路由加 requireAdmin middleware
  - `src/pages/admin/Users.jsx`：使用者列表、搜尋、停用按鈕（含確認 dialog）
  - `src/pages/admin/UserDetail.jsx`：登入記錄表格（時間、IP、成功/失敗）
  - 撰寫整合測試：非 Admin 存取回傳 403
- **完成條件**：Admin 帳號可查看記錄、停用帳號；非 Admin 存取 /admin API 回傳 403

---

## 第3階段：測試完善與部署準備

### T3.1 E2E 測試與安全檢查
- **目標**：確保核心流程在真實瀏覽器環境正確運作，並完成基本安全設定
- **工作項目**：
  - 安裝 Playwright，撰寫 E2E：完整登入流程（S1 Scenario 1）、鎖定流程（S1 Scenario 2）、完整密碼重設流程（S2 Scenario 2）
  - 設定 Helmet.js（HTTP security headers）
  - 設定 express-rate-limit（全域 API 限流）
  - 確認所有 API 回應不包含技術細節（stack trace 等）
  - 建立 `RUNBOOK.md`：記錄部署步驟、環境變數設定、rollback 流程
- **完成條件**：Playwright E2E 測試通過；Helmet headers 正確設定；RUNBOOK 完整

---

## 技術決策記錄

**選 express-session 而非 JWT**：本系統需要立即撤銷 Session 的能力（帳號停用時），JWT 的無狀態特性使立即撤銷困難。Session 儲存於 PostgreSQL 便於管理且符合免費預算。

**選 Resend 而非 Nodemailer + SMTP**：Resend 免費方案提供 3,000 封/月，API 呼叫比設定 SMTP 簡單，且有較好的送達率。

**不存在帳號的登入/重設均回傳相同訊息**：防止帳號列舉攻擊（Account Enumeration）。攻擊者無法透過不同的錯誤訊息判斷帳號是否存在。

**潛在風險**：Supabase Free Tier 有暫停機制（超過 7 天無活動會暫停），正式上線後需設定定期心跳請求或升級至付費方案。Railway Free Tier 亦有每月 500 小時限制，流量高時需考慮升級。
