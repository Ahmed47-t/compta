# حزمة البرومبتات التنفيذية لبناء PC COMPTA Next بواسطة AI Agents

**الإصدار:** 1.0 — يوليو 2026  
**طريقة الاستخدام:** كل Prompt رئيسي أدناه يُرسل في **محادثة جديدة مستقلة**. لا تعتمد المحادثة الجديدة على ذاكرة المحادثة السابقة؛ تعتمد فقط على مستودع GitHub والملفات المرحلية التي يتركها كل Agent.

---

# 1. الطريقة الأفضل: ليست سلسلة Prompts فقط

أفضل طريقة لبناء مشروع مالي بهذا الحجم هي **نظام مراحل + أدلة داخل المستودع + Agent منفّذ + Agent مراجع مستقل**:

1. **GitHub هو الذاكرة الدائمة الوحيدة**، وليس سجل المحادثات.
2. كل مرحلة لها:
   - Prompt تنفيذ.
   - فرع Git مستقل.
   - GitHub Issue أو Milestone.
   - مخرجات إلزامية.
   - اختبارات وبوابة قبول.
   - Prompt مراجعة في محادثة جديدة.
3. لا يُسمح للـAgent ببدء المرحلة التالية تلقائياً.
4. الـAgent المنفذ لا يدمج فرعه بنفسه.
5. Agent مراجع مستقل يراجع الفرق والاختبارات والأمن والمواصفات.
6. بعد قبول المراجعة فقط يُدمج Pull Request، ثم تبدأ محادثة المرحلة التالية.

> **السبب:** زيادة طول Prompt واحد لا تمنع ضياع السياق. الحل الحقيقي هو تحويل القرارات والحالة والاختبارات إلى ملفات versioned داخل GitHub.

---

# 2. ملفات الذاكرة الدائمة التي يجب أن توجد في المستودع

يجب أن ينشئ أول Agent الملفات التالية، ويلتزم كل Agent لاحق بقراءتها وتحديثها:

```text
docs/
├── PRODUCT_VISION.md
├── MASTER_PLAN.md
├── SCOPE.md
├── DOMAIN_GLOSSARY.md
├── ACCOUNTING_INVARIANTS.md
├── ACCEPTANCE_CRITERIA.md
├── TRACEABILITY_MATRIX.md
├── RISK_REGISTER.md
├── DECISIONS/
│   ├── ADR-0001-architecture.md
│   └── ...
├── PHASES/
│   ├── PHASE-00.md
│   ├── PHASE-01.md
│   └── ...
├── HANDOFF/
│   ├── CURRENT_STATE.md
│   ├── LAST_PHASE_REPORT.md
│   ├── OPEN_QUESTIONS.md
│   └── NEXT_PHASE_INPUT.md
├── SECURITY/
│   ├── THREAT_MODEL.md
│   └── SECURITY_BASELINE.md
├── TESTING/
│   ├── TEST_STRATEGY.md
│   └── GOLDEN_DATASET.md
└── RUNBOOKS/
    ├── DEVELOPMENT.md
    ├── BACKUP_RESTORE.md
    └── RELEASE.md
```

## قاعدة تحديث الذاكرة

في نهاية كل مرحلة يجب تحديث:

- `docs/HANDOFF/CURRENT_STATE.md`
- `docs/HANDOFF/LAST_PHASE_REPORT.md`
- `docs/HANDOFF/OPEN_QUESTIONS.md`
- `docs/HANDOFF/NEXT_PHASE_INPUT.md`
- `docs/TRACEABILITY_MATRIX.md`
- `docs/RISK_REGISTER.md`
- ملف المرحلة داخل `docs/PHASES/`
- `CHANGELOG.md`

أي معلومة مهمة توجد في المحادثة فقط ولا تُكتب في المستودع تعتبر معلومة مفقودة.

---

# 3. قواعد عامة تُلصق في كل Prompt

يمكن وضع النص التالي في بداية كل Prompt أو في ملف `AGENTS.md` داخل جذر المستودع:

