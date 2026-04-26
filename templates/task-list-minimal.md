# 精簡任務清單模板

**用途**：直接給 AI Coding Tool（Claude Code、Cursor、Copilot）使用的精簡格式  
**特點**：省略設計文件，只保留任務指令，適合已有技術設計共識的情況

---

## 給 AI Coding Tool 的指令前置詞

```
你是一位資深工程師，請協助我實作以下開發任務。
技術棧：{{技術棧，例如：Node.js + Express + SQLite + React}}
代碼庫位置：{{專案根目錄或 repo URL}}

規則：
- 每次只實作一個任務，完成後等我確認再繼續
- 實作前先說明你的做法，我確認後再開始寫代碼
- 如有疑問先問清楚，不要自行假設
- 每個任務完成後列出：完成項目、可能影響、建議測試方式

---

任務清單如下：
```

---

## 精簡任務清單格式

```markdown
## 第{{N}}階段：{{階段名稱}}

### T{{N.M}} {{任務名稱}}
目標：{{一句話}}
工作：
- {{步驟1}}
- {{步驟2}}
驗收：{{完成條件}}

### T{{N.M+1}} {{任務名稱}}
...
```

---

## 範例（登入模組）

```markdown
## 第1階段：資料層與認證基礎

### T1.1 建立資料庫 Schema
目標：建立 users 資料表與登入失敗記錄表
工作：
- 建立 `db/migrations/001_create_users.sql`
- users 表：id, email(unique), password_hash, is_active, created_at
- login_attempts 表：id, email, attempted_at, success
- 建立 `db/seed.sql`，新增一筆測試帳號
驗收：執行 migration 後，`SELECT * FROM users` 有資料

### T1.2 實作登入 API
目標：POST /api/auth/login 正確驗證帳號密碼並建立 Session
工作：
- 建立 `src/routes/auth.js`
- 輸入驗證：email 格式、密碼非空
- 以 bcrypt.compare 驗證密碼
- 失敗 5 次後回傳 429，鎖定 15 分鐘（以 login_attempts 表判斷）
- 成功：建立 express-session，設定 userId
驗收：curl 測試三種情境均回傳正確狀態碼

## 第2階段：前端介面

### T2.1 實作登入表單元件
...
```

---

## 搭配 Claude Code 使用

```bash
# 在專案根目錄執行
claude "請閱讀 TASKS.md，從 T1.1 開始實作，完成後等我確認"
```
