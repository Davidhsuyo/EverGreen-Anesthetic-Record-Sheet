# GitHub workflow — current state and minimal recommendation

Checked via `gh api`/`gh repo view` on 2026-08-30/31:
- Branch protection on `main`: **none** (404 from the branch-protection API).
- CI: only GitHub's own `pages-build-deployment` (no test/build workflow exists).
- GitHub Pages: builds from `main`, path `/`, legacy build type — a push to `main` is
  live within minutes, with no review gate in between.
- PR/Issue history: zero PRs and zero issues in the repo's history before this
  documentation pass. Development has been direct commits (or a branch merged by hand)
  to `main`.

## Minimal recommendation (not enacted — needs the repo owner's call)
Given `main` = production with no gate, the smallest change with real safety value:
- Do non-trivial changes on a branch and open a PR before merging, even without any
  CI checks to require — a PR at least creates a reviewable diff and a pause before
  the live site changes.
- Consider (owner decision, not done here): a branch protection rule on `main`
  requiring at least a PR (no force-push), *without* requiring any status checks,
  since there is no test suite to check yet. This is a GitHub repo-settings change and
  was intentionally left to the repo owner rather than made unilaterally.

Explicitly **not** recommended for a project this size: required status checks with a
real CI test suite, multiple reviewer requirements, or any other heavier process —
would be over-engineering for a single-maintainer static site.
