# Filing Dashboard — Developer Handoff

Finiti Legal v1. This document covers the Filing Dashboard screen: what it is, the data contract, every state and edge case, and the rules for each.

## 1. What this screen is

The filing dashboard is the home of a single filing. A filing is a versioned project. The user uploads document versions, and runs workflows (Review, Benchmark, Draft, Verify) against a version. The dashboard shows the activity history: every run, grouped by the document version it ran against, newest first.

Core principle from the product model: a filing is a versioned project, a run is one execution of a workflow pinned permanently to one version. Runs are immutable records. Finiti never edits the source document. The user revises offline in Word and uploads the result as the next version.

## 2. Layout

Three regions, top to bottom:

1. Top bar: back, filing name, filing-type badge, optional lock icon, then Export (icon button), Upload new version, New run.
2. Meta bar: version count, last activity, contributor avatars, Filter link. Optional banner sits directly under this bar when a filing-level condition applies (stale, deleted version, read-only).
3. Body: version groups. Each group is a version header (dot, label, date, current badge) followed by an indented timeline of run rows.

## 3. Data model

```
Filing
  id
  name
  filingType             10-K | 10-Q | 8-K | 20-F | 6-K ...
  accessLevel            OWNER | EDITOR | VIEWER
  versions[]             newest first
  contributors[]         distinct users across all runs

Version
  id
  label                  v1, v2, v3 ...
  uploadedAt
  isCurrent              true for the newest, non-deleted version
  status                 ACTIVE | DELETED
  runs[]                 newest first

Run
  id
  workflow               REVIEW | BENCHMARK | DRAFT | VERIFY
  actor                  { id, name, initials, avatarColor }
  description            short summary of what was done
  createdAt
  status                 IN_PROGRESS | COMPLETE | FAILED
  progress               0-100, only meaningful while IN_PROGRESS
  etaSeconds             nullable, only while IN_PROGRESS
  failureReason          nullable, set when status = FAILED
  isStale                true when the run's version is no longer current
  sourceAvailable        false when the run's version was deleted
```

## 4. Run row rendering

A run row shows: actor avatar, description, timestamp, workflow chip. The workflow chip is colored per workflow (Review blue, Benchmark purple, Draft green, Verify amber).

- Description clamps to 2 lines maximum. The backend should generate short summaries; the clamp is the safety net. No "show more"; the run detail is one click away.
- When a single contributor owns the whole filing, the actor name is not bolded in the description (there is nobody to disambiguate). With multiple contributors, the actor name is bolded.
- Clicking a row opens that run.

## 5. States

### 5.1 Empty (filing uploaded, no runs)
Version spine shows v1 as current. Body is a single centered prompt: icon, "Your filing is ready", one line of explanation, and four workflow entry buttons (Review, Benchmark, Draft, Verify). Picking one starts that run.

### 5.2 Loading
Top bar renders immediately (filing name and type are known from navigation). Meta bar and run list render as pulsing skeletons until data resolves. Do not block the whole screen.

### 5.3 Active filing with runs (happy path)
Versions listed newest first. The current version is fully expanded. Older versions are dimmed; their runs carry an "older version" label. Versions beyond the second collapse to a one-line summary. When more than roughly 4 versions exist, collapse everything past the current and the one prior into a single "N older versions (M runs)" divider that expands on click.

### 5.4 Run in progress
A run with status IN_PROGRESS renders as a live row at the top of its version group: spinner instead of avatar, an ETA line ("Running... about 20s left") derived from `etaSeconds`, a thin progress bar from `progress`, and a Cancel control.

The row updates in place. The user can navigate away and back; the row continues to reflect live state. On completion the row becomes a normal COMPLETE run with no transition fanfare. Poll or subscribe for updates; do not require a manual refresh.

### 5.5 Run failed
A run with status FAILED keeps its row in place. Avatar slot shows a warning icon. The description states the failure plainly and must reassure that nothing was changed, for example "Failed: EDGAR did not respond. Your filing was not changed." A Retry control sits on the row. A failed run never silently disappears.

### 5.6 Stale runs after new version upload
When a new version is uploaded, runs on prior versions are not deleted. They get the "older version" label and their version block dims. An info banner under the meta bar explains: "vN was just uploaded. Runs below ran on older versions and may no longer reflect the current filing." The new version shows its own empty state ("No runs on this version yet" with a Start a run link).

### 5.7 Filter active
Filter chips: All, Review, Benchmark, Draft, Verify. Selecting one shows only runs of that workflow. Version headers with no matching runs are hidden while the filter is active.

### 5.8 Empty filter result
When a filter matches no runs, show a centered message ("No Draft runs on this filing yet.") and a Clear filter action. Do not show a blank list.

## 6. Edge cases

### 6.1 Long filing name
The name truncates with an ellipsis. The type badge and top-bar buttons hold their position and are never pushed off. Full name available on hover (title attribute or tooltip).

### 6.2 Many contributors
Show up to 3 contributor avatars in an overlapping stack, then a "+N" circle for the remainder. Hovering the stack lists all names. Note: this replaces the earlier text list ("Quinn, Z, Kim, Sarah") and is a deliberate change for horizontal scalability — confirm before build.

### 6.3 Very long run description
Clamp to 2 lines (see section 4). No expansion control.

### 6.4 Deleted or unavailable version
A run whose version has `status = DELETED` still renders, because runs are immutable audit records. The version header is marked "version deleted" with a red dot. Each run on it shows a "source unavailable" tag, and its source links are disabled. A danger banner explains the situation. The run remains readable; it simply cannot be re-opened against a live document. Do not hide these runs — the audit trail is a non-negotiable product requirement.

### 6.5 Read-only filing (accessLevel = VIEWER)
A lock icon sits next to the filing name. Upload new version and New run are disabled (visibly, not hidden). Export remains enabled. All runs are fully readable. An info banner states what the user can do: "You have view access to this filing. You can read runs and export, but not upload versions or start runs."

## 7. Actions and their guards

| Action | Guard |
|---|---|
| New run | Disabled when accessLevel = VIEWER |
| Upload new version | Disabled when accessLevel = VIEWER |
| Export | Always enabled |
| Cancel run | Only on IN_PROGRESS runs |
| Retry | Only on FAILED runs |
| Open run | Always, except source-unavailable runs open in a read-only record view |
| Filter | Always enabled |

## 8. Banners

Only one filing-level banner shows at a time. Priority order if more than one condition is true: deleted version (danger) > read-only (info) > stale after upload (info/warning).

## 9. Not in scope for this screen

- The run canvases themselves (Review, Benchmark, Draft, Verify).
- Version upload flow and file parsing.
- Permission management (granting or changing access).
- Collaboration features (comments, mentions). Collaboration happens over email, outside Finiti, by product decision.

## 10. Open items to confirm before build

1. Contributor display: text list (currently locked) vs avatar stack (edge 6.2). Recommendation: avatar stack.
2. In-progress updates: polling interval vs websocket. Affects backend.
3. Cancel behavior: does cancelling an in-progress run leave a record (CANCELLED status) or remove the row entirely? Recommendation: leave a CANCELLED record for the audit trail, consistent with the immutable-run principle.
