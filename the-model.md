# Finiti v1 — Engineering Handoff

## Section 1 — The Model

The structural foundation. Every screen is built on these four layers, the
version loop, the run rules, and the two status axes.

### 1.1 Structure — four nested layers

    Workspace          One client or company.
    └─ Filing          A versioned project. Carries accessLevel.
       ├─ Version      One document draft — v1, v2, v3. status: ACTIVE | DELETED.
       └─ Run          One workflow execution, pinned to a version.
          └─ Draft revision   Output iterations within a Draft run.
                              DRAFT RUNS ONLY.

The document iteration is a **version** (v1, v2, v3). "Round" is not used.

### 1.2 The version loop

1. **Upload** — human uploads a draft document. Becomes version v(n).
2. **Run** — Finiti runs workflows against v(n).
3. **Findings** — Finiti shows results: gaps, flags, matrix, draft content.
4. **Edit** — human edits the document offline, in Word or Google Docs.
5. **Re-upload** — human uploads the revised document as v(n+1). Loop repeats.

Steps 1, 4, 5 are the human. Steps 2, 3 are Finiti. Finiti never edits the
source document; the human owns every version.

### 1.3 Run status — two separate axes

Run status is two independent things. Do not conflate them.

**Execution status** — did the job run. Set by the system.

| Value       | Meaning                              |
|-------------|--------------------------------------|
| IN_PROGRESS | Running. Has progress (0-100) + ETA. |
| COMPLETE    | Finished successfully.               |
| FAILED      | Errored. Carries failureReason.      |
| CANCELLED   | User-stopped. Kept as a record.      |

**Triage status** — has the user dealt with the result. Applies only when
execution status = COMPLETE.

| Value           | Meaning                               |
|-----------------|---------------------------------------|
| Needs attention | Complete; user has not looked yet.    |
| Resolved        | Triaged, accepted, or exported.       |

A FAILED, CANCELLED, or IN_PROGRESS run has no triage status.

### 1.4 Run rules

- **All four workflows are multi-run.** Each can run many times per filing. A
  run executes the work and saves as a reopenable record. Opening a run shows
  the frozen result; re-running is a deliberate new-run action.

- **Runs are immutable — an audit requirement.** A new version never resets or
  alters older runs. Runs are never hidden or deleted, including runs on
  deleted versions. The audit trail is a non-negotiable product requirement.

- **Two staleness conditions, distinct.**
  - `isStale` — the run's version is no longer current. The run shows an
    "older version" label and is still fully openable.
  - `sourceAvailable = false` — the run's version was deleted. The run still
    renders as a read-only audit record; its source links are disabled.
  - Neither involves document diffing.

- **Finiti never edits the source document.** It analyzes versions and produces
  findings and draft content. The human revises offline.

- **Access is gated by accessLevel** (OWNER | EDITOR | VIEWER). OWNER/EDITOR can
  upload versions and start runs. VIEWER can read and export only. Export is
  always available.
