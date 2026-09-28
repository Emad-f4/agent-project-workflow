# Project Workflow

این فایل **Lifecycle کل پروژه** را تعریف می‌کند؛ نه قوانین یک Agent خاص و نه معماری سیستم (آن‌ها در `AGENT_PROJECT_SPEC.md` و `instructions.md` هر Agent هستند).

> این فایل یک **Template پیشنهادی** است. در Bootstrap باید توسط کاربر تأیید یا اصلاح شود. Phaseهای غیرضروری (مثلاً UX برای پروژه API-only) با تأیید کاربر حذف یا Skip می‌شوند و دلیل در `docs/decisions/` ثبت می‌شود.

## 1. مسیر کلی

```text
IDEA
 ↓
DISCOVERY
 ↓
PRODUCT DEFINITION
 ↓
MVP / SCOPE
 ↓
PRD
 ↓
REQUIREMENTS
 ↓
UX / UI            (فقط پروژه‌های دارای رابط کاربری)
 ↓
ARCHITECTURE
 ↓
TASK BREAKDOWN
 ↓
IMPLEMENTATION
 ↓
QA
 ↓
DEPLOYMENT
 ↓
OPERATION
 ↓
ITERATION
```

قانون کلی: هر مرحله یک یا چند Phase است، هر Phase با Gate کاربر بسته می‌شود (Spec بخش‌های ۷.۱ و ۱۶)، و هیچ مرحله‌ای بدون تکمیل قبلی شروع نمی‌شود.

---

## 2. مراحل

### 2.1 IDEA → Goal

* **هدف:** تبدیل ایده خام به Goal و Scope مشخص، قبل از هر Phase یا Task.
* **ورودی:** ایده کاربر.
* **خروجی:** `docs/product/idea.md`، `docs/product/goal.md`.
* **Gate:** تأیید صریح Goal و Scope توسط کاربر.
* **مسئول:** Master Agent + product-agent.

### 2.2 DISCOVERY

* **هدف:** فهم مسئله، کاربران، بازار/رقبا و محدودیت‌ها.
* **خروجی:** `docs/product/problem.md`، `docs/product/users.md`، فهرست فرضیه‌ها و ریسک‌ها.
* **Exit:** مسئله و کاربر هدف مشخص است؛ فرضیه‌های حیاتی مشخص شده‌اند.

### 2.3 PRODUCT DEFINITION

* **هدف:** تعریف محصول، ارزش پیشنهادی و قابلیت‌های اصلی.
* **خروجی:** `docs/product/product-definition.md`.
* **Exit:** کاربر محصول را تأیید کرده است.

### 2.4 MVP / SCOPE

* **هدف:** مشخص کردن حداقل محصول قابل ارائه و آنچه خارج از محدوده است.
* **خروجی:** `docs/product/mvp.md` (شامل Out-of-scope).
* **Exit:** Scope تأیید شده؛ تغییر بعدی فقط از مسیر Change Management (Spec بخش ۹).

### 2.5 PRD

* **هدف:** سند کامل نیازمندی‌های محصول.
* **خروجی:** `docs/product/prd.md`.
* **Exit:** PRD توسط کاربر تأیید شده است.

### 2.6 REQUIREMENTS

* **هدف:** تبدیل PRD به Requirementهای قابل‌آزمون (Functional / Non-functional) و Business Ruleها.
* **خروجی:** `docs/requirements/`.
* **Exit:** هر Requirement شناسه و Acceptance Criteria دارد؛ ابهام باز نمانده یا در `pending_decision` ثبت شده است.

### 2.7 UX / UI  (شرطی)

* **کی اعمال شود:** فقط اگر پروژه رابط کاربری یا تعامل کاربر دارد.
* **الزامی نیست برای:** API-only، Backend-only، Infrastructure.
* **زیرمراحل:**

```text
User Flow
 ↓
Wireframe
 ↓
Prototype
 ↓
UI Design
 ↓
UX Validation
```

* **خروجی:** `docs/design/` (Flowها، Wireframeها، Prototype، Design Tokens).
* **Exit:** کاربر Prototype و UI را تأیید کرده و UX Validation انجام شده است.

### 2.8 ARCHITECTURE

