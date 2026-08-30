# Current status

_Snapshot as of 2026-08-31. This file goes stale — treat it as a starting point, then
confirm with `git log -10`, `gh pr list`, `gh issue list` for the live truth._

## Deployed on `main`
`8769b42` — `fix: allow custom anesthetists and surgeons`. Anesthetist/Surgeon fields
support both the fixed 17-person list and free-text custom names (chip UI), all stored
in the same `anesthetists`/`surgeons` arrays. `HEADER_SCHEMA_VERSION` is still `2`.

Recent history on `main` (newest first):
1. `8769b42` fix: allow custom anesthetists and surgeons
2. `c1f99e1` feat: support multiple anesthetists and surgeons
3. `aed224a` feat: add five-minute version history and restore
4. `ff90e0b` fix: final drive filename + guard against silent v3 header index shift
5. `179b0d3` feat: structure patient fields and case filenames

## Active work
None. No open issues, no open PRs, no other unmerged feature branches beyond routine
cleanup at time of writing.

## Known gaps / open questions
- No automated test suite exists — every change needs a manual browser pass (see
  `AGENTS.md` → Definition of done).
- `main` has no branch protection and GitHub Pages deploys it live immediately; the
  project has historically been pushed to directly rather than through PRs (zero PRs
  in repo history before this documentation effort). See
  `docs/references/github-workflow.md` for the recommendation on this.
- `index Ver2.html` (space in filename) is an unreferenced legacy file of unclear
  purpose — see `docs/architecture.md` → Legacy files. Not yet resolved either way.

## Repo layout at a glance
See `AGENTS.md` → Directory map, and `docs/architecture.md` for how `index.html` is
internally structured.
