# المخطط الشامل النهائي لمشروع بديل PC COMPTA
## النسخة الكاملة V1.0 - لا يُهمل أي تفصيل

**تاريخ:** 15-07-2026
**الحالة:** مأخوذ من تحليل فني عميق لملف `pc_compta_technical_review.md` + خطة الأمان V3 + طلب العميل (سرعة + أمان + OCR فقط).

---

### الفهرس
1. الرؤية والهدف
2. ماذا نحتفظ به من PC COMPTA وماذا نقتل
3. المتطلبات الوظيفية الكاملة (40+ ميزة)
4. المتطلبات غير الوظيفية (سرعة، أمان، أداء)
5. المعمارية التقنية النهائية
6. نموذج البيانات الكامل - 15 جدول
7. تصميم الوحدات الـ 9
8. محرك الأمان V3 المدمج
9. محرك OCR - مصنع الاستخراج
10. محرك التقارير والجباية (Dictionnaire)
11. خطة الواجهة (UX تحاكي السرعة)
12. خطة الهجرة من PC COMPTA الأصلي
13. خارطة الطريق 16 أسبوع
14. هيكل المجلدات والكود

---

#### 1. الرؤية والهدف

**الهدف:** ليس بناء نسخة من PC COMPTA، بل بناء **PC COMPTA لو بُني في 2026 بنفس الفلسفة:** سريع كالبرق، آمن كبنك، متوافق 100% مع SCF الجزائري، لكن بدون عيوبه (واجهة قديمة، ملفات غير مشفرة، إدخال يدوي ممل).

**شعار المشروع:** `Offline-first, Keyboard-fast, Mathematically-Secure`

**ما لن نفعل (Non-Goals):**
- لا Chatbot (بطلبك)
- لا تطبيق Web ثقيل يعتمد على الإنترنت
- لا محاسبة معقدة غير جزائرية (نركز SCF فقط في V1)

#### 2. ماذا نحتفظ وماذا نقتل من الأصلي

| نحتفظ به ونطوره | نقتله ونستبدله |
| :--- | :--- |
| فكرة Dossier + Exercice كمجلدات مستقلة | محرك BDE/dBASE -> PostgreSQL Partitioned |
| نظام Folio (01=جانفي) الذكي للسرعة | واجهة Win98 -> Tauri Modern مع نفس سرعة الكيبورد |
| طرق التصحيح الثلاث: Modification, Contre-passation, Saisie Négative | Dongle USB -> License RSA موقع |
| نقل الأرصدة مع الأسطر غير المطابقة Non lettrées | ملفات .DLG غير مشفرة -> تشفير AES-256 |
| Lettrage بـ AA, AB | نسخ احتياطي غير مشفر -> نسخ مشفر + Immutable |
| المحاسبة التحليلية 3 محاور + Budgétaire | تصدير TXT فقط -> Excel + PDF + API |
| Dictionnaire des Formules (-SOLDE, /CRD) | إدخال يدوي لكل فاتورة -> OCR مصنع |
| Consolidation + Multi-exercices en parallèle | |

#### 3. المتطلبات الوظيفية الكاملة

**A. إدارة الهيكل (مأخوذة من PCC_COM, PCC_JRN...):**
1. إنشاء Dossier (شركة) مع NIF, NIS, RC
2. إنشاء Exercice (سنة مالية) مستقل. إمكانية فتح 2025 و 2024 مفتوحان معاً (Multi-exercice)
3. إدارة Plan Comptable: كود 14 حرف رقمي/حرفي، عنوان 30 حرف، نوع (Lettrable, Soumis à auxiliaire, Analytique)
4. إدارة Journaux: كود 8 أحرف، عدد غير محدود، أنواع (ACH, VTE, BQ, CAISSE, OD, OUV)
5. إدارة Auxiliaires: عملاء 411، موردون 401، مع NIF, AI, RC
6. إدارة Natures, Unités, Budgets التحليلية (PCC_NAT, UNT, BDG)
7. قوالب SCF جاهزة: عند إنشاء شركة تجارية، ينشئ 500 حساب و 10 يوميات تلقائياً. (حل نقطة ضعف التأسيس اليدوي)

