---
title: "Langevin Dynamics"
tags: ["langevin-dynamics", "mcmc", "score-based-models", "energy-based-models", "generative-models"]
last_updated: "2026/09/21"
---

## Summary

Langevin dynamics is a stochastic process — originally a physics model of a particle jostled by fluid molecules (Brownian motion) — that has become a foundational sampling method in generative modeling. In its overdamped, machine-learning form it is gradient ascent on a log-density with calibrated Gaussian noise added at every step, and its stationary distribution is exactly the target $p(x)$. Crucially it requires only the score $\nabla_x \log p(x)$, never the intractable normalizing constant, which is why it samples [[energy_based_models]] and score-based [[diffusion_models]] ([[score_matching]]). Its weaknesses — slow mixing, mode-trapping, and finite-step discretization bias — motivate Metropolis correction (MALA) and noise annealing.

## Related Concepts
[[score_matching]]
[[energy_based_models]]
[[diffusion_samplers]]
[[diffusion_models]]
[[optimal_transport]]

## Open Questions

The unadjusted algorithm is only asymptotically correct: it samples $p(x)$ exactly only as the step size $\epsilon \to 0$ and the horizon $T \to \infty$, and at any finite $\epsilon$ its stationary distribution is biased {2022_Friedman}. Metropolis-adjusted Langevin (MALA) removes this bias for any step size but requires evaluating the energy $f(x)$, not just its gradient — unavailable to score/diffusion models that learn only $\nabla f$ {2022_Friedman}{2026_Wikipedia_LangevinDynamics}. Mixing is the deeper problem: one step size cannot both explore fast and resolve fine structure, and chains rarely hop between well-separated modes; annealing helps but "is harder to justify theoretically," shifting the burden onto schedule design {2022_Friedman}. And the algorithm remains slow, iteration-wise, in high dimensions {2022_Friedman}.

## Synthesis

### The physical origin: Brownian motion

The concept begins with **Brownian motion** — the ceaseless, jittery movement of a small particle suspended in a fluid, first documented by Robert Brown in 1827 and explained by Einstein in 1905 as the result of countless collisions with individual fluid molecules {2026_Wikipedia_BrownianMotion}. Einstein's account has two halves that recur throughout the modern sampling story. First, the probability density of the particle's position obeys a **diffusion equation** $\partial_t \rho = D\,\partial_x^2 \rho$, whose solution spreads as a Gaussian of variance $2Dt$, so the mean-squared displacement grows *linearly* in time, $\mathbb{E}[x^2] = 2Dt$ — displacement scales as $\sqrt{t}$, the signature of a random walk rather than ballistic motion {2026_Wikipedia_BrownianMotion}. Second, the **Einstein relation** $D = \mu k_B T$ ties the diffusion constant to mobility and temperature, an early instance of the fluctuation–dissipation link below {2026_Wikipedia_BrownianMotion}. The rigorous mathematical object underneath — a continuous process with independent Gaussian increments $W_t - W_s \sim \mathcal{N}(0, t-s)$ — is the **Wiener process**, and it is the noise source $dW_t$ in every SDE that follows {2026_Wikipedia_BrownianMotion}{2026_Wikipedia_LangevinDynamics}.

### The Langevin equation: force, friction, and fluctuation

Paul Langevin (1908) rewrote Brownian motion as a single stochastic differential equation for the particle rather than for its density {2026_Wikipedia_LangevinEquation}. The original form balances a viscous drag against a random kick,
$$m\frac{d\mathbf{v}}{dt} = -\lambda \mathbf{v} + \boldsymbol{\eta}(t),$$
where $\boldsymbol{\eta}(t)$ is Gaussian white noise with $\langle \eta_i(t)\,\eta_j(t')\rangle = 2\lambda k_B T\,\delta_{ij}\,\delta(t-t')$ {2026_Wikipedia_LangevinEquation}. Two features carry over to machine learning. The first is that a Langevin equation always has three ingredients — a deterministic **force** from a potential, a **friction** (damping) term that dissipates energy, and a **fluctuation** (noise) term that injects it {2026_Wikipedia_LangevinDynamics}. The second is the **fluctuation–dissipation theorem**: the very same coefficient $\lambda$ sets both the drag and the *variance* of the noise, so dissipation and fluctuation are locked together {2026_Wikipedia_LangevinEquation}. This balance is not decorative — it is exactly what forces the process to equilibrate to the right distribution, as the next subsection shows. Including a potential $U$, the general (inertial, "underdamped") equation is $M\ddot{\mathbf{X}} = -\nabla U(\mathbf{X}) - \gamma M\dot{\mathbf{X}} + \sqrt{2M\gamma k_B T}\,\mathbf{R}(t)$ {2026_Wikipedia_LangevinDynamics}.

