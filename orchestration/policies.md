<!-- orchestration/policies.md -->

# Orchestration Policies (Minimal)

This repo’s orchestration is defined by:

- `tasks/TASKS.md` – single source of truth for tasks (schema already documented there).
- `orchestration/models.yaml` – roles, tiers, and selection rules.
- `sessions/` – session logs (schema in `sessions/AGENTS.md`).

Agents should treat this file as the *short* contract for behaviour.

---

## 1. Task lifecycle (summary)

From `tasks/TASKS.md`:

- Claim a task only if:
  - `status = todo` **and**
  - `model` is empty.
- On claim:
  - Set `status = in-progress`.
  - Set `model = <executor_model_name>`.
- On finish:
  - Set `status = done`.
  - Clear `model`.
- Use `blocked` only for genuine external dependency / unresolved issues.
- Use `dropped` only when task is intentionally abandoned.

Only one executor should hold a task (`status = in-progress` + non-empty `model`) at any time.

---

## 2. Roles

### Planner

- Role: define / refine tasks in `TASKS.md`, set `difficulty`, `effort`, `priority`, `area`, `skills`.
- Models:
  - Use **strong** tier only (see `roles.planner.allowed_tiers`).
- Defaults:
  - `reasoning_effort = high`.
  - `max_output_tokens = 4096`.

### Executor

- Role: do the actual work (code / content / infra changes).
- Models:
  - Use `roles.executor.allowed_tiers` and rules in §3.
- On each session:
  - Claim/update the task row.
  - Do the changes.
  - Leave short raw notes for the logger (what changed, what’s left).

Executors do **not** write formal session records.

### Logger

- Role: write session summaries and update tables.
- Models:
  - Cheap tier only.
- Behaviour:
  - Keep outputs short.
  - Update `TASKS.md` (status, difficulty, effort, model) based on what happened.
  - Write/update `sessions/T-XXXX/*.md` and `sessions/INDEX.md`.

---

## 3. Model and tier selection

Inputs: one row from `TASKS.md`:

- `difficulty ∈ {easy, medium, hard}`
- `effort ∈ {low, medium, high}`
- `area` (e.g. `frontend`, `backend`, `pSEO`, `docs`, `infra`, `tooling`)
- `notes` (free text)
- `skills` (optional)

### 3.1 Baseline logic (all tasks)

For **executors**:

1. **Tier**
   - Start from `selection.difficulty_to_tier` in `models.yaml`:
     - `easy → cheap`
     - `medium → strong`
     - `hard → strong`

2. **Reasoning**
   - `reasoning_effort = selection.effort_to_reasoning[effort]`.

3. **Max output tokens**
   - Start from `defaults.max_output_tokens.executor` (3072).
   - You can reduce it if the task is obviously small; don’t increase it unless the task is clearly large.

For **planners** and **loggers**, just use the `defaults` for their role.

### 3.2 When to prefer Codex (code tier) vs strong tier

Codex (tier `code`) is for **heavy code editing / generation**, not high-level design/docs.

Executors should **prefer `code` tier** when ALL of the following are true:

1. `area` is clearly code-related, e.g. `frontend`, `backend`, `infra`, `tooling`.
2. The work is primarily *editing or generating code*, not writing ADRs / design docs.
3. `notes` or `skills` indicate code work (any of: `refactor`, `migrate`, `rewrite`, `implement`, `code review`, or skills like “TypeScript & Modern JavaScript Core”, “Next.js & React”, etc.).

In that case:

- Use `code` tier if available.
- If `code` tier fails/unavailable, fall back to `strong`.

Executors should **prefer `strong` tier** (not `code`) when:

- The main output is:
  - architecture / design discussion,
  - ADRs,
  - pSEO / content,
  - non-trivial reasoning about behaviour or trade-offs.
- Or `area` is `docs`, `pSEO`, or similar.

### 3.3 Code review

We don’t define a separate reviewer role.

A task is treated as **code review** if:

- `notes` explicitly say `code review` / `review PR` / similar, or
- The linked GitHub issue clearly asks for review.

In that case, executors should:

- Prefer `code` tier (Codex) if available, otherwise `strong`.
- Use at least `medium` reasoning effort (upgrade from `low` if needed).
- Keep `max_output_tokens` modest (≈ 1500–2000) per call.

No automatic review for every coding task; it’s opt-in via `notes` / issue.

---

## 4. Sessions and context cap

- Combined context per session (prompt + history + output) should stay under `session_limits.max_context_tokens` (80k).
- Before the session grows large:
  - Stop starting new subtasks.
  - Stabilise the code (or clearly describe any broken state).
  - Leave clear raw notes for the logger.
- Every non-trivial executor session should be followed by:
  - A logger session that:
    - Writes/updates the session record.
    - Updates the task row in `TASKS.md`.

---

## 5. Human overrides

Humans can override anything:

- Tier / specific model.
- `difficulty`, `effort`, `priority`, `area`.
- Whether a task is treated as “code-heavy” or as “code review”.

When doing so, they should:

- Edit the row in `TASKS.md` and/or
- Add a short explanation in `notes` or the GitHub issue.
