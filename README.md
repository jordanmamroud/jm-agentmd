# Agentic Engineering architecure we want agent md to implement. 

## The core idea

The route folder *is* the feature. Build everything in the route that uses it. Move to `src/` only when you have proof a second route needs the same thing.

Two folders matter:

- `app/` — your routes. Each folder is both a URL and a self-contained feature. Code lives in private subfolders (`_components/`, `_lib/`) that Next.js won't expose as routes.
- `src/` — only stuff that has no natural route home: the database, validated env vars, encryption helpers. Stack-level infrastructure.

That's the whole architecture.

Why this architecture wins for agentic engineering:
Each route folder is a self-contained feature. An agent gets one task, opens one folder, finds everything it needs, and ships. No traversing five layers, no taxonomy guesses ("is this a widget or a feature?"), no stepping on other agents' files. Five agents on five different routes = zero collisions by default. The folder tree itself is the spec — agents don't need a 800-line architecture doc explaining where things go (which the ETH Zurich study showed actively hurts performance).

Why blast radius matters:
Every file placement is a guess about the future. Putting code in src/lib/ claims "many routes will use this." Putting it in app/runs/_lib/ claims "only this route uses it." When you're wrong about a big claim, the cost is high — wrong abstractions, weird naming, agents pulling related code into the wrong place to "match." When you're wrong about a small claim, the cost is one mv command later.
Smaller blast radius = cheaper to be wrong. And you're going to be wrong sometimes — you don't actually know what'll be reused until the second use case shows up. So defer the big claims. Let jscpd catch real duplication on refactor day, and then promote with evidence instead of intuition.
The whole architecture reduces to one sentence: build in the route, defer to src/, and when unsure, guess small.

## Good scaffolding example

A v1 of GA Helper, two weeks in, looks like this:

```
app/
  layout.tsx
  page.tsx                          # redirects to /runs
  runs/
    page.tsx                        # upload area + history table
    actions.ts                      # uploadCsv, startRun, deleteRun
    _components/
      csv-uploader.tsx
      runs-table.tsx
      progress-panel.tsx
    _lib/
      classify.ts                   # the run pipeline
      consensus.ts
      providers/
        openai.ts
        anthropic.ts
        google.ts
      csv.ts
    [id]/
      page.tsx
      actions.ts                    # exportRunCsv
      _components/
        results-table.tsx
        export-button.tsx
  prompts/
    page.tsx
    actions.ts
    _components/
      prompt-editor.tsx
  settings/
    keys/
      page.tsx
      actions.ts
      _components/
        key-form.tsx
      _lib/
        validate-key.ts
    guardrails/
      page.tsx
      actions.ts
      _components/
        guardrails-form.tsx

src/
  db/
    schema.ts
    client.ts
  lib/
    env.ts
    encrypt.ts

AGENTS.md                           # 50 lines, no architecture diagram
```

**Why it's good:**

- Three top-level routes, each fully self-contained. Three agents could work on them in parallel without touching each other's files.
- The classify pipeline lives in `app/runs/_lib/` because only the runs route uses it right now. That's where it earns its keep.
- `src/` has four files total. Nothing speculative. Every file is genuinely shared infrastructure.
- One `actions.ts` per route, not one file per action. Easier imports, fewer files to scan.
- No `src/features/`, no `src/widgets/`, no `src/types/`. Empty folders are guesses, and guesses confuse agents.

What you'll notice: there's duplication. `app/settings/keys/_lib/validate-key.ts` calls OpenAI to test a key, and `app/runs/_lib/providers/openai.ts` also calls OpenAI to classify. That's intentional. Refactor day catches it.

## Bad scaffolding example

Same app, organized the way someone would draw it on a whiteboard before writing any code:

```
src/
  features/
    runs/
      components/
        RunsList.tsx
        RunDetails.tsx
      hooks/
        useRuns.ts
        useRunStatus.ts
      services/
        runsService.ts
        runsApi.ts
      types/
        Run.ts
        RunStatus.ts
        RunResult.ts
    prompts/
      components/
      hooks/
      services/
      types/
    settings/
      components/
      hooks/
      services/
      types/
  widgets/
    csv-uploader/
      ui/CsvUploader.tsx
      model/csvUploaderStore.ts
      api/csvUploaderApi.ts
    runs-table/
      ui/RunsTable.tsx
      model/runsTableStore.ts
  entities/
    run/
      ui/RunBadge.tsx
      model/run.ts
      api/runApi.ts
  shared/
    ui/
      Button.tsx
      Input.tsx
      Dialog.tsx
      Table.tsx
      EmptyState.tsx
      Card.tsx
    lib/
      formatters.ts
      mappers.ts
    api/
      apiClient.ts
  domain/
    classification/
      ClassificationEngine.ts
      ConsensusStrategy.ts
      providers/
        IProvider.ts
        OpenAIProvider.ts
        AnthropicProvider.ts
  infrastructure/
    persistence/
      RunRepository.ts
      PromptRepository.ts
    external/
      OpenAIClient.ts

app/
  runs/page.tsx                     # imports from src/features/runs/
  prompts/page.tsx
  settings/page.tsx

AGENTS.md                           # 800 lines, includes layer diagrams
```

**Why it's bad:**

