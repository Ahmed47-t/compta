# مواصفات المرحلة 05: Application API والعقود (PHASE-05.md)

**المرحلة:** Phase 05  
**الخبير المطلوب:** Senior API/Application Architect  
**الهدف الأساسي:** بناء واجهة برمجة التطبيقات المحلية (`FastAPI Modular Monolith API`) وعقود البيانات المستقرة الموثقة (`Pydantic v2 Contracts & OpenAPI Spec`) لربط واجهة سطح المكتب بمحركات المجال وقاعدة البيانات.

---

## 1. نطاق العمل المطلوب (Scope of Work)

1. **حالات الاستخدام والأوامر (`Use Cases / CQRS Commands & Queries`):** تطبيق الأوامر والاستعلامات الخاصة بالشركات، والسنوات المالية، والفترات، ودليل الحسابات، واليوميات، والحسابات المساعدة، والقيود، ونظام الفوليو.
2. **التحكم بالصلاحيات على مستوى الاستدعاء (`Authorization per Use Case`):** التحقق في كل نقطة اتصال (`Endpoint`) من صلاحية الدور والدخول وفق `RBAC/RLS`.
3. **عقود Pydantic ورسائل الخطأ المزدوجة (`Versioned Contracts & bilingual Errors`):** تعريف النماذج بـ `Pydantic v2` مع رموز أخطاء واضحة للعميل بالعربية والفرنسية (`Error Codes: e.g., ERR_UNBALANCED_FOLIO`).
4. **التوثيق والاختبار الآلي (`OpenAPI & Contract Tests`):** توليد ملف `OpenAPI Specification`، وكتابة اختبارات التحقق من العقود، والتحقق من التصفح والفرز والتقسيم الآمن (`Pagination/Filtering`).
5. **المصادقة المحلية وأمان الاتصال (`Local IPC/HTTP Auth`):** تطبيق بروتوكول المصادقة المحلية المعتمد في `ADR` لضمان أن الاتصال بـ `localhost/IPC` محمي ومقيد بتطبيقنا المكتبي فقط.
6. **حماية أخطاء النظام (`Error Leakage Prevention`):** التأكد من عدم كشف `stack traces` أو تفاصيل بنية قاعدة البيانات للعميل في استجابات الخطأ.

---

## 2. بوابة القبول (Acceptance Gate)

- كل `endpoint` مرتبط بمتطلب موثق في `TRACEABILITY_MATRIX.md` واختبار صلاحية ومعيار قبول واضح.
- استحالة تجاوز قيود المجال أو إدراج قيد غير متوازن عبر واجهة الـ API.
- نجاح اختبارات الأمان والتصدي لهجمات (`Broken Object-Level Authorization, Mass Assignment, Replay/Idempotency`).