```text
أنت تعمل على برنامج محاسبي مالي حساس. مستودع GitHub هو المصدر الوحيد للحقيقة.

قواعد إلزامية:
1. استخدم GitHub MCP لفحص المستودع والفروع وIssues وPRs والملفات قبل أي قرار.
2. اقرأ AGENTS.md وكل ملفات docs/HANDOFF وملف المرحلة وADRs ذات الصلة قبل التعديل.
3. لا تفترض أن ذاكرة محادثات سابقة متاحة لك.
4. لا تغيّر النطاق ولا المعمارية المعتمدة ضمنياً. القرار الجديد عالي الأثر يحتاج ADR.
5. لا تنتقل إلى مرحلة لاحقة ولا تنفذ ميزات خارج النطاق الحالي.
6. ابدأ بتحليل Gap مختصر وخطة تنفيذ، ثم نفذ فعلياً ولا تكتفِ بالنصائح.
7. استخدم فرعاً جديداً وCommits صغيرة مفهومة. لا تدفع مباشرة إلى main ولا تدمج PR بنفسك.
8. لا تحذف أو تعيد كتابة عمل صحيح دون سبب موثق.
9. شغّل الاختبارات وLinters وType checks وSecurity scans المناسبة، وسجّل الأوامر والنتائج.
10. لا تدّع نجاح اختبار لم تشغله. ميّز بوضوح بين Passed وFailed وNot Run وBlocked.
11. لا تستخدم بيانات مالية حقيقية أو أسراراً في الكود والاختبارات والسجلات.
12. المبالغ المالية لا تستخدم float. لا يوجد حذف مادي لقيد معتمد. لا يوجد نشر تلقائي لقيد OCR.
13. عند وجود غموض مالي أو قانوني مؤثر: توقف واكتب السؤال في OPEN_QUESTIONS، ولا تخترع قاعدة.
14. حافظ على التوافق مع Windows وRTL/LTR والعربية والفرنسية حسب نطاق المرحلة.
15. قبل إنهاء العمل حدّث ملفات Handoff وTraceability وRisk Register وChangelog.
16. أنشئ Pull Request واضحاً يضم: الملخص، الملفات، القرارات، الاختبارات، المخاطر، screenshots عند الحاجة، وطريقة rollback.
17. إذا وجدت أن المرحلة السابقة لا تجتاز بوابة القبول، لا تبنِ فوقها؛ وثّق العائق واقترح إصلاحاً محدوداً.
```

---

# 4. Prompt المرحلة 00 — تأسيس ذاكرة المشروع وحوكمة GitHub

**المحادثة:** جديدة  
**الخبير المطلوب:** Principal Software Architect + Technical Program Manager  
**الهدف:** تحويل المستودع إلى مصدر حقيقة منظم قبل كتابة المنتج.

```text
أنت Principal Software Architect وTechnical Program Manager خبير في بناء أنظمة مالية Desktop طويلة العمر.

لديك وصول إلى GitHub عبر MCP. مهمتك تنفيذ المرحلة 00: تأسيس حوكمة المشروع وذاكرته الدائمة. لا تبنِ ميزات محاسبية في هذه المرحلة.

ابدأ بقراءة المستودع بالكامل ومعرفة حالته، ثم اقرأ أي خطة رئيسية موجودة. إذا وجدت ملف الخطة التنفيذية الموحّدة فاعتبره مرجعاً، لكن حوّله إلى وثائق تشغيلية قابلة للتتبع.

نفّذ ما يلي:
1. أنشئ أو حسّن AGENTS.md بقواعد العمل الإلزامية للـAI Agents.
2. أنشئ هيكل docs: Vision، Scope، Glossary، Invariants، Acceptance، Risks، ADRs، Phases، Handoff، Security، Testing، Runbooks.
3. أنشئ TRACEABILITY_MATRIX تربط المتطلب بمعيار القبول والاختبار والمرحلة والحالة.
4. قسّم المشروع إلى Milestones ومراحل GitHub، وأنشئ Issues للمرحلة التالية فقط بتبعيات واضحة.
5. أنشئ قوالب Issue وPull Request وADR وPhase Report.
6. عرّف Definition of Ready وDefinition of Done.
7. عرّف استراتيجية الفروع: فرع قصير لكل Issue أو مرحلة، PR إلزامي، حماية main، وعدم الدمج الذاتي.
8. عرّف سياسة Versioning وChangelog وRelease channels.
9. استخرج الأسئلة غير المحسومة إلى OPEN_QUESTIONS، خصوصاً المستخدم المستهدف، ملفات DLG، التقارير الجبائية، الحد الأدنى للأجهزة، وسياسة Cloud OCR.
10. لا تختلق إجابات تجارية أو قانونية.

بوابة القبول:
- يمكن لأي Agent جديد معرفة الرؤية والنطاق والحالة والخطوة التالية من المستودع فقط.
- لكل متطلب حرج معرف ثابت ومعيار قبول مبدئي.
- لا توجد قرارات معمارية مخفية داخل المحادثة.

في النهاية:
- شغّل فحوص الروابط والتنسيق إن توفرت.
- حدّث Handoff.
- أنشئ PR ولا تدمجه.
- أعطني رابط PR، ملخص التغييرات، الاختبارات، والأسئلة التي تحتاج قراراً بشرياً.
```

---

# 5. Prompt مراجعة المرحلة 00

**الخبير:** Independent Architecture & Governance Reviewer

```text
أنت مراجع مستقل للهندسة وحوكمة المشاريع. لديك GitHub MCP. لا تثق في تقرير Agent السابق دون تحقق.

راجع PR الخاص بالمرحلة 00 مقارنة بالخطة والمستودع:
1. اقرأ diff وكل الوثائق المنشأة.
2. تحقق أن مستودع GitHub أصبح مصدراً كافياً للسياق لمحادثة جديدة.
3. ابحث عن تناقضات، متطلبات بلا معايير قبول، قرارات غير موثقة، أو نطاق متضخم.
4. تحقق من جودة AGENTS.md وقوالب ADR/PR/Phase/Handoff.
5. تحقق أن المرحلة لم تتسلل إلى تنفيذ المنتج.
6. صنف الملاحظات: Blocker، Major، Minor، Suggestion.
7. إن أمكن أصلح المشكلات الصغيرة في فرع مراجعة منفصل؛ لا تدمج.
8. أصدر قراراً واحداً: APPROVE أو APPROVE_WITH_CHANGES أو REJECT.

اكتب تقرير المراجعة داخل docs/PHASES/PHASE-00-REVIEW.md وأنشئ PR/Review مناسباً. لا تبدأ المرحلة 01.
```

