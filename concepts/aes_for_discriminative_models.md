---
title: "Adversarial Examples for Discriminative Models"
tags: ["adversarial-examples", "evasion-attacks", "robustness", "transferability", "threat-models"]
last_updated: "2026/08/22"
---

## Summary

Adversarial examples for discriminative models are inputs perturbed by small, usually imperceptible changes that make a trained classifier or detector produce confident errors at test time (*evasion*). Discovered by Szegedy et al. and explained through the near-linear behaviour of high-dimensional networks by Goodfellow et al., they are found by gradient-based optimization under an $L_p$ budget — FGSM, iterative BIM/PGD, and the strong Carlini-Wagner attacks. Attacks may be targeted or untargeted, white-box or black-box (via transferable surrogate models), and can survive printing and re-imaging in the physical world. This node organizes the attack algorithms, threat models, transferability, and robustness evaluation; countermeasures live in [[defenses_against_aes_for_discriminative_models]].

## Related Concepts
[[adversarial_examples]]
[[defenses_against_aes_for_discriminative_models]]
[[adversarial_transferability]]
[[aes_for_generative_models]]
[[data_poisoning]]
[[robust_features]]
[[robustness_accuracy_tradeoff]]
[[vision_transformers]]
[[contrastive_ssl]]
[[image_augmentation_methods]]

## Open Questions

Why adversarial examples exist is still contested: the original *linearity* account {2015_Goodfellow} competes with *robust vs. non-robust features* views implicit in later work, and no single explanation predicts all phenomena. Robustness evaluation is fragile — attacks routinely defeat defenses that once looked strong {2017_Carlini}, and even PGD under-estimates vulnerability unless carefully configured (restarts, surrogate loss, optimizer) {2019_Gowal}, so reported "robust accuracy" is an upper bound of unknown tightness. Transferability is only partly understood: gradient alignment, gradient magnitude, and surrogate loss variance correlate with it {2019_Demontis}, but there is no predictive theory of when a black-box surrogate will succeed. Whether architecture matters intrinsically — e.g. ViTs vs. CNNs — beyond training and augmentation is unresolved {2021_Naseer}.

## Synthesis

### Discovery and the two founding hypotheses

Szegedy et al. introduced the term *adversarial example*, showing that imperceptible perturbations found by a box-constrained L-BFGS search flip a network's prediction, and that such examples transfer across models and training sets — implying the vulnerability is a property of the learned function class, not a single model {2014_Szegedy}. They framed the phenomenon as low-probability "pockets" or discontinuities densely scattered near the data manifold. Goodfellow et al. reframed the cause as *linearity*: in high dimensions, a perturbation $\eta = \epsilon \cdot \mathrm{sign}(\nabla_x L)$ aligned with the weight vector accumulates many small coordinate changes into a large change in the pre-activation, so confidently-wrong predictions arise from models being too locally linear, not too nonlinear {2015_Goodfellow}. They noted radial-basis-function networks, which are locally non-linear, resist the same attack by returning low confidence off-manifold. These two papers set up the enduring tension between "data-manifold" and "feature/linearity" explanations, developed further under [[robust_features]].

### Attack algorithms and the $L_p$ threat model

