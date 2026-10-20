---
title: "Noise Schedules"
tags: ["noise-schedules", "diffusion-models", "signal-to-noise-ratio", "loss-weighting", "training-dynamics"]
last_updated: "2026/08/22"
---

## Summary

A noise schedule specifies how much noise corrupts the data at each point of a diffusion process, $x_t = \alpha(t) x_0 + \sigma(t)\epsilon$. It sets which noise levels — and thus which feature scales — the model spends capacity on, making it a central design lever for [[diffusion_models]]. A key modern result is that the continuous-time training loss depends only on the schedule's *endpoints* and its induced *weighting* over noise levels, not its shape, so the schedule is best understood as one of four interacting factors (with model parameterization, loss weighting, and timestep distribution) that jointly determine noise-level importance. Practical failures — residual terminal signal, resolution mismatch — trace to mis-set schedules, and adaptive schemes now tune them automatically.

## Related Concepts
[[diffusion_models]]
[[score_matching]]
[[diffusion_samplers]]
[[flow_matching]]
[[variational_autoencoders]]
[[latent_diffusion_models]]
[[diffusion_guidance]]
[[diffusion_geometry]]

## Open Questions

The cosine schedule works well empirically, and a Fisher-information derivation surprisingly recovers it, hinting at a theoretical basis not yet fully understood {2024_Dieleman_NoiseSchedules}. There is no principled recipe for jointly setting schedule, parameterization, loss weighting, and timestep distribution, and the optimal weighting differs between training and sampling {2024_Dieleman_NoiseSchedules}. Endpoints demonstrably matter — common schedules wrongly leave signal at the terminal step, biasing brightness {2024_Lin_ZeroSNR} — but how to set them per resolution and data type is still empirical {2023_Chen_NoiseScheduling}. Whether the VP/VE/sub-VP distinction is substantive or a reparameterization is subtle: it is immaterial to the loss via logSNR, yet it changes sampling-path curvature {2024_Gao_FlowMatching}.

## Synthesis

### What a noise schedule is

A diffusion process corrupts data as $x_t = \alpha(t) x_0 + \sigma(t)\epsilon$, $\epsilon \sim \mathcal{N}(0,I)$; the function $\sigma(t)$ is the noise schedule and $\alpha(t)$ its companion signal scaling {2024_Dieleman_NoiseSchedules}. Because Gaussian noise composes, one can jump to any level in one step, which is what makes diffusion trainable. Three families differ in how they relate $\alpha$ and $\sigma$: **variance-preserving (VP)**, $\alpha = \sqrt{1 - \sigma^2}$, keeps $\mathrm{Var}[x_t]$ fixed and underlies DDPM {2020_Ho_DDPM}; **variance-exploding (VE)**, $\alpha = 1$ with $\sigma$ growing large, is used by NCSN and EDM {2022_Karras_EDM}; **sub-VP / flow-matching**, $\alpha = 1 - \sigma$, targets straighter sampling paths and is used by Stable Diffusion 3 {2024_Dieleman_NoiseSchedules}{2021_Song_SDE}. All three are discretizations of the same SDE family ([[score_matching]]).

### logSNR as the universal currency

Every schedule is monotonic in the **log signal-to-noise ratio** $\lambda = \log(\alpha(t)^2 / \sigma(t)^2)$, which therefore serves as a schedule-independent axis for comparing settings {2024_Dieleman_NoiseSchedules}{2022_Luo_Understanding}. Named schedules become curves in $\lambda$: the **linear** schedule (DDPM, specified via per-step variances $\beta_t$) {2020_Ho_DDPM}, the **cosine** schedule (Nichol & Dhariwal — that paper is unread here and is characterized via secondary descriptions — which preserves signal longer) {2024_Dieleman_NoiseSchedules}, and the **trivial** $\sigma(t) = t$ of EDM, chosen precisely to demote the schedule and expose the other design choices {2022_Karras_EDM}. Since $\sigma(t)$ is monotonic and invertible, the time variable can be eliminated entirely in favor of $\sigma$ or $\lambda$ — the schedule adds no expressive power, only a change of variables {2024_Dieleman_NoiseSchedules}.

### The schedule is (mostly) a superfluous abstraction

Dieleman's central argument is that noise-level importance is set not by the schedule alone but by the interaction of four factors: the schedule, the model output **parameterization** ($x_0$, $\epsilon$, $v$, or score), the **loss weighting** $w(t)$, and the **timestep distribution** $p(t)$ {2024_Dieleman_NoiseSchedules}. Switching from $x_0$- to $\epsilon$-prediction, for instance, silently multiplies the effective loss weighting by $\mathrm{SNR}(t)$, so a schedule tuned for one parameterization does not transfer {2024_Dieleman_NoiseSchedules}. Moreover $p(t)$ and $w(t)$ are interchangeable — any non-uniform timestep distribution can be absorbed into the weighting — so the practical reason to shape $p(t)$ is variance reduction of the minibatch loss estimate, not expressiveness {2024_Dieleman_NoiseSchedules}. The recommended mental model is to reason directly in $\sigma$ or $\lambda$ and tune weighting and step-spacing separately, because centering the schedule entangles training and sampling.

### Schedule-invariance of the loss

