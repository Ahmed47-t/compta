# القيود الرياضية والمحاسبية غير القابلة للخرق (ACCOUNTING_INVARIANTS.md)

**المشروع:** PC COMPTA Next (`Ahmed47-t/compta`)  
**الإصدار:** 1.0 — يوليو 2026  
**الهدف:** توثيق القوانين والمحددات الرياضية والمحاسبية الصارمة التي يجب عدم انتهاكها أو تجاوزها تحت أي ظرف في أي طبقة من طبقات النظام (`Database, Python Domain logic, Tauri Rust UI, or API Contracts`). أي تغيير في كود يؤدي لخرق أحد هذه القيود يُعتبر **خطأً حرجاً مانعاً للقبول (`Blocker / Critical Security & Domain Defect`)**.

---

## 1. قيد الدقة المالية وحظر الفواصل العائمة (`Exact Monetary Precision Invariant`)

### القاعدة المحاسبية:
المبالغ المالية يجب أن تُحسب وتُخزن وتنقل بدقة عشرية ثابتة (`Exact Decimal Representation`) دون أي نسبة تقريب عشوائية أو فقدان دقة ناتج عن تمثيل الأرقام بالفواصل العائمة في الحواسيب (`IEEE 754 Floating-Point Inaccuracies`).

### طريقة الفرض الميكانيكي (`Technical Enforcement`):
- **قاعدة البيانات (PostgreSQL):** يتم تعريف كل الحقول المالية وتحديداً `MONTANT` بـ `NUMERIC(19, 2)` حصرياً. يمنع استخدام `REAL, DOUBLE PRECISION, or FLOAT`.
- **الخلفية (Python Backend):** يتم تمثيل المبالغ باستخدام الكائن `decimal.Decimal` فقط. يحظر التحويل إلى `float` في أي دالة حسابية أو أثناء تسلسل Pydantic (`Pydantic Custom Types with Decimal`).
- **الواجهة (Rust & TypeScript):** في Rust يتم استخدام مكتبة `rust_decimal::Decimal`. في TypeScript يتم تسلسل المبالغ كنصوص (`Strings: "100000.00"`) أو معالجتها عبر مكتبة `big.js / decimal.js` لمنع أخطاء جاڤاسكريبت الرياضية (`e.g., 0.1 + 0.2 != 0.3`).

---

## 2. قيد القيد المزدوج والتوازن الحتمي (`Double-Entry Balance Guard`)

### القاعدة المحاسبية:
لا يُسمح أبداً بوجود قيد محاسبي معتمد (`Posted / Validé`) في سجل القيود تكون فيه مجموع الأرصدة المدينة لا تساوي مجموع الأرصدة الدائنة (`Total Débit = Total Crédit`).

### طريقة الفرض الميكانيكي (`Technical Enforcement`):
- **طبقة المجال (`Domain Logic Boundary`):** التحقق من توازن السند والـ `Folio` ضمن أجزاء من السنتيم (`Tolérance 0.00 DA` في الاعتماد النهائي) قبل استدعاء أمر الحفظ.
- **خط الدفاع الثاني في قاعدة البيانات (`PostgreSQL Database Trigger`):**
  تطبيق Trigger على جدول `pcc_glv` وإجراء التحقق الإلزامي عند محاولة الإغلاق النهائي للفوليو (`Folio Status = CLOSED/POSTED`):
  ```sql
  CHECK (
    (SELECT COALESCE(SUM(montant), 0) FROM pcc_glv WHERE folio_id = NEW.folio_id AND mdebit = TRUE)
    =
    (SELECT COALESCE(SUM(montant), 0) FROM pcc_glv WHERE folio_id = NEW.folio_id AND mdebit = FALSE)
  )
  ```
  في حال عدم التوازن، يرفض محرك قاعدة البيانات التعديل ويرجع خطأ استثنائي (`Database Integrity Violation Error`).

---

## 3. قيد عدم الحذف أو التعديل المادي للقيود المعتمدة (`Immutability of Posted Entries`)

### القاعدة المحاسبية:
أي قيد تم اعتماده وترحيله النهائي (`Status: POSTED / VALIDÉ` أو في `Folio Clôturé`) يصبح غير قابل للحذف المادي (`DELETE`) أو التعديل المباشر في أرقامه أو حساباته (`Direct UPDATE`).

### الطرق القانونية المسموح بها للتصحيح:
1. **القيد العكسي (`Contre-passation`):** إنشاء قيد جديد ذي إشارة دائن/مدين معكوسة يلغي السند القديم مع توثيق العلاقة (`CTP_COM / CTP_AUX`).
2. **الإدخال العكسي بمبالغ سالبة (`Saisie Négative`):** إدخال قيد بنفس الحسابات مع مبالغ سالبة (`-1000.00`) لمنع تضخيم المجاميع في ميزان المراجعة.
3. **التصحيح المبرر الخاضع للتدقيق (`Correction avec motif`):** يُسمح به فقط إذا كان الـ `Folio` مفتوحاً، ويتطلب إدخال نصي لسبب التعديل، ويتم حفظ النسخة السابقة كاملة مع الفرق في جدول سجل التدقيق غير القابل للتعديل (`audit_logs WORM table`).

