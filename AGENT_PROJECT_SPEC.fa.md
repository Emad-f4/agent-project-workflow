# Agent Project Specification

**Version:** 2.0

## 1. Purpose

این پروژه باید به‌صورت **Agent-Ready** ساخته شود؛ به‌گونه‌ای که یک ایده خام بتواند به یک پروژه قابل اجرا تبدیل شود و اجرای آن توسط چند Agent تخصصی، تحت مدیریت یک Master Agent، به‌صورت کنترل‌شده و قابل پیگیری انجام شود.

```text
Idea
  ↓
Project Workflow
  ↓
Phase
  ↓
Task
  ↓
Master Agent
  ↓
Specialized Agent
  ↓
Skills / Tools
  ↓
Execution
  ↓
Validation
  ↓
State Update
  ↓
Next Task / Next Phase
```

این فایل **معماری کلی Agent-Ready** را تعریف می‌کند. Workflow کل پروژه در `PROJECT_WORKFLOW.md` و قوانین هر Agent در پوشه همان Agent تعریف می‌شود.

---

# 2. Separation of Responsibilities

سه سطح باید از هم جدا باشند.

## 2.1 Project Specification

فایل `AGENT_PROJECT_SPEC.md` (همین فایل) تعریف می‌کند: ساختار پروژه، ساختار Agentها، Task System، State Management، Registry، Skill/Tool Resolution، Master Agent، قوانین عمومی و محیط اجرا.

## 2.2 Project Workflow

فایل `PROJECT_WORKFLOW.md` Lifecycle کل پروژه را تعریف می‌کند:

```text
IDEA → DISCOVERY → PRODUCT DEFINITION → MVP / SCOPE → PRD → REQUIREMENTS
→ UX / UI → ARCHITECTURE → TASK BREAKDOWN → IMPLEMENTATION → QA
→ DEPLOYMENT → OPERATION → ITERATION
```

این Workflow مربوط به **کل پروژه** است، نه یک Agent خاص.

## 2.3 Agent Instructions

هر Agent قوانین و Workflow اختصاصی خودش را دارد:

```text
.agent/agents/backend/
├── AGENT.md
├── instructions.md
└── memory/
    └── feedback.md
```

`instructions.md` باید شامل این بخش‌ها باشد:

```text
Role, Responsibilities, Rules, Workflow, Inputs, Outputs, Validation,
Required Skills, Required Tools, Stop Conditions, Escalation Rules
```

```text
Project Workflow ≠ Agent Workflow
```

---

# 3. Entry Point: AGENTS.md

`AGENTS.md` نقطه ورود همه Agentهاست (Claude Code، Codex، Cursor و...) و باید **کوتاه** بماند (حداکثر حدود ۳۰ خط).

وظیفه‌اش فقط این است که Agent را به ترتیب درست هدایت کند:

1. اگر `.project/state.yaml` وجود ندارد → برو به **Bootstrap** (بخش ۴).
2. در غیر این صورت `.project/state.yaml` را بخوان.
3. فقط بخش‌های لازم از `AGENT_PROJECT_SPEC.md` را بخوان (Master Agent: بخش‌های ۶ تا ۱۳ و ۲۵؛ سایر Agentها: `instructions.md` خودشان + بخش ۲۵).
4. Task فعلی را از `tasks/` بخوان و فقط همان را اجرا کن.

نسخه آماده این فایل کنار همین Spec قرار دارد. محتوای پروژه (Business Rule، Requirement و...) هرگز داخل `AGENTS.md` نوشته نمی‌شود.

---

# 4. Bootstrap / First Run

وقتی `.project/state.yaml` وجود ندارد، Master Agent **اول باید نوع پروژه را تشخیص دهد**:

| وضعیت | حالت | مسیر |
|-------|------|------|
| پوشه خالی است یا فقط ایده دارد | پروژه جدید | بخش ۴.۱ |
| فایل‌های موجود (کد، مستندات، Config، Git History و...) دارد ولی `state.yaml` ندارد | **پروژه نیمه‌کاره / موجود (Adoption)** | بخش ۴.۲ |
| `state.yaml` دارد | پروژه فعال | چرخه عادی (بخش ۱۰) |

اگر تشخیص مطمئن نبود، Master Agent از کاربر می‌پرسد.

## 4.1 پروژه جدید

Master Agent باید این مراحل را به ترتیب انجام دهد:

