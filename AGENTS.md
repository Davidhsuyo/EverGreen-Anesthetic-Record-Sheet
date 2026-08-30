# AGENTS.md

## Project goal
Client-side veterinary anesthesia record sheet (Traditional Chinese UI). `index.html`
is the entire app — no backend, no build step. GitHub Pages serves `main:/index.html`
directly, so **every push to `main` is an instant production deploy**.

## Critical constraints
- No build/bundler/framework. Plain HTML + CSS + vanilla JS only. Do not add
  dependencies (Chart.js via CDN `<script src>` is the one exception, already in place).
- Backward data compatibility is the top priority. Vets have live patient records saved
  as JSON files, in `localStorage`, and in IndexedDB version history across multiple
  schema generations. Silently breaking a restore path corrupts real medical records.
  See `docs/architecture.md` before touching serialization, `HEADER_SCHEMA_VERSION`,
  or anything under `.ucd-form-container`.
- `index-v2-frozen.html` is a frozen legacy reader, actively referenced by
  `FROZEN_READER` in `index.html`. Never edit or delete it.
  `index Ver2.html` (note the space) is an orphaned one-off upload, not referenced by
  any code — leave it alone too; do not assume it is interchangeable with the frozen
  reader.
- Do not modify clinical calculation logic (CRI, bolus, drug dosing in `drugDB`)
  unless the task explicitly asks for it — these are patient-safety critical and are
  out of scope for docs/tooling/UI work.
- Do not push directly to `main` unless the user explicitly says so. Use a feature
  branch + let the user confirm before merging (main deploys live immediately).

## Coding rules
- Match existing style: IIFE-wrapped patch scripts with a descriptive
  `<script id="...">`, numbered `// N. Section name` comments, Traditional Chinese for
  user-facing strings and comments explaining *why* (not what).
- No comments restating obvious code. Comment only non-obvious constraints (e.g. "this
  must stay a `contenteditable`, not an `<input>`, or the DOM-index serializer shifts").
- Prefer editing existing patch-script sections over adding new top-level `<script>`
  blocks.

## Build / test / run
There is no build step, package manager, linter, or automated test suite.
- Run locally: `python -m http.server 8000` from repo root, open `http://localhost:8000/index.html`.
- Validation is manual, in a real browser. Minimum checklist before calling anything
  done: see "Definition of done" below.

## Directory map
```
index.html              production app (HTML + CSS + 3 JS blocks, see docs/architecture.md)
index-v2-frozen.html    frozen legacy reader for old v2-format patient data — do not touch
index Ver2.html         orphaned legacy upload, unreferenced — do not touch
docs/                   architecture, decisions, status, troubleshooting (read on demand)
```

## Definition of done
1. Manual browser test of the golden path + the specific edge case you changed.
2. If you touched serialization/clinician fields/header layout: confirm the
   `.ucd-form-container input/select/textarea` count and index of existing fields are
   unchanged (see `docs/troubleshooting/serializer-dom-index.md`).
3. Console has zero unexpected errors/warnings.
4. Commit message follows existing convention: `feat:`, `fix:`, `chore:`.
5. Work lands on a branch; `main` is only touched with explicit user confirmation.

## Docs index (read on demand, not up front)
- `docs/current-status.md` — fast recovery snapshot: what's deployed, what's active. Read first.
- `docs/architecture.md` — the 4 structural blocks of `index.html`, versioning/compat scheme. Read before touching save/restore or clinician/header fields.
- `docs/decisions/` — short ADRs for non-obvious mechanisms (frozen reader, header schema guard). Read the relevant one before changing that mechanism.
- `docs/troubleshooting/` — known gotchas (e.g. DOM-index shift). Check before adding form fields.
- `docs/exec-plans/active/` and `completed/` — in-flight and past task write-ups, for context on *why* something looks the way it does.
- `docs/references/` — external links (repo, Pages URL, GitHub workflow notes).

## Context discipline
- Search before read: grep/find the symbol first, then read only the matching range.
- Read minimum relevant line ranges, not whole files. `index.html` is ~3900 lines —
  never read it end-to-end "just in case."
- Don't dump unbounded shell/git/test output into context; pipe through `grep`/`head`/
  `tail`/line limits, or use `--stat`/`--short` forms first.
- Prefer targeted diffs (`git diff -- <file>`, `git show <sha> -- <path>`) over the full
  repo diff.
- Load a doc under `docs/` only when the current task actually touches that area.

## Short-session workflow
Prefer: Issue/ask → quick research → implementation → manual test → (independent
review if available) → PR → merge, over one long session accumulating unbounded
context. There is no CI test suite, so "CI checks" here means the Definition of Done
checklist above, done manually before merge.
