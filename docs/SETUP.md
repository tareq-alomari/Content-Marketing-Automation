# دليل التثبيت والإعداد — SETUP

هذا الدليل يشرح خطوة بخطوة كيفية تشغيل نظام أتمتة تسويق المحتوى على n8n.

---

## 1. تثبيت n8n

### الخيار A — Self-hosted عبر Docker (موصى به)

```bash
mkdir n8n && cd n8n
cat > docker-compose.yml <<'EOF'
version: "3.8"
services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - N8N_USER_MANAGEMENT_JWT_SECRET=${N8N_USER_MANAGEMENT_JWT_SECRET}
    volumes:
      - n8n_data:/home/node/.n8n
volumes:
  n8n_data:
EOF
docker compose up -d
# الواجهة: http://localhost:5678
```

### الخيار B — n8n Cloud

1. أنشئ حساباً في n8n.io
2. تابع الخطوات التالية نفسها

> ✅ ننصح بتثبيت n8n في VPS أو عبر Docker على جهازك المحلي للمشروع الأكاديمي.

---

## 2. إنشاء حسابات وتجهيز الواجهات

| الخدمة | الخطوات | الناتج |
|---|---|---|
| Google Cloud | تفعيل Sheets API + GA4 API + Search Console API | Service Account JSON / OAuth |
| Buffer | إنشاء حساب + ربط القنوات الست | API Key |
| OpenAI (أو Gemini/Claude) | إنشاء API Key | Key |
| Slack | إنشاء App + Webhook/Token + قناة | Token + Channel ID |
| Cloud Storage | S3 أو Google Drive Bucket | Credentials + Bucket Name |

---

## 3. إنشاء ورقة Google Sheets (البنية)

أنشئ ورقة جديدة باسم `Content Marketing Engine` مع **أوراق (tabs)** التالية بأعمدتها:

### `Content Calendar`
```
Content ID | Topic | Content Pillar | Persona | Keyword | Search Intent
Content Type | Platform | Funnel Stage | Publishing Date | Status | Priority
CTA | Target KPI | word_target | secondary_keywords
```

### `Config`
```
industry | pillars | platforms | persona_name | persona_description | brand_voice
best_times | publish_frequency
```

### `Content Ideas`
```
idea_id | title | pillar | persona_id | content_type | search_intent
target_keyword | angle | funnel_stage | composite_score | status
```

### `Content Platforms`
```
item_id | content_id | platform | format | copy | hashtags | best_time | media_ref
char_count | status | published | publish_date
```

### `Content Assets` / `Research` / `Content Approvals` / `Content Analytics` / `Performance Insights` / `Execution Logs`

> يتم إنشاء الأعمدة تلقائياً حسب أول كتابة من الـ Workflows (وضع `append`). ننصح بإنشائها يدوياً للمزيد من الوضوح.

---

## 4. إنشاء الـ Credentials في n8n

من `Credentials → Add Credential` أنشئ:

| الاسم | النوع | الاستخدام |
|---|---|---|
| Google Sheets (Service Account) | Google Sheets OAuth2 | كل عُقد Sheets |
| Google Analytics | Google Analytics OAuth2 | W09 |
| Google Service Account (generic) | Google Service Account | GSC API (HTTP) |
| OpenAI (API Key) | HTTP Header Auth | sub_ai-orchestrator |
| Buffer API | Buffer API | W08 |
| Slack App | Slack API | الإشعارات/الموافقات |
| S3 / Cloud Storage | S3 | W05 |

> 🔒 بعد الإنشاء، عدّل كل عقدة واستبدل `REPLACE_WITH_CREDENTIAL_ID` بمعرف الـ Credential الفعلي.

---

## 5. ضبط المتغيرات (Variables)

n8n → `Settings → Variables` → أضف:

```text
GOOGLE_SHEET_ID          = <معرف الورقة>
MODEL_GENERATION          = gpt-4o
MODEL_LIGHT               = gpt-4o-mini
BRAND_VOICE               = واضح، عملي، يعتمد على البيانات
SEEO_THRESHOLD            = 70
QUALITY_THRESHOLD         = 70
MAX_RETRIES               = 3
MAX_REVISIONS             = 2
SLACK_CHANNEL             = #content-ops
APPROVAL_SIGNING_KEY      = <كلمة سرية عشوائية>
GA4_PROPERTY_ID           = <رقم الخاصية>
GSC_SITE_URL              = sc-domain:example.com
IMAGE_API_URL             = https://api.openai.com/v1/images/generations
IMAGE_MODEL               = dall-e-3
```

---

## 6. استيراد الـ Workflows

1. n8n → `...` (Workflows) → **Import from File**
2. استورد بالترتيب:
   ```
   1. sub_ai-orchestrator.json
   2. W12-error-notification.json
   3. W01-plan-content-engine.json
   4. W02-idea-generator.json
   5. W03-research-pipeline.json
   6. W04-content-production.json
   7. W05-visual-production.json
   8. W06-repurposing-engine.json
   9. W07-approval-gate.json
   10. W08-publishing-distribution.json
   11. W09-analytics-collection.json
   12. W10-performance-analysis.json
   13. W11-optimization-decay.json
   14. W00-master-runner.json
   ```
3. أعد توجيه كل عقدة `Execute Workflow` إلى الـ Workflow الصحيح من القائمة (اختر `Workflow → From Database`).
4. فعّل `W12` كـ **Error Workflow**: من أي Workflow → `Settings → Error Workflow → W12-error-notification`.

---

## 7. أول تشغيل (اختبار)

### Sandbox أولاً
- أنشئ قناة Buffer اختبارية أو استخدم وضع الـ Draft.
- استخدم ورقة `Content Calendar` فيها صف واحد حالة `draft` وتاريخ استحقاق اليوم.

### التشغيل اليدوي
1. افتح `W04-content-production` → **Execute Workflow** → تحقق من مسودة + SEO Score + Quality Score.
2. افتح `W07-approval-gate` → انسخ Webhook URL → أرسل:
   ```bash
   curl -X POST <WEBHOOK_URL> \
     -H "Content-Type: application/json" \
     -d '{"content_id":"C-2026-001","decision":"approve","reviewer":"test"}'
   ```
3. افتح `W00-master-runner` → Execute → شاهد تشغيل السلسلة كاملة.

### السيناريو الكامل
شغّل `W00`، ثم تحقق من `Execution Logs` لرؤية حالة كل خطوة.

---

## 8. التشغيل الإنتاجي

- **فعّل الـ Schedule Triggers** على: W01 (يومي)، W02 (أسبوعي)، W08 (ساعي)، W09 (يومي)، W10 (أسبوعي)، W11 (أسبوعي).
- عطّل W00 اليدوي بعد أن تصبح الجدولة مفعّلة.
- راقب الـ KPIs أسبوعياً من `Performance Insights`.

---

## 9. استكشاف الأخطاء

| المشكلة | السبب المحتمل | الحل |
|---|---|---|
| 401 عند AI | مفتاح OpenAI خاطئ | أنشئ Credential جديداً |
| Sheets فارغ | المعرّف/الورقة خاطئ | تحقق من `GOOGLE_SHEET_ID` واسم الورقة |
| النشر يفشل | Buffer غير مرتبط أو Sandbox | تحقق من Credentials والقنوات |
| JSON غير صالح من AI | نموذج يعيد نصاً | تأكد `response_format=json_object` — موجودة |
| Webhook يرفض | توقيع خاطئ | طابق `APPROVAL_SIGNING_KEY` |
| تكرار المنشور | صف بلا `published=true` | حدّث الورقة يدوياً |

---

## 10. نقاط تحسين الأداء

- استخدم **PostgreSQL** بدل Sheets للإنتاج الكبير (تبديل العقد فقط).
- فعّل **Caching** في الـ AI Orchestrator لإعادة استخدام الاستجابات المتطابقة.
- خفّف جدولة التحليلات للعدّاد إلى يومية فقط عند الحاجة الفعلية.