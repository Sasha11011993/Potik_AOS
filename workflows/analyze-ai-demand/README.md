�r�^�f��ئ{Oly�'vî���# AI Demand Analyst

## Операційна інструкція

### 1. Призначення та поточний стан

AI Demand Analyst аналізує можливості у сфері AI-автоматизації, n8n, AI-агентів та інтеграцій. Він відокремлює реальні клієнтські проєкти від вакансій і пропозицій виконавців, перетворює запити на структуровані сигнали попиту та оновлює індивідуальний і агрегований звіт.

- Workflow у n8n: [Analyze AI demand from opportunities](https://n8n.mykh-automation.com/workflow/AcXgjF2Y3WVhXQuu)
- Таблиця даних і результатів: [n8n AI Opportunities Monitor](https://docs.google.com/spreadsheets/d/1rNJ3PVZp78kM-9XOIFJBP7t54mWdGVEzkHzV2QUDAO0/edit)
- Поточний стан: ручний тригер, workflow не активований і не має Schedule Trigger.
- Перевірений запуск: `ADA-2332` завершився `Success` за 171 секунду; прочитано 154 можливості, 65 проаналізовано.

### 2. Схема обробки

`Ручний запуск → Read opportunities → Filter valid opportunities → Classify opportunity via HTTP → Normalize opportunity intent usage → Keep client projects → Extract demand signals via HTTP → Normalize demand signals usage → Calculate demand score → Group demand by automation type → Score demand strength and trend → Demand Analysis → Demand Report → Automation_Runs`

Паралельно після розрахунку індивідуальні можливості записуються до `Demand Analysis`, а згрупований ринковий результат — до `Demand Report`.

### 3. Відсів шуму та правила класифікації

`Classify opportunity via HTTP` викликає OpenAI Responses API і визначає намір запису. Лише категорія `client_project` означає реальний запит клієнта і проходить до AI-аналізу.

Категорії `client_hiring`, `provider_offer` та `irrelevant` не потрапляють до `Demand Analysis` або `Demand Report`. Це зменшує вплив вакансій, портфоліо й нерелевантних публікацій на оцінку попиту.

### 4. Які сигнали витягує AI

`Extract demand signals via HTTP` викликає OpenAI Responses API та формує:

- проблему клієнта;
- тип автоматизації;
- технології;
- необхідні skills;
- тип клієнта;
- масштаб проєкту;
- потенційну послугу;
- `confidence`.

`automation_type` використовує одну основну категорію: `ai_agent`, `workflow_automation`, `system_integration`, `lead_generation`, `customer_support`, `content_automation`, `data_research` або `other`.

Вихідні дата публікації, бюджет, валюта та URL зберігаються разом із AI-сигналами.

### 5. Demand Score, сила попиту та тренд

**Demand Score v1** має діапазон 0–100:

`35 базових балів за client_project + свіжість + сигнал розкритого бюджету + scope + AI confidence`

- Свіжість: 25 балів до 7 днів, 18 до 30 днів, 10 до 90 днів, інакше 3.
- Розкритий бюджет: 15 балів; відсутній: 5.
- Scope: `ongoing` — 10, `pilot` — 7, інші — 5.
- Confidence додає від 0 до 20 балів.

**Сила попиту групи:** `min(100, кількість проєктів × 15 + середній Demand Score × 0,4)`.

- `high` — від 80;
- `medium` — від 60;
- `emerging` — менше 60.

**Тренд** порівнює останні 30 днів із попередніми 30 днями:

- `rising` — зростання від 25%;
- `declining` — падіння від 25%;
- `stable` — зміна між цими межами;
- `emerging` — записи є в поточному періоді, але відсутні в попередньому;
- `aging` — записів немає в обох періодах.

### 6. Результати у Google Sheets

**Demand Analysis** містить один рядок на проаналізовану можливість.

- Ключ оновлення: `opportunity_id (ID можливості)`.
- Перевіряйте: проблему клієнта, `automation_type`, `technologies`, `required_skills`, `client_type`, `urgency`, `ai_fit_score` та `potential_service`.

**Demand Report** містить один рядок на `demand_cluster`.

- Ключ оновлення: `demand_cluster (Кластер попиту)`.
- Перевіряйте: рейтинг, кількість можливостей, силу попиту, Demand Score групи, тренд, технології, skills, типи клієнтів і бюджетний підсумок.

Google Sheets працює в режимі **Append or Update**, тому повторний запуск оновлює записи за ключами, а не створює дублікати.

`Automation_Runs (Запуски автоматизацій)` містить один рядок на нормальний завершений запуск за ключем `run_id`. Тут перевіряйте статус, кількість записів, AI-виклики, токени та орієнтовну вартість.

### 7. Облік токенів і вартості

Обидва AI-кроки працюють через OpenAI Responses API. Ноди `Normalize opportunity intent usage` і `Normalize demand signals usage` зберігають фактичні `input_tokens`, `output_tokens`, `total_tokens` і cached input tokens із відповіді API.

Наприкінці запуску `Prepare demand run summary` підсумовує usage, а `Write demand automation run` записує `ai_calls`, `model_name`, `input_tokens`, `output_tokens`, `total_tokens`, `estimated_cost_usd` і `run_metadata` до `Automation_Runs`.

Для `gpt-5.4-mini` формула вартості використовує $0.75 / 1M input, $0.075 / 1M cached input і $4.50 / 1M output tokens:

```text
(input − cached_input) × 0.75 / 1M
+ cached_input × 0.075 / 1M
+ output × 4.50 / 1M
```

Приклад `ADA-2332`: 219 AI-викликів, 113335 input tokens, 11946 output tokens, 125281 токенів загалом, `estimated_cost_usd = 0.13582065`; статус `success`.

### 8. Інструкція оператора

1. Відкрийте workflow у n8n і натисніть **Execute Workflow**.
2. Дочекайтеся статусу **Success** у вкладці **Executions**.
3. Відкрийте `Demand Analysis` для перевірки окремих можливостей.
4. Відкрийте `Demand Report` для перегляду кластерів попиту і `Automation_Runs` для токенів, вартості та статусу запуску.

Для аналізу ринку спочатку сортуйте `Demand Report` за `rank` або `demand_score`, далі перевіряйте `key_client_problems`, `top_technologies`, `top_required_skills`, `client_types` і `potential_service`.

### 9. Діагностика

| Симптом | Що перевірити |
| --- | --- |
| Немає рядків у Demand Analysis | `Read opportunities`, `Filter valid opportunities`, `Classify opportunity via HTTP`; можливо, жоден запис не потрапив у `client_project`. |
| Неочікувані AI-дані | `Extract demand signals via HTTP` у execution data, текст оголошення та `confidence`. |
| Не оновлюється Google Sheets | Credential `Potik_AOS`, доступ до таблиці, ID документа й точні заголовки колонок. Остаточні помилки передаються до спільного Error Workflow. |
| Неочікуваний тренд | `recent_30_days_count` і `previous_30_days_count`. Тренд — порівняння двох вікон, а не прогноз. |
| Помилка OpenAI | Credential, ліміти API й повідомлення у виконанні; HTTP-ноди OpenAI виконують повторні спроби перед зупинкою виконання. |

### 10. Надійність і безпека

Усі мережеві ноди — OpenAI, Google Sheets і зовнішні HTTP-запити — налаштовані на три спроби з паузою 5 секунд. Невідновлювана помилка не поглинається workflow: n8n передає її до спільного Error Workflow.

Credentials зберігаються в n8n; робочі Google Sheets і OpenAI credentials мають назву `Potik_AOS`. Не додавайте токени, API-ключі чи паролі в Set-ноди, prompts або звичайні поля workflow.

### 11. Межі поточної версії

Workflow запускається вручну, не надсилає автоматичних сповіщень і не має Schedule Trigger. Він аналізує доступні можливості та формує сигнали попиту, але не створює ставки, не пише клієнтам і не ухвалює бізнес-рішення замість оператора.

## Переносима конфігурація після імпорту

Виберіть свій документ замість `[REDACTED_GOOGLE_SHEET_ID]` у вузлах `Read opportunities`, `Upsert demand analysis rows`, `Upsert demand report rows` і `Write demand automation run`. Підтвердьте відповідні аркуші `Opportunities`, `Demand Analysis`, `Demand Report` та `Automation_Runs`, після чого прив’яжіть Google Sheets OAuth2 і OpenAI Bearer/Auth credentials через n8n Credentials.

Для запису та Telegram-сповіщення про остаточні помилки імпортуйте й опублікуйте `Handle Potik AOS workflow failure`, а потім призначте його як Error Workflow у Settings цього workflow.

