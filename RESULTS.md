# Recorded Results

Data: synthetic, generated exactly per the assignment's described process
(hidden 16-bit arbiter PUF → middle bit → insert at position 9 → 17-bit arbiter
PUF → response → middle bit erased). 8000 train / 2000 test CRPs.
Machine: sandbox Linux, single core, numpy 2.4.4 / scikit-learn 1.8.0.

## Canonical run (data seed 42)

| Quantity | Value |
|---|---|
| `my_latent()` wall time | **0.02 s** |
| `my_latent()` train accuracy (own latents) | **1.0000** |
| `my_latent_updated()` wall time | **2.02 s** |
| `my_latent_updated()` train accuracy (predicted latents) | **0.9978** |
| `my_latent_updated()` **test** accuracy (official pipeline) | **0.9975** |

### Part-5 quantities (seed 42)

| Quantity | Value |
|---|---|
| cos(w, w̃) | −0.1825 |
| cos(flip(w), w̃) (accounting for the z→1−z flip symmetry) | +0.2346 |
| b vs b̃ | +0.2212 vs +0.9758 |
| Fraction of train CRPs with 2zᵢ−1 = sign(ũᵀφ(cᵢ)+ã) | 0.4354 (≈ chance) |
| `my_latent()` zᵢ vs true middle bits (up to global flip) | 0.5650 (≈ chance) |
| `my_latent_updated()` predicted ẑᵢ vs true middle bits (up to flip) | 0.9961 |

## Robustness across data seeds

| Seed | Alg-1 time / train acc | Alg-2 time / train acc / test acc | Part-5 fraction |
|---|---|---|---|
| 42 | 0.02 s / 1.0000 | 2.02 s / 0.9978 / 0.9975 | 0.4354 |
| 7 | 0.02 s / 0.9999 | 3.43 s / 0.9840 / 0.9800 | 0.4993 |
| 123 | 0.03 s / 1.0000 | 3.43 s / 0.9968 / 0.9945 | 0.5365 |
| 2026 | 0.02 s / 1.0000 | 2.69 s / 0.9960 / 0.9940 | 0.3950 |

## Takeaways

1. Both algorithms converge in a handful of alternations (see `convergence.png`).
2. The unconstrained algorithm (`my_latent`) achieves perfect train accuracy almost
   instantly, but its latent bits are uncorrelated with the true middle bits —
   overfitting through free latent variables.
3. The PUF-structured algorithm (`my_latent_updated`) recovers the true latent
   structure (~99%+ up to a global flip) and therefore generalizes to test CRPs.
4. The two algorithms consistently disagree on (w, b) and on the latents — the
   Part-5 fraction hovers around 0.4–0.55 (chance) in every run.
