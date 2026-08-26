> **Note:** This project's geometry already existed as a hand-built (non-parametric) FCStd
> before bootstrap. No human-authored intent.md existed, so this was reconstructed from the
> as-built model (`Funnel.FCStd`, `Funnel.3mf`) and the workspace commercial standards.
> **Bradley: please correct anything below that doesn't match your actual intent** — this is
> a starting point, not a confirmed spec.

## Goal

Parametric funnel: a revolved (lathe-profile) body with a wide flared bowl narrowing to a
straight cylindrical spout, for pouring liquids/granular material into a narrow-necked
container (bottle funnel). Single-body, single-piece print — no assembly.

Observed as-built geometry (before parametrization):
- Overall height: ~40mm (bowl base to rim)
- Rim outer radius: ~40mm (80mm rim diameter)
- Spout outer radius: ~15mm (30mm spout diameter), spout section height ~14mm
- Wall thickness: ~2mm, fairly uniform
- Small fillets/bead detail at the rim edge (drip-edge profile)
- Full 360° revolution about the vertical (spout) axis

## Constraints

- Must follow `CAD_STANDARDS.md` (mm units, manifold/watertight, Part/PartDesign
  workbenches, centered at origin for Creality K2 Plus / ELEGOO Saturn 4 beds).
- Printable without supports — revolved funnel shape prints spout-down naturally.
- Target price $36–$45 (per workspace commercial pricing guardrail).
- Wall thickness in the 2.0–3.0mm default range (current 2mm value carried forward from
  the as-built model; adjust if a test print shows it's too thin for the material).