* **هدف:** تعیین Stack، ساختار سیستم، دیتابیس، API و Dependency Managerها.
* **خروجی:** `docs/architecture/`، `docs/database/`، `docs/api/`، ADRهای تصمیم‌های کلیدی.
* **Exit:** معماری تأیید شده؛ Stack و Manager هر زبان ثبت شده است.

### 2.9 TASK BREAKDOWN

* **هدف:** شکستن Requirement و Architecture به Taskهای قابل اجرا برای Phaseهای Implementation.
* **خروجی:** Taskهای کامل در `tasks/PHASE-XX/` (Agent، Dependency، Skill/Tool، Inputs/Outputs، Acceptance Criteria).
* **Exit:** هر Requirement حداقل یک Task دارد؛ Dependencyها بدون چرخه‌اند؛ کاربر ترتیب را تأیید کرده است.
* **بررسی Agent:** Agent هر Task باید در Registry موجود باشد. اگر نبود، همین‌جا (نه در زمان اجرا) Agent جدید طبق Spec بخش ۱۲.۱ با پرسیدن از کاربر ساخته و تأیید می‌شود.

### 2.10 IMPLEMENTATION

* **پیش‌شرط:** محیط Docker/`uv` آماده باشد (Spec بخش ۲۴). اولین Task این Phase باید ساخت آن باشد.
* **هدف:** پیاده‌سازی Taskها به ترتیب Dependency.
* **خروجی:** کد در `apps/`، زیرساخت در `infrastructure/`، تست‌ها.
* **Exit:** همه Taskهای الزامی DONE و Validate شده‌اند؛ پروژه با `docker compose up -d --build` اجرا می‌شود.

### 2.11 QA

* **هدف:** تأیید تطابق با Requirementها و Acceptance Criteria.
* **خروجی:** `docs/testing/` (Test Plan، نتایج، باگ‌های باز).
* **Exit:** باگ‌های بحرانی صفر؛ کاربر گزارش QA را تأیید کرده است.
* باگ‌ها Task اصلاحی می‌شوند (Spec بخش ۱۶.۱).

### 2.12 DEPLOYMENT

* **هدف:** استقرار در محیط هدف.
* **خروجی:** `docs/deployment/`، Pipeline/اسکریپت‌های استقرار، Runbook.
* **Exit:** استقرار موفق و Smoke Test پاس شده؛ تأیید کاربر.

### 2.13 OPERATION

* **هدف:** پایش، پشتیبانی و رفع مشکلات عملیاتی.
* **خروجی:** گزارش‌های عملیاتی، Runbook به‌روز.

### 2.14 ITERATION

* **هدف:** جمع‌آوری بازخورد و شروع چرخه بعدی.
* **رفتار:** بازخورد جدید یا Requirement تازه وارد Change Management می‌شود و در صورت نیاز، Phase جدید (از Discovery یا Requirements) ساخته می‌شود.

---

## 3. قوانین عبور بین Phaseها

Phase بعدی فقط وقتی شروع می‌شود که هر پنج شرط بخش ۷.۱ Spec برقرار باشد:

```text
Required Tasks Completed
+ Required Artifacts Exist
+ Validation Passed
+ User Decisions Resolved
+ Exit Criteria Satisfied
```

سپس:

```text
Phase Completed → User Review → Feedback → Corrective Tasks
→ Validation → Phase Finalized → Next Phase
```

* پرش از مرحله (مثلاً رفتن از PRD مستقیم به Implementation) ممنوع است، مگر با تصمیم صریح کاربر و ثبت ADR.
* بازگشت به Phase قبلی در صورت تغییر Scope/Requirement مجاز است و طبق Spec بخش ۹ انجام می‌شود.

---

## 4. نگاشت مرحله به Agent (پیش‌فرض)

| مرحله | Agent اصلی | مشارکت‌کننده |
|-------|-----------|--------------|
| Idea / Goal | master | product |
| Discovery، Product، MVP، PRD | product | master |
| Requirements | product | qa |
| UX / UI | ux | frontend |
| Architecture | backend | database، devops |
| Task Breakdown | master | همه Agentهای تخصصی |
| Implementation | backend، frontend، database، devops | qa |
| QA | qa | — |
| Deployment / Operation | devops | backend |
| Iteration | master | product |

