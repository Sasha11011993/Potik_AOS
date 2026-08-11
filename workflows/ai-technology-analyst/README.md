# AI Technology Analyst

Workflow аналізує новини про AI, n8n, AI agents, MCP, інтеграції та vibe coding і формує практичні Tech Signals для бізнес-автоматизації.

## Посилання

- [Workflow в n8n](https://n8n.mykh-automation.com/workflow/2m8YgfQaLbTEVKBX)
- [Google Sheets: AI Automation Intelligence DB](https://docs.google.com/spreadsheets/d/1rNJ3PVZp78kM-9XOIFJBP7t54mWdGVEzkHzV2QUDAO0/edit)
- [Повна документація](https://docs.google.com/document/d/1xQiu98Xp7K8t0_6TEfDsHvJShcQJrre2v_PvXOIDc_o/edit)

## Потік

```text
News
  → Rule-Based Technology Filter
  → Pass Keyword Threshold (score ≥ 5)
  → Limit AI Candidates (10)
  → Classify News via HTTP
  → Keep Relevant News
  → Analyse Tech Signal via HTTP
  → Tech_Signals
  → Automation_Runs
```

## Результат

- `Tech_Signals`: технологія, постачальник, нова можливість, зміна, бізнес-застосування, maturity, інтеграції, ризики та оцінки.
- `Automation_Runs`: статус запуску, оброблені записи, AI-виклики, input/output/total tokens і `estimated_cost_usd`.

## AI та витрати

Використовується OpenAI Responses API з моделлю `gpt-5.4-mini`. Rule-based filter відсіює нерелевантні новини до AI-викликів.

Вартість оцінюється з фактичного usage API:

- input: $0.75 / 1M tokens;
- cached input: $0.075 / 1M tokens;
- output: $4.50 / 1M tokens.

## Запуск

Workflow запускається вручну через **Execute Workflow**. Після запуску перевірте статус execution, нові рядки в `Tech_Signals` і підсумковий рядок в `Automation_Runs`.

Credentials не містяться в export JSON; після імпорту в n8n потрібно прив’язати `Potik_AOS` для Google Sheets і `Potik_AOS-OpenAI-Bearer` для OpenAI API.

## Переносима конфігурація після імпорту

Виберіть свій документ замість `[REDACTED_GOOGLE_SHEET_ID]` у вузлах `Read News`, `Write Tech Signal`, `Mark News Irrelevant`, `Mark News Analyzed`, `Find Existing Tech Signal`, `Mark News Keyword Filtered` і `Write Automation Run Directly`. Підтвердьте аркуші `News`, `Tech_Signals` та `Automation_Runs`, після чого прив’яжіть Google Sheets OAuth2 і OpenAI Bearer/Auth credentials.

Усі мережеві ноди мають три спроби з паузою 5 секунд. Підсумковий success-запис формується один раз після явного бар’єра outcome-гілок; остаточні помилки передаються до спільного `Handle Potik AOS workflow failure`, який потрібно опублікувати та призначити як Error Workflow у Settings.