**B. محرك القيود (PCC_GLV):**
8. السaisie بنظام Folio: كل Folio = شهر. رقم Folio 10 أرقام، LIGNE 10 أرقام تلقائي.
9. الحقول الكاملة كما في الملف الأصلي: CODE_COM(14), CODE_AUX(14), CODE_JRN(8), FOLIO, PIECE(7), LIGNE, SDATE(YYYYMMDD), REFERENCE(12), LIBELLE(40), MDEBIT(Boolean), MONTANT(19,2 max 99B DA), ECHEANCE, POINTAGE(4), CTP_COM(8), CTP_AUX(14)
10. التحقق الفوري الأربعة: (Folio vs mois, وجود الحسابات, إلزامية Aux, توازن الفوليو قبل الإغلاق)
11. التصحيح الثلاثي + رابع محسن: Modification (إذا Folio مفتوح), Contre-passation (يولد قيد عكسي), Saisie Négative (مبالغ سالبة بدون تضخيم المجاميع), + Correction avec motif (يطلب سبب إجباري ويسجل في Audit)
12. Journal d'ouverture منفصل: أرصدة افتتاحية لا تدخل في حركة السنة الحالية.
13. Report des à-nouveaux الذكي: نقل الأسطر Non lettrées فقط (مع تاريخها الأصلي) لمتابعة أعمار الديون، وليس رصيد إجمالي.

**C. المطابقة والتحليل:**
14. Lettrage Manuel & Automatique بمعايير (نفس المبلغ, نفس Aux, نفس Pièce)
15. رموز مطابقة AA, AB... وتقرير الفواتير غير المدفوعة.
16. محاسبة تحليلية 3 محاور وتقارير Balance Analytique
17. محاسبة ميزانياتية ومقارنة Réalisé vs Prévisionnel + Écarts

**D. الأصول الثابتة (PCC_INV, IAF):**
18. تسجيل الاستثمار: Date acquisition vs Date mise en service منفصلان
19. فصل Écart de réévaluation (المادة 44) وحساب اهتلاك لكل جزء منفصل
20. توليد Tableau d'amortissement: Linéaire, Dégressif, Progressif + تصدير Liasse Fiscale

**E. التقارير والجباية:**
21. طباعة متكيفة 80/132 عمود
22. تصدير TXT, Excel, PDF
23. G50 + État TVA récupérable مع NIF, Noms, Montants (كما في الأصلي)
24. Liasse Fiscale: 8 جداول SCF و 21 جدول PCN قديم، طباعة على النماذج الرسمية
25. **Dictionnaire des Formules:** محرك يفهم `-SOLDE(10)`, `SOLDE(401)`, `/CRD`, `/DEB` ويحولها لـ SQL مع Cache.

**F. الربط:**
26. استيراد/تصدير Excel ثنائي الاتجاه لكل الجداول + Balances + Écritures
27. استيراد Folio externe بملفات `.PCC` SDF format (Chr13+10) مع فحص سلامة قبل الإدراج
28. ربط جاهز مع PC PAIE و PC STOCK عبر API (بدل ملفات في V2)
29. Traitements Annexes: تغيير جماعي لحساب، حذف جماعي Folios آمن، تصفية حسابات غير متحركة
30. Consolidation des dossiers مع إلغاء Comptes inter-unités

#### 4. المتطلبات غير الوظيفية

- **السرعة:** فتح ميزان بـ 500,000 قيد في < 300ms. بحث حساب بـ 14 حرف في < 50ms. يعمل على حاسوب i3 بـ 4GB RAM.
- **الأمان:** انظر قسم 8.
- **التوفر:** Offline-first 100%. يعمل بدون إنترنت. المزامنة اختيارية.
- **التوسعة:** إضافة وحدة جديدة (مثلاً Gestion des Stocks) بدون تعديل النواة.

#### 5. المعمارية التقنية النهائية

**Frontend:** Tauri (Rust) + React + TypeScript + Tailwind. لماذا؟ Tauri حجمه 3MB ويستخدم WebView النظام، أسرع 10 مرات من Electron ويستهلك 90% RAM أقل. هذا يحاكي سرعة Delphi الأصلي.

**Backend:** FastAPI (Python) + SQLAlchemy Core (وليس ORM الثقيل) + Pydantic v2 (مكتوب بـ Rust، سريع). Uvicorn ASGI.

**Database:** PostgreSQL 16 مع:
- Partitioning بجداول Exercice: `pcc_glv_2024, pcc_glv_2025` كل Partition ملف مستقل (مثل `C:\PCCOMPTA\...\[السنة]`)
- Extensions: `pgcrypto, pg_trgm (بحث LIBELLE), btree_gin, pgaudit`
- Materialized Views لـ Balances.

**OCR Stack:** Python Service منفصل: `PaddleOCR + TrOCR (للعربية/فرنسية) + LayoutLMv3` لاستخراج الحقول.

**مخطط التدفق:**
`[Tauri UI] -> (IPC) -> [FastAPI API] -> [PostgreSQL]`
`[Tauri UI] -> (Upload PDF) -> [OCR Service] -> JSON -> [FastAPI]`

#### 6. نموذج البيانات الكامل (15 جدول)

