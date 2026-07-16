# مواصفات المرحلة 02: تأسيس Monorepo و CI/CD والهياكل (PHASE-02.md)

**المرحلة:** Phase 02  
**الخبير المطلوب:** Staff Platform Engineer + DevSecOps  
**الهدف الأساسي:** تحويل القرارات المعمارية المعتمدة في المرحلة 01 إلى بنية مستودع إنتاجية منظمة (`Monorepo Skeleton`) وخطوط تكامل ونشر آمنة (`CI/CD Pipeline`).

---

## 1. نطاق العمل المطلوب (Scope of Work)

1. **بناء هيكلة Monorepo:** إنشاء الهيكل والمجلدات الرئيسية وفق الحدود المعتمدة:
   - `apps/desktop` (Rust Tauri Desktop Shell)
   - `apps/ui` (React/TypeScript UI App)
   - `services/accounting_api` (Python FastAPI Backend Service)
   - `services/ocr_worker` (Python Local OCR Worker Service)
   - `packages/domain` (Pure Accounting Domain Logic & Invariants)
   - `packages/contracts` (Pydantic & TypeScript Shared Interfaces)
   - `db, tests, docs, tools, installers`
2. **إدارة الحزم والإصدارات (`Dependency Management`):** تثبيت إصدارات المكتبات وإعداد ملفات الأقفال (`Lockfiles: Cargo.lock, package-lock.json, poetry.lock/requirements.txt`) وضمان اتجاه الاعتماديات (`Dependency Direction`: Domain لا يعتمد على API أو DB).
3. **أدوات النظافة والفحص (`Lint / Type-Check / Formatting`):** إعداد وتثبيت أداة `ruff/mypy` لـ Python، وأداة `eslint/prettier/tsc` لـ TypeScript، وأداة `clippy/rustfmt` لـ Rust.
4. **خطوط التكامل المستمر (`GitHub Actions CI`):** بناء Workflow آلي يشغل البناء النظيف والفحوصات على بيئة Windows وبيئة Linux.
5. **الفحص الأمني (`Security & Supply Chain Scans`):** تفعيل فحص الأسرار (`Secret Scanning`)، وفحص ثغرات الاعتماديات (`SAST / Dependency Scanning`)، وتوليد قائمة المكونات البرمجية (`SBOM`).
6. **إعدادات التطوير السريع (`Bootstrap & Logging`):** إضافة أوامر البناء السريع (`One-command bootstrap`) ونظام تسجيل أحداث مهيكل وآمن (`Structured Logging`) خالٍ من البيانات الحساسة (`PII-Free`).

---

## 2. بوابة القبول (Acceptance Gate)

- نجاح البناء النظيف (`Clean Build`) والاختبارات والفحوصات في خط الـ CI على كل من Windows و Linux من checkout جديد.
- تأكيد استقلالية حزمة `packages/domain` تماماً عن أطر العمل الخارجية (`PostgreSQL, FastAPI, Tauri, OCR`).
- خلو المستودع من أي أسرار أو ملفات ثنائية ضخمة غير مبررة، واشتمال `docs/RUNBOOKS/DEVELOPMENT.md` على خطوات إعداد واضحة.
