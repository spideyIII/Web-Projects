# U035 test seed data

Throw-away rows so the U035 screen can be exercised end to end. Every value starts with
`[TEST]`, so the rows are easy to find and delete when real data is loaded.

All values are **made up for testing**. The DRF types (A / B) and the subtask wording are
placeholders, not the real URO MRC 035 content.

## Load order
1. `U035_Equipment_Guide_Items_TEST.csv` -> table **U035 Equipment Guide Items**
2. `U035_Impact_Requirements_TEST.csv`   -> table **U035 Impact Requirements**
3. (optional) Reference images - see below.

In Power Apps / Dataverse: **Tables -> (table) -> Import -> Import data from Excel / CSV**, then map
each CSV column to the column with the same display name. Two-option columns (`IsSelectable`) accept Yes / No.

## What the rows are for
| Test | Rows that cover it |
|---|---|
| Worklist picker and subtasks | Two parents (1.A TORPEDO TUBES, 1.B HULL VALVES), 4 and 2 subtasks, MOD 70 |
| DRF keyword filter | Subtask 1.A.(2) "slide valve", 1.A.(3) "breech door", 1.A.(4) "muzzle door" each match one DRF title |
| Parent-level keyword rule | Any 1.A subtask not matching a phrase still falls back to "torpedo tube" (parent title) |
| Fallback to all DRF forms | 1.B subtasks match no keyword, so every MOD 70 DRF form is listed |
| Wall Calculation, Type A | `[TEST] TORPEDO TUBE SLIDE VALVE DEFECTS`, `[TEST] BREECH DOOR DEFECTS` |
| Wall Calculation, Type B + Amount Below Min | `[TEST] TORPEDO TUBE DEFECTS ...`, `[TEST] MUZZLE DOOR DEFECTS`, `[TEST] HULL VALVE DEFECTS` |

## Things to set up by hand
- **URO Impact Item**: the app opens the `URO Impact Items` record in the `recordId` parameter. Its
  Owning Team Code must be `OWN-WORKING` (or the screen is read-only) and its ship must resolve to **MOD 70**
  (a Virginia class ship). Without a `recordId` the app falls back to `DART-000326`.
- **Other required columns**: if either table has required columns not in the CSV (for example on
  `U035 Impact Requirements`), give them any value; this screen does not read them.
- **Reference images (Images tab)**: add two rows to `Impact URO Reference Images` with any picture in
  `Reference Image`, a `Display Label`, `Display Order` 1 and 2, `URO MRC` = 035, `URO Mod` = 70.
- **MOD number text**: the screen accepts `70` in the Mod columns. For MOD 1 it also accepts `01`.

## Cleaning up
Filter each table on the primary name column containing `[TEST]`, select all, delete. Results rows made
while testing have `Parent Code` = `1.A` / `1.B`; delete those too (filter `U035 Inspection Results` on
`Parent Code`), then delete the matching rows in `U035 Inspection DRF Results`.
