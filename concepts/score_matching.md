---
title: "Score Matching"
tags: ["score-matching", "diffusion-models", "generative-models", "energy-based-models", "denoising"]
last_updated: "2026/08/22"
---

## Summary

Score matching learns the **score** of a distribution — the gradient of its log-density with respect to the data, $\nabla_x \log p(x)$ — instead of the density itself, sidestepping the intractable normalizing constant that plagues likelihood-based and energy-based models. Because the true score is unknown, it is estimated by tractable surrogates, chiefly *denoising* score matching (predict the noise added to corrupted data) and *sliced* score matching. Estimating scores across many noise levels and sampling by Langevin dynamics gives a generative model, and this score-based view is one of the equivalent foundations of [[diffusion_models]]: DDPM's noise-prediction loss is denoising score matching in disguise. It is the training-objective face of the diffusion subgraph.

## Related Concepts
[[diffusion_models]]
[[noise_schedules]]
[[diffusion_samplers]]
[[langevin_dynamics]]
[[energy_based_models]]
[[noise_contrastive_estimation]]
[[variational_autoencoders]]
[[flow_matching]]
[[normalizing_flows]]
[[optimal_transport]]
[[diffusion_geometry]]
[[membership_inference_attacks]]

## Open Questions

Score estimation is least reliable exactly where it matters: in low-density regions and at low noise, which govern fine detail {2020_Song_NCSN}. The multiple-noise-scale fix works but is arguably a workaround rather than a principled solution, and the optimal loss weighting across noise levels is unresolved — likelihood weighting $g(t)^2$ upper-bounds the model KL yet the noise-MSE weighting that ignores it gives better FID, a likelihood-versus-perceptual-quality tension {2021_Song_SDE}{2021_Song_Blog}. Denoising score matching is exact only as the perturbation vanishes, biasing estimates, while unbiased sliced score matching costs several times more {2020_Song_NCSN}; the practical trade is unsettled. Scores are defined only for continuous distributions, leaving discrete data awkward {2021_Song_Blog}.

## Synthesis

### The score and the normalizing-constant problem

For an energy-based model $p_\theta(x) = \exp(-f_\theta(x)) / Z_\theta$, the intractable partition function $Z_\theta$ blocks maximum-likelihood training. The **score** $s_\theta(x) = \nabla_x \log p_\theta(x) = -\nabla_x f_\theta(x)$ eliminates $Z_\theta$ entirely, since the gradient of a constant is zero — and is invariant to rescaling the density {2021_Song_Blog}{2022_Luo_Understanding}. This frees the model architecture from having to produce a normalized density. Score matching is thus one answer to the unnormalized-model problem; **noise-contrastive estimation** is another, recovering the normalized model by learning to classify data against artificial noise with a logistic objective {2010_Gutmann} ([[noise_contrastive_estimation]], [[energy_based_models]]). (Terminology note: "score" here is the gradient with respect to the *data* $x$ — sometimes the Stein score — not the classical statistical score with respect to parameters.)

### From Fisher divergence to tractable surrogates

Training minimizes the **Fisher divergence** $\mathbb{E}_{p(x)}[ \|s_\theta(x) - \nabla_x \log p(x)\|^2 ]$, but the ground-truth data score is unknown. Integration by parts rewrites this using only samples, as $\mathbb{E}[ \mathrm{tr}(\nabla_x s_\theta(x)) + (1/2)\|s_\theta(x)\|^2 ]$, but the Jacobian trace scales poorly to high dimensions {2020_Song_NCSN}. Two tractable variants avoid it. **Denoising score matching (DSM)** perturbs data with a known kernel $q_\sigma(x'|x)$ and fits the score of the *perturbed* distribution, whose conditional score is closed-form: for Gaussian noise, $\nabla_{x'} \log q_\sigma(x'|x) = (x - x')/\sigma^2$ {2020_Song_NCSN}{2021_Song_Blog}. **Sliced score matching (SSM)** approximates the trace with random projections, unbiased but roughly four times costlier {2020_Song_NCSN}. DSM dominates in practice and is exact only as $\sigma \to 0$.

### Denoising score matching is denoising

