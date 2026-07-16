# سجل التغييرات والإصدارات (CHANGELOG.md)

**المشروع:** PC COMPTA Next (`Ahmed47-t/compta`)  
جميع التغييرات والإصدارات الرسمية والداخلية للمشروع يتم توثيقها في هذا الملف وفق معايير **[Keep a Changelog](https://keepachangelog.com/en/1.1.0/)**. يعتمد المشروع في الإصدارات الرسمية ترقيماً يدمج بين **[Semantic Versioning](https://semver.org/)** والتقويم السنوي.

---

## [Unreleased] - 2026-07-16

### Added (تمت الإضافة)
- **قواعد العمل والحوكمة لوكلاء الذكاء الاصطناعي (`AGENTS.md`):** تعريف القيود الصارمة المانعة لاستخدام الفواصل العائمة في المبالغ المالية (`float/double`)، وفرض التوازن المزدوج للسندات (`Double-Entry Guard`)، ومنع الحذف المادي للقيود المعتمدة، واشتراط التحقق البشري لمقترحات الـ OCR.
- **تأسيس الذاكرة الدائمة للمشروع تحت `docs/`:**
  - `PRODUCT_VISION.md`: وثيقة رؤية المنتج وركائز `Offline-first, Keyboard-fast, Mathematically-Secure`.
  - `MASTER_PLAN.md`: المخطط المعماري الشامل، ونموذج البيانات الـ 15 جدول، وتصميم المحركات الـ 9، وخارطة الطريق الـ 14 مرحلة.
  - `SCOPE.md`: تحديد النطاق الوظيفي الشامل للإصدار المكتبي V1.0 والفصل الحاسم عما هو خارج النطاق.
  - `DOMAIN_GLOSSARY.md`: قاموس المصطلحات المحاسبية والتقنية ثلاثي اللغة مع ربطها بمصطلحات PC COMPTA الأصلي.
  - `ACCOUNTING_INVARIANTS.md`: القوانين الرياضية والمحاسبية غير القابلة للخرق.
  - `ACCEPTANCE_CRITERIA.md`: معايير القبول الوظيفية وبوابات تعريف الجاهزية (`DoR`) والانتهاء (`DoD`).
  - `TRACEABILITY_MATRIX.md`: مصفوفة التتبع التي تربط كل متطلب بمعيار القبول، واختباره، والمرحلة المستهدفة وحالته.
  - `RISK_REGISTER.md`: سجل المخاطر المعمارية والمحاسبية والأمنية وتدابير تخفيفها.
- **إدارة القرارات المعمارية (`docs/DECISIONS/`):**
  - `0000-adr-template.md`: قالب توثيق القرارات المعمارية.
  - `ADR-0001-architecture.md`: توثيق مبررات ونتائج اختيار معمارية `Tauri 2.x + Python FastAPI Modular Monolith + PostgreSQL 16 Partitioned + Local OCR Worker`.
- **مجلدات مواصفات وتقارير المراحل (`docs/PHASES/`):**
  - إنشاء مواصفات المراحل الدقيقة من `PHASE-00.md` إلى `PHASE-14.md`.
  - إنشاء قالب تقارير المراحل `phase-report-template.md` وتقرير إتمام المرحلة الأولى `PHASE-00-REPORT.md`.
- **ملفات التسليم اللحظية (`docs/HANDOFF/`):**
  - `CURRENT_STATE.md`: حالة المشروع اللحظية وانتقاله من المرحلة 00 إلى المرحلة 01.
  - `LAST_PHASE_REPORT.md`: التقرير التدقيقي لإتمام وإيداع مخرجات المرحلة 00.
  - `OPEN_QUESTIONS.md`: استخراج الأسئلة المفتوحة وتوثيقها في 4 تصنيفات (الأجهزة، ملفات DLG، التقارير الجبائية، وسياسات Cloud OCR).
  - `NEXT_PHASE_INPUT.md`: مواصفات ومتطلبات إنجاز تجارب وقياسات الأداء (`PoCs & Benchmarks`) في المرحلة 01.
- **أدلة الأمان والاختبار والتشغيل (`SECURITY, TESTING, RUNBOOKS`):**
  - `THREAT_MODEL.md` (STRIDE Analysis) و `SECURITY_BASELINE.md` (Argon2id, RLS, WORM Audit, AES-256).
  - `TEST_STRATEGY.md` (Test Pyramid) و `GOLDEN_DATASET.md` (Golden fixtures for SCF, balances, DLG migration, and OCR).
  - أدلة التشغيل البرمجية والتعافي والإصدارات (`DEVELOPMENT.md, BACKUP_RESTORE.md, RELEASE.md`).
- **قوالب حوكمة GitHub (`.github/`):** قوالب الإبلاغ عن المهام والأخطاء المحاسبية وقالب مراجعة طلبات السحب (`PULL_REQUEST_TEMPLATE.md`).
