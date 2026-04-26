# Feature Planner Skill

**[English](#english) · [繁體中文](#traditional-chinese)**

> ⚠️ The tool interface and all AI-generated outputs are in **Traditional Chinese (繁體中文)**.

---

<a name="english"></a>
# English

## What Problem Does This Solve?

The path from "I have an idea" to "engineers can start coding" is often the messiest part of software development:

- Requirements are too vague for engineers to act on
- Technical designs skip security and observability until it's too late
- Tasks are either too coarse to estimate or too granular to track
- AI Coding Tools go off-track because the prompt lacked structure

Feature Planner gives this path structure, repeatability, and consistent quality.

## What You Get

Describe a feature in plain language. The skill guides you through four stages and produces a complete planning document:

```
Your feature description (plain language)
    ↓
User Stories  (BDD Given / When / Then)
    ↓
Technical Design  (architecture, security, QA, ops — 7 sections)
    ↓
Task Breakdown  (independently verifiable development tasks)
    ↓
Full Markdown document  (hand directly to engineers or an AI Coding Tool)
```

> **Note:** All output — User Stories, technical design, task list, and the final document — is written in Traditional Chinese (Mandarin). This is intentional: the tool is designed for Mandarin-speaking development teams.

## File Structure

```
skill/
├── README.md                      ← This file
├── SKILL.md                       ← Entry point: workflow and usage rules
│
├── prompts/                       ← AI system prompts for each stage
│   ├── 01-user-stories.md         ← BDD story generation
│   ├── 02-tech-design.md          ← Technical design generation
│   └── 03-task-planning.md        ← Task breakdown generation
│
├── templates/                     ← Output document formats
│   ├── full-document.md           ← Complete planning document
│   └── task-list-minimal.md       ← Minimal task list for AI Coding Tools
│
├── examples/                      ← End-to-end worked examples
│   └── login-module.md            ← Example: login module planning
│
└── references/                    ← Reference material for better outputs
    ├── bdd-patterns.md            ← BDD best practices and anti-patterns
    └── tech-design-checklist.md   ← Technical design quality checklist
```

## Three Ways to Use It

### Option 1 — Paste into a Claude conversation (simplest)

No setup required. Works in Claude.ai, the Claude app, or any Claude interface.

1. Open a new Claude conversation
2. Paste the contents of `SKILL.md` at the start, or upload the file
3. Describe your feature — Claude will guide you through each stage

**Starter message:**
```
[Paste the contents of SKILL.md here]

I want to plan the following feature:
Users can log in with Email + Password. After 5 consecutive failures,
the account is locked for 15 minutes.
```

### Option 2 — Claude API (programmatic)

Use each prompt file directly as the `system` parameter.

```javascript
const fs = require('fs');
const Anthropic = require('@anthropic-ai/sdk');
const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

// Stage 1: Generate User Stories
async function generateStories(featureDescription) {
  const system = fs.readFileSync('./prompts/01-user-stories.md', 'utf-8');
  const response = await client.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 8192,
    system,
    messages: [{ role: 'user', content: `功能需求：\n${featureDescription}` }],
  });
  return response.content[0].text;
}

// Stage 2: Generate Technical Design (include preferences)
async function generateTechDesign(stories, preferences) {
  const system = fs.readFileSync('./prompts/02-tech-design.md', 'utf-8');
  const response = await client.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 8192,
    system,
    messages: [{
      role: 'user',
      content: `User Stories：\n${stories}\n\n使用者偏好：\n${preferences}`
    }],
  });
  return response.content[0].text;
}

// Stage 3: Generate Task Breakdown
async function generateTasks(stories, techDesign) {
  const system = fs.readFileSync('./prompts/03-task-planning.md', 'utf-8');
  const response = await client.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 8192,
    system,
    messages: [{
      role: 'user',
      content: `User Stories：\n${stories}\n\n技術設計：\n${techDesign}`
    }],
  });
  return response.content[0].text;
}
```

**Preferences format (sent as plain text in zh-TW):**
```
成本預算：完全免費（只用開源或免費方案）
部署環境：個人本機、對外公開服務
開發重點：最簡架構、長期維護
```

### Option 3 — Claude Code workflow (most automated)

```bash
cp -r /path/to/skill ./planning-skill
claude "請閱讀 planning-skill/SKILL.md，我需要規劃一個新功能"
```

## Step-by-Step Walkthrough

### Step 1 — Describe your feature

Plain language is fine. Be specific about: who uses it, what the key flow is, and any constraints or edge cases.

```
Users can check out their shopping cart using a credit card.
A confirmation email is sent on success and inventory is updated.
Unpaid orders are automatically cancelled after 30 minutes.
```

### Step 2 — Review User Stories

Claude generates BDD-formatted stories. You can confirm, request changes, or add new scenarios before proceeding.

### Step 3 — Set technical preferences

After confirming stories, answer three questions:

```
Cost budget:     Completely free / Low cost (<$10/mo) / Medium ($10–$100/mo) / Unlimited
Deployment:      Local machine / Internal system / Public-facing (multiple allowed)
Dev priorities:  Minimal architecture / Security-first / Fast delivery / Long-term maintenance
```

### Step 4 — Review the technical design

Seven sections are generated. Focus on: does the tech stack fit your budget? Are security measures concrete? Is the QA strategy realistic?

You can edit any section or delete sections you don't need.

### Step 5 — Review and export tasks

Each task has a goal, work items, and acceptance criteria. Confirm the scope feels right (1–3 days per task), then export:

**For engineers:** Paste into GitHub Issues, Linear, or Notion.

**For AI Coding Tools:**
```
Please read the following planning document.
Start with Stage 1, Task T1.1. Tell me when each task is done before moving on.

[paste document]
```

## FAQ

**Can I skip a stage?**
Yes. If you already have User Stories, start from Stage 2 with `prompts/02-tech-design.md`.

**The output got cut off. What do I do?**
The prompts are set to `max_tokens: 8192`. If it still cuts off, ask Claude to continue from where it stopped, or break the request into smaller parts.

**Can I customize the prompts?**
Yes — edit any file in `prompts/`. Common customizations: add your organization's tech stack constraints to `02-tech-design.md`, or adjust task size expectations in `03-task-planning.md`.

**Can I use this for a feature in an existing project?**
Yes. Include your current stack in the feature description:
```
Existing stack: React + Node.js/Express + PostgreSQL, deployed on AWS EC2
New feature: ...
```

## Output Quality Checklist

| Stage | Check |
|-------|-------|
| User Stories | Does every Scenario have Given/When/Then? Are edge cases covered? |
| Technical Design | Does the stack fit the budget? Are security measures specific (not generic)? |
| Task Breakdown | Can each task be completed in 1–3 days? Is the acceptance criteria verifiable? |

Full checklists: `references/tech-design-checklist.md` and `references/bdd-patterns.md`

---

---

<a name="traditional-chinese"></a>
# 繁體中文

## 這個 Skill 在解決什麼問題？

在軟體開發中，「從想法到任務」這段路常常是最艱難的——

- 需求描述太模糊，工程師不知道從哪裡開始
- 技術設計缺乏安全與營運考量，上線才發現漏洞
- 任務拆得太粗或太細，估時不準、驗收困難
- 給 AI Coding Tool 的提示詞（prompt）不夠具體，生出來的程式碼走鐘

Feature Planner 的目標是讓這段路有結構、可重複、品質穩定。

## 你會得到什麼？

輸入一段功能描述，Skill 會引導你完成四個階段，最終輸出一份完整的文件：

```
功能描述（你的輸入）
    ↓
User Stories（BDD Given/When/Then 格式）
    ↓
技術設計文件（架構、資安、QA、維運等 7 個章節）
    ↓
任務規劃（可獨立驗收的開發任務）
    ↓
完整 Markdown 文件（可直接給工程師或 AI Coding Tool 使用）
```

> **注意：** 工具的所有輸出——User Stories、技術設計、任務清單與最終文件——皆以zh-TW撰寫。這是刻意設計的，本工具主要協助以正體中文溝通的開發團隊。

## 資料夾結構說明

```
skill/
├── README.md                      ← 你正在看的這份文件
├── SKILL.md                       ← Skill 入口：工作流程與使用規則
│
├── prompts/                       ← 每個階段的 AI System Prompt
│   ├── 01-user-stories.md         ← BDD Story 生成
│   ├── 02-tech-design.md          ← 技術設計生成
│   └── 03-task-planning.md        ← 任務拆解生成
│
├── templates/                     ← 輸出文件的格式模板
│   ├── full-document.md           ← 完整規劃文件
│   └── task-list-minimal.md       ← 精簡任務清單（直接給 AI Coding Tool）
│
├── examples/                      ← 完整走過四個步驟的實際範例
│   └── login-module.md            ← 範例：登入模組規劃
│
└── references/                    ← 參考資料，讓輸出品質更好
    ├── bdd-patterns.md            ← BDD 最佳實踐與常見錯誤
    └── tech-design-checklist.md   ← 技術設計品質檢查清單
```

## 三種使用方式

### 方式一：直接在 Claude 對話中使用（最簡單）

不需要任何設定，在 Claude.ai、Claude App 或任何 Claude 介面均可使用。

1. 開啟新對話
2. 將 `SKILL.md` 的內容貼到對話開頭，或直接上傳檔案
3. 描述你的功能需求，Claude 會引導你逐步完成

**範例開場白：**
```
[貼上 SKILL.md 的內容]

我想規劃以下功能：
使用者可以用 Email + 密碼登入，連續失敗 5 次後鎖定 15 分鐘。
Admin 可以在後台停用帳號。
```

### 方式二：搭配 Claude API 使用（進階）

把每個 prompt 檔案作為 `system` 參數，逐步呼叫 API。

```javascript
const fs = require('fs');
const Anthropic = require('@anthropic-ai/sdk');
const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

// 第一步：生成 User Stories
async function generateStories(featureDescription) {
  const system = fs.readFileSync('./prompts/01-user-stories.md', 'utf-8');
  const response = await client.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 8192,
    system,
    messages: [{ role: 'user', content: `功能需求：\n${featureDescription}` }],
  });
  return response.content[0].text;
}

// 第二步：生成技術設計（需附上偏好資訊）
async function generateTechDesign(stories, preferences) {
  const system = fs.readFileSync('./prompts/02-tech-design.md', 'utf-8');
  const response = await client.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 8192,
    system,
    messages: [{
      role: 'user',
      content: `User Stories：\n${stories}\n\n使用者偏好：\n${preferences}`
    }],
  });
  return response.content[0].text;
}

// 第三步：生成任務規劃
async function generateTasks(stories, techDesign) {
  const system = fs.readFileSync('./prompts/03-task-planning.md', 'utf-8');
  const response = await client.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 8192,
    system,
    messages: [{
      role: 'user',
      content: `User Stories：\n${stories}\n\n技術設計：\n${techDesign}`
    }],
  });
  return response.content[0].text;
}
```

**技術偏好格式範例：**
```
成本預算：完全免費（只用開源或免費方案）
部署環境：個人本機、對外公開服務
開發重點：最簡架構、長期維護
```

### 方式三：整合進 Claude Code 工作流（最自動化）

```bash
cp -r /path/to/skill ./planning-skill
claude "請閱讀 planning-skill/SKILL.md，我需要規劃一個新功能"
```

## 逐步操作教學

### 第一步：描述功能需求

不需要技術格式，自然語言描述即可。越具體越好，特別是：
- 主要使用者是誰
- 最重要的操作流程
- 有沒有特殊限制或邊界情況

```
使用者可以在購物車選擇商品後進行結帳，
支援信用卡付款，付款成功後寄送確認 Email 並更新庫存。
訂單在 30 分鐘內未付款自動取消並釋放庫存。
```

### 第二步：審閱並調整 User Stories

Claude 會生成 BDD 格式的 Stories。你可以確認、要求修改、或新增 Scenario 後再繼續。

### 第三步：設定技術偏好

確認 Stories 後，回答三個問題：

```
成本預算：完全免費 / 低成本（<$10/mo）/ 中等（$10~$100/mo）/ 不限
部署環境：個人本機 / 內部系統 / 對外公開（可複選）
開發重點：最簡架構 / 資安優先 / 快速交付 / 長期維護（可複選）
```

### 第四步：審閱技術設計

生成 7 個章節。重點確認：技術棧是否符合預算？資安措施是否具體？QA 策略是否務實？

可以編輯任意章節，或刪除不需要的章節。

### 第五步：審閱任務並匯出

確認每個任務的範圍合理（1~3 天）後匯出：

**給工程師：** 貼入 GitHub Issues、Linear 或 Notion

**給 AI Coding Tool：**
```
請閱讀以下規劃文件，從第一階段 T1.1 開始實作。
每完成一個任務告訴我，確認後再繼續下一個。

[貼上文件]
```

## 常見問題

**可以跳過某個階段嗎？**
可以。如果你已有 User Stories，直接使用 `prompts/02-tech-design.md` 從技術設計開始。

**輸出被截斷怎麼辦？**
Prompt 預設 `max_tokens: 8192`。若仍被截斷，請告訴 Claude「請從上次截斷的地方繼續輸出」。

**可以客製化 prompt 嗎？**
完全可以，直接編輯 `prompts/` 下的 `.md` 檔案。常見做法：在 `02-tech-design.md` 加入組織的技術棧限制，或在 `03-task-planning.md` 調整任務粒度定義。

**可以用在現有專案的新功能嗎？**
可以。在描述功能時加上現有技術棧資訊：
```
現有技術棧：React + Node.js/Express + PostgreSQL，部署在 AWS EC2
新功能需求：...
```

## 輸出品質自我檢查

| 階段 | 確認項目 |
|------|---------|
| User Stories | 每個 Scenario 都有 Given/When/Then？涵蓋了邊界案例？ |
| 技術設計 | 符合我的成本預算？技術選型有說明理由？資安設計具體？ |
| 任務規劃 | 每個任務 1~3 天可完成？完成條件可以被驗收？ |

詳細清單見 `references/tech-design-checklist.md` 與 `references/bdd-patterns.md`。