DSM's conditional-score target $(x - x')/\sigma^2$ points from the noisy sample back toward the clean one, so following the score *is* denoising {2020_Song_NCSN}. Concretely, a network trained to predict the added noise $\epsilon$ and one trained to predict the clean $x_0$ differ only by a fixed reparameterization given the noisy input, so the score, the noise, and the clean data are three views of one prediction {2022_Dieleman_DMAE}{2022_Luo_Understanding}. This is the precise sense in which a diffusion model "is" a denoising autoencoder, and it ties score matching to the denoising-autoencoder lineage of [[variational_autoencoders]] — Vincent's denoising autoencoders already reconstructed clean signals from corrupted inputs and were shown to optimize a variational bound on a generative model {2008_Vincent}. The equivalence between the DSM and score-matching objectives (same minimizer in expectation) is the enabling result {2022_Dieleman_DMAE}.

### The manifold problem and multiple noise scales

Naive score matching plus Langevin dynamics fails on real data for two coupled reasons: the score is undefined off the low-dimensional data manifold, and the Fisher divergence weights errors by $p(x)$, so scores are badly estimated in the low-density regions where high-dimensional Langevin chains almost always start {2020_Song_NCSN}{2021_Song_Blog}. The fix is to perturb data at **multiple increasing noise levels**: large noise fills in low-density regions and smooths the density (letting Langevin move between separated modes), small noise preserves the true distribution. A single **noise-conditional score network** $s_\theta(x, \sigma)$ is trained jointly across levels with a weighted sum of Fisher divergences, using $\lambda(\sigma) = \sigma^2$ to balance magnitudes {2020_Song_NCSN}. The choice and spacing of these levels is the subject of [[noise_schedules]].

### Sampling: Langevin dynamics

Given a score model, samples are drawn by **Langevin dynamics** ([[langevin_dynamics]]), $x \leftarrow x + (\epsilon/2) s_\theta(x) + \sqrt{\epsilon}\, z$, which needs only the score, not the density {2020_Song_NCSN}. **Annealed** Langevin runs chains from the highest noise level down to the lowest, so the distribution smoothly approaches the data {2020_Song_NCSN}. This Markov-chain corrector reappears in the predictor-corrector samplers of the SDE framework and is one pole of the sampler design space developed in [[diffusion_samplers]].

### Continuous time and the SDE unification

Taking the number of noise scales to infinity turns the discrete annealing into a continuous **diffusion SDE**, and score matching becomes learning a time-dependent score $s_\theta(x, t)$ over a continuum {2021_Song_SDE}. The reverse-time SDE — and its deterministic probability-flow ODE — depend on the data only through this score, so the trained model fully specifies generation while the solver is a separate choice {2021_Song_SDE}{2021_Song_Blog}. The SDE view shows score-based models and DDPM to be discretizations of one process, and Kingma & Gao make the objective side of this precise: every common diffusion loss is a weighted denoising-score-matching integral, and under a monotonic weighting it equals the ELBO with Gaussian-noise augmentation {2023_Kingma_ELBO}. See [[diffusion_models]], [[noise_schedules]].

### Geometry of score-based generation

A geometric reading casts forward and reverse diffusion as **Wasserstein gradient flows** in the space of probability measures: the reverse process is the accelerated gradient flow of the negative KL to the prior, moving mass *away* from the isotropic Gaussian {2023_Ghimire}. This exposes the key contrast with energy-based models, which flow *toward* the data distribution and therefore need an explicit data energy; the score-based process avoids ever handling the data density directly, which is precisely why score matching is tractable {2023_Ghimire}. This connects score matching to [[optimal_transport]] and the geometry threads of [[diffusion_geometry]].

## References

{2008_Vincent} 2008_Vincent_Extracting_and_Composing_Robust_Features_with_Denoising_AutoEncoders
{2010_Gutmann} 2010_Gutmann_Noise-contrastive_estimation_A_new_estimation_principle_for_unnormalized_statistical_models
{2020_Song_NCSN} 2020_Song_Generative_Modeling_by_Estimating_Gradients_of_the_Data_Distribution
{2021_Song_Blog} 2021_Song_Generative_Modeling_by_Estimating_Gradients_of_the_Data_Distribution
{2021_Song_SDE} 2021_Song_Score-Based_Generative_Modeling_Through_Stochastic_Differential_Equations
{2022_Dieleman_DMAE} 2022_Dieleman_Diffusion_Models_are_Autoencoders
{2022_Luo_Understanding} 2022_Luo_Understanding_Diffusion_Models_A_Unified_Perspective
{2023_Ghimire} 2023_Ghimire_Geometry_of_Score_Based_Generative_Models
{2023_Kingma_ELBO} 2023_Kingma_Understanding_Diffusion_Objectives_as_the_ELBO_with_Simple_Data_Augmentation
