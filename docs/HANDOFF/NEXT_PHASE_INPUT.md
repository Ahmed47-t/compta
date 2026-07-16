# مدخلات ومتطلبات المرحلة التالية (NEXT_PHASE_INPUT.md)

**المرحلة التالية المستهدفة:** **Phase 01 (`Discovery, PoCs & Architectural Benchmarks`)**  
**تاريخ الإعداد:** 2026-07-16  
**الفرع المستهدف للعمل:** `arena/019f69c9-compta` (أو الفرع المخصص للمرحلة 01)  
**الخبير المطلوب:** Principal Desktop Architect + Database/Performance Engineer  

---

## 1. الهدف والمهمة الأساسية للـ Agent القادم

مهمتك في المرحلة 01 ليست بناء شاشات أو محرك محاسبي إنتاجي؛ بل **حسم القرارات التقنية عالية المخاطر بأدلة ومقاييس أداء مادية وقابلة للتكرار (`Proof of Concepts & Benchmarks`)**. يجب أن تحول كل افتراض نظري حول السرعة والأداء إلى نصوص قياس برمجية (`Scripts`) موثقة ومؤيدة بوثائق قرارات معمارية (`ADRs`).

---

## 2. الملفات المرجعية التي يجب قراءتها قبل بدء العمل

1. `AGENTS.md`: لضمان الالتزام بقواعد العمل وحظر الفواصل العائمة وفرض التوازن.
2. `docs/MASTER_PLAN.md`: لفهم المعمارية العامة (`Tauri + FastAPI + PostgreSQL + Local OCR`).
3. `docs/SCOPE.md`: لتجنب الخروج عن نطاق المرحلة وبناء ميزات سابقة لأوانها.
4. `docs/ACCOUNTING_INVARIANTS.md`: لضمان عدم انتهاك القوانين المالية في تجارب قياس قاعدة البيانات.
5. `docs/DECISIONS/ADR-0001-architecture.md`: القرار المعماري الأساسي.
6. `docs/PHASES/PHASE-01.md`: مواصفات ونطاق المرحلة 01 التفصيلية.

---

## 3. قائمة المهام والمخرجات المطلوبة (`Scope of Tasks for Phase 01`)

يجب على الخبير المنفذ للمرحلة 01 إنجاز النماذج الستة التالية:
1. **PoC 1 (Tauri <-> FastAPI IPC Latency):**
   - بناء نص تجريبي أو تطبيق مصغر يقيس زمن استدعاء API محلي بين Tauri (Rust) و FastAPI (Python) والتأكد من أن `Latency < 10ms`.
   - توثيق النتيجة في `ADR-0002-ipc-fastapi-connection.md`.
2. **PoC 2 (PostgreSQL Local Windows vs SQLite):**
   - كتاية برنامج نصي يولد 500,000 قيد محاسبي في `PostgreSQL 16` وآخر في `SQLite/SQLCipher`.
   - تنفيذ استعلام ميزان مراجعة (`Balance`) وقياس الزمن بـ `EXPLAIN ANALYZE` ومقارنة زمن الاستجابة واستهلاك الذاكرة ومشاكل التزامن.
   - توثيق النتيجة في `ADR-0003-postgres-vs-sqlite-benchmark.md`.
3. **PoC 3 (Saisie Virtualized Grid Speed):**
   - تجربة مكون شبكة افتراضية (`React Virtualized Grid`) مع 100,000 سطر واختبار اختصارات الكيبورد (`Tab, Enter, F2, F3`) ومعدل الإطارات (`60 FPS`) واستهلاك الذاكرة.
   - توثيق النتيجة في `ADR-0004-virtualized-grid-benchmark.md`.
4. **PoC 4 (Local OCR Baseline Performance):**
   - إعداد وتشغيل قياس لأداء نموذج `PaddleOCR/ONNX` محلياً على صور فواتير اختبارية، وقياس استهلاك المعالج (`CPU/RAM`) وزمن المعالجة للصفحة الواحدة.
   - توثيق النتيجة في `ADR-0005-local-ocr-baseline.md`.
5. **PoC 5 (Scanner Adapter Mock):**
   - إعداد محاكي محول الماسح الضوئي (`WIA/TWAIN Adapter Mock`) الذي سيُستخدم في البيئات التجريبية.
6. **PoC 6 (DLG & PCC Read-Only Analysis):**
   - كتاية نص بايثون لقراءة وفحص بنية عينة ملفات `.DLG` (DBF format) و `.PCC` (SDF format) في وضع `Read-Only` دون تغيير مصدرها.

---

## 4. إرشادات إيداع المخرجات وبوابة الخروج (`Definition of Done for Phase 01`)

- تُودع جميع نصوص القياس والتجارب في المجلد: `tools/poc_benchmarks/`.
- تُودع جميع قرارات الـ `ADRs` في المجلد: `docs/DECISIONS/`.
- عند الانتهاء، قم بتحديث الملفات التالية قبل فتح الـ PR:
  - `docs/HANDOFF/CURRENT_STATE.md`
  - `docs/HANDOFF/LAST_PHASE_REPORT.md`
  - `docs/TRACEABILITY_MATRIX.md` (تحديث حالة متطلبات الأداء لـ `PoC Verified`)
  - `docs/RISK_REGISTER.md` (إضافة أو إغلاق المخاطر التي تم قياسها)
  - `CHANGELOG.md`
- افتح Pull Request موثقاً ولا تقم بدمجه بنفسك؛ انتظر وكيل المراجعة المستقل (`Phase 01 Reviewer`).
