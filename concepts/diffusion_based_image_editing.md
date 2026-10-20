---
title: "Diffusion-Based Image Editing"
tags: ["diffusion-models", "image-editing", "diffusion-inversion", "text-to-image", "controllable-generation", "generative-models"]
last_updated: "2026/08/31"
---

## Summary

Diffusion-based image editing modifies a *given* image — real or generated — so that a targeted change is applied while the rest is preserved. Its two recurring problems are getting an existing image into the model's latent (inversion) and steering regeneration to change only what is intended. Techniques span guide-image resynthesis (SDEdit), attention-map control (Prompt-to-Prompt), instruction-following (InstructPix2Pix), spatial conditioning (ControlNet), and, increasingly, editing in flow models. A single tension organizes the field: **fidelity to the source versus adherence to the edit**, which reappears as the inaccuracy of deterministic inversion under guidance. This node covers the edit *operations*, delegating the structure of the editable latent space to [[diffusion_model_semantic_spaces]] and learning of new subjects to [[diffusion_model_personalization]].

## Related Concepts
[[diffusion_models]]
[[diffusion_samplers]]
[[diffusion_guidance]]
[[diffusion_model_semantic_spaces]]
[[diffusion_model_personalization]]
[[latent_diffusion_models]]
[[flow_matching]]
[[diffusion_geometry]]
[[multimodal_ssl]]
[[generative_adversarial_networks]]
[[aes_for_generative_models]]

## Open Questions

Real-image editing hinges on inversion, yet deterministic inversion is provably imperfect: the forward and reverse discretizations use mismatched noise estimates, so error accumulates and worsens under classifier-free guidance {2022_Mokady_NullText}{2023_Hong_ExactInv}. The inverted "noise" is not Gaussian — it stays correlated with image content, especially in smooth regions, which caps editability {2025_Staniszewski}. Which network component to manipulate is contested: the influential cross-attention account {2022_Hertz_P2P} is challenged by evidence that self-attention carries the structure worth preserving and cross-attention edits often fail {2024_Liu_Attention}. There is no agreed metric or benchmark trade-off frontier between structure preservation and edit strength, though PIE-Bench is a step {2024_Ju_PnPInv}. Whether flow-model editing needs new machinery or inherits diffusion's remains unsettled {2025_Kulikov_FlowEdit}.

## Synthesis

### Two problems: inversion and controlled regeneration

Editing a real image with a pretrained generator requires two things: a starting latent that regenerates the source, and a way to perturb regeneration so only the intended attribute changes. Most methods share a **two-branch** structure — from one starting noise, a *source* branch reconstructs the original (under the source prompt) and a *target* branch generates the edit (under a modified prompt), with the source branch constraining the target so layout and identity survive {2024_Ju_PnPInv}. The starting latent is obtained by **inversion**, and the constraint is imposed by manipulating the network's attention or features. Editing a purely *generated* image is easier — its latent is already known — but the same control machinery applies. This node treats those operations; the *space* in which they act (the U-Net bottleneck "h-space" and its semantic directions) is developed under [[diffusion_model_semantic_spaces]], and the conditioning strength they all rely on under [[diffusion_guidance]].

### Inversion: mapping an image back to a latent

The workhorse is **DDIM inversion**: because the DDIM sampler is a discretized ODE, running it in reverse deterministically carries an image to a noise latent that should regenerate it ([[diffusion_samplers]]) {2022_Hertz_P2P}. It is only approximate. Hong et al. formalize why: naive DDIM inversion applies forward Euler to the inverse ODE and assumes the noise estimate is unchanged across a step, so it accumulates error; exact inversion instead solves each step implicitly (backward Euler with a fixed-point or gradient loop), and extends to higher-order DPM-solvers, but the fixed-point iteration fails to converge at high guidance because the network loses the required Lipschitz continuity {2023_Hong_ExactInv}. The guidance interaction is central: conditional gradients under CFG amplify inversion error, so image editing and high guidance are in tension {2022_Mokady_NullText}. Staniszewski et al. add a distributional diagnosis — DDIM-inverted latents are *not* Gaussian: they stay spatially correlated and retain coarse object structure, with the largest deviation in smooth, low-frequency regions, which reduces the diversity and editability of the encoding; replacing just the first few percent of inversion steps with ordinary stochastic noising removes the correlation and improves edit diversity {2025_Staniszewski}. Note that inversion here means image-to-*latent*; the word carries two other meanings in this subgraph (below).

### Reconciling reconstruction with editability

