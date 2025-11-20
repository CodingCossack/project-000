```md
# sessions/AGENTS.md

Contract for using `sessions/`.

Contents:
- `INDEX.md` – flat index of all sessions.
- `T-XXXX/*.md` – per-task session reports.

Keep this short. All detail lives in the reports, not here.

---

## 1. INDEX.md

File: `sessions/INDEX.md`

```
# Session Index

| session_id         | task_id | status    | executor_model | logger_model | tokens_used | path                               |
|--------------------|---------|-----------|---------------|--------------|-------------|------------------------------------|
```

Columns:
- session_id = YYYYMMDDThhmm-<executor_model>
- task_id = T-XXXX from tasks/TASKS.md
- status = completed | partial | failed | aborted
- executor_model
- logger_model
- tokens_used = int or -
- path = sessions/T-XXXX/<session_id>.md

Rules:
- Only logger models append/modify rows.
- One row per session file. Keep header intact.

---

## 2. Session reports

Path: sessions/T-XXXX/<session_id>.md

Naming:
- T-XXXX = task id (e.g. T-0001)
- session_id = YYYYMMDDThhmm-<executor_model> (e.g. 20251118T1900-gpt-5.1)

### 2.1 Frontmatter

Top of file:
```
***
task_id: T-0001
task_title: Short title from TASKS.md
status: partial        # completed | partial | failed | aborted
executor_model: gpt-5.1
logger_model: gpt-5.1-mini
tokens_used: 78000     # or "-"
summary: <= 250 chars, one line, no markdown.
***
```

Mandatory keys:
- task_id
- task_title
- status
- executor_model
- logger_model
- tokens_used
- summary (hard cap 250 chars)

### 2.2 Body

Structure:

```
## Scope

- ...

## Actions

- ...

## Decisions

- ...

## Notes

- ...

## Next steps

- ...
```

Rules:
- Max ~300 lines total (frontmatter + body).
- For completed + small: keep 1–3 bullets per section.
- For partial/failed: spell out what was tried, what broke, what to do next.
- No code dumps. Link to files/dirs instead of pasting big blobs.

---

## 3. Roles

Executor models (expensive):
- Defined in orchestration/models.yaml → roles.executor.
- Do work. Do not write sessions/*.md or edit INDEX.md.
- At session end, produce a short raw log (bullets) for logger.

Logger models (cheap):
- Defined in orchestration/models.yaml → roles.logger.
- Write session report file + update INDEX.md.
- May bump difficulty/effort/status/model in tasks/TASKS.md when appropriate.
- Do not make large code changes; summarise only.

---

## 4. Session lifecycle (minimal)

Executor:
- Reads TASKS.md row + latest session(s) for task_id.
- Works within token cap (see orchestration/models.yaml.session_limits).
- Emits raw log near end.

Logger:
- Creates sessions/T-XXXX/<session_id>.md with schema above.
- Appends row to INDEX.md.
- Adjusts TASKS.md if status/difficulty/effort/model changed.
- Done.
```
