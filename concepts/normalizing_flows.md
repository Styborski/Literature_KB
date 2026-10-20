---
title: "Normalizing Flows"
tags: ["normalizing-flows", "generative-models", "continuous-normalizing-flows", "variational-inference", "latent-variable-models"]
last_updated: "2026/09/06"
---

## Summary

A normalizing flow transforms a simple base density into a complex one through an invertible, differentiable map, using the change-of-variables formula to yield *exact* likelihoods — the distinguishing bargain among generative families. Expressiveness is bounded by the need for a tractable Jacobian determinant, so flows are built by composing many cheap-log-det layers (planar/radial, coupling, autoregressive) or, in the continuous limit, as a neural ODE whose log-density evolves by a simple trace (a continuous normalizing flow). Introduced to enrich variational-inference posteriors, flows matter most in this knowledge base as the conceptual root of the diffusion subgraph: a diffusion model's probability-flow ODE *is* a continuous normalizing flow, and flow matching is the simulation-free way to train one.

## Related Concepts
[[generative_models]]
[[flow_matching]]
[[diffusion_models]]
[[score_matching]]
[[variational_autoencoders]]
[[optimal_transport]]
[[neural_odes]]
[[variational_inference]]
[[manifold_hypothesis]]

## Open Questions

Whether exact-likelihood flows retain a distinct niche now that diffusion and flow matching dominate perceptual synthesis is unclear: their advantage is tractable density *evaluation*, but they trail on sample fidelity, and the corpus lacks the modern coupling/autoregressive-flow works that most narrowed that gap. The tractable-Jacobian-versus-expressiveness trade-off has no closed resolution — every layer buys a cheap log-determinant by restricting its Jacobian structure, and how much depth compensates is empirical {2016_Rezende_Flows}{2019_Chen_NeuralODE}. The diffeomorphic, dimension- and topology-preserving nature of flows sits in tension with the manifold hypothesis, leaving open how a flow should model data that concentrates on a lower-dimensional set ([[manifold_hypothesis]]). And for continuous flows the number of adaptive ODE-solver evaluations grows with model complexity during training {2019_Chen_NeuralODE}, an efficiency question inherited by the samplers that descend from them.

## Synthesis

### The exact-likelihood bargain: change of variables

A normalizing flow builds a complex density by pushing a simple base density $q_0$ (typically $\mathcal{N}(0,I)$) through an invertible, differentiable map $f$. The **change-of-variables** formula gives the transformed density exactly: for $z' = f(z)$, $q(z') = q(z)\,\lvert\det(\partial f/\partial z)\rvert^{-1}$, and after composing $K$ maps $z_K = f_K \circ \cdots \circ f_1(z_0)$,
$$\log q_K(z_K) = \log q_0(z_0) - \sum_{k=1}^{K} \log\left\lvert\det\frac{\partial f_k}{\partial z_{k-1}}\right\rvert,$$
the successive densities $q_k$ forming the "normalizing flow" {2016_Rezende_Flows}. Unlike a VAE (which optimizes a *bound* on the likelihood) or a GAN (which abandons the likelihood entirely), a flow delivers *exact* likelihoods, and supports both sampling — run $f$ forward from $z_0 \sim q_0$ — and density evaluation — run $f^{-1}$ and accumulate the log-determinants {2016_Rezende_Flows}{2019_Chen_NeuralODE}. This is the family's defining bargain and fixes its place among [[generative_models]] as the exact-likelihood corner of the design space. The price is structural: $f$ must be invertible with a tractable Jacobian log-determinant, and being a diffeomorphism it preserves dimension and topology.

### Finite-composition flows and the Jacobian bottleneck

The engineering problem is that a general Jacobian determinant costs $O(d^3)$, so much of flow design is a search for expressive maps with cheap log-determinants {2016_Rezende_Flows}. Rezende & Mohamed's original **planar** and **radial** flows apply rank-one perturbations whose log-determinants are $O(d)$: a planar flow $f(z) = z + u\,h(w^\top z + b)$ contracts or expands the density perpendicular to a hyperplane, with log-determinant $\log\lvert 1 + u^\top \psi(z)\rvert$ (where $\psi = h'$), while a radial flow does the same around a reference point {2016_Rezende_Flows}. Stacking these (up to $K=32$ in their experiments) approximates complex 2D densities, improving as $K$ grows {2016_Rezende_Flows}. They contrast this with **NICE**, a *volume-preserving* flow whose Jacobian has unit determinant (zero log-determinant), which is cheap but forfeits the density-reshaping term; planar flows reach the same asymptotic quality with far fewer parameters and outperform NICE at matched depth {2016_Rezende_Flows}. (Coverage boundary: the mainstream of practical image flows — coupling-layer flows such as RealNVP and Glow, and autoregressive flows such as IAF and MAF, which solve the same tractable-Jacobian problem with triangular Jacobians — are absent from the corpus and are named here only as background, not synthesized from source.)

