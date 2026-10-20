---
title: "Latent Diffusion Models"
tags: ["latent-diffusion-models", "diffusion-models", "autoencoders", "image-synthesis", "tokenizers"]
last_updated: "2026/08/22"
---

## Summary

A latent diffusion model runs the diffusion process not on raw pixels but in the compact latent space of a pretrained autoencoder, decoding generated latents back to data at the end. This two-stage recipe — train an autoencoder to strip perceptually irrelevant detail, then train a diffusion model on its latents — slashes the compute of iterative sampling while keeping fidelity, and is the basis of Stable Diffusion and most modern text-to-image systems. It builds on the VQ-VAE / VQGAN lineage of learned image tokenizers and pairs naturally with transformer backbones (DiT) and rectified-flow objectives. This node is the compressed-space instantiation of [[diffusion_models]], sitting at the interface with [[variational_autoencoders]].

## Related Concepts
[[diffusion_models]]
[[variational_autoencoders]]
[[generative_adversarial_networks]]
[[diffusion_samplers]]
[[noise_schedules]]
[[vision_transformers]]
[[flow_matching]]
[[diffusion_guidance]]
[[diffusion_based_image_editing]]
[[diffusion_model_personalization]]
[[generated_image_detection]]
[[aes_for_generative_models]]
[[defenses_against_aes_for_generative_models]]
[[membership_inference_attacks]]

## Open Questions

The governing trade is rate-distortion-*modelability*: latents must compress yet stay structured enough to model, and there is no cheap proxy for modelability, so the capacity setting is chosen empirically {2025_Dieleman_Latents}. Latent-space *curation and shape* — which bits to keep and how to present them — is under-theorized, addressed by a zoo of regularizers (KL, feature-alignment, equivariance) with no unifying account {2025_Dieleman_Latents}. Whether single-stage pixel-space generation will eventually overtake the two-stage recipe as hardware improves is openly uncertain {2025_Dieleman_Latents}. And whether diffusion-based decoders can match adversarial decoders without reintroducing a latency penalty is unresolved {2025_Dieleman_Latents}.

## Synthesis

### The two-stage recipe

Latent generative modeling separates two jobs. **Stage one** trains an autoencoder with a mix of losses — a regression (MSE/MAE) term, a perceptual term (e.g. LPIPS), an adversarial term, and an optional bottleneck penalty — to map data to a compact latent and back {2025_Dieleman_Latents}. **Stage two** freezes the encoder and trains a generative model (diffusion or autoregressive) directly on the extracted latents; the decoder is used only at sampling time to return to data space {2025_Dieleman_Latents}{2022_Rombach_LDM}. The motivation is that perceptual signals are mostly imperceptible detail: DDPM's own rate-distortion analysis shows most bits go to fine detail humans barely see, so pixel-space diffusion wastes capacity there {2022_Rombach_LDM}. LDM assigns that "perceptual compression" to the autoencoder and lets diffusion focus on semantic structure {2022_Rombach_LDM}.

### LDM / Stable Diffusion

Rombach et al.'s LDM instantiates the recipe with a convolutional autoencoder that downsamples an image by a factor $f$ (with $f = 4, 8, 16$ best; too little is slow, too much stagnates), regularized either KL-style or with vector quantization; crucially the latent is itself a small spatial map, so the diffusion U-Net still operates on "images," just smaller {2022_Rombach_LDM}. Conditioning (text, layout) enters through **cross-attention** in the U-Net, and the frozen autoencoder is reused across many downstream diffusion trainings {2022_Rombach_LDM}. The result matches or beats pixel-space diffusion and GANs on fidelity and coverage at a fraction of the cost, and generalizes to super-resolution, inpainting, and semantic synthesis — the architecture that became Stable Diffusion {2022_Rombach_LDM}.

### Lineage: learned image tokenizers

