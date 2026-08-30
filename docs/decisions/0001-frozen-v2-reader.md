# 0001 — Freeze a separate reader for old (`__v <= 2`) data instead of migrating it

**Status:** established (in place since commit `5a69e2a`, "chore: preserve v2 frozen
reader"). Documented here, not re-decided.

## Mechanism
`index-v2-frozen.html` is a full, self-contained snapshot of an older version of the
app. Current `index.html` refuses to load any saved payload with `__v <= 2`:

```
egDeserialize / EGSave.config.deserialize:
  if (!state || Number(state.__v || 0) <= 2) throw EG_V2_INCOMPATIBLE_USE_FROZEN_READER
```

The thrown error carries a user-facing message (`v2Message()`, `index.html` ~line
3721) telling the user to open `index-v2-frozen.html` instead of the current app.

## Why this shape (inferred from the code, not from external discussion)
The newer serializer format changed enough that silently attempting to map old data
into new fields risked putting values in the wrong place with no visible error —
unacceptable for medical records. Rather than writing a migration path for very old
data, the project keeps a working, unmodified old reader around and gates the new app
so it never tries to guess.

## Implication for future work
- `index-v2-frozen.html` must keep working standalone, forever, unmodified. Do not
  "clean it up" or try to unify it with `index.html`.
- If you're tempted to raise the `__v <= 2` cutoff to also exclude newer old data,
  that's a new decision — write a new ADR, don't edit this one.
