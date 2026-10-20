---
title: "Generated Image Detection"
tags: ["generated-image-detection", "image-forensics", "deepfake-detection", "diffusion-models", "generative-adversarial-networks"]
last_updated: "2026/09/10"
---

## Summary

Generated-image detection asks whether an image was produced by a generative model — GAN, diffusion, or autoregressive — rather than captured by a camera. It is the forensic counterpart to generative modelling, driven by misinformation, fraud, and provenance concerns. The field is organized by two demands that pull against each other: *generalization* to generators unseen at training time, and *robustness* to the compression and resizing images suffer when shared online. Its central methodological hazard is that detectors easily learn *dataset bias* — JPEG format, resolution, or semantic content — instead of genuine generation artifacts, inflating benchmark scores while failing in the wild. Methods span frequency analysis, reconstruction error, frozen foundation-model features, bias-aware data curation, and explainable multimodal LLMs.

## Related Concepts
[[generative_adversarial_networks]]
[[diffusion_models]]
[[latent_diffusion_models]]
[[diffusion_samplers]]
[[diffusion_geometry]]
[[score_matching]]
[[variational_autoencoders]]
[[multimodal_ssl]]
[[self_supervised_learning]]
[[generated_image_evaluation_metrics]]
[[aes_for_generative_models]]
[[defenses_against_aes_for_generative_models]]
[[membership_inference_attacks]]
[[image_watermarking]]
[[adversarial_examples]]
[[image_augmentation_methods]]

## Open Questions

No detector generalizes to *all* generators: a model trained on one family transfers poorly to another, and the field advances reactively as each new generator (GAN → diffusion → autoregressive) erodes the previous artifact cue {2022_Corvi}{2024_Ricker_Deepfakes}. Whether reported accuracy reflects real forensic signal or dataset shortcuts is contested — a fixed detector can flip its verdict on a fixed generator depending only on which real-image pool it is tested against {2025_Guillaro}, and pixel-aligned "fakes" still leave a JPEG/high-frequency tell {2025_Chen_DDA}. It is unresolved whether reconstruction- and likelihood-based signals are *causal* or merely correlated with a strong backbone and heavy augmentation {2025_Chen_DDA}. Robustness to online re-sharing fights the fragility of high-frequency cues, and the adversarial robustness of detectors — whether a motivated forger can evade them ([[aes_for_generative_models]]) — is only beginning to be probed, by black-box red-teaming of deployed detectors (PolyJuice) and on-manifold adversarial training against their latent-prior bias (OMAT). Passive detection versus proactive watermarking remains an open strategic split, made sharper by evidence that even robust watermarks are removable by regeneration {2024_Liu_CtrlRegen}. Attribution beyond a binary real/fake verdict — naming *which* generator produced an image, in an open world of unseen models (open-world attribution, Zheng et al.) — is a further open frontier, extending the closed-set attribution of DE-FAKE {2023_Sha}.

## Synthesis

### The problem and its two axes

A detector is a binary classifier, real vs. generated, but the useful metric is never in-distribution accuracy — it is cross-generator *generalization* and post-processing *robustness*, because deployed images come from unknown models and arrive JPEG-compressed and resized {2022_Corvi}{2025_Guillaro}. Overshadowing both is a bias problem: forensic cues are subtle, so if real and fake training images differ in *any* incidental way — lossy-JPEG reals vs. lossless-PNG fakes, resized reals vs. native fakes, or simply different subjects — a classifier learns that shortcut instead {2025_Guillaro}. The starkest demonstration is that for a fixed detector and generator, the real-vs-fake scores can invert depending only on whether the reals are drawn from one dataset or another {2025_Guillaro}. This reframes the whole field: much reported progress is dataset polarization, and harder benchmarks with content-matched reals (reverse-image-search retrieval, in-the-wild scrapes) reorder the leaderboard {2024_Cazenavette}. Evaluations run largely on shared benchmarks — most prominently the mission-scale GenImage (Zhu et al., named here from its scope, not consulted) — whose real/fake construction shapes the reported numbers, exactly the dependence this bias analysis exposes {2025_Guillaro}.

### Spatial classifiers and the generalization failure

