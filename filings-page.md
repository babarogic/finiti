# Finiti v1 — Engineering Handoff
## Section 2 — Filings Page

The workspace-level default landing screen. Distinct from the Filing
Dashboard (Section 4), which is one filing opened. This is the list of all
filings in the current workspace.

Pair this document with the state mockups (screenshots: States A–F).

---

### 2.1 What this screen is

The screen the user lands on when they open Finiti. Scoped to the current
workspace — one client's filings only. It answers "resume my work": it
surfaces what needs attention and gets the user back into a filing fast.

---

### 2.2 Layout

Three regions, top to bottom:

1. **Top bar** — workspace pill, "Filings" title, Sort control, New filing button.
2. **Search + filter row** — workspace-scoped search, Type filter, Status filter.
3. **Body** — two sections: Active filings, then Standalone sessions. Each
   section has a label and, below its visible rows, a collapsed "view all" line.

---

### 2.3 Active filings

- A filing is **Active** if it has an IN_PROGRESS run OR any activity within
  ~30 days. Everything else is collapsed into the "View all filings" line.
- Sorted by last activity, newest first. Active rarely exceeds ~8-10 rows.
- The section label shows a count: "Active — 2 of 31".

**Filing card content:**

| Element             | Source                                              |
|---------------------|-----------------------------------------------------|
| Filing name         | Filing.name                                         |
| Version + last run  | current version label + last run summary            |
| summary             | e.g. "v4 · last: Review, in progress"               |
| Attention badge     | Count of runs needing attention, or "All caught up" |
| Last activity       | Relative time of most recent run                    |
| Left accent         | Amber = needs attention · Blue = run in progress ·  |
|                     | None = calm                                         |

Clicking a card opens that filing's dashboard.

---

### 2.4 Standalone sessions

Runs not tied to a filing (e.g. a quick peer benchmark with no document of
the user's own).

- The section shows only sessions that **need follow-up** — ran, never
  actioned. Recency is not the filter; unfinished is.
- Each visible row: status dot, session description, "Not actioned" tag,
  relative time.
- The rest collapse into "See all sessions →".
- Standalone sessions are NOT a separate nav item. This section is their only
  home on this screen; the full list is a "See all" destination.

---

### 2.5 Collapsed lines

Two collapsed lines, one per section:

- "25 more filings — resolved or older" → opens All filings view.
- "21 more sessions — already reviewed" → opens All sessions view.

Both destinations are filterable list views. They are reached only from these
links, not from the nav.

---

### 2.6 States

The Filings page has six render states. Each is specified below.
See the matching mockups, States A–F.

#### State A — Default

Both sections populated. Active filings section shows active rows + a
collapsed "View all filings" line. Standalone sessions section shows
follow-up rows + a collapsed "See all sessions" line. This is the
common case.

#### State B — No active filings

The workspace has filings, but none meet the Active rule (no IN_PROGRESS
run, no activity within ~30 days).

- Active section shows a prompt: icon, "Nothing active right now", one line
  of explanation, and a New filing action.
- The collapsed "N filings — resolved or older" line still appears below the
  prompt, so the user can reach the full list.
- The Standalone sessions section renders per its own state (C or populated).

#### State C — No follow-up sessions

Standalone sessions exist, but every one has been actioned.

- The section does NOT show rows and does NOT show a prompt.
- It collapses to a single line: "24 standalone sessions — all actioned →
  See all sessions".
- No empty gap, no placeholder block. One line only.

#### State D — First-time user

The workspace has zero filings and zero standalone sessions.

- The whole body is one primary prompt: icon, "No filings yet", one line of
  explanation, and a primary New filing action.
- No section labels, no collapsed lines, no sessions block.
- Top bar shows only the New filing button (no Sort — nothing to sort).

#### State E — Loading

- Top bar renders immediately. Workspace name and "Filings" title are known
  from navigation and must not flash.
- Search + filter row renders immediately (non-interactive until data loads
  is acceptable).
- Section rows render as pulsing skeletons until data resolves.
- Do not block the whole screen on data. Progressive render.

#### State F — High volume

Not a distinct code path — the default state under load. Documented so the
behaviour is explicit.

- Dozens of filings and sessions in one workspace.
- The screen holds by filtering, not capping: only Active rows render; the
  rest collapse into the "View all filings" line. Same for sessions.
- The Active section count ("6 of 47") makes the scale visible.
- Search and the Type/Status filters are the path to the long tail.
- Active should still rarely exceed ~8-10 rows; if it does, the Active rule
  (~30 days) is too loose and should be tightened.

#### State priority

If more than one empty condition is true, the deeper one wins:
First-time user (D) overrides No active filings (B) and No follow-up
sessions (C). B and C are independent and can both apply at once.

---

### 2.7 Search, sort, filter

- Search is workspace-scoped — filings in this workspace only.
- Sort default: last activity. Other sort options appear at volume.
- Type filter: 10-K, 10-Q, 8-K, etc. Status filter: by triage state.
- Filters apply to the Active section and the All filings view.

---

### 2.8 Not in scope for this screen

- The filing dashboard (Section 4) and the run canvases.
- Cross-workspace views. There is no screen above the workspace in v1.
- Workspace switching UI (handled by the workspace pill, specified elsewhere).

---

### 2.9 Open items to confirm

- First-time user landing: empty Filings page vs. Start new. Recommendation:
  empty Filings, for one consistent landing rule.
- Exact "Active" window: ~30 days is the working value. Tune against real
  team cadence.
- Whether the All filings and All sessions views need their own full spec
  (they are simple filtered lists, but they are net-new screens).
