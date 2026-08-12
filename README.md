# Potik AOS — n8n автоматизації

Репозиторій містить експорти workflow та операційну документацію для моніторингу AI-можливостей і технологічних новин.

## Автоматизації

| Workflow | Призначення | Документація |
| --- | --- | --- |
| Збирати AI-проєкти з фриланс-платформ | Збирає релевантні фриланс-проєкти й вакансії з кількох платформ у Google Sheets. | [README](workflows/collect-ai-freelance-opportunities/README.md) |
| Збирати AI-новини з технологічних джерел | Збирає та відбирає AI / automation-новини з технологічних блогів у Google Sheets. | [README](workflows/collect-ai-technology-news/README.md) |
| Analyze AI demand from opportunities | Аналізує клієнтські AI-проєкти, оцінює попит і формує індивідуальний та агрегований звіти в Google Sheets. | [README](workflows/analyze-ai-demand/README.md) |
| AI Technology Analyst | Аналізує технологічні новини, формує Tech Signals і відстежує фактичні токени та вартість OpenAI. | [README](workflows/ai-technology-analyst/README.md) |

## Експорти

- [Фриланс-можливості](workflows/collect-ai-freelance-opportunities.json)
- [AI-новини](workflows/collect-ai-technology-news.json)
- [AI Demand Analyst](workflows/analyze-ai-demand.json)
- [AI Technology Analyst](workflows/ai-technology-analyst.json)
- [AI Opportunity Strategist — Content Ideas](workflows/ai-opportunity-strategist.json) — [README](workflows/ai-opportunity-strategist/README.md)
- [Обробник помилок workflow](workflows/workflow-error-handler.json)

Документація обробника помилок: [README](workflows/workflow-error-handler/README.md).

Усі експорти не містять API-ключів, токенів, OAuth-секретів, Google Sheets ID або Telegram chat ID. Після імпорту в n8n перевірте й прив’яжіть потрібні credentials.

## Імпорт і прив’язування

1. Імпортуйте потрібний workflow і прив’яжіть credentials лише через n8n Credentials: Google Sheets OAuth2, OpenAI Bearer/Auth, Freelancer Header Auth, Freelancehunt Bearer Auth та Telegram Bot.
2. У кожному Google Sheets-вузлі виберіть документ замість `[REDACTED_GOOGLE_SHEET_ID]`, а потім підтвердьте потрібний аркуш.
3. Для Telegram у workflow `Handle Potik AOS workflow failure` замініть `[REDACTED_TELEGRAM_CHAT_ID]` на ID чату та виберіть Telegram Bot credential.
4. Опублікуйте Error Handler і в Settings кожного основного workflow призначте його як **Error Workflow**. Це зв’язування неможливо зробити переносимим export-файлом, бо n8n призначає workflow ID після імпорту.

Error Trigger запускається для production executions. Помилки ручних або тестових запусків потрібно перевіряти у вкладці **Executions**; перед активацією автоматичних запусків протестуйте Error Handler окремим контрольним production execution.