```text
1. ساخت ساختار پوشه‌ها مطابق بخش 5
2. ساخت .project/state.yaml با وضعیت bootstrap
3. اگر PROJECT_WORKFLOW.md وجود ندارد:
     - از Template استاندارد پیشنهاد بده
     - منتظر تأیید کاربر بمان (User Decision Gate)
4. پرسیدن ایده از کاربر (هدف، مخاطب، محدودیت‌ها) و ذخیره در docs/product/idea.md
5. تبدیل ایده به Goal و Scope (هدف، محدوده، خارج از محدوده) در docs/product/goal.md
6. GATE: ارائه Goal و Scope به کاربر و گرفتن تأیید صریح
7. ساخت PHASE-01 (Discovery) و .project/phase.yaml
8. تولید Taskهای اولیه PHASE-01 در tasks/PHASE-01/
9. ثبت Agentهای موجود در .agent/registry/agents.yaml
10. ساخت .project/context.md و Update کردن state.yaml و شروع اولین Task
```

قواعد Bootstrap:

* تا زمانی که کاربر ایده را ارائه نکرده، هیچ Task اجرایی ساخته نمی‌شود.
* تا زمانی که Goal و Scope توسط کاربر تأیید نشده، هیچ Phase یا Taskی ساخته نمی‌شود و هیچ Implementationی شروع نمی‌شود. ایده خام مستقیم به Task یا کد تبدیل نمی‌شود.
* Bootstrap فقط یک بار اجرا می‌شود؛ پس از آن `state.yaml` منبع حقیقت است.
* Docker و `uv` در Bootstrap الزامی نیستند (بخش ۲۴ را ببین).

نمونه `state.yaml` در حالت Bootstrap:

```yaml
project:
  name: unnamed
  status: bootstrap
current_phase: null
active_tasks: []
blocked: false
user_decision_required: true
pending_decision: "Provide project idea"
```

## 4.2 پروژه نیمه‌کاره / موجود (Adoption Workflow)

هدف: یک پروژه‌ای که از قبل وجود دارد را **بدون خراب کردن آن** به ساختار Agent-Ready ببریم، هر فایل را در جای درست خودش بگذاریم، و بعد **وضعیت واقعی پروژه** را گزارش کنیم.

```text
A. Scan (فقط خواندن)
 ↓
B. Safety (پشتیبان‌گیری)
 ↓
C. Classification + Migration Plan
 ↓
GATE 1: تأیید کاربر
 ↓
D. Execute Migration (جابه‌جایی فایل‌ها)
 ↓
E. Scaffold ساختار Agent-Ready
 ↓
F. Status Assessment (پروژه در چه وضعیتی است؟)
 ↓
GATE 2: تأیید کاربر روی گزارش وضعیت
 ↓
G. ساخت State، Phaseها و Taskهای باقی‌مانده
```

### A. Scan (فقط خواندن)

در این مرحله **هیچ فایلی تغییر نمی‌کند.** Agent باید فهرست کامل تهیه کند:

* ساختار پوشه‌ها و فایل‌ها، زبان‌ها و Stack، Dependency Managerها
* مستندات، README، نمودارها، یادداشت‌ها
* تست‌ها، Docker، CI، فایل‌های Config و `.env`
* Git History (Commitها، Branchها، TODO/FIXME در کد)
* هر Task، Issue یا برنامه‌ای که در فایل‌ها آمده باشد

خروجی: `.project/adoption/inventory.md`.

### B. Safety

قبل از هر جابه‌جایی:

* اگر پروژه Git دارد: تغییرات Commit نشده باید Commit یا Stash شوند و کار روی Branch جدا انجام شود (مثلاً `adoption/agent-ready`).
* اگر Git ندارد: از کاربر بخواه Backup بگیرد یا `git init` و Commit اولیه انجام شود. بدون Backup، مرحله D شروع نمی‌شود.
* وضعیت پایه ثبت شود: آیا Build/Test الان کار می‌کند؟ (نتیجه در `.project/adoption/baseline.md`.)

### C. Classification + Migration Plan

Agent برای **هر فایل** مقصدش را مشخص می‌کند و جدول زیر را در `.project/adoption/migration-plan.md` می‌نویسد:

| فایل فعلی | نوع | مقصد | اقدام | دلیل |
|-----------|-----|------|-------|------|
| `notes/idea.txt` | ایده / Product | `docs/product/idea.md` | move | ... |
| `src/` | کد | `apps/<name>/` | move (نیازمند تأیید) | ... |
| `Dockerfile` | زیرساخت | ریشه (بدون تغییر) | keep | ابزار انتظار دارد |
| `misc.zip` | نامشخص | `docs/_unsorted/` | ask user | ... |

قواعد:

* **هیچ فایلی حذف نمی‌شود.** فقط جابه‌جا می‌شود (با `git mv` تا History حفظ شود).
* فایل‌هایی که ابزارها انتظار دارند در ریشه باشند (`package.json`، `Dockerfile`، `pyproject.toml`، `.gitignore` و...) در جای خود می‌مانند.
* جابه‌جایی **کد** خطر شکستن Import و Build دارد؛ فقط با تأیید صریح کاربر و با اصلاح Referenceها انجام می‌شود. اگر جابه‌جایی پرریسک است، پیشنهاد اولیه `keep` است و کاربر تصمیم می‌گیرد.
* فایل‌های نامشخص حدس زده نمی‌شوند؛ در `docs/_unsorted/` قرار می‌گیرند یا از کاربر پرسیده می‌شود.
* فایل‌های Secret (مثل `.env` واقعی) هرگز جابه‌جا یا در گزارش‌ها کپی نمی‌شوند؛ فقط به وجودشان اشاره می‌شود و باید در `.gitignore` باشند.

**GATE 1:** Plan به کاربر نشان داده می‌شود و بدون تأیید او مرحله D شروع نمی‌شود.

### D. Execute Migration

* طبق Plan تأییدشده جابه‌جایی انجام می‌شود، ترجیحاً در چند Commit کوچک.
* Referenceهای شکسته (Import، مسیرها، لینک‌های مستندات، Config) اصلاح می‌شوند.
* بعد از جابه‌جایی، Build/Test دوباره اجرا و با `baseline.md` مقایسه می‌شود. اگر بدتر شد، اصلاح یا Rollback.
* هر اقدام در `.project/adoption/migration-log.md` ثبت می‌شود.
* **در این مرحله هیچ تغییر عملکردی در کد و هیچ ارتقای Dependency انجام نمی‌شود.**

### E. Scaffold

* ساختار Agent-Ready (بخش ۵) ساخته می‌شود: `.project/`، `.agent/`، `tasks/`، `docs/` و فایل‌های Spec/Workflow/AGENTS.
* **هیچ فایل موجودی Overwrite نمی‌شود.** اگر فایلی هم‌نام وجود دارد، Merge یا از کاربر پرسیده می‌شود.
* Agentهای لازم طبق بخش ۱۲.۱ (با پرسیدن از کاربر) ساخته می‌شوند.

### F. Status Assessment

Agent وضعیت را **فقط بر پایه شواهد** (مسیر فایل‌ها، Commitها، نتیجه Build/Test) بررسی می‌کند و گزارش `docs/project-status.md` را می‌نویسد. گزارش باید به این سؤال جواب دهد: **«الان پروژه در چه وضعیتی است؟»**

محتوای گزارش:

1. **خلاصه پروژه:** چیست، برای چه کسی، Stack چیست (با ذکر شاهد).
2. **وضعیت هر مرحله از `PROJECT_WORKFLOW.md`:** برای هر مرحله (Discovery، PRD، Architecture، Implementation، QA و...) یکی از `complete` / `partial` / `missing` / `not_applicable` همراه با فایل‌های شاهد.
3. **مرحله فعلی:** پروژه دقیقاً کجای Workflow است.
4. **کد:** چه بخش‌هایی پیاده‌سازی شده، چه بخش‌هایی نیمه‌کاره یا خراب است، Build/Test چه وضعیتی دارد.
5. **شکاف‌ها (Gaps):** مستندات، تست، Docker، `uv` و... که وجود ندارند.
6. **انحراف از Spec:** مثلاً استفاده از `pip`، نبود Docker، ساختار پوشه‌ای متفاوت.
7. **ریسک‌ها و Blockerها.**
8. **سؤال‌های باز برای کاربر.**

قواعد حیاتی:

* **Requirement و Business Rule اختراع نمی‌شود.** هر سندی که از روی کد یا فایل‌ها استخراج شود (Reverse-engineered) با برچسب `status: inferred` و فهرست شواهد ذخیره می‌شود و تا تأیید کاربر «Approved Documentation» به‌حساب نمی‌آید (بخش ۲۰).
* برآورد درصد پیشرفت فقط با ذکر مبنا داده می‌شود؛ اگر مبنای معتبر نیست، «نامشخص» نوشته شود.
* هر جا شاهد کافی نیست، مورد در «سؤال‌های باز» می‌آید، نه حدس.

**GATE 2:** کاربر گزارش را بررسی و اصلاح می‌کند (مثلاً «این بخش در واقع تمام نشده است» یا «این Stack اشتباه است»). بدون تأیید او مرحله G شروع نمی‌شود.

### G. State, Phase و Task

بعد از تأیید گزارش:

* `phase.yaml` و `state.yaml` طبق **مرحله فعلی تأییدشده** ساخته می‌شوند. مرحله‌های `complete` با وضعیت `adopted` علامت می‌خورند (Task تاریخی برای آن‌ها ساخته نمی‌شود، چون کار قبلاً انجام شده است).
* Task فقط برای **کار باقی‌مانده** و **شکاف‌ها** ساخته می‌شود، مثلاً:
  * تکمیل مستندات و تأیید Docs استخراج‌شده (`inferred` → `approved`)
  * مهاجرت Dependency به `uv` (اگر Python است) یا Manager تأییدشده
  * ساخت Dockerfile و Docker Compose
  * نوشتن تست‌های ناقص
  * کارهای نیمه‌کاره کد
