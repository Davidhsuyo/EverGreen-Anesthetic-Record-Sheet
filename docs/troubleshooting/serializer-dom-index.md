# Adding a form field without shifting the legacy serializer's index

## Symptom this prevents
`egSerialize`/`egDeserialize` (see `docs/architecture.md`) read and write most header
fields **by position**, not by id:
```
document.querySelectorAll('.ucd-form-container input, .ucd-form-container select, .ucd-form-container textarea')
```
Every element matched by that selector gets a fixed index. Old saved data (JSON
exports, localStorage drafts, IndexedDB version history / safety snapshots) stores
values against those indices. If you add, remove, or reorder a real `<input>`,
`<select>`, or `<textarea>` inside `.ucd-form-container`, every field *after* it shifts
by one — and old saved data silently restores into the wrong field. There is a guard
(`HEADER_SCHEMA_VERSION`, see `docs/decisions/0002-header-schema-guard.md`) that can
catch this for header-layout changes, but the right move is to avoid causing the shift
at all when possible.

## How to add new interactive UI inside `.ucd-form-container` without shifting the index
Options, in order of preference:
1. **Don't add a real form control.** Use a `contenteditable` element instead of
   `<input>`/`<textarea>` if you need free-text entry — it is not matched by the
   selector above. This is what the custom-clinician-name input uses
   (`.eg-clinician-custom-input`, patch block section 1).
2. **Put the new control outside `.ucd-form-container`** in the DOM, and position it
   visually with CSS if needed.
3. If neither works and the index must shift, that's a real breaking change — bump
   `HEADER_SCHEMA_VERSION` deliberately and say so to the user first.

## How to verify you didn't shift anything
Before and after your change, in the browser console:
```js
const els = document.querySelectorAll('.ucd-form-container input, .ucd-form-container select, .ucd-form-container textarea');
els.length; // total count
[...els].findIndex(el => el.dataset && el.dataset.egV22ClinicianCompat === 'anesthetists'); // or whatever anchor field matters for your change
```
Compare against the same check on `main` before your change (e.g. via `git stash`).
Counts and indices for every pre-existing field must match exactly.
