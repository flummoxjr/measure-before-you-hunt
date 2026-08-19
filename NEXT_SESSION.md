# Track D — guide for the next session

_Written 2026-08-18. Read this first, then `LOG.md` for the chronological detail._
_Canonical plan: `../WORKPLAN.md`. Ben's action list: `report/BEN_TODO.md`._

---

## 0. Sixty-second orientation

Ben is competing for Vesuvius Challenge prizes. Deadline for the monthly progress prize:
**Aug 31 2026, 11:59pm Pacific**, Google Form at scrollprize.org/prizes.

Track D is the contrarian track: **measure whether ink *could* be found before hunting it.**
Tracks A (nnUNet surface models) and B (villa bug fixes) are owned partly by a **parallel
session working in the same repo and on the same RunPod account** — coordinate, don't collide.

The submission is written and verified: **`report/REPORT.md`** + `report/sections/01-04` +
`report/REPRODUCIBILITY.md`. `report/scripts/verify_report.py` re-asserts every headline number
against primary artifacts — **run it after any edit to the report** (currently 11 checks, 0 problems — now including the v2 corpus screen and a sweep that fails if a superseded v1 number reappears unmarked).

---

## 1. What is PROVEN (do not re-litigate)

| Result | Number | Primary artifact |
|---|---|---|
| Our stack reproduces published letters at 9 µm | pixel AUC **0.9991 / 0.9982** (2 seeds) on PHerc0139 w035 | `out/ink9um_w035_scores.json` |
| Our tifxyz→surface-volume renderer is faithful | r = **0.813** vs the published surface volume, passes the ruling test | `runpod/render_tifxyz_sv.py`, LOG 08-17 |
| Entire published GP segment corpus screened | **80/80 rows, 66 unique surfaces, 416 cm², 0 tripwire hits** | `out/survey/survey_all.json` |
| Corpus shows no text under a hardened protocol | **0 of 71 pass all 5 gates**; control passes 5/5 at z=+16.3, p=0.005, 4.678 mm, 11 cycles | `out/survey/corpus_analysis_v2.json`, `PROTOCOL_V2.md` |
| Scan-quality index for all 13 GP scrolls | two tiers, **3.0× gap**, campaign correlation Fisher p=0.007 | `out/k2b_index/*.json` |
| villa cache bug + fix | 4 GB budget → **5.98 GB**; fixed → **4.00 GB**, real PHerc1203 data | `issue_drafts/PR_cache_budget.md` |
| Paris 4 dual-energy Pb screen | 2 hits in 2.7×10⁸ (clean null) — but only excludes ~10× beyond published Pb loadings | `out/k3_sensitivity_bound.md` |
| First surfaces ever grown on PHerc0813 | 8 patches, **99.9 cm²**, 5 on material (68.5 cm²) — but all 8 sit at a median **67.3° to the lamellae** and cannot be used (§3) | `hunt/pherc0813_meshes/`, `hunt/pherc0813_mesh_qc.json`, `out/k2c_separability/pherc0813_mesh_alignment.json` |
| Sheet-separability axis, all 14 scrolls | control ranks **#1** (0.748 vs a 0.105 isotropic floor); ρ=+0.34 with SNR, i.e. a genuinely different axis | `out/k2c_separability/k2c_analysis.json` |
| K2b's ROI picker samples incrustation, not papyrus | random vs intensity-picked **2.95×**, higher in **14/14** scrolls, p=5.4e-25 | `out/k2c_separability/k2c_analysis.json` |

## 2. What is REFUTED (killed with numbers — don't resurrect without new data)