* Taskهای مربوط به Docker و `uv` طبق بخش ۲۴ باید قبل از ادامه Implementation انجام شوند.
* `context.md` ساخته و چرخه عادی (بخش ۱۰) شروع می‌شود.

وضعیت `state.yaml` در حین Adoption:

```yaml
project:
  name: existing-project
  status: adoption
adoption:
  step: F        # A تا G
  gate_pending: GATE-2
current_phase: null
active_tasks: []
user_decision_required: true
pending_decision: "Review project-status.md"
```

### قواعد کلی Adoption

* Adoption یک **Phase موقت** است؛ در آن کدنویسی یا تغییر عملکرد ممنوع است.
* هر مرحله پیش از Gateها فقط خواندن یا برنامه‌ریزی است.
* اگر Agent در هر مرحله چیزی را نفهمید (فایل ناشناس، Stack مبهم)، می‌پرسد؛ حدس نمی‌زند.
* مهاجرت‌های بزرگ (تغییر Stack، بازنویسی، ارتقای Dependency) داخل Adoption انجام نمی‌شود و Task جدا با Gate می‌شود.

---

# 5. Project Structure

```text
project/
│
├── AGENT_PROJECT_SPEC.md
├── AGENTS.md
├── PROJECT_WORKFLOW.md
│
├── .project/
│   ├── state.yaml
│   ├── phase.yaml
│   ├── context.md
│   └── adoption/          # فقط برای پروژه‌های موجود (بخش 4.2)
│       ├── inventory.md
│       ├── baseline.md
│       ├── migration-plan.md
│       └── migration-log.md
│
├── tasks/
│   ├── PHASE-01/
│   │   ├── TASK-001.yaml
│   │   └── ...
│   └── PHASE-02/
│       └── ...
│
├── docs/
│   ├── product/
│   ├── requirements/
│   ├── design/
│   ├── architecture/
│   ├── database/
│   ├── api/
│   ├── testing/
│   ├── deployment/
│   └── decisions/        # ADRها
│
├── .agent/
│   ├── agents/
│   │   ├── master/
│   │   ├── product/
│   │   ├── ux/
│   │   ├── backend/
│   │   ├── frontend/
│   │   ├── database/
│   │   ├── qa/
│   │   └── devops/
│   ├── skills/
│   ├── tools/
│   ├── registry/
│   │   └── agents.yaml
│   └── factory/
│
├── apps/
├── infrastructure/
├── scripts/
│   └── progress.py       # محاسبه Progress از روی Taskها
│
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
├── uv.lock
├── .env.example
└── .dockerignore
```

فایل‌های Docker و `pyproject.toml` از فاز Implementation به بعد الزامی هستند (بخش ۲۴).

---

# 6. Project State

دایرکتوری `.project/` وضعیت فعلی پروژه را نگه می‌دارد و باید به این سؤال پاسخ دهد: **«الان پروژه کجاست؟»** Agent نباید این اطلاعات را از Conversation یا حافظه خود حدس بزند.

## 6.1 state.yaml

شامل: Phase فعلی، Taskهای فعال، Agentهای فعال، وضعیت پروژه، Blocker، و نیاز به تصمیم کاربر.

**Progress در `state.yaml` ذخیره نمی‌شود.** Progress همیشه از روی Taskها محاسبه می‌شود (بخش ۱۹)، تا تناقض ایجاد نشود.

```yaml
project:
  name: example-project
  status: active

current_phase:
  id: PHASE-01
  name: Product Definition

active_tasks:
  - task: TASK-010
    agent: product-agent

blocked: false
user_decision_required: false
pending_decision: null
```

## 6.2 context.md

`.project/context.md` خلاصه کوتاه و فعلی پروژه برای شروع سریع هر Agent است (حداکثر یک صفحه). فقط **اشاره** و خلاصه است، نه منبع اصلی دانش.

محتوا:

```text
- Goal پروژه (یک پاراگراف، با لینک به docs/product/goal.md)
- Scope فعلی و Out-of-scope (خلاصه، با لینک)
- Phase فعلی و هدف آن
- تصمیم‌های کلیدی اخیر (لینک به ADRها)
- Stack و Dependency Managerهای تأییدشده
- سؤال‌ها یا ریسک‌های باز
```

قوانین:

* Master Agent آن را در Bootstrap می‌سازد و در شروع/پایان هر Phase و بعد از هر تغییر Scope یا Architecture Update می‌کند.
* در تعارض با `docs/` یا `state.yaml`، آن‌ها معتبرند و `context.md` باید اصلاح شود.
* Requirement یا Business Rule جدید فقط در `docs/` نوشته می‌شود، نه اینجا.

