# Project rules — Funnel

This project was originally hand-modeled directly in the Sketcher (no VarSet, no Params
doc) before being brought into bootstrap compliance on 2026-08-25. The primary risk here
is exactly what the global rule warns about: the sketch's 12 dimensional constraints and
the Revolution's `Angle` property were all literal numbers — one careless "fix" typed
directly into a constraint value (instead of into `Params.FCStd`) silently reintroduces
the same non-parametric state this conversion just repaired. Read the full rules below
before making any geometry change.

> **How to use this template:** Every `[FILL: …]` marker is a required edit. Leave none. A
> thin CLAUDE.md (one that just restates the global rules) is the failure mode this
> template exists to prevent — the point is to capture *this project's* specific topology,
> history, and current state, so the user does not have to re-explain it each session.

---

## Hard rules (this project)

These restate the global rules in `~/.claude/CLAUDE.md` with project-specific context.

1. **Everything parametric.** The single `Sketch` inside `Body` is the entire model — it
   has 12 dimensional constraints (7 `DistanceX`, 2 `DistanceY`, 3 `Distance`, 1 `Radius`)
   plus `Revolution.Angle`, all of which were literal numbers until the 2026-08-25
   parametrization pass. All 13 now bind to `Params.FCStd`'s `VarSet` — see the table in
   `plan.md`. If you add new sketch geometry, it needs a new Params variable before you
   dimension it, not a typed-in number.

2. **No fixing geometry by editing raw sketch coordinates.** No prior coordinate-edit
   incident in this project (it was destroyed once as a *general* pattern on the Spade
   Connector project — see `~/.claude/CLAUDE.md`); this rule applies here preventively.
   If the funnel's proportions are wrong, change the relevant `Params.FCStd` value or add
   a constraint bound to one — never touch `LineSegment`/`ArcOfCircle` X/Y/Center directly.

3. **Attach sketches to datum planes, not feature faces.** `Sketch` is attached to
   `XZ_Plane` (`FlatFace`, a principal plane, not a feature face) — already compliant, no
   prior DAG incident in this project. If you add a second sketch/body, attach it to a
   `PartDesign::Plane` datum, not to `Revolution.FaceN`.

4. **Clearance concepts stay decoupled.** This project has **no mating interfaces** — it's
   a single-piece print with no assembly, no lid, no press-fit, no bolt holes. No clearance
   Params exist and none are needed until a second part (e.g. a stand, a cap) is added. If
   one is added later, give it its own `*Clearance` Param per interface — don't reuse
   `WallThickness` or any geometry Param for a fit tolerance.

---

## Assembly architecture

Single-part print. No assembly — `Body` → `Sketch` → `Revolution` is the whole model.
The revolved profile is a lathe cross-section: a straight cylindrical **spout** (outer
radius `SpoutRadius`, height `SpoutHeight`) that blends (`NeckBlendRadius`) into a flared
conical **bowl** wall up to the **rim** (`RimOuterRadius`, at `FunnelHeight` above the
base), with a small rounded **drip-edge bead** (`RimBeadHeight` × `RimBeadWidth`) and
fillet (`RimFilletRadius`) finishing the rim edge. Wall thickness (`WallThickness`) is
uniform throughout the shell. Revolved 360° (`RevolveAngle`) about the sketch's `V_Axis`
(vertical/spout centerline).

---

## Files in this project

| File | Role | Depends on | Status |
|---|---|---|---|
| `Params.FCStd` | VarSet — all parametric variables | — | ✅ (created 2026-08-25) |
| `Funnel.FCStd` | The funnel body — `Body`/`Sketch`/`Revolution` | `Params.FCStd` | ✅ (parametrized 2026-08-25; see `plan.md`) |

No broken files.

---

## Params variables (summary)

`Params.FCStd` has 10 variables, all under the `Funnel` property group (see `plan.md`
PARAMETERS table for full detail):

- **Overall form:** `FunnelHeight`, `RimOuterRadius`, `SpoutRadius`, `SpoutHeight`
- **Shell:** `WallThickness`
- **Rim detail:** `NeckBlendRadius`, `RimFilletRadius`, `RimBeadHeight`, `RimBeadWidth`
- **Feature:** `RevolveAngle`