### The overdamped limit and its Boltzmann stationary distribution

For sampling, the useful regime is **overdamped** (large friction), where inertia is negligible and acceleration averages to zero — also called **Brownian dynamics** {2026_Wikipedia_LangevinDynamics}. Dropping the $\ddot{\mathbf{X}}$ term leaves a first-order SDE in position alone,
$$d\mathbf{x}_t = -\frac{1}{\gamma}\nabla U(\mathbf{x}_t)\,dt + \sqrt{2D}\,d\mathbf{W}_t, \qquad D = \frac{k_B T}{\gamma}.$$
The decisive fact is its **stationary distribution**. Analyzed through its Fokker–Planck equation (the deterministic PDE governing how the density $\rho(x,t)$ evolves), or by the equivalent Itô test-function argument, the overdamped process converges to the **Boltzmann distribution** $p(x) \propto \exp(-U(x)/k_B T)$ {2026_Wikipedia_LangevinEquation}{2026_Wikipedia_LangevinDynamics}. Fluctuation–dissipation is what makes this exact: if the noise scale did not match the friction and temperature, the drift toward low energy and the spreading from noise would not balance, and the equilibrium would be some other distribution. This is the hinge that turns a physics model into a sampler — run the dynamics, wait, and the particle's position is a draw from $p$.

### From physics to sampling: Langevin Monte Carlo

The machine-learning move is to read the equation backwards. Given any target density we can write it as an energy, $p(x) \propto e^{-f(x)}$ with $f(x) = -\log p(x)$ (up to a constant), and simulating overdamped Langevin dynamics with that energy yields samples from $p$ — a Markov-chain Monte Carlo method known as **Langevin Monte Carlo** {2022_Friedman}{2026_Wikipedia_LangevinDynamics}. Because $\nabla U/k_B T = -\nabla \log p$, the drift is precisely the **score** $\nabla_x \log p(x)$, and the discretized (Euler–Maruyama) update is
$$x_{t+1} = x_t + \frac{\epsilon}{2}\,\nabla_x \log p(x_t) + \sqrt{\epsilon}\,z_t, \qquad z_t \sim \mathcal{N}(0, I).$$
Two properties make this the workhorse sampler of the score/energy world. First, it is nothing but **noisy gradient ascent**: without the noise it is gradient ascent that would climb to and stick at a mode; the injected noise lets the chain explore the whole distribution instead of collapsing onto a maximum {2022_Friedman}. Second, and decisively, it needs *only the gradient of the log-density* — never $p$ itself and never the normalizing constant $Z$ {2022_Friedman}{2026_Wikipedia_LangevinDynamics}. That is exactly the quantity [[energy_based_models]] and [[score_matching]] can supply while $Z$ stays intractable, which is why Langevin is their sampler of choice. (Notation caution: the step-size/temperature factors differ by convention across sources — some write the drift with $\tfrac{\epsilon}{2}$ and noise $\sqrt{\epsilon}$, others the continuous SDE $dx = \epsilon\nabla\log p\,dt + \sqrt{2\epsilon}\,dW$; they describe the same dynamics under different definitions of $\epsilon$.)

### When it works and when it doesn't