## 6.3 Concurrency Rule

* **پیش‌فرض:** در هر لحظه فقط **یک Task فعال** وجود دارد (`active_tasks` حداکثر یک عضو دارد).
* کار موازی فقط وقتی مجاز است که کاربر صریحاً اجازه دهد و Taskها Dependency و خروجی مشترک نداشته باشند.
* در حالت موازی، هر Task در `active_tasks` جداگانه ثبت می‌شود و هیچ دو Taskی روی یک فایل خروجی کار نمی‌کنند.

---

# 7. Phase State

فایل `.project/phase.yaml` وضعیت Phase فعلی را نگه می‌دارد.

```yaml
id: PHASE-01
name: Product Definition
status: in_progress

objective: >
  تبدیل ایده اولیه به هدف، محدوده و قابلیت‌های مشخص.

required_artifacts:
  - docs/product/problem.md
  - docs/product/users.md
  - docs/product/mvp.md
  - docs/product/prd.md

exit_criteria:
  - product_goal_defined
  - target_users_defined
  - mvp_defined
  - required_tasks_completed
  - user_review_completed
```

## 7.1 Phase Completion

Phase فقط وقتی کامل است که **همه** شرایط زیر هم‌زمان برقرار باشند:

```text
Required Tasks Completed   (همه Taskهای الزامی DONE؛ CANCELLED/SKIPPED مجاز)
        +
Required Artifacts Exist   (همه فایل‌های required_artifacts موجودند)
        +
Validation Passed          (validation_result همه Taskها passed=true)
        +
User Decisions Resolved    (pending_decision خالی و user_decision_required=false)
        +
Exit Criteria Satisfied    (تک‌تک موارد exit_criteria تحقق یافته)
```

بنابراین:

```text
Task Done ≠ Phase Done
Phase Done ≠ Project Done
```

تکمیل همه Taskها به‌تنهایی Phase را کامل نمی‌کند. Master Agent باید هر پنج شرط را جداگانه بررسی و نتیجه را به کاربر گزارش کند، سپس Gate بخش ۱۶ را اجرا کند.

---

# 8. Task System

Task واحد اصلی اجرای پروژه است. Taskها در `tasks/` و بر اساس Phase سازمان‌دهی می‌شوند.

Taskها بر اساس Status جابه‌جا **نمی‌شوند**؛ Status داخل خود Task ذخیره می‌شود.

## 8.1 Task Definition

```yaml
id: TASK-010
title: Define MVP Features
phase: PHASE-01
status: ready
priority: high
agent: product-agent

dependencies:
  - TASK-007
  - TASK-008

required_skills:
  - product-discovery
  - requirements-analysis
required_tools: []

inputs:
  - docs/product/problem.md
  - docs/product/users.md

outputs:
  - docs/product/mvp.md

acceptance_criteria:
  - MVP features are explicitly defined
  - Scope is documented
  - Out-of-scope functionality is identified

# فیلدهای زیر هنگام اجرا پر می‌شوند
started_at: null
completed_at: null

blocked_by: []          # شناسه Taskها یا موارد مسدودکننده
blocked_reason: null

validation_result:
  passed: null          # true / false
  checked_by: null
  checked_at: null
  details: null

notes: []
```

## 8.2 Task Lifecycle

```text
BACKLOG → READY → IN_PROGRESS → REVIEW → DONE
```

حالت‌های جانبی:

```text
IN_PROGRESS → BLOCKED → (رفع مشکل) → IN_PROGRESS
هر وضعیتی → CANCELLED   (Task دیگر لازم نیست؛ دلیل در notes ثبت شود)
BACKLOG/READY → SKIPPED (عمداً انجام نمی‌شود؛ نیاز به تأیید Master Agent)
```

* Task فقط وقتی `DONE` می‌شود که Acceptance Criteria آن Validate شده و `validation_result.passed = true` باشد.
* Taskهای `CANCELLED` و `SKIPPED` در محاسبه Progress جدا شمرده می‌شوند و مانع تکمیل Phase نیستند.
* وقتی Task به `BLOCKED` می‌رود، `blocked_reason` اجباری است.

---

# 9. Change Management

وقتی Requirement، Scope یا Architecture تغییر می‌کند:

1. Master Agent تغییر را از کاربر تأیید می‌گیرد (User Decision Gate).
2. تصمیم در `docs/decisions/` به‌صورت ADR ثبت می‌شود (`ADR-NNN-عنوان.md` شامل: Context، Decision، Consequences، تاریخ).
3. Taskهای `DONE` که تحت‌تأثیر قرار می‌گیرند شناسایی می‌شوند و:
   * اگر فقط نیاز به بازبینی دارند → به `REVIEW` برگردند.
   * اگر باید دوباره انجام شوند → به `READY` برگردند.
   * علت برگشت در `notes` همان Task ثبت شود.
