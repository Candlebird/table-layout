# Table Layout — notes

Single file app: `index.html`. No build, no dependencies.

## Running

- Double-click `index.html` to open it in a browser, **or**
- serve the folder: `python -m http.server 8765` → http://127.0.0.1:8765/index.html

The layout autosaves to the browser's localStorage (key `tableLayout.v1`). Use **Save** to
download a `.json` file you can **Load** later or on another machine.

## How to use

| Action | How |
|---|---|
| Add a table | Pick a seat count (6–10) in the toolbar → **+ Add table** |
| Move a table | Drag its tabletop (snaps to a 20px grid) |
| Rename a table | Double-click the tabletop name; Enter saves, Esc cancels |
| Resize / delete a table | Seat dropdown / ✕ on the tabletop |
| Import names | **Import names** → one name per line, optionally into a group |
| Seat someone | Drag a name from the list onto a seat |
| Move / swap | Drag a seated person onto another seat (occupied seats swap) |
| Unseat | Double-click a seated name, or drag it back onto the sidebar |
| Edit a person | Right-click (or double-click in the list) → name, group, delete |
| Tag a group | Drag a name onto a group row in the sidebar |
| Filter by group | Click a group row (click again to clear) |
| Auto-fill | **Auto-fill** seats everyone in the list, keeping groups together |
| Pan / zoom | Drag empty floor / mouse wheel, − + Fit buttons |
| Undo / redo | Buttons or Ctrl+Z / Ctrl+Y (Ctrl+Shift+Z) |
| Export | **Export CSV** (Table, Seat, Name, Group) or **Print** (floor plan + guest list) |

## Decisions I made on your behalf ("decide, note it")

- **Dropping a list person onto an occupied seat** sends the occupant back to the list; a
  seated person dropped onto an occupied seat swaps (as you chose).
- **Shrinking a table** keeps people in their seats. People in removed seats move into any empty
  seats left, and only those who still don't fit go back to the list.
- **Deleting a table** doesn't ask for confirmation, since Undo covers it. **Clear all** still asks.
- **Import** appends and skips duplicate names (case-insensitive). **Renaming** a person
  to an existing name is allowed (two real guests can share a name).
- **Groups**: one group per person. Colors cycle through a 10-color palette and can be changed
  with the swatch. Deleting a group ungroups its members (people are kept). Ungrouped seats are gray.
- **Auto-fill**:
  1. It never moves anyone already seated.
  2. Groups first, largest first. Anyone whose group already has someone seated joins that
     table first (all groups get this pass before any group claims new tables).
  3. The rest of a group goes to the table with the fewest free seats that still fits the
     whole group (ties go to tables with fewer people from other groups). If no table fits,
     the group is split across the emptiest tables.
  4. Groups sit in adjacent seats, starting next to their existing members.
  5. Ungrouped people fill the remaining seats, going through tables in name order.
  6. Anyone without a seat stays in the list, and a message says how many more seats are needed.
- **Undo history** keeps 200 steps in memory only; it resets when the page reloads.
- **Group filter and search** are not saved. Zoom/pan is saved (file and autosave) but isn't part of undo.
- **The list** is sorted alphabetically, with numbers in natural order ("Guest 2" before "Guest 10").
- **New tables** are placed in the first free grid spot near the view center, without overlapping.
- **Saved file format**: `{app: "TableLayout", version: 2, people, groups, tables, view}`.
  When loading, unknown people and anyone listed in two seats are dropped.
- **File names** use the local date: `table-layout-YYYY-MM-DD.json`, `guest-list-YYYY-MM-DD.csv`.
  The CSV starts with a UTF-8 BOM so Excel shows accented names correctly.
- Desktop mouse only (no touch support), as you chose.

## Tested in Chrome

Import (group, duplicates, blank lines, CRLF), adding 6/8/10-seat tables, list→seat,
replacing an occupied seat, seat↔seat moves, dragging tables, renaming tables, auto-fill with
groups, undo/redo, shrinking a table, Save→Load round trip, CSV, print view, edit dialog with a
new group, drag-to-group tagging, wheel zoom, pan, Fit, autosave across reload. No console errors.
