---
title: "Adversarial Examples"
tags: ["adversarial-examples", "adversarial-machine-learning", "robustness", "security", "attacks-and-defenses"]
last_updated: "2026/08/22"
---

## Summary

Adversarial examples are inputs deliberately perturbed to make a machine-learning model fail, typically while looking benign to humans. First identified by Szegedy et al. and popularized by Goodfellow et al.'s FGSM, they revealed that high-accuracy networks are brittle in directions the training distribution never exercises. This node is the hub for the adversarial-machine-learning subgraph: it fixes the shared vocabulary (evasion vs. poisoning, white- vs. black-box, targeted vs. untargeted, $L_p$ threat models), tracks the attack-defense arms race, and routes to the specialized nodes — attacks on classifiers ([[aes_for_discriminative_models]]), their defenses ([[defenses_against_aes_for_discriminative_models]]), attacks on generative models ([[aes_for_generative_models]]), and the train-time and language-model variants collected as roadmap nodes below.

## Related Concepts
[[aes_for_discriminative_models]]
[[defenses_against_aes_for_discriminative_models]]
[[aes_for_generative_models]]
[[defenses_against_aes_for_generative_models]]
[[data_poisoning]]
[[backdoor_attacks]]
[[llm_jailbreaks]]
[[adversarial_transferability]]
[[robustness_accuracy_tradeoff]]
[[certified_robustness]]
[[robust_features]]
[[image_augmentation_methods]]
[[contrastive_ssl]]
[[self_supervised_learning]]
[[vision_transformers]]

## Open Questions

The field's foundational question — *why* adversarial examples exist — is unresolved: the linearity account {2015_Goodfellow}, the data-manifold "pockets" view {2014_Szegedy}, and robust-vs-non-robust-feature accounts coexist without a decisive test. Progress is structurally hampered by evaluation: defenses repeatedly fall to stronger attacks {2017_Carlini}, so the field advances by refutation and "robust accuracy" is only ever an upper bound. Whether robustness must cost clean accuracy, data, and compute is contested {2019_Zhang_TRADES}. It is also unclear how much of the classical CNN/ImageNet picture transfers to Vision Transformers, self-supervised encoders, generative models, and LLMs, each of which shows a partly different vulnerability profile. The relationship between adversarial robustness and human-aligned perception (shape vs. texture bias) is suggestive but not settled {2018_Geirhos}.

## Synthesis

### What adversarial examples are, and why they matter

An adversarial example is an input $x' = x + \delta$, with $\delta$ restricted to a small set (usually an $L_p$ ball), chosen so a model errs even though $x'$ is, to a human, equivalent to $x$. Szegedy et al. discovered that imperceptible perturbations reliably flip predictions and *transfer* across models, arguing the vulnerability is intrinsic to the learned function {2014_Szegedy}. Goodfellow et al. showed the effect follows from the near-linear behaviour of high-dimensional models and made attacks trivially cheap with FGSM, simultaneously proposing adversarial training as a fix {2015_Goodfellow}. The significance is twofold: a *security* concern for deployed systems (including physical-world and black-box settings), and a *scientific* signal that networks generalize on-distribution while relying on features that do not match human perception — connecting to texture-vs-shape bias {2018_Geirhos} and the [[robust_features]] hypothesis.

### The shared taxonomy

The subfield is organized by a few orthogonal axes, most of them crystallized in Demontis et al.'s transferability study {2019_Demontis}:
- **Evasion vs. poisoning** — perturb *test* inputs to cause misclassification, vs. perturb *training* data to corrupt the learned model. This node and [[aes_for_discriminative_models]] center evasion; train-time attacks are [[data_poisoning]] and [[backdoor_attacks]].
- **White-box vs. black-box** — attacker knows the target's weights and can use its gradients, vs. attacker must use a *surrogate* model and rely on [[adversarial_transferability]].
- **Targeted vs. untargeted** — force a specific wrong class vs. any wrong class.
- **Threat model / budget** — the allowed perturbation set, almost always an $L_0$, $L_2$, or $L_\infty$ ball of radius $\epsilon$; digital vs. physical realizability adds another dimension.
These distinctions determine which attack and which defense are even applicable, so they recur in every node of the subgraph.

### The attack-defense arms race

The field advances as a cycle: a defense is proposed, then a stronger, defense-aware attack either breaks it or certifies it. Carlini & Wagner's refutation of defensive distillation is the canonical episode — apparent robustness was *gradient masking*, and stronger attacks recovered near-total success {2017_Carlini}. Madry et al. reframed defense as a min-max problem and argued PGD is a near-universal first-order attack, so PGD-based adversarial training became the empirical standard {2019_Madry}. The tradeoff between robustness and clean accuracy was then formalized (TRADES) as a decomposition of robust error into natural plus boundary error {2019_Zhang_TRADES}. Because empirical defenses never prove the absence of adversarial examples, a parallel effort seeks *certified* guarantees ([[certified_robustness]]). The mechanics of attacks live in [[aes_for_discriminative_models]] and of defenses in [[defenses_against_aes_for_discriminative_models]].

### Scope: beyond classifiers

Adversarial phenomena extend well past supervised image classifiers, and this hub links the specialized areas even where the present knowledge base covers them only lightly:
- **Generative models** — attacks that manipulate reconstructions, latents, or generations (VAEs, diffusion), and adversarial perturbations used *protectively* to prevent unauthorized image editing or style imitation, plus the purification/robustness countermeasures that strip them. Scoped as [[aes_for_generative_models]] and [[defenses_against_aes_for_generative_models]], where the usual attacker/defender roles invert (the "attacker" is often the data owner).
- **Train-time attacks** — [[data_poisoning]] (including availability / "unlearnable-example" poisons) and [[backdoor_attacks]] (trigger-implanted behaviour), a large cluster in `articles/` reserved for future nodes.
- **Language models** — jailbreaks and prompt-injection against aligned LLMs, collected as [[llm_jailbreaks]].
- **Self-supervised and robust pretraining** — robustness as a property of representations, via adversarial contrastive learning and unsupervised adversarial training, bridging to [[contrastive_ssl]] and [[self_supervised_learning]] (detailed in the defenses node).
These pointers are a roadmap: the discriminative attack/defense nodes are populated now, the rest are planned. Perturbation-based augmentation also connects here — adversarial examples are, formally, worst-case augmentations ([[image_augmentation_methods]]).

## References

{2014_Szegedy} 2014_Szegedy_Intriguing_Properties_of_Neural_Networks
{2015_Goodfellow} 2015_Goodfellow_Explaining_and_Harnessing_Adversarial_Examples
{2017_Carlini} 2017_Carlini_Towards_Evaluating_the_Robustness_of_Neural_Networks
{2018_Geirhos} 2018_Geirhos_ImageNet-Trained_CNNs_are_Biased_Towards_Texture_Increasing_Shape_Bias_Improves_Accuracy_and_Robustness
{2019_Demontis} 2019_Demontis_Why_Do_Adversarial_Attacks_Transfer_Explaining_Transferability_of_Evasion_and_Poisoning_Attacks
{2019_Madry} 2019_Madry_Towards_Deep_Learning_Models_Resistant_to_Adversarial_Attacks
{2019_Zhang_TRADES} 2019_Zhang_Theoretically_Principled_Trade-off_between_Robustness_and_Accuracy
