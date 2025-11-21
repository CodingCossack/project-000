# tasks/ – Task Management Contract

This folder holds the **canonical task index** for the project.

- `TASKS.md` is the **single source of truth** for:
  - what exists
  - status
  - difficulty
  - priority
  - model assignment
  - links to GitHub issues and skills
- No per-task `.md` specs by default. If a task needs detailed discussion, use a GitHub issue or a design doc under `docs/`, and link it from `TASKS.md`.

All agents (Claude, GPT, Gemini, Codex, etc.) must respect this schema and lifecycle.

---

## 1. Table schema (TASKS.md)

`TASKS.md` is a Markdown table with these columns, in this exact order:

| Column       | Type    | Allowed values / format                                                                 |
|--------------|---------|------------------------------------------------------------------------------------------|
| `id`         | string  | `T-0001`, `T-0002`, … zero-padded, stable per task                                      |
| `title`      | string  | Short human-readable summary (≤ 80 chars)                                              |
| `status`     | enum    | `todo`, `in-progress`, `in-review`, `blocked`, `done`, `dropped`                        |
| `difficulty` | enum    | `easy`, `medium`, `hard`                                                                |
| `effort`     | enum    | `low`, `medium`, `high` (maps to model_reasoning_effort and output budget)             |
| `priority`   | enum    | `P0`, `P1`, `P2`, `P3` (P0 = urgent, P3 = nice-to-have)                                |
| `area`       | string  | Logical surface, e.g. `frontend`, `backend`, `pSEO`, `infra`, `tooling`, `docs`        |
| `model`      | string  | Name of the **current executor** model while active, empty when unassigned or reviewed |
| `issue`      | string  | GitHub issue reference like `#12` or `-` if none                                       |
| `skills`     | string  | Comma-separated skill names from `/skills` (or `-` if none)                            |
| `notes`      | string  | 1–2 very short notes (≤ 120 chars)                                                     |

Example header (no data rows shown here):

```md
# Task Backlog

| id     | title | status | difficulty | effort | priority | area | model | issue | skills | notes |
|--------|-------|--------|-----------|--------|----------|------|-------|-------|--------|-------|
````

Agents append rows below the header.

---

## 2. Status lifecycle

Allowed transitions:

* `todo → in-progress`
* `todo → dropped`
* `in-progress → in-review`
* `in-progress → blocked`
* `in-progress → dropped`
* `in-review → done`
* `in-review → blocked`
* `blocked → in-progress`
* `blocked → dropped`

### Semantics

- `todo` – defined but untouched.
- `in-progress` – branch / edits in flight; executor is actively changing code.
- `in-review` – implementation done and diff/PR exists, waiting for review + merge.
- `blocked` – cannot proceed (dependency, design decision, infra issue).
- `done` – merged into `main` and accepted.
- `dropped` – intentionally abandoned; kept for history.

### 1. Claiming a task

Only claim tasks where:

- `status = todo`
- `model` is empty

On claim:

- Set `status = in-progress`
- Set `model = <executor_model_name>`
- Optionally set `issue` if a GitHub issue is created.

### 2. Moving to review

When an executor has completed the implementation and opened a PR or produced a diff for review:

- Set `status = in-review`
- Keep `model = <executor_model_name>` until the PR is merged or explicitly handed off.
- Ensure `notes` includes a reference to the branch/PR or session id.

### 3. Finishing a task

When the changes are merged into `main` and accepted:

- Set `status = done`
- Clear `model` (executor is no longer holding it).
- Optionally update `notes` with merge info (PR number, date).

### 4. Blocked / dropped

- Use `blocked` when an external or unresolved dependency prevents progress.
  - Update `notes` with a short cause and pointer to session/issue.
- Use `dropped` only when the task is intentionally abandoned.

---

## 3. Difficulty and effort semantics

### Difficulty (`difficulty`)

* `easy` – small, well-scoped, mostly mechanical tasks. Low risk, minimal architectural impact.
* `medium` – moderate complexity, some design decisions, touches multiple files / components.
* `hard` – architecture, cross-cutting changes, or high uncertainty. Likely to need multiple sessions.

### Effort (`effort`)

* `low` – simple implementation, minimal reasoning. Suitable for cheap models and short outputs.
* `medium` – non-trivial reasoning, but not deep exploration. Mid-range models ok.
* `high` – heavy reasoning, planning, multiple options/constraints, or high blast radius.

**Mapping:** difficulty + effort drive model selection and reasoning effort. See `orchestration/models.yaml` for the matrix. After a failed or painful session:

* `difficulty` may be bumped up (e.g. `easy → medium`).
* `effort` may be bumped up (e.g. `medium → high`) to force stronger models / higher reasoning.

---

## 4. Model assignment and conflicts

- Each task can have **at most one active executor model** in `model` while `status ∈ {in-progress, in-review}`.
- Before touching a task, an agent must:
  - Re-read `TASKS.md`.
  - Only pick tasks where `status = todo` and `model` is empty.

Concurrency / conflicts:

- Avoid multiple `in-progress` tasks in the **same `area`** unless explicitly coordinated (via GitHub issue or human).
- Prefer to complete and merge one task in an `area` before starting another, especially for `backend-data`, `search-routing`, `seo-metadata`, `infra-*`.

Hand-off rules:

- If an executor cannot finish:
  - Set `status = blocked` (or leave as `in-progress`), update `notes` with why, and clear or change `model`.
- After review/merge:
  - Move `status = done` and clear `model`.

---

## 5. GitHub issues

* Use GitHub issues for tasks that require:

  * Discussion with humans.
  * Longer-term tracking.
  * Review comments.
* For such tasks:

  * Create an issue named `T-000X: <title>`.
  * Put its number (`#12`, etc.) into the `issue` column.
* `TASKS.md` stays the canonical state table. GitHub issues are commentary, review, and context.

---

## 6. Interaction with sessions

* Execution history for a task lives in `sessions/T-XXXX/*.md`.
* Session records:

  * Are written by **logger models** (cheap models).
  * Are indexed in `sessions/INDEX.md`.
* When a session ends, the logger may update `TASKS.md`:

  * Adjust `status` (`in-progress → done/blocked`).
  * Adjust `difficulty` / `effort` if mis-estimated.
  * Clear or change `model` if the task is handed off.

All agents manipulating `TASKS.md` or `sessions/INDEX.md` must keep the table format valid Markdown.