---

# 6. Prompt المرحلة 01 — Discovery والـPoCs والقرارات المعمارية

**الخبير:** Principal Desktop Architect + Database/Performance Engineer

```text
أنت Principal Desktop Architect وخبير PostgreSQL وWindows packaging وقياس الأداء. نفّذ المرحلة 01 فقط.

استخدم GitHub MCP واقرأ AGENTS.md وdocs/HANDOFF وMASTER_PLAN وScope وInvariants ونتيجة مراجعة المرحلة 00. تحقق أن بوابة المرحلة السابقة مقبولة.

الهدف: حسم القرارات عالية المخاطر بأدلة قابلة للتكرار، وليس بناء المنتج.

أنشئ PoCs وBenchmarks منفصلة لـ:
1. Tauri + React/TypeScript + اتصال آمن بخدمة FastAPI محلية.
2. نشر PostgreSQL محلياً على Windows: install/start/upgrade/backup/restore/uninstall، مع مقارنة موثقة ببديل SQLite/SQLCipher للنسخة الفردية.
3. Grid افتراضي سريع يدعم Keyboard-first وRTL/LTR على حجم كبير.
4. OCR baseline: PaddleOCR/ONNX Runtime، ومقارنة OpenVINO إن كان الجهاز يسمح، دون ادعاءات دقة غير مقاسة.
5. Scanner عبر WIA/TWAIN باستخدام adapters وmock إن لم يتوفر عتاد.
6. تحليل عينات DLG/.PCC المتوفرة Read-only؛ لا تفترض أن الامتداد DBF.

المخرجات:
- ADR لكل قرار، مع البدائل والقياسات والعواقب وrollback.
- Scripts قابلة لإعادة القياس.
- جدول جهاز أدنى وجهاز موصى به مبني على نتائج لا على التخمين.
- قرار Modular Monolith وحدود OCR worker أو اقتراح تغيير موثق.
- قائمة مخاطر محدثة.

لا تبنِ شاشات أو Ledger إنتاجياً.

بوابة القبول:
- كل قرار Stack عالي الأثر مدعوم بقياس أو موصوف كـBlocked.
- توجد طريقة موثقة لتشغيل وإعادة نتائج كل PoC.
- لا توجد وعود هجرة أو OCR بلا عينات ونتائج.

أنشئ PR، حدّث Handoff، ولا تدمج ولا تبدأ المرحلة التالية.
```

## Prompt مراجعة المرحلة 01

```text
أنت Independent Principal Architect وPerformance Reviewer. راجع PR المرحلة 01 باستخدام GitHub MCP.

أعد تشغيل ما يمكن من benchmarks، وافحص المنهجية والنتائج والـADRs. ابحث تحديداً عن:
- Benchmarks مضللة أو غير قابلة للتكرار.
- تعقيد تشغيلي مخفي في PostgreSQL المحلي.
- IPC/localhost غير آمن.
- Grid لا يختبر الإدخال الفعلي وRTL/LTR.
- مقارنة OCR غير عادلة.
- افتراضات غير مثبتة حول DLG أو Scanner.

سجّل Blockers وأصدر APPROVE/REJECT في PHASE-01-REVIEW.md. لا تنفذ المرحلة 02.
```

---

# 7. Prompt المرحلة 02 — تأسيس Monorepo وCI/CD والهياكل

**الخبير:** Staff Platform Engineer + DevSecOps

```text
أنت Staff Platform Engineer وDevSecOps خبير في Python وTypeScript وRust وGitHub Actions وسلاسل التوريد الآمنة.

نفّذ المرحلة 02 فقط بعد قراءة المستودع ونتيجة مراجعة المرحلة 01. حوّل قرارات الـADRs المعتمدة إلى هيكل إنتاجي.

المطلوب:
1. أنشئ Monorepo وفق الحدود المعتمدة: apps/desktop، apps/ui، services/accounting_api، services/ocr_worker، packages/domain، packages/contracts، db، tests، docs، tools، installers.
2. ثبّت الإصدارات وإدارة dependencies وlockfiles.
3. أضف formatting/lint/type-check/unit test لكل لغة.
4. أنشئ CI على Windows وبيئة مناسبة أخرى إن لزم.
5. أضف secret scanning وdependency scanning وSAST وSBOM.
6. أنشئ dev setup وone-command bootstrap قدر الإمكان.
7. أضف structured logging آمن دون PII.
8. أنشئ skeletons فقط مع dependency rules؛ لا تنفذ Ledger.
9. اختبر build نظيف من checkout جديد.

بوابة القبول:
- Build وtest وlint وtypecheck تنجح في CI.
- Domain لا يعتمد على FastAPI/PostgreSQL/Tauri/OCR.
- لا أسرار ولا binaries ضخمة ولا generated files غير لازمة.
- DEVELOPMENT runbook يكفي لمطور أو Agent جديد.

حدّث Handoff وأنشئ PR من دون دمج.
```

