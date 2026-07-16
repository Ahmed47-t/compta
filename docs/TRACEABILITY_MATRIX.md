# مصفوفة التتبع والربط المنهجي (TRACEABILITY_MATRIX.md)

**المشروع:** PC COMPTA Next (`Ahmed47-t/compta`)  
**الإصدار:** 1.0 — يوليو 2026  
**الهدف:** ربط كل متطلب وظيفي أو غير وظيفي (`Requirement`) بمعيار القبول (`Acceptance Criteria`) الخاص به، واستراتيجية الاختبار المقررة، والمرحلة التنفيذية (`Phase`) المستهدفة، وحالته الحالية (`Status`) لضمان عدم وجود فجوات في التغطية أو تنفيذ أكواد خارج النطاق.

---

> ### ⚠️ مفتاح قراءة عمود الحالة (`Status Legend`) — مُحدَّث بواسطة المراجع المستقل
> **لا يوجد حالياً أي كود منفَّذ.** القيم في عمود الحالة تعني **اكتمال المواصفة/التوثيق فقط (`Specified`)، وليس التنفيذ أو اجتياز الاختبار**. سيتم تحديثها إلى `Implemented` ثم `Verified` تدريجياً بعد إنجاز كل مرحلة واعتمادها من المراجع المستقل.
> - `Specified — Not Implemented` = المتطلب موثَّق ومُعرَّف بمعيار قبول، لكنه **لم يُبنَ ولم يُختبر** بعد.
> - `Implemented` = كُتب الكود ضمن مرحلته (يحدّثه المنفِّذ).
> - `Verified` = اجتاز اختباراته واعتمده المراجع المستقل.
>
> *(ملاحظة: القيم القديمة المكتوبة كـ "Completed (...)" كانت مضلِّلة؛ صُحِّحت إلى `Specified — Not Implemented`. تبقى التعليقات بين قوسين توضيحية للمواصفة فقط لا للتنفيذ.)*

---

## 1. محرك القيود وإدخال اليوميات (`Core & Saisie Engine`)

