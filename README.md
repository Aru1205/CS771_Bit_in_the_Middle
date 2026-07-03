# CS771 Assignment 2 — Bit-in-the-Middle Attack

Solution to CS771 (IIT Kanpur, 2025-26-III) Assignment 2: learning a 17-bit arbiter
PUF from CRPs whose middle challenge bit was tampered with and then erased, posed as
a latent-variable MLE problem solved by alternating optimization (hard-EM /
winner-take-all).

## Repository contents

| File | What it is |
|---|---|
| `submit.py` | The code deliverable. Implements `my_latent()` (Part 2) and `my_latent_updated()` (Part 4) using only `numpy` + `sklearn` linear models, as the assignment allows. |
| `report.tex` / `report.pdf` | The report deliverable (Parts 1, 3, 5): full derivations of both alternating optimization algorithms and the agreement analysis. NeurIPS 2025 preprint format. |
| `submit.zip` | Password-protected ZIP containing only `submit.py`, per the submission instructions. Password is in `zip_password.txt` (10 alphanumeric chars). |
| `gen_data.py` | Synthetic data generator that follows the assignment's exact generation process (hidden 16-bit PUF → middle bit → insert → 17-bit PUF → response → erase). Used because the official package is on IITK servers. |
| `evaluate.py` | Local evaluation harness mirroring the official judge (train accuracy for `my_latent`, test accuracy via predicted middle bits for `my_latent_updated`) plus the Part-5 quantities. |
| `make_plots.py` | Generates `convergence.png` used in the report. |
| `train.dat`, `test.dat` | Synthetic 8000/2000 CRP files in the assignment's format (16 challenge bits + 1 response bit per row). **Replace these with the real files from `assn2.zip` and re-run to get real numbers.** |
| `RESULTS.md` | Recorded results. |

## How to run

```bash
python3 gen_data.py 42     # skip this if you drop in the real train.dat / test.dat
python3 evaluate.py        # runs both methods, prints timings + accuracies, saves results.npz
python3 make_plots.py      # regenerates convergence.png for the report
pdflatex report.tex && pdflatex report.tex
```

## Using the real dataset

1. Download `assn2.zip` from the course page and copy its public `train.dat` and
   `test.dat` here (same 17-numbers-per-row format).
2. Re-run `evaluate.py` and `make_plots.py`, then update the three numbers in the
   Part-5 section / Table 1 of `report.tex` and recompile. The qualitative story
   (Part 5) is robust across data seeds.
3. Paste the bodies of `my_latent()` / `my_latent_updated()` into the official
   `submit_template.py` from the package (keep its non-editable regions intact) and
   validate on the Google Colab script before any submission.

## Method in one paragraph

Both methods are hard-EM. `my_latent()`: with a uniform latent prior, the z-step
picks, per CRP, the middle bit whose score best explains the observed response, and
the (w,b)-step is a plain logistic regression on the completed 17-bit embeddings —
alternate to convergence with random restarts. `my_latent_updated()`: the latent
prior is itself a 16-bit arbiter PUF (u,a), so the z-step scores each candidate bit
by the joint likelihood (response term + prior term), and there are two logistic
regressions per alternation — responses on completed 17-bit features for (w,b), and
current latent estimates on 16-bit features for (u,a). A key trick: flipping the
inserted middle bit just negates the first 9 coordinates of the 17-dim arbiter-PUF
embedding, so both candidate feature vectors per CRP are precomputed once.

## Results (synthetic data, seed 42; see RESULTS.md)

| Method | Time | Train acc | Test acc |
|---|---|---|---|
| `my_latent()` | 0.02 s | 1.0000 | — |
| `my_latent_updated()` | 2.0 s | 0.9978 | 0.9975 |

Part 5: the two algorithms **disagree** — cos(w, w̃) ≈ −0.18, b ≠ b̃, and the
fraction with 2zᵢ−1 = sign(ũ·φ(cᵢ)+ã) is ≈ 0.44 (chance). The unconstrained latents
overfit the train responses (their bits match the true middle bits only ~56%, chance
up to the global flip symmetry), whereas the PUF-structured prior recovers the true
middle bits ~99.6% of the time and generalizes. Full explanation in the report.

## Note on academic integrity

This repo is a self-study solution. If you are enrolled in CS771, submitting this
work (in whole or part) as your own violates the course's plagiarism policy — and
keeping this repo public before the assignment deadline exposes enrolled students
(and you) to plagiarism-detection matches. Keep it private until after the deadline.