4. اگر Phase قبلاً تکمیل شده بود و تغییر روی آن اثر دارد، وضعیتش به `in_progress` برمی‌گردد و Gate مجدد لازم است.
5. Docs مرتبط Update و `state.yaml` بازنویسی می‌شود.

---

# 10. Master Agent

Master Agent مسئول Orchestration پروژه است. در هر چرخه باید:

1. State را بخواند (اگر وجود نداشت → Bootstrap).
2. Phase و Task فعلی را تشخیص دهد.
3. Dependencyها را بررسی کند.
4. Agent مناسب را از Registry انتخاب کند. **اگر Agent مناسبی وجود نداشت:** Task را `BLOCKED` کند (`blocked_reason: agent_missing`)، `user_decision_required: true` بگذارد و فرآیند ساخت Agent (بخش ۱۲.۱) را با پرسیدن از کاربر شروع کند. Agent را خودش حدس نزند و Task را به Agent نامناسب ندهد.
5. Skill و Tool موردنیاز را Resolve کند.
6. Task را به Agent واگذار کند.
7. نتیجه را Validate کند.
8. Task State و Project State را Update کند.
9. Task بعدی را مشخص کند.
10. Exit Criteria Phase را بررسی و در صورت تکمیل، Gate بعدی را اجرا کند.

Master Agent نباید بدون Task مشخص وارد Implementation شود.

---

# 11. Agent Registry

Agentها در `.agent/registry/agents.yaml` ثبت می‌شوند:

```yaml
agents:
  backend-agent:
    path: .agent/agents/backend
    capabilities: [python, fastapi, api, backend]

  frontend-agent:
    path: .agent/agents/frontend
    capabilities: [frontend, ui]

  qa-agent:
    path: .agent/agents/qa
    capabilities: [testing, validation]
```

Master Agent بر اساس Capability و نیاز Task، Agent را انتخاب می‌کند.

---

# 12. Agent Definition

هر Agent حداقل این ساختار را دارد:

```text
.agent/agents/<agent>/
├── AGENT.md
├── instructions.md
└── memory/
    └── feedback.md
```

* **AGENT.md:** شناسنامه Agent (ID، Name، Version، Purpose، Capabilities، Path، Status).
* **instructions.md:** قوانین و Workflow اختصاصی. باید قبل از اجرای هر Task توسط Agent خوانده شود.

## 12.1 Creating a New Agent

**Trigger:** هر زمان که Agent موردنیاز یک Task در Registry وجود نداشته باشد (در Task Breakdown، هنگام واگذاری Task، یا هنگام Bootstrap)، ساخت Agent جدید اجباری و پرسیدن از کاربر الزامی است. Master Agent آن را حدس نمی‌زند و Task را به Agent نامرتبط نمی‌دهد.

`.agent/factory/` باید حداقل شامل این فایل‌ها باشد:

```text
.agent/factory/
├── questions.md            # فهرست سؤال‌هایی که باید از کاربر پرسیده شود
├── instructions.template.md
└── AGENT.template.md
```

Factory برای Agent جدید این موارد را مشخص و در `instructions.md` ثبت می‌کند:

```text
Role, Responsibilities, Rules, Workflow, Inputs, Outputs, Validation,
Required Skills, Required Tools, Stop Conditions, Escalation Rules
```

**این اطلاعات را Agent خودش حدس نمی‌زند؛ باید از کاربر بپرسد.** فرآیند:

1. Master Agent/Factory برای هر یک از موارد بالا از کاربر سؤال می‌کند (Role چیست؟ چه مسئولیت‌هایی دارد؟ چه قوانینی؟ Workflow چیست؟ و...). سؤال‌ها به‌صورت گروهی و کوتاه پرسیده می‌شوند.
2. اگر کاربر مورد را نمی‌داند، Factory می‌تواند **پیشنهاد** بدهد، ولی آن پیشنهاد فقط با تأیید صریح کاربر معتبر می‌شود.
3. پاسخ‌های تأییدشده در `.agent/agents/<agent>/instructions.md` ثبت می‌شود و `AGENT.md` ساخته می‌شود.
4. Agent در `.agent/registry/agents.yaml` ثبت می‌شود.
5. تا قبل از تأیید نهایی کاربر، Agent جدید Task دریافت نمی‌کند و Taskهای منتظر آن `BLOCKED` می‌مانند.
6. پس از تأیید و ثبت در Registry، Taskهای مسدودشده به `READY` برمی‌گردند (`blocked_reason` پاک می‌شود) و Master Agent چرخه را ادامه می‌دهد.

Agent نباید Workflow و Rules اختصاصی خودش را بدون تعریف و تأیید اختراع کند.