نگاشت نهایی در Task تعیین می‌شود؛ این جدول فقط پیش‌فرض است.

---

## 5. Workflow پروژه‌های موجود / نیمه‌کاره (Adoption)

وقتی پروژه از قبل فایل و کار دارد ولی Agent-Ready نیست، به‌جای شروع از IDEA، این مسیر اجرا می‌شود (جزئیات کامل: `AGENT_PROJECT_SPEC.md` بخش ۴.۲):

```text
SCAN → SAFETY (Backup/Branch) → CLASSIFY + MIGRATION PLAN
   → GATE 1 (تأیید کاربر)
   → MIGRATE (هر فایل در جای درست)
   → SCAFFOLD (ساختار Agent-Ready)
   → STATUS ASSESSMENT (پروژه در چه وضعیتی است؟)
   → GATE 2 (تأیید کاربر روی گزارش)
   → ورود به مرحله فعلی Workflow اصلی
```

### نگاشت فایل‌ها (پیش‌فرض)

| نوع فایل موجود | مقصد پیشنهادی |
|----------------|----------------|
| ایده، یادداشت، مسئله، کاربران، MVP، PRD | `docs/product/` |
| Requirement و Business Rule | `docs/requirements/` |
| طراحی، Wireframe، Prototype | `docs/design/` |
| معماری، Diagram، تصمیم‌ها | `docs/architecture/`، `docs/decisions/` |
| Schema، Migration، ERD | `docs/database/` |
| مستندات API | `docs/api/` |
| برنامه تست و گزارش‌ها | `docs/testing/` |
| مستندات استقرار | `docs/deployment/` |
| کد برنامه | `apps/` (فقط با تأیید کاربر) |
| Terraform، K8s، CI/CD | `infrastructure/` |
| اسکریپت‌ها | `scripts/` |
| فایل‌های Config که ابزار در ریشه انتظار دارد | ریشه (بدون تغییر) |
| نامشخص | `docs/_unsorted/` یا پرسیدن از کاربر |

### تعیین مرحله فعلی

بعد از Scan، هر مرحله از مسیر اصلی (بخش ۱) با شاهد ارزیابی می‌شود:

```text
complete | partial | missing | not_applicable
```

* مرحله فعلی = اولین مرحله‌ای که `complete` نیست.
* مراحل `complete` با وضعیت `adopted` علامت می‌خورند و Task تاریخی ندارند.
* مراحل `partial` و `missing` Task می‌گیرند (مثلاً نوشتن PRD ناقص، ساخت Docker، مهاجرت به `uv`).
* مستنداتی که از کد استخراج شده‌اند `inferred` هستند و تا تأیید کاربر معتبر نیستند.

### مثال

```text
پروژه: کد FastAPI نیمه‌کاره + یادداشت‌های پراکنده + بدون Docker
Discovery / Product / MVP  → partial  (فقط یادداشت‌ها؛ نیاز به تأیید)
Requirements / Architecture → missing
Implementation              → partial
QA / Deployment             → missing
مرحله فعلی: MVP / SCOPE  (اول Scope تأیید شود، بعد ادامه)
```

### تفاوت‌ها با پروژه جدید

* نیازی به شروع از IDEA نیست؛ ولی Goal و Scope باید توسط کاربر تأیید شوند.
* تا Gate 2 هیچ کدنویسی یا تغییر عملکردی انجام نمی‌شود.
* الزامات Docker و `uv` به Task تبدیل می‌شوند و قبل از ادامه Implementation انجام می‌شوند.

---

## 6. شخصی‌سازی برای پروژه

در Bootstrap، Master Agent از کاربر می‌پرسد:

1. آیا پروژه رابط کاربری دارد؟ (اگر نه، مرحله UX/UI حذف می‌شود.)
2. Stack تقریبی چیست؟ (Dependency Managerها را مشخص می‌کند.)
3. کدام مراحل ساده‌تر یا ادغام شوند؟ (مثلاً PRD و Requirements در پروژه کوچک.)

نتیجه در `docs/decisions/` ثبت و Phaseهای واقعی پروژه در `tasks/` ساخته می‌شود.
