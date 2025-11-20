````markdown
# Multi-Agent Starter – Gemini + Claude + Cloudflare (No Local Setup)

This project lets you build a web app using:

- **Gemini 3.0 (AI Studio)** for fast UI building.
- **Claude Code** (and optionally ChatGPT Codex) for deeper code changes.
- **Cloudflare Pages** for hosting and automatic deployments.
- A simple file layout so multiple AI “agents” don’t trip over each other.

You do **not** need to install anything locally to start.

---

## 0. Accounts you need

Make sure you have:

1. **GitHub** account  
2. **Cloudflare** account  
3. **Google** account with **Gemini 3 / AI Studio** access  
4. **Claude** account (Pro or higher if possible)

Once those exist, follow the steps below in order.

---

## 1. Create your own repo from this template

You are starting from this public template repo:

> `CodingCossack/project-000`

1. Open this URL in your browser:  
   `https://github.com/CodingCossack/project-000`
2. At the top right, click the green button **“Use this template”**.
3. Choose **“Create a new repository”**.
4. Fill the form:
   - **Owner**: your GitHub account
   - **Repository name**: e.g. `my-ai-project`
   - **Description**: anything
   - **Privacy**: choose **Private**
5. Click **“Create repository from template”**.

Now you have your **own copy** under your account.

### 1.1 Check the folder structure

On your new repo’s main page you should see:

```text
AGENTS.md

orchestration/
  models.yaml
  policies.md

sessions/
  INDEX.md
  AGENTS.md

tasks/
  TASKS.md
  AGENTS.md

skills/
  (empty for now)
````

Don’t worry what these files say yet. The AIs will use them.

---

## 2. Connect your repo to Cloudflare Pages (hosting)

Goal: every time `main` branch changes, Cloudflare builds and hosts your site.

1. Log in to **Cloudflare** in your browser.
2. Top navigation bar: click **“Workers & Pages”**.
3. Left sidebar: click **“Pages”**.
4. Click the button **“Create application”**.
5. Under the **Pages** section, click **“Connect to Git”**.
6. If Cloudflare asks to connect GitHub:

   * Click **“Connect GitHub account”**.
   * Authorise GitHub when prompted.
7. In the list of repos, find and select **your new repo** (e.g. `my-ai-project`).
8. Click **“Begin setup”**.

### 2.1 Build settings

On the setup screen:

1. **Project name**: keep the default or rename.
2. **Production branch**: select `main`.
3. **Framework preset**:

   * If you later use Next.js, set this to **“Next.js”**.
   * For now, if the project is empty, you can leave the preset as **“None”** – we’ll fix later when Gemini creates the app.
4. **Build command**: leave as default for now.
5. **Build output directory**: leave as default for now.

Scroll down and click **“Save and Deploy”**.

If the first build fails (because nothing exists yet), ignore it. Once Gemini builds a proper app, the build will succeed.

### 2.2 Enable preview deployments

1. Go to your new Pages project in Cloudflare.
2. Click the **“Settings”** tab.
3. In the left sidebar, click **“Builds & deployments”**.
4. Find **“Preview deployments”** and make sure it is **enabled for all branches / pull requests**.

You now have:

* A **Production URL** like `https://your-project.pages.dev` for the `main` branch.
* Automatic **Preview URLs** for other branches (e.g. `feature/something`).

---

## 3. Connect your repo to Gemini 3 AI Studio (for live UI building)

Gemini AI Studio lets you edit code and instantly see UI changes **inside the browser**. For startups and prototypes, you will do most of the early work here.

### 3.1 Create an app in AI Studio

1. Open **AI Studio** in your browser (Gemini).
2. Go to the **“Apps”** section (in the top or side navigation).
3. Click **“New app”** (or similar).
4. When asked for a source, choose **“Connect GitHub repo”**.
5. Authorise access to GitHub if asked.
6. Select **your new repo** (e.g. `my-ai-project`).
7. Choose **branch**: `main`.
8. Turn on **“Auto-deploy on push”** or similar option so every push to `main` redeploys the app.
9. Confirm / create the app.

### 3.2 How you’ll work in Gemini

* You open your app in AI Studio.
* There will be a **code view** and a **UI preview panel**.
* When you or the AI change the code and save/commit:

  * It pushes to your `main` branch on GitHub.
  * Cloudflare rebuilds and updates your **production** site.
  * The same site is visible in the AI Studio preview.

Use Gemini for:

* Page layouts
* Styling
* Simple logic
* Fast experiments

---

## 4. Set up Claude Code with a Custom Cloud Environment

Claude Code will be used for bigger refactors, background jobs, and more complex work. It won’t show you a live UI, so we keep Claude on **branches**, not directly on `main`.

### 4.1 Create a Custom Cloud Environment

1. Open **Claude** in your browser.
2. Go to **Settings** (click your profile icon).
3. Find the section called **“Cloud Environments”** (or similar wording).
4. Click **“New environment”**.
5. Name it something like: `my-project-cloud-env`.

### 4.2 Network settings

1. In the new environment, for **Network**:

   * Select **“Custom network”**.
   * Tick **“Include default allowed domains”**.