The baseline is a CNN trained end-to-end on one generator's images (classically ProGAN). Such detectors catch their training family but treat "real" as a *sink* that absorbs anything lacking that family's artifacts, so diffusion images are misread as real {2024_Ojha}. Several architectural remedies raise transferability: constraining the receptive field so the classifier keys on *local* texture artifacts that are more consistent across architectures than global ones {2020_Chai}; ensembling deliberately "orthogonal" CNNs trained on different content/models/post-processing, with an asymmetric prior trusting the real class more than the synthetic {2022_Mandelli}; and — repeatedly — fine-tuning a *pretrained* ImageNet backbone (EfficientNet, Xception) rather than training a forensic net from scratch, which improves cross-architecture generalization when paired with aggressive JPEG/blur/noise augmentation {2021_Gragnaniello}{2022_Corvi}. Face-swap "deepfake" detection is a related but distinct branch that keys on blending-boundary and identity-inconsistency artifacts, often via synthetic self-blended training images {2022_Shiohara}{2020_Qian}, and face-specific benchmarks show the same frequency-generalization patterns {2024_Lu}.

### Frequency-domain artifacts

A long line reads the tell in the spectrum. GAN upsampling leaves periodic grid patterns in the DCT/Fourier domain that persist across architectures and resolutions; classifiers trained on frequency representations are more data-efficient and more robust to blur/crop than pixel-space ones {2020_Frank}, and retraining such spectral classifiers on diffusion data generalizes back to GANs {2024_Ricker_Deepfakes}. Radial power-spectrum profiles give a compact universal signature — flat for synthetic, fluctuating for real {2024_Alam} — and color-distribution statistics separate camera-corrected reals from oversaturated fakes {2025_Jia}. Two cautions temper this. Diffusion models leave *fewer* spectral artifacts than GANs, so frequency cues weaken as generators improve {2024_Ricker_Deepfakes}{2022_Corvi} (dedicated diffusion-era frequency detectors such as Synthbuster still exploit what remains). And the high-frequency cue is partly a *bias*: because reals are usually JPEG-compressed (attenuating high frequencies) while fakes are not, a "high-frequency richness ⇒ synthetic" shortcut survives even pixel-level content alignment and must be neutralized by matched compression {2025_Chen_DDA}.

### The reconstruction-error paradigm

The dominant recent approach for diffusion detection exploits the model's own prior: a diffusion (or its autoencoder) reconstructs a *generated* image more faithfully than a real one, because real images sit slightly outside the learned manifold. DIRE inverts an image to noise via DDIM and back, and classifies the reconstruction error $|x - \mathrm{Recon}(x)|$ {2023_Wang_DIRE} (see [[diffusion_samplers]] for the inversion). Variants proliferate along clear axes: use *intermediate* stepwise error rather than full reconstruction {2023_Ma_SeDID}; use only the LDM *autoencoder* for a training-free detector {2024_Ricker_Aeroblade}; compute latent error in a *single* denoising step for efficiency {2025_Luo_LaRE2}; use the *noise estimates* themselves rather than pixel error {2025_Zhang_DNF}; condition the error on *frequency* bands, since diffusion specifically mis-reconstructs mid-band content of real images {2025_Chu_FIRE}; or track error across *multiple* noise strengths as a trajectory {2026_Ameen}. FakeInversion grounds the paradigm in likelihood — DDIM inversion approximates the PF-ODE change-of-variables, so the noise map and imperfect reconstruction estimate an image's log-likelihood, which an overgeneralizing generator assigns differently to real and fake — and it generalizes to unseen text-to-image models {2024_Cazenavette}. A zero-shot cousin needs no fake training data at all: generated images *converge* to their target faster along the denoising trajectory (a coarse-to-fine, low-to-high-frequency emergence), so per-step similarity to the original is itself the signal {2026_Liang_DenoisingTraj}. This whole family is a direct application of diffusion theory — inversion, PF-ODE likelihood, and the frequency/manifold structure of [[diffusion_geometry]] and [[score_matching]].

### Frozen foundation-model features

Rather than train a discriminative feature space that overfits, one can classify in the frozen feature space of a general model. UnivFD uses CLIP-ViT features with a nearest-neighbor or linear probe: because CLIP was never trained to separate real from fake, its features are not asymmetrically biased, giving strong generalization across GANs, diffusion, and autoregressive generators {2024_Ojha}. DE-FAKE similarly uses CLIP embeddings — optionally fused with the prompt's text embedding — for both detection and *source attribution* (which model generated it) {2023_Sha}, connecting to the vision-language representations of [[multimodal_ssl]]. Frozen self-supervised backbones (DINOv2/DINOv3) recur as the substrate for the data-curation methods below.

### It's the data, not the model

