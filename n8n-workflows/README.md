# n8n Workflows — Marketing Content Engine

هذه المجلد يحتوي على Workflows قابلة للاستيراد في n8n (Import from File / Clipboard).

## طريقة الاستيراد

1. افتح n8n
2. اضغط `...` أعلى يسار (Workflows) ثم **Import from File** أو **Import from Clipboard**
3. استورد الملفات بالترتيب أدناه
4. عدّل الـ Credentials لكل Node (Google, OpenAI, Buffer, Slack, ...)
5. حدّث Variables: `GOOGLE_SHEET_ID`, `BRAND`, `MAX_RETRIES`, `MODEL_*`, `WEBHOOK_SIGNING_KEY`

## ترتيب الاستيراد والتشغيل

| # | الملف | التبعيات |
|---|---|---|
| 1 | `sub_ai-orchestrator.json` | لا تبعيات (يُستدعى من غيره) |
| 2 | `W12-error-notification.json` | لا تبعيات (يُفعَّل كـ Error Workflow) |
| 3 | `W01-plan-content-engine.json` | يستدعي `W04` |
| 4 | `W02-idea-generator.json` | يستدعي `sub_ai-orchestrator` |
| 5 | `W03-research-pipeline.json` | يستدعي `sub_ai-orchestrator` |
| 6 | `W04-content-production.json` | يستدعي `sub_ai-orchestrator` + `W07` |
| 7 | `W05-visual-production.json` | يستدعي `sub_ai-orchestrator` |
| 8 | `W06-repurposing-engine.json` | يستدعي `sub_ai-orchestrator` |
| 9 | `W07-approval-gate.json` | Webhook + Slack/Telegram |
| 10 | `W08-publishing-distribution.json` | Buffer + APIs مباشرة |
| 11 | `W09-analytics-collection.json` | GA4 + GSC + Sheets |
| 12 | `W10-performance-analysis.json` | يستدعي `sub_ai-orchestrator` |
| 13 | `W11-optimization-decay.json` | يستدعي `sub_ai-orchestrator` + `W08` |

## ملاحظات مهمة

- **Execute Workflow Nodes:** عندما تستورد ملفاً يستدعي `sub_ai-orchestrator`، أعد توجيه Node التنفيذ إلى
  الـ Workflow الصحيح (اختَر `Workflow` → `From Database` ثم ابحث باسمه).
- **Credentials:** لن يعمل أي Workflow قبل ربط الـ Credentials الصحيحة (تنشأ مرة واحدة وتُعاد تسميتها).
- **Error Workflow:** فعّل `W12` من إعدادات الـ Workflow عبر `Settings → Error Workflow`.
- **Variables (Environment):** ضعها في `Settings → Variables` أو ملف `.env`:
  - `GOOGLE_SHEET_ID` — معرف ورقة التقويم
  - `BRAND_VOICE` — وصف صوت العلامة
  - `MAX_AI_RETRIES` = 2 ، `MAX_REVISIONS` = 2
  - `MODEL_GENERATION` / `MODEL_LIGHT` — أسماء النماذج
- **Webhook:** بعد نشر `W07` انسخ الـ Webhook URL وضعه في متغير `APPROVAL_WEBHOOK_URL` لاستخدامه في عُقد Slack/Telegram.

## خريطة العقد (للإسناد أثناء الـ Presentation)

راجع `Content-Marketing-Automation-Blueprint.md` قسم 6 و 19 للحصول على الشرح الكامل لكل عقدة.