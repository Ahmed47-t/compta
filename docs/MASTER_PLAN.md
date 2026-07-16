# المخطط الرئيسي والتصميم المعماري الشامل (MASTER_PLAN.md)

**المشروع:** PC COMPTA Next (`Ahmed47-t/compta`)  
**الإصدار:** 1.0 — يوليو 2026  
**المرجع الأساسي:** مُستخلص ومبني على التحليل الفني الشامل في ملفات `MASTER_PLAN_PCCOMPTA_COMPLETE.md`، `improved_plan_pccompta_v2.md`، و `ocr_accuracy_offline_vs_hybrid.md`.

---

## 1. الملخص التنفيذي

يحدد هذا المخطط الرئيسي البنية المعمارية وخطة البناء التفصيلية لنظام **PC COMPTA Next**، وهو نظام محاسبي مالي متطور يعمل على أنظمة سطح المكتب (Desktop Monolith Modular Architecture)، مصمم ليحل محل برنامج PC COMPTA التقليدي في السوق الجزائري.

يجمع النظام بين ثلاث مزايا محورية:
1. **السرعة الفائقة (Delphi-like Speed):** استجابة لحظية وبحث فوري في جداول ضخمة دون تأخير بفضل استخدام Tauri (Rust) و SQLAlchemy Core مع فهرسة ثلاثية الحروف (`pg_trgm`).
2. **الأمان البنكي المدمج (Security V3 Engine):** تشفير كامل، تقييد صلاحيات بـ `RLS`، سجلات تدقيق مقاومة للمحو (`WORM Append-Only`)، وسلسلة تجزئة (`Hash Chain`) لضمان عدم التلاعب بالقيود.
3. **الأتمتة بالذكاء الاصطناعي المحلي (Offline OCR Factory):** استخراج الفواتير ومطابقة الحقول وتوليد المقترحات المحاسبية محلياً بنسبة 100% دون إرسال البيانات السحابية.

---

## 2. ماذا نحتفظ به من PC COMPTA الأصلي وماذا نستبدل؟

| الميزة / الفلسفة في PC COMPTA الأصلي | القرار | التحسين والمحاكاة في PC COMPTA Next |
| :--- | :--- | :--- |
| **هيكلة الملفات (Dossier + Exercice)** | احتفاظ وتطوير | فصل كل شركة كـ `Company` وكل سنة كـ `Exercice` مستقل مع إمكانية فتح سنوات متعددة بالتوازي (`Multi-exercice en parallèle`). |
| **نظام Folio (شهر = Folio ذكي)** | احتفاظ وتطوير | ترقيم إلزامي ومحكم: `Folio 01 = جانفي`، `LIGNE` تسلسلي تلقائي، وتحقق فوري من تطابق شهر القيد مع رقم الفوليو. |
| **طرق التصحيح (Modif / CTP / Négative)** | احتفاظ وتطوير | دعم الطرق الثلاث + طريقة رابعة مؤمنة (`Correction avec motif`) تطلب سبباً إجبارياً وتسجل في `Audit Logs`. |
| **نقل الأرصدة والأسطر (Report à-nouveaux)** | احتفاظ وتطوير | نقل الأسطر غير المطابقة (`Non lettrées`) وتاريخها الأصلي بدقة لمتابعة أعمار الديون، وليس مجرد نقل رصيد إجمالي صامت. |
| **المطابقة (Lettrage بـ AA, AB)** | احتفاظ وتطوير | مطابقة يدوية وآلية فورية بالاعتماد على المبالغ ورقم الفاتورة، مع تقارير للأرصدة المفتوحة. |
| **المحاسبة التحليلية والميزانياتية** | احتفاظ وتطوير | دعم 3 محاور تحليلية (`Natures, Unités, Budgets`) ومقارنة الفعلي بالمخطط. |
| **قاموس الصيغ (Dictionnaire des Formules)** | احتفاظ وتطوير | بناء محرك `AST Compiler` آمن يفهم صيغ الجباية مثل `-SOLDE(10)` و `/CRD` ويحولها لـ SQL مُفهرس وسريع. |
| **قاعدة بيانات BDE/dBASE (.DLG / .PCC)** | **إلغاء واستبدال** | **PostgreSQL 16+** مع تقسيم الجداول زمنياً (`Partitioning by Exercice`) وتشفير البيانات في قاعدة البيانات (`AES-256`). |
| **واجهة المستخدم (Win98 / Win32)** | **إلغاء واستبدال** | **Tauri 2.x + React + TypeScript** بتصميم حديث يحاكي سرعة الكيبورد التامة دون استهلاك مفرط للذاكرة (`< 10% RAM compared to Electron`). |
| **مفتاح الحماية المادي (Dongle USB)** | **إلغاء واستبدال** | **مفاتيح ترخيص مشفرة وموقعة بـ RSA (`Signed RSA Licenses`)** مرتبطة بالبصمة العتادية (`Hardware Fingerprint`) وحالة المؤسسة. |
| **الإدخال اليدوي الكامل لكل فاتورة** | **إلغاء واستبدال** | **مصنع استخراج ذكي محلي (Offline OCR Service)** يقترح القيد فوراً مع الحفاظ على اعتماد المحاسب البشري كشرط أساسي للتأكيد. |