---

# 13. Skill and Tool Resolution

Skillها و Toolها با یک ترتیب یکسان Resolve می‌شوند:

```text
Agent-level  (.agent/agents/<agent>/skills/  یا  tools/)
      ↓
Global       (.agent/skills/  یا  tools/)
      ↓
Unavailable
```

* Skill و Tool عمومی نباید برای Agentهای مختلف Duplicate شوند.
* Agent نباید بدون بررسی Global، اعلام کند Skill یا Tool وجود ندارد.
* اگر Unavailable بود، Agent مطابق Escalation Rules به Master Agent یا کاربر ارجاع می‌دهد.

---

# 14. Documentation

Documentation اصلی پروژه در `docs/` قرار دارد. Project Knowledge نباید داخل Agent Memory ذخیره شود. Memory فقط برای Feedback و اطلاعات پایدار درباره نحوه همکاری با Agent است.

---

# 15. State vs Task vs Documentation

```text
.project/  →  Where are we?
tasks/     →  What must be done?
docs/      →  What do we know?
```

مثال: `.project/state.yaml` می‌گوید Task فعال `TASK-010` است؛ `tasks/PHASE-01/TASK-010.yaml` تعریف کامل Task را دارد؛ و `docs/product/mvp.md` محتوای Product را نگه می‌دارد.

---

# 16. User Decision Gates

در تصمیمات مهم Agent نباید تصمیم نهایی را خودکار بگیرد:

```text
Phase Completed → User Review → User Feedback → Apply Changes
→ Validation → Phase Complete → Next Phase
```

تا زمانی که Gate تکمیل نشده، Phase بعدی شروع نمی‌شود. در زمان انتظار، `user_decision_required: true` و `pending_decision` در state ثبت می‌شود.

## 16.1 Feedback Handling

هر Feedback یا درخواست تغییر از کاربر در Gate (یا هر زمان دیگر) **مستقیم و بدون Task اجرا نمی‌شود**:

1. Master Agent Feedback را در `notes` یا `docs/decisions/` ثبت می‌کند.
2. برای آن یک **Task اصلاحی** در Phase جاری ساخته می‌شود (مثلاً `TASK-025`، با `title` که با `Fix:` یا `Revise:` شروع شود و فیلد `notes` شامل ارجاع به Feedback اصلی).
3. Task اصلاحی مثل هر Task دیگر Agent مناسب، Acceptance Criteria و Validation دارد.
4. Taskهای DONE متأثر مطابق بخش ۹ به `REVIEW`/`READY` برمی‌گردند.
5. بعد از Validation، نتیجه دوباره به کاربر ارائه می‌شود و فقط با تأیید او Phase نهایی می‌شود.

اگر Feedback Scope یا Requirement را عوض کند، بخش ۹ (Change Management) هم اعمال می‌شود.

---

# 17. Validation

هر Task باید Validation متناسب با نوع خودش داشته باشد.

```text
Execute → Validate → Pass? ── YES → DONE
                          └── NO  → Fix / BLOCKED
```

نتیجه Validation در `validation_result` همان Task ثبت می‌شود.

---

# 18. State Updates

State و Task باید بعد از این رویدادها Update شوند: شروع Task، پایان Task، Block شدن، تغییر Agent، User Feedback، User Decision، تغییر Requirement/Scope/Architecture، تکمیل Phase، شروع Phase جدید.

---

# 19. Progress Tracking

Progress **هرگز دستی نوشته نمی‌شود**؛ فقط از روی فایل‌های Task محاسبه می‌شود:

```bash
uv run python scripts/progress.py            # Phase فعلی
uv run python scripts/progress.py PHASE-02   # Phase مشخص
```

خروجی نمونه:

```text
Phase: PHASE-01 — Product Definition
Total: 20 | Done: 9 | In Progress: 1 | Review: 2 | Ready: 6 | Blocked: 2
Cancelled/Skipped: 0
Active: TASK-010 (product-agent)
```

در پاسخ به «الان کجای پروژه هستیم؟» Master Agent باید اطلاعات واقعی State و خروجی این اسکریپت را ارائه دهد، نه حدس یا خاطره Conversation.

---

# 20. Source of Truth

در تعارض اطلاعات، ترتیب اعتبار:

```text
Current User Decision
  ↓ Approved Documentation
  ↓ Requirements
  ↓ Architecture
  ↓ Task Specification
  ↓ Project State
  ↓ Agent Instructions
  ↓ Agent Memory
  ↓ Assumption
```

Agent نباید بر اساس Assumption، اطلاعات معتبرتر را نقض کند.

---

# 21. Environment Configuration

* Configuration از Environment Variables یا Secret Management خوانده می‌شود.
* `.env.example` باید در Repository باشد.
* `.env` واقعی و Secretها هرگز Commit نمی‌شوند.