- **S1a v1 and v2** (letter-contrast on PHerc1667 w032): backgrounds were no-data / off-sheet air. `qc/s1a_verification.md`, `qc/s1a_v2_verification.md`
- **The corpus flag** (`z_dbg_gen_00166_inp_hr`, z=+5.94): → +0.97, p=0.16, 0/5 gates. E[#z≥5 among 80] = **1.07 vs 1 observed**. `verify_flag/FLAG_VERDICT.md`
- **Mesh-misplacement hypothesis for the PUBLISHED corpus**: GP meshes are as well-placed as the control (worst 2.6 vox; ±3 vox costs only 2.4% of signal), and independently confirmed by orientation — published meshes sit 14.6° from their local sheet normal, 7/9 within 30°. `hunt/geometry_compare.md`, `out/k2c_separability/published_mesh_alignment.json`. **This does NOT extend to our own PHerc0813 patches**, which are misaligned by 67.3° (§3) — do not conflate the two.
- **Input-side domain adaptation** for ink_3d: verified histogram matching **doubled** false firing (f05 0.039→0.081).
- **"Damage atlas" reframe**: sign-inverted (fires on *intact* material, ρ=−0.54 with crack texture) and dominated by free CT statistics.
- **K1 depth-contrast as a portable detector**: 0.5717 on discovery segment, 0.4934 mean on held-out (one *inverted*). Real but region-unstable.
- **ContextVar/zarr cache crash**: could not be reproduced on a live pod → not filable.

## 3. The separability axis — BUILT, and it overturned this section's original claim (2026-08-18, late)

This section previously said: *"PHerc0813 ranks #1 of 14 on structural SNR… High SNR, unresolvable
sheets."* **That was wrong, and the correction is the finding.** Scripts: `k2c_separability.py`,
`k2c_analyze.py`, `hunt/mesh_lamella_alignment.py`, `hunt/alignment_control.py`.
Artifacts: `out/k2c_separability/`.

**The axis.** Sheet separability = median over 32³ blocks of structure-tensor planarity
(λ1−λ2)/(λ1+λ2). Measured on 12 uniformly-random ROIs per scroll, all 14 scrolls, 168 cubes.

| finding | number |
|---|---|
| Control PHerc0139 (letters proven at 9 µm) ranks **#1 of 14** | **0.748** [0.738, 0.775], tightest CI of any scroll |
| Isotropic floor (28 real in-scan air windows) | **0.105** |
| Separability vs K2b structural SNR | Spearman ρ = **+0.336, p = 0.24** — genuinely different axes |
| Ordering is parameter-invariant (block 16/32/64, σ 0.5/1/2) | Spearman ρ = **+0.978 … +0.996** (n=14) |
| Induced clipping does **not** lower it (confound refuted, conservative direction) | sat 0.001→0.37 moves 0.456→0.525 |

**PHerc0813 scores 0.665 — rank 3 of 14, six times the floor.** Its sheets are *not* unresolvable;
the images show clean laminae at every depth from z=4,736 to z=13,360.

**Why the grown patches looked flat — root cause found.** The 8 PHerc0813 meshes sit at a median
**67.3° to the local sheet normal, 0 of 8 within 30°** (random-orientation null: 60°). Published
GP meshes measured with identical code sit at **14.6°, 7 of 9 within 30°** (vs 1.2 expected under
random, binomial p = 2.2e-05); Mann-Whitney ours-vs-published **p = 0.00078**. The cause is in our
own `meta.json`: we ran `vc_grow_seg_from_seed` with **no `direction_fields` and no
`normal_grid_path`**, while the published pipeline guides growth with a structure-tensor normal
field (`st_3down.zarr/normal/`, scale 3, weight 0.05). The grower had nothing telling it which way
the sheets run, so it produced surfaces oblique to them, and a surface at 67° samples *across*
lamellae and averages the modulation away. **The flat depth profiles are a tooling omission, not a
property of the scroll.**

**The bigger finding: K2b's ROI picker samples the wrong material.** `pick_rois` scores candidates
`np.where(fill > 0.98, inten, 0)` and takes the **highest** — the brightest dense material in each
scroll, which is mineral incrustation, not papyrus. Sampling the *identical* frame uniformly at
random instead: **random 0.564 vs intensity-picked 0.168, median ratio 2.95×, random higher in
14 of 14 scrolls, Mann-Whitney p = 5.4e-25.** Picked ROIs run 24–68 DN brighter in every scroll, at a median saturation of
0.039 against 0.0004 for random. This is the project's own recurring trap — *a reference
class that silently contains something other than what it names* — sitting in the flagship §1 index.

**External check.** The three scrolls the community actually segmented (PHerc1447 52 segments,
PHerc1203 22, PHerc0800 6) sit at separability ranks **5, 6, 7 of 14** but SNR ranks **8, 13, 14**.
PHerc1447 has the **worst scan quality of all 14** (SNR 8.4) and **the most published segments**.
SNR says don't bother; separability says rank 5. (n = 3 vs 11, so this is a qualitative inversion,
not a significant test — MW p = 0.277.)