Run `python3 scripts/audit_parametric.py` to confirm all sketch/feature dimensions remain
bound — it enumerates unbound dimensions if any Params drift out of sync.

---

## How to verify your change didn't break parametric

After any FreeCAD edit, before considering the task done:

```bash
python3 scripts/audit_parametric.py
```

This script flags:
- Sketches with 0 constraints
- Sketches with dimensional constraints lacking expression bindings
- Sketches attached to feature faces (DAG risk)
- Params variables used nowhere (dead Params)

The script is authoritative. If it reports violations, fix them via a `macros/*.FCMacro`
change before saving or committing — never by direct coordinate edits or FCStd XML surgery.

**Audit exemption note:** this project's copy of `audit_parametric.py` is **not** byte-identical
to the canonical Spade Connector copy — the canonical copy's `DIMENSIONAL_TYPES` constraint-enum
table is known-wrong (see `reference_audit_parametric_type_bug` in the global memory index),
and this project's only content is exactly the constraint types that bug mishandles
(Distance/DistanceX/DistanceY/Radius/Diameter). This copy uses the corrected FreeCAD 1.1
Constraint.h enum instead. If the canonical copy is ever fixed upstream, this file can be
re-synced; until then, treat the discrepancy as intentional, not drift.

---

## Memory files (deeper context)

No project-scoped memories yet for Funnel — this is a fresh bootstrap. If this project
develops its own recurring issues (a DAG cycle, a coordinate-edit incident, a print-fit
saga), record them under
`~/.claude/projects/-Users-bradleycorner-Documents-3dPrinting-Commercial-License-Funnels/memory/`
so future sessions don't re-derive the same context.

---

## Workflow notes

**Invariant (apply to every FreeCAD project — do not edit):**

- **Inspect/edit FreeCAD models via the MCP bridge — never with shell tools.** Do **not**
  `unzip`/`grep`/`cat`/`sed`/`strings`/etc. a `.FCStd`. Use the FreeCAD Robust MCP server:
  `get_connection_status` first, then `open_document`, `list_objects`, `inspect_object`,
  `execute_python`, and macros. This is **enforced** by a PreToolUse hook in
  `.claude/settings.json` (copied from the root `settings.template.json` during bootstrap)
  — raw shell access to `.FCStd` is blocked. Only fall back to read-only `unzip` if the MCP
  bridge is genuinely unreachable, and ask the user first.
- **MCP server auto-starts with FreeCAD.** If an `mcp__freecad__*` call fails, the right
  interpretation is "FreeCAD isn't running" — ask whether to launch it. Do **not** silently
  fall back to `unzip` + XML parsing, and do **not** retry the same MCP call.
- **Write changes as `macros/*.FCMacro` files**, not direct XML edits. Reasons: reviewable,
  re-runnable, idempotent-friendly, uses FreeCAD's own serialization.
- **Cross-document expressions**: use the canonical form `<<Params>>#VarSet.VarName`. The
  shorter `<<Params>>.VarName` form sometimes fails with "Params not found."
- **Run `python3 scripts/audit_parametric.py` before committing.** If it reports
  violations, fix via macro, not by editing FCStd XML.

**Project-specific:**

- As of 2026-08-25, the `mcp__freecad__inspect_object` and `mcp__freecad__get_screenshot`
  typed tools were erroring in this session (confirmed by repeated calls, including on a
  plain `Body` container with no shape) — inspection fell back to read-only
  `execute_python`. If they're still broken in a future session, don't assume the bridge
  itself is down (`get_connection_status`/`list_objects`/`execute_python` all worked fine)
  — check those two tools specifically before relying on them.
- `Funnel.FCStd`, `Params.FCStd` live in the project root (bootstrap convention — not in a
  subdirectory). Deliverables export to `stl/` and `3mf/` at the project root.

---

## Print profile

**No successful test print yet — profile TBD.** A `Funnel.3mf` slicer file exists in the
project root from before bootstrap; it was produced from the pre-parametrization geometry
(dimensionally identical to the now-parametric model) and has been moved to `3mf/`. No
confirmed print/material/setting data accompanies it — fill this table in after the next
test print.
