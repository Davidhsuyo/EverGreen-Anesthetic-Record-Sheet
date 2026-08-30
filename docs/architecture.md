# Architecture

`index.html` is the entire application: no build step, no bundler. It is one HTML
document containing inline CSS and three `<script>` blocks that were added in layers
over the project's history. Read only the block relevant to your task — do not read
the file end-to-end.

## The three script blocks

### 1. Base app (`<script>` at line ~567–1599)
Original app logic: `drugDB` (drug reference data), the monitoring-grid `addRecord()`
flow (line 1008), Chart.js setup (`new Chart(...)` at line 796), textarea auto-resize.
This predates the save/restore and multi-clinician layers below.

### 2. Data-preservation module ("資料保全模組 v3.0", `<script>` at line ~1600–2561)
Autosave / version history / safety snapshots / shared-drive write, referred to in
code and here as **EGSave**. Layers, in the code's own terms:
- **L1** — `localStorage` draft, debounced autosave (`saveDraft`, ~line 1853).
- **L2** — version history, one snapshot per `versionIntervalMs` (5 min) in IndexedDB
  (`maybeCreateVersionSnapshot`, ~line 2048).
- **L3** — safety snapshot taken immediately before any risky restore/load/finalize
  (`createSafetySnapshot`, ~line 2064).
- **L4** — direct write to a user-chosen shared folder via the File System Access API
  (`drive.*`, ~line 2106).

Serialization lives here too: `egSerialize()` (line 1642) / `egDeserialize()` (line
1692) convert the DOM to/from a plain object. The original scheme reads/writes form
fields **by DOM index** (`.ucd-form-container input, select, textarea`, queried as a
flat NodeList) — this is why field order inside that container is load-bearing. See
`docs/troubleshooting/serializer-dom-index.md` before adding any form control there.

Public surface for consoles/tests: `window.EGSave` (line 2520) — `.config.serialize()`,
`.config.deserialize()`, `.save()`, `.createSafetySnapshot()`, `.listVersionHistory()`,
`.restoreFromHistorySnapshot()`, `.debugCheckVersionSnapshot()` (manually trigger the
5-minute version-snapshot check without waiting).

### 3. Consolidated patch layer (`<script id="eg-v22-multi-clinician-patch">`, line ~2562–3935)
Despite the `id`, this single IIFE has accumulated most feature work since the v2.1/2.2
line, in 10 numbered sections (search for `// N. ` inside the block):
1. Anesthetist/Surgeon multi-select (chip UI, fixed 17-person list + custom names,
   `normalizeClinicianList`).
2. Patient header fields (Name/Species/Sex/Reproductive Status/Date).
3. Anesthetic agent (ISO/SEVO + %).
4. Drug DB adjustments.
5. Regular-dose validation.
6. CRI (constant rate infusion) calculation.
7. Bolus drug rows.
8. `addRecord` wrapper.
9. Header/version compatibility gate + manual JSON save/load routed through the same
   `EGSave` serializer (`v2Message()`, `headerSchemaMessage()`, ~line 3720).
10. Shared-drive observability, init.

New feature work in this app has generally meant adding a new numbered section here
rather than a new top-level `<script>` block — follow that pattern unless there's a
strong reason not to.

## Versioning / compatibility scheme
Two independent version numbers matter:
- **`__v`** (payload format version, e.g. `3`) — data with `__v <= 2` is refused by
  `egDeserialize`/`EGSave.config.deserialize` with `EG_V2_INCOMPATIBLE_USE_FROZEN_READER`;
  the user is told to open `index-v2-frozen.html` instead (`v2Message()`, line 3721).
- **`HEADER_SCHEMA_VERSION`** (currently `2`, line ~2573 in the patch block) — guards
  the *patient header* field layout specifically, independent of `__v`. If saved data's
  `headerSchema` is older, restore is refused with `EG_HEADER_SCHEMA_INCOMPATIBLE`
  rather than silently loading values into the wrong fields. Bumping this number is a
  real compatibility break for every existing saved record — see
  `docs/decisions/0002-header-schema-guard.md` before doing it.

## Legacy files
- `index-v2-frozen.html` — frozen reader for `__v <= 2` data, actively referenced by
  `FROZEN_READER` in the patch block. Never edit.
- `index Ver2.html` (space in the name) — a one-off "Add files via upload" commit,
  never touched again, **not referenced anywhere in `index.html`**. Do not confuse it
  with the frozen reader above; do not assume it is safe to delete without asking —
  it predates this documentation effort and its purpose outside the repo is unverified.

## Deploy
GitHub Pages serves `main` branch, path `/`, legacy build type (no Actions build step
beyond GitHub's own `pages-build-deployment`). A push to `main` is live within minutes.