**Method caveat, stated because it bit me:** a phase-randomisation null was tried first and is
**invalid** for this statistic — `J_ij = Σ_q q_i q_j |F(q)|²`, so the structure tensor is phase-blind
by construction and that null could never have failed. Removed. The statistic is the *angular*
anisotropy of gradient power; K2b's SNR is a *radial* property of the same spectrum.

### Ranked separability, all 14

| rank | scroll | separability | 95% CI | SNR rank |
|---|---|---|---|---|
| 1 | **PHerc0139** (control) | 0.748 | [0.738, 0.775] | 2 |
| 2 | **PHerc0358** | 0.713 | [0.689, 0.749] | 7 |
| 3 | **PHerc0813** | 0.665 | [0.600, 0.735] | 1 |
| 4 | PHerc0826 | 0.634 | [0.505, 0.704] | 9 |
| 5 | PHerc1447 | 0.605 | [0.566, 0.681] | 14 |
| 6 | PHerc1203 | 0.570 | [0.532, 0.603] | 8 |
| 7 | PHerc0800 | 0.563 | [0.511, 0.674] | 13 |
| 8 | PHerc1545 | 0.541 | [0.455, 0.608] | 4 |
| 9 | PHerc0211 | 0.530 | [0.468, 0.599] | 5 |
| 10 | PHerc0191 | 0.506 | [0.418, 0.643] | 6 |
| 11 | PHerc0125 | 0.415 | [0.360, 0.516] | 3 |
| 12 | PHerc1218 | 0.389 | [0.334, 0.611] | 10 |
| 13 | PHerc0257 | 0.374 | [0.295, 0.483] | 12 |
| 14 | PHerc0268 | 0.337 | [0.291, 0.397] | 11 |

Many CIs overlap, so treat this as **tiers, not a strict ranking**. PHerc0358 is the standout
non-control target and is *not* near the top on SNR.

---

## 4. Next actions, ranked

| # | Action | Cost | Why now |
|---|---|---|---|
| ~~1~~ | ~~Add the separability axis to the index~~ — **DONE**, see §3. All 14 scrolls; control ranks #1; ROI-picker bias found | $0 | Superseded by the two rows below |
| 1 | **Write the separability axis + the ROI-picker correction into report §1**, and add k2c assertions to `report/scripts/verify_report.py` | $0, local | The index is the flagship deliverable and its ROI values are drawn from the wrong material (§3). This is now the highest-value edit in the report. |
| 2 | **Re-grow PHerc0813 surfaces WITH a normal direction field**, then render + infer + battery | ~$0.70 + growth | The existing patches sit at 67.3° to the sheets and can test nothing (§3). Build a structure-tensor normal field for 0813 and pass `direction_fields` + `normal_grid_path` as the published pipeline does; seed from the high-separability ROI coordinates in `out/k2c_separability/PHerc0813.json`. |
| ~~2b~~ | ~~Render + infer + battery the 5 on-material patches as grown~~ | — | **Do not.** They are oblique to the lamellae: a null would be uninformative and a positive would be luck. §2.9's `[BEN:]` placeholder should record that they were withdrawn and why. |
| 3 | **Ben files Track B** (see `issue_drafts/FILING_CHECKLIST.md`) | $0, ~40 min | Reliable $1K-class money; #1480 already merged, so the pattern works. |
| 4 | **Ben's report voice pass** (`report/BEN_TODO.md` items A1–A9) | ~2 h | The report cannot ship without it. |
| 5 | Grow surfaces on the next-best scrolls — **PHerc0358 (sep 0.713, rank 2) first**, then PHerc0826 (0.634). NOT 0125/1545/0211: they rank 8, 9, 11 on separability despite good SNR | ~$0.35 each | §3 now says where to aim. Always pass a normal direction field. |
| 6 | The 2.4 µm path on PHerc1203 (`hunt/two_micron_path.md`) — transform already derived | ~$0.50 control first | Partly independent physics; laminae measured as merged, so odds moved down. |

---

## 5. Operational knowledge (hard-won — re-reading this saves hours)

**Budget.** `$45.54 of the $80 cap spent` as of writing; **the cap is shared with the parallel
session** (its pods are named `c2-*`/`c3-*` — do NOT terminate them). Burn was $2.25/hr with three
of their pods up. `runpod/rp.py status` shows spend/burn; `rp.py` refuses new pods past $70.
**Kill your own pods the moment a result is read** — an idle pod cost us ~$1 before it was noticed.