The result that licenses this reframing is from variational diffusion models: written as an integral over $\lambda$, the continuous-time diffusion loss $L = \int w(\lambda)\, \mathbb{E}[\|\epsilon' - \epsilon\|^2]\, d\lambda$ depends only on the endpoints $\lambda_{\max}$, $\lambda_{\min}$ and the weighting $w(\lambda)$ — the schedule's shape in between cancels {2023_Kingma_VDM}. Kingma & Gao extend this across models: all common diffusion objectives share this form and differ *only* in $w(\lambda)$, and under a monotonic weighting the objective equals the ELBO with noise augmentation {2023_Kingma_ELBO}. The invariance is exact for the loss but **not** for its Monte-Carlo estimator: the schedule reappears as an importance-sampling distribution controlling gradient variance, which is why adaptive schedules speed training without changing what is learned {2023_Kingma_VDM}{2023_Kingma_ELBO}{2024_Gao_FlowMatching}.

### Training versus sampling, and step spacing

The optimal weighting of noise levels differs between the two phases. In **training** it is governed by perceptual relevance and the difficulty of the denoising task at each level; in **sampling** it also depends on prediction accuracy and error accumulation, with errors at high noise levels (early steps) more consequential {2024_Dieleman_NoiseSchedules}. Once trained, only the **spacing** of noise levels at inference remains adjustable, and uniform spacing in $t$ gives non-uniform spacing in noise level: EDM deliberately concentrates steps at low noise {2022_Karras_EDM}, while DPM-Solver found uniform spacing in $\lambda$ optimal {2024_Dieleman_NoiseSchedules}. This links schedule design to solver design in [[diffusion_samplers]].

### The zero-terminal-SNR flaw

A concrete demonstration that endpoints matter: common implementations clip the final step so a small fraction of signal survives at $t = T$ — Stable Diffusion's terminal $x_t$ retains about 6% of $x_0$ — so the model learns to read residual low-frequency cues (e.g. mean brightness) during training, then at inference, fed pure zero-mean noise, defaults to medium-brightness images regardless of the prompt {2024_Lin_ZeroSNR}. The fix is a package: rescale the schedule to enforce zero terminal SNR, switch to **v-prediction** (noise-prediction becomes degenerate and numerically singular at $\mathrm{SNR} = 0$), start the sampler from the true final step ("trailing" spacing, which helps most at low step counts), and rescale classifier-free guidance to prevent over-exposure {2024_Lin_ZeroSNR}. This connects noise schedules to [[diffusion_guidance]] and sampler step-spacing.

### Resolution and adaptive schedules

Noise-level importance is data-dependent. Larger images have more redundant pixels, so a fixed schedule destroys less perceptual information at a given $t$; the effective remedy is simply to scale the input by a factor $b$, shifting the logSNR curve rather than reshaping the schedule — a single hyperparameter that outperforms schedule-shape changes and whose optimum decreases with image size {2023_Chen_NoiseScheduling}. Stable Diffusion 3 makes the same move, shifting the timestep distribution toward higher noise at higher resolution {2024_Esser_SD3} ([[latent_diffusion_models]]). Rather than tune by hand, several **adaptive** schemes track per-level loss statistics: VDM learns a schedule that minimizes gradient variance at fixed weighting {2023_Kingma_VDM}, Kingma & Gao build an adaptive importance-sampling $p(\lambda)$ while dividing it back out of $w$ to leave the effective objective unchanged {2023_Kingma_ELBO}, and EDM2 rebalances gradient magnitudes across noise levels {2024_Dieleman_NoiseSchedules}.

### Terminology

The literature is notation-heavy and inconsistent, which is much of the motivation for the logSNR reframing. The same process is specified via per-step variances $\beta_t$, cumulative $\bar{\alpha}_t$, SNR, or logSNR; "noise schedule" is used for $\sigma(t)$, for $\beta_t$, and loosely for the whole weighting; VP, VE, and sub-VP name different $\alpha$/$\sigma$ couplings; and the direction convention is inverted between diffusion ($t = 0$ is clean) and flow matching ($t = 0$ is noise) {2024_Dieleman_NoiseSchedules}{2023_Dieleman_Perspectives}.

## References

{2020_Ho_DDPM} 2020_Ho_Denoising_Diffusion_Probabilistic_Models
{2021_Song_SDE} 2021_Song_Score-Based_Generative_Modeling_Through_Stochastic_Differential_Equations
{2022_Karras_EDM} 2022_Karras_Elucidating_the_Design_Space_of_Diffusion-Based_Generative_Models
{2022_Luo_Understanding} 2022_Luo_Understanding_Diffusion_Models_A_Unified_Perspective
{2023_Chen_NoiseScheduling} 2023_Chen_On_the_Importance_of_Noise_Scheduling_for_Diffusion_Models
{2023_Dieleman_Perspectives} 2023_Dieleman_Perspectives_on_Diffusion
{2023_Kingma_ELBO} 2023_Kingma_Understanding_Diffusion_Objectives_as_the_ELBO_with_Simple_Data_Augmentation
{2023_Kingma_VDM} 2023_Kingma_Variational_Diffusion_Models
{2024_Dieleman_NoiseSchedules} 2024_Dieleman_Noise_Schedules_Considered_Harmful
{2024_Esser_SD3} 2024_Esser_Scaling_Rectified_Flow_Transformers_for_High-Resolution_Image_Synthesis
{2024_Gao_FlowMatching} 2024_Gao_Diffusion_Meets_Flow_Matching_Two_Sides_of_the_Same_Coin
{2024_Lin_ZeroSNR} 2024_Lin_Common_Diffusion_Noise_Schedules_and_Sample_Steps_are_Flawed
