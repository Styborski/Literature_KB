---
title: "Diffusion Model Personalization"
tags: ["diffusion-models", "personalization", "few-shot-learning", "fine-tuning", "text-to-image", "generative-models"]
last_updated: "2026/08/31"
---

## Summary

Personalization teaches a pretrained text-to-image diffusion model a *new, user-specified concept* — a particular object, person, style, or action — from only a handful of example images, then lets that concept be recomposed into novel scenes by prompt. Methods differ mainly in **what they optimize**: a new text embedding (Textual Inversion), the full model weights (DreamBooth), a small weight subset or low-rank adapter (Custom Diffusion, LoRA), or an amortized encoder that predicts the concept in one forward pass (ELITE). All fight the same hazards — overfitting, forgetting, and a reconstruction-versus-editability trade-off — and all struggle most at composing *multiple* learned concepts. This node covers those methods and trade-offs, delegating the editable-space structure to [[diffusion_model_semantic_spaces]] and single-image edits to [[diffusion_based_image_editing]].

## Related Concepts
[[diffusion_models]]
[[latent_diffusion_models]]
[[diffusion_guidance]]
[[diffusion_based_image_editing]]
[[diffusion_model_semantic_spaces]]
[[multimodal_ssl]]
[[parameter_efficient_finetuning]]
[[disentanglement]]
[[diffusion_model_concept_erasure]]
[[aes_for_generative_models]]
[[membership_inference_attacks]]

## Open Questions

The central unresolved trade-off is **fidelity versus editability**: capturing a subject's identity precisely tends to push the learned embedding or weights out of distribution, degrading recomposition {2022_Gal_TI}{2024_Zhang_CompInv}. **Multi-concept composition** is largely unsolved — jointly generating two learned concepts causes identity loss and attention domination, and fusion of separately trained concepts conflicts {2023_Kumari_CustomDiff}{2023_Gu_MixOfShow}. It is unclear what a learned concept token *should* be: a single embedding, one per layer, or per-image tokens, and why out-of-distribution embeddings hijack cross-attention {2023_Voynov_Pplus}{2024_Zhang_CompInv}. Which weights matter for a concept, and whether the low-rank finetuning assumption holds for images as it does for language, is only partly characterized {2023_Kumari_CustomDiff}{2021_Hu_LoRA}. Whether concepts beyond object appearance — actions, relations — can be inverted without hand-holding remains open {2023_Motamed_Lego}.

## Synthesis

### The task: few-shot concept acquisition

Personalization asks a pretrained generator to learn a concept from ~3–5 images and then wield it compositionally — "a photo of [my dog] on the moon," "a painting in the style of [S*]" {2022_Gal_TI}{2023_Ruiz_DreamBooth}. Naively fine-tuning the whole model on a few images overfits and induces catastrophic forgetting, so the field's methods are distinguished by how they inject the concept while retaining the model's prior. Two quality axes recur throughout: **subject fidelity** (does the output look like the reference concept, measured by CLIP-I / DINO-I image similarity) and **editability / prompt fidelity** (can the concept be recomposed by new prompts, measured by CLIP-T text similarity) {2022_Gal_TI}{2023_Ruiz_DreamBooth}. Improving one typically costs the other — the organizing trade-off of the node, and the same axis as the distortion–editability trade-off in [[diffusion_based_image_editing]].

### Embedding-space personalization: textual inversion

The lightest approach freezes the entire model and learns a single **new pseudo-word embedding** $v_*$ by the ordinary denoising loss, so the concept is captured purely in text-conditioning space {2022_Gal_TI}:
$$v_* = \arg\min_v\; \mathbb{E}_{z,y,\epsilon,t}\big\lVert \epsilon - \epsilon_\theta(z_t, t, c_\theta(y))\big\rVert_2^2,$$
where $y$ embeds a prompt containing the placeholder. Because nothing but the embedding changes, the model's prior is fully preserved and the concept composes with all existing knowledge; the cost is limited expressiveness, and Gal et al. map an explicit reconstruction–editability curve on which a single-word embedding sits near the elbow, with multi-word "extended" embeddings buying reconstruction at the expense of editability {2022_Gal_TI}. Textual inversion is also the origin of the term "inversion" in personalization — inverting images into *text-embedding* space — which is distinct from the latent-noise inversion of [[diffusion_based_image_editing]].

