---
title: "Non-Contrastive Self-Supervised Learning"
tags: ["self-supervised-learning", "non-contrastive-learning", "siamese-networks", "collapse-avoidance", "byol", "simsiam"]
last_updated: "2026/08/21"
---

## Summary

Non-contrastive SSL learns visual representations from positive pairs alone, without negative samples, using asymmetric Siamese networks. The canonical methods — BYOL and SimSiam — attach a predictor to one "online" branch, stop the gradient on the "target" branch, and (in BYOL) update the target as a momentum average of the online network. Remarkably, these avoid the collapse (all inputs mapped to one point) that negatives were thought necessary to prevent. Explaining *why* they do not collapse has produced an unusually deep dynamics literature that ultimately links non-contrastive learning to [[feature_decorrelation_ssl]], [[teacher_student_ssl]], and [[contrastive_ssl]]. "Non-contrastive" is used broadly (any negative-free method) and narrowly (the BYOL/SimSiam distillation branch); this node centers the narrow sense.

## Related Concepts
[[self_supervised_learning]]
[[contrastive_ssl]]
[[feature_decorrelation_ssl]]
[[cluster_based_ssl]]
[[teacher_student_ssl]]
[[multimodal_ssl]]
[[dimensional_collapse]]
[[predictor_network]]
[[stop_gradient]]
[[momentum_encoder]]
[[siamese_networks]]
[[image_augmentation_methods]]
[[adversarial_contrastive_learning]]

## Open Questions

There is still no single agreed mechanism for collapse avoidance. Candidate explanations — batch-norm as implicit negatives {2020_Fetterman}, predictor-as-expectation {2020_Grill}, eigenspace alignment {2021_Tian_DirectPred}, implicit variance regularization {2022_Halvagal}, spectral filtering {2023_Zhuo}, orthogonal projection {2023_Richemond} — are partly complementary and partly contradictory, and most are proven only for linear encoders without output normalization, whereas the empirically important regime has nonlinearity and batch norm {2023_Wen}. Whether asymmetry (predictor/stop-grad/EMA) is fundamentally necessary, or merely one convenient route to the same variance/decorrelation effect achieved explicitly by [[feature_decorrelation_ssl]], is unresolved {2022_Pokle}{2023_Garrido}. When and why partial dimensional collapse hurts downstream performance is only partially characterized {2022_Li_Collapse}.

## Synthesis

### The two canonical methods

BYOL {2020_Grill} uses an online network (encoder + projector + predictor) trained to predict the projection produced by a momentum-averaged *target* network (encoder + projector) on a different augmented view, with a stop-gradient on the target and a squared-error/cosine loss on $L_2$-normalized embeddings. It requires no negatives and outperformed contrastive peers on ImageNet transfer while being more robust to removing augmentations (contrastive methods lean on color augmentation to avoid the color-histogram shortcut; BYOL does not). SimSiam {2020_Chen_SimSiam} strips out the momentum encoder entirely — the target is just the online network with a stop-gradient — and shows the *predictor* and *stop-gradient* are the crucial pieces. Both original papers advanced hypotheses that later work substantially revised.

### The collapse-avoidance debate

The original explanations were quickly contested. BYOL's authors suggested the momentum encoder plus predictor prevent collapse and hedged that a momentum of 0 collapses {2020_Grill}; SimSiam then achieved stable training with no momentum at all {2020_Chen_SimSiam}. A widely-circulated blog argued batch normalization supplies an *implicit* contrastive term (the batch mean acting as a negative) {2020_Fetterman}; BYOL's authors rebutted this by matching performance with group norm + weight standardization, which use no batch statistics, arguing BN instead compensates for poor initialization {2020_Richemond}. SimSiam's own "predictor approximates an expectation" account was undercut by {2022_Zhang_SimSiam}, which built the expectation explicitly ("mirror SimSiam"), observed collapse, and reframed the mechanism via a de-centering + de-correlation decomposition of the extra gradient.

### Learning-dynamics theory

The most productive line analyzes linear Siamese networks. DirectPred {2021_Tian_DirectPred} derived the coupled dynamics of predictor and encoder, showed the predictor's eigenspace aligns with the projector-output correlation matrix, and used this to *set* the predictor directly (no gradient descent), even beating BYOL. This spawned a family of "Direct" methods: DirectSet/DirectCopy generalize the predictor as a power of the correlation matrix and give a fixed-point-stability account separating invariant from nuisance feature subspaces {2022_Wang_DirectSet}. IsoLoss showed predictor + stop-grad implicitly regularize the correlation eigenvalues toward 1 — an *implicit* version of the explicit variance term in VICReg/Barlow Twins — and used this to replace EMA with an isotropic loss {2022_Halvagal}. The Edge-of-Orthogonality analysis proved the optimal linear predictor is a projection matrix, cast BYOL's inner loop as k-PCA / Riemannian gradient descent on the Stiefel manifold, and derived several new closed-form predictors that beat DirectCopy {2023_Richemond}. The Rank Differential Mechanism unifies these: asymmetry (predictor, EMA, or DINO's centering/sharpening and SwAV's Sinkhorn) acts as a *spectral filter* giving the target encoder higher effective rank than the online encoder, which pulls the online encoder away from dimensional collapse; this reframes stop-gradient as forcing the online (not target) encoder to improve its rank {2023_Zhuo}. A key caveat, stressed by {2023_Wen}: this linear, normalization-free theory predicts a symmetric converged predictor, but with nonlinearity and batch norm the predictor passes through a non-symmetric, non-diagonal intermediate phase (their four stages: strong-feature learning, substitution, acceleration, convergence) that is itself essential to learning.