**الجداول الأصلية المطورة (9):**
1. `companies` (بدل مجلد Dossier)
2. `exercices` (id, company_id, year, status: OUVERT/CLOTURE_PROVISOIRE/CLOTURE_DEFINITIVE, hash_root, signature)
3. `pcc_com` (plan comptable): code_com PK 14, libelle 30, lettrable bool, soumis_aux bool, nature...
4. `pcc_jrn`: code_jrn PK 8
5. `pcc_aux`: code_aux PK 14, code_com FK, nif, rc...
6. `pcc_glv` (مقسم Partitioned by exercice_id): كل حقول الأصلي + `company_id, exercice_id, hash_folio, created_by`
7. `pcc_inv`: investimenti
8. `pcc_iaf, pcc_bdg, pcc_nat, pcc_unt`

**الجداول الجديدة للأمان والذكاء (6):**
10. `users`: id, username, password_hash Argon2id, role, mfa_secret
11. `audit_logs`: id, user_id, company_id, action_type, table_name, record_id, old_value JSONB, new_value JSONB, diff, computer_name, ip, mac, reason, timestamp, prev_hash, curr_hash
12. `folio_hashes`: folio_id, hash, prev_hash, timestamp, sealed_by
13. `exercice_seals`: exercice_id, root_hash, rsa_signature, sealed_at
14. `ocr_jobs`: id, file_path, status, extracted_json, confidence, created_folio_id
15. `licenses`: license_key, company_nif, hardware_fingerprint, expiry, features

#### 7. تصميم الوحدات الـ 9

**Engine 1 - Core Engine:** إدارة Companies/Exercices + Templates SCF + Double-Entry Guard (Trigger يمنع عدم التوازن)

**Engine 2 - Saisie Engine:** منطق Folio, التحقق الأربعة, التصحيح الأربعة. أهم Engine.

**Engine 3 - Lettrage Engine:** خوارزمية مطابقة: تجميع مبالغ متساوية بـ tolérance 0.01 DA.

**Engine 4 - Analytique & Budgétaire Engine:** توزيع 3 محاور.

**Engine 5 - Immobilisation Engine:** حساب Amortissement مع séparation Écart réévaluation.

**Engine 6 - Reporting & Liasse Engine:** محرك Dictionnaire. يحول `-SOLDE(10)/CRD` إلى `SELECT SUM(CASE WHEN MDEBIT THEN MONTANT ELSE -MONTANT END) FROM pcc_glv WHERE code_com LIKE '10%' AND MONTANT >0`.

**Engine 7 - Import/Export Engine:** Excel, .PCC SDF Parser, .DLG migrator (يقرأ DBF القديم مباشرة).

**Engine 8 - Security Engine:** RBAC, RLS, Hash Chain, Audit WORM, Backup.

**Engine 9 - OCR Engine (طلبك):** بالتفصيل في قسم 9.

#### 8. محرك الأمان V3 المدمج (ملخص)

(راجع ملف security_plan_v3_ultimate.md الكامل)
- Argon2id + RBAC 5 أدوار + RLS
- تشفير AES-256 at-rest + مفاتيح في OS Keychain
- Hash Chain للـ Folios + توقيع RSA عند Clôture Définitive
- Audit Logs Append-Only مع نسختين + Hash Chain للسجلات نفسها
- Backup 3-2-1-1-0 مشفر + Immutable S3
- License RSA موقع بدل Dongle
- ACID حقيقي + PITR

#### 9. محرك OCR - المصنع (بدون Chatbot)

**Pipeline في 5 خطوات:**

**Step 1 - Upload:** المستخدم يسحب PDF فاتورة مورد إلى شاشة Achats. Tauri يحفظه في `ocr/inbox/`.

**Step 2 - Pre-process:** OCR Service يشتغل في الخلفية:
- إزالة ضوضاء, تصحيح ميل, تحويل رمادي.
- استخدام PaddleOCR (يدعم العربية والفرنسية ممتاز) لاستخراج كل النصوص مع Bounding Boxes.

**Step 3 - Extract (الأهم - يربط بـ PCC_GLV):**
نستخدم LayoutLMv3 مدرب على فواتير جزائرية. يخرج JSON جاهز:
```json
{
  "CODE_AUX": "401001",
  "PIECE": "FA2024-889",
  "SDATE": "20240715",
  "REFERENCE": "889",
  "LIBELLE": "Achat Ciment",
  "MONTANT_HT": 100000,
  "TVA": 19,
  "MONTANT_TTC": 119000,
  "NIF": "1234567890",
  "FOURNISSEUR": "SARL BTP"
}
```

