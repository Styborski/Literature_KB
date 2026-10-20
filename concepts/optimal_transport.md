---
title: "Optimal Transport"
tags: ["optimal-transport", "generative-models", "diffusion-models", "flow-matching", "deep-clustering"]
last_updated: "2026/09/09"
---

## Summary

Optimal transport (OT) is the mathematics of moving one probability distribution to another at minimal cost. Its two classical forms — Monge's deterministic *maps* and Kantorovich's *couplings* — define the Wasserstein distance, and a dynamic (Benamou-Brenier) form recasts it as a least-cost flow. Entropic regularization, solved by the Sinkhorn algorithm, made OT cheap and differentiable enough for deep learning. OT is a recurring thread across this knowledge base rather than a single method: it explains diffusion latent structure and rectified flow, underlies balanced cluster assignment in self-supervised learning, and generalizes to Schrödinger bridges. This node consolidates the OT concept and its Sinkhorn tooling, delegating each application to the node that develops it.

## Related Concepts
[[score_matching]]
[[langevin_dynamics]]
[[flow_matching]]
[[diffusion_geometry]]
[[cluster_based_ssl]]
[[multimodal_ssl]]
[[normalizing_flows]]

## Open Questions

How far the OT–generative-model correspondence actually holds is unsettled. DDPM provably coincides with the Monge map only for Gaussians; the general case is inferred from experiments, not proven {2022_Khrulkov}. Rectified flow's *conditional* paths are OT displacements, but its learned *marginal* field is generally not $c$-optimal, so "diffusion learns OT" is at best partial {2022_Liu_MargOT}. Entropic OT trades exactness for a tractable, differentiable Sinkhorn iteration, but the accuracy-versus-regularization tradeoff is problem-specific. The Schrödinger-bridge generalization — entropic OT in continuous time — is barely explored in the corpus (its primary works are unread), leaving the link between stochastic bridges and OT-flow-matching under-developed here.

## Synthesis

### The transport problem: Monge, Kantorovich, Wasserstein

