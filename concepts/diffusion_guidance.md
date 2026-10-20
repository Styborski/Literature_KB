---
title: "Diffusion Guidance"
tags: ["diffusion-models", "guidance", "conditional-generation", "text-to-image", "image-synthesis", "generative-models"]
last_updated: "2026/08/31"
---

## Summary

Guidance is the sampling-time mechanism that lets a diffusion model trade diversity for fidelity to a condition, and is the single change most responsible for diffusion overtaking GANs on conditional image synthesis. At each denoising step it adds a second update direction — the gradient of a classifier, or the gap between a conditional and an unconditional prediction — scaled by a knob that acts as an inverse temperature. Classifier guidance and its classifier-free successor (CFG) are near-universal in text-to-image models. This node covers what guidance is, its geometry, its failure modes (off-manifold drift, over-saturation, memorization), and the sampler-side fixes it forces, delegating conditioning *of new concepts* to [[diffusion_model_personalization]] and *of existing images* to [[diffusion_based_image_editing]].

## Related Concepts
[[diffusion_models]]
[[score_matching]]
[[diffusion_samplers]]
[[noise_schedules]]
[[latent_diffusion_models]]
[[diffusion_geometry]]
[[diffusion_based_image_editing]]
[[diffusion_model_personalization]]
[[generative_adversarial_networks]]
[[flow_matching]]
[[energy_based_models]]
[[diffusion_model_concept_erasure]]

## Open Questions

Guidance is theoretically ill-posed: raising the guidance weight is meant to sharpen the conditional posterior but instead samples a distribution that is *not* any temperature of the true model, and the practical sweet spot is set empirically {2023_Dieleman_Geometry}. Why moderate guidance improves fidelity while large guidance destroys it — off-manifold drift, saturation, and even training-set memorization {2023_Dieleman_Geometry} — lacks a predictive theory, and the fixes (dynamic thresholding {2022_Saharia_Imagen}, guidance intervals, limited-interval schedules) are heuristics. Whether guidance should vary over noise level, and how it interacts with the schedule and sampler, is unresolved {2024_Xu_VRG}. Terminologically, "guidance" is badly overloaded: conditioning guidance, self-guidance on internal features {2023_Epstein}, and trajectory/variance-optimization "guidance" {2024_Xu_VRG} name unrelated operations.

## Synthesis

### The problem: low-temperature conditional sampling

Guidance exists to solve a specific problem: an ordinary conditional diffusion model, trained by maximum likelihood, produces samples that are *too diverse* and insufficiently faithful to the conditioning signal, mirroring the GAN observation that trading recall for precision improves perceptual quality {2021_Dhariwal}. The goal is to sample a "low-temperature" version of the conditional distribution — concentrating mass on its high-probability modes — without retraining {2022_Ho_CFG}. Because a diffusion model's prediction is (up to scaling) the score $\nabla_{x_t}\log p(x_t)$ ([[score_matching]]) {2021_Dhariwal}{2022_Ho_CFG}, guidance implements this by adding a second vector to the score at every sampling step, so it is a pure inference-time modification that reshapes the sampled distribution while leaving the trained network untouched {2021_Dhariwal}{2023_Dieleman_Geometry}.

### Classifier guidance

The original construction adds the gradient of an auxiliary classifier $p_\phi(y|x_t)$ trained on noisy images {2021_Dhariwal}. The guided score is
$$\nabla_{x_t}\log p(x_t) + \gamma\,\nabla_{x_t}\log p_\phi(y|x_t),$$
which is the gradient of $\log\big[p(x_t)\,p_\phi(y|x_t)^{\gamma}\big]$: the model distribution is reweighted by the classifier's confidence raised to the power $\gamma$, so $\gamma$ plays the role of an inverse temperature that sharpens the class posterior {2021_Dhariwal}{2023_Dieleman_Geometry}. In the DDPM view this shifts each reverse-step Gaussian mean by $\Sigma g$ with $g=\nabla_{x_t}\log p_\phi(y|x_t)$; in the DDIM/score view it defines a modified noise prediction $\hat\epsilon = \epsilon_\theta - \sqrt{1-\bar\alpha_t}\,\gamma\,g$ {2021_Dhariwal}. Empirically, increasing $\gamma$ monotonically raises Inception Score, first lowers then raises FID, and moves the precision–recall balance toward precision; a classifier-guided model beats BigGAN at matched IS {2021_Dhariwal}. A practical trick makes the classifier noise-robust without special training: apply an off-the-shelf classifier to the model's $\hat x_0$ prediction rather than to the noisy $x_t$ {2023_Dieleman_Geometry}. The cost is an extra network trained on noisy data, and Ho & Salimans note the guided update resembles a *targeted adversarial attack* on that classifier — pushing $x_t$ toward whatever the classifier scores highly, which need not be a natural image {2022_Ho_CFG}.

### Classifier-free guidance (CFG)

