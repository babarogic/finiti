# Finiti v1 — Engineering Handoff
## Section 3 — Start New

The launchpad: the destination for starting fresh work. Distinct from the
Filings page (Section 2), which is the default landing screen.

Pair this document with the Start New mockups (default + propose state).

---

### 3.1 What this screen is

A destination, reached deliberately from the nav. It is NOT the default
landing screen — the Filings page is. Start new answers "begin something
new"; the Filings page answers "resume my work".

Its job: start new work fast, and show the user what Finiti can do.

---

### 3.2 Layout

Three regions, top to bottom:

1. **Greeting** — user name, current workspace, a "How Finiti works" link.
2. **Start field** — the action-scoped input. See 3.3.
3. **Capability list** — "Explore what Finiti can do": the four workflows as
   entry rows, plus Search as a disabled fifth row.

---

### 3.3 The start field — three input paths

The start field is action-scoped. The user states intent; the system does
not guess unless asked to.

#### Path 1 — Chip selected

- User taps a workflow chip: Review, Benchmark, Draft, Verify.
- The chip states the workflow explicitly.
- Any typed text rides along as an instruction that pre-fills that
  workflow's setup.
- When a chip is set, the system never guesses the workflow.

#### Path 2 — Upload

- User attaches a filing via the paperclip control.
- Filing type (10-K, 10-Q, 8-K, 20-F...) is detected from the document
  cover page and shown as a confirmable pill. The user is not asked to
  pick it from a dropdown.
- Upload runs a jurisdiction-aware form check automatically.
- No exchange dropdown — the product stays exchange-agnostic.

#### Path 3 — Free text, no chip

- User types a request without selecting a chip.
- The system classifies the request, proposes one workflow, and asks for
  confirmation before running.
- A guessed workflow never runs silently. See state 3.5.

---

### 3.4 Capability list

"Explore what Finiti can do" — five rows.

- **Review, Benchmark, Draft, Verify** — each row is an entry point that
  starts that workflow's setup. These are entry points, not a duplicate of
  the nav.
- **Search** — the fifth row, visibly disabled, with a "Coming soon" tag.
  Search produces no saved run, so it is deliberately set apart from the
  four workflows. Disabled in v1.

---

### 3.5 State — workflow proposed (Path 3)

When the user submits free text with no chip selected:

- The system classifies the text and proposes one workflow.
- An inline panel appears inside the start field: "This looks like a
  [Benchmark] run. Start one, or pick a different workflow above."
- Two actions: "Start [workflow]" (primary) and "Choose another".
- "Choose another" returns focus to the chip row; the typed text is kept.
- Nothing runs until the user confirms.

---

### 3.6 States

Start new has five render states. Each is specified below.
See the matching mockups, States A–E.

#### State A — Default

Greeting, empty start field, full capability list.

- No chip selected.
- Placeholder teaches the three input paths: "Drop a filing, pick a
  workflow, or describe what you need…".
- Start field shows the four workflow chips, a divider, then the disabled
  Search chip.
- Form-check hint line sits below the start field.
- Capability list shows all five rows (four workflows + disabled Search).

#### State B — Chip selected

User has tapped one workflow chip.

- The selected chip is highlighted; the other three stay neutral.
- The placeholder changes to a workflow-specific prompt, e.g. for
  Benchmark: "Describe what to benchmark, or attach a filing to compare…".
- A helper line confirms the behaviour: submitting goes straight to that
  workflow's setup, with no workflow guess.
- Typed text is carried into that workflow's setup as an instruction.

#### State C — File attached

User has attached a filing via the paperclip.

- A file pill shows the filename and the detected filing type, e.g.
  "10-Q · detected".
- The type pill is confirmable — tapping it lets the user correct the type.
- Placeholder shifts to "Add an instruction, or pick a workflow…".
- A helper line notes that upload runs a jurisdiction-aware form check.
- If type detection fails, the pill shows an unset state prompting the user
  to pick the type (see open items).

#### State D — Workflow proposed

User submitted free text with no chip selected (Path 3).

- The typed text stays visible in the start field.
- An inline propose panel appears inside the start field: "This looks like
  a [Benchmark] run. Start one, or pick a different workflow above."
- Two actions: "Start [workflow]" (primary), "Choose another".
- "Choose another" returns focus to the chip row and keeps the typed text.
- Nothing runs until the user confirms.

#### State E — Loading

- Greeting renders immediately — name and workspace are known from
  navigation and must not flash.
- The start field renders immediately and is always interactive. A user
  must be able to start work before the capability list resolves.
- The capability list renders as pulsing skeletons until data resolves.

#### State priority

States B, C, and D are mutually exclusive submit/input states and supersede
the Default (A) presentation while active. Loading (E) only applies on
initial screen load.

---

### 3.7 Rules

- **Not the default landing.** Reached from the nav. Filings is the default.
- **Workspace-scoped greeting.** Shows user name + current workspace. The
  time-of-day greeting is cosmetic.
- **The system never guesses a workflow when a chip is set.** Guessing only
  happens on Path 3, and even then it proposes, it does not run.
- **Filing type is detected, not asked.** Confirmable pill on upload.
- **Search is disabled in v1** — dashed chip with SOON tag in the start
  field (after a divider), and a "Coming soon" row in the capability list.

---

### 3.8 Not in scope for this screen

- The workflow setup panels themselves (per-workflow run setup).
- The run canvases.
- Search functionality (v1 disabled).
- The workflow classifier model — see open items.

---

### 3.9 Open items to confirm

- Workflow classifier confidence: Path 3's propose-and-confirm only works
  if the proposed workflow is usually right. Needs an engineering answer on
  classifier accuracy, and a fallback when confidence is low (e.g. propose
  nothing, just ask the user to pick a chip).
- First-time user landing: empty Filings page vs. Start new. Recommendation:
  empty Filings. (Same open item as Section 2.)
- Whether typed text on Path 1 (chip + text) always pre-fills cleanly, or
  whether some workflows ignore free text. Needs per-workflow confirmation.
