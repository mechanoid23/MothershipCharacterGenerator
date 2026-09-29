# CLAUDE.md — MothershipCharacterGenerator

Module ID: `mothership-character-creator`
Manifest: `https://raw.githubusercontent.com/mechanoid23/MothershipCharacterGenerator/master/module.json`

## Pack format

This module uses **LevelDB binary format** for all compendium packs (folders under `packs/`). You cannot edit the contents as plain text. To modify macros or journals:

```bash
# Unpack to editable JSON
fvtt package unpack -n character-generator-macros \
  --in packs/ --out /tmp/mcg-src/macros --compendiumType Macro

# Edit the JSON files in /tmp/mcg-src/macros/

# Repack back to LevelDB
fvtt package pack -n character-generator-macros \
  --in /tmp/mcg-src/macros --out packs/ --compendiumType Macro
```

## Releasing updates

1. Bump `"version"` in `module.json`
2. Update `"download"` to the new release tag URL, e.g. `releases/download/v1.0.3/mothership-character-creator.zip`
3. Commit and push to `master`
4. Zip from the repo root (module.json must be at the zip root):
   ```bash
   zip -r /tmp/mothership-character-creator.zip . --exclude "*.git*" -q
   ```
5. Create the GitHub release:
   ```bash
   gh release create v1.0.3 /tmp/mothership-character-creator.zip \
     --repo mechanoid23/MothershipCharacterGenerator --title "v1.0.3"
   ```

Always use a tagged release (not `archive/refs/heads/master.zip`) — the branch archive is cached by GitHub's CDN and may serve stale content.

## Choose Skills macro

The **Choose Skills** macro (`mcgChooseSkillsA`, sort 1200000) handles two skill selection flows after the class macro runs:

- **`choose_skill_and`**: free picks from the class's `common_skills` pool. Opens a checkbox dialog for each rank bucket (Trained / Expert / Master). `expert_full_set` / `master_full_set` draw from the full skill list across both PSG and RWC compendiums.
- **`choose_skill_or`**: one dialog per group with one button per option. After picking an option, a follow-up checkbox dialog lets the player pick specific skills from that option's pool.

Skills are resolved via `fromUuid()` and added to the actor with `createEmbeddedDocuments("Item", [...])` with the chosen rank set on `system.rank`.

The macro reads class data from `fvtt_mosh_1e_rwc.items_classes_1e` first, then falls back to `fvtt_mosh_1e_psg.items_classes_1e`.

## Dependencies

- `fvtt_mosh_1e_psg` — required; provides base classes, skills, items, and rolltables
- Does NOT require `fvtt_mosh_1e_rwc` at runtime (RWC content is handled by MothershipCharacterGeneratorRWC)
