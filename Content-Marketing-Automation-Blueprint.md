# نظام أتمتة التسويق بالمحتوى — Marketing Content Engine

## Content Marketing Automation Blueprint باستخدام n8n

**وثيقة تنفيذية كاملة — Production-Ready Design**

---

## الفهرس

1. [نظرة تنفيذية (Executive Summary)](#1-نظرة-تنفيذية)
2. [تحليل استراتيجية التسويق بالمحتوى (Business Workflow)](#2-تحليل-استراتيجية-التسويق-بالمحتوى)
3. [المتطلبات النظامية (System Requirements)](#3-المتطلبات-النظامية)
4. [المعمارية التقنية (Technical Architecture)](#4-المعمارية-التقنية)
5. [معمارية Workflows في n8n](#5-معمارية-workflows-في-n8n)
6. [تصميم العُقد Node-by-Node](#6-تصميم-العقد-node-by-node)
7. [نموذج البيانات (Data Model)](#7-نموذج-البيانات-data-model)
8. [معمارية الذكاء الاصطناعي (AI Architecture)](#8-معمارية-الذكاء-الاصطناعي)
9. [معمارية الـ Prompts (Prompt Architecture)](#9-معمارية-الـ-prompts)
10. [نظام الموافقات (Approval System)](#10-نظام-الموافقات)
11. [نظام النشر والتوزيع (Publishing System)](#11-نظام-النشر-والتوزيع)
12. [نظام التحليلات (Analytics System)](#12-نظام-التحليلات)
13. [حلقة التحسين المغلقة (Optimization Loop)](#13-حلقة-التحسين-المغلقة)
14. [إدارة الأخطاء (Error Handling)](#14-إدارة-الأخطاء)
15. [الأمان (Security)](#15-الأمان)
16. [تحسين التكلفة (Cost Optimization)](#16-تحسين-التكلفة)
17. [إطار مؤشرات الأداء (KPI Framework)](#17-إطار-مؤشرات-الأداء)
18. [استراتيجية الاختبار (Testing Strategy)](#18-استراتيجية-الاختبار)
19. [خارطة التنفيذ (Implementation Roadmap)](#19-خارطة-التنفيذ)

---

## 1. نظرة تنفيذية

هذه الوثيقة تصمّم **Marketing Content Engine**: نظام أتمتة متكامل لدورة حياة التسويق بالمحتوى مبنية على **n8n**، يغطي السلسلة الكاملة:

```
Content Strategy → Idea Generation → Research → Keyword Research → Content Planning
→ Editorial Calendar → AI Content Creation → SEO Optimization → Human Review
→ Visual Content → Approval → Publishing → Distribution → Promotion → Analytics
→ Performance Evaluation → Optimization → Repurposing → (من جديد) New Ideas
```

### مبادئ التصميم الأساسية

| المبدأ | القرار |
|---|---|
| **Modularity** | النظام 12 Workflow مستقل وقابل للاختبار، وليس Workflow واحد ضخم |
| **Human-in-the-Loop** | لا نشر تلقائي كامل — بوابة موافقة بشرية قبل كل نشر |
| **AI حيث يضيف قيمة فقط** | لا يُستخدم AI للعدّ أو التحويل أو التحقق — تُستخدم Code Nodes |
| **Structured Outputs** | كل مخرجات AI بصيغة JSON Schema مُتحقق منها بعد كل Node |
| **Closed-Loop** | نتائج التحليلات تعود إلى مرحلة التخطيط لتوليد أفكار جديدة |
| **No Infinite Loops** | كل حلقة إعادة لها `Max Retry Count` (افتراضياً 3) |
| **Security by Default** | لا مفاتيح API داخل العُقد؛ كل الأسرار في Credentials |
| **Multi-Provider AI** | دعم OpenAI + Gemini + Claude عبر طبقة تجريد |

### التكوين الافتراضي (Primary Configuration)

| عنصر | الاختيار |
|---|---|
| مصدر الحقيقة (Source of Truth) | **Google Sheets** — تقويم المحتوى والعمليات |
| منصات النشر الست | LinkedIn, Instagram, X (Twitter), Facebook, TikTok, YouTube |
| جدولة النشر | **Buffer** (Node رسمي) + **APIs مباشرة** عند توفرها |
| المحتوى المرئي | صور AI + تسليم قوالب Canva/HTML |
| التحليلات | Google Analytics 4 + Google Search Console + Platform Insights |
| قواعد البيانات | Google Sheets (للعرض) / PostgreSQL (لإنتاجي) — قابل للتبديل |

---

## 2. تحليل استراتيجية التسويق بالمحتوى

### 2.1 Business Goals → Marketing Goals → Content Goals

| Business Goal | Marketing Goal | Content Goal |
|---|---|---|
| نمو الإيرادات | توليد Leads مؤهلين | محتوى Funnel Middle/Bottom |
| بناء الوعي | زيادة الوصول والمتابعين | محتوى Top-of-Funnel تعليمي |
| تمييز العلامة | ترسيخ Voice و Trust | محتوى Evergreen + إثبات الخبرة |
| تقليل كلفة الاكتساب | تحسين SEO العضوي | محتوى Keyword-Driven |

### 2.2 الجمهور المستهدف (Target Audience)

يُعرَّف الجمهور في **Config Sheet** داخل Google Sheets كمتغيرات (وليس كود ثابت):
- الشريحة الديموغرافية: العمر، المنطقة، اللغة
- السلوكية: المنصات، أوقات النشاط، نوع المحتوى المفضّل
- الاحتياجات والألم (Pain Points)
- مرحلة الوعي: Unaware / Problem Aware / Solution Aware / Product Aware / Most Aware

### 2.3 شخصيات العملاء (Buyer Personas) — 2-3 شخصيات

يُخزَّن كل شخصية كصف في جدول `personas` ويُمرَّر إلى AI كـ Context في كل عملية توليد:

```json
{
  "persona_id": "P01",
  "name": "سارة — مسوقة رقمي",
  "demographics": "27 سنة، الرياض، لغة عربية",
  "goals": "بناء محتوى يعتمد على البيانات دون فريق كبير",
  "pain_points": "وقت محدود، لا رؤية واضحة للنتائج، محتوى متكرر",
  "preferred_platforms": ["LinkedIn", "Instagram"],
  "funnel_stage": "Solution Aware",
  "tone_preferences": "عملي، مدعوم بأرقام، بدون مبالغة"
}
```

### 2.4 Customer Journey → Content Mapping

| المرحلة | البحث | نوع المحتوى | المقياس |
|---|---|---|---|
| الوعي (Awareness) | مشكلة وحل عام | مقالات تعليمية، Reels، نصائح | Impressions, Reach |
| التفكير (Consideration) | مقارنات، دراسات حالة | مقارنات، إرشادات، Webinar | Engagement, Clicks |
| القرار (Decision) | مراجعات، عروض | دراسة حالة، عرض، FAQ | CTA Clicks, Leads |

### 2.5 Content Pillars

4-6 ركائز محتوى تمثّل مجالات الخبرة وتُخزَّن في جدول `content_pillars`:

```
Pillar 1: تعليمي (Educational)   — حلول مشاكل الجمهور
Pillar 2: إلهامي (Inspirational) — قصص نجاح، Behind-the-scenes
Pillar 3: ترويجي (Promotional)   — منتجات/خدمات بحدود صارمة (≤20%)
Pillar 4: مجتمعي (Community)     — تفاعل، أسئلة، آراء الجمهور
Pillar 5: بياناتي (Data-Driven)  — أبحاث، أرقام، دراسات
```

**قاعدة 80/20:** 80% قيمة تعليمية/إلهامية و 20% ترويج — تُفرض في مرحلة التخطيط وليس الإنتاج.

### 2.6 البحث عن الكلمات (Keyword Research)

- **Primary Keyword:** الكلمة الرئيسية للموضوع
- **Secondary Keywords:** 3-5 كلمات داعمة
- **Long-tail Keywords:** عبارات أطول أقل تنافساً
- **Semantic Keywords:** كلمات مرتبطة دلالياً (Latent Semantic Indexing)
- **Search Intent:** Informational / Navigational / Transactional / Commercial

### 2.7 قنوات التوزيع (Distribution Channels)

| المنصة | دورها | تنسيق المحتوى |
|---|---|---|
| LinkedIn | B2B، قيادة فكرية | مقالات، نصوص طويلة، كاروسيل |
| Instagram | بصرية، نموّ الوعي | صور، Reels، Stories |
| X (Twitter) | أخبار، آراء، نقاش | نصوص قصيرة، خيوط (Threads) |
| Facebook | مجتمع، إعلانات | منشورات، صور، فيديو |
| TikTok | وصول واسع، فيديو قصير | Reels قصيرة |
| YouTube | محتوى طويل، SEO فيديو | فيديوهات تعليمية |

### 2.8 تردد النشر (Publishing Frequency)

قاعدة افتراضية (قابلة للتكوين):
- LinkedIn: 3-5 أسبوعياً · Instagram: 4-5 أسبوعياً · X: 5-10 أسبوعياً
- Facebook: 3-4 أسبوعياً · TikTok: 3-4 أسبوعياً · YouTube: 1-2 أسبوعياً

### 2.9 أهداف التحويل (Conversion Goals)

كل قطعة محتوى تحمل **CTA** واحد محدد (اشتراك، تنزيل، حجز، رابط مقال، متابعة) — يخزّن في عمود `CTA` في التقويم ويُتحقق منه في Quality Check.

---

## 3. المتطلبات النظامية

### 3.1 المتطلبات الوظيفية (Functional)

| # | المتطلب | الوصف |
|---|---|---|
| FR-1 | قراءة تقويم المحتوى | جلب عناصر التقويم من المصدر المركزي بشكل دوري |
| FR-2 | توليد أفكار مقيمة | AI يولّد أفكاراً ويسجّل كل فكرة مع Score |
| FR-3 | بحث وتكامل كلمات مفتاحية | جمع أسئلة/مواضيع/كلمات ذات صلة |
| FR-4 | إنتاج محتوى بسياق كامل | AI يتلقى Brand Voice + Persona + Keywords |
| FR-5 | تحليل SEO و QC | درجات قابلة للمقارنة مع Thresholds |
| FR-6 | موافقة بشرية | عبر Slack/Telegram/Email بأزرار قبول/رفض |
| FR-7 | إنتاج مرئيات | توليد صور وتتبع الأصول |
| FR-8 | إعادة استخدام (Repurposing) | تحويل قطعة لعدة صيغ |
| FR-9 | نشر وجدولة | عبر Buffer أو APIs مباشرة |
| FR-10 | جمع تحليلات | GA4 + GSC + منصات اجتماعية |
| FR-11 | تحليل أداء بالـ AI | إجابات عن 12 سؤالاً تحليلياً |
| FR-12 | اكتشاف تراجع المحتوى | مراقبة Decay وإعادة إنعاش |

### 3.2 المتطلبات غير الوظيفية (Non-Functional)

| المتطلب | المواصفة |
|---|---|
| Availability | وقت تشغيل 99%؛ Retry تلقائي لفشل الواجهات |
| Reliability | لا حلقة لا نهائية؛ معالجة صريحة للفشل في كل مرحلة |
| Security | OAuth + Secrets في Credentials + Least Privilege |
| Scalability | تقسيم عبر Sub-workflows و RabbitMQ/Redis للعمل الكبير |
| Observability | تسجيل كل Execution في Logging |
| Cost | كشف استهلاك Tokens وتكلفة تقديرية لكل تشغيل |
| Testability | كل Sub-workflow قابل للاختبار المستقل |

---

## 4. المعمارية التقنية

### 4.1 المكوّنات (Components)

```
┌─────────────────────────── n8n (Self-hosted / Cloud) ──────────────────────────┐
│                                                                                │
│  Triggers ──► Workflow Orchestrator ──► Sub-workflows (تنفيذ كلي أو جزئي)      │
│                                                     │                          │
│  ┌──────────────────────────────────────────────────┴───────────────┐          │
│  │  AI Layer (طبقة تجريد متعددة المزودات)                            │          │
│  │  OpenAI │ Gemini │ Claude — عبر نفس الـ Interface                │          │
│  └──────────────────────────────────────────────────┬───────────────┘          │
│                                                     │                          │
│  Integration Layer (واجهات خارجية)                                            │
│  Google Sheets │ Buffer │ GA4 │ GSC │ LinkedIn │ Meta │ X │ TikTok │ YouTube │
│  Slack/Telegram/Email │ Cloud Storage │ Postgres                             │
└────────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 طبقة تجريد الـ AI (Multi-Provider Abstraction)

لعدم الاعتماد على مزود واحد:

- **AI Orchestrator Sub-workflow** يستقبل: `{task_type, payload, model_preference}`.
- **Router (Switch Node)** يرسل كل `task_type` إلى الـ Model المناسب:
  - توليد (Generation) → نموذج قوي (مثل `gpt-4o` / `gemini-2.0-pro` / `claude-sonnet`)
  - تحليل وتصنيف (Classification/Scoring) → نموذج اقتصادي (`gpt-4o-mini` / `gemini-2.0-flash` / `claude-haiku`)
  - استخراج بيانات (Extraction) → Code/Regex حيثما أمكن، وإلا نموذج خفيف
- **Fallback Chain:** إذا فشل المزود الأساسي → التبديل للمزود الثاني تلقائياً (ضمن حد أقصى للمحاولات).
- **Cache:** إعادة استخدام نفس الاستدعاء عند تكرار نفس الـ Input (Hash للـ payload).

### 4.3 تدفق البيانات (Data Flow)

```
Google Sheets (Calendar) ──► n8n Plan WF ──► Due item
        │
        ▼
Sub-workflow: Content Production (AI + SEO + QC)
        │
        ▼
Approval (Human) ── YES ──► Publishing WF ──► Buffer / APIs ──► Platforms
        │ NO ──► Revision loop (max 2)
        ▼
Analytics WF (Schedule) ──► GA4 + GSC + Platform Insights ──► Performance DB
        │
        ▼
Analysis WF (AI) ──► Insights ──► Optimization Loop ──► Strategy (جديد)
```

### 4.4 بيئات متعددة (Environment Separation)

- **Development:** n8n بدون نشر فعلي، بيانات وهمية (Mock), `NODE_ENV=development`
- **Production:** عقد نشر مفعّلة، قوالب محتوى حقيقية
- **الفصل:** عبر Variables (`dev` vs `prod`) — لا تُكتب القيم في العُقد إطلاقاً.

---

## 5. معمارية Workflows في n8n

النظام مقسّم إلى **12 Workflow** مستقل (قابل للاختبار والتوسع):

| # | الاسم | Trigger | الوظيفة |
|---|---|---|---|
| W01 | `plan-content-engine` | Schedule + Manual | قراءة التقويم، تجهيز عناصر الاستحقاق، التوجيه |
| W02 | `idea-generator` | Schedule (weekly) + Manual | توليد وتقييم أفكار المحتوى |
| W03 | `research-pipeline` | Sub-workflow + Manual | بحث دلالي، أسئلة، كلمات طويلة |
| W04 | `content-production` | Sub-workflow | توليد AI + SEO + Quality Control |
| W05 | `visual-production` | Sub-workflow | صور + مرئيات + تتبع أصول |
| W06 | `repurposing-engine` | Sub-workflow | تحويل القطعة إلى 6+ صيغ |
| W07 | `approval-gate` | Sub-workflow + Webhook | موافقة بشرية (Slack/Telegram/Email) |
| W08 | `publishing-distribution` | Schedule + Sub-workflow | نشر وجدولة عبر Buffer + APIs |
| W09 | `analytics-collection` | Schedule (daily/weekly) | جلب GA4 + GSC + Platform Insights |
| W10 | `performance-analysis` | Schedule (weekly/monthly) | AI تحليلي وتوليد رؤى |
| W11 | `optimization-decay` | Schedule (weekly) | كشف التراجع وإنعاش المحتوى |
| W12 | `error-notification` | Error Trigger | تسجيل الأخطاء وإخطار Admin |

### قواعد معمارية الـ Workflows

1. **Orchestrator:** كل تدفق طويل يستخدم `Execute Workflow` للـ Sub-workflows (تحميل السياق وقابلية الفشل المستقلة).
2. **Execution Mode:** الافتراضي `own` لكل Workflow؛ للمعالجة الجماعية `queue` (مع Redis).
3. **Naming Convention:** `area-action` (مثل `content-generate-draft`).
4. **لا عُقد مشتركة:** كل Workflow يعرّف Credentials الخاصة به من المتغيرات.

---

## 6. تصميم العُقد Node-by-Node

### 6.1 W01 — Content Planning (التخطيط)

```
[Schedule Trigger] cron: 0 7 * * * (يومياً 07:00)
        │
        ▼
[Google Sheets] Read rows (Content Calendar, filter status=draft & date<=today+3)
        │
        ▼
[Code] Normalize payload → schema {content_id, topic, pillar, keyword, ...}
        │
        ▼
[IF] is_due && has_all_required_fields
   ├─ YES ─► [Execute Workflow] content-production (item واحد)
   └─ NO  ─► [Google Sheets] Update status=missing_data + log warning
```

| Property | القيمة |
|---|---|
| Node Name | `Calendar Fetch` |
| Node Type | `Google Sheets → Read rows` |
| Purpose | جلب عناصر التقويم المستحقة |
| Input | Sheet: `Content Calendar`, tab: `Plan` |
| Output | JSON items مطابقة للمخطط |
| Credentials | Google Service Account / OAuth2 (Sheets API) |
| Error Handling | Stop + log؛ Retry 2× (بعد 30 ثانية) |
| Conditions | `status == "draft"` و `publish_date - today <= buffer_days` |

### 6.2 W02 — Idea Generation

```
[Schedule Trigger] cron: 0 6 * * 1 (أسبوعياً)
        │
        ▼
[Google Sheets] Read Config (pillars, personas, past topics) + Content Calendar (منع التكرار)
        │
        ▼
[Code] Build dedup set + context payload
        │
        ▼
[AI Orchestrator] task=idea_generation (JSON Schema: 10 ideas × {title, pillar, persona, intent, angle})
        │
        ▼
[Code] Validate JSON schema + Score التلقائي
        │
        ▼
[IF] score >= 60 ──► [Google Sheets] Append to Ideas (status=review)
        └─ < 60  ──► Drop + log
```

**التقييم التلقائي (التحقق منه بالـ Code):** يُحسب Composite Score:

```
Score = 0.20*Relevance + 0.20*SearchPotential + 0.15*AudienceValue
      + 0.15*BusinessValue + 0.10*CompetitionInverse + 0.10*ContentPotential + 0.10*ConversionPotential
```

(يستخرج AI قيماً رقمية 1-10 لكل بعد، ويحسب الجمع بالـ Code وليس بالـ AI.)

### 6.3 W03 — Research Pipeline

```
[Execute Workflow] / [Manual Trigger]
        │
        ▼
[HTTP Request] Keyword suggestion source (free: Google autocomplete / Keyword Planner via Sheets)
        │
        ▼
[Google Sheets] Read related keywords sheet
        │
        ▼
[Code] Merge + dedup + cluster keywords by intent
        │
        ▼
[AI Orchestrator] task=research (استخراج أسئلة الجمهور، مواضيع مرتبطة، فجوات محتوى)
        │
        ▼
[Code] Validate schema → save to sheet `Research`
```

> **ملاحظة تكامل:** لا نفترض وجود API لكلمات مفتاحية احترافية (مثل SEMrush/Ahrefs) — التصميم يتحقق من الـ API عبر HTTP Request فإذا فشل يتجه لمصدر بديل (صفحة نتائج البحث / ورقة يدوية). التكامل الأساسي: **Google Sheets بيانات بحثية جاهزة + أسئلة AI** وليس اعتماداً على خدمة مفترضة.

### 6.4 W04 — Content Production (الأساسية)

```
[Execute Workflow] (input: content item)
        │
        ▼
[Code] Build Generation Context (brand_voice + persona + pillar + keywords + intent + CTA + format)
        │
        ▼
[AI Orchestrator] task=draft_generation → Structured JSON
   { title, sections[{h2,h3,content}], summary, faqs[], cta_copy, suggested_slug, meta_desc }
        │
        ▼
[Code] JSON Schema Validation + Length checks
        │
        ▼
[AI Orchestrator] task=seo_analysis → { seo_title, meta_desc, slug, h1, h2[], h3[], faq[], internal_links[], external_links[], alt_text[], schema_type, seo_score }
        │
        ▼
[Code] Compute SEO Score (0-100) من مدخلات AI + فحوص برمجية
        │
        ▼
[IF] seo_score >= 70
   ├─ YES ─► [AI Orchestrator] task=quality_check → { quality_scores, issues[], overall_score }
   │          │
   │          ▼
   │        [IF] overall >= 70 && no critical issues ──► PASS ──► [Visual Production] / [Approval]
   │          └─ FAIL ─► [AI Orchestrator] task=revision (retry count) ──► Recheck (Max 2)
   └─ NO  ──► [AI Orchestrator] task=seo_revision ──► Recheck (Max 2) ──► ESCALATE
```

**Quality Scores (weighted):**

```
Overall = 0.25*ContentQuality + 0.20*SEO + 0.15*Readability + 0.15*BrandCompliance
        + 0.10*Originality + 0.10*Structure + 0.05*CTA_Clarity
```

**الفحوص البرمجية (بدون AI):** كشف التكرار ضد محتوى موجود (hash/similarity عبر Code)، حساب عدد الكلمات، التحقق من وجود CTA، التحقق من وجود الكلمة الرئيسية في العنوان.

### 6.5 W05 — Visual Production

```
[Code] Determine platform dimensions per format (map table)
        │
        ▼
[AI Orchestrator] task=image_generation → prompt نصي حسب Brand Kit
        │
        ▼
[HTTP Request] Image gen API (Stable Diffusion / DALL·E / Flux) → binary
        │
        ▼
[Code] Resize / crop لجميع المقاسات المطلوبة (Featured, Square, Story, Thumbnail)
        │
        ▼
[Cloud Storage] (S3/GDrive) Upload → public URL
        │
        ▼
[Google Sheets] Append to Content Assets (content_id, type, url, platform, dims)
```

**تكامل القوالب:** Canva عبر API (عند توفر مفتاح) أو تسليم HTML Template — في التكوين الافتراضي: **AI ينتج الصور + يحدد قالب Canva** ثم يُرسل معرضاً يدوياً (نقطة تحكم بشرية مضمّنة).

### 6.6 W06 — Repurposing Engine

```
Input: المقال المعتمد + Asset URLs
        │
        ▼
[Code] Split content → sections
        │
        ▼
[AI Orchestrator] task=repurpose_blog
   → 1 version لكل صيغة: {platform, format, copy, hashtags[], best_time, media_ref}
   (LinkedIn post ×3 | X post ×5 | IG caption ×5 | FB post ×3 | Newsletter | Video script ×3 | Carousel | FAQ)
        │
        ▼
[Code] Validate per-platform constraints (char limits, hashtag count) + fix/trim
        │
        ▼
[Google Sheets] Append إلى `Content Platforms` (كل صف = منشور منصة)
        │
        ▼
[Approval Gate] (اختياري: batch approval لكل صيغة)
```

### 6.7 W07 — Approval Gate

```
[Webhook] receive: {content_id, decision: approve/reject/revise, comment}
        │
        ▼
[IF] decision
   ├─ approve ─► [Google Sheets] status=approved ─► [Publishing WF]
   ├─ reject   ─► [Google Sheets] status=rejected + notify author
   └─ revise   ─► [Google Sheets] status=revision + comment ─► [Content Production] (max 2)
        │
        ▲
        │ (from production, awaiting approval)
[Slack] PostBlocks: preview + scores + buttons (approve / reject / revise) 
[Telegram] Alternativa للعرض والنقر
```

- **كل إجراء بشري يُسجَّل** في `content_approvals` (من، متى، قرار، تعليق).
- **Timeout:** إن لم يُرد الشخص خلال 48 ساعة → إشعار تذكير؛ بعد 5 أيام → يتصعد إلى المدير.

### 6.8 W08 — Publishing & Distribution

```
[Schedule Trigger] cron: 0 8 * * * + [Manual]
        │
        ▼
[Google Sheets] Read status=approved & publish_date=today
        │
        ▼
[Code] Build per-platform payloads (from repurposed rows) + validation (media exists)
        │
        ▼
[IF] platform == buffer_supported ──► [Buffer] Create Post (text, media URL, schedule)
   └─ platform == direct_api ──► [HTTP Request] Platform API (LinkedIn/X/Meta/...) 
        │
        ▼
[IF] success
   ├─ YES ─► [Google Sheets] Update publication status + post URL + log
   └─ NO  ─► Retry (2×) ──► FAIL ─► [Error WF] + mark needs_manual
        │
        ▼
[Slack] Notify team (published summary)
```

**Distribute flow (بعد نشر المقال الأساسي):** توليد منشورات وسائل + جدولتها في Buffer + إرسال Newsletter (عبر Mailchimp/API) + إشعار الفريق — كل ذلك عبر نفس الـ Workflow بفرع موازٍ.

### 6.9 W09 — Analytics Collection

```
[Schedule Trigger] cron: 0 5 * * * (يومياً) + weekly
        │
        ▼
[Google Analytics] GA4 RunReport (sessions, users, pageviews, engagement, conversions)
        │
        ▼
[Google Search Console] Performance query (clicks, impressions, ctr, position)
        │
        ▼
[HTTP Request] Platform Insights (حسب توفر API: LinkedIn API / Meta Graph / TikTok Analytics)
        │
        ▼
[Code] Merge by content_id + normalize (rename, dedup, fill missing=0)
        │
        ▼
[PostgreSQL / Sheets] Append إلى Content Analytics
```

> **ملاحظة:** منصات لا توفر API تحليلية كاملة لكل الأغراض — عند غيابها، يُجمع Engagement يدوياً (Code تعبئة placeholder) أو من Buffer Insights. لا يفترض التصميم وجود بيانات غير متوفرة.

### 6.10 W10 — Performance Analysis

```
[Schedule Trigger] weekly
        │
        ▼
[PostgreSQL] Query aggregation (best/worst by pillar, type, platform, time, CTA)
        │
        ▼
[Code] Build dataset + stats summary (لا يعتمد على AI للأرقام)
        │
        ▼
[AI Orchestrator] task=performance_analysis → JSON { insights[], top_performers[], weak_performers[], opportunities[], recommendations[] }
        │
        ▼
[Code] Validate schema
        │
        ▼
[Slack] Report summary ─► [Google Sheets] Append Performance Insights
        │
        ▼
[Execute Workflow] optimization-decay (للأجزاء الضعيفة/القديمة)
```

**الأسئلة الـ 12 التي يجيب عنها AI** (تجدونها في قسم الـ Prompts — §9).

### 6.11 W11 — Optimization & Decay Detection

```
[Schedule Trigger] weekly
        │
        ▼
[PostgreSQL] Query: traffic/engagement/position trend + content age
        │
        ▼
[IF] decay_detected (تراجع ≥30% خلال 8 أسابيع) أو age > 12 شهراً
   ├─ YES ─► [AI Orchestrator] task=content_audit → issues + update_recommendations
   │          │
   │          ▼
   │        [AI Orchestrator] task=content_refresh → نسخة محدثة (title, intro, keywords, links, CTA)
   │          │
   │          ▼
   │        [Approval Gate] ── approve ──► [Republishing via Publishing WF]
   └─ NO  ──► Log "healthy"
```

### 6.12 W12 — Error & Notification (مركزي)

```
[Error Trigger] (عند فشل أي Workflow / Node)
        │
        ▼
[Code] Capture: workflow_id, execution_id, node, error_message, stack, timestamp
        │
        ▼
[Google Sheets] Append to Logs (أو Postgres)
        │
        ▼
[IF] error is transient (rate limit / timeout / 429/5xx)
   ├─ YES ─► [Execute Workflow] Retry مع Exponential Backoff (30s, 60s, 120s) حتى 3 مرات
   └─ NO  ─► [Slack/Telegram] إخطار Admin + وضع علامة needs_manual_attention
```

---

## 7. نموذج البيانات (Data Model)

### 7.1 الجداول (PostgreSQL — أو Sheets tab لكل جدول في الوضع التجريبي)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              المستودع المركزي                                │
│                                                                             │
│  config                    ── brand_voice, pillars, thresholds, frequency   │
│  personas                  ── person_id, name, demographics, pain, funnel   │
│  content_pillars           ── pillar_id, name, goal, allowed_ratio          │
│  content_calendar          ── content_id, topic, pillar, persona, keyword,   │
│                               intent, type, platform, funnel, publish_date,  │
│                               status, priority, cta, target_kpi             │
│  content_ideas             ── idea_id, title, pillar, persona, score, status │
│  content                    ── content_id, title, body, slug, summary,       │
│                                seo_score, quality_score, word_count, version │
│  content_versions           ── version_id, content_id, body_hash, change_log │
│  content_keywords           ── keyword_id, content_id, keyword, type, intent │
│  content_platforms          ── item_id, content_id, platform, format, copy,  │
│                                hashtags, best_time, media_ref, status        │
│  content_publications       ── pub_id, content_id, platform, post_url,       │
│                                published_at, scheduled_at, status            │
│  content_analytics          ── metric_id, pub_id, platform, date, impressions,│
│                                reach, engagement, clicks, ctr, saves, ...    │
│  content_performance        ── perf_id, content_id, period, rank, trend,     │
│                                ai_score, ai_summary                          │
│  content_reviews            ── review_id, content_id, reviewer, score, notes │
│  content_approvals          ── approval_id, content_id, decision, comment,   │
│                                by_who, at_when                              │
│  content_assets             ── asset_id, content_id, type, url, platform,    │
│                                dims, alt_text, brand_compliant               │
│  campaigns                  ── campaign_id, name, goal, funnel, budget       │
│  audiences                  ── audience_id, segment, demographics, behavior  │
│  execution_logs             ── log_id, workflow_id, exec_id, content_id,     │
│                                start, end, status, error, api, model,        │
│                                tokens, cost_est, retry_count                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 العلاقات (Relations)

```
campaigns 1───* content_calendar *───1 content (via content_id)
content 1───* content_versions
content 1───* content_keywords
content 1───* content_platforms (repurposed pieces)
content_platforms 1───* content_publications
content_publications 1───* content_analytics
content 1───* content_reviews / content_approvals / content_assets
content_calendar 1───* content_analytics (aggregate view for dashboards)
config ── used by: personas, pillars, calendars (FK مرجعية)
```

### 7.3 صيغة JSON لكائن محتوى (Canonical Schema)

```json
{
  "content_id": "C-2026-041",
  "topic": "كيف تبني تقويم محتوى في 30 دقيقة",
  "pillar": "تعليمي",
  "persona": "P01",
  "keyword": "تقويم المحتوى",
  "search_intent": "informational",
  "content_type": "blog",
  "platforms": ["linkedin", "instagram", "x"],
  "funnel_stage": "consideration",
  "publish_date": "2026-08-20",
  "status": "draft",
  "priority": "high",
  "cta": "download_guide",
  "target_kpi": "leads",
  "seo": { "title": "...", "meta_description": "...", "slug": "...", "h1": "...", "h2": ["..."], "faq": ["..."] },
  "quality": { "overall": 82, "seo": 78, "readability": 85, "brand": 90, "originality": 88 },
  "assets": ["https://.../featured.png"]
}
```

---

## 8. معمارية الذكاء الاصطناعي

### 8.1 مبدأ "AI حيث يضيف قيمة فقط"

| المهمة | الطريقة | السبب |
|---|---|---|
| حساب درجات Scores | Code Node | قطعية ويمكن اختبارها |
| التحقق من الـ Schema | Code Node | لا يعتمد على AI |
| التحقق من الحروف/الشروط | Code Node | خالية من الأخطاء |
| إلغاء التكرار | Hash + Similarity (Code) | قابل للقياس |
| توليد الأفكار والمحتوى | AI | قيمة إبداعية حقيقية |
| تحليل الأداء واستخراج الرؤى | AI | توليف عبر سياق واسع |
| بحث وتجميع الأسئلة | AI + APIs | جمع غير منظم |
| اقتراح الروابط الداخلية | AI (سرد أولي) + Code (تحقق) | دقة |

### 8.2 الوكلاء (Agents) ودور كل واحد

| الوكيل | الوظيفة | Model Hint |
|---|---|---|
| `idea_generator` | توليد 10 أفكار مقيمة | قوي |
| `researcher` | أسئلة/مواضيع/فجوات/كلمات طويلة | متوسط |
| `draft_writer` | كتابة المسودة بالسياق الكامل | قوي |
| `seo_optimizer` | بناء SEO kit + score | متوسط |
| `quality_reviewer` | تقييم الجودة وإيجاد العيوب | قوي |
| `revision_agent` | مراجعة المحتوى حسب ملاحظات محددة | قوي |
| `repurposer` | تحويل القطعة لصيغ منصات | قوي |
| `performance_analyst` | إجابة الأسئلة الـ12 التحليلية | قوي |
| `decay_auditor` | تشخيص تراجع المحتوى واقتراح تحديث | متوسط |
| `image_director` | توليد أوصاف الصور (Image prompts) | متوسط |

### 8.3 Validation بعد كل AI Node (إلزامي)

```
AI Node ──► [Code] JSON Schema validation
             ├─ valid ──► continue
             ├─ invalid (parsable) ──► repair via Code (defaults) ──► retry 1×
             └─ invalid (unparsable) ──► retry 1× بأمر صريح "output valid JSON only"
                  └─ فشل ──► mark failed + escalate (لا تكرار لا نهائي)
```

---

## 9. معمارية الـ Prompts

> منهجية عامة لكل Prompt: **Role → Context (بيانات حقيقية) → Task → Output Schema → Constraints → Tone**.

### 9.1 Prompt لتوليد الأفكار (`idea_generator`)

```
الدور: أنت مدير استراتيجية محتوى خبير في {industry}.

السياق:
- Content Pillars: {pillars}
- Persona المستهدفة: {persona}
- الأهداف التسويقية: {goals}
- موضوعات منشورة بالفعل (لا تكررها): {existing_topics}
- قناة النشر: {platforms}

المهمة:
اقترح 10 أفكار محتوى جديدة ومتنوعة تشمل الأنواع التالية:
Blog, Social Post, LinkedIn Post, Instagram Post, Short Video/Reel,
Educational, Promotional (≤2), Evergreen, Trending, FAQ, Problem/Solution.

لكل فكرة أعد:
{ "title", "pillar", "persona_id", "content_type", "search_intent",
  "target_keyword", "angle", "funnel_stage",
  "scores": { relevance, search_potential, audience_value,
              business_value, competition_inverse, content_potential, conversion_potential } }

القواعد:
- تجنب المواضيع المكررة.
- اجعل كل فكرة محددة وقابلة للتنفيذ.
- الترويج لا يزيد عن فكرتين.
أعد JSON فقط بدون أي نص إضافي.
```

### 9.2 Prompt للكتابة (`draft_writer`)

```
الدور: أنت كاتب محتوى محترف يعرف جمهور {persona}.

هوية العلامة (Brand Voice): {brand_voice}
الجمهور: {persona_description}
الركيزة: {pillar} — الموضوع: {topic}
الكلمة الرئيسية: {primary_keyword} — كلمات داعمة: {secondary_keywords}
قصد البحث: {search_intent}
مرحلة القمع: {funnel_stage}
الهدف من المحتوى: {content_goal}
الدعوة للإجراء (CTA): {cta} — يجب أن تظهر في النهاية بشكل طبيعي.
التنسيق المطلوب: {format} (عدد كلمات: {word_target})

أعد بالبنية التالية (JSON فقط):
{ "title", "sections": [ {"h2","h3","content"} ], "summary",
  "faqs": [{"question","answer"}], "cta_copy", "suggested_slug" }

المتطلبات:
- محتوى أصلي، غير مكرر، مفيد، واضح.
- استخدم الكلمة الرئيسية في العنوان والمقدمة وعنوان فرعي واحد.
- جملاً قصيرة ومقروءة (مستوى قراءة بسيط).
- لا معلومات مختلقة — إن لم تتأكد من رقم/حقيقة، ضع placeholders "[تحقق]".
```

### 9.3 Prompt لتحليل SEO (`seo_optimizer`)

```
السياق: نص المحتوى: {content} — الكلمة الرئيسية: {primary}، {secondary}.
المهمة — أنشئ:
{ "seo_title": ≤60 حرفاً, "meta_description": ≤160, "slug": slug URL,
  "h1": "...", "h2": ["..."], "h3": ["..."], "faq": ["سؤال"→"إجابة"],
  "internal_links": [أقترح 3 روابط داخلية], "external_links": [3],
  "image_alt": ["بدائل نصية"], "schema_type": "Article|FAQPage|HowTo",
  "seo_score": 0-100 }

عوامل الدرجة: وجود الكلمة الرئيسية في المواضع الصحيحة، تغطية الموضوع،
ارتباط دلالي، وجود أسئلة، طول، قابلية القراءة.
أعد JSON فقط.
```

> **SEO Score النهائي** يُعاد حسابه برمجياً (Code) بمزج نتيجة الـ AI مع فحوص برمجية (طول العنوان، وجود الكلمة في slug، إلخ) لضمان دقة قابلة للثقة.

### 9.4 Prompt لمراجعة الجودة (`quality_reviewer`)

```
راجع المحتوى التالي كمدقق جودة صارم:
{content}
قيّم: grammar, readability, seo, accuracy, relevance, duplication,
       brand_voice, cta, structure, factual_consistency
أعد:
{ "quality_scores": { grammar, readability, seo, accuracy, relevance,
                      originality, brand_compliance, structure, cta_clarity },
  "issues": [ {"severity":"critical|warning","text":"...","fix":"..."} ],
  "overall_score": 0-100 }
أي خلل حرج (Factual error أو عدم تطابق Brand Voice أو تكرار) = رسوب تلقائي.
أعد JSON فقط.
```

### 9.5 Prompt للمراجعة (`revision_agent`)

```
أعد كتابة النسخة التالية معالِجاً الملاحظات حصراً دون تغيير الجودة العامة:
الملاحظات: {issues}
النسخة الحالية: {content}
أعد المسودة الكاملة JSON بنفس مخطط draft_writer.
```

### 9.6 Prompt لإعادة الاستخدام (`repurposer`)

```
حول المقال التالي إلى محتوى منصات دون نسخ حرفي:
{content}
لكل منصة: {platforms}
أعد:
{ "platform": "...", "format": "post|thread|story|video_script|carousel|newsletter",
  "copy": "...", "hashtags": ["..."], "best_time": "HH:MM",
  "media_ref": "asset_id|null", "char_count": n }

قواعد المنصات:
- LinkedIn: نبرة احترافية، ≤3000 حرف.
- X: ≤280 حرفاً لكل منشور، واضح ومختصر.
- Instagram: عناوين + هاشتاقات 5-8.
- TikTok/Reels: سكربت فيديو ≤60 ثانية بفتحة لافتة.
- Newsletter: ملخص + 3 نقاط + CTA.
عدّل الزاوية لكل منصة؛ لا تكرر النص نفسه حرفياً.
أعد JSON فقط.
```

### 9.7 Prompt للتحليل الأدائي (`performance_analyst`)

```
البيانات: {aggregated_metrics}
أجب بالتحليل التالي (JSON فقط):
1) ما المحتوى الأفضل؟ ولماذا نجح؟
2) ما المحتوى الضعيف؟ ولماذا فشل؟
3) ما الموضوعات الأعلى أداءً؟
4) ما أفضل ركائز المحتوى؟
5) ما أفضل أنواع المحتوى؟
6) ما أفضل المنصات؟
7) ما أفضل أوقات النشر؟
8) ما المحتوى الذي يحتاج تحديثاً؟
9) ما المحتوى الجاهز لإعادة الاستخدام؟
10) ما الفرص الجديدة للمحتوى؟

{ "top_performers": [...], "weak_performers": [...], "insights": [...],
  "pillar_ranking": [...], "platform_ranking": [...], "best_times": [...],
  "needs_update": [...], "repurpose_candidates": [...], "opportunities": [...],
  "recommendations": ["إجراء عملي 1", "..."] }
استند فقط إلى الأرقام المقدمة؛ لا تخترع بيانات.
```

### 9.8 Prompt لتشخيص التراجع (`decay_auditor`)

```
المحتوى: {title} — الترتيب الحالي: {current_position} — سابقاً: {prev_position}
الترافيك: {traffic_trend} — التفاعل: {engagement_trend} — العمر: {content_age}
شخّص الأسباب المحتملة وأعد:
{ "probable_causes": [...], "update_recommendations": [
    {"field":"title|intro|keywords|structure|cta|meta|links","change":"..."} ],
  "refresh_priority": "low|medium|high" }
أعد JSON فقط.
```

---

## 10. نظام الموافقات

### 10.1 متى نحتاج موافقة بشرية؟

| الحالة | القرار |
|---|---|
| محتوى طويل/مقال (Pillar تعليمي) | موافقة إلزامية |
| منشور ترويجي/حملة | موافقة إلزامية |
| Reels/Scripts فيديو | موافقة مبسطة (نقطة واحدة) |
| منشورات Repurposed | موافقة Batch واحدة للمجموعة |
| تحليلات/تقارير | بدون موافقة (تُرسل كإشعار) |

### 10.2 مخطط الموافقة

```
[Production WF] ready ──► [Slack/Telegram] طلب موافقة (preview + scores + أزرار)
        │
        ▼
[Webhook] decision
   ├─ approve ──────────► status=approved ──► Publishing
   ├─ reject ───────────► status=rejected + notify + أرشيف للتعلم
   └─ revise + comment ─► status=revision ──► Revision agent (max 2) ──► re-request
```

### 10.3 محتوى رسالة الموافقة (Slack Block)

```
📝 مراجعة محتوى: {title}
📈 SEO: 78/100 | الجودة: 82/100
🏷️ Pillar: تعليمي | Persona: P01
📅 النشر: 2026-08-20 | المنصة: LinkedIn
🎯 CTA: تنزيل الدليل
[معاينة النص] [الوسائط]
✅ قبول  |  ❌ رفض  |  ✏️ طلب تعديل (+ تعليق)
```

### 10.4 تسجيل ومسؤولية

- كل قرار يُسجَّل في `content_approvals` (من قرر، متى، ماذا، ولماذا).
- Timeout 48h → تذكير؛ 5 أيام → تصعيد للمدير.
- `max_revisions = 2` لمنع Loop لا نهائي — عند تجاوزها يتحول للبشر كلياً.

---

## 11. نظام النشر والتوزيع

### 11.1 بوابة النشر (Pre-Publish Checks)

```
[Code] كل الفحوص قبل النشر:
1. approved == true
2. media موجودة (URL صالح)
3. publish_date صالح (<= today للساعة المحددة)
4. منصة جاهزة (Credential مفعّل)
5. لا تكرار (لم يُنشر نفس content_id+platform من قبل)
        │
        ▼
[IF] all pass ──► publish
[IF] any fail ──► defer + log (بدون نشر جزئي خاطئ)
```

### 11.2 استراتيجية النشر لكل منصة

| المنصة | الطريقة الافتراضية | بديل |
|---|---|---|
| LinkedIn | Buffer node | LinkedIn API (HTTP) |
| X | Buffer node | X API v2 (HTTP) |
| Facebook | Buffer node | Meta Graph API |
| Instagram | Buffer node (Reels/Image) | Meta Graph API |
| TikTok | Buffer node | TikTok Content Posting API |
| YouTube | Buffer node (Video) | YouTube Data API v3 |
| Newsletter | Mailchimp / Email API | n8n Email node |

### 11.3 إعادة المحاولة عند فشل النشر

```
publish ──► [IF] 429/5xx/network
   ├─ Retry ×3 with backoff (60s, 180s, 300s)
   ├─ النجاح ──► تحديث publication status + URL + notify
   └─ الفشل النهائي ──► Error WF + needs_manual_attention (لا نشر تلقائي مكرر)
```

### 11.4 التوزيع (Distribution)

بعد نجاح النشر:
```
Publish OK ──► Generate social posts (من repurposed) ──► Buffer schedule
            ──► Newsletter (عناصر جديدة أسبوعية)
            ──► إشعار الفريق (Slack)
            ──► تحديث تقويم (status=published)
```

---

## 12. نظام التحليلات

### 12.1 مصادر البيانات

| المصدر | المكون | المؤشرات |
|---|---|---|
| GA4 | Sessions, Users, Engagement, Conversions | Views, Bounce, Conversion |
| GSC | Clicks, Impressions, CTR, Position | Search performance |
| LinkedIn/X/IG/FB/TikTok/YT APIs | حسب التوفر | Likes, Shares, Comments, Saves |
| Buffer Insights | Engagement عبر المنصات | مقياس موحد |
| Newsletter API | Opens, Clicks | Subscribers, CTA Clicks |

### 12.2 مؤشرات مركبة (Calculated in Code)

```
CTR            = clicks / impressions
EngagementRate = (likes+comments+shares+saves) / impressions
ConversionRate = leads_or_sales / sessions (من GA4 events/UTM)
ROI            = (revenue_attributed − content_cost) / content_cost
```

### 12.3 تجميع (Aggregation) وتقارير

- **يومي:** جلب خام لكل pub_id وتخزين.
- **أسبوعي:** تجميع + حساب Trends (ق/Δ مقابل الأسبوع السابق).
- **شهري:** تقرير تنفيذي لـ AI + Dashboard (يمكن تصديره لـ Google Data Studio / Sheets).

### 12.4 تنبيهات (Alerts)

- ارتفاع مفاجئ في Engagement → إشعار فرصة "انشر المزيد من هذا النوع".
- انخفاض حاد في الترتيب/الترافيك → إشعار "تحقق من Decay".

---

## 13. حلقة التحسين المغلقة (Closed-Loop Optimization)

### 13.1 حلقة الأداء

```
Publishing ──► Analytics ──► Performance Analysis (AI) ──► Insights
        ▲                                              │
        │                                              ▼
        │                                    Optimization Loop
        │                                      │  (title/intro/keywords/
        │                                      │   links/CTA/meta/social copy)
        │                                      ▼
        └──────────── New Content Ideas ←──── Approval ──► Republish/Update
```

### 13.2 التحسين التلقائي للمحتوى الضعيف

```
Low performance ──► AI Diagnosis ──► Recommendations ──► Generate Optimized Copy
      ──► Human Approval ──► Republish (أو Update نفس الرابط)
```

### 13.3 اكتشاف تراجع المحتوى (Content Decay)

| الإشارة | العتبة |
|---|---|
| Traffic decline | -30% على 8 أسابيع |
| Position decline | -5 مراكز أو أكثر |
| Engagement decline | -40% |
| Content age | > 12 شهراً (فحص دوري) |

**إجراء:** AI Audit → Update Recommendations → Refresh → Approval → Republish.

### 13.4 ما يغذي مرحلة التخطيط (Feed-back to Strategy)

- الركائز الأعلى أداءً → رفع نسبة تخصيصها.
- الموضوعات المتكررة النجاح → Pillar جديد أو حملة.
- أفضل أوقات النشر → تحديث `config.best_times`.
- الفجوات المكتشفة → إدخال مباشر في `content_ideas`.

---

## 14. إدارة الأخطاء

### 14.1 استراتيجية شاملة

| فئة الخطأ | المعالجة |
|---|---|
| Rate limit (429) | Retry + Exponential Backoff + احترام Retry-After |
| Timeout | Retry ×2 مع زيادة المهلة |
| API Failure (5xx) | Retry ×3 ثم escalate |
| Invalid response (JSON خاطئ) | Repair via Code + retry 1× ثم escalate |
| Missing data | فحص قبل المعالجة؛ log + تخطي العنصر |
| Duplicate content | منع عبر Hash قبل الكتابة |
| Auth failure (401) | إيقاف فوري + notify Admin (لا retry أعمى) |
| Publishing failure | Retry ×3 ثم needs_manual_attention |
| AI failure | Fallback لمزود آخر (ضمن حد) ثم escalate |

### 14.2 مخطط Error Workflow المركزي

```
[Error Trigger]
        │
        ▼
[Code] Capture + enrich (workflow, exec, node, error, severity)
        │
        ▼
[Postgres/Sheets] log execution_logs (مع token usage و cost_est)
        │
        ▼
[IF] retryable?
   ├─ YES ──► [Execute Workflow] retry (backoff) ──► إذا نجح: mark resolved
   └─ NO  ──► [Slack] notify admin + mark needs_manual_attention
```

### 14.3 مبادئ حماية من الحلقات

- كل حلقة مراجعة: `max_retries` مخزّن في سياق العنصر.
- قبل أي Retry: زيادة عداد + تحقق `counter <= max`.
- لا `Wait` غير محدود؛ كل Wait له وقت أقصى.
- كل AI Node يملك Fallback مزود واحد على الأقل.

---

## 15. الأمان

### 15.1 إدارة الأسرار (Secrets Management)

| القاعدة | التنفيذ |
|---|---|
| لا مفاتيح داخل العُقد | كل Credentials عبر n8n Credential Store |
| Environment Variables | كل إعداد عبر `n8n variables` / `.env` |
| Rotation | صلاحيات قصيرة المدى حيثما أمكن (OAuth) |
| Least Privilege | حساب Google خدمة بصلاحيات "Read only" للجلب، "Write" لمنطقة واحدة للنشر |

### 15.2 حماية الـ Webhooks

- **Verification token / Header Signature** على كل Webhook (خاصة Approval Gate).
- `Verify API Key` أو JWT على أي Webhook عام.
- تحديد IP/بالأذونات حيثما أمكن.

### 15.3 التحقق من المدخلات والمخرجات (Input/Output Validation)

- **Input:** schema validation قبل أي استخدام لبيانات Webhook/Sheets.
- **Output:** تنظيف نص AI (إزالة HTML خبيث، نصوص طويلة جداً) قبل أي نشر.
- **SQL Injection:** لا بناء استعلامات بالConcatenation — استخدم Placeholders (parameters).

### 15.4 ضوابط إضافية

- **Access Control:** أدوار n8n — Editor vs User (لا يقوم User بتعديل العقد الحرجة).
- **Rate Limiting** على الـ Webhooks والـ API calls (عتبة داخلية).
- **Audit Log** لكل نشر/موافقة/تغيير إعداد.
- **مدقق محتوى AI** قبل النشر العام لأي ادعاء تسويقي حساس (FTC-style disclosure عند الترويج).

---

## 16. تحسين التكلفة

### 16.1 تكاليف النظام

| المكوّن | التكلفة النموذجية |
|---|---|
| AI Generation (نص) | حسب Tokens؛ نموذج قوي أغلى من الخفيف |
| AI Image Generation | لكل صورة (الأغلى نسبياً) |
| API Calls (منصات/تحليلات) | غالباً مجانية ضمن حدود |
| n8n Self-hosted | مجاني (شيفرة مفتوحة) + تكلفة الاستضافة |
| Buffer / Mailchimp | اشتراكات حسب الحجم |

### 16.2 استراتيجيات التخفيض

| الطريقة | التنفيذ |
|---|---|
| Model Selection | نماذج خفيفة للتصنيف/الاستخراج، قوية للتوليد فقط |
| Caching | Hash للـ prompt+payload → إعادة استخدام نتائج متطابقة |
| Batch Processing | توليد أفكار 10 دفعة واحدة بدل 10 استدعاءات |
| Conditional Execution | لا Image gen إلا بعد موافقة سابقة |
| Avoid duplicate AI calls | لا تكرار لتحليل نفس النص (استخدم cached seo score) |
| Reuse Generated Content | Repurposing يبني على نفس الـ draft (استدعاء واحد + تحويل) |
| Token Budget | `max_tokens` لكل Node + truncation آمن للسياق الطويل |
| Summary context | إرسال ملخصات بدل نصوص كاملة للـ analysts |

### 16.3 نموذج تقدير التكلفة (يُسجَّل في logs)

```
cost_est = prompt_tokens × price_in + completion_tokens × price_out
أمثلة أسعار تقريبية للتصميم (تتغير): gpt-4o-mini أرخص بكثير من gpt-4o.
```

---

## 17. إطار مؤشرات الأداء (KPI Framework)

### 17.1 KPIs لكل مرحلة

| المرحلة | KPIs |
|---|---|
| Strategy | نسبة أهداف متحققة، تغطية Pillars، توافق SEO |
| Idea Generation | معدل قبول الأفكار، معدل تحويل الفكرة لمنشور |
| Research | عدد كلمات/أسئلة مؤهلة لكل موضوع |
| Production | SEO Score، Quality Score، إنتاجية (قطعة/ساعة) |
| Approval | مدة دورة الموافقة، معدل الرفض أول مرة |
| Visual | نسبة الأصول المعتمدة، الامتثال للهوية |
| Publishing | نسبة نشر ناجح أول مرة، فشل النشر |
| Distribution | وصول، تفاعل، نسبة منشورات مجدولة في موعدها |
| Analytics | اكتمال البيانات، دقة التتبع (UTM) |
| Performance | TOP KPIs أدناه |
| Optimization | نسبة محتوى مُحسَّن، تحسن الترتيب بعد التحديث |
| Repurposing | عدد الصيغ لكل قطعة، إجمالي الوصول لكل أصل |

### 17.2 KPIs الرئيسية (Platform-agnostic)

| المقياس | التعريف | المصدر |
|---|---|---|
| Reach / Impressions | عدد الظهور | المنصات + Buffer |
| Engagement Rate | (تفاعلات / ظهور) ×100 | منصات |
| CTR | Clicks / Impressions | GA4/GSC/منصات |
| Conversion Rate | Leads أو Sales / Sessions | GA4 |
| Bounce Rate | جلسات صفحة واحدة | GA4 |
| Search Position (avg) | متوسط الموضع | GSC |
| Leads / Revenue | أهداف التحويل النهائية | CRM |
| Content Cost per Lead | (تكلفة الإنتاج+النشر) / Leads | النظام |

### 17.3 تتبع التحديثات والمراجعة

- تحديث مؤشرات كل أسبوع وشهر.
- مقارنة بنسخ سابقة (Trend).
- خطة عمل من توصيات AI → Tasks قابلة للتتبع (يمكن ربطها بجدول مهام).

---

## 18. استراتيجية الاختبار

### 18.1 مستويات الاختبار

| المستوى | المحتوى | الأداة |
|---|---|---|
| Unit (Code Nodes) | تحويلات JSON، الحسابات، الفحوص | n8n Code + Node fixtures |
| Integration | كل Sub-workflow مع بيانات وهمية (Mock) | n8n Manual Trigger |
| Contract | Schemas لكل مخرجات AI | JSON Schema validator |
| E2E | سيناريو كامل (فكرة → نشر وهمي) | Sandbox قنوات اختبار |
| Performance | عدد عناصر التقويم الثقيلة | Load على 100 عنصر |

### 18.2 حالات اختبار مطلوبة

1. عنصر تقويم ناقص (Missing data) → يعالج ولا يكسر الباقي.
2. AI يعيد JSON خاطئ → repair/retry/escalate.
3. موافقة رفض → Revision (تحقق ألا يتجاوز Max).
4. فشل API منصة → Retry ثم escalate.
5. حشو التكرار → يمنع نشر نسخة مكررة.
6. Decay على قطعة → ينتج تحديث ويصل للموافقة.
7. Rate limit → backoff صحيح.
8. مفتاح خاطئ (401) → إيقاف آمن + notify.
9. تكلفة عالية → verify caching يمنع استدعاءات مكررة.
10. تغيير Persona → ينتقل عبر كل المراحل بلا أخطاء.

### 18.3 بيئة Sandbox

- قنوات اختبار (LinkedIn/X خاصة) أو `Buffer` في وضع Preview.
- Sheets اختبارية منفصلة (`_TEST`).
- مقاييس وهمية للتحليلات.

---

## 19. خارطة التنفيذ

> كل مرحلة تنتهي باختبار مستقل قبل الانتقال للتالية.

### المرحلة 0 — الإعداد (Week 1)
- تثبيت n8n (Self-hosted Docker).
- إنشاء Google Service Account (Sheets) + Credentials.
- إنشاء Buffer account + ربط المنصات.
- إعداد Variables بيئية و GitHub repo للـ Workflows (JSON export).

### المرحلة 1 — الأساسيات (Week 1-2)
- W01 Content Planning + Google Sheets Calendar (Schema كامل).
- W12 Error/Logging الأساسي.
- **اختبار:** جلب عنصر تقويم وتسجيل Log.

### المرحلة 2 — الإنتاج (Week 2-3)
- AI Orchestrator (Multi-provider) + Schema Validation.
- W04 Content Production (draft + seo + quality) + Revision Loop.
- **اختبار:** توليد قطعة كاملة مع Mock.

### المرحلة 3 — المرئيات والتحويل (Week 3-4)
- W05 Visual Production + W06 Repurposing.
- **اختبار:** مقال → 6 صيغ + أصول.

### المرحلة 4 — الموافقات والنشر (Week 4-5)
- W07 Approval Gate (Slack/Telegram) + Webhook.
- W08 Publishing (Buffer + APIs) + Pre-Publish Checks.
- **اختبار:** موافقة → نشر في Sandbox.

### المرحلة 5 — التحليلات (Week 5-6)
- W09 Analytics (GA4 + GSC) + W10 Performance Analysis (AI).
- **اختبار:** بيانات وهمية → تقرير AI.

### المرحلة 6 — الحلقة المغلقة (Week 6-7)
- W11 Optimization/Decay + ربط الرؤى بـ W02 (أفكار جديدة).
- **اختبار:** محتوى ضعيف → تحديث → إعادة نشر.

### المرحلة 7 — الصلابة والأمان (Week 7-8)
- تكامل Error Workflow الكامل + Retry Policies.
- Security hardening (webhook signatures, least privilege).
- Cost dashboard + ضبط النماذج.

### المرحلة 8 — الإنتاج الفعلي (Week 8+)
- تشغيل مع بيانات حقيقية (وضع Monitoring).
- جدولة Review أسبوعي + ضبط الـ KPIs.
- توثيق التشغيل (Runbook) وتهيئة النسخ الاحتياطي.

---

## ملحق أ — قائمة عُقد الـ Workflows (Quick Reference)

| Workflow | العُقد الرئيسية |
|---|---|
| W01 | Schedule, Google Sheets, Code, IF, Execute Workflow |
| W02 | Schedule, Sheets, Code, AI Orchestrator, IF |
| W03 | HTTP Request, Sheets, Code, AI, IF |
| W04 | Execute Workflow, Code, AI ×4, IF ×2, Sheets |
| W05 | Code, AI (image), HTTP, Cloud Storage, Sheets |
| W06 | Code, AI (repurpose), Code validate, Sheets, IF |
| W07 | Slack, Telegram, Webhook, IF, Sheets |
| W08 | Schedule, Sheets, Buffer, HTTP (APIs), IF, Slack |
| W09 | Schedule, GA, GSC, HTTP, Code, Postgres |
| W10 | Schedule, Postgres, Code, AI, Slack, Sheets |
| W11 | Schedule, Postgres, IF, AI ×2, Approval, Publishing |
| W12 | Error Trigger, Code, Sheets, IF, Execute Workflow, Slack |

## ملحق ب — الحد الأدنى للتشغيل (Checklist)

- [ ] n8n instance + Variables (env)
- [ ] Google Sheets: Config, Calendar, Ideas, Content, Platforms, Assets, Logs
- [ ] Credentials: Google (Sheets/GA4/GSC), OpenAI/Gemini/Claude, Buffer, Slack/Telegram
- [ ] Buffer channels متصلة بالمنصات الست
- [ ] Webhook Signature لـ Approval Gate
- [ ] Sandbox قنوات + تقويم `_TEST`
- [ ] Postgres (أو Sheets) لـ Execution Logs
- [ ] حدود Retry لكل Loop (Max=3, Revisions=2)

---

*انتهت الوثيقة — جاهزة للتحويل إلى Workflows فعلية في n8n عبر تصدير JSON من كل تصميم.*