- Adding "delete run" requires touching `src/features/runs/components/`, `src/features/runs/services/`, `src/features/runs/types/`, possibly `src/entities/run/`, and finally the page in `app/runs/page.tsx`. Five folders for one feature.
- Every empty folder is a taxonomy decision. Is this a `widget` or a `feature` or an `entity`? Two agents will pick differently and the codebase will drift.
- `IProvider.ts` interface, `ClassificationEngine` class — premature abstractions written before the second provider was even tested. The interface won't fit the second use case.
- `src/shared/ui/Button.tsx` exists with one usage. Promoted speculatively, not on proof.
- `RunRepository`, `PromptRepository` — repository pattern overhead for what's actually three SQL queries.
- The 800-line AGENTS.md tries to explain this taxonomy to agents, which the ETH Zurich study found *increases* inference cost by ~20% without improving success.

When five agents try to add features to this codebase in parallel, they all need to touch `src/shared/ui/`, `src/entities/run/`, and `src/features/runs/types/`. Merge conflicts every commit.

## Steinberger's blast radius rule

Peter Steinberger talks about "blast radius" — how many files and routes a change affects. When unsure where to put something, **pick the smaller blast radius**.

Putting `formatCost.ts` in `src/lib/` makes a claim: *"this will be used by many routes."* Blast radius is large — every route can now reach it, and naming/conventions for cost formatting are now project-wide.

Putting `formatCost.ts` in `app/runs/_lib/` makes a smaller claim: *"the runs route uses this."* Blast radius is one folder. If you guessed wrong about reuse, no harm done. If you guessed right, you move it later. Moving is one `mv` and a few imports — cheap.

The rule applies to every placement decision:

| Decision | Bigger radius (avoid when unsure) | Smaller radius (prefer) |
|---|---|---|
| New helper | `src/lib/foo.ts` | `app/<route>/_lib/foo.ts` |
| New component | `src/ui/Button.tsx` | `app/<route>/_components/button.tsx` |
| New type | `src/types/Run.ts` | inline in the file using it |
| New hook | `src/hooks/useRuns.ts` | `app/<route>/_lib/use-runs.ts` |
| New abstraction | Generic interface upfront | Concrete impl, abstract on second use |
| Splitting a file | Split now into 3 files | Wait until file >300 lines with clear sub-pieces |

Steinberger's reasoning: every guess about future structure has a cost when wrong. Smaller guesses cost less. So when uncertain, guess small.

## Why this works for parallel agents specifically

When you run five agents at once, the bottleneck isn't agent intelligence — it's coordination. The architecture has to make coordination automatic, not manual.

Three properties matter:

**1. One folder per task.** When an agent is told *"add a CSV preview to the upload area,"* it should be able to scan one folder (`app/runs/_components/`) and find everything relevant. If it has to traverse `src/features/runs/`, `src/widgets/csv-uploader/`, and `src/shared/ui/`, it loads three times the context and makes three times the assumptions.

**2. Zero taxonomy decisions.** *"Is this a widget or a feature or an entity?"* — that judgment call is where agents get inconsistent. Two agents will pick differently. The good architecture removes the question. There are routes, and there's the basement (`src/`). That's it.

**3. Files don't overlap between agents.** Agent A working on `app/runs/` and Agent B working on `app/prompts/` literally cannot touch the same file unless they're both editing `app/layout.tsx` or `src/db/schema.ts`. Those are the documented collision-risk files in `AGENTS.md`. Everything else is automatic isolation.

## How the system stays clean over time

The architecture would degrade if duplication piled up forever. Steinberger's answer: refactor days. Roughly weekly, you run `jscpd` (duplication detector) and `knip` (dead code finder). The output tells you what to promote.

Workflow:

1. Run `pnpm refactor-day`.
2. `jscpd` says: *"`app/runs/_lib/providers/openai.ts` and `app/settings/keys/_lib/validate-key.ts` share 40 lines."*
3. Prompt an agent: *"Extract the shared OpenAI call logic to `src/domain/providers/openai.ts` and update both files. Atomic commit."*
4. `src/domain/` gets created on the first promotion that needs it.
5. Repeat.

The folder `src/domain/` doesn't exist on day one. It exists after duplication earns it. Same for `src/ui/`, `src/hooks/`, anything else. **Folders appear when proven, not when imagined.**

## The mental model that ties it together

Think of it as three rules in priority order:

1. **Build in the route** that uses the code. Default location for everything except infrastructure.
2. **Defer promotion** until you have proof: a second importer, jscpd flagging duplication, or a 300+ line file with clear sub-pieces.
3. **When unsure, smaller blast radius wins.** Smaller guesses are cheaper to undo.

Everything else — the AGENTS.md template, the underscore folders, the refactor-day tooling — is supporting machinery. The architecture itself is just those three rules.

The bad example fails by inverting all three: it builds in `src/features/` not the route, promotes everything upfront not on proof, and prefers large abstractions over small concrete files. It looks tidy. It works terribly when five agents try to use it.

The good example is messier in places — that duplicated provider call, that 250-line component, that `_lib/` with a dozen small files. But every messiness is *deferred work*, not technical debt. Refactor day pays it down deliberately, on a schedule, in batches, when the right shape has actually become visible.