---

## 3. النطاق الوظيفي الشامل (Scope Overview)

ينقسم المشروع إلى 6 حزم وظيفية مترابطة:

### A. إدارة الهيكل المحاسبي والتأسيس (Configuration & Templates)
- إنشاء وإدارة الشركات (`Companies`) والسنوات المالية (`Exercices`) وحالات الفتح والإغلاق (`Ouvert, Clôture Provisoire, Clôture Définitive`).
- دليل الحسابات (`Plan Comptable - PCC_COM`): كود حتى 14 حرف/رقم، تصنيفات الحساب (`Lettrable, Soumis à Auxiliaire, Analytique`).
- اليوميات المحاسبية (`Journaux - PCC_JRN`): كود 8 أحرف وأنوع (`ACH, VTE, BQ, CAISSE, OD, OUV`).
- الحسابات المساعدة (`Auxiliaires - PCC_AUX`): العملاء (`411`)، الموردون (`401`)، وبيانات التعريف الجبائية (`NIF, NIS, RC, AI`).
- قوالب جاهزة للنظام المحاسبي الجزائري (`SCF Templates`): توليد تلقائي لـ 500 حساب و 10 يوميات للشركات التجارية أو الصناعية عند الإنشاء.

### B. محرك القيود وإدخال اليوميات (Core Accounting & Saisie)
- الإدخال بنظام الفوليو (`Saisie par Folio`): التحقق الرباعي الآني (تطابق الفوليو مع شهر القيد، وجود الحساب، إلزامية الحساب المساعد، توازن الفوليو قبل الإغلاق).
- ضمان التوازن الدائم (`Double-Entry Guard`): استحالة اعتماد قيد غير متوازن رياضياً في قاعدة البيانات.
- تصحيح اليوميات: الحظر المطلق لحذف القيود المعتمدة (`Posted Entries`)؛ واستخدام القيود العكسية (`Contre-passation`) أو القيود السالبة (`Saisie Négative`).
- اليومية الافتتاحية (`Journal d'Ouverture`): فصل حركات الافتتاح عن حركة السنة الجارية ونقل الأرصدة غير المطابقة.

### C. المطابقة والتحليل (Reconciliation & Analytical)
- المطابقة اليدوية والآلية (`Lettrage AA, AB...`) مع سماحية تقريب مضبوطة (`Tolérance 0.01 DA`).
- توزيع القيود على المحاور التحليلية ومتابعة الميزانيات التقديرية والفروقات (`Écarts`).

### D. الأصول والاهتلاكات (Immobilisation & Amortissements)
- بطاقات الأصول (`PCC_INV, IAF`): فصل تاريخ الاقتناء (`Date acquisition`) عن تاريخ بدء الخدمة (`Date mise en service`).
- معالجة فارق إعادة التقييم (`Écart de réévaluation` - المادة 44) وحساب الاهتلاك الخطي (`Linéaire`) وتوليد جداول الاهتلاك والقيود التلقائية.

### E. التقارير والجباية الجزائرية (Reporting & Tax Engine)
- استخراج اليوميات العام، دفتر الأستاذ (`Grand Livre`)، ميزان المراجعة (`Balance 6/8 Colonnes`).
- محرك الصيغ الجبائية (`Dictionnaire des Formules Compiler`): ترجمة الرموز مثل `-SOLDE(10)` إلى استعلامات SQL سريعة.
- إعداد وتصدير التصريحات الضريبية: `G50`، جدول الرسم على القيمة المضافة (`TVA Récupérable/Collectée`)، والجداول الرسمية للحزمة الجبائية (`Liasse Fiscale SCF`).