**Step 4 - Smart Suggest:**
- إذا CODE_AUX غير موجود, يقترح إنشاءه مع NIF.
- إذا LIBELLE فيه "SONELGAZ" -> يقترح CODE_COM=607100 (Électricité).
- يفحص: MONTANT_TTC = HT + TVA؟ إذا لا, ينبه.
- يولد قيدين: `D 6xx / C 401` + `D 4456 TVA / C 401`.

**Step 5 - Validation UI:**
يظهر Folio مقترح في جدول. المحاسب يراجع ويضغط Enter لتأكيد. 80% من العمل آلي. كل يوم يتعلم من تصحيحات المحاسب (Fine-tuning محلي).

#### 10. محرك التقارير والجباية

- إعادة بناء Dictionnaire كـ AST Compiler: `parse("-SOLDE(10)/CRD") -> SQL`
- Cache للصيغ المتكررة.
- تصدير Liasse على نموذج DGI الحقيقي PDF قابل للطباعة مباشرة.

#### 11. خطة الواجهة - سرعة الكيبورد

- لا تستخدم الماوس. تصميم Keyboard-First: Tab, Enter, Esc, F2 (بحث حساب), F3 (بحث Aux), F9 (توازن Folio).
- شاشة Saisie تحاكي PC COMPTA: جدول Excel-like مع Folio في الأعلى.
- بحث فوري: كتابة 401 -> يظهر كل الموردين في <50ms بفضل pg_trgm.

#### 12. خطة الهجرة

- **مهمة 1:** كتابة `DLG Reader`: يقرأ ملفات DBF القديمة (.DLG) مباشرة بـ Python `dbfread` وينقلها لـ PostgreSQL مع الحفاظ على SDATE كنص.
- **مهمة 2:** Parser ملفات `.PCC` SDF: كل سطر طول ثابت, يفصل بـ Chr13+10.
- **مهمة 3:** مستورد Excel الجاهز.

#### 13. خارطة الطريق 16 أسبوع (4 أشهر)

**المرحلة 0 - التحضير (أسبوع 1):**
إعداد Tauri + FastAPI + PostgreSQL Partitioned + Git Repo جديد.

**المرحلة 1 - النواة الصلبة (أسابيع 2-4):**
- إنشاء 15 جدول + RLS + Argon2id + Audit Append-Only + Hash Chain Base
- إدارة Companies, Exercices, Plan Comptable Templates SCF
- اختبار: إنشاء 100 شركة وهمية.

**المرحلة 2 - محرك القيود (أسابيع 5-8):**
- بناء Saisie Engine مع Folio + التحقق الأربعة + التصحيح الثلاثة + Report à-nouveaux الذكي + Lettrage
- اختبار: إدخال 500,000 قيد وقياس السرعة vs PC COMPTA الأصلي. يجب أن تكون أسرع.

**المرحلة 3 - الأصول والتقارير (أسابيع 9-11):**
- Immobilisation Engine + Analytique 3 محاور + Dictionnaire Compiler + G50 + État TVA + Liasse 8 جداول
- Import/Export Excel + .PCC

**المرحلة 4 - مصنع OCR + الأمان المتقدم (أسابيع 12-14):**
- بناء OCR Service + Smart Suggest + UI التحقق
- تطبيق Backup المشفر + License RSA + توقيع Clôture Définitive

**المرحلة 5 - الصقل والإطلاق (أسابيع 15-16):**
- Consolidation + Multi-exercices + Traitements Annexes
- اختبار اختراق بسيط (حاول حذف السجل, حاول نسخ المجلد)
- بناء Installer Windows (NSIS) + Auto-Updater
- كتابة دليل المستخدم.

#### 14. هيكل المجلدات المقترح

```
/src-tauri (Rust Tauri)
/src (React)
  /components/SaisieGrid (يحاكي PC COMPTA)
  /pages/Dossier, Exercice, Journal...
/backend (FastAPI)
  /engines/core, saisie, lettrage, immo, liasse, security, ocr, import_export
  /models (SQLAlchemy Core)
  /alembic (migrations)
/ocr_service (PaddleOCR + TrOCR)
/scripts/migrate_dlg_to_pg.py
/backups (مشفرة)
```

---

### القرار النهائي لك

لا تبدأ بالواجهة. **ابدأ بالطبقة التي لا يراها العميل لكنها تبيع المنتج: الأمان + السرعة.**

1. أول كود تكتبه: `audit_logs` المقاوم للحذف + `folio_hashes`.
2. ثاني كود: Saisie Engine مع Materialized Balances.

بهذا الترتيب، في نهاية الشهر الأول سيكون عندك نواة أقوى من PC COMPTA ولو لم تكن هناك واجهة بعد.

هل تريد أن أبدأ الآن في كتابة الكود الحقيقي لـ `schema.sql` للـ 15 جدول + `Hash Chain` لتطلقه مباشرة في مشروعك الجديد؟
