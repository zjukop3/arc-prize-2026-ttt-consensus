# Kaggle Writeup 版本（正文 1,500 词内；当前约 1,330 词）

**Title**（填 Writeup 标题栏）:
Consensus over Orbits: Candidate Selection as a First-Class Variable for Test-Time Training on ARC-AGI-2

**Subtitle**:
A controlled ablation on a fixed public TTT backbone: vote-count consensus beats probability-product confidence, ensembling adds nothing, and selection headroom is nearly saturated.

**Track**: ARC-AGI-2

**Linked submission ID**（Accuracy 项会要求填）: **55904132**（TTT v39, selector `kgmon`; public LB 30.97; local eval 32.3/117）

---

## 正文（粘贴区）

### The overlooked knob

ARC-AGI tasks are under-determined by design: the training pairs rarely pin down one transformation, so every solver must resolve ambiguity at test time. Public work on ARC-AGI-2 has converged on a recipe — grid-serialized LLM, per-task test-time training (TTT), symmetry-augmented decoding — and nearly all discussion concerns the *training* side: augmentation counts, LoRA settings, decoding thresholds. The final step, choosing attempt_1 and attempt_2 from the candidate pool, is usually an undocumented heuristic.

This writeup treats it as a variable. On a fixed backbone we vary only the selection rule and measure pass rates on the full evaluation set, using one shared candidate store from a single TTT run. Our leaderboard score is mid-table by design; the contribution is the measurement.

### Setup

**Backbone (inherited).** The public LB-33.89 baseline by mikelou1: Qwen3-4B SFT'd on grid-serialized data (`sorokin/qwen3_4b_grids15_sft139`), per-task LoRA (rank 256, rsLoRA, lr 5e-5, 1 epoch) on the task's training pairs ×16 symmetry augmentations, DFS decoding over the 12 ARC tokens with NLL-threshold pruning, and teacher-forced augmented-NLL scoring. Every candidate record carries a grid, a beam score, and augmented-NLLs. Selectors are deterministic functions of these records; selection never re-runs the model.

**Five selectors.** *probmul_3* (probability product; confidence-heavy); *kgmon* (vote count, tie-broken by augmented NLL; consensus-heavy); *ensemble* (round-robin interleave of the two); *portfolio* (hand-weighted blend: 2.0·votes − 0.35·aug_mean − 0.15·beam − 0.10·aug_best); *ensemble_plus* (attempt_1 from ensemble, attempt_2 = first distinct candidate under portfolio).

**Evaluation.** Full 120-task evaluation set = 117 puzzles after splitting multi-grid tasks (re-aggregated per the official metric), 2 attempts. All numbers below come from one candidate store; the run took 5.4 h on 4×L4, ≈45% of the 12 h window.

### Results

| Selector | Solved (/117) | Puzzle-keys | Δ vs probmul_3 |
|---|---|---|---|
| **kgmon** | **32.3** | 44 | **+1.0** |
| probmul_3 | 31.3 | 43 | — |
| ensemble | 31.3 | 43 (identical set) | 0.0 |
| ensemble_plus | 31.2 | 43 | −0.1 |
| portfolio | 30.2 | 42 | −1.1 |
| oracle union | ≈33.3 | 45 | ≈+1.0 vs best |

Three facts stand out. First, **kgmon strictly dominates** probmul_3: its solved set contains all 43 of probmul's keys plus one more (6e453dd6_0). Second, **ensembling adds nothing**: the interleaved ranking solves exactly the same set as probmul_3. Third, the **oracle union** of all five selectors adds only ≈1 fractional task over the best single rule.

Candidate statistics: 394 of 2,339 candidate records (16.8%) are exactly correct, with beam scores spanning 0.271 (avg) to 1.553 (max) — confidence cannot filter out wrong candidates; ranking is the selector's whole job.

For reference: our linked submission (v39, `kgmon`) scores 30.97 public / 32.3-of-117 local; a control run of the unmodified baseline scores 31.39 public; a no-TTT enumeration baseline scores 0.00.

### Why consensus won — a falsified hypothesis

Scoring candidate *c* under augmented views *g·task* is a Monte-Carlo estimate of the orbit-marginal E_g[log p(c | g·task)] over the symmetry group G (dihedral × color permutations); the vote count estimates the size of c's basin of attraction — the fraction of augmented decoding paths that reach c at all. Confidence and popularity measure different things, so we predicted interleaving would recover tasks either parent missed.

**The data says no.** Near the decision boundary (top-2), the confidence order is essentially a monotone transform of the vote order — the two signals are not orthogonal where it matters. Once decoding runs over enough augmentations, basin size dominates: a candidate reached from many augmented starting points is almost always the orbit-stable mode, and confidence adds no independent evidence. Consistently, the hand-weighted blend underperforms both pure rules, and forced attempt_2 diversification picks worse candidates twice (−0.1 net).

### The saturation bound

The selector spread (30.2–32.3, ±1.8 pp) is larger than many published training-side tweaks, and it is free. But the oracle-union bound says selection on this candidate store is within ≈1 task of saturated: for most unsolved puzzles the correct grid is simply absent from the store, and no selector can recover it. **The headroom provably lives in candidate generation, not ranking** — the practical recommendation is: use vote-count consensus, stop tuning selection, and spend effort on generation diversity.

### Universality

The same principle — *adapt at test time, then reach consensus under a budget* — instantiates in ARC-AGI-3 as a budget-aware search agent (our Pathfinder v2: per-game time budgets, background-threaded solving, prioritized replay; public LB 0.25). We report it as principle-level evidence, not a competitive score: consensus over augmented views in the static track corresponds to spending search where the remaining budget is scarcest in the interactive one.

### Reproducibility

Full pipeline code, the agent, this paper's PDF, and the ablation harness: https://github.com/zjukop3/arc-prize-2026-ttt-consensus (our contributions CC0; derived pipeline under the original notebook license). The benchmark is one function call (`ArcDecoder.benchmark_selection_algos()`) on any decoded-candidate store, so future backbones can re-run this ablation in minutes.

### Limitations

Everything inherits the 4B backbone's ceiling. The ablation is a single run on one store; selectors are deterministic given the store, but the store reflects one TTT run's sampling. Stronger generators may reopen selection headroom — which this harness measures cheaply, by design.

### References

Chollet (2019) arXiv:1911.01547 · Akyürek et al. (2024) arXiv:2411.07279 (TTT) · Wang et al. (2022) arXiv:2203.11171 (self-consistency) · Ellis et al. (2021) DreamCoder · mikelou1, *ARC AGI2 LB33.89 Minimal Perfpatch* (Kaggle) · sorokin, *qwen3_4b_grids15_sft139* (Kaggle Models) · arcprize.org.
