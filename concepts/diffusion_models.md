---
title: "Diffusion Models"
tags: ["diffusion-models", "generative-models", "score-based-models", "image-synthesis", "iterative-refinement"]
last_updated: "2026/08/22"
---

## Summary

A diffusion model generates data by learning to reverse a gradual noising process: a fixed forward process corrupts data into Gaussian noise, and a neural network is trained to denoise, so that iterating the reverse process from pure noise produces samples. Introduced from nonequilibrium thermodynamics and made practical by DDPM, diffusion is now the dominant paradigm for perceptual generation (images, audio, video). Its power comes from splitting a hard one-shot sampling problem into many easy denoising steps, generating coarse-to-fine. This node is the hub for the diffusion subgraph, delegating the forward process to [[noise_schedules]], the training objective to [[score_matching]], the reverse process to [[diffusion_samplers]], and compressed-space operation to [[latent_diffusion_models]].

## Related Concepts
[[noise_schedules]]
[[score_matching]]
[[diffusion_samplers]]
[[latent_diffusion_models]]
[[generative_models]]
[[variational_autoencoders]]
[[generative_adversarial_networks]]
[[normalizing_flows]]
[[flow_matching]]
[[diffusion_guidance]]
[[diffusion_geometry]]
[[diffusion_language_models]]
[[diffusion_model_semantic_spaces]]
[[diffusion_based_image_editing]]
[[diffusion_model_personalization]]
[[generated_image_detection]]
[[generated_image_evaluation_metrics]]
[[aes_for_generative_models]]
[[membership_inference_attacks]]

## Open Questions

Why diffusion models *generalize* rather than memorize is unsettled: independently trained networks converge to nearly the same score, implying strong, well-matched inductive biases {2024_Kadkhodaie}, yet the biases are not aligned with the low-dimensional manifold they supposedly exploit {2024_Kadkhodaie}, and the transition from memorization to generalization with dataset size is only partly characterized. Models provably interpolate between training modes, producing out-of-support "hallucinations" {2024_Aithal}. What a model actually learns about data geometry — manifold dimension {2023_Stanczuk}, curvature, spectra — is an active theory frontier ([[diffusion_geometry]]). Terminologically, "diffusion" has drifted toward a synonym for any iterative refinement, blurring what the term denotes {2023_Dieleman_Perspectives}.

## Synthesis

### What a diffusion model is

A diffusion model pairs a fixed **forward process** that adds Gaussian noise to data $x_0$ over a continuum of levels, $x_t = \alpha(t) x_0 + \sigma(t) \epsilon$, with a learned **reverse process** that removes it {2015_SohlDickstein}{2020_Ho_DDPM}. Because each denoising step is small, the reverse transition $p(x_{t-1} | x_t)$ is well-approximated as Gaussian, so a network need only predict its mean — equivalently, predict the added noise, the clean data, or the score {2020_Ho_DDPM}{2022_Luo_Understanding}. Sampling starts from $x_T \sim \mathcal{N}(0,I)$ and iterates the learned denoiser back to $t=0$. The defining trade is compute for tractability: a hard one-shot generative problem is decomposed into many easy ones, at the cost of many network evaluations per sample — the central tension that [[diffusion_samplers]] exists to relieve.

### Historical evolution

Diffusion originated in **nonequilibrium thermodynamics**: Sohl-Dickstein et al. built a Markov chain converting data to a known prior and back, learning the reverse kernels by a variational bound and even learning the noise rate {2015_SohlDickstein}. **DDPM** made it practical, fixing the schedule, adopting a simple noise-prediction MSE loss, and matching GAN sample quality {2020_Ho_DDPM}. Concurrently, **score-based models** arrived from a different direction — estimating $\nabla \log p$ at many noise levels and sampling by Langevin dynamics (NCSN) {2020_Song_NCSN} — and the two lineages were unified by the **SDE framework**, which shows DDPM and score matching are discretizations of one continuous-time process {2021_Song_SDE}. Subsequent milestones added **guidance** for controllable, high-fidelity conditional generation {2021_Dhariwal}{2022_Ho_CFG}, moved generation into learned **latent spaces** ([[latent_diffusion_models]]), and recast the process as **flow matching** ([[flow_matching]]). The DDPM/score/SDE convergence is the field's foundational synthesis {2021_Song_Blog}{2022_Luo_Understanding}.

### One model, many perspectives

A recurring theme is that the same trained network supports several equivalent framings, each illuminating different aspects {2023_Dieleman_Perspectives}. As a **denoising autoencoder** it maps noisy inputs to clean data, but without a bottleneck and with the noise level as an explicit input {2022_Dieleman_DMAE}. As a **deep latent-variable model** (DDPM) it is a hierarchical VAE with fixed Gaussian encoders, fit by an ELBO {2022_Luo_Understanding} — a link developed under [[variational_autoencoders]]. As a **score / reverse-SDE** model it estimates $\nabla \log p$ and decouples training from sampling ([[score_matching]]). As a **probability-flow ODE** it is a continuous normalizing flow with a deterministic, invertible data-to-noise map and exact likelihoods {2021_Song_SDE}, generalizing to [[flow_matching]]. All the common training targets — noise $\epsilon$, clean $x_0$, $v$, or score — are linear reparameterizations of one another given $x_t$, so any can be converted to any other after training {2022_Luo_Understanding}{2023_Dieleman_Geometry}; which is used matters for numerical stability and implicit loss weighting, developed under [[noise_schedules]].

