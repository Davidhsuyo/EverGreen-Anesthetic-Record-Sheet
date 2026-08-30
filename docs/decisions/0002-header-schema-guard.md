# 0002 — `HEADER_SCHEMA_VERSION` guards header layout independently of `__v`

**Status:** established (in place since commit `ff90e0b`, "fix: final drive filename +
guard against silent v3 header index shift"). Documented here, not re-decided.

## Mechanism
`index.html`'s patch block defines:
```
const HEADER_SCHEMA_VERSION = 2; // ~line 2573
```
with the comment (translated): the patient-header redesign (adding Name /
Reproductive Status, changing Species/Sex to `<select>`) changed the number and order
of form elements inside `.ucd-form-container`. Old drafts serialized under the same
`__v` but an older header layout would, if restored via the DOM-index scheme, put
values into the wrong fields (e.g. an old Species value landing in the new Name
field) — silently, with no error.

`EGSave.config.deserialize` checks `state.headerSchema` separately from `__v`:
```
if (Number(state.headerSchema || 1) < HEADER_SCHEMA_VERSION) throw EG_HEADER_SCHEMA_INCOMPATIBLE
```
and refuses to restore rather than risk a silent field-index shift.

## Why a second version number instead of bumping `__v`
`__v` tracks the overall payload format; header layout changes are a narrower,
higher-frequency risk (any time a field is added/removed/reordered inside
`.ucd-form-container`) that needed its own guard so it doesn't force every other part
of the payload format to be considered "changed" too.

## Implication for future work
- Any change to the number, type, or order of form controls inside
  `.ucd-form-container` is a candidate for bumping `HEADER_SCHEMA_VERSION`. Before
  bumping it, first check whether the change can avoid the container entirely (see
  `docs/troubleshooting/serializer-dom-index.md` — e.g. `contenteditable` instead of
  `<input>` was used for exactly this reason when custom clinician names were added).
- If you must bump it, that's a real compatibility break for existing saved records —
  say so explicitly to the user before doing it, and write a new ADR.
