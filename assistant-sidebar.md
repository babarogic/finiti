# Finiti v1 — Engineering Handoff
## Section 5 — Assistant Sidebar

The right-side assistant present across the filing workspace. It is always a
general chat; slash commands trigger full workflows. This section covers the
sidebar shell and its states; the workflow canvases it links to are out of
scope here.

Pair this document with the Assistant Sidebar mockups (States A–E).

---

### 5.1 What this is

A persistent right-side panel. Two jobs in one surface:

1. **General chat** — ask questions about the filing in context, get
   answers with source citations.
2. **Workflow launcher** — typing a slash command (`/benchmark`, `/draft`,
   `/verify`) starts a full workflow run, surfaced as an artifact card.

The sidebar is a slot, not a fixed component. Its scoped suggestions and
context chip reflect whichever workflow or filing the user is currently in.

---

### 5.2 Layout

Three regions, top to bottom:

1. **Header** — assistant label and icon. Fixed.
2. **Body** — context chip (when in a filing), suggestions or chat history.
   Scrolls.
3. **Input** — free-text box at the bottom. Always present. Placeholder
   teaches slash commands.

---

### 5.3 Core behaviour rules

- **Always a chat.** The default mode is conversation. Slash commands are an
  addition, not a separate mode.
- **The user decides quick answer vs full workflow.** The same highlighted
  text can produce either a quick inline answer (State B) or a full workflow
  run (State C). The system never auto-escalates a question into a run.
- **Workflow runs are auto-recorded.** A run triggered from the sidebar is
  saved to the filing exactly like a run started anywhere else. It is not a
  throwaway.
- **Citations are mandatory on factual answers.** Any answer drawn from a
  document or the vault carries a source citation. No source, no claim.
- **Context chip reflects current scope.** Inside a filing, the chip shows
  the filing, its current version, and the active workflow. Outside a
  filing, there is no chip (State D).

---

### 5.4 States

The sidebar has five render states. Each is specified below.
See the matching mockups, States A–E.

#### State A — Default

In a filing, no query yet.

- Context chip shows filing + version + active workflow, e.g.
  "10-Q v4 · Review".
- A "Suggested" list shows 2–4 scoped prompts relevant to the current
  workflow.
- Input box placeholder: "Ask, or type / for a workflow".
- A hint line below the input shows available slash commands.

#### State B — Quick search from highlight

User highlighted text in the document and asked about it.

- A highlight bar pins the selected text at the top of the body, labelled
  "Selected text".
- The user's question and the assistant's answer render as chat messages.
- Factual answers carry source citations (e.g. "vault · 2 docs").
- Follow-ups stay in the chat. This does NOT create a saved run.
- Input placeholder shifts to "Ask a follow-up…".

#### State C — Workflow trigger

User typed a slash command (e.g. `/benchmark ...`).

- The command renders as a user message.
- A short confirmation line: "Started a Benchmark run. It is saved to this
  filing."
- An **artifact card** appears: workflow icon, run title, one-line status
  ("Risk factors · 2 peers · running"), short description, and an "Open in
  canvas" action.
- The run is auto-recorded on the filing. The card links to the full
  workflow canvas.

#### State D — No filing context

Sidebar open outside any filing.

- No context chip.
- Suggestions are generic (how workflows work, what Finiti can check, start
  a filing).
- Slash commands still work, but a hint line warns: a workflow run started
  here will ask which filing to attach to.

#### State E — Loading / thinking

A query is in flight.

- Header and input remain.
- The answer area shows a thinking state (skeleton lines).
- The input shows a "Thinking…" affordance and accepts no new submit until
  the answer resolves or the user cancels.

#### State priority

B, C, and E are interaction states layered on top of A. D replaces A
entirely when there is no filing context.

---

### 5.5 Slash commands

- Recognised: `/benchmark`, `/draft`, `/verify`. (`/review` to confirm —
  see open items.)
- Typing `/` surfaces the command list inline.
- A slash command always produces an artifact card and a saved run.
- Free text without a slash is always treated as a chat question, never a
  run.

---

### 5.6 Not in scope for this screen

- The four workflow canvases the artifact cards link to.
- The document/highlight mechanics in the main canvas (covered by the
  workflow run views).
- Chat history persistence rules across sessions — see open items.

---

### 5.7 Open items to confirm

- Whether `/review` is a valid slash command, or Review is only started
  from the filing dashboard and Start new. Needs a product call.
- Chat history: does the sidebar conversation persist per filing across
  sessions, or reset each visit? Recommendation: persist per filing.
- Quick-search answer length: how much can an inline answer say before it
  should become a full run. Needs a guideline.
- Whether the artifact card updates live as the run progresses (running →
  complete), or is a static link. Recommendation: live, consistent with the
  filing dashboard in-progress rows.