### Asymmetry, collapse types, and empirical structure

Some source of asymmetry is necessary: the online/"source" branch should carry higher encoding variance than the target branch, and interventions that raise source variance or lower target variance (multi-crop, scale-mix, mean-encoding, asymmetric augmentation) consistently improve results {2022_Wang_Asymmetry}. It is important to distinguish *complete/total collapse* (all inputs to one point) from *dimensional/partial collapse* (embeddings occupy a low-rank subspace): asymmetry is unnecessary to prevent complete collapse but necessary to prevent dimensional collapse {2023_Zhuo}. Empirically, SimSiam with small encoders on large datasets does suffer partial dimensional collapse, BYOL's EMA is *not* redundant (it collapses less than SimSiam), and downstream kNN accuracy is predictable from a linear combination of the cosine loss and a collapse (explained-variance-AUC) metric without fine-tuning {2022_Li_Collapse}. A gradient-level unification (UniGrad) shows SimSiam, BYOL, SimCLR, MoCo, Barlow Twins and VICReg all reduce to a positive-gradient + negative-gradient + balancing-scalar template {2022_Tao}.

### Convergence with other families, and a dissent

Multiple analyses converge on the same conclusion: asymmetric non-contrastive learning implicitly performs the feature decorrelation / whitening that [[feature_decorrelation_ssl]] does explicitly {2021_Tian_DirectPred}{2022_Halvagal}{2023_Zhuo}, and BYOL/SimSiam connect to the sample- vs dimension-contrastive duality {2023_Garrido}. The EMA target is the same [[momentum_encoder]] used by MoCo and by [[teacher_student_ssl]] methods (DINO, data2vec), so BYOL sits at the intersection of the two families. A notable dissent: on a simple sparse-signal generative model, contrastive objectives provably recover the ground-truth signal while non-contrastive ones admit infinitely many non-collapsed but useless optima, converging only under warm-starts or a predictor {2022_Pokle} — a caution that empirical success may depend on network/initialization inductive biases the clean theory omits. See [[dimensional_collapse]], [[predictor_network]], [[stop_gradient]], [[momentum_encoder]].

## References

{2020_Chen_SimSiam} 2020_Chen_Exploring_Simple_Siamese_Representation_Learning
{2020_Fetterman} 2020_Fetterman_Understanding_Self-Supervised_and_Contrastive_Learning_with_Bootstrap_Your_Own_Latent_BYOL
{2020_Grill} 2020_Grill_Bootstrap_Your_Own_Latent_A_New_Approach_to_Self-Supervised_Learning
{2020_Richemond} 2020_Richemond_BYOL_Works_Even_Without_Batch_Statistics
{2021_Tian_DirectPred} 2021_Tian_Understanding_Self-Supervised_Learning_Dynamics_without_Contrastive_Pairs
{2022_Halvagal} 2022_Halvagal_Predictor_Networks_and_Stop-Grads_Provide_Implicit_Variance_Regularization_in_BYOL-SimSiam
{2022_Li_Collapse} 2022_Li_Understanding_Collapse_in_Non-Contrastive_Siamese_Representation_Learning
{2022_Pokle} 2022_Pokle_Contrasting_the_Landscape_of_Contrastive_and_Non-Contrastive_Learning
{2022_Tao} 2022_Tao_Exploring_the_Equivalence_of_Siamese_Self-Supervised_Learning_via_a_Unified_Gradient_Framework
{2022_Wang_Asymmetry} 2022_Wang_On_the_Importance_of_Asymmetry_for_Siamese_Representation_Learning
{2022_Wang_DirectSet} 2022_Wang_Towards_Demystifying_Representation_Learning_with_Non-contrastive_Self-supervision
{2022_Zhang_SimSiam} 2022_Zhang_How_Does_SimSiam_Avoid_Collapse_Without_Negative_Samples_A_Unified_Understanding_With_Self-Supervised_Contrastive_Learning
{2023_Garrido} 2023_Garrido_On_the_Duality_between_Contrastive_and_Non-Contrastive_Self-Supervised_Learning
{2023_Richemond} 2023_Richemond_The_Edge_of_Orthogonality_A_Simple_View_of_What_Makes_BYOL_Tick
{2023_Wen} 2023_Wen_The_Mechanism_of_Prediction_Head_in_Non-contrastive_Self-supervised_Learning
{2023_Zhuo} 2023_Zhuo_Towards_a_Unified_Theoretical_Understanding_of_Non-Contrastive_Learning_via_Rank_Differential_Mechanism
