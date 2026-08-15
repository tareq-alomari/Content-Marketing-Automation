# Content Marketing Automation Workflow with n8n

**Marketing Content Engine — نظام أتمتة تسويق المحتوى باستخدام n8n**

نظام متكامل (Production-Ready) لأتمتة دورة حياة المحتوى بالكامل: من الاستراتيجية والأفكار حتى النشر والتحليلات والتحسين المستمر. مصمم بنمط **Modular** على n8n مع بوابة موافقة بشرية (Human-in-the-Loop) وذكاء اصطناعي متعدد المزودات (Multi-Provider).

[![n8n](https://img.shields.io/badge/n8n-Compatible-ff6d5a)](https://n8n.io)
[![License](https://img.shields.io/badge/License-MIT-blue)](#)
[![Workflows](https://img.shields.io/badge/Workflows-14-success)](#)

---

## 📦 المحتويات

```
.
├── Content-Marketing-Automation-Blueprint.md   ← الوثيقة المعمارية الكاملة (19 قسماً)
├── n8n-workflows/                              ← 14 Workflow جاهزة للاستيراد
│   ├── W00-master-runner.json                  ← تشغيل النظام كاملاً بضغطة واحدة
│   ├── W01-plan-content-engine.json            ← التخطيط وقراءة التقويم
│   ├── W02-idea-generator.json                 ← توليد وتقييم أفكار المحتوى
│   ├── W03-research-pipeline.json              ← البحث الدلالي والكلمات المفتاحية
│   ├── W04-content-production.json             ← إنتاج AI + SEO + Quality Control
│   ├── W05-visual-production.json              ← توليد المرئيات والأصول
│   ├── W06-repurposing-engine.json             ← إعادة استخدام المحتوى لمنصات
│   ├── W07-approval-gate.json                  ← بوابة الموافقة البشرية (Webhook)
│   ├── W08-publishing-distribution.json        ← النشر والجدولة عبر Buffer
│   ├── W09-analytics-collection.json           ← جمع GA4 + Search Console
│   ├── W10-performance-analysis.json           ← تحليل الأداء بالـ AI
│   ├── W11-optimization-decay.json             ← كشف تراجع المحتوى وإنعاشه
│   ├── W12-error-notification.json             ← إدارة الأخطاء والإشعارات المركزية
│   └── sub_ai-orchestrator.json                ← طبقة AI متعددة المزودات
└── docs/
    ├── SETUP.md                                ← دليل التثبيت والإعداد خطوة بخطوة
    └── ARCHITECTURE.md                         ← المعمارية التقنية والمخططات
```

---

## ✨ الميزات الرئيسية

| الميزة | التفاصيل |
|---|---|
| **دورة حياة كاملة** | Strategy → Ideas → Research → SEO → Production → Review → Visuals → Approval → Publish → Distribute → Analytics → Optimize → Repurpose |
| **Closed-Loop** | نتائج التحليل تعود إلى مرحلة التخطيط لتوليد أفكار جديدة |
| **Human-in-the-Loop** | لا نشر تلقائي بدون موافقة بشرية (Slack / Telegram / Email) |
| **AI متعدد المزودات** | OpenAI / Gemini / Claude عبر طبقة تجريد واحدة مع Fallback |
| **AI بذكاء** | يُستخدم فقط حيث يضيف قيمة — الحسابات والتحقق في Code Nodes |
| **Structured Outputs** | كل مخرجات AI بصيغة JSON Schema مُتحقق منها |
| **بلا حلقات لا نهائية** | `MAX_REVISIONS = 2` ، `MAX_RETRIES = 3` مع Exponential Backoff |
| **الأمان** | لا مفاتيح داخل العقد؛ Credentials + Variables + Webhook Signature |
| **مراقبة وتكلفة** | تسجيل كل Execution + تقدير التكلفة + الـ KPIs لكل مرحلة |

---

## 🚀 البداية السريعة

### المتطلبات

- n8n (Self-hosted عبر Docker أو n8n Cloud) — v1.x
- حسابات Google: Sheets API + Google Analytics + Search Console
- حساب Buffer + ربط منصاتك الست (LinkedIn, Instagram, X, Facebook, TikTok, YouTube)
- مفتاح AI واحد على الأقل: OpenAI / Google Gemini / Anthropic Claude
- Slack (اختياري للإشعارات والموافقات)

### التثبيت (5 خطوات)

1. **استورد الـ Workflows** من `n8n-workflows/` بالترتيب التالي:
   `sub_ai-orchestrator` → `W12` → `W01` → `W02` → `W03` → `W04` → `W05` → `W06` → `W07` → `W08` → `W09` → `W10` → `W11` → `W00`
2. **اربط الـ Credentials** (انظر `docs/SETUP.md` للتفاصيل الكاملة).
3. **اضبط المتغيرات** في n8n: `Settings → Variables` (انظر الجدول أدناه).
4. **أعد توجيه عُقد `Execute Workflow`** إلى الـ Workflows الفعلية (بعد الاستيراد تُكتب الأسماء فقط).
5. **فعّل `W12`** كـ Error Workflow من إعدادات أي Workflow، ثم شغّل `W00` لاختبار النظام كاملاً.

### المتغيرات المطلوبة (Variables / Env)

| المتغير | الغرض | مثال |
|---|---|---|
| `GOOGLE_SHEET_ID` | معرف ورقة Google Sheets (التقويم) | `1AbC...xyz` |
| `MODEL_GENERATION` | نموذج التوليد القوي | `gpt-4o` |
| `MODEL_LIGHT` | النموذج الاقتصادي للتحليل | `gpt-4o-mini` |
| `BRAND_VOICE` | صوت العلامة التجارية | `واضح، عملي، يعتمد على البيانات` |
| `SEEO_THRESHOLD` | حد قبول الـ SEO (0-100) | `70` |
| `QUALITY_THRESHOLD` | حد قبول الجودة (0-100) | `70` |
| `MAX_RETRIES` | أقصى محاولات إعادة | `3` |
| `MAX_REVISIONS` | أقصى مراجعات AI | `2` |
| `SLACK_CHANNEL` | قناة الإشعارات | `#content-ops` |
| `APPROVAL_SIGNING_KEY` | مفتاح توقيع Webhook الموافقات | `كلمة سرية` |
| `GA4_PROPERTY_ID` | معرف خاصية Google Analytics 4 | `123456789` |
| `GSC_SITE_URL` | موقع Search Console | `sc-domain:example.com` |

> **مهم:** لا تضع أي مفاتيح API داخل العقد. أنشئ Credential لكل خدمة واستخدم `Credentials` و `Variables` فقط.

---

## 🔄 كيف يعمل النظام؟

```
Google Sheets (Content Calendar)
        │
        ▼
[W01] Plan ──► [W04] AI Production (Draft → SEO → Quality)
        │                    │
        │                    ▼
        │              [W05] Visuals + [W06] Repurpose
        │                    │
        ▼                    ▼
[W07] Human Approval ◄── Webhook (Slack/Telegram)
        │ YES
        ▼
[W08] Publish (Buffer + APIs) ──► LinkedIn, IG, X, FB, TikTok, YouTube
        │
        ▼
[W09] Analytics (GA4 + GSC) ──► [W10] AI Performance Analysis
        │                              │
        ▼                              ▼
[W11] Decay Detection ◄────── [W02] New Ideas (Closed-Loop)
        │
        ▼
[W12] Error Handling (مركزي لكل الفشل)
```

> 📖 الشرح المعماري التفصيلي لكل عقدة وتدفق البيانات: `docs/ARCHITECTURE.md`
> 📖 الوثيقة الكاملة بجميع مراحلها الـ 19: `Content-Marketing-Automation-Blueprint.md`

---

## ⚖️ منهجية التصميم

| المبدأ | التطبيق |
|---|---|
| Modularity | 12+ Workflow مستقل قابل للاختبار، وليس Workflow ضخم واحد |
| AI Only Where Valuable | الحسابات والتحقق والتجميع في Code Nodes |
| Grounded Integrations | لا افتراض وجود API — بدائل HTTP/Hand-off موثقة |
| No Infinite Loops | كل حلقة مراجعة محدودة العدد |
| Security by Default | Credentials + Least Privilege + Signature Webhooks |
| Cost Awareness | Caching + Batch + نموذج اقتصادي للتحليل |

---

## 📊 الـ KPIs

- **المراحل العليا:** Reach، Impressions، Engagement Rate، CTR
- **منتصف القمع:** Conversion Rate، Bounce Rate، Search Position (GSC)
- **النتيجة النهائية:** Leads، Revenue، Cost per Lead
- **تشغيلي:** مدة دورة الموافقة، معدل الفشل، تكلفة التشغيل لكل قطعة

التفاصيل الكاملة في قسم 17 من الـ Blueprint.

---

## 🗺️ خارطة التنفيذ

| المرحلة | المدة | المخرج |
|---|---|---|
| الإعداد | أسبوع 1 | n8n + Credentials + Sheets |
| الأساسيات | أسبوع 1-2 | W01 + W12 + الجدولة |
| الإنتاج | أسبوع 2-3 | W04 + AI Orchestrator |
| المرئيات والتحويل | أسبوع 3-4 | W05 + W06 |
| الموافقات والنشر | أسبوع 4-5 | W07 + W08 |
| التحليلات | أسبوع 5-6 | W09 + W10 |
| الحلقة المغلقة | أسبوع 6-7 | W11 + تغذية الأفكار |
| الصلابة والإنتاج | أسبوع 7-8 | Error Handling + Security + Monitoring |

---

## 📄 الترخيص

MIT — انظر [LICENSE](LICENSE).

## 🤝 المساهمة

المرجو فتح Issue أو Pull Request للتحسينات. الالتزام بقاعدة: لا أسرار في الكود، وكل ميزة جديدة تأتي مع اختبارها في Sandbox.

---

*مشروع أكاديمي — مقرر التسويق بالمحتوى · تم بناؤه كتصميم تنفيذي كامل جاهز للتشغيل على n8n.*