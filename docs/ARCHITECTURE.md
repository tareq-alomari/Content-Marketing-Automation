# المعمارية التقنية — ARCHITECTURE

توثيق معماري تفصيلي لنظام **Marketing Content Engine** المبني على n8n.

---

## 1. المخطط العام (High-Level Diagram)

```
                    ┌───────────────────┐
                    │  Content Strategy │
                    └─────────┬─────────┘
                              ↓
                    ┌───────────────────┐
                    │ Content Calendar  │  (Google Sheets — Source of Truth)
                    └─────────┬─────────┘
                              ↓
                    ┌───────────────────┐
                    │ Research + SEO    │  W03
                    └─────────┬─────────┘
                              ↓
                    ┌───────────────────┐
                    │   AI Production   │  W04 (Draft → SEO → QC)
                    └─────────┬─────────┘
                              ↓
                    ┌───────────────────┐
                    │   Visual Content  │  W05
                    └─────────┬─────────┘
                              ↓
                    ┌───────────────────┐
                    │  Repurposing      │  W06
                    └─────────┬─────────┘
                              ↓
                    ┌───────────────────┐
                    │ Human Approval    │  W07 (Slack/Telegram Webhook)
                    └─────────┬─────────┘
                              ↓
                    ┌───────────────────┐
                    │    Publishing     │  W08 (Buffer + APIs)
                    └─────────┬─────────┘
                              ↓
                    ┌───────────────────┐
                    │   Distribution    │  W08 (Social + Newsletter)
                    └─────────┬─────────┘
                              ↓
                    ┌───────────────────┐
                    │    Analytics      │  W09 (GA4 + GSC)
                    └─────────┬─────────┘
                              ↓
                    ┌───────────────────┐
                    │   AI Analysis     │  W10 (Performance + Insights)
                    └─────────┬─────────┘
                              ↓
                    ┌───────────────────┐
                    │   Optimization    │  W11 (Decay + Refresh)
                    └─────────┬─────────┘
                              │
                              └──────────→ Content Strategy (Closed-Loop)
```

---

## 2. مكونات النظام (Components)

### 2.1 طبقة التنسيق (Orchestration Layer)

- **W00-master-runner** — نقطة تشغيل واحدة: يسلّس تنفيذ الـ Workflows بالترتيب.
- **Execute Workflow Nodes** — كل Workflow فرعي يُستدعى عبر Input/Output JSON محدد.

### 2.2 طبقة الـ AI (Multi-Provider)

- **sub_ai-orchestrator** — نقطة عبور واحدة لكل استدعاءات AI.
- **Model Router** (Switch): يوجّه `task_type` للنموذج القوي أو الاقتصادي.
- **Fallback**: دعم التبديل لمزود بديل عند فشل المزود الأساسي.
- **Parse & Validate**: يفحص JSON ويعيد المحاولة مرة واحدة عند حدوث خطأ تحليل.

### 2.3 طبقة التكامل (Integration Layer)

| النوع | الخدمة | Workflows |
|---|---|---|
| Source of Truth | Google Sheets | W01, W02, W06, W07, W08, W09, W10, W11, W12 |
| النشر | Buffer + Platform APIs | W08 |
| التحليلات | GA4 + Google Search Console | W09 |
| الإشعارات | Slack / Telegram / Email | W07, W08, W10, W12 |
| التخزين | S3 / Google Drive | W05 |

### 2.4 طبقة البيانات (Data Layer)

- **الوضع التجريبي:** Google Sheets (جداول منفصلة لكل كيان).
- **الوضع الإنتاجي:** PostgreSQL (نفس المخطط — تبديل العقد فقط).
- الجداول: `content`, `content_ideas`, `content_calendar`, `content_versions`, `content_keywords`, `content_platforms`, `content_publications`, `content_analytics`, `content_performance`, `content_reviews`, `content_approvals`, `content_assets`, `campaigns`, `audiences`, `personas`, `content_pillars`, `execution_logs`.

---

## 3. خريطة الـ Workflows والتبعيات