### F. الهجرة والاستيراد والتكامل (Migration & Integration)
- قراءة البيانات التاريخية مباشرة من ملفات `DLG` (DBF Format) وملفات `.PCC` (SDF Fixed-length format).
- استيراد وتصدير متبادل ومحمي لملفات `Excel/CSV`.
- توفير APIs داخلية آمنة للربط في المستقبل مع برامج الأجور (`PC PAIE`) والمخزون (`PC STOCK`).

---

## 4. المعمارية التقنية ومكدس التطوير (Tech Stack & Architecture)

```text
+-----------------------------------------------------------------------------------+
|                         Tauri Desktop App (Windows 10/11)                         |
|                                                                                   |
|  +-----------------------------------------------------------------------------+  |
|  |                 Frontend UI (React + TypeScript + Tailwind)                 |  |
|  |    [Saisie Grid Keyboard-First] | [Dossiers/Exercices] | [OCR Validation UI]  |  |
|  +-----------------------------------------------------------------------------+  |
|                                        | IPC / Local API                          |
+----------------------------------------|------------------------------------------+
                                         v
+-----------------------------------------------------------------------------------+
|                        Python Backend API (FastAPI + ASGI)                        |
|                                                                                   |
|  +-----------------------------------------------------------------------------+  |
|  |                           Modular Monolith Engines                          |  |
|  |  [Core Engine] | [Saisie] | [Lettrage] | [Analytique] | [Immo] | [Reporting]|  |
|  |  [Security V3 Engine: RLS, Argon2id, Hash Chain, Audit WORM, RSA License]  |  |
|  +-----------------------------------------------------------------------------+  |
|            | SQLAlchemy Core               | Local Pipe (IPC/HTTP Local)          |
+------------|-------------------------------|--------------------------------------+
             v                               v
+-----------------------------+     +-----------------------------------------------+
|  PostgreSQL 16+ Database    |     |             Local OCR Worker Service          |
|                             |     |  Python / ONNX Runtime / PaddleOCR / TrOCR    |
| - Partitioned by Exercice   |     |  LayoutLMv3 (Extraction + Calibration)        |
| - NUMERIC(19,2) Exact       |     |  100% Offline by Default (Optional Cloud)     |
| - pgcrypto, pg_trgm, pgaudit|     +-----------------------------------------------+
+-----------------------------+
```

### المبررات التقنية للخيارات المعتمدة:
1. **لماذا Tauri بدلاً من Electron أو Web SaaS؟**  
   Tauri ينتج تطبيقاً بحجم لا يتجاوز `5–10 MB` ويستهلك ذاكرة RAM أقل بنسبة 90% مقارنة بـ Electron، مما يحاكي خفة وسرعة تطبيقات Delphi القديمة ويوفر أداءً استثنائياً على الحواسيب المكتبية الضعيفة والمتوسطة.
2. **لماذا FastAPI + SQLAlchemy Core؟**  
   الفصل الواضح بين واجهة سطح المكتب ومحرك الأعمال عبر واجهة برمجة تطبيقات محلية (`Local API`) يضمن استقرار النواة المحاسبية وإمكانية اختبارها آلياً بشكل مستقل 100%. كما أن استخدام `SQLAlchemy Core` (بدلاً من ORM الثقيل) يمنحنا سرعة فائقة في معالجة الاستعلامات المعقدة وموازين المراجعة.
3. **لماذا PostgreSQL Partitioned بدلاً من SQLite؟**  
   البيانات المحاسبية لمكاتب الخبراء الجزائريين تمتد لسنوات وتضم مئات الآلاف من القيود لكل شركة. نظام `Partitioning` بناءً على السنة المالية (`exercice_id`) يمنحنا استقلالية تامة وسرعة هائلة في البحث، بالإضافة إلى القوة الأمنية الكبرى المتمثلة في `Row-Level Security (RLS)` و `pg_trgm` و `pgaudit`.

---

## 5. نموذج البيانات الكامل (15 جدول رئيسي)