### Coarse-to-fine generation

Different noise levels govern different feature scales: high noise destroys and thus controls large-scale structure, low noise controls fine detail, because large-scale features survive pixel-wise noise while small-scale ones are drowned out {2022_Dieleman_DMAE}. Diffusion therefore generates coarse-to-fine, resolving global layout early in sampling and detail late {2020_Ho_DDPM} — approximately autoregression in frequency space, since natural images have $1/f$ spectra and Gaussian noise is spectrally flat {2023_Dieleman_Perspectives}. This is also why the models are compute-efficient for perceptual data (humans are most sensitive to the low frequencies that dominate the signal) and why the relative weighting of noise levels is the key design lever, treated in [[noise_schedules]].

### Guidance

Conditional diffusion is sharpened by **guidance**, which trades diversity for fidelity. **Classifier guidance** adds the gradient of a noisy-image classifier to the score, $\nabla \log p(x_t) + \gamma \nabla \log p(y|x_t)$, scaling $\gamma$ to concentrate probability on the conditioned class and letting diffusion surpass GANs on FID/IS {2021_Dhariwal}. **Classifier-free guidance (CFG)** removes the auxiliary classifier by training one network with conditioning dropout and extrapolating between conditional and unconditional predictions, $(1+w)\epsilon(x_t,c) - w\epsilon(x_t)$, which works better in practice but requires the conditioning at training time {2022_Ho_CFG}. Geometrically, guidance is a second update direction added at each step, and too-large $\gamma$ pushes samples off the data manifold into distorted, over-saturated regions {2023_Dieleman_Geometry} — a failure mode that motivates thresholding and sampler-side fixes ([[diffusion_samplers]]). Guidance and its many downstream uses are collected under [[diffusion_guidance]].

### Positioning among generative families

Diffusion is one of the three dominant deep generative paradigms and is framed largely by contrast. Against [[generative_adversarial_networks]], it is stable to train and covers modes well but samples slowly; guided diffusion surpasses GANs on both fidelity and diversity {2021_Dhariwal}. Against [[variational_autoencoders]], it shares the ELBO/latent-variable formalism (a diffusion model is an infinite-depth VAE {2023_Kingma_VDM}) but produces far sharper samples. Its chief liability is sampling cost, addressed by fast solvers and distillation ([[diffusion_samplers]]) and by moving to compact latents ([[latent_diffusion_models]]). Diffusion has since spread beyond continuous perceptual data to discrete sequences ([[diffusion_language_models]]) and to applied control of pretrained generators — editing existing images ([[diffusion_based_image_editing]]) and few-shot personalization to new concepts ([[diffusion_model_personalization]]); the broad three-way tension among quality, coverage, and sampling cost is the organizing frame for [[generative_models]].

### What diffusion models learn about data

A growing analysis literature probes the geometry a trained model encodes. The score near the data manifold points along normal directions, so the singular spectrum of score estimates recovers the manifold's local dimension {2023_Stanczuk}. Generalization coincides with **low model variance** — networks trained on disjoint data learn the same score and generate near-identical samples once the dataset is large — and with an inductive bias toward "geometry-adaptive harmonic" bases that give sparse image representations {2024_Kadkhodaie}. But the same smoothing that yields generalization also causes **mode interpolation**: models place mass between well-separated training modes, generating semantically incoherent "hallucinations" detectable by high variance in the sampling trajectory {2024_Aithal}. These threads — geometry, generalization, memorization — are gathered under [[diffusion_geometry]].

## References

{2015_SohlDickstein} 2015_Sohl-Dickstein_Deep_Unsupervised_Learning_using_Nonequilibrium_Thermodynamics
{2020_Ho_DDPM} 2020_Ho_Denoising_Diffusion_Probabilistic_Models
{2020_Song_NCSN} 2020_Song_Generative_Modeling_by_Estimating_Gradients_of_the_Data_Distribution
{2021_Dhariwal} 2021_Dhariwal_Diffusion_Models_Beat_GANs_on_Image_Synthesis
{2021_Song_Blog} 2021_Song_Generative_Modeling_by_Estimating_Gradients_of_the_Data_Distribution
{2021_Song_SDE} 2021_Song_Score-Based_Generative_Modeling_Through_Stochastic_Differential_Equations
{2022_Dieleman_DMAE} 2022_Dieleman_Diffusion_Models_are_Autoencoders
{2022_Ho_CFG} 2022_Ho_Classifier-Free_Diffusion_Guidance
{2022_Luo_Understanding} 2022_Luo_Understanding_Diffusion_Models_A_Unified_Perspective
{2023_Dieleman_Geometry} 2023_Dieleman_The_Geometry_of_Diffusion_Guidance
{2023_Dieleman_Perspectives} 2023_Dieleman_Perspectives_on_Diffusion
{2023_Kingma_VDM} 2023_Kingma_Variational_Diffusion_Models
{2023_Stanczuk} 2023_Stanczuk_Your_Diffusion_Model_Secretly_Knows_the_Dimension_of_the_Data_Manifold
{2024_Aithal} 2024_Aithal_Understanding_Hallucinations_in_Diffusion_Models_through_Mode_Interpolation
{2024_Kadkhodaie} 2024_Kadkhodaie_Generalization_in_Diffusion_Models_Arises_from_Geometry-Adaptive_Harmonic_Representations
