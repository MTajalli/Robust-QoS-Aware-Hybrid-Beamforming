# Robust and Feasible QoS-Aware mmWave Massive MIMO Hybrid Beamforming

MATLAB simulation code for the paper:

> M. Tajallifar, A. R. Sharafat, and H. Yanikomeroglu, "Robust and Feasible QoS-Aware mmWave Massive MIMO Hybrid Beamforming," *IEEE Transactions on Wireless Communications*, vol. 23, no. 2, pp. 1520–1534, Feb. 2024. DOI: [10.1109/TWC.2023.3290141](https://doi.org/10.1109/TWC.2023.3290141)

- Published version (IEEE Xplore): https://doi.org/10.1109/TWC.2023.3290141
- Preprint (arXiv): https://arxiv.org/abs/2607.16306

## Overview

The paper studies QoS-aware hybrid beamforming (HB) for single- and multi-user mmWave massive MIMO systems when the channel state information (CSI) is imperfect. The goal is to minimize transmit power while guaranteeing a per-stream mean-squared error (MSE) target for every channel inside a norm-bounded uncertainty region (worst-case robustness).

Main ideas implemented in this repository:

- **Effective-channel uncertainty.** The RF precoder/combiner are steered by approximating the eigenvectors of the estimated channel (two-stage structure). The uncertainty region of the channel is mapped to the uncertainty region of the much smaller effective channel, so less transmit power is needed for the same outage level.
- **Feasibility.** Infeasible data streams are detected and removed, so the problem stays feasible. The remaining streams meet their MSE constraints.
- **Iterative scheme.** Based on the cutting-set method. It alternates between worst-case MSE maximization (an SDP) and transmit-power minimization over a finite set of sampled channels (a QCQP), with elastic variables to detect infeasible streams.
- **Low-complexity scheme.** Block diagonalization at the baseband precoder, MMSE combining at the baseband combiner, a simple sufficient feasibility condition per stream, and closed-form power allocation.
- Both **fully-connected (FCS)** and **partially-connected (PCS)** RF structures are supported.

## Requirements

- MATLAB (R2018a or newer recommended)
- [CVX](http://cvxr.com/cvx/) (version 2.2)
- [MOSEK](https://www.mosek.com/) solver, used through CVX (`cvx_solver mosek`). Free academic licenses are available.
- MATLAB toolboxes:
  - Parallel Computing Toolbox (`parpool`, `parfor`)
  - Curve Fitting Toolbox (`smooth`, used for plotting)
  - Signal Processing Toolbox (`dftmtx`, used by the DFT-based baseline)

## Getting started

1. Install CVX and MOSEK, and make sure `cvx_solver mosek` works.
2. Clone or download this repository and set it as the MATLAB working folder (all files are in the root folder, so no path setup is needed).
3. Run one of the `main_*.m` scripts listed below. Each script runs the simulation and then plots the figure.

**Notes**

- Several scripts open a parallel pool with a fixed number of workers (for example, `parpool('local', 10)`). If your machine has fewer cores, reduce the number or remove the `parpool` line (`parfor` then runs serially). If a pool is still open from a previous script, close it first with `delete(gcp('nocreate'))`.
- No random seed is fixed, so channel realizations and therefore curves vary slightly from run to run.
- The SDP-based and iterative methods call CVX many times and can take a long time for large systems.

## Reproducing the figures

| Figure in paper | Script | Output (in `Figures/`) | System setup |
|---|---|---|---|
| Fig. 3: Acceptance ratio vs. ρ | `main_AR_SU.m` | `AR_SU` | Single user, 20×8, N<sub>s</sub> = 2, N<sub>RF</sub> = 2 |
| Fig. 4: Transmit power vs. ρ (single user) | `main_TP_SU.m` | `TP_vs_rho_SU` | Single user, 20×8, N<sub>s</sub> = 2, N<sub>RF</sub> = 4 (uses `H_sdp.mat`) |
| Fig. 5: Worst-case spectral efficiency vs. SNR | `main_SE_SU.m` | `SE_SNR` | Single user, 20×8, N<sub>s</sub> = 2, N<sub>RF</sub> = 4 (uses `H_sdp.mat`) |
| Fig. 6: Transmit power vs. N<sub>RF</sub> | `main_TPvsNRF_SU.m` | `TP_vs_NRF` | Single user, 20×8, N<sub>s</sub> = 2, N<sub>RF</sub> = 2…8 (uses `H_sdp.mat`) |
| Fig. 7: CDF of achieved MSE | `main_CDF_MU.m` | `CDF_MU` | 2 users, 64×16, N<sub>t</sub><sup>RF</sup> = 4, N<sub>r</sub><sup>RF</sup> = 2, ρ = 0.2 |
| Fig. 8: Outage probability vs. σ<sub>e</sub> | `main_Outage_MU.m` | `Outage_MU` | 2 users, 64×16, N<sub>t</sub><sup>RF</sup> = 4, N<sub>r</sub><sup>RF</sup> = 2, ρ = 0.2 |
| Fig. 9: Transmit power vs. ρ (multi-user) | `main_TP_MU.m` | `TP_vs_rho_MU` | 4 users, 100×16, N<sub>t</sub><sup>RF</sup> = 12, N<sub>r</sub><sup>RF</sup> = 3 |
| Fig. 10: Convergence of the cutting-set method | `main_CS_Convergence.m` | `CUT_Convergence` | 3 users, 128×36, N<sub>s</sub> = 3, N<sub>t</sub><sup>RF</sup> = 9, N<sub>r</sub><sup>RF</sup> = 3 |

The system parameters are set at the top of each script. Each figure is provided in `Figures/` as `.fig` (editable MATLAB figure), `.eps` and `.png`.

### Single-user simulations: stored channel vs. Monte Carlo

By default, the single-user scripts `main_TP_SU.m`, `main_SE_SU.m` and `main_TPvsNRF_SU.m` load the stored channel in `H_sdp.mat`. This is one 8×20 realization of the clustered ULA channel. For this channel, the SDP-based scheme and all HB methods are feasible (all data streams are accepted), so these scripts always run to the end and reproduce the figures in the paper. Keep N<sub>t</sub> = 20 and N<sub>r</sub> = 8 when using the stored channel.

To run a **Monte Carlo** simulation over random channel realizations instead, generate the channels with `channel_generation_ULA.m` and do not load `H_sdp.mat`:

**`main_TP_SU.m` and `main_TPvsNRF_SU.m`**

1. Comment out the two lines `load('H_sdp')` and `H = H_sdp;`.
2. Uncomment the block above them that generates `N_max` channel realizations:
   ```matlab
   H = zeros(N_r, N_t, U, N_max);
   for u = 1:U
       for k = 1:N_max
           [H(:,:,u,k), ~] = channel_generation_ULA(N_t, N_r, N_cl);
       end
   end
   ```
3. Set `N_it` to the number of channel realizations to average over (keep `N_max` larger than `N_it`).
4. Replace the remaining references to `H_sdp` with the current realization, `H(:,:,:,inf_sdp+k)`. These are in the `jsac_joint(...)` call and in the fallback lines inside the `while` loops.

For a fair comparison, the scripts skip any realization where the SDP-based scheme is infeasible, or where a partially-connected scheme cannot accept all data streams. They then move on to the next realization (counters `inf_sdp`, `inf_it`, `inf_lc`), so every method is averaged over the same set of channels.

**`main_SE_SU.m`**

Here `H` is used as a single matrix, and `N_it` is the number of channel-error samples per channel. Replace `load('H_sdp')` and `H = H_sdp;` with

```matlab
[H, ~] = channel_generation_ULA(N_t, N_r, N_cl);
```

To average over several channels, repeat the simulation for multiple realizations and average the resulting transmit powers (`P_*`) and rates (`Rate_*`).

**`main_AR_SU.m`** already generates `N_it` random channels with `channel_generation_ULA.m`, so it is a Monte Carlo simulation as provided.

<p align="center">
  <img src="Figures/AR_SU.png" width="45%" alt="Fig. 3: Acceptance ratio"/>
  <img src="Figures/TP_vs_rho_MU.png" width="45%" alt="Fig. 9: Transmit power, multi-user"/>
</p>

## Code structure

**Proposed schemes**

| File | Description |
|---|---|
| `fun_iterative.m` | Iterative robust HB (cutting-set method with elastic feasibility detection) |
| `fun_low_complexity.m` | Low-complexity robust HB (block diagonalization, MMSE combining, closed-form power allocation) |
| `analog_beamforming_mu.m` | RF precoder/combiner for the fully-connected structure (eigenvector approximation) |
| `analog_beamforming_mu_pcs.m` | RF precoder/combiner for the partially-connected structure |
| `sampled_mu.m` | Baseband design over the sampled channel set (alternating minimization) |
| `bb_precoding_mu.m` | Baseband precoder: power minimization with elastic MSE constraints (CVX) |
| `bb_combining_mu.m` | Baseband combiner: min-max MSE over sampled channels (CVX) |
| `worst_case_mu.m` | Worst-case effective-channel error per stream (SDP, CVX) |
| `Feasibility_mu.m` | Sufficient feasibility condition and removal of infeasible streams |
| `PA_lesscomplex_mu.m` | Closed-form robust power allocation for the low-complexity scheme |

**Benchmarks**

| File | Description |
|---|---|
| `fun_sdp.m`, `fd_precoder.m`, `fd_precoder_initial.m`, `fd_combiner.m` | Robust fully digital SDP-based beamforming (Ref. [33] in the paper) |
| `fun_sdp_feasible.m` | Feasibility check of the SDP-based scheme |
| `jsac_joint.m` | Non-robust joint hybrid precoding/combining (Ref. [7]) and the optimal fully digital power |
| `non_robust_dft.m` | Non-robust HB with DFT-based RF beamformers and block diagonalization (Ref. [5]) |
| `non_robust_one.m` | Non-robust single-stage HB (Ref. [6]) |
| `non_robust_art.m` | Transmit power of the baselines in Refs. [5] and [6] (alternative, not used by the main scripts) |
| `arab.m` | ARAB hybrid beamforming (Ref. [8]) |

**Channel models and performance metrics**

| File | Description |
|---|---|
| `channel_generation.m` | Clustered (extended Saleh–Valenzuela) channel, uniform planar arrays |
| `channel_generation_ULA.m` | Clustered (extended Saleh–Valenzuela) channel, uniform linear arrays |
| `laprand.m` | Laplacian random number generator for angles of arrival/departure |
| `cdf_mu.m`, `cdf_square_mu.m` | CDF of MSE for norm-bounded and hypercube channel errors |
| `outage_mu.m` | Outage probability Pr{MSE > ρ} under Gaussian channel errors |
| `sumrate.m` | Achievable rate for given precoder/combiner |
| `H_sdp.mat` | Stored 8×20 channel realization (feasible for all methods) used by the single-user scripts; see [Monte Carlo](#single-user-simulations-stored-channel-vs-monte-carlo) |

## Citation

If you use this code in your research, please cite:

```bibtex
@article{Tajallifar2024Robust,
  author  = {Tajallifar, Mohsen and Sharafat, Ahmad R. and Yanikomeroglu, Halim},
  title   = {Robust and Feasible {QoS}-Aware {mmWave} Massive {MIMO} Hybrid Beamforming},
  journal = {IEEE Transactions on Wireless Communications},
  volume  = {23},
  number  = {2},
  pages   = {1520--1534},
  year    = {2024},
  doi     = {10.1109/TWC.2023.3290141}
}
```

Related work by the same authors (the two-stage HB structure with perfect CSI):

> M. Tajallifar, A. R. Sharafat, and H. Yanikomeroglu, "QoS-Aware Hybrid Beamforming With Minimal Power in mmWave Massive MIMO Systems," *IEEE Access*, vol. 9, pp. 164668–164680, 2021.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE).

## Contact

For questions or bug reports, please open an issue in this repository.
