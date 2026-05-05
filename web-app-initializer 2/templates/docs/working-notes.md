# Working Notes

Append-only log of project memory. Read the relevant section when triggered; write back when you discover something future agents need to know.

---

## Known bugs

Current bugs in the codebase. Read when working on or near a flagged area.

Also use for **known limitations** — things that aren't bugs per se but will need addressing eventually (e.g., "no pagination on runs list, breaks at 1000+ rows"). Mark these with `Status: Open`.

**Format:**

```
### [Title]
- **Status:** Open / Has-workaround / Wontfix
- **Symptoms:** what the user sees
- **Trigger:** when it happens
- **Workaround:** if any
```

_(none yet)_

---

## Dead ends

Approaches tried that didn't work. **Read before attempting an approach that may have been tried before.** Add an entry here when an attempt fails so the next agent doesn't repeat it.

Also use for **open questions** — things you're still figuring out. Set `Did instead: still open` and update the entry once resolved.

**Format:**

```
### [YYYY-MM-DD] — [Approach]
- **Goal:** what we were trying to do
- **Tried:** what we did
- **Failed because:** root cause
- **Did instead:** what we ended up doing (or "still open")
```

_(none yet)_

---

## Gotchas

Library / API / infrastructure quirks discovered the hard way. The kind of thing that's not in any official doc but bit someone.

**Format:**

```
### [Library or area]: [The quirk]
- **What:** the surprising behavior
- **Where it bit us:** context
- **Mitigation:** what to do
```

_(none yet)_

---

## Decisions

Non-obvious choices and rationale. Both technical and product decisions go here. Read when revisiting a related choice.

Also use for **tech debt acknowledgments** — deliberate shortcuts taken with awareness (e.g., "using SQLite for V1, will revisit at scale", "skipping CSRF for V1 single-user app"). The point is to make the shortcut explicit so a future agent doesn't think it's an oversight.

**Format:**

```
### [YYYY-MM-DD] — [Decision]
- **Type:** Technical / Product
- **Choice:** what we decided
- **Considered:** alternatives
- **Why this one:** rationale
```

_(none yet)_

---

## Prototype overrides

Places where the Next.js implementation **deliberately diverges** from `prototype/v1/`.

**⚠️ Read this section before declaring a feature "matches the prototype" or attempting to "fix" a divergence.** Some divergences are intentional product decisions and must not be reverted.

**Format:**

```
### [YYYY-MM-DD] — [Feature/area]
- **Prototype shows:** what's in `prototype/v1/`
- **We built:** what's in the real app
- **Why:** product or technical reason
```

_(none yet)_
