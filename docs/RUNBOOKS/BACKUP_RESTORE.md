# دليل النسخ الاحتياطي المشفر واستعادة الكوارث (BACKUP_RESTORE.md)

**المشروع:** PC COMPTA Next (`Ahmed47-t/compta`)  
**الإصدار:** 1.0 — يوليو 2026  
**الهدف:** توثيق إجراءات وبروتوكولات النسخ الاحتياطي الدوري المشفر واستعادة البيانات على أجهزة جديدة بضمان سلامة الأختام وعدم ضياع أي قيد محاسبي معتمد.

---

## 1. استراتيجية النسخ الاحتياطي المعتمدة (`3-2-1-1-0 Backup Strategy`)

لحماية مكاتب المحاسبة من مخاطر تلف الأقراص الصلبة، أو هجمات الفدية (`Ransomware`)، أو انقطاع التيار الكهربائي، يعتمد النظام استراتيجية 3-2-1-1-0 الصارمة:
- **3** نسخ من البيانات المحاسبية (البيانات الحية في قاعدة البيانات + نسختين احتياطيتين).
- **2** نوعين مختلفين من وسائط التخزين (القرص الصلب المحلي للحاسوب + قرص خارجي USB/NAS أو سيرفر المكتب).
- **1** نسخة خارجية خارج الموقع (`Off-site / External Storage`).
- **1** نسخة غير قابلة للتعديل أو المحو (`Immutable Snapshot / WORM Archive`) محمية ضد التشفير الخبيث.
- **0** أخطاء في الاسترجاع؛ بحيث يتم فحص وتأكيد إمكانية استعادة النسخة تلقائياً وبانتظام (`Automated Restore Verification with Zero Errors`).

---

## 2. إجراءات وتوليد النسخ الاحتياطي المشفر (`Backup Execution`)

### 2.1 الجدولة والتشفير التلقائي:
- يقوم محرك الأمان `Security Engine` بإنشاء نسخة احتياطية يومية أو عند إغلاق كل فوليو، باستخدام أداة `pg_dump` مع تقسيم وإدراج التوقيع التشفيري:
  ```bash
  pg_dump -U pccompta_admin -d pccompta_prod -F c | openssl enc -aes-256-gcm -pbkdf2 -pass pass:$KEY > /backups/pccompta_backup_YYYYMMDD.enc
  ```
- يتم توليد ملف تجزئة للنسخة (`SHA-256 Checksum`) وحفظه بملف مستقل:
  ```bash
  sha256sum /backups/pccompta_backup_YYYYMMDD.enc > /backups/pccompta_backup_YYYYMMDD.enc.sha256
  ```

### 2.2 إدارة مفاتيح التشفير (`Key Management`):
- يحظر حفظ كلمة مرور التشفير في ملفات نصية مكشوفة. يتم حفظ المفتاح بأمان داخل `Windows Credentials / OS Keychain` المرتبط بجهاز المكتب.

---

## 3. خطوات الاسترجاع والتعافي على جهاز نظيف (`Clean Restore Procedure`)

عند حدوث عطل عتادي ونقل عمل المكتب إلى جهاز حاسوب جديد، يتم اتباع الخطوات التالية بدقة:

### الخطوة 1: فحص سلامة النسخة وتطابق التجزئة (`Integrity Check`):
قبل الشروع في فك التشفير، يتم التحقق من أن النسخة لم تتعرض لأي تلف أو عبث:
```bash
sha256sum -c /backups/pccompta_backup_YYYYMMDD.enc.sha256
```
*(إذا كانت النتيجة `FAILED`، يتم إيقاف العملية فوراً والتبديل للنسخة الاحتياطية التي تسبقها).*

### الخطوة 2: فك تشفير النسخة واستعادة قاعدة البيانات (`Decryption & Restore`):
```bash
# 1. فك تشفير النسخة
openssl enc -d -aes-256-gcm -pbkdf2 -pass pass:$KEY -in /backups/pccompta_backup_YYYYMMDD.enc > /tmp/restore_clean.dump

# 2. إنشاء قاعدة بيانات جديدة واستعادة الجداول
createdb -U pccompta_admin pccompta_restored
pg_restore -U pccompta_admin -d pccompta_restored /tmp/restore_clean.dump
```

### الخطوة 3: التحقق التشفيري والمحاسبي بعد الاسترجاع (`Post-Restore Validation`):
يقوم محرك الأمان تلقائياً بتشغيل بروتوكول فحص سلامة الأختام وسلسلة التجزئة:
1. التحقق من أن جذر التجزئة (`Root Hash`) لكل سنة مالية مغلَقة يطابق التوقيع المشفر المخزن في جدول `exercice_seals`.
2. التحقق من تطابق مجاميع ميزان المراجعة (`SUM(Débit) == SUM(Crédit)`) وإصدار تقرير سلامة التعافي (`Disaster Recovery Audit Certificate`).
