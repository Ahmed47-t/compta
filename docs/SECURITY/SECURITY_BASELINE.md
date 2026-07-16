# خط الأساس الأمني والمعايير التقنية للحماية (SECURITY_BASELINE.md)

**المشروع:** PC COMPTA Next (`Ahmed47-t/compta`)  
**الإصدار:** 1.0 — يوليو 2026  
**الهدف:** تحديد المعايير الفنية والخوارزميات والإعدادات الأمنية الصارمة الواجب تطبيقها في كل سطر كود وفي بيئة التشغيل لضمان الصلابة الأمنية (`Mathematically-Secure Architecture`).

---

## 1. معايير التشفير وإدارة كلمات المرور (Cryptography & Password Policies)

### 1.1 تجزئة كلمات المرور (`Password Hashing — Argon2id`)
- **الخوارزمية الإلزامية:** `Argon2id` (يحظر استخدام `MD5, SHA-1, SHA-256, or bcrypt` لحفظ كلمات المرور).
- **المعايير المعتمدة لـ Argon2id:**
  - `Time cost (Iterations): 3`
  - `Memory cost: 65536 KiB (64 MB)`
  - `Parallelism (Threads): 4`
  - `Salt length: 16 bytes minimum (Randomly generated per user)`

### 1.2 التشفير في حالة الراحة والتبادل (`Data at Rest & Transit`)
- **تشفير قاعدة البيانات والملفات الساكنة:** استخدام خوارزمية `AES-256-GCM` لتشفير النسخ الاحتياطية وملفات التصدير الحساسة.
- **تشفير المفاتيح:** يتم حفظ مفتاح التشفير الرئيسي (`Master Encryption Key`) ومفتاح توقيع الأختام داخل خزانة أسرار نظام التشغيل (`Windows Credential Manager / DPAPI / OS Keychain`) وليس أبداً في نصوص الكود أو ملفات الإعدادات النصية (`.env`).

### 1.3 أختام السنوات المالية وسلسلة التجزئة (`Cryptographic Seals & Hash Chains`)
- **تجزئة السجلات المحاسبية (`Hash Chain`):** تُحسب تجزئة كل سطر في اليومية (`pcc_glv`) باستخدام `SHA-256` وفق المعادلة:
  $$\text{curr\_hash} = \text{SHA-256}(\text{prev\_hash} + \text{folio\_id} + \text{code\_com} + \text{montant} + \text{sdate} + \text{piece})$$
- **التوقيع الرقمي للإغلاق (`RSA Signature`):** عند الإغلاق النهائي للسنة المالية (`Clôture Définitive`)، يتم توليد زوج مفاتيح `RSA-4096` أو استخدام المفتاح المعتمد للمكتب لتوقيع الجذر التراكمي (`Root Hash / Merkle Root`) لحركات السنة وحفظه في جدول `exercice_seals`.

---

## 2. التحكم في الوصول وتقييد الصفوف (RBAC & Row-Level Security)

### 2.1 أدوار المستخدمين المعتمدة (`RBAC Roles`)
1. **المدير المالي / المشرف العام (`Super-Admin / DAF`):** صلاحية كاملة لإدارة الشركات، وإعداد دليل الحسابات، وفتح/إغلاق السنوات المالية، وتعيين الصلاحيات.
2. **الخبير المحاسبي (`Expert-Comptable / Chef de Mission`):** صلاحية اعتماد وتصحيح القيود (`Correction avec motif / Contre-passation`)، ومراجعة المطابقة (`Lettrage`)، وإصدار الحزمة الجبائية (`Liasse Fiscale`).
3. **محاسب الإدخال (`Comptable-Saisie`):** صلاحية الإدخال في الفوليو المفتوح فقط (`Draft & Saisie in Open Folio`)، واقتراحات الـ OCR، والمطابقة اليدوية الأولية. يحظر عليه إغلاق اليوميات أو التعديل في القيود المعتمدة.
4. **المراجع خارجي / المدقق (`Auditeur / Commissaire aux Comptes`):** صلاحية القراءة فقط (`Read-Only Access`) لكل اليوميات والموازين وسجل التدقيق غير القابل للمحو (`WORM Audit Log`).
5. **مستخدم ضيف / مساعد (`Guest / Stagiaire`):** صلاحية قراءة أو إدخال مسودات مقيدة في يوميات محددة فقط.

### 2.2 سياسات أمان قاعدة البيانات (`PostgreSQL RLS Policies`)
- يجب تفعيل `ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;` على جميع الجداول الحساسة (`pcc_glv, pcc_aux, audit_logs, exercice_seals`).
- فرض سياسة الرفض الافتراضي (`Deny-by-default`): أي استعلام لا يحمل سياق شركة صالح (`app.current_company_id`) وسياق دور صالح يُرفض فوراً من طرف قاعدة البيانات.

---

## 3. سجل التدقيق المقاوم للمحو (WORM Audit Append-Only Engine)

- **جدول `audit_logs`:** يُعرف في قاعدة البيانات بحيث يتم منح تطبيق أو خدمة المحاسبة صلاحية `INSERT` و `SELECT` فقط عليه. يُمنع نهائياً منح أو تنفيذ `UPDATE` أو `TRUNCATE` أو `DELETE` عليه لأي دور في التطبيق.
- **إضافة `pgaudit`:** تُفعل على مستوى PostgreSQL لتسجيل أي محاولة وصول إداري أو تعديل هيكلي (`DDL/DML`) خارج الواجهة وتخزينها في سجلات نظام منفصلة.
- **بيانات السجل الإلزامية:** كل عملية إدخال أو تعديل قيد يجب أن تسجل: `(user_id, company_id, action_type, table_name, record_id, old_value JSONB, new_value JSONB, computer_name, ip, timestamp, prev_hash, curr_hash)`.

---

## 4. فحص الأمان وإدارة الاعتماديات (CI/CD Security Gate)

- **مسح الأسرار (`Secret Scanning`):** تشغيل أداة `gitleaks / trufflehog` في كل Commit وفي خط الـ CI للتأكد من عدم تسريب مفاتيح أو كلمات مرور أو شهادات في الكود المصدر.
- **تحليل الثغرات الساكن (`SAST & Dependency Scanning`):** تشغيل `bandit` و `pip-audit` لـ Python، و `npm audit / snyk` لـ TypeScript، و `cargo audit` لـ Rust لضمان خلو الحزم من الثغرات المكتشفة (`CVEs`).