A "bias-free" school argues the training set matters more than the architecture. B-Free generates fakes *from* each real image by self-conditioned diffusion reconstruction, so real and fake share exact content, format, and resolution and only generation artifacts remain; with inpainting-based content augmentation this yields state-of-the-art accuracy and, crucially, *calibration* across 27 generators {2025_Guillaro}. DDA adds the missing *frequency* alignment — matched JPEG compression plus mixup — after showing that pixel-faithful reconstruction still leaves a high-frequency gap {2025_Chen_DDA}. DRCT reconstructs real images into hard negatives that carry diffusion artifacts and trains contrastively, wrapping and improving existing detectors {2024_Chen_DRCT}. A related trick contaminates real images through only the *final* component of a generator (VAE decoder, VQ de-tokenizer), which many architectures share, to generalize across paradigms from a few hundred samples {2026_Liu_FinalComp}. At the extreme, *real-only* methods characterize the natural-image distribution alone: a self-supervised feature space trained on camera EXIF metadata flags off-distribution images {2026_Zhong}, and RealStats aggregates existing detectors as statistical tests, computing p-values against the real population with no fake data {2026_Zisman}.

### Explainability, deployment, and the arms race

The newest thread makes detection *explainable* by fine-tuning multimodal LLMs to emit a verdict plus a forensic rationale — via reinforcement learning {2025_Huang_ThinkFake}, supervised instruction tuning on annotated forgery explanations {2025_Tan_ForenX}, staged SFT-plus-DPO with human-preference alignment {2025_Zhou_Holmes}, or spatially grounded bounding-box rationales {2026_Ji_FakeXplain}. Practical deployment adds its own axis: quantizing detectors for edge devices while preserving the subtle features they rely on {2026_Anon_DeFakeQ}. Underlying all of it is a generator–detector arms race that mirrors the attacker–defender cycle of [[adversarial_examples]]: as generators improve, explicit artifacts (GAN spectral grids) fade and detectors migrate toward reconstruction, likelihood, and foundation-feature signals; detectors are themselves attackable by adversarial perturbation and evadable by purification ([[aes_for_generative_models]], [[defenses_against_aes_for_generative_models]]); and proactive watermarking competes with passive detection as an alternative provenance strategy. Detection also shares tooling with generative *evaluation* — Inception/CLIP features, LPIPS/SSIM reconstruction distances, and frequency statistics all recur in [[generated_image_evaluation_metrics]].

### Provenance and watermarking

The proactive alternative to passive detection is to embed a provenance signal at generation time — a watermark — added either *post-hoc* to a finished image or *in-generation* during sampling {2024_Liu_CtrlRegen}. Watermarking sidesteps the generalization problem, because the signal is designed and known rather than learned from artifacts, but it inherits an arms race of its own: watermarks are *removable*, and removal reduces to the same regeneration/purification machinery that strips protective perturbations ([[defenses_against_aes_for_generative_models]]). Pushing a watermarked image through a diffusion model's add-noise-then-resample cycle erases low-perturbation watermarks, while robust high-perturbation schemes (e.g. StegaStamp, Tree-Ring) survive it; CtrlRegen defeats even the robust schemes by regenerating a clean image from *pure* noise under external semantic ([[multimodal_ssl]] DINOv2) and spatial (ControlNet, [[diffusion_based_image_editing]]) guidance rather than from the watermarked image, trading the mark away while preserving content and quality {2024_Liu_CtrlRegen}. Parallel work argues invisible watermarks are provably removable by regeneration (Zhao et al.) and studies transferable watermark attacks with adversarial defenses (Gluch et al.). The taxonomy of watermark attacks — editing (crop/compress), regeneration, and adversarial evasion of the watermark *detector* — mirrors the attack/defense structure of [[aes_for_generative_models]] and [[adversarial_examples]], so provenance-by-watermark faces the same durability problem as image protection: an adversary with generative-model access can regenerate the signal away, and a single clean regeneration defeats it. A fuller treatment of embedding schemes and their robustness is reserved for [[image_watermarking]].

## References

