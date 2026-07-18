---
trigger: always_on
description: Same system as the Execution Status project: local edits accumulate as ops,
---

# Family Tree — working notes

## Change tracking & publish ("chat = server, baseline = database")
Same system as the Execution Status project: local edits accumulate as ops,
the user copies a change-set from the Save dialog and pastes it in the chat,
and Claude "bakes" it into the shared baseline.

### Architecture
- **Working copy** (auto-saved sidecars, shared by everyone in the design tool):
  - `.image-slots.state.json` — photo drops & crops as data URLs (written by `image-slot.js`)
  - `.ft-doc.state.json` — the ops log + tree-exclusion overrides + **user-authored moments** (written by `tree-store.js`)
- **Baseline**: `tree-seed.js` (`window.FT_SEED = { seedStamp, baseV, slots, excluded }`).
  `slots[id] = { src, w, h, s, x, y }` where `src` is a REAL project file under `photos/`.
  `image-slot.js` getSlot() falls back to the seed; a `{del:1}` sidecar marker shadows a seeded photo.
- **Ops** (`tree-store.js`): every photo add/replace/remove, avatar crop, hide/show-on-tree,
  field edit, and STRUCTURAL change (person/union/side-link add+remove — Edit mode) records
  `{ts, kind, id|key|ekey|pid|uid|side, label, prev*, pub}`. `prevRaw` = raw sidecar value before the
  change (for undo/discard). Slot ids: `ph-<person>-e<idx>` milestone photo, `av-<person>-e<idx>`
  avatar crop. Exclusion keys: `<person>-e<idx>`.
- **Field edits (Records Room)**: `doc.edits[ekey] = value` + op `{ts, kind:'edit', ekey, label, prevV, prevENone?, pub}`.
  ekey grammar: `p:<personId>:<dot.path>` person field (name.en, name.he, bio.en, bio.he, birth.year,
  birth.place, death.year, death.place, sheet.look, sheet.items — the last two are the character
  sheet feeding image prompts; bake as a `sheet: { look, items }` object on the person) ·
  `m:<personId>:<idx>:<field>` milestone field (en, he, post, postHe,
  year, tier, hideStory; idx = index in that person's `milestones[]`; `he`/`postHe` are the Hebrew
  title/story shown when the app language is Hebrew; `hideStory:true` hides the
  event from the focus view — bake as a boolean field on the milestone) · `u:<unionId>:<field>` union field
  (year, endYear) · `w:<evId>` world-event on/off (bool; `wevEnabled` is wrapped to consult it) ·
  `w:<evId>:<field>` world-event text field (title, titleHe, short, shortHe, desc, descHe, long,
  longHe — bake into that event's entry in `world-events.js`; empty string → delete the field).
  Values apply LIVE onto `window.FAMILY` objects on load and on each edit. The Records Room UI
  (`records-room.jsx`, editor-mode-only "Records" button in the top bar) is the editor for these.
- **UI**: `publish.jsx` — `SaveChip` in the top bar (dirty → "Save · N changes", copied →
  "N awaiting paste", else "vN · in sync") + `PublishDialog` (op list, undo/redo, per-op discard,
  trash-all, 3-step explainer, "Copy changes (for Claude)" → `FTStore.changeSet()` JSON +
  `markPublished()`). Global ⌘Z/⇧⌘Z → FTStore.undo/redo (app.jsx).
- On load, `tree-store.js` REBASES: if `FT_SEED.seedStamp` > doc's, published ops are dropped
  (they're in the seed now), unpublished ops survive, matching exclusion overrides are pruned.

### Baking a change-set (DO THIS when a `{"type":"ft-change-set",...}` JSON is pasted)
1. Read `.image-slots.state.json` and `.ft-doc.state.json` (run_script `readFile`).
2. Collect the slot ids from the pasted `ops` (kinds `photo-*` and `crop`). For each id:
   - Sidecar value has a data URL `u` → decode the base64 into a file
     `photos/<id>.webp` (or .png/.jpeg per the data URL mime) and set
     `FT_SEED.slots[id] = { src: 'photos/<id>.webp', w, h, s, x, y }` (s/x/y from the sidecar value, default 1/0/0).
     run_script: `const b64 = u.split(',')[1]; const bin = atob(b64); const arr = new Uint8Array(bin.length); ...; await saveFile(path, new Blob([arr], {type: mime}))`.
   - Sidecar value is `{del:1}` (or the op is `photo-remove` with no sidecar value) → DELETE
     `FT_SEED.slots[id]` and the old `photos/<id>.*` file.
   - Framing-only sidecar value (`{s,x,y}`, no `u`) on a seeded slot → merge s/x/y into the seed entry.
3. Exclusion ops (`tree-hide`/`tree-show`): apply the CURRENT override from `.ft-doc.state.json`
   `excluded[key]` to `FT_SEED.excluded` (add or remove the key).
3b. Field-edit ops (`kind:'edit'`): the pasted set carries an `edits` snapshot (`ekey → value`).
   Bake each into the source file:
   - `p:<id>:<path>` / `m:<id>:<idx>:<field>` / `u:<unionId>:<field>` → edit `family-data.js`
     (person fields, that person's `milestones[<idx>]`, or the union entry).
     `p:<id>:parents` (array value) rewrites that person's `parents` list.
   - `w:<evId>` → edit `world-events.js`: value true → remove `off:1` from that event; value
     false → add `off:1`.
   After baking, remove the baked `edits` entries from `.ft-doc.state.json` (`doc.edits`) along
   with their ops.
3c. Structural ops (`person-add`/`person-remove`/`union-add`/`union-remove`/`side-add`/`side-remove`):
   the pasted set carries a `struct` snapshot — `{ people, delPeople, unions, delUnions, sides, delSides }`.
   Bake into `family-data.js`:
   - `struct.people[id]` → insert the person object into the `people` array (place by `gen`,
     formatted like its neighbours). New-person field edits may also appear in `edits` — apply

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aviranrevach/my-family-story](https://github.com/aviranrevach/my-family-story) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
