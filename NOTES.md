# Table Layout — notes

A single-file app in `index.html`, with no build step and no dependencies.

## Running

- Double-click `index.html` to open it in a browser, **or**
- serve the folder with `python -m http.server 8765` and open http://127.0.0.1:8765/index.html

The layout autosaves to the browser's localStorage (key `tableLayout.v1`). **Save** downloads a
`.json` file that you can **Open** later or on another machine.

## How to use

| Action | How |
|---|---|
| Add a table | Pick a seat count (6–10) in the toolbar, then **+ Add table** (or press T) |
| Add the couple's table | **♥ Sweetheart** (or H) adds a long table with two seats side by side. There is only one; pressing it again shows the existing one |
| Move a table | Drag its tabletop. It snaps to a 20px grid, and if you drop it on another table it slides to the nearest free spot |
| Tidy up | **Arrange** (grid icon, or G) puts all tables in a neat grid in name order |
| Rename a table | Double-click the table's name. Enter saves and Esc cancels |
| Resize or delete a table | Hover the tabletop to show the seat dropdown and trash button |
| Import names | **Import** (or I). One name per line, optionally into a group. Ctrl+Enter imports |
| Seat someone | Drag a name from the list onto a seat |
| Move or swap | Drag a seated person onto another seat. If that seat is taken, the two people swap |
| Unseat | Double-click a seated name, or drag it back onto the sidebar. The eraser button next to **Auto-fill** unseats everyone |
| Cancel a drag | Press Esc while dragging |
| Edit a person | Right-click a name (or double-click it in the list) to change the name or group, or to delete |
| Tag a group | Drag a name onto a group row in the sidebar |
| Filter by group | Click a group row (click again to clear). Hover a group to light up its seats |
| Group priority | Drag a group row up or down. The top group gets the tables closest to the sweetheart table |
| Sit near | Click a group's pencil and pick **Sit near**. Auto-fill puts the two groups at neighboring tables |
| Find someone | Type in the search box (or press /). Seats that don't match are dimmed, and the seated matches are listed; click one to fly to that seat |
| Auto-fill | **Auto-fill** (or A) seats everyone in the list, keeps groups together and places them by priority, working outward from the sweetheart table |
| Pan and zoom | Drag the empty floor, or scroll to zoom. Shift+scroll pans sideways and touchpad scrolling pans. The − + and Fit buttons (F) also work, and dragging near an edge pans the floor |
| Undo and redo | The toolbar buttons, Ctrl+Z / Ctrl+Y (Ctrl+Shift+Z), or **Undo** in the message that pops up |
| Export | **CSV** (Table, Seat, Name, Group) or **Print** (floor plan plus guest list by table) |

### Keyboard shortcuts

T add table · H sweetheart table · I import · A auto-fill · F fit · G arrange · / search · + − 0 zoom ·
Ctrl+Z / Ctrl+Y undo/redo · Ctrl+S save · Ctrl+O open · Ctrl+P print · Esc cancel/close · ? tips.
The single-letter shortcuts are ignored while you're typing in a text field.

## Decisions I made on your behalf ("decide, note it")

- **Dropping a person from the list onto an occupied seat** sends the occupant back to the list.
  **Dropping a seated person onto an occupied seat** swaps the two, as you chose.
- **Shrinking a table** keeps people in their seats where possible. People in removed seats move
  into any empty seats left, and only those who still don't fit go back to the list.
- **Deleting a table** doesn't ask for confirmation, because the message offers Undo. **Clear all**
  asks first and can also be undone.
- **Import** appends and skips duplicate names (case-insensitive). **Renaming** a person to a
  name that already exists is allowed, since two real guests can share a name.
- **Groups**: each person is in at most one group. Colors cycle through a 10-color palette, and you
  can change one with the swatch. Deleting a group ungroups its members and keeps the people.
  Ungrouped seats are gray.