| المعرف (`ID`) | المتطلب (`Requirement`) | معيار القبول (`Acceptance Criteria`) | استراتيجية الاختبار (`Test Strategy`) | المرحلة (`Phase`) | الحالة (`Status`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **REQ-CORE-01** | حظر تمثيل المبالغ بالفواصل العائمة (`No Floating Point`) | الدقة المالية التامة باستخدام `NUMERIC(19,2)` في DB و `Decimal` في Python/Rust. | `Unit & Property-based Tests` لاختبارات التقريب والفواصل. | Phase 03 & 04 | **Specified — Not Implemented** (System Spec Verified) |
| **REQ-CORE-02** | منع عدم التوازن في القيود المعتمدة (`Double-Entry Guard`) | عدم إمكانية اعتماد قيد أو إغلاق فوليو إذا كان `SUM(Débit) != SUM(Crédit)`. | `Domain Invariant Test + DB Trigger Constraint Test`. | Phase 03 & 04 | **Specified — Not Implemented** (Specified & Enforced) |
| **REQ-CORE-03** | عدم الحذف المادي للقيود المعتمدة (`No Physical Deletion`) | حظر `DELETE / Direct UPDATE` للقيود بحالة `POSTED / VALIDÉ`. | `Security Integration Test + DB Rule Enforcement`. | Phase 03 & 04 | **Specified — Not Implemented** |
| **REQ-CORE-04** | التحقق الآني من توافق الشهر والفوليو (`Folio Consistency`) | رفض إدخال قيد يحمل تاريخ شهر لا يطابق رقم الفوليو (`01=جانفي..12=ديسمبر`). | `Saisie Engine Unit Tests + DB Constraints`. | Phase 03 & 06 | **Specified — Not Implemented** |
| **REQ-CORE-05** | استقلالية القيود الافتتاحية ونقل الأرصدة (`Report à-nouveaux`) | فصل قيد الافتتاح (`OUV`) ونقل الأسطر غير المطابقة بالتاريخ الأصلي للديون. | `Domain Unit Test + Reconciliation Integration Test`. | Phase 03 & 10 | **Specified — Not Implemented** |
| **REQ-SAISIE-01** | دعم طرق التصحيح المحاسبية (`Modif/CTP/Négative/Motif`) | توفير القيد العكسي، والقيد السالب، والتصحيح المبرر مع الحفظ في سجل التدقيق. | `Domain Workflows Test + Audit Log Verification`. | Phase 03 & 06 | **Specified — Not Implemented** |

---

## 2. واجهة سطح المكتب وسرعة لوحة المفاتيح (`Desktop UI & Keyboard-First`)

| المعرف (`ID`) | المتطلب (`Requirement`) | معيار القبول (`Acceptance Criteria`) | استراتيجية الاختبار (`Test Strategy`) | المرحلة (`Phase`) | الحالة (`Status`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **REQ-UI-01** | التشغيل والتنقل الحصري بـ Keyboard (`Keyboard-First UI`) | إتمام إدخال السندات واليوميات والبحث عبر (`Tab, Enter, Esc, F2, F3, F9`). | `E2E Keyboard Navigation Test + UX Review`. | Phase 06 | **Specified — Not Implemented** (Specified) |
| **REQ-UI-02** | الاستجابة اللحظية لبحث الحسابات (`Instant Lookup < 50ms`) | استجابة بحث الحسابات أو المساعدين في أقل من `50ms` على جهاز i3/4GB RAM. | `Performance Benchmark Script + pg_trgm Index Check`. | Phase 01 & 06 | **Specified — Not Implemented** (PoC Defined) |
| **REQ-UI-03** | سلاسة عرض الجداول الافتراضية (`Virtualized Grid Scrolling`) | عرض 100,000 سطر بسلاسة (`60 FPS`) باستهلاك ذاكرة RAM أقل من `150 MB`. | `UI Stress Performance Test + Memory Profiler`. | Phase 01 & 06 | **Specified — Not Implemented** |

---

## 3. المطابقة والتقارير والجباية (`Lettrage, Reporting & Tax`)

| المعرف (`ID`) | المتطلب (`Requirement`) | معيار القبول (`Acceptance Criteria`) | استراتيجية الاختبار (`Test Strategy`) | المرحلة (`Phase`) | الحالة (`Status`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **REQ-REP-01** | المطابقة اليدوية والآلية (`Lettrage AA, AB...`) | تجميع الأرصدة المتطابقة بالرمز التسلسلي مع سماحية فارق التقريب (`0.01 DA`). | `Lettrage Algorithm Unit Tests + Golden Data Case`. | Phase 03 & 10 | **Specified — Not Implemented** |
| **REQ-REP-02** | مترجم الصيغ الجبائية (`Dictionnaire Formula Compiler`) | ترجمة وتنفيذ الصيغ (`-SOLDE(10)`, `/CRD`) وحساب ميزان 500k قيد في `< 300ms`. | `AST Compiler Unit Tests + SQL Benchmark Script`. | Phase 07 | **Specified — Not Implemented** |
| **REQ-REP-03** | توليد التصريحات الجبائية المعتمدة (`G50 & Liasse Fiscale`) | استخراج تقارير `G50` وتفاصيل `TVA` وجداول الميزانية بصيغ قابلة للطباعة والتصدير. | `Tax Output Comparison vs Official DGI Golden PDF/Excel`. | Phase 07 & 14 | **In Progress** (Phase 07 ready, Phase 14 pending DGI validation) |

---

## 4. محرك الـ OCR واستخراج الوثائق محلياً (`Offline OCR Engine`)

| المعرف (`ID`) | المتطلب (`Requirement`) | معيار القبول (`Acceptance Criteria`) | استراتيجية الاختبار (`Test Strategy`) | المرحلة (`Phase`) | الحالة (`Status`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **REQ-OCR-01** | الاستخراج المحلي 100% دون إنترنت (`100% Offline Extraction`) | استخراج البيانات من فواتير المشتريات/المبيعات عبر `LayoutLMv3` محلياً. | `Offline Environment Test + CER/WER/F1 Benchmark`. | Phase 01 & 09 | **Specified — Not Implemented** |
| **REQ-OCR-02** | الحظر المطلق للاعتماد التلقائي (`No Auto-Posting`) | حفظ القيود المقترحة بحالة `DRAFT` واشتراط تأكيد المحاسب البشري في `Validation UI`. | `Workflow Security Test + API Permission Boundary Check`. | Phase 09 | **Specified — Not Implemented** |
| **REQ-OCR-03** | خوارزمية الاقتراح الذكي والتحقق من المبالغ (`Smart Suggestion`) | التحقق الرياضي من `HT + TVA == TTC` واقتراح المورد والحساب المناسب بناءً على القاموس. | `OCR Rule Calibration Tests + Sample Invoice Dataset`. | Phase 09 | **Specified — Not Implemented** |

---

## 5. الأمان والنسخ الاحتياطي والترخيص (`Security & Operations`)

| المعرف (`ID`) | المتطلب (`Requirement`) | معيار القبول (`Acceptance Criteria`) | استراتيجية الاختبار (`Test Strategy`) | المرحلة (`Phase`) | الحالة (`Status`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **REQ-SEC-01** | تشفير البيانات وإدارة الصلاحيات (`Encryption & RBAC/RLS`) | تشفير `AES-256`، صلاحيات 5 أدوار بالـ `RBAC`، وتقييد الوصول بالـ `RLS`. | `Red Team Penetration Tests + SQL Access Bypass Attempt`. | Phase 04 & 11 | **Specified — Not Implemented** |
| **REQ-SEC-02** | سجل التدقيق المقاوم للحذف (`WORM Audit Append-Only`) | تسجيل كافة التعديلات والحركات بحقول غير قابلة للتعديل وسلسلة تجزئة مستقلة. | `Audit Tamper Test + DB Trigger & Hash Integrity Check`. | Phase 04 & 11 | **Specified — Not Implemented** |
| **REQ-SEC-03** | أختام السنة المالية وسلسلة التجزئة (`Hash Chain & RSA Seals`) | ربط القيود بـ `Hash Chain` وتوقيع الإغلاق النهائي للسنة بمفتاح `RSA Signed Seal`. | `Cryptographic Seal Verification Test + Modification Detection`. | Phase 04 & 11 | **Specified — Not Implemented** |
| **REQ-SEC-04** | النسخ الاحتياطي المشفر (`Encrypted Backup 3-2-1-1-0`) | إجراء نسخ احتياطي دوري مشفر مع التحقق التلقائي من إمكانية الاستعادة بنجاح. | `Disaster Recovery Simulation + Clean Restore Check`. | Phase 11 | **Specified — Not Implemented** |

---

## 6. الهجرة والاستيراد والتبادل (`Migration & Interoperability`)

| المعرف (`ID`) | المتطلب (`Requirement`) | معيار القبول (`Acceptance Criteria`) | استراتيجية الاختبار (`Test Strategy`) | المرحلة (`Phase`) | الحالة (`Status`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **REQ-MIG-01** | هجرة ملفات PC COMPTA الأصلي (`DLG DBF Migration`) | استيراد قواعد بيانات `.DLG` مباشرة مع تطابق مجاميع الأرصدة (`Écart = 0.00 DA`). | `Bit-for-Bit Migration Script Test + Reconciliation Report`. | Phase 08 | **Specified — Not Implemented** |
| **REQ-MIG-02** | استيراد وتصدير الفوليو وملفات Excel (`PCC & Excel Exchange`) | قراءة ملفات `.PCC` (Fixed-length SDF) والتصدير والاستيراد المتبادل عبر Excel/CSV. | `Parser Boundary Tests + Corrupted File Error Handling Test`. | Phase 08 | **Specified — Not Implemented** |
