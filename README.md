# Funnel

Parametric bottle funnel — a revolved lathe profile with a wide flared bowl narrowing to
a straight cylindrical spout. Single-piece print, no assembly.

---

## Files

| File | Description |
|------|-------------|
| `Funnel.FCStd` | The funnel body — `Body` / `Sketch` / `Revolution` |
| `Params.FCStd` | VarSet — all parametric variables |
| `stl/Funnel.stl` | Printable mesh export |
| `3mf/Funnel.3mf` | Slicer-ready export |

---

## Parametric usage

1. Open `Funnel.FCStd` in FreeCAD 1.1+ (with `Params.FCStd` open alongside it — the
   sketch's constraints and `Revolution.Angle` are expression-bound to it).
2. Edit `Params.FCStd`'s `VarSet` to change dimensions — see `plan.md` for the full
   parameter table.
3. Recompute — the profile and revolved solid update automatically.

Key parameters:
- `FunnelHeight` — overall height, bowl base to rim (default: 40mm)
- `RimOuterRadius` — outer radius of the rim (default: 40mm)
- `SpoutRadius` / `SpoutHeight` — spout outer radius / straight-section height (default: 15mm / 14mm)
- `WallThickness` — shell wall thickness, uniform (default: 2mm)

---

## Standards

Built to this workspace's `CAD_STANDARDS.md`: mm units, watertight/manifold geometry,
centered at origin for Creality K2 Plus / ELEGOO Saturn 4, target price $36–$45.

Before committing any change, run:

```bash
python3 scripts/audit_parametric.py
```