{2020_Chai} 2020_Chai_What_Makes_Fake_Images_Detectable_Understanding_Properties_that_Generalize
{2020_Frank} 2020_Frank_Leveraging_Frequency_Analysis_for_Deep_Fake_Image_Recognition
{2020_Qian} 2020_Qian_Thinking_in_Frequency_Face_Forgery_Detection_by_Mining_Frequency-aware_Clues
{2021_Gragnaniello} 2021_Gragnaniello_Are_GAN_Generated_Images_Easy_to_Detect_A_Critical_Analysis_of_the_State-of-the-Art
{2022_Corvi} 2022_Corvi_On_the_Detection_of_Synthetic_Images_Generated_by_Diffusion_Models
{2022_Mandelli} 2022_Mandelli_Detecting_GAN-Generated_Images_by_Orthogonal_Training_of_Multiple_CNNs
{2022_Shiohara} 2022_Shiohara_Detecting_Deepfakes_with_Self-Blended_Images
{2023_Ma_SeDID} 2023_Ma_Exposing_the_Fake_Effective_Diffusion-Generated_Images_Detection
{2023_Sha} 2023_Sha_DE-FAKE_Detection_and_Attribution_of_Fake_Images_Generated_by_Text-to-Image_Generation_Models
{2023_Wang_DIRE} 2023_Wang_DIRE_for_Diffusion-Generated_Image_Detection
{2024_Alam} 2024_Alam_UGAD_Universal_Generative_AI_Detector_utilizing_Frequency_Fingerprints
{2024_Cazenavette} 2024_Cazenavette_FakeInversion_Learning_to_Detect_Images_from_Unseen_Text-to-Image_Models_by_Inverting_Stable_Diffusion
{2024_Chen_DRCT} 2024_Chen_DRCT_Diffusion_Reconstruction_Contrastive_Training_towards_Universal_Detection_of_Diffusion_Generated_Images
{2024_Liu_CtrlRegen} 2024_Liu_Image_Watermarks_are_Removable_Using_Controllable_Regeneration_from_Clean_Noise
{2024_Lu} 2024_Lu_Towards_the_Detection_of_AI-Synthesized_Human_Face_Images
{2024_Ojha} 2024_Ojha_Towards_Universal_Fake_Image_Detectors_that_Generalize_Across_Generative_Models
{2024_Ricker_Aeroblade} 2024_Ricker_Aeroblade_Training-Free_Detection_of_Latent_Diffusion_Images_Using_Autoencoder_Reconstruction_Error
{2024_Ricker_Deepfakes} 2024_Ricker_Towards_the_Detection_of_Diffusion_Model_Deepfakes
{2025_Chen_DDA} 2025_Chen_Dual_Data_Alignment_Makes_AI-Generated_Image_Detector_Easier_Generalizable
{2025_Chu_FIRE} 2025_Chu_FIRE_Robust_Detection_of_Diffusion-Generated_Images_via_Frequency-Guided_Reconstruction_Error
{2025_Guillaro} 2025_Guillaro_A_Bias-Free_Training_Paradigm_for_More_General_AI-Generated_Image_Detection
{2025_Huang_ThinkFake} 2025_Huang_ThinkFake_Reasoning_in_Multimodal_Large_Language_Models_for_AI-Generated_Image_Detection
{2025_Jia} 2025_Jia_Secret_Lies_in_Color_Enhancing_AI-Generated_Images_Detection_with_Color_Distribution_Analysis
{2025_Luo_LaRE2} 2025_Luo_LaRE2_Latent_Reconstruction_Error_Based_Method_for_Diffusion-Generated_Image_Detection
{2025_Tan_ForenX} 2025_Tan_ForenX_Towards_Explainable_AI-Generated_Image_Detection_with_Multimodal_Large_Language_Models
{2025_Zhang_DNF} 2025_Zhang_Diffusion_Noise_Feature_Accurate_and_Fast_Generated_Image_Detection
{2025_Zhou_Holmes} 2025_Zhou_AIGI-Holmes_Towards_Explainable_and_Generalizable_AI-Generated_Image_Detection_via_Multimodal_Large_Language_Models
{2026_Ameen} 2026_Ameen_Detecting_AI-Generated_Images_via_Diffusion_Snap-Back_Reconstruction_A_Forensic_Approach
{2026_Anon_DeFakeQ} 2026_Anonymous_DeFakeQ_Towards_Adaptive_Bidirectional_Quantization_for_Deepfake_Detection
{2026_Ji_FakeXplain} 2026_Ji_FakeXplain_AI-Generated_Image_Detection_via_Human-Aligned_Grounded_Reasoning
{2026_Liang_DenoisingTraj} 2026_Liang_Denoising_Trajectory_Biases_for_Zero-Shot_AI-Generated_Image_Detection
{2026_Liu_FinalComp} 2026_Liu_Exploiting_the_Final_Component_of_Generator_Architectures_for_AI-Generated_Image_Detection
{2026_Zhong} 2026_Zhong_Self-Supervised_AI-Generated_Image_Detection_A_Camera_Metadata_Perspective
{2026_Zisman} 2026_Zisman_RealStats_A_Rigorous_Real-Only_Statistical_Framework_for_Fake_Image_Detection