---

# 22. Dependency Management

هر Dependency Manager باید مشخص، ثابت و دارای Lock File باشد:

| Stack | Manager | Lock File |
|-------|---------|-----------|
| Python | `uv` | `uv.lock` (+ `pyproject.toml`) |
| Node / Frontend | `pnpm` (مگر اینکه کاربر Manager دیگری تعیین کند) | `pnpm-lock.yaml` |

قوانین:

* برای Python، `pip install` به‌عنوان Dependency Manager اصلی مجاز نیست (چه روی Host، چه داخل Docker).
* Manager هر Stack در `docs/architecture/` ثبت می‌شود و Agent نباید بدون تأیید Manager دیگری وارد کند.
* نمونه Python: `uv add fastapi`، `uv add --dev pytest`، `uv sync`.

---

# 23. Docker

محیط رسمی و قابل تکرار پروژه Docker-based است.

حداقل فایل‌ها: `Dockerfile`، `docker-compose.yml`، `.dockerignore`.

```bash
docker compose up -d --build    # اجرا
docker compose down             # توقف
docker compose logs -f          # مشاهده Log
```

* تمام Serviceهای موردنیاز پروژه در Docker Compose تعریف می‌شوند.
* Dependencyها داخل Docker هم با Manager تعیین‌شده نصب می‌شوند (برای Python: `uv sync` از روی `uv.lock`).
* اجرای رسمی نباید به نصب دستی Dependency روی Host وابسته باشد.
* استفاده شخصی Developer از Virtual Environment ممنوع نیست، اما محیط رسمی باید Docker باشد.

---

# 24. When Docker and Dependency Rules Apply

* در فازهای **Discovery تا Task Breakdown** (بدون کد) الزامات Docker، `uv` و `pyproject.toml` اعمال **نمی‌شود**.
* از **شروع فاز Implementation** این الزامات اجباری است، و ساخت آن‌ها باید خودش یک Task مشخص (معمولاً برای devops-agent) در ابتدای فاز Implementation باشد.
* Master Agent نباید Task کدنویسی را شروع کند مگر اینکه محیط Docker پروژه آماده شده باشد.

---

# 25. Non-Negotiable Rules

Agentها نباید:

* بدون هدف مشخص یا بدون Task مشخص، کار اجرایی را شروع کنند.
* Requirement یا Business Rule اختراع کنند.
* Scope را بدون مجوز تغییر دهند.
* Phase را بدون Exit Criteria تغییر دهند.
* Task را بدون Validation `DONE` کنند.
* State را نادیده بگیرند یا Progress را دستی و بدون محاسبه بنویسند.
* Project Knowledge را در Memory ذخیره کنند.
* Global Skill/Tool را بی‌دلیل Duplicate کنند یا قبل از بررسی Global، اعلام کنند وجود ندارد.
* Dependencyها را خارج از Manager تعیین‌شده مدیریت کنند.
* اجرای رسمی پروژه را به Environment دستی Host وابسته کنند (از فاز Implementation).
* Secret را داخل Repository قرار دهند.
* بدون نیاز واقعی Complexity ایجاد کنند.
* تصمیمات مهم Product یا Architecture را بدون Gate و بدون ثبت ADR اتخاذ کنند.

---

# 26. Core Execution Model

```text
USER → MASTER AGENT → PROJECT STATE → PROJECT WORKFLOW → CURRENT PHASE
→ CURRENT TASK → AGENT SELECTION → AGENT INSTRUCTIONS → SKILL RESOLUTION
→ TOOL RESOLUTION → EXECUTION → VALIDATION → TASK UPDATE
→ PROJECT STATE UPDATE → NEXT TASK → PHASE GATE → NEXT PHASE
```

اگر `state.yaml` وجود نداشته باشد، مسیر با **Bootstrap** (بخش ۴) شروع می‌شود.

---

# 27. Final Principle

> **Project Workflow مسیر کل پروژه را مشخص می‌کند؛ Task مشخص می‌کند الان چه کاری باید انجام شود؛ Master Agent مشخص می‌کند چه Agentی آن را انجام دهد؛ Agent Instructions مشخص می‌کند آن Agent چگونه کار کند؛ Skills و Tools قابلیت اجرا را فراهم می‌کنند؛ و Project State مشخص می‌کند دقیقاً در چه نقطه‌ای هستیم.**

```text
Project Workflow   = مسیر کل پروژه
Task               = واحد اجرایی
Master Agent       = Orchestrator
Agent Instructions = Rules + Workflow اختصاصی Agent
Skills / Tools     = قابلیت‌های قابل استفاده
Project State      = وضعیت لحظه‌ای پروژه
```

این تفکیک باید در تمام پروژه حفظ شود.