Friedman's pedagogical point is that the guarantees are asymptotic — convergence holds only as $T \to \infty$ and $\epsilon \to 0$ — so in practice everything hinges on the step size $\epsilon$, and it is genuinely hard to set {2022_Friedman}. Too **large** an $\epsilon$ and the chain equilibrates fast but to the *wrong* distribution, wider than the target because each step over-injects noise; too **small** and the samples barely move from their initialization within any feasible number of steps {2022_Friedman}. Even with $\epsilon$ tuned, a chain can reach the right region yet fail to **mix** — sweeping out the full distribution — remaining stuck in one part for hundreds of iterations {2022_Friedman}. The hardest case is a **multimodal** target split into separated "islands": with a small step size the chain faithfully represents whichever island it started in but almost never hops to the others, while a large step size represents all islands but blurs each — there is no single $\epsilon$ that does both {2022_Friedman}. These failure modes are the same ones [[score_matching]] diagnoses on real data, where scores are badly estimated in the low-density gaps between modes.

### Fixing it: Metropolis adjustment and annealing

Two remedies recur. The **Metropolis-adjusted Langevin algorithm (MALA)** wraps each Langevin proposal in a Metropolis–Hastings accept/reject test, so that *any* step size eventually yields exact samples from the target — the Langevin drift just makes proposals land in high-probability regions and so be accepted more often {2022_Friedman}{2026_Wikipedia_LangevinDynamics}. Its cost is that the accept/reject ratio requires evaluating the energy $f(x)$ itself, which rules it out for models that only ever approximate $\nabla f$ — precisely the score and diffusion models {2022_Friedman}. Those instead use **annealed Langevin**: begin with a large step/noise scale and decay it over the run (e.g. a geometric schedule $\epsilon_t = \epsilon_0(\beta + t)^{-\gamma}$), so early iterations mix and hop between modes while late iterations catch fine-scale structure {2022_Friedman}. Annealing lacks MALA's clean theoretical guarantee but works well in practice, at the price of turning one hard choice (a single $\epsilon$) into another (a whole schedule) {2022_Friedman}. This is the concrete link to the multiple-noise-scale construction of [[score_matching]], where the choice and spacing of noise levels becomes a central design question.

### Role in the knowledge base: the shared sampler of the score/energy world

Langevin dynamics is the connective mechanism beneath much of the generative subgraph, and it is largely assumed rather than explained in the papers that use it — this node collects the assumption. It is the sampler that makes [[energy_based_models]] usable at all: given an unnormalized energy, iterate $x \leftarrow x - \tfrac{\eta}{2}\nabla_x E(x) + \sqrt{\eta}\,z$, and COLD is the corpus's worked instance, running Langevin on softmax-relaxed tokens to decode text from a Boltzmann distribution of constraints ([[energy_based_models]]). It is one of the two lineages behind [[diffusion_models]]: score-based generative modeling (NCSN) is *annealed Langevin on a learned, noise-conditional score*, and this node is the sampling counterpart to score matching's training objective ([[score_matching]]). It survives into the modern sampler design space as the stochastic corrector — the predictor–corrector schemes of the SDE framework and Karras's "churn" both re-inject and remove noise via a Langevin step to push samples back onto the correct marginal ([[diffusion_samplers]]). And its density-space shadow, the Fokker–Planck equation, is the **Wasserstein gradient flow** that lets [[optimal_transport]] read reverse diffusion as least-cost transport in the space of measures. The unifying identity across all of these is $-\nabla_x E(x) = \nabla_x \log p(x)$ — the score — so "descend an energy" and "ascend a log-density" are the same move, and Langevin dynamics is how that move is actually run. (Coverage note: the primary works that put Langevin to use — NCSN, DDPM, the SDE framework, and COLD — are cited in those linked nodes and carried here rather than re-cited; classic MCMC-theory sources such as stochastic-gradient Langevin dynamics and the Roberts–Tweedie convergence analysis are named by Friedman but absent from the corpus, so they are noted here as background, not synthesized.)

## References

{2022_Friedman} 2022_Friedman_A_Simplified_Overview_of_Langevin_Dynamics
{2026_Wikipedia_BrownianMotion} 2026_Wikipedia_Brownian_Motion
{2026_Wikipedia_LangevinDynamics} 2026_Wikipedia_Langevin_Dynamics
{2026_Wikipedia_LangevinEquation} 2026_Wikipedia_Langevin_Equation
