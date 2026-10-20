---
title: "Generative Models"
tags: ["generative-models", "image-synthesis", "latent-variable-models", "autoregression", "energy-based-models"]
last_updated: "2026/09/09"
---

## Summary

Generative models learn a data distribution well enough to draw new samples from it, and divide into a few families distinguished by how they cope with the intractable likelihood: variational autoencoders (latent-variable, trained on a bound), generative adversarial networks (implicit, trained by a discriminator), autoregressive models (tractable factorized likelihood), normalizing flows (exact likelihood via invertible maps), energy-based models (unnormalized densities), and diffusion / score-based models (iterative denoising). They are compared along a three-way tension between sample quality, diversity, and sampling speed. This node is the top-level hub of the generative subgraph: it delegates each family to its own node and collects the cross-family unifications that increasingly show them to be facets of one another.

## Related Concepts
[[variational_autoencoders]]
[[generative_adversarial_networks]]
[[diffusion_models]]
[[flow_matching]]
[[autoregressive_models]]
[[normalizing_flows]]
[[energy_based_models]]
[[score_matching]]
[[latent_diffusion_models]]
[[diffusion_language_models]]
[[diffusion_model_semantic_spaces]]
[[self_supervised_learning]]
[[generated_image_evaluation_metrics]]
[[aes_for_generative_models]]

## Open Questions

No single paradigm dominates generation: diffusion leads perceptual synthesis while autoregression still leads language, a boundary that diffusion language models are actively contesting {2023_Dieleman_DLM}. A recurring tension is that likelihood and perceptual quality disagree — a model optimized for the best bits/dim produces worse FID than a perceptually-tuned one {2023_Kingma_VDM}. Whether the unification results (diffusion as an infinite-depth VAE, flow matching as diffusion) are genuinely predictive or merely re-descriptive is unsettled {2023_Kingma_VDM}{2024_Gao_FlowMatching}. Whether one network can be simultaneously the best generator and the best representation learner is open. And cross-family sample-quality comparison is itself contested, resting on metrics with known blind spots ([[generated_image_evaluation_metrics]]).

## Synthesis

### The generative modeling problem

A generative model approximates a data distribution $p_\mathrm{data}(x)$ from samples, with the practical goal of drawing new samples from — and sometimes evaluating likelihoods under — the fit {2015_SohlDickstein}. The central obstacle is that a flexible density written as $p_\theta(x) = e^{-f_\theta(x)} / Z_\theta$ carries an intractable normalizing constant $Z_\theta = \int e^{-f_\theta(x)}\,dx$, so exact maximum likelihood is unavailable {2010_Gutmann}. The families are best read as different bargains with this intractability: **bound** it (VAEs), **avoid the density entirely** and match samples adversarially (GANs), **restrict the architecture** so the likelihood factorizes or is exactly invertible (autoregressive models, normalizing flows), **learn the unnormalized energy or its gradient** (energy-based and score-based models), or **convert one hard sampling step into many easy ones** (diffusion). Each bargain fixes a family's characteristic strengths and failure modes.

### The families

- **Variational autoencoders** posit latent variables and maximize an evidence lower bound through an amortized encoder and the reparameterization trick, buying a tractable objective at the price of blurry samples ([[variational_autoencoders]]) {2022_Kingma_VAE}.
- **Generative adversarial networks** skip the density altogether: a generator is trained to fool a discriminator in a minimax game, giving sharp samples but unstable training and mode collapse ([[generative_adversarial_networks]]) {2020_Zhao_DiffAugment}. (The corpus lacks the foundational GAN methodology papers, a gap flagged in that node, so GAN claims here rest on GAN-adjacent works.)
- **Autoregressive models** factorize the likelihood by the chain rule and predict one token or pixel at a time, giving exact tractable likelihoods and the dominant approach to language, at the cost of strictly sequential sampling ([[autoregressive_models]], [[diffusion_language_models]]) {2023_Dieleman_DLM}.
- **Normalizing flows** build a complex density from a simple base through a chain of invertible maps, so the change-of-variables formula yields exact likelihoods; expressiveness is limited by the need for tractable Jacobian determinants ([[normalizing_flows]]) {2016_Rezende_Flows}.
- **Energy-based models** learn an unnormalized energy $f_\theta$ (density $\propto e^{-f_\theta}$) and sample by MCMC such as Langevin dynamics; they are maximally flexible, but the intractable $Z_\theta$ makes both training and sampling hard ([[energy_based_models]]) {2010_Gutmann}.
- **Diffusion / score-based models** learn to reverse a gradual noising process, decomposing a hard one-shot generative problem into many easy denoising steps; they are now dominant for perceptual data — images, audio, video ([[diffusion_models]], [[score_matching]]) {2015_SohlDickstein}{2020_Ho_DDPM}.

### The quality–diversity–speed tension

