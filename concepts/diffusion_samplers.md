---
title: "Diffusion Samplers"
tags: ["diffusion-samplers", "diffusion-models", "ode-solvers", "distillation", "fast-sampling"]
last_updated: "2026/08/22"
---

## Summary

A diffusion sampler is the procedure that turns a trained denoiser into images by numerically integrating the reverse process from noise to data. Because training fixes only the score, the sampler is a separate design choice, and the field's central problem is cutting the many network evaluations diffusion needs. Two broad approaches exist: **training-free solvers** that discretize the reverse SDE or probability-flow ODE more cleverly (DDIM, DPM-Solver, DEIS, higher-order and predictor-corrector methods), reaching good samples in ~10-20 steps; and **distillation** that trains a student for few- or single-step generation (progressive, consistency, rectified-flow, adversarial). This node covers the reverse process of [[diffusion_models]]; the forward side is [[noise_schedules]] and the objective is [[score_matching]].

## Related Concepts
[[diffusion_models]]
[[score_matching]]
[[langevin_dynamics]]
[[noise_schedules]]
[[flow_matching]]
[[knowledge_distillation]]
[[latent_diffusion_models]]
[[diffusion_guidance]]
[[diffusion_based_image_editing]]

## Open Questions

How few steps can preserve quality without a teacher is unresolved: training-free solvers plateau, and distillation buys speed by trading variance for bias — quality or diversity — with no free lunch {2024_Dieleman_Paradox}. Whether stochastic sampling is necessary is contested: injected noise corrects accumulated error but its value shrinks as models improve {2022_Karras_EDM}. Guided sampling destabilizes higher-order solvers at large guidance scales and pushes predictions out of the data range; thresholding is a patch rather than a principled fix {2025_Lu_DPMSolverpp}. And whether the straight-path limit that unifies consistency models, rectified flow, and diffusion is the right target for few-step generation is an open and active direction {2024_Dieleman_Paradox}.

## Synthesis

### Sampling as solving the reverse process

The trained score defines a **reverse-time SDE** and an equivalent deterministic **probability-flow ODE** with the same marginals; sampling is numerically integrating either from $t = T$ to $t = 0$, and the discretization is a sampling-time choice independent of training {2021_Song_SDE}. The original **ancestral sampler** (DDPM) is one discretization, injecting fresh Gaussian noise at each step {2020_Ho_DDPM}. Each step predicts $\mathbb{E}[x_0|x_t]$ and moves a small distance toward it: the prediction only points to a *region* of high likelihood, so a large step lands at a blurry average, and because the direction changes as uncertainty resolves, the sampling path is **curved** and must be followed in many small steps {2024_Dieleman_Paradox}. Cutting that step count without losing quality is the problem the whole node addresses.

### DDIM: deterministic sampling and inversion

DDIM reformulates the forward process as a family of **non-Markovian** processes sharing DDPM's training objective, so a pretrained DDPM can be resampled without retraining {2022_Song_DDIM}. A stochasticity parameter $\sigma_t$ interpolates between DDPM (fully stochastic) and **DDIM** ($\sigma_t = 0$, deterministic); the deterministic limit needs far fewer steps for good samples and, being a discretization of an ODE, makes the map from $x_T$ to $x_0$ consistent {2022_Song_DDIM}. Determinism has two consequences exploited widely: simple interpolation in $x_T$ yields smooth semantic interpolation in image space, and the ODE can be run forward to **encode** a real image to its latent $x_T$ — DDIM inversion, the basis of much diffusion-based editing {2022_Song_DDIM}{2021_Song_SDE} ([[diffusion_based_image_editing]]). DDIM is the first-order Euler discretization of the reparameterized sampling ODE {2024_Gao_FlowMatching}.

### Training-free ODE solvers

Faster solvers exploit the **semilinear** structure of the diffusion ODE — a linear term in the data plus a nonlinear neural term. **DPM-Solver** solves the linear part *exactly* via an exponential integrator and Taylor-expands only the noise prediction, cutting to ~10 steps; its first-order case is exactly DDIM {2022_Lu_DPMSolver}. **DEIS** combines the same exponential integrator with noise-prediction and polynomial (Adams-Bashforth) extrapolation of past evaluations, again recovering DDIM as a special case {2023_Zhang_DEIS}. **DPM-Solver++** derives higher-order solvers on the $x_0$-prediction ODE because noise-prediction high-order solvers become unstable at large guidance scales, and adds thresholding and multistep variants {2025_Lu_DPMSolverpp}. The unifying view: all training-free samplers discretize the diffusion ODE or SDE, differing in integrator order, parameterization, and step spacing {2025_Lu_DPMSolverpp}. Step spacing is itself a tunable, with EDM concentrating steps at low noise and DPM-Solver preferring uniform logSNR spacing ([[noise_schedules]]).

