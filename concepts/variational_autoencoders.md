---
title: "Variational Autoencoders"
tags: ["variational-autoencoders", "generative-models", "latent-variable-models", "variational-inference", "representation-learning"]
last_updated: "2026/08/22"
---

## Summary

A variational autoencoder (VAE) is a deep latent-variable generative model trained by maximizing the evidence lower bound (ELBO): an amortized inference network (encoder) approximates the intractable posterior $q_\phi(z|x)$, a decoder models the likelihood $p_\theta(x|z)$, and the reparameterization trick makes the whole objective end-to-end differentiable. The ELBO balances a reconstruction term against a KL term pulling the posterior toward a prior. VAEs are foundational to deep generative modeling and to probabilistic representation learning, spawning disentanglement methods (β-VAE), richer posteriors (normalizing flows), and — reinterpreted as a hierarchy of latents — variational diffusion models. They also serve as a probabilistic lens on self-supervised learning and as a testbed for attacks on generative models.

## Related Concepts
[[generative_models]]
[[diffusion_models]]
[[latent_diffusion_models]]
[[score_matching]]
[[generative_adversarial_networks]]
[[self_supervised_learning]]
[[variational_inference]]
[[normalizing_flows]]
[[disentanglement]]
[[mutual_information_maximization]]
[[feature_suppression]]
[[aes_for_generative_models]]
[[generated_image_evaluation_metrics]]
[[masked_image_modeling]]
[[diffusion_model_semantic_spaces]]

## Open Questions

VAEs expose a persistent likelihood-vs-sample-quality tension: they optimize a principled likelihood bound yet produce blurry samples, and pushing likelihood further (as variational diffusion models do) still trails other diffusion models on perceptual FID {2023_Kingma_VDM}. Disentanglement is not cleanly resolved — β-VAE achieves it empirically but only when generative factors are genuinely independent, and increasing β trades reconstruction for disentanglement with no principled setting {2017_Higgins}. The Gaussian posterior systematically underestimates variance, motivating flows, but richer posteriors add cost and instability {2016_Rezende_Flows}. Whether the deep unification claims — SSL as variational inference {2023_Nakamura}, diffusion as an infinite-depth VAE {2023_Kingma_VDM} — are predictive or merely re-descriptive is unsettled. Robustness is under-explored: latent-space attacks succeed with distortion scaling linearly {2016_Tabacof}.

## Synthesis

### The VAE framework

The VAE {2022_Kingma_VAE} targets latent-variable models where the marginal $p_\theta(x)$ and posterior $p_\theta(z|x)$ are intractable. It introduces an amortized inference network $q_\phi(z|x)$ and maximizes the ELBO, writable as $-\mathrm{KL}(q_\phi(z|x) \| p_\theta(z)) + \mathbb{E}_q[ \log p_\theta(x|z) ]$ — a KL term regularizing the approximate posterior toward the prior plus an expected reconstruction term. The key enabling trick is **reparameterization**: sampling $z = \mu + \sigma \cdot \epsilon$ with $\epsilon \sim \mathcal{N}(0,I)$ moves the stochasticity off the parameters so gradients flow through the sampler, turning Monte-Carlo ELBO estimates into a differentiable objective trainable by SGD {2022_Kingma_VAE}. In the standard instantiation both $q_\phi(z|x)$ and the prior $p_\theta(z) = \mathcal{N}(0,I)$ are Gaussian, making the KL term analytic. Kingma & Welling's related-work note that regularized autoencoders correspond to an InfoMax bound on $I(x;z)$ links the VAE to [[mutual_information_maximization]].

### Lineage: from autoencoders to variational inference

VAEs sit at the confluence of two lines. The **autoencoder** line runs through denoising autoencoders {2008_Vincent}, which corrupt inputs and reconstruct the clean signal; Vincent et al. show a DAE maximizes a variational bound on a generative model and give it a manifold-learning reading (map corrupted points back to the data manifold) — an early, explicit tie between reconstruction, robustness, and generative modeling that prefigures the VAE. The **variational inference** line supplies the ELBO and amortization. The two co-foundational 2014 papers are Kingma & Welling's VAE {2022_Kingma_VAE} and Rezende, Mohamed & Wierstra's deep latent Gaussian models / stochastic backpropagation; the latter's own summary is unread here, and it is characterized via the DLGM framing recounted in the normalizing-flows paper {2016_Rezende_Flows} rather than consulted directly.

### Disentanglement: β-VAE

β-VAE {2017_Higgins} multiplies the KL term by $\beta > 1$, tightening the latent bottleneck so the model must encode information efficiently; when the data's underlying generative factors are independent, efficiency pushes the posterior toward axis-aligned, statistically independent latents — i.e. *disentangled* representations where single latent units track single factors (position, scale, rotation). The cost is a **reconstruction-disentanglement tradeoff**: higher β yields cleaner disentanglement but blurrier reconstructions, and the authors argue disentanglement, not pixel-perfect reconstruction, is the right goal {2017_Higgins}. They also introduced an early supervised disentanglement metric. A limitation they note — if the latent dimensionality is too small to hold all factors, some remain entangled — foreshadows the general difficulty of unsupervised disentanglement. This anchors the roadmap node [[disentanglement]].

