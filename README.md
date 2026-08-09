# Potik AOS — n8n автоматизації

Репозиторій містить експорти workflow та операційну документацію для моніторингу AI-можливостей і технологічних новин.

## Автоматизації

| Workflow | Призначення | Документація |
| --- | --- | --- |
| Збирати AI-проєкти з фриланс-платформ | Збирає релевантні фриланс-проєкти й вакансії з кількох платформ у Google Sheets. | [README](workflows/collect-ai-freelance-opportunities/README.md) |
| Збирати AI-новини з технологічних джерел | Збирає та відбирає AI / automation-новини з технологічних блогів у Google Sheets. | [README](workflows/collect-ai-technology-news/README.md) |

## Експорти

- [Фриланс-можливості](workflows/collect-ai-freelance-opportunities.json)
- [AI-новини](workflows/collect-ai-technology-news.json)

Усі експорти не містять API-ключів, токенів або OAuth-секретів. Після імпорту в n8n перевірте й прив’яжіть потрібні credentials.
