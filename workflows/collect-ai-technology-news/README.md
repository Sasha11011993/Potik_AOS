# Збирати AI-новини з технологічних джерел

Ручний n8n workflow для збору релевантних AI, agentic AI, developer-tools та automation-новин у спільну Google Sheets-таблицю. Поточна версія неактивна й не має Schedule Trigger або сповіщень.

## Посилання

- [Workflow в n8n](https://n8n.mykh-automation.com/workflow/pW2DbVQtTAKzBfYY)
- [Google Sheets — n8n AI Opportunities Monitor](https://docs.google.com/spreadsheets/d/1rNJ3PVZp78kM-9XOIFJBP7t54mWdGVEzkHzV2QUDAO0/edit)
- Аркуш результатів: `News (Новини)`
- [Повна документація в Google Docs](https://docs.google.com/document/d/1Rqv2K86jv0C1lvr-n42BAgb0_HtJ4UHXnPYNJTqHGJk/edit)
- [Експорт workflow](../collect-ai-technology-news.json)

## Поточний стан

Workflow має ручний запуск і залишається неактивним. Останній успішний ручний запуск записав **477** унікальних новин у `News (Новини)`.

## Схема обробки

```text
Ручний запуск
  └─ 10 джерел
       ↓
Нормалізація → фільтрація релевантності
       ↓
Об’єднати релевантні новини (Merge, 10 входів)
       ↓
Прибрати дублікати за dedupe_key
       ↓
Зберегти новини в Google Sheets: News
```

Кожна гілка виконує отримання даних, нормалізацію до спільної структури та фільтрацію за повним набором ключових фраз відповідного джерела.

## Підключені джерела

| Джерело | Спосіб отримання |
| --- | --- |
| n8n Blog | RSS |
| OpenAI News / Codex | RSS |
| Anthropic News | HTML-сторінка |
| Model Context Protocol Blog | RSS |
| LangChain Blog | RSS |
| Google Developers Blog | RSS |
| GitHub Blog — AI & ML | RSS |
| Vercel Blog / v0 | Atom |
| Zapier Blog | RSS |
| Cursor Blog | HTML-сторінка |

У таблиці-реєстрі є додаткові джерела P1/P2/P3. Make Blog поки не підключено: стандартний публічний HTTP-запит повертає `403 Forbidden`. Для нього потрібен офіційний RSS/API або окремо погоджений сервіс веб-скрапінгу.

## Структура даних

Кожна новина перед записом має такі поля:

| Поле | Призначення |
| --- | --- |
| `news_id` | Ідентифікатор новини; у поточній реалізації дорівнює `dedupe_key`. |
| `dedupe_key` | Ключ унікальності в межах запуску та ключ upsert у Google Sheets. |
| `source_id`, `source_name`, `source_type` | Ідентифікація походження. |
| `title`, `description`, `url`, `author` | Основний зміст і посилання. |
| `published_at`, `collected_at`, `updated_at` | Дати публікації, збору та оновлення. |
| `categories`, `search_text` | Категорії та нормалізований текст для фільтрації. |
| `review_status` | Початково `new`; змінюється оператором під час перевірки. |

## Дедуплікація та запис

Вузол `Об’єднати релевантні новини` має 10 входів — по одному від кожного Filter-вузла. Далі `Прибрати дублікати за dedupe_key` залишає один запис на ключ у межах запуску.

Вузол `Зберегти новини в Google Sheets` виконує **Append or Update Row** в аркуші `News (Новини)` за колонкою `dedupe_key (Ключ дедуплікації)`. Повторні запуски оновлюють запис, а не створюють копію.

Для запису використовується Google Sheets OAuth2 credential `Potik_AOS`; секрети не входять до export-файлу. Масив `categories` перетворюється на текст. `description` і `search_text` обмежуються 45 000 символів, щоб не перевищувати ліміт Google Sheets у 50 000 символів на клітинку.

## Переносима конфігурація після імпорту

У вузлі `Зберегти новини в Google Sheets` виберіть свій документ замість `[REDACTED_GOOGLE_SHEET_ID]` і підтвердьте аркуш `News (Новини)`. Google Sheets підключайте через OAuth2 credential; не додавайте ID, токени або OAuth-дані до JSON-export.

Усі RSS, HTTP та Google Sheets-вузли мають три спроби з паузою 5 секунд. Після імпорту призначте спільний `Handle Potik AOS workflow failure` як Error Workflow у Settings.

## Інструкція оператора

1. Відкрийте workflow у n8n і натисніть **Execute Workflow**.
2. Відкрийте **Executions** і перевірте вузли Merge, дедуплікації та Google Sheets.
3. Відкрийте аркуш `News (Новини)`, перевірте нові рядки, посилання та `review_status = new`.
4. Змініть `review_status` вручну після перевірки новини.

## Діагностика

| Симптом | Що перевірити |
| --- | --- |
| Немає нових новин | Вихід вузлів джерел, нормалізації та Filter у **Executions**. |
| Дублікати | `dedupe_key`, усі 10 входів Merge і matching column Google Sheets. |
| Помилка Google Sheets | Credential `Potik_AOS`, ID документа/аркуша та точні назви заголовків. |
| Помилка 50 000 символів | Обмеження 45 000 символів у `description` і `search_text`. |
| HTTP/API-помилка | Статус відповіді, rate limit і credential відповідного джерела. |
| Make Blog 403 | Не підключати нестабільну HTML-гілку без офіційного RSS/API або погодженого скрапера. |

## Наступні кроки

- Підключити решту джерел з реєстру за пріоритетами P1/P2/P3.
- Додати Schedule Trigger після погодження частоти запуску.
- Надсилати повідомлення лише про нові релевантні новини.
- Підключити окремий workflow обробки помилок.

