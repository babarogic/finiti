# Benchmark Matrix — Developer Handoff

Finiti Legal v1, Benchmark feature. This document covers the Benchmark matrix screen only: data contract, rendering rules, interactions, and dependencies.

## 1. What this screen is

A comparison table. The user's filing sits beside 2 to 3 peer filings. Rows are individual risk factors. The user scans across each row to compare language, then decides per row whether to keep their version, draft a revised one, or draft a new one.

This is a read-and-decide surface. Finiti never edits the source document. The output of a decision is a draft the user takes back to Word.

## 2. Data model

The backend delivers the matrix fully pre-computed. The frontend does not perform alignment, diffing, or verdict logic. It renders what it is given and reports user decisions back.

```
BenchmarkRun
  id
  filingVersionId        which version of the user's filing this ran against
  userCompany            { ticker, name }
  peers[]                2-3 peers: { id, ticker, name, sourceFilingType, sourceFilingDate }
  sections[]

Section
  id
  title                  e.g. "Risks related to financial position and capital needs"
  rows[]

Row
  id
  verdict                enum: ORIGINAL_OK | REVISE | COMBINE | GAP | UNIQUE
  cells[]                one per company column, fixed order: user first, then peers
  reasoning              AI text. null when verdict = ORIGINAL_OK
  status                 enum: OPEN | KEPT | DRAFTED | DISMISSED | CONFIRMED
  selectedBasisCellId    which peer cell the user picked as the draft basis. nullable

Cell
  companyId
  text                   full risk factor text, or null
  isEmpty                true means this company has no comparable risk factor
  source                 { filingType, item, page, edgarUrl }. null when isEmpty
```

### The isEmpty flag carries different meaning by column

An empty cell on a peer column means the peer has no comparable risk factor.
An empty cell on the user column means a gap, the user is missing this disclosure.

Same flag, the column position changes the meaning. The amber highlight applies only to an empty user cell.

## 3. Verdict types

The verdict is delivered by the backend. The frontend renders it and shows the matching action set. It does not compute verdicts.

| Verdict | Meaning | Actions shown |
|---|---|---|
| ORIGINAL_OK | User language is fine | None. Thin dashed strip only. |
| REVISE | A peer's language is stronger | Keep original, Draft using [peer], Dismiss |
| COMBINE | Two peers each have a useful part | Keep original, Draft combined, Dismiss |
| GAP | Peers disclose it, user does not | Draft from [peer], Dismiss |
| UNIQUE | User-only, no peer to compare | Confirm |

## 4. Row rendering rules

1. ORIGINAL_OK rows render compact: three text cells, no reasoning, and a thin dashed strip reading "Original language OK, no action needed." No action bar.
2. All other rows render the full action bar below the three cells: AI reasoning text, verdict badge, decision buttons.
3. Row heights are natural and ragged. Cells are not forced to equal height. Cell text is never truncated.
4. An empty cell renders italic muted text ("Not disclosed", "No comparable risk factor"). An empty user cell additionally gets the amber background.
5. Every non-empty cell shows a source link under the text. The link is an EDGAR deep link from `cell.source.edgarUrl`.
6. Sections render as a header row showing the section title and a risk factor count.

## 5. Interactions

### Select a basis cell
Clicking a peer cell sets `selectedBasisCellId` for that row and applies the selected style: green background, green border, filled radio. Only one cell per row can be selected. The user's own cell is not selectable as a basis. Selecting a cell updates the primary button label to name that peer, for example "Draft using GANX".

### Keep original
Sets `status = KEPT`. The row collapses to the resolved state. The row leaves the active progress count.

### Draft action
Sets `status = DRAFTED`. Triggers a Draft run using `selectedBasisCellId` text as the precedent and the filing's vault as the fact source. The Draft target screen is not yet designed. For now, emit a `draftRequested` event carrying the row id and the basis cell id. The row then shows the resolved state: "Draft created using [peer] language" with an Undo control.

### Dismiss
Sets `status = DISMISSED`. The row collapses and leaves the active count. Dismiss is different from Keep original: Keep original means the user reviewed the suggestion and their current language stands. Dismiss means the row is not relevant to this filing at all. Both leave the active queue. The distinction matters for analytics and for round-over-round re-run behavior, so the two statuses must be stored separately.

### Confirm
UNIQUE rows only. Sets `status = CONFIRMED`. Acknowledges that the user-only risk factor is intentional.

### Undo
Available on any resolved row. Reverts status to OPEN and restores the row to its working state.

## 6. Filter and progress

Filter chips: All, Revise, Combine, Gap, Original OK. Each chip shows a count. Filtering hides non-matching rows. If a section has no visible rows after filtering, hide that section header too.

Progress counter reads "X of Y actioned."
Y is the count of rows where verdict is not ORIGINAL_OK. These are the rows that need a decision.
X is the count of rows in that set with status in KEPT, DRAFTED, DISMISSED, or CONFIRMED.
ORIGINAL_OK rows are never included in either number.

## 7. Export

Two buttons, Export Excel and Export Word. Export reflects the current state of the matrix: every row, its cells, verdict, reasoning, the user's decision, and any draft text generated.

Export is always the full matrix. It is not filtered by the active chip. This is a hard requirement from client feedback: eBay needs the Excel matrix, Morgan Lewis needs Word, K&E needs downloadable reports.

## 8. Key dependency and risk

Row alignment is the backend's responsibility and it is the highest-risk part of this feature. The backend decides which peer risk factor corresponds to which user risk factor, produces the row groupings, and assigns the verdicts. If alignment is wrong, for example a peer clause mapped to the wrong user clause, the entire matrix is misleading and the frontend cannot detect or correct it.

Action: confirm with the backend team that alignment quality is being measured before this screen reaches pilot.

## 9. Not in scope for this screen

- Editing source document text. This never happens in Finiti.
- The Draft canvas itself. The Draft action only emits a `draftRequested` event for now.
- Peer data refresh. Peer filings refresh quarterly and are handled outside this screen.

## 10. Reference

- Matrix structure is modeled on a real lawyer artifact: `APLT_-_Risk_Factor_Comparison__2025_10-K_.docx` (Quinn). Each section holds multiple risk factor rows; each row aligns the closest-matching language across companies; a reasoning column carries the per-row verdict.
- Provenance on every output, scannable not exhaustive output, and Excel/Word export are the three non-negotiables across all client feedback.
