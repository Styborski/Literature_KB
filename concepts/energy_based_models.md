---
title: "Energy-Based Models"
tags: ["energy-based-models", "generative-models", "score-based-models", "controllable-generation", "language-models"]
last_updated: "2026/09/06"
---

## Summary

An energy-based model (EBM) represents a distribution by an unnormalized scalar energy, $p(x) = \exp(-E(x))/Z$, learning the shape of the density and leaving the normalizing constant $Z$ implicit. This makes EBMs the most flexible generative family — any scalar network is a valid energy — but the intractable $Z$ curses both training (no direct maximum likelihood) and sampling (requires MCMC). The family is therefore defined by how it *evades* $Z$: noise-contrastive estimation, score matching, and Langevin/MCMC sampling. In this knowledge base EBMs are less a standalone method than a connective idea: score-based diffusion learns an energy's gradient, guidance adds an energy to the score, and constrained text decoding samples an energy by Langevin dynamics.

## Related Concepts
[[generative_models]]
[[score_matching]]
[[langevin_dynamics]]
[[noise_contrastive_estimation]]
[[diffusion_models]]
[[diffusion_guidance]]
[[diffusion_language_models]]
[[normalizing_flows]]

## Open Questions

The defining difficulty is unresolved by construction: training and sampling both require approximating the intractable partition function, and every method is a different compromise — MCMC-based maximum likelihood is expensive and biased by short chains, score matching sidesteps $Z$ but is unreliable at low noise ([[score_matching]]), and noise-contrastive estimation depends on a well-matched noise distribution {2010_Gutmann}. Whether an *explicit* energy buys anything over a score/diffusion model that only ever needs the energy's gradient is genuinely open — the KB's diffusion nodes suggest the gradient is usually enough. Applying EBMs to discrete data (text) needs a continuous relaxation to make gradients well-defined, and the accuracy of that relaxation is a live question {2023_Qin_COLD}. (The corpus lacks the foundational EBM-training literature, so these are framed from the adjacent works that use EBM machinery, not from the primaries.)

## Synthesis

### What an energy-based model is

An EBM assigns every configuration $x$ a scalar **energy** $E(x)$ and defines a probability through the Boltzmann/Gibbs form $p(x) = \exp(-E(x))/Z$, with partition function $Z = \int \exp(-E(x))\,dx$ {2023_Qin_COLD}. Low energy means high probability. Because $E$ can be *any* scalar-valued function — no invertibility, no normalized output, no prescribed latent structure — EBMs are the most flexible generative family and the most direct expression of the unnormalized-density problem that organizes [[generative_models]]: a flexible density $p_\theta(x) = \exp(-f_\theta(x))/Z_\theta$ always carries an intractable normalizer $Z_\theta$, and each generative family is a different bargain with it {2010_Gutmann}. The EBM "bargain" is to refuse the bargain — model the unnormalized shape directly and pay the cost at training and sampling time. This is the opposite corner of the design space from [[normalizing_flows]], which buys an *exact* normalized likelihood by constraining the model to an invertible map with a tractable Jacobian.

### The partition function: a double curse

The intractable $Z$ blocks the two things one wants from a generative model. **Training** by maximum likelihood is unavailable because $\log p(x) = -E(x) - \log Z$, and the gradient $\nabla_\theta \log Z$ is itself an expectation over the *model's own* distribution — requiring samples one cannot yet draw {2010_Gutmann}{2023_Qin_COLD}. **Sampling** is hard because there is no direct sampler for an unnormalized density; one must run a Markov chain to equilibrium. The whole family is organized around evading $Z$, and this knowledge base contains three of the evasions.

### Training without the partition function

**Noise-contrastive estimation (NCE)** turns density estimation into classification: fit the model by learning to distinguish real data from samples of an artificial noise distribution with a logistic (BCE-like) objective {2010_Gutmann}. Gutmann & Hyvärinen show this recovers the correct model — the objective is maximized when the fitted density equals the data density, it is consistent as the sample size grows, and it lets the normalizer be treated as a *free parameter* that the objective drives to the self-normalizing value (the estimate integrates to one automatically) {2010_Gutmann}. The catch is that the noise distribution must be chosen with care — nonzero everywhere and not too far from the data — or the classification task is too easy to be informative {2010_Gutmann}. This NCE principle is the planned focus of [[noise_contrastive_estimation]] and reappears, in InfoNCE form, across contrastive self-supervised learning. **Score matching** takes the complementary route of never touching $Z$ at all: the score $\nabla_x \log p(x) = -\nabla_x E(x)$ is independent of the normalizer, so fitting the score fits the energy up to a constant — which is exactly why denoising score matching underlies diffusion, and the precise sense in which a diffusion model *is* an EBM whose energy is only ever accessed through its gradient (developed under [[score_matching]]). (The classical MCMC route — maximum likelihood via contrastive divergence on Boltzmann machines and their deep successors — is absent from the corpus and named here only as background.)

