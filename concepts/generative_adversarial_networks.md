---
title: "Generative Adversarial Networks"
tags: ["generative-adversarial-networks", "generative-models", "adversarial-training", "image-synthesis", "mode-collapse"]
last_updated: "2026/08/22"
---

## Summary

A generative adversarial network (GAN) trains a generator and a discriminator in a minimax game: the discriminator learns to tell real data from the generator's samples, and the generator learns to fool it, so that at equilibrium the generator's distribution matches the data. GANs were the dominant high-fidelity image-synthesis paradigm through the late 2010s, prized for sharp samples but notorious for training instability, mode collapse, and a diversity-fidelity tradeoff. This node is a **seed**: the present knowledge base has no foundational GAN methodology papers, so it is anchored on the engaged material that touches GANs — data-efficient training via augmentation, the GAN-versus-diffusion comparison, and adversarial attacks on generative models — with the coverage gap flagged where it bears on claims.

## Related Concepts
[[generative_models]]
[[variational_autoencoders]]
[[diffusion_models]]
[[latent_diffusion_models]]
[[image_augmentation_methods]]
[[aes_for_generative_models]]
[[generated_image_detection]]
[[generated_image_evaluation_metrics]]
[[adversarial_examples]]
[[adversarial_contrastive_learning]]

## Open Questions

GANs' defining difficulties remain only partly resolved. The **diversity-fidelity tradeoff** — GANs achieve high sample quality by covering fewer modes, i.e. high precision but low recall — is robust across models but has no clean fix {2021_Dhariwal}. **Training instability and mode collapse** persist, with the low-data regime especially fragile because the discriminator memorizes the real set; augmentation mitigates but does not eliminate this {2020_Zhao_DiffAugment}. Whether GANs retain any lasting advantage now that diffusion models surpass them on fidelity *and* diversity — perhaps single-step sampling speed — is an open positioning question {2021_Dhariwal}. Reliable, generalizable detection of GAN-generated imagery is an active and unsettled area (a large but here-unread literature, flagged under [[generated_image_detection]]).

## Synthesis

### The adversarial framework

A GAN couples a generator $G$, mapping a latent $z \sim p(z)$ to a sample, with a discriminator $D$ that scores inputs as real or generated; training is the minimax game $\min_G \max_D \mathbb{E}_x[ \log D(x) ] + \mathbb{E}_z[ \log(1 - D(G(z))) ]$, alternating discriminator and generator updates so that $G$ is pushed toward the data distribution while $D$ supplies the learning signal. The discriminator loss is typically a hinge or cross-entropy on real-vs-fake, and the generator is trained to maximize the discriminator's error {2020_Zhao_DiffAugment}. **Coverage note:** the foundational GAN papers (Goodfellow et al. 2014, and the architecture line DCGAN / WGAN / StyleGAN / BigGAN / CycleGAN) are not present in `articles/`, so this framework description is background context rather than a synthesis of consulted sources; the subsections below draw on the GAN-adjacent works the knowledge base *does* contain.

### Training pathologies

GANs are hard to train, and the knowledge base's clearest window on this is the low-data regime. DiffAugment shows that with limited data a GAN's discriminator overfits — its training accuracy diverges from validation accuracy and it begins to memorize the real images — after which the generator's feedback degrades and sample quality (FID) collapses {2020_Zhao_DiffAugment}. This is one facet of the broader, long-standing GAN failure modes of **mode collapse** (the generator emits a few modes to exploit discriminator weaknesses) and unstable, oscillatory optimization; the diversity side of the problem is quantified by Dhariwal & Nichol, who show GANs systematically trade recall (coverage) for precision (fidelity) {2021_Dhariwal}.

### Data-efficient training via augmentation

The engaged GAN-methodology result is DiffAugment {2020_Zhao_DiffAugment}: augmentation stabilizes data-efficient GAN training, but only if the *same* differentiable transform is applied to **both** real and generated images at the discriminator (and backpropagated to the generator). Augmenting reals only makes the generator reproduce augmentation artifacts; augmenting the discriminator's inputs only leaves it accurate on augmented but not clean images. Applying augmentation everywhere and differentiably ("Augment everything") sharply improves FID in the low-data regime across BigGAN and StyleGAN {2020_Zhao_DiffAugment}. Here "differentiable" means backprop-able to the generator, not that the augmentation parameters are learned — a distinction developed under [[image_augmentation_methods]].

### Evaluation and the diversity-fidelity tradeoff

GAN progress is measured by Fréchet Inception Distance (FID) and Inception Score (IS), refined by precision (fidelity) and recall (diversity). Dhariwal & Nichol use these to make the GAN-vs-diffusion case: GANs win FID partly by sacrificing diversity, and *classifier guidance* can import the same knob into diffusion — scaling classifier gradients trades recall for precision, improving IS/precision while a moderate setting optimizes FID — after which guided diffusion surpasses BigGAN at matched IS {2021_Dhariwal}. This both explains GANs' historical FID advantage and shows it is not intrinsic to the adversarial framework. Robust, transferable detection of generated images is a distinct downstream concern collected under [[generated_image_detection]].

### GANs among the generative families

GANs are one of three dominant deep generative paradigms, and the knowledge base frames them mostly by contrast. Against [[variational_autoencoders]], GANs give sharper samples but no likelihood and less stable training (VAEs give stable likelihood optimization but blurry samples). Against [[diffusion_models]], GANs are faster to sample but are surpassed on both fidelity and diversity by guided diffusion {2021_Dhariwal}. This three-way tension among sample quality, mode coverage, and sampling cost is the organizing frame for [[generative_models]]; hybrid models such as VAE-GAN combine an encoder, decoder, and discriminator {2017_Kos}.

### Adversarial vulnerability of generative models

Like classifiers, generative models admit adversarial inputs. Kos et al. attack a VAE-GAN so that a slightly perturbed input reconstructs an attacker-chosen target, comparing three attack objectives — a classifier loss on the latent, a VAE reconstruction loss toward the target, and a latent-distance ("$L_\mathrm{lat}$") loss — and find the latent-targeting attacks most effective {2017_Kos}. This complements the VAE-focused attacks in [[variational_autoencoders]] and is collected, with the discriminative-model case, under [[aes_for_generative_models]] and [[adversarial_examples]]. Note the terminological overlap the "adversarial" label creates: a GAN is adversarial by *construction* (generator vs. discriminator), which is unrelated to being *attacked* by adversarial examples.

## References

{2017_Kos} 2017_Kos_Adversarial_Examples_for_Generative_Models
{2020_Zhao_DiffAugment} 2020_Zhao_Differentiable_Augmentation_for_Data-Efficient_GAN_Training
{2021_Dhariwal} 2021_Dhariwal_Diffusion_Models_Beat_GANs_on_Image_Synthesis