### Stochasticity, higher order, and correction

Two orthogonal knobs improve accuracy per step. **Order**: second-order methods such as Heun's give the best truncation-error/compute trade-off, and multistep methods act like momentum by aggregating past evaluations {2022_Karras_EDM}{2024_Dieleman_Paradox}. **Stochasticity**: Karras generalizes sampling as a probability-flow ODE plus a Langevin "churn" term that re-injects and removes noise, driving samples back toward the correct marginal and correcting accumulated error — analogous to SGD's regularizing noise — though its benefit diminishes as the model improves {2022_Karras_EDM}{2024_Gao_FlowMatching}. The **predictor-corrector** samplers of the SDE framework combine a numerical predictor step with Langevin-dynamics correction ([[langevin_dynamics]]), outperforming either alone {2021_Song_SDE}{2021_Song_Blog}. The stochastic-versus-deterministic choice (DDPM churn 1, DDIM churn 0, with a continuum between) is thus a quality/consistency trade rather than a fixed property {2024_Gao_FlowMatching}.

### Distillation for few- and single-step sampling

When solvers hit their floor, **distillation** trains a student to skip steps, converging faster than from-scratch training because the teacher already outputs the target expectation $\mathbb{E}[x_0|x_t]$, reducing loss variance at the cost of some bias {2024_Dieleman_Paradox}. **Progressive distillation** repeatedly halves the step count by training a student to match two teacher steps in one, reaching single-step in $\log_2 N$ rounds, and introduced v-prediction for stability {2024_Dieleman_Paradox}. **Consistency models** instead predict the ODE trajectory's endpoint from any point on it, enforcing a self-consistency property with a boundary condition, enabling one-step sampling and trainable either by distillation or from scratch {2023_Song_CM}; **Latent Consistency Models** apply this in Stable Diffusion's latent space for 1-4 step high-resolution generation {2023_Luo_LCM} ([[latent_diffusion_models]]). Other threads: **rectified-flow reflow** replaces the paths with straighter ones rather than approximating existing ones ([[flow_matching]]), **guidance distillation** folds classifier-free guidance into one pass, and **adversarial distillation** (ADD, used for SDXL-Turbo) adds a discriminator to sharpen few-step outputs {2024_Dieleman_Paradox}. All are collected under [[knowledge_distillation]].

### The paradox and its trade-offs

There is a tension at the heart of this work: diffusion's strength is splitting generation into many small steps, yet distillation collapses that back toward one {2024_Dieleman_Paradox}. The resolution is that distillation is never free — it trades quality (blurriness) or diversity (mode-dropping) for speed, but gives control over which {2024_Dieleman_Paradox}. Whether a distilled few-step model could simply have been trained directly depends on method: consistency training works without a teacher but less well, because the teacher's multi-step computation provides a learning signal hard to replicate {2024_Dieleman_Paradox}{2023_Song_CM}. In the limit of perfectly straight paths, consistency models and diffusion make identical predictions, suggesting a deeper unity and tying few-step sampling to the optimal-transport view of [[flow_matching]].

## References

{2020_Ho_DDPM} 2020_Ho_Denoising_Diffusion_Probabilistic_Models
{2021_Song_Blog} 2021_Song_Generative_Modeling_by_Estimating_Gradients_of_the_Data_Distribution
{2021_Song_SDE} 2021_Song_Score-Based_Generative_Modeling_Through_Stochastic_Differential_Equations
{2022_Karras_EDM} 2022_Karras_Elucidating_the_Design_Space_of_Diffusion-Based_Generative_Models
{2022_Lu_DPMSolver} 2022_Lu_DPM-Solver_A_Fast_ODE_Solver_for_Diffusion_Probabilistic_Model_Sampling_in_Around_10_Steps
{2022_Song_DDIM} 2022_Song_Denoising_Diffusion_Implicit_Models
{2023_Luo_LCM} 2023_Luo_Latent_Consistency_Models_Synthesizing_High-Resolution_Images_with_Few-Step_Inference
{2023_Song_CM} 2023_Song_Consistency_Models
{2023_Zhang_DEIS} 2023_Zhang_Fast_Sampling_of_Diffusion_Models_with_Exponential_Integrator
{2024_Dieleman_Paradox} 2024_Dieleman_The_Paradox_of_Diffusion_Distillation
{2024_Gao_FlowMatching} 2024_Gao_Diffusion_Meets_Flow_Matching_Two_Sides_of_the_Same_Coin
{2025_Lu_DPMSolverpp} 2025_Lu_DPM-Solver-pp_Fast_Solver_for_Guided_Sampling_of_Diffusion_Probabilistic_Models
