---
title: "Defenses Against Adversarial Examples for Discriminative Models"
tags: ["adversarial-training", "robustness", "certified-defenses", "robustness-accuracy-tradeoff", "adversarial-examples"]
last_updated: "2026/08/22"
---

## Summary

Defenses aim to keep a discriminative model's prediction correct across an $L_p$ ball of perturbations. The dominant empirical defense is *adversarial training* (AT) — solving the min-max problem by training on PGD-generated worst-case inputs {2019_Madry} — with variants that manage the robustness-accuracy tradeoff explicitly (TRADES {2019_Zhang_TRADES}). A parallel line gives *certified* guarantees, most scalably via randomized smoothing {2019_Cohen}. Structural routes (sparsity, Lipschitz control) and label-free / self-supervised robustness (UAT {2019_Uesato_UAT}; adversarial contrastive learning) round out the space. A recurring theme is that robustness is expensive — in data, model capacity, clean accuracy, and optimization stability — and that any defense must be judged against adaptive attacks ([[aes_for_discriminative_models]]).

## Related Concepts
[[adversarial_examples]]
[[aes_for_discriminative_models]]
[[defenses_against_aes_for_generative_models]]
[[robustness_accuracy_tradeoff]]
[[certified_robustness]]
[[adversarial_transferability]]
[[robust_features]]
[[adversarial_contrastive_learning]]
[[contrastive_ssl]]
[[self_supervised_learning]]
[[image_augmentation_methods]]
[[knowledge_distillation]]
[[vision_transformers]]

## Open Questions

Whether robustness and clean accuracy are *inherently* opposed is unsettled: TRADES treats the tradeoff as fundamental {2019_Zhang_TRADES}, yet Yang et al. show real datasets are $r$-separated so a both-robust-and-accurate classifier exists in principle, and the gap is an artifact of current training {2020_Yang}. How much *data* robustness needs — and whether unlabeled data closes the gap — is only partly answered {2019_Uesato_UAT}. AT's optimization is poorly understood: the adversarial loss is second-order non-smooth, with no convergence guarantee for large budgets {2020_Liu}. Certification remains loose and hard to scale beyond $L_2$ {2019_Cohen}. Finally, whether robustness earned at ImageNet/CIFAR scale on CNNs transfers to ViTs, foundation models, and self-supervised pipelines is open, and the mechanism by which AT changes learned features is still debated {2020_Liu}.

## Synthesis

### Adversarial training and the min-max view

Adversarial training originates with the FGSM-augmented objective of Goodfellow et al. {2015_Goodfellow} and is put on firm footing by Madry et al., who cast robust learning as the saddle-point problem $\min_\theta \mathbb{E}[ \max_{\delta \in S} L(\theta, x+\delta, y) ]$ {2019_Madry}. Danskin's theorem justifies solving it by substituting each clean point with its inner-maximizer (a PGD adversarial example) and taking ordinary gradient steps on the outer minimization. Two robustness prerequisites emerge: **model capacity** — small networks cannot fit the harder adversarial objective and collapse toward constant classifiers, so robust models must be wider/deeper than their clean counterparts — and a **strong inner attack** — training against weak FGSM (especially large-$\epsilon$ single-step) fails, while PGD works because it approximates the true worst case {2019_Madry}. PGD-AT remains the empirical backbone against which other defenses are measured.

### The robustness-accuracy tradeoff

