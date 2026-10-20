---
title: "Diffusion Language Models"
tags: ["diffusion-models", "language-models", "discrete-diffusion", "masked-diffusion", "autoregression", "generative-models"]
last_updated: "2026/08/23"
---

## Summary

Diffusion language models (DLMs) apply the denoise-from-noise recipe to discrete text, challenging autoregression's monopoly on language generation. Two families dominate: **discrete corruption**, which masks or perturbs tokens directly (D3PM, LLaDA), and **continuous embedding**, which diffuses token vectors and rounds back (Diffusion-LM, Plaid). Masking-based corruption has proven strongest and ties DLMs directly to BERT-style masked language modelling. Autoregression remains a very tough baseline — DLMs cannot cache activations and are less compute-efficient — but they offer bidirectional context, natural infilling, controllable generation by guidance, and, at the current frontier, superior data efficiency at the cost of extra compute. This node is the discrete-data extension of the diffusion subgraph.

## Related Concepts
[[diffusion_models]]
[[noise_schedules]]
[[flow_matching]]
[[diffusion_guidance]]
[[variational_autoencoders]]
[[masked_image_modeling]]
[[energy_based_models]]
[[autoregressive_models]]
[[generative_models]]

## Open Questions

Whether diffusion can ever be more *efficient* than autoregression at scale is unresolved: the break-even point requires reaching high quality in far fewer steps than the sequence length, and DLMs forfeit the activation caching that makes AR sampling cheap {2023_Dieleman_DLM}. The perceptual/frequency argument that makes diffusion natural for images has no clear analog for discrete tokens {2023_Dieleman_DLM}. The discrete-versus-continuous choice is unsettled — continuous embedding preserves guidance and fast samplers but poses an ill-posed embedding-learning problem {2023_Dieleman_DLM}{2023_Gulrajani_Plaid}. Scaling laws are poorly characterized and contested: one likelihood study finds DLMs ~64x less compute-efficient than AR {2023_Gulrajani_Plaid}, while data-constrained studies find DLMs win with repeated data, though the two leading analyses disagree sharply on methodology and on whether the cause is masking-as-augmentation or bidirectionality {2025_Prabhudesai}{2025_Ni}.

## Synthesis

### Why language resists diffusion

Diffusion took over image generation partly because different noise levels correspond to different spatial frequencies, and the common uniform weighting aligns with human perception — whereas the autoregressive likelihood over-invests in perceptually trivial high-frequency detail {2023_Dieleman_DLM}. This rationale *does not transfer* to language: text concentrates meaning into discrete tokens, so notions of frequency and perceptual relevance largely evaporate, and diffusion loses its built-in advantage {2023_Dieleman_DLM}. Autoregression is also an exceptionally strong baseline for sequential text on two efficiency axes: it obtains gradient signal from every position in one pass (DLMs must sample a noise level per example), and it caches activations across decoding steps (DLMs cannot, because the noise level changes each step, forcing a full forward pass) {2023_Dieleman_DLM}. What keeps DLMs worth pursuing is architectural freedom (no causal mask), natural conditioning on non-prefix context, and guidance-based control without retraining — plus the sense that iterative refinement, not autoregression specifically, is the deeper principle behind strong generative models {2023_Dieleman_DLM}.

### Discrete corruption: D3PM and the masking family

The most direct approach corrupts tokens in place. **D3PM** defines the forward process by categorical transition matrices $Q_t$, unifying several corruptions: uniform (tokens flip to random tokens), **absorbing** (tokens flip to a `[MASK]`), discretized-Gaussian, and embedding-distance {2023_Austin_D3PM}. Absorbing (masking) corruption performs best on text, and the framework exposes deep equivalences: **BERT is a one-step D3PM**, an autoregressive model is a D3PM whose forward process deterministically masks tokens left-to-right, and a generative masked language model is D3PM with absorbing transitions under a reweighted objective {2023_Austin_D3PM}. Parameterizing the reverse process to predict clean tokens $x_0$ and adding an auxiliary cross-entropy term stabilizes training {2023_Austin_D3PM}. **LLaDA** scales masked diffusion to an 8B-parameter LLM competitive with LLaMA3-8B: a single mask predictor recovers all masked tokens in parallel, trained by a masked cross-entropy that upper-bounds the NLL {2025_Nie_LLaDA}. Because it lacks a left-to-right inductive bias, LLaDA handles **reversal reasoning** symmetrically where autoregressive models fail badly, and it admits flexible remasking (random, low-confidence-first, or semi-autoregressive block-wise) at sampling time {2025_Nie_LLaDA}. This masking lineage is the discrete-sequence counterpart of [[masked_image_modeling]].

### Continuous-embedding diffusion