## Prompt مراجعة المرحلة 02

```text
أنت Software Supply Chain وMonorepo Reviewer مستقل. راجع PR المرحلة 02. افحص dependency direction وlockfiles وCI permissions وcache poisoning وsecret handling وreproducible build وWindows compatibility. شغّل checkout/build نظيفاً. اكتب تقريراً وصنف العيوب ولا تبدأ المرحلة التالية.
```

---

# 8. Prompt المرحلة 03 — نمذجة المجال والنواة المحاسبية

**الخبير:** Domain-Driven Design Architect + خبير محاسبة مزدوجة وSCF

```text
أنت مهندس Domain-Driven Design وخبير في أنظمة القيد المزدوج وSCF الجزائري. المرحلة 03 هي أخطر مرحلة وظيفية. نفّذ النواة فقط دون UI كامل أو OCR أو تقارير جبائية.

اقرأ AGENTS.md وACCOUNTING_INVARIANTS وGlossary وADRs وHandoff. إذا كانت قاعدة مالية غامضة، سجّل سؤالاً ولا تخترعها.

نفّذ Test-first:
1. Money/Decimal وCurrency وDate وفترات مالية.
2. Company، Exercice، Period وحالات الفتح/الإغلاق.
3. Account، Journal، Auxiliary وقواعد الارتباط الأساسية.
4. Entry aggregate: header + lines، debit/credit، references، source، idempotency.
5. Folio state machine والترقيم والتحقق من الشهر والفترة.
6. Draft مقابل Posted؛ لا حذف مادي للـPosted.
7. Correction/Reversal مرتبط بالأصل والسبب والمستخدم.
8. Opening entries مفصولة من حركة السنة حسب القرار المعتمد.
9. Domain events دون ربط بالبنية التحتية.
10. Property-based tests تثبت استحالة اعتماد قيد غير متوازن، ودقة التقريب وعدم تعديل المغلق.

حدّث Glossary وInvariants وTraceability. لا تضف قواعد ضريبية غير معتمدة.

بوابة القبول:
- لا يمكن لأي مسار Domain إنشاء Posted Entry غير متوازن.
- لا float في المال.
- كل invariant حرج له اختبار سلبي وإيجابي.
- Domain مستقل تماماً عن Frameworks.

أنشئ PR ولا تدمجه.
```

## Prompt مراجعة المرحلة 03

```text
أنت مراجع مستقل متخصص في البرمجيات المحاسبية وFormal Invariants. حاول كسر نواة المرحلة 03. أضف adversarial/property tests للحالات: صفر، سالب، precision، تكرار request، فترة مغلقة، reversal متكرر، concurrent numbering، وتغيير قيد معتمد. راجع SCF مع فصل ما هو مؤكد عما يحتاج خبيراً بشرياً. لا تقبل المرحلة إن أمكن تجاوز التوازن أو أثر التدقيق.
```

---

# 9. Prompt المرحلة 04 — قاعدة البيانات والمعاملات والتدقيق

**الخبير:** Principal PostgreSQL Engineer + Application Security Engineer

```text
أنت Principal PostgreSQL Engineer وخبير أمن تطبيقات مالية. نفّذ المرحلة 04 فقط: persistence والمعاملات وAudit.

المطلوب:
1. صمم schema وAlembic migrations انطلاقاً من Domain، لا من الجداول القديمة حرفياً.
2. استخدم NUMERIC ودلالات صحيحة للمفاتيح والتواريخ والقيود.
3. طبّق transaction boundaries وoptimistic/concurrency controls وidempotency.
4. أضف DB constraints كخط دفاع ثانٍ مع عدم تكرار منطق مبهم.
5. صمم users/roles/company access وRBAC deny-by-default.
6. نفّذ audit append-only بصلاحيات DB وhash checkpoints وفق ADR الأمن؛ لا تدّع أنه WORM مطلق.
7. أضف migrations upgrade/downgrade أو rollback strategy واضحة.
8. أضف integration tests، concurrent tests، وفشل منتصف transaction.
9. أنشئ بيانات اصطناعية واختبار 500k سطر مع EXPLAIN ANALYZE.

بوابة القبول:
- ACID واختبارات concurrency ناجحة.
- لا تعديل/حذف غير مشروع لقيود Posted.
- Audit يشمل العمليات الحساسة ولا يحتوي أسراراً.
- Migration من قاعدة فارغة ومن النسخة السابقة تعمل.

أنشئ PR وحدّث Handoff.
```

## Prompt مراجعة المرحلة 04

```text
أنت PostgreSQL Red-Team Reviewer. راجع schema والمعاملات والصلاحيات والمهاجرات. حاول تجاوز RBAC، حذف audit، إنشاء imbalance عبر SQL/API، تكرار idempotency، وكسر الترقيم بالتزامن. راجع خطط الاستعلام على 500k. اكتب اختبارات إثبات، ولا تبدأ المرحلة التالية.
```

---

# 10. Prompt المرحلة 05 — Application API والعقود