| # | Workflow | Input | Output | يستهلك | يستدعي |
|---|---|---|---|---|---|
| W00 | master-runner | — | Run summary | كل ما سبق | W01→W02→W08→W09→W10→W11 |
| W01 | plan-content-engine | Calendar | Due items | Sheets | W04, W12 |
| W02 | idea-generator | Config | Ideas (scored) | Sheets | sub_ai |
| W03 | research-pipeline | Topic | Research JSON | Sheets, Google Suggest | sub_ai |
| W04 | content-production | Content item | Draft+SEO+QC | Sheets | sub_ai (×4), W07 |
| W05 | visual-production | Content item | Assets | Image API, S3 | sub_ai |
| W06 | repurposing-engine | Article | Platform posts | Sheets | sub_ai |
| W07 | approval-gate | Webhook decision | Status update | Sheets | — |
| W08 | publishing-distribution | Approved items | Publications | Buffer, Sheets, Slack | — |
| W09 | analytics-collection | — | Metrics | GA4, GSC, Sheets | — |
| W10 | performance-analysis | Analytics | Insights | Sheets | sub_ai |
| W11 | optimization-decay | Analytics | Refresh queue | Sheets | sub_ai |
| W12 | error-notification | Any error | Alert + log | Sheets, Slack | — |

---

## 4. تدفق البيانات داخل W04 (الإنتاج — الأهم)

```
[Execute Workflow Trigger]
        ↓
[Code] Build Generation Context (brand_voice + persona + keywords + intent + CTA)
        ↓
[sub_ai] task=draft_writer ──► [Code] Validate Draft (Schema)
        ↓ fail ─────────────────────────────┐
        ↓                                   │
[Code] Build SEO Prompt ──► [sub_ai] seo_optimizer ──► [Code] Compute SEO Score
        ↓                                                        │ fail
        ↓                                                        └─────────┐
[Code] Build QC Prompt ──► [sub_ai] quality_reviewer ──► [Code] Compute QC Score
        ↓ fail                                                   │
        └────────► [Code] Build Revision (max 2) ──► [sub_ai] revise ──► re-check
        ↓ pass
[Code] Mark Ready for Approval ──► W07
```

**قواعد الحماية:**
- كل درجة تُحسب في Code (قابلة للاختبار) ولا تُسلَّم للـ AI.
- المراجعات محدودة بـ `MAX_REVISIONS` (افتراضياً 2).
- عند تجاوز الحد → `escalate` (لا تكرار لا نهائي).

---

## 5. تدفق البيانات داخل W07 (الموافقات)

```
[Webhook] POST /approval
        ↓
[Code] Verify Signature (APPROVAL_SIGNING_KEY)
        ↓
[Switch] decision
   ├─ approve ──► [Code] status=approved ──► (W08 يلتقط الصفوف المعتمدة)
   ├─ reject   ──► [Code] status=rejected
   └─ revise   ──► [Code] status=revision + comment ──► W04
        ↓
[Sheets] Log Approval (من، متى، قرار، تعليق)
```

---

## 6. حلقة التحسين المغلقة (Closed-Loop)

```
[W09 Analytics] ──► [W10 AI Analysis] ──► Insights (Sheets)
       ▲                                       │
       │                                       ▼
[W08 Publish] ◄── [W07 Approval] ◄── [W11 Decay Refresh]
                                          │
                                          └──► New topics ──► [W02 Ideas] ──► [W01 Calendar]
```

**آلية الإدخال الجديد:** توصيات W10 تُكتب في `Content Ideas` بحالة `review`، ثم تُعرض في الجولة التالية من W02 → W01.

---

## 7. الأمان والمعمارية

- **Credentials** فقط، بدون مفاتيح في JSON.
- **Webhook Signature** على W07.
- **Variables** لكل الإعدادات السرية.
- **Error Workflow مركزي (W12)** لجميع الـ Workflows.
- **Least Privilege**: حساب Google للجلب Read-only، ومنطقة نشر واحدة للنشر.

---

## 8. القرارات المعمارية (ADR Summary)

| القرار | السبب |
|---|---|
| Google Sheets كمصدر حقيقة | سهولة العرض والمشاركة والتدقيق الأكاديمي |
| Buffer كطبقة نشر موحدة | Node رسمي في n8n يغطي 6 منصات |
| sub_ai-orchestrator | تجريد Multi-Provider + Fallback + تحقق مركزي |
| Code بدل AI للحسابات | قطعية وقابلية الاختبار وتقليل التكلفة |
| Workflows منفصلة لا Workflow ضخم | Modularity، قابلية الصيانة والاختبار المستقل |
| MaxRetries محدودة | منع Infinite Loops مع الحفاظ على المرونة |

---

## 9. ملاحظات التوسع (Scalability)

- **Backend معالجة:** `queue` mode مع Redis للعمل الجماعي الثقيل.
- **قاعدة بيانات:** PostgreSQL في الإنتاج مع فهارس على `content_id` و `publish_date`.
- **النسخ الاحتياطي:** تصدير الـ Workflows JSON دورياً + نسخ ورقة Sheets.
- **Monorepo:** يمكن تقسيم النظام إلى تقويم منفصل + إنتاج منفصل + تحليلات منفصلة في repos مستقلة.