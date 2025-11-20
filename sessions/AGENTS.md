### `sessions/AGENTS.md`

````md
# sessions/AGENTS.md

Purpose: sessions/ exists so LLMs can resume a task without replaying old chats.
Keep logs tiny and structured.

Contents:
- `INDEX.md` – flat table of sessions.
- `T-XXXX/*.md` – per-task session reports.

---

## 1. INDEX.md format

File: `sessions/INDEX.md`

```md
# Session Index

| session_id              | task_id(s)              | status    | executor_model | logger_model | tokens_used | path                                  |
|-------------------------|-------------------------|-----------|----------------|--------------|-------------|---------------------------------------|
````

Columns:

* `session_id` – `YYYYMMDDThhmm-<executor_model>`
* `task_id(s)` – one or more task ids, comma-separated (e.g. `T-0001,T-0002`)
* `status` – `completed` | `partial` | `failed` | `aborted`
* `executor_model`
* `logger_model`
* `tokens_used` – int or `-`
* `path` – `sessions/T-XXXX/<session_id>.md` (pick a primary T-XXXX)

Rules:

* Only **logger** models edit this file.
* One row per session report.

---

## 2. Session report format

Path: `sessions/T-XXXX/<session_id>.md`

* `T-XXXX` = primary task id.
* `session_id` = matches INDEX.

### Frontmatter (required)

```md
---
task_ids: [T-0001]          # or [T-0001, T-0002]
status: partial             # completed | partial | failed | aborted
executor_model: gpt-5.1
logger_model: gpt-5.1-mini
summary: "One-line summary, <= 200 chars."
---
```

Only these keys are required:

* `task_ids`
* `status`
* `executor_model`
* `logger_model`
* `summary`

You MAY add `tokens_used`, `branch`, `commits`, etc. if useful.

### Body (LLM-optimised)

```md
## actions
- What changed (files/areas), 1–5 bullets.

## decisions
- Non-obvious choices, 0–5 bullets.

## next
- How to resume, 1–5 bullets.
```

Constraints:

* Hard cap: **≤ 40 lines total** (frontmatter + body).
* No code dumps; refer to files/commits instead.
* If `status != completed`, `next` must say exactly what’s needed to continue.

---

## 3. Roles and lifecycle

**Executor (expensive models; do work)**

* Defined in `orchestration/models.yaml → roles.executor`.

* Reads `tasks/TASKS.md` row and latest session(s) for the task.

* Does the work.

* Near the end, emits a `[RAW LOG]` in the chat, e.g.:

  ```md
  [RAW LOG]
  tasks: T-0009,T-0010
  actions:
    - Directory.tsx: add container id, fix multiselect, badge classes.
    - globals.css: add padding on filter container.
  decisions:
    - Keep Unicorn ID-scoped CSS; match React to it.
  next:
    - Fix push 403 and push commit 441eb14.
    - Run mobile visual pass on filters/cards.
  ```

* Executors do **not** edit `sessions/*.md` or `INDEX.md`.

**Logger (cheap models; write logs)**

* Defined in `orchestration/models.yaml → roles.logger`.
* Takes the `[RAW LOG]` + minimal context.
* Creates `sessions/T-XXXX/<session_id>.md` matching §2.
* Appends a row to `sessions/INDEX.md`.
* Updates `tasks/TASKS.md` if status/difficulty/effort/model changed.