### Richer posteriors: normalizing flows

The Gaussian $q_\phi(z|x)$ is the VAE's main expressiveness bottleneck — it cannot match a complex true posterior, underestimates variance, and biases estimates {2016_Rezende_Flows}. Normalizing flows fix this by transforming a simple base density through a sequence of invertible maps, tracking the density via the change-of-variables Jacobian; planar and radial flows give cheap Jacobian determinants and, stacked deep enough (large $K$), approximate complex posteriors, improving DLGM likelihoods on binarized MNIST {2016_Rezende_Flows}. This makes the approximate posterior itself a learned, arbitrarily-flexible distribution rather than a fixed family, and defines the roadmap node [[normalizing_flows]].

### VAEs as a lens on self-supervised learning

The variational-inference view reaches beyond generation. Nakamura et al. reframe SimSiam, SimCLR, and DINO as variational inference: their objectives decompose into an alignment term (a posterior expectation) plus uniformity and KL terms, with stop-gradient and EMA reinterpreted as expectation-maximization {2023_Nakamura}. Building on this, VI-SimSiam adds a second predictor head that outputs a von Mises-Fisher *concentration*, giving SimSiam a calibrated representation-uncertainty signal that correlates with linear-probe accuracy and (in DINO) with output entropy {2023_Nakamura}. This connects VAEs to [[self_supervised_learning]] and [[contrastive_ssl]] — the probabilistic-latent perspective supplies the uncertainty that deterministic joint-embedding methods lack.

### From VAEs to diffusion models

Variational diffusion models make the VAE-diffusion link explicit: a diffusion model is a deep hierarchy of Gaussian latents $z_t$ at increasing noise, trained on an ELBO of prior + reconstruction + diffusion-loss terms, and jointly learning the noise schedule "turns the model into a type of VAE" {2023_Kingma_VDM}. A key result is that, in continuous time, the diffusion loss depends only on the signal-to-noise endpoints and is otherwise invariant to the schedule's shape, and variance-preserving and variance-exploding formulations coincide {2023_Kingma_VDM}. VDM attains state-of-the-art likelihood (bits/dim) but worse FID than DDPM — a concrete instance of the likelihood-vs-perceptual-quality tension. This positions VAEs as one end of a continuum ending in [[diffusion_models]].

### Applications and failure modes

Beyond generation, the VAE's structured latent is useful for control. Chroma-VAE {2022_Yang_ChromaVAE} attacks *shortcut learning* by partitioning the latent: a small sub-latent trained with a classifier head preferentially captures the easy shortcut, while the complementary sub-latent — used to retrain the final classifier — carries the shortcut-free signal, improving group robustness on ColoredMNIST and CelebA. Notably it works best at $\beta = 1$ (contrary to β-VAE), which the authors attribute to joint training with supervision. This links VAEs to [[feature_suppression]]. Characteristic VAE failure modes — blurry samples and, under strong KL pressure, latents being ignored — surface across these works {2017_Higgins}{2023_Kingma_VDM}.

### Adversarial vulnerability

VAEs are themselves attackable. Tabacof et al. craft adversarial inputs that make a VAE reconstruct an attacker-chosen target by optimizing a latent-space objective (minimizing KL/L2 between the adversarial and target encodings under a distortion penalty), finding that attack success grows *linearly* with input distortion — unlike classifiers, whose softmax makes success grow sharply — and that VAEs are somewhat more robust than plain autoencoders {2016_Tabacof}. This is the generative-model face of the adversarial literature; further attacks on generative models (including VAE-GAN hybrids) are collected under [[aes_for_generative_models]].

### Coverage and terminology notes

This node draws on the VAE core and several variants; some clearly relevant works were not consulted because their summaries are unread — the co-foundational DLGM paper (Rezende et al. 2014), the Introduction-to-VAEs monograph (Kingma & Welling 2019), hyperspherical VAEs (Davidson et al.), and multimodal VAEs (MMVAE, Shi et al.; the multimodal ELBO, Sutter et al.) — the last two would extend this node toward [[multimodal_ssl]]. Terminology varies: the ELBO is equivalently the "variational lower bound" or "negative free energy"; the encoder/decoder are the "inference/recognition network" and "generative network."

## References

{2008_Vincent} 2008_Vincent_Extracting_and_Composing_Robust_Features_with_Denoising_AutoEncoders
{2016_Rezende_Flows} 2016_Rezende_Variational_Inference_with_Normalizing_Flows
{2016_Tabacof} 2016_Tabacof_Adversarial_Images_for_Variational_Autoencoders
{2017_Higgins} 2017_Higgins_Beta-VAE_Learning_Basic_Visual_Concepts_with_a_Constrained_Variational_Framework
{2022_Kingma_VAE} 2022_Kingma_Auto-Encoding_Variational_Bayes
{2022_Yang_ChromaVAE} 2022_Yang_Chroma-VAE_Mitigating_Shortcut_Learning_with_Generative_Classifiers
{2023_Kingma_VDM} 2023_Kingma_Variational_Diffusion_Models
{2023_Nakamura} 2023_Nakamura_Representation_Uncertainty_in_Self-Supervised_Learning_as_Variational_Inference
