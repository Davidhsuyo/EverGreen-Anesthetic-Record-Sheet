# 0001 — Custom (non-list) anesthetist/surgeon names

**Status:** completed. Commit `8769b42` on `main` ("fix: allow custom anesthetists and
surgeons"), branched from `v2.2.1-custom-clinician` off `v2.2-multi-clinician`.

## Goal
v2.2 added multi-select chip UI for Anesthetist/Surgeon, but only from a fixed
17-person list. There was no way to add someone not on that list (visiting vets,
new hires), even though the underlying data format and restore paths already
tolerated arbitrary names.

## Approach
- Added an "其他／自訂姓名" (other/custom name) input + add button at the bottom of
  each clinician popover, writing straight into the existing `anesthetists`/
  `surgeons` arrays via the existing `normalizeClinicianList` — no new data structure.
- Used a `contenteditable` div instead of a real `<input>` for the text entry, to avoid
  shifting the legacy DOM-index serializer (see
  `docs/troubleshooting/serializer-dom-index.md` — this task is where that doc's
  advice came from).
- Added IME composition guards (`compositionstart`/`compositionend` +
  `event.isComposing`) on the Enter-to-add handler, so confirming a Zhuyin/Pinyin
  candidate doesn't get misread as "submit."

## Verification
Manual browser pass covering: trim/empty/whitespace/duplicate handling, chip removal,
localStorage+F5 restore, version-history and safety-snapshot restore (via
`EGSave.restoreFromHistorySnapshot`), JSON export shape (array, not a concatenated
string) and re-import, restoring old payloads containing a name outside the fixed
list, v2.1-style single-value payload restore, and confirming
`.ucd-form-container input/select/textarea` count and the Anesthetist/Surgeon indices
were unchanged before vs. after (63 elements, indices 9/10, via `git stash` A/B).

## Known limitation hit during verification
The automated browser tool used for testing sends a broken synthetic `Enter` keydown
(`event.key` empty, reproduced even on a plain native `<input>`) — not a bug in this
feature. The Enter-to-add path was verified instead by dispatching a real
`KeyboardEvent({key: 'Enter'})` directly, which behaved correctly. A human should
still press Enter once for real before trusting this blindly on the next similar task.
