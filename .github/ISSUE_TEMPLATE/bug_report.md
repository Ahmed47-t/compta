---
name: تقرير خطأ / مشكلة محاسبية أو تقنية (Bug Report)
about: الإبلاغ عن خطأ برمجي، أو عدم توازن محاسبي، أو تباطؤ في شبكة الإدخال
title: "[BUG / SEV-X]: وصف الخطأ الموجز"
labels: ["bug", "triage"]
assignees: []
---

### 1. تصنيف الخطأ والخطورة (`Severity & Classification`)
- **تصنيف الخطورة (`Severity`):**
  - [ ] `Sev-1 (Critical Block/Unbalanced Ledger/Data Loss)` — عيب حرج يكسر التوازن المحاسبي أو يسبب فقدان بيانات.
  - [ ] `Sev-2 (Major Performance Degradation / UI Crash)` — تباطؤ شديد أو انهيار في الشاشة.
  - [ ] `Sev-3 (Minor Bug / UX Glitch)` — خطأ بسيط أو تشوه في النص.
- **الوحدة أو المحرك المتأثر (`Engine/Module`):** [مثلاً `Core Engine`, `Saisie Grid`, `DLG Migrator`]

### 2. خطوات إعادة الإنتاج (`Steps to Reproduce`)
1. اذهب إلى الشاشة/الأمر '...'
2. أدخل البيانات '...'
3. اضغط على الزر '...'
4. شاهد الخطأ الذي حدث '...'

### 3. السلوك المتوقع مقابل السلوك الفعلي (`Expected vs Actual Behavior`)
- **السلوك المتوقع:** [ما الذي يجب أن يحدث وفق قواعد المجال والمحاسبة]
- **السلوك الفعلي (الخطأ):** [ما الذي حدث فعلياً]

### 4. سجلات الخطأ والأدلة (`Logs & Screenshots`)
```text
[ألصق هنا نصوص الخطأ من ruff / pytest / FastAPI logs دون تضمين أي أسرار أو بيانات حقيقية]
```