**الخبير:** Senior API/Application Architect

```text
أنت Senior Application Architect خبير FastAPI والعقود المستقرة. ابنِ Use Cases وAPI للميزات المنجزة فقط.

نفّذ:
- Commands/queries للشركات والسنوات والفترات والحسابات واليوميات والمساعدين والقيود وFolio.
- Authorization في كل use case.
- Pydantic contracts versioned وأخطاء Domain قابلة للفهم بالعربية/الفرنسية عبر error codes.
- OpenAPI واختبارات Contract وidempotency.
- pagination/filtering/sorting آمنة.
- localhost/IPC authentication وفق ADR.
- عدم كشف stack traces أو بيانات حساسة.

لا تنفذ OCR أو Liasse أو Cloud API.

بوابة القبول: كل endpoint مرتبط بمتطلب واختبار صلاحية ومعيار قبول؛ لا يمكن للAPI تجاوز Domain invariants.
```

## Prompt المراجعة

```text
أنت API Security وContract Reviewer مستقل. اختبر broken object-level authorization، mass assignment، injection، pagination abuse، replay/idempotency، error leakage، وcontract drift. أصدر تقريراً وقرار قبول.
```

---

# 11. Prompt المرحلة 06 — واجهة الإدخال Keyboard-First

**الخبير:** Principal UX Engineer للأنظمة المالية + Accessibility

```text
أنت Principal UX Engineer متخصص في شاشات المحاسبة كثيفة البيانات وKeyboard-first وRTL/LTR. نفّذ واجهة المرحلة 06 فوق العقود الموجودة فقط.

المطلوب:
1. Design tokens ومكونات ثنائية العربية/الفرنسية.
2. Company/Exercice selector وحالات الفترات.
3. Folio/Saisie virtualized grid.
4. Tab/Shift+Tab/Enter/Esc وF2 للحساب وF3 للمساعد وF9 للتوازن، مع خريطة اختصارات موثقة وقابلة للتغيير.
5. بحث حساب/مساعد سريع، أخطاء inline، منع فقد draft.
6. Autosave للـDraft فقط، ولا اعتماد ضمني.
7. رسائل واضحة للتعديل والعكس والفترة المغلقة.
8. اختبارات component/E2E وقياس الأداء وRTL/LTR.
9. لقطات أو فيديو قصير داخل PR للرحلات الحرجة.

لا تضف Dashboard تجميلياً ولا OCR الآن.

بوابة القبول:
- محاسب يستطيع إكمال سيناريو شراء دون ماوس تقريباً.
- لا keyboard traps، ولا تغيير مالي غير مؤكد.
- P95 البحث والحفظ ضمن الميزانية المعتمدة.
```

## Prompt المراجعة

```text
أنت مراجع UX محاسبي مستقل. راجع PR عملياً من منظور محاسب محترف ومستخدم لوحة مفاتيح. اختبر السرعة، focus، IME عربي/فرنسي، الأخطاء، فقدان draft، قارئ الشاشة الأساسي، و1000+ صف. لا تكتفِ بمراجعة الكود؛ وثّق سيناريوهات فعلية وقرار القبول.
```

---

# 12. Prompt المرحلة 07 — التقارير الأساسية ومحرك الصيغ

**الخبير:** Reporting Engine Architect + خبير SCF

```text
أنت مهندس محركات تقارير مالية وخبير SCF. نفّذ التقارير الأساسية المعتمدة فقط: Journal، Grand Livre، Balance، ونسخ أولية معتمدة من Bilan/Result إذا كانت قواعدها موثقة.

المطلوب:
1. Reporting read model performant.
2. Formula parser آمن: lexer/parser/AST/evaluator أو compiler محدود، لا eval ولا SQL خام من المستخدم.
3. Versioned report definitions.
4. Golden datasets وGolden outputs يوقعها خبير المجال.
5. PDF/Excel مع metadata وتطابق المجاميع.
6. Cache صحيح مع invalidation واضح.
7. اختبارات rounding، debit/credit، opening، closed periods، وحسابات prefix.

لا تنفذ G50/Liasse من الذاكرة. أي نموذج رسمي غير موثق يبقى Blocked.

بوابة القبول: كل رقم في التقرير قابل للتتبع إلى قيود المصدر وتعريف صيغة versioned.
```

## Prompt المراجعة

```text
أنت Financial Reporting Auditor ومراجع أمن Parser. أعد حساب العينات مستقلاً، اختبر formula injection وprefix ambiguities وcache stale وrounding وPDF/Excel totals. قارن Golden outputs ولا تقبل ادعاءات توافق ضريبي بلا مرجع.
```

---

# 13. Prompt المرحلة 08 — الهجرة والاستيراد والمطابقة

**الخبير:** Data Migration Architect + Forensic Data Engineer