Because inversion under guidance drifts, a family of methods reconciles faithful reconstruction with editability without retraining the model. **Null-text inversion** keeps the source latent on the DDIM inversion path by optimizing, per timestep, the *null (unconditional) embedding* used in CFG — leaving the model weights and the real text embedding untouched, which preserves editability while restoring near-perfect reconstruction (PSNR rising from ~14 to ~26) {2022_Mokady_NullText}. **PnP Inversion** achieves a similar end without optimization: it disentangles the two branches and, at each step, adds the reconstruction gap (inversion latent minus source-branch prediction) back to the *source branch only*, keeping the guiding latent glued to the inversion trajectory while the target branch edits freely; three lines of code improve nearly every editing method {2024_Ju_PnPInv}. Both target the **distortion–editability trade-off** first articulated by Prompt-to-Prompt: exact reconstruction with high guidance yields high editability but shifts unintended details (distortion), and the two must be balanced {2022_Hertz_P2P}. This is the same axis as the reconstruction–editability trade-off in [[diffusion_model_personalization]], viewed from the editing side.

### The attention control surface

Text-to-image U-Nets expose their control surface through attention, and manipulating it is the dominant tuning-free editing mechanism. **Prompt-to-Prompt** observed that cross-attention maps bind text tokens to image regions ("pixels are attracted to the words that describe them") and are stable across timesteps; injecting the source image's cross-attention maps into generation under a modified prompt then preserves layout while changing content, supporting word-swap, phrase-addition (global/style edits), and attention **reweighting** to strengthen or weaken a word's influence {2022_Hertz_P2P}. This cross-attention-centric view was subsequently challenged. Liu et al. show cross-attention maps carry *object-attribution semantics*, so replacing them for editing tends to fail; it is the **self-attention** maps that encode geometry and spatial structure, and injecting source self-attention (empirically in the middle U-Net layers) preserves structure while allowing texture to change, outperforming cross-attention editing on every metric {2024_Liu_Attention}. The two accounts agree that structure and content are separable in attention but disagree on which map holds structure — a genuine terminological/empirical inconsistency across the editing literature. Related tuning-free schemes (e.g. mutual self-attention control) build on the same surface, as characterized by the papers here {2024_Ju_PnPInv}{2024_Liu_Attention}.

### Guide-image and instruction-based editing

An older, inversion-free route resynthesizes from a coarse guide. **SDEdit** ("img2img") adds an intermediate amount of noise to a guide image — a stroke painting, a composite, a low-quality photo — then denoises: enough noise erases artifacts and reaches the model's typical set, while the surviving low frequencies fix composition, so a single noise-level knob trades realism against faithfulness to the guide; a masked variant preserves fixed regions {2022_Meng_SDEdit}. A different paradigm edits by **instruction**: InstructPix2Pix distills GPT-3 (to turn a caption into an edit instruction and target caption) and Stable Diffusion with Prompt-to-Prompt (to synthesize before/after image pairs) into a supervised dataset, then trains a model that applies a written instruction ("make it winter") to an input image in a forward pass — using **dual classifier-free guidance** with separate scales for the input image and the text instruction to trade structure preservation against edit strength {2023_Brooks_IP2P}. Instruction editing shifts the cost from per-image inversion to one-time dataset construction and training, and generalizes to real images and human-written instructions despite an all-synthetic training set {2023_Brooks_IP2P}.

### Structural and spatial control

Text under-specifies layout, pose, and geometry. **ControlNet** adds explicit spatial conditioning by cloning the encoder of a frozen text-to-image U-Net into a trainable copy connected through **zero-initialized convolutions**, so training starts as an identity map and gradually learns to inject a control image — canny edges, depth, segmentation, human pose — without degrading the base model; multiple controls compose by summation {2023_Zhang_ControlNet}. It sits between editing and controllable generation: it does not modify a given image so much as constrain new generation to a structural template, and it is routinely combined with the attention- and inversion-based operations above. Its zero-conv, additive-adapter design is also a bridge to the parameter-efficient adaptation used for [[diffusion_model_personalization]].

### Editing in flow models

As the base generators shift to rectified-flow models (SD3, FLUX; [[flow_matching]]), editing has followed, inheriting the same inversion problem. Straightforward RF inversion approximates the current-step velocity with the next-step velocity, so forward and reverse steps disagree and reconstruction error accumulates — the flow analogue of DDIM inversion error {2025_Xie_DNAEdit}{2025_Wang_RFEdit}. Three responses recur, mirroring the diffusion fixes: **higher-order solvers plus attention injection** (RF-Edit adds a second-order RF solver and injects self-attention values from inversion into denoising to hold structure) {2025_Wang_RFEdit}; **noise re-alignment** (DNAEdit iteratively updates the target noise during inversion to keep paths straight, with a mobile-velocity guidance term for editing) {2025_Xie_DNAEdit}; and **avoiding inversion entirely** (FlowEdit fuses the source-inversion and target-denoising ODEs into a direct source-to-target "bridge" whose velocity is a Gaussian expectation, never routing through the noise distribution — an SDEdit-like idea realized in velocity space, achieving lower transport cost and better structure/edit balance) {2025_Kulikov_FlowEdit}. That the flow-editing frontier re-derives the diffusion-editing playbook underscores flow matching and diffusion as one family.

