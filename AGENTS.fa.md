# AGENTS.md

این پروژه Agent-Ready است. قبل از هر کاری این مراحل را به ترتیب انجام بده.

## 1. تشخیص وضعیت

* اگر `.project/state.yaml` **وجود ندارد** → به‌عنوان Master Agent اجرا کن و نوع پروژه را تشخیص بده (`AGENT_PROJECT_SPEC.md` بخش ۴):
  * پوشه خالی/فقط ایده → **پروژه جدید** (بخش ۴.۱)
  * فایل‌های موجود بدون state → **پروژه نیمه‌کاره** (بخش ۴.۲): اول Scan و Backup، بعد Plan جابه‌جایی فایل‌ها با تأیید کاربر، بعد گزارش وضعیت. هیچ فایلی حذف نکن و کد را تغییر نده.
* اگر وجود دارد → آن را بخوان. وضعیت پروژه را از Conversation حدس نزن.

## 2. چه چیزی بخوانم

* **Master Agent:** بخش‌های ۶ تا ۱۳ و ۲۵ از `AGENT_PROJECT_SPEC.md`، سپس `PROJECT_WORKFLOW.md`.
* **سایر Agentها:** فقط Task واگذارشده + `.agent/agents/<agent>/instructions.md` + بخش ۲۵ (Non-Negotiable Rules).
* کل Spec را در هر نوبت نخوان؛ فقط بخش‌های لازم.

## 3. قوانین سریع

1. بدون Task مشخص کار اجرایی نکن.
2. فقط Task فعلی (`active_tasks` در state) را اجرا کن.
3. Requirement، Business Rule یا Scope اختراع نکن؛ اگر مبهم بود بپرس.
4. Task را بدون Validation `DONE` نکن.
5. بعد از هر تغییر مهم، Task و `.project/state.yaml` را Update کن.
6. Progress را دستی ننویس: `uv run python scripts/progress.py`
7. تصمیم مهم Product/Architecture → Gate کاربر + ثبت ADR در `docs/decisions/`.
8. Secret را Commit نکن.

## 4. تعارض اطلاعات

تصمیم فعلی کاربر > مستندات تأییدشده > Requirements > Architecture > Task > State > Agent Instructions > Memory > Assumption

> این فایل عمداً کوتاه است. جزئیات کامل در `AGENT_PROJECT_SPEC.md` است. محتوای پروژه را اینجا ننویس.