```text
أنت Data Migration Architect خبير في الأنظمة المالية القديمة والتحليل الجنائي للبيانات. نفّذ مسارات Excel/CSV و.PCC وDLG فقط بقدر ما تسمح العينات المثبتة.

المبادئ:
- المصدر Read-only.
- نسخة bit-for-bit وhash قبل التحليل.
- Staging ثم validation ثم commit.
- كل تحويل mapping versioned.
- rerun idempotent.

نفّذ:
1. Profiler للترميز والحقول والسجلات التالفة.
2. Parsers صارمة وآمنة بحدود حجم.
3. Preview وdry-run وتقارير صفوف مرفوضة.
4. Reconciliation: counts، debit، credit، journal/account/aux balances، opening، unmatched.
5. Cutover وrollback runbook.
6. Fixtures مجهولة وتمثل ملفات صحيحة وتالفة.
7. واجهة اعتماد تقرير الفروق قبل commit النهائي.

لا تدّع دعم إصدار DLG لم تختبر عينة منه.

بوابة القبول: فرق 0.00 دج أو استثناءات مفصلة وموقعة، ولا كتابة على ملفات المصدر.
```

## Prompt المراجعة

```text
أنت Migration Auditor مستقل. حاول كسر parsers بترميزات مختلفة، أسطر ناقصة، مبالغ كبيرة، تواريخ غير صالحة، duplicates، interruption وإعادة التشغيل. تحقق مستقلاً من reconciliation وعدم تعديل المصدر. ارفض المرحلة إذا كان نجاح الاستيراد يمكن أن يخفي سجلات مرفوضة.
```

---

# 14. Prompt المرحلة 09 — OCR المحلي وواجهة المراجعة

**الخبير:** Principal Document AI/OCR Engineer + MLOps

```text
أنت Principal OCR/Document AI Engineer خبير بالعربية والفرنسية وONNX وقياس النماذج. نفّذ OCR المحلي فقط، مع بقاء المحاسب صاحب القرار.

اقرأ سياسة الخصوصية وADR وDataset docs. لا تستخدم بيانات حقيقية دون موافقة.

نفّذ Pipeline:
1. آمن لإدخال PDF/images/scanner، وفحص النوع والحجم.
2. quality score وdeskew/orientation/denoise/crop.
3. PaddleOCR/ONNX baseline مع bounding boxes.
4. استخراج vendor/NIF/RC/invoice number/date/HT/TVA/TTC/currency.
5. normalization وقواعد تحقق حسابي وتاريخ وقاموس موردين.
6. confidence على مستوى الحقل، مع calibration لا threshold ثابت عشوائي.
7. Validation UI: الصورة بجانب الحقل، source box، confidence، corrections.
8. اقتراح قيد فقط؛ ممنوع auto-posting.
9. حفظ model/version/config/hash وربط الوثيقة بالقيد.
10. Dataset versioned خارج Git للأصول الحساسة، وتقارير CER/WER/F1/exact match/slices.

لا تنفذ Cloud fallback أو fine-tuning تلقائي في هذه المرحلة.

بوابة القبول:
- Benchmark قابل للتكرار على test set منفصل.
- الادعاءات تطابق النتائج ولا تستخدم نسبة 99% Offline.
- كل قيد OCR يحتاج اعتماداً بشرياً واضحاً.
- فشل OCR لا يعطل المحاسبة.
```

## Prompt المراجعة

```text
أنت Independent OCR Benchmark Auditor وPrivacy Reviewer. تحقق من تسرب train/test، تنوع الموردين، calibration، slices، PII logs، malicious PDFs، resource exhaustion، وربط model version. راجع UI لمنع automation bias. أعد تشغيل عينة benchmark وأصدر قراراً.
```

---

# 15. Prompt المرحلة 10 — الأصول والمطابقة والافتتاحيات المتقدمة

**الخبير:** خبير محاسبة SCF + Financial Domain Engineer

```text
أنت خبير SCF ومهندس مجال مالي. نفّذ فقط المتطلبات التي اعتمدها خبير بشري للأصول والمطابقة وReport à-nouveaux.

يشمل النطاق حسب الوثائق المعتمدة:
- Lettrage يدوي ثم آلي بقواعد قابلة للشرح.
- رموز المطابقة وتقارير غير المسدد.
- نقل العناصر غير المطابقة مع التاريخ الأصلي وفق قاعدة معتمدة.
- سجل الأصول ومكونات الأصل وتاريخ الاقتناء/بدء الخدمة.
- خطة الاهتلاك الخطي أولاً؛ الطرق الأخرى فقط إذا وُثقت القواعد.

استخدم Golden cases، property tests، وتوليد قيود قابل للعكس والتتبع. لا تخترع مادة قانونية أو نسباً.
```

## Prompt المراجعة

```text
أنت Accounting Rules Auditor. أعد حساب حالات المطابقة والافتتاحيات والاهتلاك يدوياً من Golden cases، واختبر partial matching والتواريخ والبيع/الإخراج والتعديل بعد الإغلاق. ارفض أي قاعدة غير موثقة.
```

---

# 16. Prompt المرحلة 11 — الأمن والنسخ والاسترجاع والترخيص

**الخبير:** Security Architect + Windows Release Engineer

