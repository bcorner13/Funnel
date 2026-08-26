# Plan — Funnel parametric conversion

Scope: the geometry already exists (`Body` → `Sketch` → `Revolution`, one PartDesign body).
Nothing about the shape is changing. This plan converts every literal dimensional value in
`Sketch` and the `Revolution.Angle` property into `setExpression` bindings against a new
`Params.FCStd` VarSet, per the workspace's hard parametric rules. No coordinates are edited —
only constraint *values* move from literal numbers to expressions on the existing constraints.

## PARAMETERS

New `Params.FCStd` VarSet (`App::PropertyLength` unless noted), one Param per distinct
physical concept per rule "one knob, one concern" — even where two dimensions currently
share a numeric value, they get separate Params unless they are genuinely the same
constant applied at multiple points (wall thickness, rim bead width — noted below).

| Name | Type | Default | Binds | What it controls |
|---|---|---|---|---|
| `FunnelHeight` | Length | 40 mm | `Sketch.Constraints[9]` (Distance) | Overall height, bowl base to rim top |
| `RimOuterRadius` | Length | 40 mm | `Sketch.Constraints[7]` (DistanceX) | Outer radius of the flared rim before wall thickness |
| `WallThickness` | Length | 2 mm | `Sketch.Constraints[8]`, `[23]`, `[30]` (DistanceX ×3) | Uniform shell wall thickness — same physical constant reused at rim, spout base, and lip |
| `SpoutRadius` | Length | 15 mm | `Sketch.Constraints[14]` (DistanceX) | Outer radius of the straight spout/neck section |
| `SpoutHeight` | Length | 14 mm | `Sketch.Constraints[13]` (DistanceY) | Height of the straight spout section before it flares into the bowl |
| `NeckBlendRadius` | Length | 1 mm | `Sketch.Constraints[25]` (Distance) | Offset controlling the blend radius where spout meets the flared bowl wall |
| `RimFilletRadius` | Length | 1 mm | `Sketch.Constraints[31]` (Radius) | Fillet radius at the top-outer rim corner |
| `RimBeadHeight` | Length | 1.2 mm | `Sketch.Constraints[26]` (DistanceY) | Height of the small drip-edge bead detail at the rim |
| `RimBeadWidth` | Length | 0.4 mm | `Sketch.Constraints[49]`, `[50]` (Distance ×2) | Flat width of the rim bead detail — same constant on both mirrored edges |
| `RevolveAngle` | Angle (`App::PropertyAngle`) | 360° | `Revolution.Angle` | Revolution sweep angle (full solid of revolution) |

10 Params covering all 13 currently-unbound dimensional bindings flagged by the audit.
`WallThickness` and `RimBeadWidth` are each intentionally reused across multiple
constraints — that's one physical concept applied consistently, not conflation.
`NeckBlendRadius` and `RimFilletRadius` both currently equal 1mm but are kept as separate
Params (different locations, different concepts) so they can be tuned independently later.

## FEATURE TREE

No new features. Existing tree, unchanged:

1. `Body` (PartDesign::Body)
2. `Sketch` — attached to `XZ_Plane` (principal plane, `FlatFace` — already compliant,
   not a feature face, no datum-plane change needed)
3. `Revolution` — revolves `Sketch` 360° about the sketch's `V_Axis` (the vertical
   spout axis) — this is the Body's `Tip`

## CONSTRAINT STRATEGY

For each row in the PARAMETERS table:
1. If the Param doesn't yet exist in `Params.FCStd`, add it via
   `addProperty("App::PropertyLength", name, "Funnel", doc)` (or `PropertyAngle` for
   `RevolveAngle`), set its default value.
2. `setExpression` on the target constraint(s)/property to `<<Params>>#VarSet.Name`
   (canonical cross-document form).
3. Idempotent: macro checks `if name in vs.PropertiesList` before adding, and checks
   existing `ExpressionEngine` bindings before re-setting — safe to re-run.
4. Recompute the document after all bindings are set; verify the shape's `BoundBox` and
   `Volume` are unchanged (within float tolerance) from the pre-change values
   (BBox 83.95 × 83.92 × 41.00 mm, Volume 14062.30 mm³) — proves the binding pass changed
   no geometry, only how the existing values are driven.

Delivered as `macros/parametrize_funnel.FCMacro`, run via the FreeCAD MCP `execute_python`
tool is **not** used for the edit itself — the macro is created with `create_macro` and run
with `run_macro`, per the typed-tool-preference rule (no typed "setExpression" tool exists,
so a macro is the correct tier, not raw ad-hoc `execute_python`).

## VALIDATION

1. `python3 scripts/audit_parametric.py` from the project root — must report `0` issues
   after the macro runs (currently 14).
2. Recompute in FreeCAD (`recompute_document`) — must succeed with no errors, `Revolution`
   still valid (`shape.isValid()`).
3. Shape BoundBox/Volume unchanged vs. the pre-parametrization baseline recorded above.
4. Spot-check parametric responsiveness: temporarily bump `WallThickness` from 2→3mm,
   recompute, confirm the shell thickens as expected and the shape stays valid and
   manifold, then set it back to 2mm and recompute again before saving.
5. Re-export `stl/Funnel.stl` and `3mf/Funnel.3mf` from the validated, parametric model
   (the existing root-level `Funnel.3mf` was produced from the pre-conversion geometry —
   filenames/dimensions are identical so this is a re-export, not a redesign).

## Not in scope for this pass

- No new geometry, no new features, no dimension changes — this is a pure
  literal→expression migration.
- Bootstrap structural compliance (folders, CLAUDE.md, git, standards docs) is handled
  separately from this geometry-level plan; see the CLAUDE.md fill for that inventory.
