# ARC Prize 2026 — TTT + Candidate Selection Ablation

Companion code for the ARC Prize 2026 Paper Track submission:

> **Consensus over Orbits: Candidate Selection as a First-Class Variable for Test-Time Training on ARC-AGI-2**

- Linked code submission (ARC-AGI-2 track): Kaggle ref **55904132** (`kgmon` selector; public LB 30.97; local eval 32.3/117)
- Control run: ref **55679855** (unmodified public baseline; public LB 31.39)
- ARC-AGI-3 track: ref **55757351** (Pathfinder v2; public LB 0.25)

## Contents

- `arc-agi-2/` — TTT pipeline: `arc_loader.py` (grid serialization, symmetry augmentation), `arc_solver.py` (per-task LoRA TTT, turbo-DFS, teacher-forced scoring), `arc_decoder.py` (**the five selection algorithms benchmarked in the paper**), `starter.py` (4-worker 4×L4 spawner).
- `arc-agi-3/` — `my_agent.py`: Pathfinder v2 budget-aware search agent.

## Provenance & license

- The ARC-AGI-2 pipeline is derived from the public baseline notebook
  [ARC AGI2 LB33.89 Minimal Perfpatch](https://www.kaggle.com/code/mikelou1/arc-agi2-lb33-89-minimal-perfpatch) by mikelou1 (itself derived from Koushik Rudra's "Failed in AIMO"), used under its Kaggle notebook license (Apache-2.0). Model: `sorokin/qwen3_4b_grids15_sft139`.
- Our contributions (selection algorithms and benchmark harness in `arc_decoder.py`, the GPU-side DFS/scoring changes in `arc_solver.py`, `starter.py` parallelism fixes, and `arc-agi-3/my_agent.py`) are released under **CC0 1.0 Universal** (see LICENSE).

## Reproduce the ablation

Run the v39 kernel on Kaggle (GPU 4×L4, internet off, competition data `arc-prize-2026-arc-agi-2` attached). In non-rerun mode it evaluates on the 120-task evaluation set and prints the per-selector table via `ArcDecoder.benchmark_selection_algos()`:

```
acc:  31.3/117 ('score_full_probmul_3')
acc:  32.3/117 ('score_kgmon')
acc:  31.3/117 ('score_ensemble')
acc:  30.2/117 ('score_portfolio')
acc:  31.2/117 ('score_ensemble_plus')
```

Full run: 5.4 h wall-clock on 4×L4 (≈2.7 min/puzzle, ≈45% of the 12 h window).

Paper: see `paper.pdf` in this repo / the Kaggle paper-track submission.