### Weight-space personalization: DreamBooth

The complementary approach fine-tunes the **model weights** so the concept is stored in the network, not just a token {2023_Ruiz_DreamBooth}. DreamBooth binds the concept to a *rare-token* unique identifier paired with a coarse class word ("a [V] dog"), fine-tunes all layers, and — to counter the language drift and mode collapse that whole-model few-shot tuning causes — adds a **prior-preservation loss**: the model is simultaneously trained to reproduce its *own* pre-finetuning samples of the class, anchoring the broader class distribution while the subject is learned {2023_Ruiz_DreamBooth}:
$$\mathbb{E}\big[\lVert x_\theta(z_t,c) - x\rVert^2 + \lambda\,\lVert x_\theta(z_{t'},c_{\text{pr}}) - x_{\text{pr}}\rVert^2\big].$$
This yields higher subject fidelity than textual inversion (and higher still on a stronger base model), at the cost of storing a full fine-tuned model per concept and greater risk of overfitting; ablating prior preservation visibly collapses class diversity {2023_Ruiz_DreamBooth}. Textual inversion and DreamBooth stake out the extremes — pure embedding versus full weights — that the remaining methods interpolate.

### The design spectrum: what to optimize

Most subsequent work chooses an intermediate parameterization trading capacity against overfitting, storage, and forgetting.

- **A weight subset.** Custom Diffusion measures which weights move most during fine-tuning and finds the **cross-attention key/value projections** dominate (roughly twice the relative change of other weights); tuning only $W_k, W_v$ plus a new token matches DreamBooth at far lower storage and overfitting {2023_Kumari_CustomDiff}. That the text-conditioning cross-attention is where concepts live echoes the editing finding that cross-attention carries object semantics ([[diffusion_based_image_editing]]).
- **A low-rank adapter.** LoRA freezes $W_0$ and learns a low-rank update $W_0 + BA$ with $r\!\ll\!d$, motivated by the hypothesis that fine-tuning updates are intrinsically low-rank; it was introduced for language models but is now the default parameter-efficient route for diffusion personalization, storing a few megabytes per concept and composing additively {2021_Hu_LoRA}. LoRA analysis shows the update chiefly *amplifies directions already latent but under-emphasized* in $W_0$ — a useful lens on why light personalization works ([[parameter_efficient_finetuning]]) {2021_Hu_LoRA}.
- **An amortized encoder.** Rather than optimize per concept, ELITE trains a network mapping a CLIP image encoding to text embeddings (a global branch for identity, a local branch for spatial detail), personalizing in a *single forward pass* (~0.05 s) instead of thousands of optimization steps {2023_Wei_ELITE}. This trades a costly encoder-training stage for near-instant test-time personalization.
- **Per-layer embeddings.** P+ / Extended Textual Inversion learns a *separate* token embedding for each cross-attention layer rather than one shared embedding, which is more expressive, converges faster, and — because the per-layer embeddings stay closer to the real token distribution — is *more* editable than textual inversion {2023_Voynov_Pplus}. Its layer-attribute analysis is broadly useful: coarse object identity is governed by the low-resolution (middle) U-Net layers, style by intermediate layers, and color by the highest-resolution layers {2023_Voynov_Pplus}.

### Overfitting, forgetting, and their regularizers

A single hazard underlies the design spectrum: fitting a few images tends to overfit them and drift the model off its prior (language drift, mode collapse). The countermeasures form a family — DreamBooth's class prior-preservation loss {2023_Ruiz_DreamBooth}, Custom Diffusion's regularization set of retrieved real images of the class {2023_Kumari_CustomDiff}, and Compositional Inversion's **anchor loss** pulling the learned embedding toward a set of real token embeddings {2024_Zhang_CompInv}. The diagnosis is consistent across papers: learned embeddings land *outside* the core of the token distribution (higher entropy), which lets them **dominate cross-attention** and crowd out other concepts at composition time {2024_Zhang_CompInv}. This is the concrete failure mode behind the fidelity–editability trade-off and the chief obstacle to composing multiple concepts.

### Beyond object appearance

Most methods invert a *noun* — an object or subject. Extending inversion to **actions, relations, and adjectives** requires disentangling the concept from the subject that carries it. Lego learns multi-token concepts and separates a concept ("frozen," "walking") from its subject by training the subject on concept-free images and the concept on paired images, adding an InfoNCE **context loss** using synonym/antonym embeddings to shape the concept token {2023_Motamed_Lego}; the paper notes its "disentanglement" is partly manual (via the curated image split), a candid caveat. Relation inversion (ReVersion) and appearance-beyond-object variants pursue the same goal, as characterized here {2023_Motamed_Lego}. These efforts connect personalization to [[disentanglement]].

### Multi-concept composition: the hard frontier

Combining several learned concepts in one image is where methods break. Textual inversion composes poorly; DreamBooth and Custom Diffusion do better, and Custom Diffusion offers two multi-concept routes — **joint training** on merged data with separate tokens, and **closed-form weight merging** that solves a constrained least-squares problem to fuse independently trained $W_k, W_v$ while preserving their effect on general concepts {2023_Kumari_CustomDiff}. Mix-of-Show diagnoses why naive fusion (weighted averaging of LoRAs) loses identity — embeddings capture in-domain semantics while LoRAs carry out-of-domain identity, so averaging LoRAs creates conflict — and proposes **gradient fusion** (optimize a single weight whose layer outputs match each concept's individual outputs) together with layer-wise embeddings and region-controlled sampling {2023_Gu_MixOfShow}. Compositional Inversion adds anchor and detection-based spatial losses to keep composed concepts in-distribution and spatially separated, though the source's own critique is that the fixed spatial targets induce near-identical layouts, a form of mode collapse — a useful reminder that composition fixes can trade diversity for controllability {2024_Zhang_CompInv}.

### Evaluation and positioning

Evaluation is standardized on paired similarity metrics — CLIP-I / DINO-I for subject fidelity against CLIP-T for prompt fidelity — over concept benchmarks such as CustomConcept101 {2023_Kumari_CustomDiff}{2023_Ruiz_DreamBooth}, tying the node to CLIP and thus [[multimodal_ssl]]. Personalization is a close sibling of [[diffusion_based_image_editing]]: both adapt a pretrained generator to specific content, both center on an "inversion," and both are governed by the fidelity-vs-editability trade-off — but editing changes one image whereas personalization mints a reusable concept. It runs on latent text-to-image models ([[latent_diffusion_models]]) and is sampled with guidance well above 1 ([[diffusion_guidance]]). Finally, personalization has an adversarial mirror image. Users' images can be *protected* from non-consensual personalization by adding imperceptible adversarial perturbations that break DreamBooth-style fine-tuning (Anti-DreamBooth, Glaze) — the generative-model instantiation of adversarial examples ([[aes_for_generative_models]]); the complementary route of *erasing* a concept from the model weights is the safety cluster [[diffusion_model_concept_erasure]].

## References

{2021_Hu_LoRA} 2021_Hu_LoRA_Low-Rank_Adaptation_of_Large_Language_Models
{2022_Gal_TI} 2022_Gal_An_Image_is_Worth_One_Word_Personalizing_Text-to-Image_Generation_using_Textual_Inversion
{2023_Gu_MixOfShow} 2023_Gu_Mix-of-Show_Decentralized_Low-Rank_Adaptation_for_Multi-Concept_Customization_of_Diffusion_Models
{2023_Kumari_CustomDiff} 2023_Kumari_Multi-Concept_Customization_of_Text-to-Image_Diffusion
{2023_Motamed_Lego} 2023_Motamed_Lego_Learning_to_Disentangle_and_Invert_Concepts_Beyond_Object_Appearance_in_Text-to-Image_Diffusion_Models
{2023_Ruiz_DreamBooth} 2023_Ruiz_DreamBooth_Fine_Tuning_Text-to-Image_Diffusion_Models_for_Subject-Driven_Generation
{2023_Voynov_Pplus} 2023_Voynov_Pplus_Extended_Textual_Conditioning_in_Text-to-Image_Generation
{2023_Wei_ELITE} 2023_Wei_ELITE_Encoding_Visual_Concepts_into_Textual_Embeddings_for_Customized_Text-to-Image_Generation
{2024_Zhang_CompInv} 2024_Zhang_Compositional_Inversion_for_Stable_Diffusion_Models
