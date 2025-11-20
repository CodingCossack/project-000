# AGENTS.md (root)

Purpose: coordinate multi-agent work with **minimal** instructions.  
Primary consumers: LLMs, not humans.

---

## 1. Global rules

- `tasks/TASKS.md` = **single source of truth** for work.
- `orchestration/models.yaml` = **single source of truth** for model/tier/reasoning rules.
- `sessions/` = **session history**, written **only by logger models**.
- Do **not** invent new formats. Follow contracts in:
  - `tasks/README.md` or `tasks/AGENTS.md`
  - `orchestration/policies.md`
  - `sessions/AGENTS.md`
- If unsure what to do: read the relevant sub-AGENTS/contract file, then act.

---

## 2. Directories

- `tasks/`  
  - Holds `TASKS.md` table (id, status, difficulty, effort, area, model, etc.).  
  - All planning, claiming, and status updates must go through `TASKS.md`.

- `orchestration/`  
  - `models.yaml`: roles, tiers (`cheap/strong/code`), selection rules.  
  - `policies.md`: how difficulty/effort map to model choice and reasoning.

- `sessions/`  
  - `INDEX.md` + `T-XXXX/<session_id>.md`.  
  - Format + size limits defined in `sessions/AGENTS.md`.

- `skills/`  
  - Each folder = one skill.  
  - Pick skills by folder name; if unclear, read **only** YAML frontmatter from that skill’s `SKILL.md`.

---

## 3. Roles

### planner

- Reads `tasks/TASKS.md`.
- Adds/refines tasks (id, title, difficulty, effort, priority, area, skills, notes).
- Uses **strong** tier models (via `orchestration/models.yaml`).
- Does **not** touch `sessions/`.

### executor

- Reads `tasks/TASKS.md`, chooses a `todo` task with empty `model`.
- Optionally reads latest `sessions/T-XXXX/*.md` for that task.
- Uses `orchestration/models.yaml` to pick model/tier/reasoning.  
  - Code-heavy → may choose `code` tier.  
  - Docs/architecture → `strong`/`cheap` tiers.
- Performs code/content changes.
- At end, emits a short `[RAW LOG]` for the logger (tasks, actions, decisions, next).

### logger

- Uses **cheap** tier models.
- Reads `[RAW LOG]` + minimal context.
- Writes `sessions/T-XXXX/<session_id>.md` and appends `sessions/INDEX.md`.
- Updates `tasks/TASKS.md` (status, difficulty, effort, model) if needed.
- Does **not** make substantial code changes.

---

## 4. Minimal lifecycle

- **Planner loop**
  1. Read `TASKS.md`.
  2. Add/update tasks (fields + difficulty/effort).
  3. Stop.

- **Executor loop**
  1. Read `TASKS.md` → pick one task (`todo` + empty `model`).
  2. Read latest session(s) for that task (optional).
  3. Do work, respecting `orchestration/models.yaml.session_limits`.
  4. Emit `[RAW LOG]` (tasks, actions, decisions, next).

- **Logger loop**
  1. Take `[RAW LOG]`.
  2. Create session file + index row (per `sessions/AGENTS.md`).
  3. Update `TASKS.md` if state changed.
