---
title: "Cleanup Plan"
tags: [moc, housekeeping]
type: note
status: draft
created: 2026-10-05
updated: 2026-10-05
---
# Cleanup Plan

Moves and renames are best done inside Obsidian (File explorer drag, or `F2`) so links update. Nothing below has been moved yet.

## 1. Attachments
- 41 loose files sit in the vault root: ~34 `WhatsApp Image ...jpeg`, 5 `Pasted image` / `image*.png`, plus 2 `cds_full_pricer*.html`. 29 are embedded in notes; 12 are not referenced anywhere (check before deleting).
- Move all of them into `attachement/` (select in the file explorer and drag). Embeds follow automatically.
- Set Settings → Files and links → *Default location for new attachments* to `attachement` so new pastes stop landing in the root. (The `attachment-management` plugin is already installed and can automate this.)
- Consider fixing the folder spelling `attachement` → `attachments` via rename in Obsidian.

## 2. Stray root files
- `Untitled.md`, `Welcome.md`: empty/default; delete or fold into [[Home]].
- `cds_full_pricer.html` and `cds_full_pricer 1.html`: likely duplicates; keep one, move into `Projects/` or `attachement/`.

## 3. Empty and stub notes
See the *Stubs to fill in* view in [[All Notes.base|All notes]]. Zero-byte notes include `9 - Math needed`, `PIVAUTOFIT`, `PNL Explain`, `Parametric Build Methods` and three `irhelper.cpp` pages. Also `Misc/Untitled 1`–`6`, which can be merged or removed.

## 4. Duplicates and naming
- Three notes named `Untitled` (root, `Misc`, `Fin - DT_IR Investigations`): rename to something descriptive.
- `Interest Rate Spread Curve` vs `Interest rate spread curve 2`: likely merge.
- `HYBRID_FORWARD` (Curve/BuildMethod) vs `BuildMethod - Hybrid Forward` (IR): likely the same topic; link them or merge.
- Folder `FIn - Math` has a typo (`FIn`): rename to `Fin - Math`.

## 5. Suggested folder layout (optional)
Folders are coarse here; MOCs and tags do the navigating. Current top-level folders are fine. If you want a numeric scheme: `00-inbox/`, `10-notes/`, `20-mocs/` (current `MOCs/`), `30-projects/`, `90-assets/`, `99-archive/`.

## 6. `Rubbish/` and git
`Rubbish/tmp` sits outside the vault, in the same git repo. Your git tree already shows uncommitted changes from before this pass; review with `git diff` before committing.