### الجداول المحاسبية الأساسية (المستوحاة والمطورة من PC COMPTA):
1. `companies` (الشركات / الملفات الأساسية مع `NIF, NIS, RC, AI, Address`).
2. `exercices` (السنوات المالية المرتبطة بالشركة مع `year, status, hash_root, rsa_signature`).
3. `pcc_com` (دليل الحسابات الأساسي مع `code_com PK 14, libelle 30, lettrable, soumis_aux`).
4. `pcc_jrn` (يوميات المحاسبة مع `code_jrn PK 8, name, nature`).
5. `pcc_aux` (الحسابات المساعدة للعملاء والموردين مع `code_aux PK 14, code_com FK, NIF, RC`).
6. `pcc_glv` (جدول القيود المركزي المقسم زمنياً بـ `Partitioning`، يضم: `code_com, code_aux, code_jrn, folio_id, piece, ligne, sdate, reference, libelle, mdebit, montant, echeance, pointage, ctp_com, ctp_aux, hash_folio`).
7. `pcc_inv` (الأصول والاستثمارات مع بيانات الاقتناء والتقييم).
8. `pcc_iaf` (جدول أقساط وحركات الاهتلاك المرتبطة بالأصول).
9. `pcc_bdg, pcc_nat, pcc_unt` (جداول المحاسبة التحليلية والميزانياتية).

### جداول الأمان والذكاء الاصطناعي والإدارة (الجديدة):
10. `users` (المستخدمون مع `username, password_hash Argon2id, role, mfa_secret`).
11. `audit_logs` (سجل التدقيق المقاوم للحذف `WORM` مع `user_id, company_id, action_type, table_name, old_value, new_value, reason, timestamp, prev_hash, curr_hash`).
12. `folio_hashes` (سلسلة التجزئة الخاصة بكل فوليو لضمان عدم تزييف القيود القديمة).
13. `exercice_seals` (أختام إغلاق السنة المالية الموقعة رقمياً بـ `RSA Signature`).
14. `ocr_jobs` (حالة ونتائج مهام استخراج الفواتير بالـ `OCR Worker` ومؤشرات الثقة `confidence`).
15. `licenses` (تراخيص تشغيل التطبيق الموقعة والمربوطة بالبصمة العتادية للمكتب).

---

## 6. تصميم الوحدات (Engines Modular Monolith)

يتألف الباك اند من 9 محركات متخصصة معزولة عن بعضها من حيث المسؤولية (`Domain Separation`):
1. **Core Engine:** إدارة الحوكمة، الشركات، سنوات الفتح، وقاعدة منع عدم التوازن (`Double-Entry Invariants Guard`).
2. **Saisie Engine:** محرك الإدخال ونظام الفوليو والتحقق الرباعي الآني وخوارزميات التصحيح (`Modif / CTP / Négative`).
3. **Lettrage Engine:** خوارزميات المطابقة وتجميع الأرصدة المفتوحة ومتابعة فواتير العملاء والموردين.
4. **Analytique & Budgétaire Engine:** توزيع القيود على مراكز التكلفة وحساب الانحرافات.
5. **Immobilisation Engine:** حساب الاهتلاكات وفصل فوارق إعادة التقييم وإصدار القيود التلقائية.
6. **Reporting & Liasse Engine:** محرك التقارير ومترجم صيغ الجباية (`Dictionnaire AST Parser`) وإعداد الحزمة الضريبية.
7. **Import/Export Engine:** قارئ ملفات `DLG` (DBF Format)، وقارئ `PCC` (SDF Format)، ومحرك التبادل عبر `Excel/CSV`.
8. **Security V3 Engine:** إدارة الصلاحيات `RBAC`، تطبيق `RLS`، سلسلة التجزئة `Hash Chain`، سجل التدقيق `Audit WORM`، والنسخ الاحتياطي المشفر (`Backup 3-2-1-1-0`).
9. **OCR Worker Engine:** مصنع معالجة الفواتير محلياً (`Pre-processing, LayoutLMv3 Extraction, Smart Suggestion, Calibration`).

---

## 7. خارطة الطريق ومراحل البناء (14 Phases Roadmap)