Robust models often generalize worse on clean data. Zhang et al.'s **TRADES** makes the tradeoff explicit by decomposing robust error into natural error plus a *boundary* error, and minimizing $\mathbb{E}[\phi(f(x)y)] + (1/\lambda)\,\mathbb{E}[\max_{x'} \phi(f(x)f(x'))]$ — a classification term plus a regularizer pushing the decision boundary away from data; $\lambda$ dials robustness against accuracy {2019_Zhang_TRADES}. Whether the tradeoff is *fundamental* is contested. Yang et al. show that standard image datasets are $r$-separated (classes are $\ge 2r$ apart, far exceeding attack radii), so a classifier that is both astute and locally Lipschitz *exists*; every practical training method nonetheless sacrifices one for the other, and the generalization gap can be narrowed with dropout — evidence the tradeoff is a property of current methods, not the data {2020_Yang}. This tension defines the open concept [[robustness_accuracy_tradeoff]].

### The optimization landscape of adversarial training

AT is harder to optimize than clean training, and Liu et al. explain why: while the adversarial loss $L_\epsilon(\theta)$ retains first-order (gradient-Lipschitz) smoothness, its *second-order* smoothness is destroyed — subgradients change discontinuously in parameter space, with the discontinuity growing linearly in the budget $\epsilon$, so SGD has no convergence guarantee for $\epsilon > 0$ {2020_Liu}. Empirically, large $\epsilon$ early in training traps the model in suboptimal, near-constant-classifier regions (worsened by dead ReLUs that decouple the network), and high budgets sharpen the loss Hessian. Their fix — **adversarial budget scheduling / warmup**, ramping $\epsilon$ from small to large — improves both clean and robust accuracy, especially at higher learning rates {2020_Liu}. This complements Madry's capacity argument: robust optimization needs wide networks *and* careful budget control.

### Certified defenses

Empirical defenses give no guarantee; *certified* defenses prove no adversarial example exists within a radius. Cohen et al.'s **randomized smoothing** is the most scalable: the smoothed classifier $g(x) = \arg\max_c \Pr_{\eta \sim \mathcal{N}(0, \sigma^2)}[ f(x+\eta) = c ]$ provably has a constant prediction within an $L_2$ radius that grows with the margin between the top-two class probabilities and with the noise level $\sigma$ {2019_Cohen}. The base network is trained with Gaussian-noise augmentation so its boundaries are smooth on noisy inputs. $\sigma$ does double duty (training noise and certification scale) and induces its own accuracy-radius tradeoff — larger $\sigma$ certifies bigger radii but lowers center accuracy. This anchors the broader subfield [[certified_robustness]].

### Structural and feature-level robustness

Robustness can also be built into the hypothesis class. Guo et al. show weight/activation **sparsity** improves robustness for linear and DNN classifiers: sparser weights lower the gradient/Lipschitz bounds that upper-bound perturbation sensitivity, helping against FGSM (though $L_1$-sparsity does little for $L_2$/DeepFool attacks, and gains vanish once accuracy degrades) {2019_Guo}. Sulam et al. give provable robustness for supervised **sparse coding**, bounding accuracy under attack by an "encoder gap" as long as perturbations stay below a threshold {2021_Sulam}. Robustness of *attributions* (keeping saliency stable under perturbation while the prediction holds) is a related regularization target {2019_Chen_RAR} — though consulted only via a brief summary here. These connect adversarial vulnerability to the [[robust_features]] hypothesis: robust models rely on features that resist small input changes.

### Label-free and self-supervised robustness

Since AT's data appetite is large, a natural question is whether *unlabeled* data suffices. Uesato et al. answer yes: the excess adversarial risk $L_\mathrm{adv} - L_\mathrm{nat}$ is upper-bounded by a smoothness term independent of labels, so **UAT** minimizes prediction consistency between clean and adversarial inputs using unlabeled data (online targets, fixed targets from a base classifier, or both), nearly matching supervised AT and staying robust under heavy label noise {2019_Uesato_UAT}. This dovetails with a family of **adversarial contrastive learning** methods that fold PGD into self-supervised pretraining ([[contrastive_ssl]], [[self_supervised_learning]]):
- **RoCL** perturbs the contrastive anchor with an instance-wise attack on the InfoNCE loss, clustering adversarial examples with their clean sources; a "robust linear evaluation" head makes it competitive with supervised AT/TRADES {2020_Kim_RoCL}.
- **ACL** combines SimCLR with AT via clean and adversarial branches sharing weights but *not* batch-norm statistics; its Dual-Stream variant is strongest {2020_Jiang_ACL}.
- **AdvCL** adds high-frequency and pseudo-label (clustering) views so robustness survives *standard* linear finetuning rather than requiring adversarial full finetuning {2021_Fan_AdvCL}.
- **DynACL** observes that strong augmentation *helps* clean but *hurts* robust accuracy, and anneals augmentation strength over training with a matched loss reweighting {2023_Luo_DYNACL} — a direct link to [[image_augmentation_methods]].
- **RINCE** makes the contrastive loss itself robust to noisy/adversarial views via a symmetric InfoNCE, bounding mutual information under a Wasserstein rather than KL objective {2021_Chuang_RINCE}.

An earlier pretext-task line (jigsaw/rotation/inpainting pretraining plus adversarial finetuning) established that adversarial *finetuning* supplies most of the robustness gain and that different pretext tasks carry different vulnerabilities {2020_Chen_SSLRobust}. These methods together define the open node [[adversarial_contrastive_learning]].

### Augmentation, distillation, and defenses that break

Data augmentation interacts with AT in non-obvious ways: generic strong augmentations that help clean training often *fail* to help AT, while feature-preserving ones (Cutout, CutMix) do; DAJAT combines dual-strength augmentation branches with TRADES and JS-divergence consistency to get robustness efficiently {2022_Addepalli_DAJAT} — see [[image_augmentation_methods]]. **Distillation** appears on both sides: *defensive distillation* was an early defense that Carlini & Wagner showed to be gradient masking rather than real robustness {2017_Carlini}, whereas robust distillation transfers robustness from a robust teacher (via [[knowledge_distillation]]). The overarching caution: any defense reporting only non-adaptive robustness may be masking gradients, so evaluation must use strong adaptive attacks {2017_Carlini}{2019_Gowal}. A separate *purification* family — denoising a perturbed input (e.g. with a diffusion model) before classification — is the shared technique linking this node to the generative-model setting, where the same diffusion purifiers strip protective perturbations ([[defenses_against_aes_for_generative_models]]).

## References

{2015_Goodfellow} 2015_Goodfellow_Explaining_and_Harnessing_Adversarial_Examples
{2017_Carlini} 2017_Carlini_Towards_Evaluating_the_Robustness_of_Neural_Networks
{2019_Chen_RAR} 2019_Chen_Robust_Attribution_Regularization
{2019_Cohen} 2019_Cohen_Certified_Adversarial_Robustness_via_Randomized_Smoothing
{2019_Gowal} 2019_Gowal_An_Alternative_Surrogate_Loss_for_PGD-based_Adversarial_Testing
{2019_Guo} 2019_Guo_Sparse_DNNs_with_Improved_Adversarial_Robustness
{2019_Madry} 2019_Madry_Towards_Deep_Learning_Models_Resistant_to_Adversarial_Attacks
{2019_Uesato_UAT} 2019_Uesato_Are_Labels_Required_for_Improving_Adversarial_Robustness
{2019_Zhang_TRADES} 2019_Zhang_Theoretically_Principled_Trade-off_between_Robustness_and_Accuracy
{2020_Chen_SSLRobust} 2020_Chen_Adversarial_Robustness_From_Self-Supervised_Pre-Training_to_Fine-Tuning
{2020_Jiang_ACL} 2020_Jiang_Robust_Pre-Training_by_Adversarial_Contrastive_Learning
{2020_Kim_RoCL} 2020_Kim_Adversarial_Self-Supervised_Contrastive_Learning
{2020_Liu} 2020_Liu_On_the_Loss_Landscape_of_Adversarial_Training_Identifying_Challenges_and_How_to_Overcome_Them
{2020_Yang} 2020_Yang_A_Closer_Look_at_Accuracy_vs_Robustness
{2021_Chuang_RINCE} 2021_Chuang_Robust_Contrastive_Learning_Against_Noisy_Views
{2021_Fan_AdvCL} 2021_Fan_When_Does_Contrastive_Learning_Preserve_Adversarial_Robustness_from_Pretraining_to_Finetuning
{2021_Sulam} 2021_Sulam_Adversarial_Robustness_of_Supervised_Sparse_Coding
{2022_Addepalli_DAJAT} 2022_Addepalli_Efficient_and_Effective_Augmentation_Strategy_for_Adversarial_Training
{2023_Luo_DYNACL} 2023_Luo_Rethinking_the_Effect_of_Data_Augmentation_in_Adversarial_Contrastive_Learning