- **Auto-fill** works from where the tables actually are:
  1. Anyone already seated stays where they are. The sweetheart table is never auto-filled;
     seat the couple by hand.
  2. The "center" is the sweetheart table. Without one, it is the middle of all the tables.
  3. If someone in a group is already seated, the rest of the group joins those tables first.
     Every group gets this pass before any group claims a new table.
  4. Groups are then placed in sidebar order, so the top group gets the tables closest to the
     center. A group's "sit near" partner is placed right after it.
  5. Each group aims at where it already sits, else at its seated "sit near" partner, else at the
     center. It takes the best **cluster of neighboring tables** with enough free seats. The best
     cluster is close to that aim, wastes few seats, is tightly packed, and has few other groups'
     guests. A group that fits at one table normally gets one table.
  6. Within the cluster, the closest table fills first, and the group sits in neighboring seats.
  7. Ungrouped people fill the remaining seats, closest tables to the center first.
  8. Anyone left without a seat stays in the list, and a message says how many more seats are needed.
- **Sit near** is one choice per group, and either side of the pair counts. Deleting a group
  clears any "sit near" that pointed to it.
- **The sweetheart table** has 2 seats, can't be resized, and counts toward the seat totals.
  It comes first in Arrange, the CSV and the printed lists. It collides with other tables by its
  rectangle, not a circle.
- **Reordering groups** can be undone and is saved in the file. Esc during the drag puts the row back.
- **Clear seats** (eraser next to Auto-fill) empties every seat, the sweetheart table's too, and
  keeps tables, guests and groups. It doesn't ask first, because the message offers Undo. It is
  grayed out when nobody is seated. It has no keyboard shortcut, so it can't be pressed by accident.
- **Undo history** keeps 200 steps in memory and resets when the page reloads. Zoom and pan
  are saved but are not undo steps.
- **The group filter and search** are not saved.
- **The list** is sorted alphabetically, with numbers in natural order ("Guest 2" before "Guest 10").
- **New tables** go in the first free grid spot near the center of the view. If that spot is off
  screen, the view glides to it.
- **Arrange** sizes each column to its widest table and each row to its tallest table, uses
  about as many columns as the window's shape suits, and starts at the current layout's top-left
  corner. It can be undone.
- **Saved file format**: `{app: "TableLayout", version: 2, people, groups, tables, view}`.
  When a file is loaded, seats that point to unknown people are emptied, and anyone listed in two
  seats keeps only the first.
- **File names** use the local date: `table-layout-YYYY-MM-DD.json`, `guest-list-YYYY-MM-DD.csv`.
  The CSV starts with a UTF-8 BOM so Excel shows accented names correctly. Names that start with
  `= + - @` get a leading `'` so spreadsheets don't run them as formulas.
- **Printing** uses a print-only page: an SVG floor plan scaled to fit, then the guest list in
  three columns. Long names wrap (also after a hyphen) and shrink to fit their seat, on screen
  and in print. Extremely long names may still be cut off on a seat; hover shows the full name.
- **Accessibility**: icon-only buttons have screen-reader labels, dialogs keep Tab inside them
  and give focus back to the button that opened them, every control shows a focus ring when you
  use the keyboard, and animations are turned off when the system asks for reduced motion.
- **Desktop mouse only**, with no touch support, as you chose.

## Tested in Chrome

- Import:
  - into a group;
  - duplicates, blank lines and CRLF line endings;
  - 200 generated names with accents and long hyphenated surnames.
- Tables:
  - adding 6-, 8- and 10-seat tables;
  - 28 tables at once;
  - dragging (including a fast drag), and dropping one table on another;
  - renaming, shrinking, deleting with Undo;
  - Arrange.
- Seating:
  - list→seat, replacing an occupied seat, seat↔seat swaps;
  - Esc while dragging;
  - auto-fill with groups (214 people seated in 6ms);
  - placement-aware auto-fill: top group nearest the sweetheart table, "sit near" partner on
    the neighboring table, reordering groups changes the result.
- Sweetheart table: adding, the second-press message, dragging it onto a table, print.
- Clear seats, then Undo.
- Group drag-reorder, the "sit near" dialog, deleting a group that is someone's partner, Undo.
- Undo/redo, Save→Open round trip, CSV, print view with 28 tables.
- Edit dialog with a new group, drag-to-group tagging.
- Search:
  - dimming;
  - seated matches and fly-to;
  - group hover highlight.
- Views: wheel zoom, pan, Fit.
- Keyboard: shortcuts, dialog focus and Tab trapping.
- Autosave across reloads.

No console errors in any of these tests.