تتم إدارة البناء عبر 14 مرحلة دقيقة، يُخصص لكل منها وثيقة تفصيلية وفرع ومراجعة مستقلة:
- **Phase 00:** تأسيس حوكمة المشروع، وثائق الذاكرة الدائمة، وهيكل المستودع.
- **Phase 01:** إثبات المفهوم (`PoCs`) وقياسات الأداء للقرارات المعمارية الحساسة.
- **Phase 02:** تأسيس الـ Monorepo، خطوط CI/CD، وإعدادات التحقق والفحص الآلي.
- **Phase 03:** نمذجة المجال (`Domain-Driven Design`) ونواة القيد المزدوج و `SCF` (بدون UI أو قاعدة بيانات فنية).
- **Phase 04:** قاعدة البيانات (`PostgreSQL 16 Partitioned`)، المعاملات، القيود الميكانيكية، وسجل التدقيق المقاوم للحذف.
- **Phase 05:** واجهة برمجة التطبيقات (`FastAPI Application API`)، العقود (`Pydantic Contracts`)، والأمان المحلي.
- **Phase 06:** واجهة المستخدم المكتبية (`Keyboard-First UI`) ونظام شبكة الإدخال السريع بـ Tauri + React.
- **Phase 07:** محرك التقارير الأساسية (`Journal, Grand Livre, Balance`) ومحرك مترجم صيغ الجباية (`Formula Parser`).
- **Phase 08:** الهجرة والاستيراد والمطابقة (`Migration Engine لملفات DLG و PCC و Excel`).
- **Phase 09:** محرك الـ OCR المحلي (`Offline Document Factory`) وواجهة المراجعة وتأكيد المقترحات.
- **Phase 10:** الأصول الثابتة، المطابقة المتقدمة (`Lettrage Auto/Manuel`)، وتدوير الأرصدة الافتتاحية الدقيق.
- **Phase 11:** الأمن المتقدم (`Security Hardening`)، أختام السنة المالية (`Seals`)، النسخ والتعافي، ونظام التراخيص `RSA`.
- **Phase 12:** الجاهزية للإطلاق التجريبي (`Pilot Readiness`)، اختبارات التحمل الشاملة (`Go/No-Go Gate`).
- **Phase 13:** التعزيز السحابي الاختياري للـ OCR (`Cloud Fallback Opt-in`) والتعلم النشط (`Active Learning`).
- **Phase 14:** التقارير الجبائية الرسمية المتقدمة والتكامل مع النماذج الرسمية المحدثة لمديرية الضرائب (`DGI`).

---

## 8. هيكل المجلدات المعتمد (Monorepo Directory Structure)

```text
compta/
├── .github/                   # قوالب Issues و Pull Requests وحوكمة CI/CD
├── docs/                      # الذاكرة الدائمة ومستندات المجال والمراحل والقرارات
│   ├── DECISIONS/             # القرارات المعمارية الموثقة (ADRs)
│   ├── HANDOFF/               # ملفات التسليم وحالة المشروع اللحظية
│   ├── PHASES/                # ملفات وتقارير ومواصفات المراحل الـ 14
│   ├── SECURITY/              # نماذج التهديد وخطوط الأمان الأساسية
│   ├── TESTING/               # استراتيجية الاختبار ومجموعات البيانات المرجعية
│   └── RUNBOOKS/              # أدلة التشغيل والنسخ والاسترجاع والإصدار
├── apps/
│   ├── desktop/               # نواة Tauri 2.x Rust (Windows Packaging & IPC API)
│   └── ui/                    # واجهة React + TypeScript + Tailwind (Keyboard-First Grid)
├── services/
│   ├── accounting_api/        # خدمة FastAPI (Modular Monolith 9 Engines + SQLAlchemy)
│   └── ocr_worker/            # خدمة Python Offline OCR Worker (PaddleOCR + TrOCR/LayoutLMv3)
├── packages/
│   ├── domain/                # منطق المجال المحاسبي المستقل (Pure Domain Logic & Invariants)
│   └── contracts/             # العقود المشتركة (Pydantic Models & TypeScript Interfaces)
├── db/                        # مخططات قاعدة البيانات، الهجرات (Alembic/SQL)، وملفات التأسيس
├── tests/                     # الاختبارات الشاملة (Unit, Integration, E2E, Property/Adversarial)
├── tools/                     # أدوات التحويل وهجرة ملفات DLG/.PCC والنصائح التشغيلية
├── installers/                # نصوص إعداد وبناء الحزم (Windows NSIS Installers)
├── AGENTS.md                  # قواعد العمل الإلزامية لـ AI Agents
├── CHANGELOG.md               # سجل التغييرات والإصدارات الرسمية
└── README.md                  # الصفحة التعريفية وبوابة المشروع
```