2. In the **Allowed domains** area, add these (one per line):

   * `github.com`
   * `developer.mozilla.org`
   * `typescriptlang.org`
   * `tc39.es`
   * `react.dev`
   * `reactjs.org`
   * `nextjs.org`
   * `vercel.com`
   * `prisma.io`
   * `neon.tech`
   * `postgresql.org`
   * `supabase.com`
   * `developers.cloudflare.com`
   * `cloudflare.com`
   * `nodejs.org`
   * `tailwindcss.com`
   * `shadcn.com`
   * `radix-ui.com`
   * `tanstack.com`
   * `zod.dev`
   * `trpc.io`
   * `vitest.dev`
   * `jestjs.io`
   * `playwright.dev`
   * `testing-library.com`
   * `eslint.org`
   * `typescript-eslint.io`
   * `developers.google.com`
   * `search.google.com`

3. Save the environment.

### 4.3 Environment variables (API keys etc.)

In the **same Cloud Environment**, there will be a place to set **environment variables**.

Typical examples (you can add later when needed):

* `CLOUDFLARE_API_TOKEN`
* `DATABASE_URL`
* `OPENAI_API_KEY`
* `ANTHROPIC_API_KEY`

Claude Code will read these. You **don’t** need a `.env` file for this setup.

---

## 5. Basic workflow: Gemini on `main`, Claude on branches

### 5.1 Fast UI work – use Gemini on `main`

* Open AI Studio.
* Use the UI to ask Gemini to build pages and components.
* When you like a change, let Gemini commit to `main`.
* Production site updates automatically via Cloudflare.

This is where you’ll spend most of your time at the beginning.

### 5.2 Risky / complex changes – use Claude on branches

Whenever a change feels risky (deep refactor, new backend logic, etc.):

1. Go to your repo on GitHub.
2. Click the branch selector (usually shows `main`).
3. Type a new branch name, e.g. `feature-auth-system`, and press **Enter** to create it.
4. In Claude Code, open your repo and switch to this **feature branch**.
5. Let Claude work there.

Cloudflare will:

* Build a **preview URL** for that branch.
* Example: `https://feature-auth-system.your-project.pages.dev`

You can open that URL to check the branch.

If it looks good:

1. Go to GitHub.
2. Click **“Compare & pull request”** for that branch.
3. Create a **Pull Request** into `main`.
4. Optionally, ask Claude or another AI to **review the PR**.
5. When you’re happy, click **“Merge”**.

After merging:

* Cloudflare redeploys **production** from `main`.

---

## 6. What the special files and folders are for

You don’t need to edit these perfectly on day 1. Just know the purpose.

### 6.1 `AGENTS.md` (root)

* Explains **what different AI agents are allowed to do**.
* Example roles:

  * “Planner” – breaks ideas into tasks.
  * “Executor” – writes code.
  * “Reviewer” – checks code.
  * “Session logger” – writes summaries of what happened.
* When you prompt an AI, you can say:

  * “Read `AGENTS.md` first. Act as the appropriate agent for this task.”

### 6.2 `orchestration/models.yaml` and `orchestration/policies.md`

* `models.yaml` decides **which AI model** is used for which job.
* `policies.md` contains rules like:

  * “Always work on a branch, not on `main`.”
  * “Don’t edit task files and session files at the same time” etc.

Most people don’t need to change these early on.

### 6.3 `sessions/INDEX.md` and `sessions/AGENTS.md`

* `sessions/AGENTS.md` tells an AI how to behave as a **session recorder**:

  * what to log
  * where to store logs
  * how to name session files
* `sessions/INDEX.md` is a **table of contents** for past sessions:

  * each time you finish a big chunk of work, the logger agent can append a short summary and a link to the new session file.

### 6.4 `tasks/TASKS.md` and `tasks/AGENTS.md`

* `tasks/TASKS.md` is your main **task list**:

  * backlog
  * in-progress
  * done
* `tasks/AGENTS.md` tells an AI how to behave as a **task manager**:

  * how to add tasks
  * how to mark them done
  * how to assign a task to a specific agent / model

### 6.5 `skills/`

* This folder will hold **reusable skills**.
* Example: a skill that explains “How to work with Next.js and Cloudflare Pages”, or “How to structure React components”.
* Each skill can be a folder with its own `SKILL.md` and extra notes.

When you later ask an AI for help, you can say:

> “Load the relevant skill from `skills/` if it exists.”

---

## 7. How to actually start building something

Once all connections are set:

1. In **AI Studio**:

   * Open your app.
   * Ask Gemini to:

     * create a minimal Next.js app (or other framework) in this repo,
     * wire it so it builds correctly on Cloudflare Pages.
2. Wait for the first successful deploy to Cloudflare.
3. Open your production URL (`https://your-project.pages.dev`) to confirm the app works.
4. From then on:

   * Use **Gemini** for day-to-day UI and simple logic.
   * Use **Claude / Codex** on separate branches for heavy changes.
   * Use `tasks/` and `sessions/` to keep track of what’s happening.

If something breaks:

* Ask an AI:

  * “Read `README.md`, `AGENTS.md`, `orchestration/policies.md`, and the latest files. I’m stuck on X. Walk me through fixing it step-by-step.”

That’s enough to get you from **zero** to a working multi-agent, cloud-hosted project without touching local dev tools.