**The villa checkout is shared.** It moves between branches (`cycle2` → `cycle3` today) because the
parallel session works there. **Never switch branches in it.** Use `git worktree add` to a temp dir,
and note that `.venv314` has vesuvius installed *editable* pointing at the shared tree, so pytest in
a worktree still imports the shared code unless you set `PYTHONPATH=<worktree>/vesuvius/src`.

**Pod recipe that works** (`runpod/provision_survey.sh`): RunPod pytorch image → `uv sync --extra
models` → **pin `torch==2.11.0` + `torchvision==0.26.0` cu128** (host driver caps at CUDA 12.8;
PyPI torch 2.13 fails) → explicit dep list → `TORCH_COMPILE_DISABLE=1` (no triton on some images).
For **custom images** (e.g. `ghcr.io/scrollprize/villa/volume-cartographer:edge`) there is **no sshd** —
inject one via `dockerStartCmd` (see `runpod/launch_grow_0813.py`).
Throughput: render+infer ≈ 2.5–5 min/segment on a 5090; 80 segments ≈ $5.5.

**Traps that have already bitten us:**
- **Zero-fill ambiguity**: a failed chunk fetch and an empty volume both read as zeros. Count
  server 404s separately (`hunt/qc_new_meshes.py`). This is the #1 recurring trap in this project.
- **Small-null z inflation**: 16 permutations underestimated the null sd by 2.2×. Use ≥200 and
  report **empirical p**, not z.
- **`gh search issues` silently omits PRs** — duplicates here arrive as PRs. Use
  `gh api -X GET search/issues`. (gh IS authenticated as `flummoxjr`.)
- **`git apply` fails across differing checkouts** — copy the whole file instead.
- **PowerShell mangles `$(...)`, `<`, quotes** in remote commands — write a `.sh`, `scp` it, run it.
- **Tiled inference creates patch-boundary halos**: 90.8% of one segment's hot pixels lived in the
  stride lattice's halo ring. Erode rims before scoring prediction maps.
- **villa's cache overshoots its budget ~2×** until our fix lands — budget disk accordingly.

**QC discipline that has repeatedly saved us** (11-row ledger in `report/sections/04_methodology.md`):
reproduce the numbers exactly → attack the meaning with a *new* experiment → every positive must
beat a **matched** null → name the mundane explanation → account for autocorrelation. Five of the
eleven corrections were the same trap: *a reference class that silently contained something other
than what it named*.

---

## 6. Where things live

```
trackD/
  LOG.md                     chronological record (read after this file)
  NEXT_SESSION.md            this file
  report/                    THE SUBMISSION — REPORT.md, sections/, figures/, BEN_TODO.md,
                             COMMUNITY_POST.md, REPRODUCIBILITY.md, scripts/verify_report.py
  out/k2b_index/             the 14-scroll index
  out/survey/                corpus screen: survey_all.json, corpus_analysis_v2.json,
                             PROTOCOL_V2.md, maps_shard*/ (150 prediction maps)
  out/ink9um_w035*           the positive control
  out/k3_*                   Paris 4 dual-energy screen + sensitivity bound
  hunt/                      the five letter-hunt investigations + HUNT_PLAN.md +
                             pherc0813_meshes/ + geometry_compare.md
  qc/, qc_live/, salvage/,
  comb/, verify_flag/        the audit trail (shipped as first-class artifacts)
  issue_drafts/              Track B: FILING_CHECKLIST.md + filing/ (paste-ready bodies)
  runpod/                    all cloud tooling (rp.py, fleets, screener, renderer)
```

---

## 7. Honest odds (from `hunt/HUNT_PLAN.md`, unchanged by today's results)

P(defensible First Letters claim by Jun 2027): **12–18%**. By Aug 31 2026: **4–7%**.
Failure across the six good scrolls is **correlated**, not six independent draws — our index sees
scan quality, and (as of today) not sheet separability, so it tells us where we can best *look*,
not what is there.

**Plan the monthly submission around the deliverables, not around letters.** The deliverables are
real, verified, and unclaimed by anyone else: the index, the validated instrument, the corpus
screen, the two reusable gates, the transfer-failure characterization with its fine-tune recipe,
the dual-energy null, and the corrections ledger.