```text
أنت Security Architect وWindows Release Engineer. نفّذ hardening والتشغيل قبل Pilot.

المطلوب:
1. Threat model محدث وabuse cases.
2. Argon2id وMFA للأدوار العليا وسياسة جلسات.
3. OS Keychain وتدوير المفاتيح وعدم تضمين secrets.
4. تشفير النسخ، backup 3-2-1-1-0 حسب إمكانات المنتج.
5. Restore على جهاز نظيف مع تحقق hashes وبيانات التطبيق.
6. Exercice seal وsigned audit checkpoints مع توضيح حدود الحماية.
7. Signed Windows installer وsigned updates وrollback.
8. License signature دون fingerprint متطفل؛ graceful offline policy.
9. SAST/DAST/dependency/SBOM/secrets scans.
10. Runbooks للحوادث والنسخ والتحديث.

لا تستخدم MAC address منفرداً كهوية أو ترسل telemetry دون opt-in.

بوابة القبول: restore ناجح موثق، installer/update/rollback ناجح، ولا Critical/High غير مقبول رسمياً.
```

## Prompt المراجعة

```text
أنت Financial Application Red Team Lead. راجع المرحلة 11 وعدّل بيئة اختبار فقط. حاول tamper مع audit/seals، سرقة/استبدال backup، downgrade update، bypass license/RBAC، malicious restore، واستخراج secrets من binaries/logs. وثق الأدلة والإصلاحات ولا تنفذ هجوماً خارج المستودع وبيئة الاختبار.
```

---

# 17. Prompt المرحلة 12 — Pilot Readiness وإطلاق Beta محدود

**الخبير:** Release Manager + QA Lead + SRE

```text
أنت Release Manager وQA Lead وSRE لتطبيق مالي Desktop. لا تضف ميزات جديدة. جهّز Pilot محدوداً فقط.

نفّذ:
1. راجع Traceability: كل Must له اختبار وحالة.
2. Regression كامل وUAT scripts للمحاسب والرئيس والمدقق.
3. Performance على الجهاز الأدنى والموصى به.
4. اختبارات install/upgrade/rollback/backup/restore/migration.
5. Privacy-safe crash diagnostics opt-in.
6. Support runbook وknown issues وseverity/SLA داخلي.
7. Pilot plan لـ5 مكاتب، onboarding، training، exit criteria.
8. Release candidate موقّع وchecksums وrelease notes.
9. Freeze الميزات؛ عالج Blockers فقط.

معايير Go:
- لا Sev-1 مفتوح.
- لا فروق مالية غير مفسرة.
- restore وmigration ناجحان.
- خبير SCF وQA وSecurity يوقعون.

إذا لم تتحقق المعايير أصدر NO-GO بوضوح. لا تجامل ولا تدّع الجاهزية.
```

## Prompt المراجعة النهائية قبل Pilot

```text
أنت لجنة Go/No-Go مستقلة مكونة ذهنياً من: مدقق مالي، Principal Engineer، Security Lead، QA Lead، وProduct Risk Manager. استخدم GitHub MCP وافحص الأدلة الفعلية، لا التقارير فقط. أنشئ RELEASE-GATE-REVIEW.md بقرار GO أو CONDITIONAL GO أو NO-GO، مع شروط محددة ومالكيها ومواعيدها. لا تضف ميزات.
```

---

# 18. Prompt المرحلة 13 — Cloud OCR الاختياري وActive Learning

لا تستخدمه قبل نجاح Pilot المحلي وموافقة قانونية/خصوصية.

```text
أنت Privacy-Preserving ML Architect. صمم ونفذ Cloud OCR fallback كميزة منفصلة opt-in ومغلقة افتراضياً للعملاء الذين يمنعون السحابة.

المطلوب:
- Provider abstraction دون ربط المنتج بمورد واحد.
- موافقة واضحة لكل مؤسسة وسياسة على مستوى المستخدم.
- Data minimization/redaction حيث يمكن.
- DPA/retention/residency/config documentation.
- تكلفة وحدود وtimeouts/retries/circuit breaker.
- مقارنة field-level قبل/بعد، مع عدم استبدال تصحيح المستخدم تلقائياً.
- سجل موافقة وprovider/model/version دون حفظ أسرار.
- Active Learning pipeline منفصل: consent، anonymization، dataset version، approval، benchmark، model registry، rollback.

ممنوع إرسال أي وثيقة في الاختبارات إلى مزود حقيقي دون موافقة صريحة وبيانات اختبار مصطنعة.
```

---

# 19. Prompt المرحلة 14 — التقارير الجبائية وDGI

لا تبدأ إلا بعد جمع مراجع رسمية محدثة والتحقق القانوني.

```text
أنت Tax Software Architect تعمل مع خبير جباية جزائري. لا تعتمد على الذاكرة العامة أو مواد غير رسمية.

قبل الكود:
1. اجمع داخل المستودع المراجع الرسمية المصرح باستخدامها، مع التاريخ والإصدار.
2. أنشئ Legal/Tax Requirements Matrix يوقعها خبير بشري.
3. ميّز بين G50 وTVA وLiasse والتكامل الإلكتروني، ولا تفترض وجود API DGI.
4. صمم report definitions versioned وeffective dates وfeature flags.
5. ابنِ Golden cases لكل نموذج، وتصدير PDF/Excel حسب النموذج المعتمد.
6. إن لم تتوفر وثائق API رسمية، أنشئ adapter interface/mock فقط ولا تدّع التكامل.

بوابة القبول: كل خانة في كل نموذج مرتبطة بمرجع رسمي وصيغة واختبار وتوقيع خبير.
```