### Sampling by Langevin dynamics

Given an energy, samples are drawn by **Langevin dynamics** ([[langevin_dynamics]]), a gradient-based MCMC that needs only $\nabla_x E$: iterate $x \leftarrow x - \tfrac{\eta}{2}\nabla_x E(x) + \sqrt{\eta}\,z$ with $z \sim \mathcal{N}(0,I)$, descending the energy while injecting noise to keep exploring {2023_Qin_COLD}. Annealing the noise from large to small — the same **annealed Langevin** scheme score-based models use — lets the chain cross between well-separated modes before settling into one, and is the sampling bridge from EBMs to diffusion ([[score_matching]], [[diffusion_models]]). This shared sampler is why the score-based and energy-based readings of generation sit so close together: both ultimately move a particle along $-\nabla_x E = \nabla_x \log p$, one framing the target as an energy and the other as a score.

### The energy view inside the diffusion subgraph

Although the knowledge base has no standalone deep-EBM method node beyond this one, the energy lens recurs across the diffusion subgraph. **Score-based diffusion** learns the gradient of an implicit, noise-conditional energy without ever forming $Z$ ([[score_matching]]). **Guidance** is explicitly energy arithmetic: classifier and classifier-free guidance add $\nabla_x g(x)$ for an energy $g$ scoring agreement with a condition, so a classifier serves as a *negative* energy and self-guidance reads the energy off the network's own attention maps and activations (developed under [[diffusion_guidance]]). And the geometric contrast that separates the two families is that an EBM's dynamics flow *toward* the data distribution — needing an explicit data energy — whereas the score-based reverse process flows *away* from the Gaussian prior and never handles the data density directly, which is precisely why score matching is tractable where EBM maximum likelihood is not ([[score_matching]]).

### Applied EBMs: constrained text decoding

The corpus's one worked modern EBM is **COLD**, which casts controllable text generation as sampling from an energy $E(y) = -\sum_i \lambda_i f_i(y)$ whose terms are task constraints — fluency under a reference language model, keyword coverage, future-token prediction, n-gram similarity to a reference — so decoding becomes drawing $y \sim \exp(-E(y))/Z$ with *no* task-specific fine-tuning {2023_Qin_COLD}. Because text is discrete and gradients are undefined on tokens, COLD runs Langevin dynamics on **soft** sequences — vectors of vocabulary logits passed through a temperature softmax — annealing the noise across sampling steps before rounding to discrete tokens, and it downselects the vocabulary with a reference model to keep outputs fluent {2023_Qin_COLD}. It is a compact illustration of the entire family — an intractable Boltzmann distribution made samplable by gradient MCMC — and the energy-based instance of controllable generation in [[diffusion_language_models]], paralleling how guidance steers continuous-space diffusion.

### Positioning and limitations

EBMs occupy the flexible, hard-to-use end of the generative spectrum: maximal modeling freedom bought at the price of intractable training and slow MCMC sampling, so they are rarely used raw for high-fidelity synthesis and appear here mostly as the theoretical substrate that better-behaved relatives are built on. Their enduring contribution to this knowledge base is the framing they supply — the unnormalized-density problem, the score-as-negative-energy-gradient identity, and Langevin sampling — which recurs, usually implicitly, across [[score_matching]], [[diffusion_models]], and [[diffusion_guidance]]. Coverage boundary: the foundational EBM methodology (Boltzmann and restricted Boltzmann machines, contrastive divergence, and modern deep EBMs such as implicit-generation and joint energy models) is absent from the corpus, and the latent-variable-EBM program is present but unread, so this node is an honest seed anchored on the unnormalized-density principle {2010_Gutmann} and one applied EBM {2023_Qin_COLD}, with the rest carried by links to the diffusion nodes that put energy thinking to work.

## References

{2010_Gutmann} 2010_Gutmann_Noise-contrastive_estimation_A_new_estimation_principle_for_unnormalized_statistical_models
{2023_Qin_COLD} 2023_Qin_COLD_Decoding_Energy-based_Constrained_Text_Generation_with_Langevin_Dynamics