Across families, generation is organized by a three-way tension — often called the generative "trilemma" — among **sample quality** (fidelity, precision), **diversity** (mode coverage, recall), and **sampling speed** {2021_Dhariwal}. GANs historically won sample-quality metrics such as FID partly by sacrificing diversity — high precision, low recall — and sample in a single forward pass, but train unstably; VAEs and flows are stable and cover modes but sample softly; diffusion covers modes and, with guidance, surpasses GANs on both fidelity and diversity, but samples slowly {2021_Dhariwal}. Guidance is the knob that made this reframing possible, importing the precision–recall trade directly into diffusion ([[diffusion_guidance]]) {2021_Dhariwal}. The persistent liability of each family names a research program of its own: GAN stability, VAE sharpness, and diffusion sampling cost ([[diffusion_samplers]]).

### Convergence and unification

The most striking recent development is that the families, long taught as rivals, are increasingly shown to be facets of one another. A diffusion model is an **infinite-depth / hierarchical VAE** with fixed Gaussian encoders, fit by an ELBO — indeed jointly learning its noise schedule "turns the model into a type of VAE" {2023_Kingma_VDM}{2022_Luo_Understanding}. **Flow matching** (a continuous normalizing flow trained by regression) coincides with diffusion under a particular noise schedule and loss weighting, so a score network and a velocity field interconvert without retraining ([[flow_matching]]) {2024_Gao_FlowMatching}. The probability-flow ODE view in turn makes a diffusion model a **continuous normalizing flow** with a deterministic, invertible data–noise map and exact likelihoods, tying the diffusion and flow families back to [[normalizing_flows]] {2022_Luo_Understanding}. **Score matching** learns the gradient of the log-density and thereby sidesteps the very normalizing constant that defines energy-based models, so score-based and energy-based models are two readings of the unnormalized-density problem ([[score_matching]], [[energy_based_models]]) {2010_Gutmann}. And **latent diffusion** binds three families into one pipeline — a VAE-style encoder, a GAN-sharpened decoder, and a diffusion (or autoregressive) prior over the latents ([[latent_diffusion_models]]) {2025_Dieleman_Latents}. The connective idea is that **iterative refinement**, rather than any single parameterization, is the deeper principle behind strong generative models {2023_Dieleman_Perspectives}.

### Generative models as representation learners

Fitting $p_\mathrm{data}$ well tends to yield useful *representations*, not only samples — the theme that makes the generative subgraph a sibling of, rather than a stranger to, [[self_supervised_learning]]. VAEs give structured probabilistic latents and a lens that even reframes self-supervised objectives as variational inference ([[variational_autoencoders]]); diffusion models grow semantic latent spaces and act as competitive zero-shot classifiers despite training only to denoise ([[diffusion_model_semantic_spaces]]); and generative pretraining-then-finetuning is a recurring strong baseline. The unresolved counterpoint is that generative representations still trail the best discriminative ones (DINO, SimCLR) on linear probing, so it remains open whether generation and recognition genuinely want the same features.

### Historical arc and positioning

The field's center of gravity has shifted repeatedly. VAEs and GANs defined deep generative modeling in the mid-2010s; diffusion displaced GANs for perceptual synthesis around 2021, when guided diffusion beat the strongest GANs on both fidelity and diversity {2021_Dhariwal}; and attention has since split by modality — diffusion and flow matching for continuous perceptual data, autoregression for discrete language, with diffusion language models now contesting that last stronghold ([[diffusion_language_models]]) {2023_Dieleman_DLM}. Two concerns are handled subgraph-wide rather than per family: sample-quality **evaluation** compares all families by how far their samples sit from the real-data manifold ([[generated_image_evaluation_metrics]]), and every family admits **adversarial inputs**, from the classic VAE/VAE-GAN latent attacks to protective perturbations against diffusion mimicry ([[aes_for_generative_models]]).

## References

{2010_Gutmann} 2010_Gutmann_Noise-contrastive_estimation_A_new_estimation_principle_for_unnormalized_statistical_models
{2015_SohlDickstein} 2015_Sohl-Dickstein_Deep_Unsupervised_Learning_using_Nonequilibrium_Thermodynamics
{2016_Rezende_Flows} 2016_Rezende_Variational_Inference_with_Normalizing_Flows
{2020_Ho_DDPM} 2020_Ho_Denoising_Diffusion_Probabilistic_Models
{2020_Zhao_DiffAugment} 2020_Zhao_Differentiable_Augmentation_for_Data-Efficient_GAN_Training
{2021_Dhariwal} 2021_Dhariwal_Diffusion_Models_Beat_GANs_on_Image_Synthesis
{2022_Kingma_VAE} 2022_Kingma_Auto-Encoding_Variational_Bayes
{2022_Luo_Understanding} 2022_Luo_Understanding_Diffusion_Models_A_Unified_Perspective
{2023_Dieleman_DLM} 2023_Dieleman_Diffusion_Language_Models
{2023_Dieleman_Perspectives} 2023_Dieleman_Perspectives_on_Diffusion
{2023_Kingma_VDM} 2023_Kingma_Variational_Diffusion_Models
{2024_Gao_FlowMatching} 2024_Gao_Diffusion_Meets_Flow_Matching_Two_Sides_of_the_Same_Coin
{2025_Dieleman_Latents} 2025_Dieleman_Generative_Modelling_in_Latent_Space
