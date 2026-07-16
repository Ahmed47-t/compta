# دليل التطوير والتشغيل السريع للمطورين والوكلاء (DEVELOPMENT.md)

**المشروع:** PC COMPTA Next (`Ahmed47-t/compta`)  
**الهدف:** توفير دليل عملي ومباشر (`One-Command Bootstrap`) للمطورين ووكلاء الذكاء الاصطناعي لإعداد بيئة التطوير محلياً وتشغيل كافة الفحوصات والاختبارات المعتمدة.

---

## 1. المتطلبات المسبقة للبناء (`Prerequisites`)

يجب التأكد من تثبيت الأدوات التالية على نظام التشغيل (Windows / Linux / macOS):
- **Git** & **GitHub CLI (`gh`)**
- **Python 3.11+** مع أداة إدارة الحزم `poetry` أو `pip/venv`.
- **Node.js 20+** مع `npm` أو `pnpm`.
- **Rust Toolchain (1.75+)** مع `cargo` (لبناء واجهة Tauri).
- **PostgreSQL 16+** المحلي أو عبر حاوية Docker (`docker-compose`).

---

## 2. إعداد البيئة السريع (`One-Command Bootstrap`)

من جذر المستودع، قم بتشغيل أوامر الإعداد التالية:

```bash
# 1. إعداد المتغيرات وبيئة قاعدة البيانات الاختبارية عبر Docker (إن توفر Docker)
docker run --name pccompta-pg-test -e POSTGRES_PASSWORD=secret -e POSTGRES_DB=pccompta_test -p 5432:5432 -d postgres:16

# 2. تثبيت اعتماديات الباك اند Python (FastAPI Engine)
cd services/accounting_api
python -m venv .venv
source .venv/bin/activate  # على Windows: .venv\Scripts\activate
pip install -r requirements.txt || poetry install

# 3. تثبيت اعتماديات الواجهة الأمامية (Tauri + React UI)
cd ../../apps/ui
npm install

# 4. تثبيت وفحص بيئة Rust Tauri
cd ../desktop
cargo check
```

---

## 3. تشغيل أدوات النظافة والفحص الآلي (`Lint / Type-Check / Formatting`)

قبل إجراء أي Commit، يجب التأكد من نجاح جميع الأوامر التالية دون أخطاء:

### 3.1 فحص الباك اند والمجال المحاسبي (Python / Domain):
```bash
cd services/accounting_api
ruff check .           # فحص الأخطاء البرمجية ونظافة الكود
ruff format --check .  # فحص التنسيق
mypy .                 # فحص الأنواع الصارم (Strict Type Checking)
```

### 3.2 فحص واجهة المستخدم (TypeScript / React):
```bash
cd apps/ui
npm run lint           # تشغيل ESLint
npx tsc --noEmit       # فحص التوافق ونوعية TypeScript
```

### 3.3 فحص نواة سطح المكتب (Rust / Tauri):
```bash
cd apps/desktop
cargo clippy -- -D warnings  # فحص أخطاء وتحسينات Rust
cargo fmt -- --check         # فحص تنسيق Rust
```

---

## 4. تشغيل الاختبارات الآلية (`Running Tests`)

### 4.1 تشغيل اختبارات المجال والنواة المحاسبية (Python / Domain):
```bash
cd services/accounting_api
pytest -v               # تشغيل كل الاختبارات Unit & Integration
pytest tests/domain/    # تشغيل اختبارات المجال المحاسبي المستقلة
pytest --cov=engines    # التحقق من أن التغطية لا تقل عن 95%
```

### 4.2 تشغيل اختبارات الواجهة (Frontend):
```bash
cd apps/ui
npm test                # تشغيل Vitest / Jest Component Tests
```

---

## 5. قواعد ارتكاب التغييرات وإدارة الفروع (`Branching & Commit Guidelines`)

1. **الفرع الإلزامي:** اعمل دائماً على الفرع المخصص للمرحلة أو جلسة العمل (مثل `arena/019f69c9-compta`). يحظر الدفع المباشر (`Push`) إلى `main`.
2. **الـ Commits الصغيرة:** اجعل كل Commit معبّراً ومرتبطاً بتغيير محدد (`Conventional Commits`):
   - `feat(core): enforce double-entry DB check trigger`
   - `fix(saisie): correct month validation for Folio 02`
   - `docs(handoff): update phase 00 deliverables`
3. **الـ Pull Request:** لا تقم بدمج الـ PR الخاص بك بنفسك؛ اتركه للمراجع المستقل بعد التحقق من كل الاختبارات.

---

## 6. إدارة دورة حياة جلسات الذكاء الاصطناعي (`AI Session Lifecycle Management`)

1. **الوعي بحدود المحادثة:** كل محادثة مع AI Agent مخصصة لإنجاز مرحلة واحدة فقط أو مراجعة واحدة فقط. بمجرد إتمام المهمة وفتح الـ Pull Request، تنتهي مهمة الـ Agent في تلك الجلسة.
2. **بروتوكول التسليم وإشعار المحادثة الجديدة:** يجب على الـ Agent عند اكتمال عمله الالتزام بالقسم رقم 6 من `AGENTS.md` عبر إرسال صندوق إشعار مرئي في رده الأخير (`🛑 إشعار إتمام المهمة وبدء محادثة جديدة`) مرفقاً بالبرومبت الجاهز للنسخ للمرحلة التالية.
3. **منع تراكم السياق (`No Context Bloat`):** لا يُسمح بطلب تنفيذ مرحلة جديدة في نفس المحادثة؛ الانتقال للمرحلة التالية يتم حصرياً في محادثة جديدة تستمد ذاكرتها من ملفات `docs/HANDOFF/` في GitHub.
