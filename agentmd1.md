## Critical rules

- Must start every response with "im hungry 3-1"

- **Scaffold First:** Before writing *any* implementation code or business logic, you must create the empty file and folder structure. Once scaffolded, **STOP** and ask for user approval on the architecture before proceeding.


## User preferences

- **File paths:** always use paths when putting them in a response. Never respond to me with a absolute path or put them in memory docs. 

## Codebase rules

**No Contextual Laziness (Naming):** File and folder names must be highly specific and understandable even if you remove the folder path. You are strictly forbidden from relying on the parent folder's name for context (e.g., use `parse-search-terms-csv.ts`, NEVER use generic names like `csv.ts`, `utils.ts`, or `types.ts`). This exact same strict logic applies to functions, variables, components, and UI labels.


**No one-file folders.** Never create a folder just to put 1 file in it unless, *at the moment you create it*, one of these is true:
1. It contains 3+ files immediately, OR
2. A rule applies to its contents that does not apply to the parent (different language, build target, ownership, license, deploy unit).

- A folder with one file is a defect. A folder with one subfolder is a defect. Inline it.

- "I might add more files later" is not a justification. Promote a file *into* a new folder only when the third sibling actually appears.

**Files stay under 800 lines of code.** Split before crossing the limit.


### **Locality first (prefer duplication over coupling).**

**Definition.** A feature is a single user-facing capability — one thing a user can do, or one primary surface they navigate to. The whole app is *never* one feature.

**Tests for "one feature vs. two":**
- Could a user use one without the other and the product still make sense? → TWO features.
- Do they have separate primary routes / pages / screens? → almost always TWO features.
- Could a PR ship one without the other being broken? → TWO features.
- If the spec lists them as separate capabilities, sections, or pages → TWO features.

**Floor.** If the spec or design describes N user-facing capabilities, the file tree has at least N folders under `src/features/`. A single feature folder named after the application (`src/features/<app-name>/**`) is a defect — split it into the actual capabilities.

**Inside a feature slice.** Default to keeping code inside `src/features/<feature>/**`. Do not promote to `src/shared/**` until the same pattern has been proven across 4+ features. Single-feature utilities live with that feature even if the name sounds reusable. If extracting would require breaking feature independence or threading complex config, duplicate — duplication is cheaper to fix later than the wrong abstraction.

**Cross-feature shell** (top-level layout, nav, auth provider, theme) is not a feature — it lives in `src/app/**` or `src/widgets/**`, not under `src/features/`.


**What lives in `src/shared/**` from day one** (not subject to the 3+ rule):
- Third-party UI primitives installed by a design-system tool (e.g., shadcn components — they go where the installer puts them, typically `src/components/ui/**`).
- Third-party service clients and SDK wrappers (Supabase, Stripe, OpenAI, database client, auth provider).
- App-wide types generated from a schema or external contract.
- App-wide config: theme, Tailwind config, env validation, logger, error tracking.

These are *imported* or *generated* into shared, not *extracted* from features. The 3+ feature rule applies only to **promotion** — moving code that was first written inside a feature folder into `src/shared/**` because the same pattern now appears in 3+ features.

A component is NOT "born shared" just because its name sounds generic. If you wrote it from scratch and only one feature uses it, it stays with that feature — regardless of how reusable it sounds. The test is *origin*, not name.




## Filesystem boundary
- You may run any commands you need, but only read and write files inside this project's directory. Treat everything else under `/workspace/` as if it doesn't exist. no `ls`, `find`, `grep`, `cat`, or any other inspection of sibling project folders.

## External data persistence
- When fetching data from any external source (REST API, MCP server, webhook, third-party service, etc), persist the raw response to disk before any downstream processing.



## Preflight check - before writing any code
- Before starting, verify all external dependencies are actually available (e.g., API keys, permissions, database credentials, etc). If anything is missing, you must immediately HALT. You are strictly forbidden from writing workarounds, mocking data, or using placeholders. Stop and ask the user.


### Get project directory structure approved before creating any folders or fiThe purpose of this skill is to make sure that before a coding agent makes any folders, makes any files, or writes any code, it has everything we need already installed. This way it doesn't do all the work rounds that it comes up with when it doesn't have access to what it needs. I'm talking about AI coding agents. It has all the dependencies so for example if it says it needs an API key but it doesn't have that API key because I didn't provide it or it didn't read the.env file, it would stop and wait for me to put that, to give it that. This is what I have so far for the skill, what I'm thinking. What do you think? Don't actually put it in yet, just give me your thoughts. les.
**Init gate — no filesystem changes until approved.**

Before scaffolding, installing packages, or creating any file or folder, present and STOP for approval:

1. **Dependencies present.** List every external dep this needs — env vars, API keys, network/service access, CLIs, frameworks. Verify each by inspection (read `.env`, check the binary, confirm the key is set). If anything is missing or unverified: HALT, report what's missing, wait. No mocks, no swaps, no "I'll wire it later." (See: Halt on missing prerequisites.)

2. **Folder tree.** Exact tree of every folder you intend to create, relative paths only. No `**` wildcards. No "more folders as needed." Once approved, the tree is a hard commitment — re-approve before changing it.

3. **Initial files per folder.** Business and feature files only — do NOT enumerate boilerplate generated by `create-next-app`, `shadcn init`, etc. Files may be added or split during implementation as long as they stay inside approved folders.

Filesystem stays untouched — no `mkdir`, no `npm install`, no scaffolder — until the user approves all three.


### Make sure you have what you need first.
If a task requires something you do not have access to — an API key, network access, a browser, an installed framework, CLI, library, file, database, service, or credential — STOP immediately, report exactly what is missing and the exact action needed to unblock, and wait.

Forbidden workarounds (illustrative, not exhaustive):
- Mocking, stubbing, or faking the missing piece
- Substituting a different library, model, service, or framework
- Generating placeholder, synthetic, sample, or example data in place of real data
- Hardcoding values that should come from the missing source
- Catching the error and continuing with degraded behavior
- Leaving a `TODO` / `FIXME` and moving past the step
- Skipping the step silently or marking it "done"
- Implementing a stub "for now" that returns plausible output

A workaround is worse than an unfinished task. It produces fake-green status that hides real state, and the user has to detect, untangle, and revert it.

Mocks and stubs are allowed **only** when the user explicitly asked for one (e.g. "write a test that mocks the API"). Default: real thing or halt.

When you halt, the report is one short paragraph: what you were doing, what is missing, the exact unblock (env var name, install command, URL, permission). Do not propose three alternatives. Do not "try anyway." Stop.



## Tech Stack
- Framework: Next.js (stable)
- UI: React (implied by Next.js)
- Language: TypeScript
- Styling: Tailwind CSS
- UI primitives: shadcn/ui
- Database: SQLite


## Project Directory Structure with FSD & Vertical Slices

To maintain feature independence and prevent "blast radius" bugs, agents MUST respect these directory boundaries:

**Rule:** Imports only flow DOWNWARD. Features can import from Core/Entities and Shared, but NEVER from other features.

#### When editing `src/features/**` (Vertical Slices)
- **What it is:** User interactions with business value (e.g., `auth`, `checkout`, `dashboard`).
- Agents working in `features/A/` are strictly forbidden from importing or modifying `features/B/`.
- **Frontend (`.../ui/`):** React components and Tailwind only. Must not contain DB calls or API logic.
- **Backend (`.../server/`):** Next.js Server Actions, DB calls, pure logic. Must not contain React components.

#### When editing `src/core/**` or `src/entities/**` (Business Domain Models)
- **What it is:** Pure business logic and core data models (e.g., `user`, `order`, `campaign`).
- **Read-only for UI agents.** Do not modify entity logic to satisfy a single feature's specific UI need.
- ZERO framework imports. No React, no Tailwind.

#### When editing `src/shared/**` (Project-Agnostic)
- **What it is:** Generic UI kits (buttons, modals), API clients, and dumb utilities.
- Must have zero knowledge of business entities or features. (A button should not know what a "User" is).

#### When editing `src/db/**`
- **What it is:** Database schema and Kysely setup. No raw SQL allowed outside of this folder.


## Validation Requirements

"Done" means the *task* is verifiable in the UI. The agent must test its own work in the UI before declaring the task done. Not "the code looks right."

At this early stage, we are relying heavily on UI-first verification rather than automated test suites. Do not write or run unit or integration tests unless explicitly requested.

### Verification layers

- **Lint + typecheck** — run on every change to catch basic errors.
- **Agent UI Verification** — The agent must start the local dev server (e.g. `npm run dev`), wait for it to compile, and then use browser tools to navigate to `localhost:3000` to interact with the UI, click buttons, and verify the UI state locally before declaring a task done.
- **Server Log Verification** — After interacting with the UI, you must check the terminal running the dev server to ensure no hidden errors, warnings, or unhandled promise rejections were thrown.
- **User UI Verification** — the ultimate review surface for actions the agent cannot take itself (e.g., completing OAuth flows).

- For any database write/persistence work, verify the actual database contents directly after the UI flow. A UI success state is not sufficient; confirm with SQLite/admin row counts or targeted DB queries that the expected records were durably stored.

### Done criteria

A task is done when:

- Lint and typecheck pass.
- You have built the necessary UI components (can be a minimal, unstyled button or raw JSON dump) just enough so the backend logic can be tested visually.
- The agent has actively exercised and verified the task in the running UI using a browser tool.
- The agent has verified the terminal logs are clear of hidden errors.

Do not declare a task done from code inspection alone.