Latent diffusion inherits its encoder from a line of **learned tokenizers**. **VQ-VAE** introduced discrete, spatially-structured latents via vector quantization — encode to a grid of vectors, snap each to the nearest codebook entry, train through the non-differentiable step with straight-through gradients and a commitment loss — which also avoids the posterior collapse of continuous VAEs {2018_Oord_VQVAE}. **VQGAN** added perceptual and adversarial losses to the VQ-VAE, achieving sharp reconstructions at 16x downsampling and enabling an autoregressive transformer over the code indices {2021_Esser_VQGAN}. LDM followed naturally by swapping that autoregressive transformer for a diffusion model on VQGAN-style latents {2025_Dieleman_Latents}. This ties latent diffusion to [[variational_autoencoders]] (the encoder is a KL- or VQ-regularized autoencoder) and to [[generative_adversarial_networks]] (the decoder's sharpness comes from an adversarial loss) {2025_Dieleman_Latents}.

### Backbones and objectives in latent space

The generative model on the latents has evolved. **DiT** replaces the U-Net with a vision transformer operating on patchified latents, injecting timestep and class conditioning through adaptive layer-norm (AdaLN-zero) and scaling predictably with compute {2023_Peebles_DiT} ([[vision_transformers]]). **Stable Diffusion 3** combines a rectified-flow objective with a multimodal DiT that keeps separate text and image streams joined only at attention, and widens the autoencoder latent to 16 channels {2024_Esser_SD3} — connecting latent diffusion to [[flow_matching]] and to resolution-dependent timestep shifting ([[noise_schedules]]). Fast few-step sampling in latent space is provided by Latent Consistency Models, which apply consistency distillation to Stable Diffusion {2023_Luo_LCM} ([[diffusion_samplers]]).

### Rate, distortion, and modelability

The design tension unique to *generative* (versus merely compressive) latents is that they must also be **modelable** — retain enough structure for a downstream model to exploit {2025_Dieleman_Latents}. Entropy coding is the cautionary opposite: it removes redundancy losslessly but destroys topology, so tiny input changes cause large latent changes. Good latents instead look like "advanced pixels" — spatially structured, low-resolution, recognizable by eye (Stable Diffusion latents visibly resemble their images) {2025_Dieleman_Latents}. Capacity is governed by the downsampling factor and channel count (summarized as a tensor-size-reduction factor); the binding constraint is that highly nonlinear encoders which pack much information into few numbers destroy the smooth structure that makes latents modelable, so this is undesirable even when it improves reconstruction {2025_Dieleman_Latents}. Choosing the capacity is empirical, as there is no cheap modelability proxy.

### Curating and shaping the latent space

Beyond how many bits, practitioners control *which* bits are kept and *how* they are presented {2025_Dieleman_Latents}. In practice the KL penalty is scaled down by orders of magnitude relative to a proper VAE ELBO, so its Gaussianizing effect is negligible and it mainly suppresses outliers — Dieleman argues the "V" in these "VAEs" is vestigial and they are really KL-regularized autoencoders {2025_Dieleman_Latents}. Reconstruction losses do double duty (decoder quality and latent shape): a regression loss alone gives blurry output because image power concentrates at low frequencies, so perceptual and adversarial losses recover texture, and the adversarial loss's mode-dropping is actually desirable for a decoder {2025_Dieleman_Latents}. A growing set of **modelability regularizers** shapes latents explicitly — supervising them to predict pretrained features like DINOv2, enforcing equivariance to spatial transforms, or co-training a lightweight generative prior — while the SDXL autoencoder is cited as poor curation for faithfully encoding imperceptible noise the generator cannot use {2025_Dieleman_Latents}.

### The pixel-space alternative and outlook

Not all high-fidelity diffusion is latent. **Imagen** generates in pixel space via a cascade (a 64x64 base model plus super-resolution stages), pairs it with a large frozen T5 text encoder, and uses dynamic thresholding to tolerate high classifier-free-guidance scales {2022_Saharia_Imagen}. Such resolution cascades have largely fallen out of favor relative to latents, apparently due to a suboptimal division of labor between stages {2025_Dieleman_Latents}. Emerging **diffusion decoders** iteratively refine the decode step for higher quality but reintroduce input-space latency (mitigated by distillation), and **1D / variable-length tokenizers** push back on the "tyranny of the grid" that forces uniform capacity across a scene {2025_Dieleman_Latents}. The reliance on an autoencoder also leaves a forensic fingerprint: reconstructing a suspect image through a latent-diffusion autoencoder underlies several detectors ([[generated_image_detection]]). That same encoder is the principal *attack surface* — protective adversarial perturbations target the latent encoding to prevent mimicry or editing ([[aes_for_generative_models]]), and because they exploit the encoder rather than the diffusion prior, pixel-domain diffusion purification can strip them ([[defenses_against_aes_for_generative_models]]). Whether single-stage generation eventually wins on cost is left open {2025_Dieleman_Latents}.

## References

{2018_Oord_VQVAE} 2018_Oord_Neural_Discrete_Representation_Learning
{2021_Esser_VQGAN} 2021_Esser_Taming_Transformers_for_High-Resolution_Image_Synthesis
{2022_Rombach_LDM} 2022_Rombach_High-Resolution_Image_Synthesis_with_Latent_Diffusion_Models
{2022_Saharia_Imagen} 2022_Saharia_Photorealistic_Text-to-Image_Diffusion_Models_with_Deep_Language_Understanding
{2023_Luo_LCM} 2023_Luo_Latent_Consistency_Models_Synthesizing_High-Resolution_Images_with_Few-Step_Inference
{2023_Peebles_DiT} 2023_Peebles_Scalable_Diffusion_Models_with_Transformers
{2024_Esser_SD3} 2024_Esser_Scaling_Rectified_Flow_Transformers_for_High-Resolution_Image_Synthesis
{2025_Dieleman_Latents} 2025_Dieleman_Generative_Modelling_in_Latent_Space
