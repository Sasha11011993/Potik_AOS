# Збирати AI-проєкти з фриланс-платформ

Автоматизація n8n для пошуку AI, n8n та automation-можливостей на фриланс-платформах. Workflow отримує дані з джерел, нормалізує їх, фільтрує за ключовими словами, прибирає дублікати й зберігає результати для ручної перевірки в Google Sheets.

## Вміст репозиторію

- `workflows/collect-ai-freelance-opportunities.json` — експорт workflow для імпорту в n8n.
- `README.md` — технічна й операційна документація.
- [Повна документація в Google Docs](https://docs.google.com/document/d/18DWuHrSLCPEgYwQMUn47Bs2M97zaYnupn_AVDnkSwVY/edit).

## Посилання

- [Workflow в n8n](https://n8n.mykh-automation.com/workflow/XhZ61bzyzk3TL89y)
- [Google Sheets — n8n AI Opportunities Monitor](https://docs.google.com/spreadsheets/d/1rNJ3PVZp78kM-9XOIFJBP7t54mWdGVEzkHzV2QUDAO0/edit)
- Аркуш результатів: `Opportunities (Можливості)`

## Поточний стан

Workflow має ручний тригер і не активований. Автоматичний запуск за розкладом та повідомлення про нові можливості поки не ввімкнені.

## Схема обробки

```text
Ручний запуск
  ├─ n8n Community — Jobs ────────────────┐
  ├─ Freelancer.com ──────────────────────┤
  ├─ Freelancehunt ───────────────────────┼─> Об’єднання
  └─ Djinni ──────────────────────────────┘       ↓
                                          Підготовка dedupe_key
                                                    ↓
                                          Видалення дублікатів
                                                    ↓
                                          Google Sheets: Opportunities
```

Кожна гілка: отримання даних → нормалізація → фільтрація релевантності. Після об’єднання workflow створює `dedupe_key`, видаляє дублікати й виконує upsert у Google Sheets.

## Джерела

| Джерело | Спосіб | Особливості |
| --- | --- | --- |
| n8n Community — Jobs | RSS | Бюджет зазвичай не повертається структуровано. |
| Freelancer.com | API через HTTP Request | Повертає `budget.minimum`, `budget.maximum` і валюту, якщо вони оприлюднені. |
| Freelancehunt | API через HTTP Request | Один бюджет записується у `budget_min` і `budget_max`. |
| Djinni | RSS | Структуровані бюджет і валюта не гарантуються. |

## Нормалізована структура та дедуплікація

Основні поля: `source_id`, `external_id`, `title`, `content_text`, `url`, `author`, `source_type`, `published_at`, `fetched_at`, `budget_min`, `budget_max`, `currency` і `search_text`.

Ключ дедуплікації:

```text
source_id:external_id
```

Вузол `Прибрати дублікати за dedupe_key` залишає один запис на можливість. Вузол Google Sheets використовує `Append or Update Row` за колонкою `dedupe_key (Ключ дедуплікації)`, тому повторні запуски оновлюють запис, а не створюють копію.

## Фільтрація

Вузли `Залишити релевантні …` перевіряють `search_text` за повним набором ключових слів моніторингу. `search_text` містить назву, опис, доступні категорії, навички й теги. Нерелевантні записи не потрапляють до Google Sheets.

## Google Sheets

Використовуються ключові колонки:

- `opportunity_id`, `dedupe_key`, `source_id`, `external_id`
- `title`, `description`, `url`, `client_name`
- `budget_min`, `budget_max`, `currency`
- `published_at`, `collected_at`, `review_status`, `updated_at`

Для Freelancer.com зберігається діапазон бюджету. Для Freelancehunt одна сума дублюється як мінімальний і максимальний бюджет. Порожні бюджет або валюта означають, що джерело не віддало ці значення структуровано.

## Імпорт workflow

1. У n8n відкрийте **Import from File**.
2. Виберіть `workflows/collect-ai-freelance-opportunities.json`.
3. Прив’яжіть наявні credentials до відповідних вузлів.
4. Запустіть workflow вручну та перевірте Execution data і таблицю.

Експорт навмисно не містить токенів, API-ключів або OAuth-даних.

## Credentials і безпека

- Google Sheets OAuth2: `Potik_AOS`
- Freelancer API: окремий credential для Freelancer.com API
- Freelancehunt API: `Freelancehunt API`

Зберігайте секрети тільки у розділі **Credentials** n8n. Не додавайте токени до Set, Code або HTTP-заголовків як звичайний текст.

## Діагностика

| Симптом | Що перевірити |
| --- | --- |
| Немає нових записів | Вихід вузлів джерела та фільтра у розділі Executions. |
| Порожній бюджет | Raw output API; RSS-джерела часто не мають структурованої суми. |
| Дублікати | `external_id`, `dedupe_key` і вузол видалення дублікатів. |
| `undefined:undefined` або порожні рядки | `Підготувати dedupe_key` і мапінг Google Sheets. |
| API-помилка | Credential, rate limit та повідомлення HTTP Request. |
| Google Sheets-помилка | Credential `Potik_AOS`, ID документа/аркуша й назви заголовків. |

## Наступні кроки

- Додати Schedule Trigger після погодження частоти запуску.
- Підключити workflow «Обробник помилок» для Telegram-повідомлень про збої.
- Надсилати повідомлення лише про нові релевантні можливості.
- За потреби додати оцінку релевантності та чергу ручної перевірки.

## Межі поточної версії

Workflow збирає та структурує можливості. Він не створює ставки, не надсилає відгуки, не контактує з клієнтами й не приймає рішення замість оператора.