### Normalizing flows for variational inference

Normalizing flows entered deep learning to fix the *posterior* in variational inference, not as image generators {2016_Rezende_Flows}. A VAE's approximate posterior $q_\phi(z|x)$ is usually a diagonal Gaussian — too simple to match the true posterior, so it underestimates variance and biases estimates {2016_Rezende_Flows}. Rezende & Mohamed replace it with a flow: draw $z_0$ from a Gaussian, push it through $K$ planar/radial maps, and write the evidence lower bound (negative free energy) as an expectation under the *simple* $q_0$ via the law of the unconscious statistician, so the flow's log-determinants enter the bound and the model trains by the same reparameterized Monte-Carlo gradients as ordinary variational inference {2016_Rezende_Flows}. On binarized MNIST, flow posteriors beat both NICE and Hamiltonian variational inference, improving with $K$ {2016_Rezende_Flows}. This is the concrete tie to [[variational_autoencoders]] — a richer posterior family — and to [[variational_inference]] generally. The paper also introduces the **infinitesimal-flow** limit ($K \to \infty$), in which the density evolves by a PDE realized as a Langevin SDE or Hamiltonian dynamics {2016_Rezende_Flows} — the discrete-to-continuous bridge that the next section makes literal.

### Continuous normalizing flows: the neural-ODE realization

Taking that infinitesimal limit literally replaces the discrete stack with an ODE $dz/dt = f(z,t)$ whose vector field is a neural network — a **neural ODE**, the continuous-depth limit of a residual network $h_{t+1} = h_t + f(h_t)$ {2019_Chen_NeuralODE}. The payoff for flows is that the change-of-variables *determinant* collapses to a **trace**:
$$\frac{d\log p(z(t))}{dt} = -\operatorname{tr}\!\left(\frac{\partial f}{\partial z}\right),$$
an $O(d)$ quantity that stays cheap even for wide networks because the trace is linear across hidden units {2019_Chen_NeuralODE}. Sampling and density evaluation both reduce to calling an ODE solver; gradients flow through it by the memory-efficient **adjoint sensitivity method** (a second, reverse-time ODE) rather than by storing the solver's internals; and the reverse map costs about as much as the forward one, so a **continuous normalizing flow (CNF)** trains for density estimation by direct maximum likelihood and then reverses to generate {2019_Chen_NeuralODE}. Chen et al. show CNFs achieve lower density-matching loss than discrete planar flows {2019_Chen_NeuralODE}. The broader neural-ODE substrate — adaptive-cost forward solves, the adjoint method, latent-ODE time-series models — is developed under [[neural_odes]].

### The bridge to diffusion and flow matching

The CNF is the hinge that binds this node to the rest of the diffusion subgraph. A diffusion model's **probability-flow ODE** — the deterministic map carrying data to noise with the same marginals as the reverse SDE — *is* a continuous normalizing flow, which is what endows diffusion models with exact likelihoods and a deterministic, invertible data-to-noise latent map (developed under [[diffusion_models]] and [[score_matching]]). The very cost that kept CNFs marginal — maximum-likelihood training requires simulating the ODE in the loop — is what **flow matching** removes, regressing the same velocity field onto a prescribed conditional probability path instead of maximizing likelihood, so a CNF becomes as cheap to train as a diffusion model ([[flow_matching]]). Read this way, diffusion and flow matching are the scalable descendants of the CNF: they keep its deterministic, invertible, exact-likelihood transport while discarding its simulation cost. The dynamic optimal-transport reading — a least-kinetic-cost flow between two densities (Benamou–Brenier) — is the measure-space form of the same object ([[optimal_transport]]).

### Limitations and positioning

Two structural constraints bound the family. Because a flow is a diffeomorphism between equal-dimensional spaces, it **preserves dimension and topology**: it cannot natively concentrate mass on a lower-dimensional data manifold (the [[manifold_hypothesis]] setting), forcing a continuous density over the full ambient space — one reason pure flow samples historically trailed GANs and diffusion on image fidelity, a limitation the diffusion families sidestep precisely by adding noise off-manifold. And the tractable-Jacobian requirement caps the expressiveness of any single layer, repaid only by depth {2016_Rezende_Flows}{2019_Chen_NeuralODE}. The family's lasting contribution to this knowledge base is therefore conceptual rather than architectural: the change-of-variables / exact-likelihood machinery and the CNF formulation that diffusion and flow matching now carry at scale. That is why normalizing flows sit under [[generative_models]] as a foundational corner of the design space even though pure flows are no longer the state of the art in perceptual synthesis, and why the node's densest links run to the flow-and-score side of the diffusion subgraph rather than to any standalone flow architecture.

## References

{2016_Rezende_Flows} 2016_Rezende_Variational_Inference_with_Normalizing_Flows
{2019_Chen_NeuralODE} 2019_Chen_Neural_Ordinary_Differential_Equations
