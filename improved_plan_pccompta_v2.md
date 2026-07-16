# الخطة المحسنة لبناء بديل PC COMPTA - التركيز على السرعة والأمان و OCR

> هذه الخطة هي نسخة 2.0 محسنة بناء على ملفك الفني `pc_compta_technical_review.md` وملاحظاتك: لا Chatbot، فقط OCR + الاستخراج الآلي.

## أولاً: لماذا PC COMPTA سريع وآمن؟ (التشريح الذي يجب أن نحاكيه)

### سر السرعة:
1.  **نظام الملفات لا قاعدة بيانات ثقيلة:** هو ليس SQL Server، هو ملفات `DBF` مع ملفات فهرسة `MDX`. القراءة مباشرة من القرص. لا يوجد Network Latency. كل سنة مالية هي مجلد مستقل `C:\PCCOMPTA\Dossier\Annee\`. هذا هو Partitioning يدوي.
2.  **الفهارس الخفيفة:** البحث لا يتم بـ Full Scan، بل بفهرس ثنائي جاهز. هذا ما يجعل البحث عن حساب 401001 فوري حتى مع 500 ألف قيد.
3.  **هيكل Folio الذكي:** تجميع القيود في صفحات شهرية (Folio 01 = جانفي) يقلل حجم البحث. لا تبحث في كل القيود، تبحث في Folio واحد.
4.  **تنسيق التاريخ `YYYYMMDD` كنص:** المقارنة نصية سريعة جداً بدون تحويل Date Objects.
5.  **متطلبات تشغيل منعدمة:** يشتغل على Pentium 4 بـ 256MB RAM لأنه مكتوب بـ Delphi Native وليس Electron ثقيل.

### سر الأمان:
1.  **40 سنة من المنطق المحاسبي الصارم:** `Verrouillage par Date`, `par Ecriture`, `Sélectif`, و `Clôture Définitive` غير الرجعية. هذا ليس مجرد Feature، هو جوهر الثقة.
2.  **سجل تدقيق مرعب الدقة:** `Journal des Événements` يسجل: من، متى، من أي جهاز (Computer Name + Windows Session)، القيمة القديمة والجديدة. هذا هو Audit Trail بمستوى بنكي.
3.  **فصل الدورات:** كل سنة ملفات مستقلة = استحالة تلويث سنة بسنة أخرى.
4.  **Dongle USB:** ترخيص مادي يمنع النسخ العشوائي.

**نقاط ضعفه التي سنصلحها:** واجهة Retro، ضعف أمان الملفات (نسخ مجلد C:\PCCOMPTA يسرق كل شيء)، محرك BDE لا يدعم SQL الحديث.

---

## ثانياً: الخطة المحسنة V2 - معمارية هجينة (الهجين هو السر)

لكي نحافظ على السرعة، لن نبني تطبيق Web عادي ثقيل. سنبني **تطبيق Desktop سريع مثل الأصلي، لكن بقلب حديث.**

### 1. الـ Stack التقني المقترح لضمان السرعة

**A. الواجهة الأمامية (Frontend):**
لا تستخدم Electron (بطيء). استخدم **Tauri + React**.
- Tauri مكتوب بـ Rust، حجمه 3MB ويستهلك RAM أقل بـ 90% من Electron. هذا سيعطيك نفس احساس سرعة Delphi الأصلي.
- حافظ على DNA لوحة المفاتيح: اجعل `Enter` ينتقل للحقل التالي و `Esc` يغلق. هذا ما يجعل المحاسبين يحبون PC COMPTA. واجهة حديثة لكن بمنطق قديم سريع.
- وضع Offline-First: مثل الأصلي، يشتغل كاملاً بدون انترنت. المزامنة اختيارية.

**B. قاعدة البيانات (Backend DB):**
لا نعود لـ DBF. نستخدم **PostgreSQL + تقسيم Partitioning يحاكي PC COMPTA:**
- سنقلد هيكل الملفات: `PCC_GLV.DLG` سيصبح جدولاً مقسماً (Partitioned Table) حسب `exercice_year`. كل Partition هو ملف فعلياً مستقل على القرص (تماماً مثل `C:\PCCOMPTA\...\[السنة]`). هذا يعطي سرعة النسخ الاحتياطي والنقل.
- **الفهارس المطابقة لـ .MDX:** 
  ```sql
  CREATE INDEX ON glv (code_jrn, folio, ligne);
  CREATE INDEX ON glv (code_com);
  CREATE INDEX ON glv (code_aux) WHERE code_aux IS NOT NULL;
  CREATE INDEX ON glv (sdate); -- بصيغة YYYYMMDD كنص أيضاً للسرعة
  ```
- استخدم **Materialized Views** لحساب الأرصدة (Balances) لحظياً. بدلاً من حساب `SUM(MONTANT)` في كل مرة، نحسبها مرة ونحدثها عند كل إدخال، مثلما كان PC COMPTA يفعل بملفاته المجمعة.

**C. محرك الصيغ (Dictionnaire des Formules):**
هذه ميزة ثورية في PC COMPTA يجب أن ننسخها لكن بشكل أسرع. نبني Compiler صغير بلغة Python يفهم:
`-SOLDE(10)` و `/CRD` و `/DEB`
ويحولها إلى SQL جاهز. هذا أسرع من التفسير في كل مرة.

### 2. نظام الأمان V2 - أقوى من الأصلي

سنأخذ كل ما في PC COMPTA ونضيف عليه طبقة تشفير حديثة:

| ميزة PC COMPTA الأصلية | كيف نطورها في مشروعك |
| :--- | :--- |
| **Dongle USB** | نستبدله بـ License Key مشفر + Hardware Fingerprint + تفعيل Offline. لا يحتاج USB لكن يمنع النسخ. أمان أعلى وتجربة أفضل. |
| **Mot de passe utilisateur + صلاحيات لكل Dossier** | **RBAC + ABAC حقيقي:** جدول `roles`: Super Admin, Chef Comptable, Opérateur Saisie. Opérateur لا يستطيع تعديل `PCC_COM` (Plan Comptable) أو رؤية `PCC_INV`. نطبقها على مستوى الصف (Row Level Security) في PostgreSQL. |
| **Verrouillage par Date / par Ecriture / Sélectif** | نحتفظ بها كـ Status Flags حقيقية: `exercices.status = 'CLOTURE_DEFINITIVE'`. إذا `CLOTURE_DEFINITIVE`، نطبق Trigger يمنع أي INSERT/UPDATE/DELETE حتى من الـ Admin. |
| **Clôture Définitive غير رجعية** | نضيف عليها **Hash Chain (Blockchain محاسبي):** كل Folio يتم إغلاقه نولد له Hash = SHA256(محتوى Folio + Hash السابق). أي محاولة تعديل مستقبلية ستكسر السلسلة وتُكتشف فوراً. هذا أقوى من PC COMPTA بمراحل. |
| **Journal des Événements** | نبني جدول `audit_logs` يحاكي الأصلي 100% بل وأفضل:<br>`id, user_id, computer_name, ip_address, action_type (MODIFICATION/SUPPRESSION...), table_name, record_id, old_value (JSONB), new_value (JSONB), timestamp`. نحتفظ بالقيمة القديمة والجديدة كـ JSONB لسهولة المراجعة. |
| **أمان الملفات (نقطة ضعف الأصلي)** | في الأصلي، نسخ مجلد PCCOMPTA = سرقة. في مشروعك: **تشفير كامل على القرص (Encryption at Rest)** + تشفير النسخ الاحتياطي + صلاحيات NTFS تلقائية يضعها المثبت. |
| **Vérificateur d'intégrité + Compacter et Réindexer** | PostgreSQL يقوم بها تلقائياً: `pg_checksums`, `VACUUM FULL`, `REINDEX`. نضيف زر "فحص وصيانة" في الواجهة يقوم بهذه العمليات مع تقرير. |
| **Sauvegarde Automatique** | نبنيها أفضل: نسخ عند الإغلاق، لكن إلى 3 أماكن: مجلد محلي + USB تلقائي + S3 متوافق (اختياري). مع ضغط Gzip وتشفير AES-256. مع PITR (Point-in-Time Recovery). |

### 3. طبقة الذكاء الاصطناعي - OCR فقط 

هذا هو قلب التميز. لن نضيف Chatbot، سنركز على **مصنع يستخرج ويعبي.**

**Pipeline مقترح `ai_analyzer.py` V2:**

**المرحلة 1: الاستقبال (Input):**
المستخدم يسحب صورة فاتورة (JPG/PDF) إلى شاشة المشتريات.

**المرحلة 2: المعالجة الذكية (Pre-processing):**
- تحسين الصورة، إزالة التشويش.
- استخدام **PaddleOCR + TrOCR** لأنهما الأفضل للعربية والفرنسية (PC COMPTA جزائري).

**المرحلة 3: الاستخراج الهيكلي (Extraction - الأهم):**
هنا نربط مباشرة بهيكل `PCC_GLV.DLG` الذي درسناه:
نطلب من النموذج إخراج JSON بهذا الشكل بالضبط:
```json
{
  "CODE_AUX": "401001 - ETB EL AMEL",
  "PIECE": "FA-2024-123",
  "SDATE": "20240715",
  "REFERENCE": "FA-123",
  "LIBELLE": "Achat Materiaux",
  "MONTANT": 125000.50,
  "MDEBIT": false,
  "TVA": 19,
  "NIF_FOURNISSEUR": "..."
}
```
لاحظ: نستخرج مباشرة الحقول التي يحتاجها `PCC_GLV` و `PCC_AUX` و `Etat TVA récupérable`.

**المرحلة 4: الاقتراح والتحقق (Smart Suggestion):**
- النظام يقترح `CODE_COM`: إذا رأى كلمة "SONELGAZ" يقترح 607xxx (خدمات) تلقائياً.
- يتحقق من `CODE_AUX`: هل المورد موجود؟ إذا لا، يقترح إنشاءه.
- يتحقق من التوازن: المدين = الدائن قبل الحفظ (نفس منطق PC COMPTA).

**المرحلة 5: التعبئة (Auto-Fill):**
يعبأ Folio كامل جاهز للمراجعة. المحاسب يضغط Enter للتأكيد فقط. هذا يوفر 80% من وقت الإدخال اليدوي (نقطة ضعف PC COMPTA).

**ميزة إضافية مستوحاة من ملفك: استيراد .PCC**
لكي تسهل الانتقال من PC COMPTA، ابنِ مستورد `import_pcc_file()` يقرأ ملفات `.PCC` (SDF format مع Chr13+10) ويحولها مباشرة إلى مشروعك. هذه ميزة هجرة قاتلة.

### 4. خارطة الطريق التنفيذية المحدثة (4 أشهر)

**الشهر 1: النواة السريعة والآمنة (The Fast & Secure Core)**
- إعداد Tauri + React + PostgreSQL Partitioned.
- بناء الجداول التسعة `PCC_COM`...`PCC_UNT` بنفس المواصفات الحرفية (14 حرف للحساب، 40 للبيان...).
- بناء محرك الأقفال الأربعة + Audit Log + Hash Chain.
- زر "Compacter et Réindexer" الحديث.

**الشهر 2: محرك المحاسبة و Folio**
- واجهة السaisie تحاكي سرعة PC COMPTA (Keyboard only).
- تطبيق طرق التصحيح الثلاث: Modification, Contre-passation, Saisie Négative.
- نظام Lettrage (AA, AB...) الآلي واليدوي.
- نقل الأرصدة `Report des à-nouveaux` مع الأسطر غير المطابقة (الميزة التي ذكرتها في الملف).

**الشهر 3: محرك OCR (الضربة القاضية)**
- بناء Pipeline الـ OCR كما هو موضح.
- ربطه بـ `PCC_GLV` و `PCC_AUX`.
- بناء Dictionnaire des Formules Compiler.
- بناء Etat de TVA و G50.

**الشهر 4: التقارير والهجرة**
- بناء Bilan, TCR, TFT بـ 132 عموداً (توافق طباعة) وتصدير Excel ثنائي الاتجاه.
- بناء مستورد .PCC و Excel.
- ميزة Consolidation و Multi-exercices en parallèle (نفس الأصلي).
- تحسين الأداء: Benchmark على 100 ألف قيد ومقارنته بـ PC COMPTA لإثبات السرعة.

---

## الخلاصة: كيف تتفوق على PC COMPTA بدون أن تفقده؟

لا تحارب نقاط قوته (السرعة والأمان و SCF). **احتضنها وطورها:**
- **السرعة:** احتضنتها بـ Tauri + Partitioning + Materialized Views
- **الأمان:** احتضنته بـ Audit Log + Locks + وأضفت Hash Chain + تشفير
- **التوافق:** احتضنت Folio و Saisie Négative و Report à-nouveaux المفصل

وضربته في نقاط ضعفه التي ذكرتها أنت:
- **الإدخال اليدوي:** قتلته بـ OCR
- **قلة SQL:** قتلتها بـ PostgreSQL
- **أمان الملفات:** قتلته بالتشفير
- **واجهة قديمة:** قتلتها بـ Tauri Modern مع الحفاظ على سرعة الكيبورد

بهذه الخطة، مشروعك لن يكون "نسخة من PC COMPTA"، بل سيكون "PC COMPTA لو تم بناؤه في 2026 بنفس الفلسفة".

هل نبدأ الآن في تصميم Schema لقاعدة البيانات `PCC_GLV` الحديثة؟
