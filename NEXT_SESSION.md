# Curriculum Planner — Next Session Handoff
**Date written:** 24.07.26 | **Current version:** 24.07.26c

## Repo
`~/Desktop/claude_code/curriculum-planner/index.html` → `https://wallscourtfarm.github.io/curriculum-planner/`
gh CLI authenticated as `imcl75` for `wallscourtfarm` org.

## State of the tool right now
- Single-file static HTML, all data embedded, saves to localStorage per year group
- 6 year groups (Y1–Y6), WFA class names: Beech/Willow/Acer/Maple/Hazel/Elm
- All enquiry cards populated (Y1–Y6, 113 total)
- All 11 wider curriculum subjects populated for all 6 year groups (from CLF docs + Y5 reference HTML)
- 39-week Year at a Glance grid, clickable cells scroll to enquiry card
- Week labels show T2·W1–2 format (local weeks)
- **Bug fixed (24.07.26c):** Y6 wider curriculum data was displaced out of YEAR_DATA and into the exportJSON() function body, causing a JS syntax error that broke the whole page. Fixed by repositioning the Y6 wider block and closing exportJSON() correctly.
- **Excel importer added (24.07.26c):** 3-step wizard accessible via "Import Excel" button in nav

---

## Excel Importer (Priority 1 — DONE)

### What was built
An in-browser Excel import panel accessible via the green "Import Excel" button in the nav bar.

**3-step wizard:**
1. **Select file**: Year group picker (pre-selects current year) + drag-drop / browse file picker. Loads SheetJS from CDN (`https://unpkg.com/xlsx/dist/xlsx.full.min.js`) lazily on first open.
2. **Map columns**: Auto-detects common column names (Term, State of Being, Key Question, Week Start/End, Overview, Linked Text/Book, Genre, Writing Outcome, Content/Skills). Column mapping saved to `localStorage` key `wfa_cp_import_mapping` so repeat imports don't re-prompt.
3. **Review & import**: Diff table showing status (changed / new / unchanged / ambiguous) per enquiry row. "Changed" rows show field-by-field old → new with checkboxes to include/exclude. "New" rows show the proposed key question and offer to add. "Ambiguous" rows (multiple enquiries match same term + SOB) are flagged but not auto-applied.

**Matching logic:** term number + stateOfBeing (finds the enquiry in YEAR_DATA with matching term and State of Being). One-to-one match → diff. Multiple matches → ambiguous warning. No match → flagged as new.

**Fields imported:** keyQuestion, weekStart, weekEnd, overview, linkedText, genre, writingOutcome, contentSkills
**Fields NOT changed by import:** id, term, stateOfBeing (structural)

**Known Excel format (from CLF planning files):**
Columns: Year Group | Term | State of Being | Week Start | Week End | Key Question | Overview | Linked Text / Book | Genre | Writing Outcome

### What to test with a real file
- Ask Innes if teachers have produced their 2025-26 Excel files yet (expected during summer 2026 holidays)
- The real file path was `~/Downloads/` — check for `.xlsx` files there
- Test with a real file to verify column auto-detection works on the actual format

### Known limitation
"New" enquiries added via import are stored in `localStorage` under `ld.newEnquiries[]` but the `renderYear()` function does NOT currently display them — the render loop iterates over `YEAR_DATA[k].enquiries` (the embedded defaults), not the localStorage new entries. **If adding new enquiries via import is important, this needs a render-side fix to merge `ld.newEnquiries` into the displayed list.**

---

## Priority 2 — Curriculum Coverage Tracker

### Why
Teachers need to confirm that across their enquiries they are covering what the CLF curriculum documents require — specifically for Science, Geography, History (the enquiry-heavy subjects).

### What to build
A **Coverage** tab per year group showing:
- For each enquiry, a checklist of CLF curriculum expectations that should be covered
- Green = covered (teacher has ticked it), Amber = partially noted, Red = not yet addressed
- Summary view: "Y4 — 14/22 Science objectives covered across enquiries"

### Data sources
CLF curriculum documents are at: `~/Downloads/Curriculum 26-27/New Cucciculum Docs/`
Key files: `CLF Primary Curriculum Scientists July 2026.md`, `CLF Primary Curriculum Geographers July 2026.md`, `CLF Primary Curriculum Historians July 2026.md`
Also `CLF Primary Curriculum Writers July 2026.md` (Writing coverage linked to enquiries already)

### Implementation approach
- Extract per-year-group coverage expectations from the CLF .md files (agent task — read all 3 subject docs, pull out Y1–Y6 objective lists)
- Embed as `COVERAGE_EXPECTATIONS` JS constant in the HTML
- Store tick states in localStorage under `wfa_cp_Y4.coverage` etc.
- Render as a collapsible panel below each enquiry card: "CLF objectives for this enquiry" with checkboxes
- Aggregate summary bar at the top of each year group

---

## Priority 3 — Term Zoom-In (Gantt week view)

### Why
The 39-week grid shows enquiries at term level. Teachers want to see the within-term week-by-week picture.

### What to build
Click on a term header in the 39-week grid → expand an inline Gantt panel showing:
- Rows: each enquiry active in that term
- Columns: each week in the term (T1=8 cols, T2=7 cols, etc.)
- Enquiry bars span weekStart–weekEnd
- Subject/wider curriculum rows below the enquiry bars
- Colour-coded by State of Being

---

## Priority 4 — Drag and Drop

Reorder enquiries within a term, and move wider curriculum content between terms, without editing raw text.

---

## Version history
| Version | Date | What changed |
|---------|------|-------------|
| 23.07.26d | 23.07.26 | Initial build matching y5-long-term-plan.html reference |
| 23.07.26e | 23.07.26 | Y5 wider curriculum injected; intermediate bump |
| 24.07.26a | 24.07.26 | Fixed week labels (T2·W1–2 local format); grid click-to-scroll; Y5 wider data in YEAR_DATA |
| 24.07.26b | 24.07.26 | CLF wider curriculum data for Y1–Y4 and Y6 (all 11 subjects) |
| 24.07.26c | 24.07.26 | Fixed JS syntax error (Y6 wider data displaced from YEAR_DATA); added Excel importer (3-step wizard) |

---

## Useful file locations
| File | Purpose |
|------|---------|
| `~/Desktop/claude_code/curriculum-planner/index.html` | The tool |
| `~/Downloads/Curriculum 26-27/New Cucciculum Docs/` | All CLF curriculum .md and .pdf files |
| `~/Downloads/y5-long-term-plan.html` | Original Y5 reference design |

## Key JS constants in index.html
- `YEAR_DATA` — all enquiry + wider curriculum defaults per year group (Y6 wider now correctly embedded)
- `WIDER_DEF` — 11 wider curriculum subjects with row labels, colours, icons
- `TERM_WK = [8,7,6,6,5,7]` — weeks per term
- `TERM_GS = [0,8,15,21,27,32]` — global week offsets (used internally, NOT for display)
- `IMP_FIELDS` — importable fields list (used by importer)
- localStorage keys: `wfa_cp_Y1` … `wfa_cp_Y6`, `wfa_cp_import_mapping`