The alternative embeds tokens into a continuous space, runs ordinary Gaussian diffusion there, and rounds back to tokens — preserving diffusion's machinery (classifier-free guidance, fast ODE samplers) at the cost of an embedding map. **Diffusion-LM** jointly learns token embeddings, predicts $x_0$ directly (with a clamping-to-nearest-token trick), and enables fine-grained **controllable generation** via classifier guidance on the continuous latents, excelling at global-structure tasks (length, syntax, parts-of-speech) that are awkward for left-to-right models {2022_Li_DiffusionLM}. **Plaid** makes the objective likelihood-based in the VDM style (a learned noise schedule, an ELBO), and identifies the embedding as the crux — prior losses give ill-posed, collapsing embeddings, and a categorical reparameterization plus jointly learned embeddings are the highest-impact components {2023_Gulrajani_Plaid}. Plaid-1B out-likelihoods GPT-2 and supports zero-shot control (span infilling, lexical constraints, negation) through guidance, but its scaling analysis quantifies the gap: DLMs improve with compute at the same rate as AR yet ~64x less efficiently, so compute-optimal DLMs should be ~4x smaller trained ~4x longer {2023_Gulrajani_Plaid}. The embedding-space route links DLMs to [[variational_autoencoders]]; **Language Rectified Flow** takes it further, running rectified flow ([[flow_matching]]) in a VAE latent to transport between source and target text distributions for controllable editing and infilling {2024_Zhang_LRF}.

### The data-efficiency frontier versus autoregression

The most active recent question is not quality but the **data-versus-compute tradeoff**. Prabhudesai et al. train 100 masked-diffusion and 100 autoregressive models under matched settings and find a clean split: AR is more *compute*-efficient (learns fast from a batch, then saturates or overfits) while masked diffusion is more *data*-efficient (needs far more compute and epochs but keeps improving on repeated data without overfitting), crossing over to win in data-constrained regimes; they attribute the gap to masking acting as **data augmentation** and fit data-constrained scaling laws with a critical-compute crossover {2025_Prabhudesai}. Ni et al. corroborate the "super data learner" phenomenon — DLMs keep improving on HellaSwag/MMLU past hundreds of epochs where AR plateaus — but argue the cause is **bidirectional modelling plus the much higher FLOPs** DLMs spend (repeatedly processing data across timesteps), not augmentation alone, and level pointed methodological critiques at the comparison (validation loss is a poor proxy since AR downstream accuracy keeps rising while its loss "overfits"; DLM loss is only an upper bound; best-checkpoint selection is unfair) {2025_Ni}. Both agree DLMs *can* eventually overfit given small enough data {2025_Ni}. The debate is unresolved and turns on subtle experimental choices.

### Distinctive capabilities and terminology

Across families the recurring advantages are **bidirectionality** and **controllability**: because generation is not locked to left-to-right order, DLMs naturally support infilling and non-prefix conditioning {2023_Dieleman_DLM}{2022_Li_DiffusionLM}, planning tasks with global constraints {2022_Li_DiffusionLM}, symmetric forward/reverse reasoning {2025_Nie_LLaDA}, and guidance-based constraint satisfaction without retraining {2022_Li_DiffusionLM}{2023_Gulrajani_Plaid} — the language instance of [[diffusion_guidance]]. Terminology is inconsistent and worth flagging: "diffusion language model" spans both the discrete/masked and continuous-embedding families, which are mechanistically quite different; "masked diffusion," "absorbing-state diffusion," and BERT-style "masked language modelling" name closely related but not identical objects (they differ in whether generation is iterative and in the loss weighting) {2023_Austin_D3PM}{2025_Prabhudesai}; and "discrete diffusion" is used both for token-space corruption and, loosely, for any DLM. The foundational works are D3PM (discrete) {2023_Austin_D3PM} and Diffusion-LM (continuous) {2022_Li_DiffusionLM}; the scaling frontier is LLaDA {2025_Nie_LLaDA} and the data-efficiency studies {2025_Prabhudesai}{2025_Ni}.

## References

{2022_Li_DiffusionLM} 2022_Li_Diffusion-LM_Improves_Controllable_Text_Generation
{2023_Austin_D3PM} 2023_Austin_Structured_Denoising_Diffusion_Models_in_Discrete_State-Spaces
{2023_Dieleman_DLM} 2023_Dieleman_Diffusion_Language_Models
{2023_Gulrajani_Plaid} 2023_Gulrajani_Likelihood-Based_Diffusion_Language_Models
{2024_Zhang_LRF} 2024_Zhang_Language_Rectified_Flow_Advancing_Diffusion_Language_Generation_with_Probabilistic_Flows
{2025_Ni} 2025_Ni_Diffusion_Language_Models_are_Super_Data_Learners
{2025_Nie_LLaDA} 2025_Nie_Large_Language_Diffusion_Models
{2025_Prabhudesai} 2025_Prabhudesai_Diffusion_Beats_Autoregressive_in_Data-Constrained_Settings