### طريقة الفرض الميكانيكي (`Technical Enforcement`):
- تطبيق صلاحيات قاعدة بيانات صارمة بـ `Row-Level Security (RLS)` ومنع أوامر `DELETE` على جدول `pcc_glv` للقيود المعتمدة.
- أي محاولة تنفيذ `UPDATE` على قيد معتمد من قبل أي مستخدم أو API يتم رفضها مباشرة عبر `PostgreSQL Before Update Trigger`.

---

## 4. قيد الاتساق الزمني لنظام الفوليو (`Folio & Month Consistency Invariant`)

### القاعدة المحاسبية:
كل قيد يُدرج في اليومية يجب أن يكون تاريخه الفعلي (`SDATE - YYYYMMDD`) متوافقاً بدقة مع الشهر المخصص لرقم الفوليو (`Folio ID`). لا يُقبل قيد مؤرخ في شهر جوان (`SDATE: 20260615`) داخل الفوليو الخاص بشهر مارس (`Folio: 03`).

### طريقة الفرض الميكانيكي (`Technical Enforcement`):
- التحقق التلقائي في طبقة `Saisie Engine` وفي قاعدة البيانات:
  ```text
  Folio 01 -> SDATE month must be 01 (Janvier)
  Folio 02 -> SDATE month must be 02 (Février)
  ...
  Folio 12 -> SDATE month must be 12 (Décembre)
  Folio OUV (00) -> SDATE must be YYYY0101 (Journal d'Ouverture)
  Folio CLO (13) -> SDATE must be YYYY1231 (Journal de Clôture)
  ```

---

## 5. قيد استقلالية القيود الافتتاحية (`Opening Entry & Aging Carry-Forward`)

### القاعدة المحاسبية:
حركات القيود الافتتاحية (`Journal d'Ouverture - OUV / Folio 00`) تمثل الأرصدة المنقولة من السنة السابقة؛ ولا يجوز دمجها أو حسابها ضمن مجاميع دوران السنة المالية الحالية (`Mouvements de l'Exercice`) إلا في التقارير التراكمية المحددة صراحة (`Cumul avec à-nouveaux`).

### قاعدة نقل الأرصدة غير المطابقة (`Report à-nouveaux non lettré`):
عند فتح سنة جديدة ونقل حسابات العملاء (`411`) أو الموردين (`401`) أو أي حساب خاضع للمطابقة (`Soumis au lettrage`)، يجب ألا يتم نقل رصيد إجمالي صامت (`Solde Global`)؛ بل يتم نقل **كل سطر غير مطابق (`Ligne non lettrée`) على حدة** مع الاحتفاظ بتاريخه الأصلي (`SDATE`) ورقم سنده (`PIECE`) وأصل فاتورته من أجل الحفاظ على دقة أعمار الديون (`Suivi de l'ancienneté des créances`).

---

## 6. قيد الحظر المطلق للاعتماد التلقائي لقيود الذكاء الاصطناعي (`No Auto-Posting for OCR`)

### القاعدة المحاسبية والأمنية:
محرك الذكاء الاصطناعي واستخراج الوثائق (`OCR Worker / LayoutLMv3`) يُصنف كأداة مساعدة لاقتراح الإدخال (`Smart Suggestion Tool`). لا يملك الـ OCR أي صلاحية لإنشاء قيود نهائية معتمدة في الدفاتر المحاسبية.

### طريقة الفرض الميكانيكي (`Technical Enforcement`):
- أي قيد يُولده محرك الـ OCR يُسجل في قاعدة البيانات بحالة مسودة (`Status: DRAFT / OCR_SUGGESTION`).
- يحظر برمجياً وجود أي وظيفة خلفية التلقائية (`Cron Job / Background Worker`) تقوم بتبديل حالة القيد المقترح من `DRAFT` إلى `POSTED` أو إدراجه النهائي في الفوليو دون استلام إشعار تأكيد صريح (`Validation Action via Keyboard Enter`) من جلسة المحاسب البشري الموثقة.

---

## 7. قيد التسلسل التشفيري والأختام الجبائية (`Cryptographic Hash Chain & Seals`)

### القاعدة المحاسبية والأمنية:
ضمان شفافية وسلامة الدفاتر المحاسبية وتوفير إثبات غير قابل للطعن في المحاكم وأمام المدقق الجبائي بأن البيانات لم تتعرض لأي تعديل خلفي بعد إغلاق الفترات.

### طريقة الفرض الميكانيكي (`Technical Enforcement`):
- **سلسلة القيود (`Record Hash Chain`):** كل سجل في جدول `pcc_glv` وفي جدول السجلات `audit_logs` يحتوي على حقل `prev_hash` (تجزئة السطر السابق) وحقل `curr_hash` (تجزئة السطر الحالي مع سابقه `SHA-256(prev_hash + row_data)`).
- **أختام الإغلاق (`Exercice Seals`):** عند تنفيذ إغلاق الفوليو أو الإغلاق النهائي للسنة المالية (`Clôture Définitive`)، يقوم النظام بحساب الجذر الكلي (`Root Hash / Merkle Root`) لكامل حركة السنة وتوقيعه رقمياً باستخدام مفتاح تشفير غير متماثل `RSA Private Key` الخاص بالمؤسسة وحفظه في جدول `exercice_seals`. أي تلاعب في بت واحد داخل قاعدة البيانات سيؤدي لكسر التوقيع واكتشافه فوراً.
