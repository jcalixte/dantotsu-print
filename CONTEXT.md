# Dantotsu Print

Tooling that moves code-quality defects from the IDE onto the physical
"Daily — Dantotsu and Problem Solving" board with minimal friction, preserving
the Dantotsu ritual of *visualizing the defect* — which for developers means
seeing the actual lines of code.

## Language

**Board**:
A **magnetic whiteboard** displaying one or more **A3 sheets** held by magnets, refreshed on a **daily** cadence.
_Avoid_: Notion database — the low-friction digital habit this tool exists to displace.

**A3 sheet**:
The printed "Daily — Dantotsu and Problem Solving" 5-column template, magnet-mounted on the Board, one row per defect.

**Defect card**:
The printed **monochrome peel-and-stick label** bearing the defective code + `path·lines·@sha` footer (+ optional QR), stuck into the **Defect** cell of an A3 sheet — the physical "photo of the defect."

**Defect** (the printed artifact; column 2 of the board):
A code-quality defect — the defective lines of code are themselves the nonconformance (e.g. a switch missing a case). This is what gets printed and placed in the **Defect** cell: the "photo of the defect," the *before* state.
_Avoid_: "Problem" for the artifact — the board column is labeled **Defect**. Never use it to mean the user-visible symptom (the crash), which is only the consequence that made the defect visible.

**Occurrence cause**:
Why the defect was introduced. (Handwritten, Causes column.)

**Late detection cause**:
Why the defect was not caught earlier. (Handwritten, Causes column.) — the Toyota dual-cause split.

**Countermeasure**:
The fix plus the process change that prevents recurrence. (Handwritten.)

**Status quadrant**:
The 4-step visual progress marker on each row: 1) Defect identified → 2) Cause found → 3) Countermeasure done → 4) Defect eradicated or **yokoten** (horizontally deployed).

## Board columns (left → right)

Date | **Defect** | Causes (Occurrence / Late detection) | Counter measures | Resp. person | Status

## Division of labor

- **Machine prints** only the **Defect** cell: defective code snippet + file/line + commit SHA + QR permalink.
- **Human handwrites** everything else: date, causes, countermeasures, responsible person, status.

## Relationships

- A **Board** holds many **Defects**, one per row.
- A **Defect** is visualized by a printed snippet of the defective code placed in the Defect cell.

## Flagged ambiguities

- "problem" vs "defect": the board column is literally **Defect**. **Resolved:** *Defect* is canonical for the printed artifact; "Problem solving" names the overall ritual/board, not the artifact.
- "problem" was also used to mean the consequence (app crash). **Resolved:** the crash is the consequence, not the defect; it is not printed.