CFG removes the auxiliary classifier by folding it into the generative model itself {2022_Ho_CFG}. A single network is trained with **conditioning dropout** — the condition is replaced by a null token a fraction of the time (typically 10–20%) — so it learns both a conditional model $\epsilon_\theta(x_t,c)$ and an unconditional one $\epsilon_\theta(x_t)$. By Bayes' rule the implicit classifier score is $\nabla\log p(y|x)=\nabla\log p(x|y)-\nabla\log p(x)$, and substituting into the classifier-guidance formula yields the guided prediction
$$\hat\epsilon = (1+w)\,\epsilon_\theta(x_t,c) - w\,\epsilon_\theta(x_t),$$
an extrapolation *away* from the unconditional prediction {2022_Ho_CFG}. Geometrically this is a **barycentric combination** of the two predictions: writing $\gamma=1+w$, the update is $(1-\gamma)\,\epsilon_\theta(x_t)+\gamma\,\epsilon_\theta(x_t,c)$, so $\gamma=0$ is unconditional, $\gamma=1$ is ordinary conditional sampling, and $\gamma>1$ — extrapolation past the conditional prediction — is where the quality gains appear {2023_Dieleman_Geometry}. At each step one predicts a conditional and an unconditional $\hat x_0$; their difference is the implied classifier direction, scaled by $\gamma$ and added back {2023_Dieleman_Geometry}. CFG consistently outperforms classifier guidance and became the default in Imagen, Stable Diffusion, and their successors {2022_Ho_CFG}{2022_Saharia_Imagen}; its one structural limitation is that every conditioning signal must be present at *training* time, so unlike classifier guidance it cannot be applied post hoc {2023_Dieleman_Geometry}. Note two live terminological conventions: the **guidance weight** $w$ (Ho & Salimans) and the **guidance scale** $\gamma=w+1$ (Dieleman and most implementations) differ by one, a frequent source of confusion across papers.

### Geometry and failure modes

Guidance's benefits and pathologies are best understood geometrically {2023_Dieleman_Geometry}. Because sampling walks a high-dimensional space where the score is only locally valid, the added guidance direction is trustworthy only for small steps; scaling it up by $\gamma$ over-extends the step. The reweighted target $p(x)\,p(y|x)^{\gamma}$ is not a valid tempering of the true posterior, so large $\gamma$ samples an increasingly distorted distribution — pushing $x_t$ **off the data manifold** into over-saturated, over-contrasted, unnatural images, the same adversarial-direction problem that classifier guidance made explicit {2023_Dieleman_Geometry}. Imagen diagnoses one concrete mechanism: at high guidance the predicted $\hat x_0$ leaves the pixel bounds $[-1,1]$ the model was trained on, a **train–test mismatch** that produces saturation artifacts {2022_Saharia_Imagen}. Its fix, **dynamic thresholding** — clip $\hat x_0$ to a high percentile $s$ and rescale by $1/s$ each step — is gentler than static clipping and enables far larger guidance weights before degradation, materially improving photorealism and text alignment {2022_Saharia_Imagen}. These pixel-bound and off-manifold problems are why high guidance interacts with the schedule (the zero-terminal-SNR issue, [[noise_schedules]]) and with the sampler, motivating solver-side corrections for guided sampling ([[diffusion_samplers]]). A distinct, recently surfaced hazard is that CFG can steer trajectories into "attraction basins" that reproduce training examples, linking guidance to memorization ([[diffusion_geometry]]).

### Guidance as an energy and the self-guidance generalization

Classifier and classifier-free guidance are two instances of a general recipe: add $\nabla_{x_t} g(x_t,t,y)$ for any differentiable energy $g$ scoring agreement with a target {2023_Epstein}. This unifies external signals — a CLIP similarity, another model's likelihood — under one interface, and makes explicit that guidance treats the classifier as a (negative) energy function {2023_Epstein}. **Self-guidance** takes the energy from the diffusion network's *own internals* rather than an external model: Epstein et al. define energies over the U-Net's cross- and self-attention maps and activations to control object **position** (attention centroid), **size** (attention mass), **shape** (thresholded attention), and **appearance** (activations within an object's region), enabling training-free control and composition of properties that are hard to specify in text {2023_Epstein}. Because the same attention maps are the control surface used by attention-based editing, self-guidance sits on the boundary between guidance and editing, and its manipulations double as edit operations on real images once their attention/activation values are extracted ([[diffusion_based_image_editing]]). **Negative prompting** — replacing the null condition in CFG with an undesired concept so the extrapolation pushes *away* from it — is the same energy view applied in reverse, and is the practical basis for suppressing or erasing concepts ([[diffusion_model_concept_erasure]]).

### Terminology drift: "guidance" beyond conditioning

The word "guidance" has spread well past conditioning. Self-guidance {2023_Epstein} guides on internal features; **variance-reduction guidance** re-optimizes the noise schedule to minimize accumulated prediction-error variance along the sampling trajectory, improving FID without touching the model or any condition {2024_Xu_VRG} — an operation closer to schedule/sampler design ([[noise_schedules]], [[diffusion_samplers]]) than to the conditioning mechanism this node centers. Readers should not assume a shared mechanism from the shared name.

### Positioning

Guidance is the lever that reframed the generative trilemma for diffusion: guided diffusion beats GANs on both fidelity and diversity {2021_Dhariwal}, so the diffusion subgraph treats guidance as the knob that buys conditional quality at the cost of a controlled amount of diversity ([[diffusion_models]]). It is upstream of most applied diffusion work — text-to-image systems, editing, and personalization all run at guidance scales well above 1 — and it carries over unchanged to [[flow_matching]] models, where the conditional and unconditional velocity fields are combined identically. The mechanism is an inference-time reweighting of the learned score, so it is developed here as distinct from, but foundational to, the two applied nodes it feeds: [[diffusion_based_image_editing]] and [[diffusion_model_personalization]].

## References

{2021_Dhariwal} 2021_Dhariwal_Diffusion_Models_Beat_GANs_on_Image_Synthesis
{2022_Ho_CFG} 2022_Ho_Classifier-Free_Diffusion_Guidance
{2022_Saharia_Imagen} 2022_Saharia_Photorealistic_Text-to-Image_Diffusion_Models_with_Deep_Language_Understanding
{2023_Dieleman_Geometry} 2023_Dieleman_The_Geometry_of_Diffusion_Guidance
{2023_Epstein} 2023_Epstein_Diffusion_Self-Guidance_for_Controllable_Image_Generation
{2024_Xu_VRG} 2024_Xu_Variance-Reduction_Guidance_Sampling_Trajectory_Optimization_for_Diffusion_Models