Attacks are defined relative to a *threat model*: an allowed perturbation set, almost always an $L_p$ ball $\|x'-x\|_p \le \epsilon$, with $L_0$ (number of pixels changed), $L_2$ (Euclidean), and $L_\infty$ (max per-pixel change) dominant {2017_Carlini}. A second axis is *targeted* (force a chosen class $t$) vs. *untargeted* (any wrong class). The canonical algorithms:
- **L-BFGS** {2014_Szegedy}: minimize $c \cdot \|x-x'\|_2^2 + L_{F,t}(x')$ with line search over $c$ — accurate but slow.
- **FGSM** {2015_Goodfellow}: one $L_\infty$ step along $\mathrm{sign}(\nabla_x L)$ — fast, weak.
- **BIM / iterative FGSM** {2017_Kurakin}: many small clipped FGSM steps; the untargeted form ascends the true-class loss and a "least-likely" variant descends toward a target class. BIM is essentially PGD without random restarts.
- **JSMA** {2017_Carlini}: greedy $L_0$ attack using a saliency map $\nabla Z(x)_l$ to pick the few most influential pixels.
- **DeepFool** {2017_Carlini}: untargeted $L_2$ attack that linearizes the boundary and steps to it iteratively.
- **Carlini-Wagner (C&W)** {2017_Carlini}: recasts the search as $\min \|x-x'\|_p + c \cdot f(x')$ with a well-chosen surrogate $f$ (a logit-margin $f(x') = \max(\max_{i \neq t} Z(x')_i - Z(x')_t, -\kappa)$), a $\tanh$ change of variables to enforce the box constraint, and Adam. The confidence margin $\kappa$ trades imperceptibility for attack strength. C&W's $L_0$/$L_2$ attacks find perturbations 2–10x smaller than prior attacks and, crucially, break defensive distillation (see below).

### Optimizing the inner maximization: PGD and its limits

Adversarial-example generation is the inner maximization $\max_{\delta \in S} L(\theta, x+\delta, y)$. Madry et al. argued that **PGD** — projected gradient descent with random restarts — is a "universal" first-order attack: across thousands of restarts the local maxima it finds have similar loss, so a model robust to PGD is empirically robust to all first-order attacks {2019_Madry}. This made PGD both the standard attack and the engine of adversarial training ([[defenses_against_aes_for_discriminative_models]]). But PGD is not optimal: its success depends on the starting point, the surrogate loss (cross-entropy vs. logit margin), the optimizer (Adam more consistent than plain sign-SGD), the step-size schedule, and the number of restarts {2019_Gowal}. Gowal et al.'s **MultiTargeted** attack loops over target classes with a per-target margin loss $-z_y + z_i$, provably finding the optimal perturbation for locally-linear models and beating standard PGD at fixed compute against Madry-AT, TRADES, and UAT {2019_Gowal}. The practical lesson — a defense is only as evaluated as its strongest adaptive attack — recurs throughout the field.

### Transferability and black-box attacks

When the attacker lacks the target's weights (*black-box*), it crafts perturbations on a *surrogate* model and hopes they *transfer*. Demontis et al. give the most systematic account, deriving three quantities that govern transfer of both evasion and poisoning attacks {2019_Demontis}: (i) **target loss-gradient magnitude** $S = \|\nabla_x L\|_q$ — high-complexity, weakly-regularized targets have large gradients and are easier to fool; (ii) **surrogate-target gradient alignment** $R$ — the cosine similarity between surrogate and target loss gradients, the single best predictor of transfer; and (iii) **surrogate loss variance** $V$ — complex surrogates have turbulent loss surfaces with many idiosyncratic optima that transfer poorly. The counter-intuitive practical implications: *low-complexity* (smooth, regularized) surrogates transfer better because they approximate the average target, and low-complexity targets are inherently harder to attack. This links black-box attack strength to model regularization — the same knob defenses turn.

### Physical-world, architectures, and other input types

Adversarial examples are not confined to digital pixels. Kurakin et al. printed adversarial images, photographed them, and still fooled the classifier, establishing a physical-world threat and introducing BIM/LL-BIM as robust-under-transformation attacks {2017_Kurakin}. The attack surface also depends on architecture: Naseer et al. find Vision Transformers are generally *more* robust than ResNet-50 to FGSM, PGD, and patch attacks (and to occlusion and natural corruptions), attributing this to attention's dynamic, global receptive field — though Stylized-ImageNet training, which raises shape bias, *lowers* adversarial robustness, evidence of a shape-bias-vs-robustness tension {2021_Naseer}. Attacks generalize beyond supervised classifiers too: contrastive encoders admit *instance-wise* attacks that maximize the InfoNCE loss to confuse instance identity (the attack half of the adversarial-contrastive-learning story in [[defenses_against_aes_for_discriminative_models]] and [[contrastive_ssl]]). Attacks on generative models are scoped separately in [[aes_for_generative_models]], and train-time attacks in [[data_poisoning]] — Demontis's transferability analysis spans both evasion and poisoning {2019_Demontis}.

### Robustness evaluation as an arms race

Because there is no way to enumerate all inputs in an $\epsilon$-ball, measured robustness is only ever an upper bound set by the attacker's strength, which makes evaluation the field's central methodological problem. Carlini & Wagner's demolition of *defensive distillation* is the paradigm case: distillation appeared to grant robustness only because it flattened gradients and stalled existing attacks (L-BFGS, JSMA, DeepFool), while the adversarial examples were still there — C&W with a suitable loss and confidence margin reached ~100% success {2017_Carlini}. This exposed *gradient masking* as a pseudo-defense and set the standard that defenses must be tested against adaptive, gradient-aware attacks — a standard PGD {2019_Madry} and MultiTargeted {2019_Gowal} were designed to meet. Certified defenses that give guarantees rather than empirical numbers are one response, treated under [[certified_robustness]] via [[defenses_against_aes_for_discriminative_models]].

## References

{2014_Szegedy} 2014_Szegedy_Intriguing_Properties_of_Neural_Networks
{2015_Goodfellow} 2015_Goodfellow_Explaining_and_Harnessing_Adversarial_Examples
{2017_Carlini} 2017_Carlini_Towards_Evaluating_the_Robustness_of_Neural_Networks
{2017_Kurakin} 2017_Kurakin_Adversarial_Examples_in_the_Physical_World
{2019_Demontis} 2019_Demontis_Why_Do_Adversarial_Attacks_Transfer_Explaining_Transferability_of_Evasion_and_Poisoning_Attacks
{2019_Gowal} 2019_Gowal_An_Alternative_Surrogate_Loss_for_PGD-based_Adversarial_Testing
{2019_Madry} 2019_Madry_Towards_Deep_Learning_Models_Resistant_to_Adversarial_Attacks
{2021_Naseer} 2021_Naseer_Intriguing_Properties_of_Vision_Transformers