---

# 20. Prompt صيانة/إصلاح Bug في محادثة جديدة

```text
أنت Senior Maintenance Engineer لنظام مالي. استخدم GitHub MCP واقرأ AGENTS.md وHandoff وInvariants.

المشكلة: [ألصق وصف المشكلة ورابط Issue فقط]

نفّذ منهجياً:
1. أعد إنتاج المشكلة باختبار فاشل أولاً.
2. حدد root cause وblast radius.
3. أصلح أصغر مساحة ممكنة دون refactor غير ضروري.
4. أضف regression tests، وافحص invariants والهجرة والتقارير المتأثرة.
5. لا تغيّر عقد API أو schema دون ADR/migration.
6. حدّث Issue وChangelog وHandoff.
7. أنشئ PR مع خطوات reproduction وbefore/after وrollback.
```

---

# 21. Prompt مراجعة شاملة دورية للمستودع

استخدمه بعد كل 2–3 مراحل في محادثة جديدة.

```text
أنت Principal Engineer مستقل لم يشارك في التنفيذ. نفّذ Repository Health Audit شاملاً باستخدام GitHub MCP.

راجع:
- تطابق الكود مع Vision/Scope/ADRs.
- Traceability وتغطية Must requirements.
- dependency architecture والانحراف المعماري.
- TODO/FIXME/dead code والديون التقنية.
- جودة الاختبارات وحقيقة CI.
- security/dependencies/secrets/SBOM.
- الأداء والمهاجرات والنسخ والاسترجاع.
- تناقض Handoff مع الواقع.
- ميزات بُنيت قبل أوانها أو قواعد مالية بلا مرجع.

لا تبدأ refactor واسعاً. أنشئ AUDIT-[DATE].md وIssues مرتبة حسب الخطر والقيمة، مع قرار هل المشروع جاهز للمرحلة التالية.
```

---

# 22. Prompt اختياري لإنشاء المرحلة التالية تلقائياً دون تنفيذها

```text
أنت Technical Program Manager. لا تكتب كوداً. اقرأ GitHub بالكامل ونتيجة آخر مراجعة، ثم جهّز فقط NEXT_PHASE_INPUT.md وIssues للمرحلة التالية.

لكل Issue اكتب:
- الهدف والقيمة.
- In scope / Out of scope.
- التبعيات.
- الملفات/الوحدات المتوقعة.
- معايير القبول القابلة للاختبار.
- اختبارات الأمن والأداء ذات الصلة.
- Definition of Done.
- المخاطر والأسئلة المفتوحة.

لا تنفذ Issues ولا تغيّر المعمارية.
```

---

# 23. تسلسل الاستخدام الموصى به

لكل مرحلة اتبع الترتيب:

1. افتح محادثة جديدة والصق **Prompt التنفيذ**.
2. راجع خطة Agent الأولية قبل السماح له بتغييرات كبيرة إذا كانت منصتك تدعم الموافقة المرحلية.
3. اجعله ينشئ فرعاً وPR ولا يدمج.
4. افتح محادثة جديدة تماماً والصق **Prompt المراجعة** مع رابط PR.
5. أصلح Blockers في PR منفصل أو بواسطة Agent التنفيذ نفسه ضمن المحادثة المناسبة.
6. شغّل Prompt المراجعة مجدداً عند التغييرات الجوهرية.
7. ادمج فقط بعد APPROVE وبنجاح CI.
8. شغّل Prompt تجهيز المرحلة التالية إن كانت Issues غير كافية.
9. ابدأ المرحلة التالية في محادثة جديدة.

---

# 24. ما يحتاجه المستخدم إضافته في كل Prompt

استبدل أو أضف في أول الرسالة:

```text
Repository: [owner/repo]
Target branch: main
Phase issue/milestone: [الرابط]
Previous accepted review: [الرابط]
Human decisions already made: [القرارات فقط]
Constraints: [الوقت/الجهاز/اللغة/الميزانية]
```

لا تلصق تاريخ المشروع كاملاً. أعطِ الروابط، ودع الـAgent يقرأ المصدر من GitHub.

---

# 25. نصيحة نهائية

- استخدم **Agent قوياً للتنفيذ وAgent مختلفاً للمراجعة** إن أمكن.
- لا تطلب: «ابنِ المشروع كاملاً». اطلب مرحلة لها بوابة قبول.
- لا تساوِ بين كثرة الملفات والتقدم؛ التقدم هو اختبار مالي ناجح وميزة قابلة للاستعمال.
- لا تسمح للـAgent أن يقرر القواعد المحاسبية أو القانونية وحده.
- أول نقطة قيمة حقيقية ليست OCR، بل: **قيد صحيح + إدخال سريع + هجرة قابلة للمطابقة + استرجاع مضمون**.
- أفضل حماية من ضياع Context هي: ADR + Handoff + Traceability + اختبارات، وليس Prompt أطول.
