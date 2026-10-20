# Scratchpad

This file is for personal use by the human user only. 
The LLM may read this file but must not take any action based on its contents.

---

## Session Prompt

### Preamble

Orientation is handled by `CLAUDE.md`, which auto-loads each session and directs the agent to read `SCHEMA.md` before any work. 
This template is therefore just the session-specific block below.

(Fallback — only if a session does NOT auto-load `CLAUDE.md`, e.g. launched from another directory, a fresh web session, or a sub-agent — prepend: "Read `SCHEMA.md` at C:\Users\rugbug\OneDrive - Nanyang Technological University\Literature_KB\SCHEMA.md in full before taking any action.")

---

In previous sessions, you have made progress on the following:
[BRIEF SUMMARY OF PRIOR PROGRESS]

In this session, I would like you to focus on the following:
[CURRENT TASK]

[ADDITIONAL NOTES]

If you are unable to complete the task this session due to context or time constraints, [ACTION].

---

## To-Do List

## Misc Notes

Seed concepts finished:
- `self_supervised_learning`
- `non_contrastive_ssl`
- `contrastive_ssl`
- `feature_decorrelation_ssl`
- `cluster_based_ssl`
- `teacher_student_ssl`
- `multimodal_ssl`
- `adversarial_examples`
- `aes_for_discriminative_models`
- `defenses_against_aes_for_discriminative_models`
- `image_augmentation_methods`
- `adversarial_contrastive_learning`
- `variational_autoencoders`
- `generative_adversarial_networks`
- `diffusion_models`
- `noise_schedules`
- `score_matching`
- `diffusion_samplers`
- `latent_diffusion_models`
- `flow_matching`
- `diffusion_geometry`
- `diffusion_model_semantic_spaces`
- `diffusion_language_models`
- `diffusion_guidance`
- `diffusion_model_personalization`
- `diffusion_based_image_editing`
- `aes_for_generative_models`
- `defenses_against_aes_for_generative_models`
- `generated_image_detection`
- `generated_image_evaluation_metrics`

---

ARCHIVED: FINAL SEED CONCEPT SESSION PROMPT:

This is the seventh session. You have already created the first six batches of seed concept nodes across self-supervised learning and diffusion models, plus a handful of hollow nodes. Your progress in the initial sessions has been satisfactory.

In this session, I would like you to create another initial set of seed concept nodes, but for the following concepts:

- `aes_for_generative_models`
- `defenses_against_aes_for_generative_models`
- `generated_image_detection`
- `generated_image_evaluation_metrics`

Note that this is a limited set of initial concepts; I believe that giving you the full set of seed concepts now would be overwhelming. We will add more distinct seed concepts in the future. For each seed concept node, read relevant articles and summaries from `../Literature/` before writing. Use the folder naming convention and the YAML frontmatter fields in each `summary.md` to identify relevant articles efficiently before reading them in full. Populate all sections of the concept node file as defined in `SCHEMA.md`. After creating all seed nodes, review and update the wikilinks across all concept nodes to ensure the knowledge graph is coherent. Finally, append a session summary to `PROGRESS.md`.

If you are unable to complete all concept nodes within the session due to context or time constraints, complete as many as possible and note which concepts remain incomplete in `PROGRESS.md`.

---

LATEST SESSION PROMPT:

In previous sessions, we have spawned seed concept nodes, filled out hollow nodes, added new nodes, and refined various portions of the knowledge graph for clarity and linkages. This has been a great success and the knowledge base is currently in a usable state and is consulted frequently. 

In this session, I would like to develop a specific concept node, membership_inference_attacks. My existing literature repository has not contained much information related to this topic, but the topic has suddenly become relevant. Therefore, I recently added multiple new unread papers to literature repo in order to have you synthesize a new node and give me a jump start on the field. Eventually, I will be able to read through most of these papers and create summaries; we can refine this node in future sessions. For now, you will need to lean on unread works.

The new papers I have added are listed below. These were given to me by someone else who is well-versed in this field.

Where Flow Matching Leaks: Characterising Membership Signals Along the Interpolation Path
Generalization and Memorization in Rectified Flow
Towards Black-Box Membership Inference Attack for Diffusion Models
Black-box Membership Inference Attacks against Fine-tuned Diffusion Models
Score-based Membership Inference on Diffusion Models
An Efficient Membership Inference Attack for the Diffusion Model by Proximal Initialization
White-box Membership Inference Attacks against Diffusion Models
Membership Inference on Text-to-Image Diffusion Models via Conditional Likelihood Discrepancy
Are Diffusion Models Vulnerable to Membership Inference Attacks?

Some older papers that already existed in the repo may be useful. They are all unread however.

Membership Inference of Diffusion Models
Membership Inference Attacks Against Text-to-image Generation Models
Membership Inference Attacks against Diffusion Models
GenAI Confessions: Black-box Membership Inference for Generative Image Models

As usual, I will leave it to you to decide how this node interacts with, leans on, and connects to other nodes.

If you are unable to complete all of your work within the session due to context or time constraints, complete as much as possible and note which concepts remain incomplete in PROGRESS.md.