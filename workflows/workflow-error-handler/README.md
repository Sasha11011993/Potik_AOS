�r�^�f��ئ{Oly�'vî���# Handle Potik AOS workflow failure

Спільний Error Trigger workflow для всіх експортів Potik AOS. Після остаточного збою він формує `ERR-<workflowId>-<executionId>`, записує failed-run до `Automation_Runs` і надсилає Telegram-повідомлення з workflow, execution URL, вузлом і помилкою.

## Налаштування після імпорту

1. Прив’яжіть Google Sheets OAuth2 і Telegram Bot credentials у n8n Credentials.
2. У `Record failed run in Automation_Runs` виберіть документ замість `[REDACTED_GOOGLE_SHEET_ID]` та аркуш `Automation_Runs (Запуски автоматизацій)`.
3. У `Send Telegram failure alert` замініть `[REDACTED_TELEGRAM_CHAT_ID]` на ID потрібного чату.
4. Опублікуйте workflow, а потім у Settings кожного основного workflow виберіть його як **Error Workflow**.

Error Trigger викликається n8n лише для production execution. Ручні й test-запуски перевіряйте у **Executions**; перед увімкненням production-сценаріїв виконайте контрольний збій у тестовому workflow.