Optimal transport asks for the cheapest way to move mass from a source distribution $\mu$ to a target $\nu$ under a cost $c(x,y)$, usually the squared Euclidean cost $\|x-y\|^2$ {2022_Khrulkov}. The **Monge** formulation seeks a deterministic *map* $S$ pushing $\mu$ onto $\nu$ (written $\nu = S_\# \mu$) that minimizes $\int c(x, S(x))\,d\mu(x)$; the **Kantorovich** formulation relaxes this to a *coupling* — a joint distribution with the prescribed marginals — which always exists and is a linear program, whereas a Monge map need not exist {2022_Khrulkov}{2022_Liu_MargOT}. The square root of the minimized quadratic cost is the **Wasserstein-2 distance** $W_2(\mu,\nu)$, a metric on probability measures {2022_Khrulkov}. A third, **dynamic** (Benamou-Brenier) view recasts OT as finding a continuous-time flow that carries $\mu$ to $\nu$ with least kinetic cost, so the optimal path between two points is a straight-line, constant-speed geodesic — the form that ties OT to continuous normalizing flows and diffusion {2022_Liu_MargOT}{2022_Khrulkov}. (The foundational OT texts are not in the corpus; these formulations are drawn from the diffusion papers that use them.)

### Entropic regularization and the Sinkhorn algorithm

Exact OT is expensive — the Kantorovich linear program scales poorly — which long limited its use in learning. The enabling trick is **entropic regularization**: subtracting an entropy term $\lambda\, h(T)$ from the transport-plan objective $\langle T, C\rangle$ makes the problem strictly convex and solvable by the **Sinkhorn algorithm**, a fast, differentiable iteration that alternately rescales the rows and columns of $\exp(-C/\lambda)$ (with $C$ the cost matrix) to match the target marginals, giving a plan of the form $T^* = \mathrm{diag}(u)\,\exp(-C/\lambda)\,\mathrm{diag}(v)$ {2023_Chen_PLOT}. This entropic-OT / Sinkhorn-distance construction is due to Cuturi, and the underlying matrix-scaling iteration to Sinkhorn-Knopp — both works are unread in the corpus and named here rather than synthesized; a debiased variant, the **Sinkhorn divergence**, was introduced by Genevay et al. as a generative training loss interpolating between OT and maximum-mean-discrepancy (also unread). The regularizer $\lambda$ trades transport accuracy for speed and smoothness, and it is Sinkhorn's *differentiability* that lets OT sit inside end-to-end training — the property the SSL and vision-language uses below depend on {2023_Chen_PLOT}.

### OT and diffusion models

OT gives a precise account of what diffusion latents *are*. Khrulkov et al. show that the DDPM (variance-preserving) encoding map — the probability-flow ODE carrying data to noise — coincides with the **Monge optimal-transport map** onto the Gaussian prior: exactly for multivariate Gaussians, and to machine precision empirically on real image models {2022_Khrulkov}. The latent code is therefore not arbitrary but carries transportable structure (pose, texture, colour), so the same noise decodes to similar images across models and datasets — behaviour GANs lack {2022_Khrulkov}. The reverse process itself reads as a **Wasserstein gradient flow** in the space of measures: the density trajectory is the gradient flow of a functional, and the PF-ODE realizes the local OT map between consecutive densities {2022_Khrulkov} — the same measure-space gradient flow that overdamped [[langevin_dynamics]] induces through its Fokker–Planck equation (the score-based-model form of this reading is developed under [[score_matching]]). Rectified flow makes the connection constructive — its straight-line conditional paths are the OT displacement between paired endpoints, and its **reflow** operation provably lowers transport cost while preserving marginals, at rate $O(1/k)$ over rounds {2022_Liu_MargOT}. The essential caveat, developed under [[flow_matching]], is that this OT property is only partial: reflow reduces cost for *every* convex cost without solving any particular one, so a straight coupling is necessary but *not sufficient* for $c$-optimality, and the learned marginal velocity field is generally not an OT solution {2022_Liu_MargOT}. These threads are the OT face of [[diffusion_geometry]].

### OT and self-supervised learning

The same Sinkhorn machinery solves a different problem in self-supervised learning: **balanced assignment**. Cluster-based methods (SeLa, SwAV) cast pseudo-labelling as transporting equal mass to each cluster — an OT problem with an equipartition constraint, solved by a few Sinkhorn iterations — which blocks the degenerate solution of collapsing all samples into one cluster; this is developed under [[cluster_based_ssl]]. In vision-language, **PLOT** uses OT to align *multiple* learnable prompts with the set of local image features of a CLIP model: one prompt underfits a class and naively adding prompts makes them collapse together, so PLOT alternates an inner Sinkhorn step (fix features, solve the prompt-to-feature transport plan) with an outer step (fix the plan, learn the prompts), then reads the OT distance as the class score for few-shot classification {2023_Chen_PLOT}. Both uses exploit OT the same way — as a soft, balanced matching between two sets that avoids the collapse a naive hard assignment would suffer — tying the tool to [[multimodal_ssl]].

### Schrödinger bridges: the stochastic generalization

Adding entropy to *dynamic* OT yields the **Schrödinger bridge** — the most likely stochastic evolution between two distributions — the continuous-time, noisy generalization of OT and the natural meeting point of OT and diffusion. Denoising Diffusion Bridge Models make the link explicit: in the noiseless limit a bridge's drift reduces exactly to the OT / rectified-flow velocity, and to ordinary diffusion when one endpoint is Gaussian (developed under [[flow_matching]]). Dedicated bridge methods extend diffusion to transport between *arbitrary* paired distributions — I2SB for image-to-image translation, and DSBM's iterative Markovian fitting — but both are unread in the corpus and are named here as the frontier of OT-based generative modelling rather than developed. This is the least-covered branch of OT in the knowledge base.

### Positioning

Optimal transport is connective tissue rather than a standalone method: it is the shared language of "move one distribution to another at minimal cost," which is exactly what a generative model does (noise to data), what cluster-based SSL does (features to balanced clusters), and what domain translation does (one image set to another). That is why OT surfaces independently across the subgraphs — as the geometry of diffusion latents ([[diffusion_geometry]]), the objective behind rectified flow ([[flow_matching]]), the measure-space reading of score-based generation ([[score_matching]]), and the assignment step in clustering ([[cluster_based_ssl]]). This node holds the shared concept and its computational tool (Sinkhorn); the deep application detail lives in those nodes. The chief honesty caveat is that the OT-theory primaries (Cuturi, Genevay) and the Schrödinger-bridge works (I2SB, DSBM) are unread, so the foundations here are carried through the applied papers that put OT to work.

## References

{2022_Khrulkov} 2022_Khrulkov_Understanding_DDPM_Latent_Codes_Through_Optimal_Transport
{2022_Liu_MargOT} 2022_Liu_Rectified_Flow_A_Marginal_Preserving_Approach_to_Optimal_Transport
{2023_Chen_PLOT} 2023_Chen_PLOT_Prompt_Learning_with_Optimal_Transport_for_Vision-Language_Models
