# repair branch handoff

> **⚠ DO NOT fabricate or merge from this branch.** It is an unreviewed repair
> proposal for dinhv to review commit-by-commit — hence the branch name.

**Branch:** `DO-NOT-USE-vincent-repair-unreviewed` (local only, NOT pushed). Base: upstream `b84c83a` "Footprint and symbol fix".
**Status: all four r5 mechanical blockers resolved. DRC 0 errors / parity clean / 0 unconnected. Production/ regenerated as one consistent set.**
Context: `DESIGN_REVIEW-r5-council.md` (incl. the b84c83a addendum).

## Commits (review in order)

1. `89fd9d0` **Re-annotate 9 clobbered PCB refs** — GND×2/VBUS×2/CC1/CC2/DIR/VCONN/V_OUT → TP1–TP7, DIR_LED2, VBUS_LED2. Matched by footprint `path` UUID (all already pointed at the right symbols; only the Reference text was clobbered). Schematic parity: 20 issues → 0.
2. `d0d5537` **TX1/TX2 reference fix** — the B.Cu via dives are polarity crossovers (U1 pad order is inverted vs J1 A2/A3+B2/B3), so an F.Cu-only reroute is topologically impossible (r4's suggestion was wrong). Fix: two GND zones on In2 (priority 2) under the crossover regions + 7 GND return vias (0.6/0.3) adjacent to the 8 signal vias. B.Cu under-passes now reference GND end-to-end (verified point-in-polygon against the saved fills).
3. `6df6a20` **J1 hole-clearance waiver** — the 9 DRC errors were J1's own plated shell-peg holes 0.144 mm from its own GND shield pads (manufacturer land pattern; both features are the grounded shell). Rule scoped to `A.Reference=='J1' && B.Reference=='J1'`, board-wide hole clearance stays 0.2 mm.
4. `91ecfe2` **Production/ regenerated** — everything from one board revision, using the board's saved plot params (identical 19-file gerber recipe + zip). New: `bom.csv` (replaces stale Jun-15 `bom.xlsx`; R5 LCSC consolidated to C413111 = R6/R7's SKU, same 10K 0402) and `HD3SS3220-breakout-cpl.csv` (pick-and-place, previously missing). Fresh IPC-D-356 `netlist.ipc`.

5. `e077eae` **R5 footprint field sync** — the SKU consolidation initially edited only the schematic; the footprint's cached LCSC field tripped the parity field check at the branch tip. Now synced; final gate fully clean (fab outputs unaffected — non-Reference/Value fields are not plotted).

## Verification (artifacts in `repair-verify/`)

| gate | result |
|---|---|
| `kicad-cli pcb drc --schematic-parity --severity-all` | **0 errors, parity clean, 0 unconnected**, 207 warnings (baseline 206: silk noise; the 2 `connection_width` pinches my first-cut patches created were fixed by growing the patches over two pre-existing GND stitch vias) |
| ERC | 34 warnings, unchanged (sch edit was one LCSC field) |
| eyeline r5 (`eyeline-r5/`) | all 7 pairs pass, eyes ≥0.98 UI @5G, Δρ ≤0.0001 vs ngspice — same numbers as r4. **Caveat: eyeline's plane_map is per-layer (`In2=VBUS`), it cannot see zone patches, so its TX1/TX2 "ref swap!" marks are now false positives.** The reference fix is proven at board level: all 4 under-pass midpoints sit over `gnd_ref_patch_*` fill, In2 GND objects 4→7 in the gerber. |
| gerbers (kicad-happy) | 1 warning: GR-004 paste-vs-copper ratio = TH pads without paste, known false positive |
| fab-output proof | F.Cu gerber VBUS-attributed objects **3 (old shorted export) → 18**; drill count 143→184 fully accounted (+28 vias & +6 pad holes from b84c83a, +7 return vias from this branch) |

## Residual / decisions still owed (unchanged from r5, not this branch's scope)

- **VBUS hard-parallel J2↔J1 with no switch** (r5 blocker #5) — architecture decision; TI says bulk cap must be switched out in UFP; dead-battery Rd makes C-side back-drive real. Needs a load switch or an explicit acceptance.
- TX1− entry via has no *dedicated* return via (D± runs on B.Cu 0.85 mm west, no room); it shares the cluster patch + adjacent vias. TX2− entry's return via is 1.35 mm out for the same reason.
- D+/D− B.Cu diagonals cross the TX1 patch edge (~1–2 mm): two extra reference transitions at 480 Mb/s — benign, and GND is a better reference than the VBUS plane it replaces.
- Pre-existing `.kicad_dru` "Allow tight corners on U1" still waives 4 real 0.071–0.078 mm different-net gaps (< JLC 0.09 min) — decide: fix U1 fanout or get fab sign-off.
- CC pins and D± still have no IEC-rated ESD; VBUS has no TVS; 0.3 mm VBUS entry trace vs 3 A straps; DIP6-closed-at-boot vs tENnCC_HI ≥2 ms; VCONN test point is actually VCONN_FAULT_N (open-drain, no pull-up); J3 has no GND pin; /SSRX+ netclass captured by Power class.
- BOM: TP1–TP7 have no LCSC (Keystone test points — decide populate/DNP); TP rows also appear in the CPL (TH parts; delete the rows if not assembling them).

## How to take this

Diff is reviewable per commit: `git log -p b84c83a..DO-NOT-USE-vincent-repair-unreviewed`. If upstream wants it: cherry-pick or merge locally and push from an authorized account — this branch was deliberately not pushed anywhere.