### Benchmarks and the three meanings of "inversion"

Evaluation is converging on paired metrics — structure/background preservation (PSNR, LPIPS, SSIM, MSE) against edit adherence (CLIP image–text similarity, often the CLIP *directional* similarity that measures change-vs-change) — standardized by **PIE-Bench**, 700 images across 10 edit types with masks and source/target prompts {2024_Ju_PnPInv}{2023_Brooks_IP2P}. A persistent hazard for readers is that "inversion" names three distinct operations in this region of the graph: **latent/noise inversion** (image → editable noise, this node) {2023_Hong_ExactInv}{2025_Staniszewski}; **textual/concept inversion** (images → pseudo-word embedding, [[diffusion_model_personalization]]) {2022_Mokady_NullText}; and **prompt inversion** (image → human-readable hard prompt), which Mahajan et al. solve by projected-gradient optimization over token embeddings, finding that high-level semantics are encoded at *high* timesteps {2023_Mahajan_PromptInv}. Mokady et al. draw the sharpest contrast: textual inversion "maps all noise vectors to a single image," whereas null-text (latent) inversion "focuses on a single trajectory of noisy vectors" {2022_Mokady_NullText}.

### Positioning

Image editing is where the diffusion subgraph's machinery is put to applied use: it consumes guidance ([[diffusion_guidance]]), the DDIM/ODE inversion of the samplers ([[diffusion_samplers]]), the semantic structure of the latent space ([[diffusion_model_semantic_spaces]]), and increasingly the velocity fields of [[flow_matching]]. It is tightly coupled to [[diffusion_model_personalization]] — both invert images and both fight the fidelity-vs-editability trade-off — but distinct in aim: editing changes a specific image, personalization teaches the model a reusable concept. The inversion-error and manifold questions it raises connect to [[diffusion_geometry]], and its reliance on CLIP for both conditioning and evaluation ties it to [[multimodal_ssl]]. Unauthorized editing is also the misuse that protective adversarial perturbations (PhotoGuard, EditShield) are designed to prevent — the adversarial-examples flip side of this node ([[aes_for_generative_models]]).

## References

{2022_Hertz_P2P} 2022_Hertz_Prompt-to-Prompt_Image_Editing_with_Cross_Attention_Control
{2022_Meng_SDEdit} 2022_Meng_SDEdit_Image_Synthesis_and_Editing_with_Stochastic_Differential_Equations
{2022_Mokady_NullText} 2022_Mokady_Null-text_Inversion_for_Editing_Real_Images_using_Guided_Diffusion_Models
{2023_Brooks_IP2P} 2023_Brooks_InstructPix2Pix_Learning_to_Follow_Image_Editing_Instructions
{2023_Hong_ExactInv} 2023_Hong_On_Exact_Inversion_of_DPM-Solvers
{2023_Mahajan_PromptInv} 2023_Mahajan_Prompting_Hard_or_Hardly_Prompting_Prompt_Inversion_for_Text-to-Image_Diffusion_Models
{2023_Zhang_ControlNet} 2023_Zhang_Adding_Conditional_Control_to_Text-to-Image_Diffusion_Models
{2024_Ju_PnPInv} 2024_Ju_PnP_Inversion_Boosting_Diffusion-Based_Editing_with_3_Lines_of_Code
{2024_Liu_Attention} 2024_Liu_Towards_Understanding_Cross_and_Self-Attention_in_Stable_Diffusion_for_Text-Guided_Image_Editing
{2025_Kulikov_FlowEdit} 2025_Kulikov_FlowEdit_Inversion-Free_Text-Based_Editing_using_Pre-Trained_Flow_Models
{2025_Staniszewski} 2025_Staniszewski_There_and_Back_Again_On_the_Relation_Between_Noise_and_Image_Inversions_in_Diffusion_Models
{2025_Wang_RFEdit} 2025_Wang_Taming_Rectified_Flow_for_Inversion_and_Editing
{2025_Xie_DNAEdit} 2025_Xie_DNAEdit_Direct_Noise_Alignment_for_Text-Guided_Rectified_Flow_Editing
