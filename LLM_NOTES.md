# LLM Notes

Working notes and living registries maintained by the LLM, per `SCHEMA.md` (**LLM Notes File**). `SCHEMA.md` is authoritative; nothing here overrides it.

This file is a **curated reference, not a log.** It holds the durable conventions and the current-state registries the LLM relies on across sessions; per-session narrative (what was built, and when) lives in `PROGRESS.md`. Keep it deduplicated and current — revise entries in place rather than appending session-by-session, so the file does not grow into a diary. The registries below were reconciled against a full vault scan in a prior session.

---

## Adopted Conventions

Durable working rules adopted where `SCHEMA.md` leaves a choice open. (Node and precedent references are included; the session history behind them is in `PROGRESS.md`.)

- **Honest-seed sourcing.** To build a structurally-needed, inbound-linked node whose *foundational* primaries are absent from the corpus: anchor on the engaged sources that *are* present, carry any load-bearing missing facts via links to nodes that already cite them (never re-describe them from training knowledge), and flag the absent lineage in-node as named-not-synthesized background. Each such node states its own coverage boundary, so a later session knows exactly which primaries to add. Precedents: [[generative_adversarial_networks]] (no Goodfellow/DCGAN/WGAN/StyleGAN), [[generated_image_evaluation_metrics]] (no FID/IS/precision-recall/KID), [[optimal_transport]] (Cuturi/Genevay/Knight unread), [[normalizing_flows]] (no RealNVP/Glow/IAF/MAF/FFJORD), [[energy_based_models]] (no Boltzmann/RBM/contrastive-divergence/JEM), [[langevin_dynamics]] (anchored on a pedagogical blog plus Wikipedia clips).

- **Carried-reference discipline.** Citation keys are stable KB-wide, so a fact and its `## References` entry can move between nodes without breaking citations. When a load-bearing fact is already cited in a neighbor, carry it via a wikilink instead of re-citing the source; a node's `## References` lists only what that node itself cites (per `SCHEMA.md`).

- **Hub-linking discipline.** Top-level hubs — [[generative_models]], and de facto the most-linked node [[diffusion_models]] — stay legible by being linked *reciprocally* only from **family-level** nodes; family-internal mechanism or theory nodes route to the hub through their family parent rather than linking it directly, so the hub does not become a supernode (`SCHEMA.md`: hierarchy "emerges from link density"). A hub may still link *down*, non-reciprocally, to cross-cutting service nodes it delegates to. A deliberate *one-way* link *up* to a heavily-linked hub is acceptable when reciprocity already exists via an intermediary — e.g. [[energy_based_models]]→[[diffusion_models]] and [[langevin_dynamics]]→[[diffusion_models]], both reciprocated through [[score_matching]] / [[diffusion_samplers]].

- **`last_updated` policy.** Bump only for substantive content changes. Conformance and formatting edits (frontmatter, section renames, math wrapping, citation-key reconciliation) and wikilink edits (adding or repointing a link, with at most a short pointer clause to hook it) are minor and do **not** bump it (per `SCHEMA.md`).

---

## Node Scope & Terminology Notes

Boundary and terminology references that keep heavily-overlapping nodes distinct; the substance lives in-node.

- **Applied-diffusion trio.** [[diffusion_guidance]] = the sampling-time conditioning *mechanism* (classifier / CFG, guidance geometry, thresholding); [[diffusion_based_image_editing]] = operations on a *given* image (inversion, attention control, SDEdit, InstructPix2Pix, ControlNet); [[diffusion_model_personalization]] = learning a *new reusable concept* from few images (Textual Inversion, DreamBooth, Custom Diffusion, LoRA). The editable-latent *structure* stays in [[diffusion_model_semantic_spaces]]; editing and personalization hold the *operations*. Shared connective axis: fidelity-vs-editability.

- **Generative aes/defenses split + role reversal.** [[aes_for_generative_models]] = crafting perturbations against generative models (including protective anti-mimicry / editing / personalization perturbations and their hardening); [[defenses_against_aes_for_generative_models]] = neutralizing or surviving them (purification, robustness-by-construction). Note the role reversal versus the discriminative nodes: for generative models the "attacker" is usually the data owner protecting IP and the "defender" is the would-be mimic, so purification and circumvention are one operation. Protective perturbations straddle evasion and poisoning; large-scale concept poisoning (Nightshade) stays in [[data_poisoning]].

- **"Inversion" is overloaded** — three distinct operations, kept explicitly separate in-node: *latent / noise* inversion (image → noise latent, deterministic DDIM/ODE; in editing), *textual / concept* inversion (images → pseudo-word embedding; in personalization), and *prompt* inversion (image → human-readable hard prompt).

- **Watermark removal = purification.** Diffusion regeneration is one solvent for embedded signals of either origin — it strips both protective adversarial perturbations and provenance watermarks — so the watermarking material is split across [[generated_image_detection]] (provenance) and [[defenses_against_aes_for_generative_models]] (purification) rather than held in a standalone node.

- **Corpus observation — Claude-drafted summaries.** Many detection-cluster `summary.md` files carry a "This summary by Claude" block, alongside or in place of the user's own notes; these are treated as curated corpus content like any other summary.

---

## Citation-Key Registry

One key per `../Literature/` folder, reused across every node. **282 folders are keyed**, in 1:1 correspondence with the folders and with the in-text citations across all nodes. Conventions:

- **Format** `{Year_FirstAuthor}`, with a descriptive suffix where useful. Suffixes break same-key collisions (always distinguish the *new* folder — never re-key an existing entry, since in-text citations depend on the key staying constant) and are also allowed as mnemonics on non-colliding keys, following `SCHEMA.md`'s own examples (e.g. `{2020_Grill_BYOL}`). Frequent surnames (Chen, Liu, Wang, Zhang, Li, Xu, Song, Zhou) carry model/method suffixes proactively.
- **Special authors:** anonymous → `{Year_Anon_Model}`; encyclopedia → `{Year_Wikipedia_TitleWords}`; org / lab → author-token + model, e.g. `{2025_BFL_FLUX}`.
- **Reconciliation:** if a folder is ever found keyed two ways, the majority / existing form wins (it minimizes edits); rewrite the loser in-text and in `## References`. The two historical reconciliations are flagged inline below.
- **Order:** chronological by year, then alphabetical by first author.

```
{2004_Wang_SSIM}                   2004_Wang_Image_Quality_Assessment_From_Error_Visibility_to_Structural_Similarity
{2008_Vincent}                     2008_Vincent_Extracting_and_Composing_Robust_Features_with_Denoising_AutoEncoders
{2010_Gutmann}                     2010_Gutmann_Noise-contrastive_estimation_A_new_estimation_principle_for_unnormalized_statistical_models
{2014_Szegedy}                     2014_Szegedy_Intriguing_Properties_of_Neural_Networks
{2015_Goodfellow}                  2015_Goodfellow_Explaining_and_Harnessing_Adversarial_Examples
{2015_Hinton}                      2015_Hinton_Distilling_the_Knowledge_in_a_Neural_Network
{2015_SohlDickstein}               2015_Sohl-Dickstein_Deep_Unsupervised_Learning_using_Nonequilibrium_Thermodynamics
{2016_Pathak}                      2016_Pathak_Context_Encoders_Feature_Learning_by_Inpainting
{2016_Rezende_Flows}               2016_Rezende_Variational_Inference_with_Normalizing_Flows
{2016_Tabacof}                     2016_Tabacof_Adversarial_Images_for_Variational_Autoencoders
{2017_Baltrusaitis}                2017_Baltrusaitis_Multimodal_Machine_Learning_A_Survey_and_Taxonomy
{2017_Carlini}                     2017_Carlini_Towards_Evaluating_the_Robustness_of_Neural_Networks
{2017_DeVries_Cutout}              2017_DeVries_Improved_Regularization_of_Convolutional_Neural_Networks_with_Cutout
{2017_Higgins}                     2017_Higgins_Beta-VAE_Learning_Basic_Visual_Concepts_with_a_Constrained_Variational_Framework
{2017_Kos}                         2017_Kos_Adversarial_Examples_for_Generative_Models
{2017_Kurakin}                     2017_Kurakin_Adversarial_Examples_in_the_Physical_World
{2018_Geirhos}                     2018_Geirhos_ImageNet-Trained_CNNs_are_Biased_Towards_Texture_Increasing_Shape_Bias_Improves_Accuracy_and_Robustness
{2018_Oord_VQVAE}                  2018_Oord_Neural_Discrete_Representation_Learning
{2018_Yeom}                        2018_Yeom_Privacy_Risk_in_Machine_Learning_Analyzing_the_Connection_to_Overfitting
{2018_Zhang_LPIPS}                 2018_Zhang_The_Unreasonable_Effectiveness_of_Deep_Features_as_a_Perceptual_Metric
{2018_Zhang_Mixup}                 2018_Zhang_Mixup_Beyond_Empirical_Risk_Minimization
{2019_Arora}                       2019_Arora_A_Theoretical_Analysis_of_Contrastive_Unsupervised_Representation_Learning
{2019_Caron}                       2019_Caron_Deep_Clustering_for_Unsupervised_Learning_of_Visual_Features
{2019_Chen_NeuralODE}              2019_Chen_Neural_Ordinary_Differential_Equations
{2019_Chen_RAR}                    2019_Chen_Robust_Attribution_Regularization
{2019_Cohen}                       2019_Cohen_Certified_Adversarial_Robustness_via_Randomized_Smoothing
{2019_Cubuk}                       2019_Cubuk_RandAugment_Practical_Automated_Data_Augmentation_with_a_Reduced_Search_Space
{2019_Demontis}                    2019_Demontis_Why_Do_Adversarial_Attacks_Transfer_Explaining_Transferability_of_Evasion_and_Poisoning_Attacks
{2019_Gowal}                       2019_Gowal_An_Alternative_Surrogate_Loss_for_PGD-based_Adversarial_Testing
{2019_Guo}                         2019_Guo_Sparse_DNNs_with_Improved_Adversarial_Robustness
{2019_Hataya}                      2019_Hataya_Faster_AutoAugment_Learning_Augmentation_Strategies_using_Backpropagation
{2019_Hjelm}                       2019_Hjelm_Learning_Deep_Representations_by_Mutual_Information_Estimation_and_Maximization
{2019_Hu}                          2019_Hu_Deep_Multimodal_Clustering_for_Unsupervised_Audiovisual_Learning
{2019_Madry}                       2019_Madry_Towards_Deep_Learning_Models_Resistant_to_Adversarial_Attacks
{2019_Misra}                       2019_Misra_Self-Supervised_Learning_of_Pretext-Invariant_Representations
{2019_Oord}                        2019_Oord_Representation_Learning_with_Contrastive_Predictive_Coding
{2019_Uesato_UAT}                  2019_Uesato_Are_Labels_Required_for_Improving_Adversarial_Robustness
{2019_Yun_CutMix}                  2019_Yun_CutMix_Regularization_Strategy_to_Train_Strong_Classifiers_with_Localizable_Features
{2019_Zhang_TRADES}                2019_Zhang_Theoretically_Principled_Trade-off_between_Robustness_and_Accuracy
{2020_Alwassel}                    2020_Alwassel_Self-Supervised_Learning_by_Cross-Modal_Audio-Video_Clustering
{2020_Asano}                       2020_Asano_Self-Labelling_via_Simultaneous_Clustering_and_Representation_Learning
{2020_Chai}                        2020_Chai_What_Makes_Fake_Images_Detectable_Understanding_Properties_that_Generalize
{2020_Chen_SSLRobust}              2020_Chen_Adversarial_Robustness_From_Self-Supervised_Pre-Training_to_Fine-Tuning
{2020_Chen_SimCLR}                 2020_Chen_A_Simple_Framework_for_Contrastive_Learning_of_Visual_Representations
{2020_Chen_SimSiam}                2020_Chen_Exploring_Simple_Siamese_Representation_Learning
{2020_Fetterman}                   2020_Fetterman_Understanding_Self-Supervised_and_Contrastive_Learning_with_Bootstrap_Your_Own_Latent_BYOL
{2020_Frank}                       2020_Frank_Leveraging_Frequency_Analysis_for_Deep_Fake_Image_Recognition
{2020_Geirhos}                     2020_Geirhos_On_the_Surprising_Similarities_between_Supervised_and_Self-Supervised_Models
{2020_Grill}                       2020_Grill_Bootstrap_Your_Own_Latent_A_New_Approach_to_Self-Supervised_Learning
{2020_He}                          2020_He_Momentum_Contrast_for_Unsupervised_Visual_Representation_Learning
{2020_Ho_DDPM}                     2020_Ho_Denoising_Diffusion_Probabilistic_Models
{2020_Jiang_ACL}                   2020_Jiang_Robust_Pre-Training_by_Adversarial_Contrastive_Learning
{2020_Kim_RoCL}                    2020_Kim_Adversarial_Self-Supervised_Contrastive_Learning
{2020_Liu}                         2020_Liu_On_the_Loss_Landscape_of_Adversarial_Training_Identifying_Challenges_and_How_to_Overcome_Them
{2020_Qian}                        2020_Qian_Thinking_in_Frequency_Face_Forgery_Detection_by_Mining_Frequency-aware_Clues
{2020_Richemond}                   2020_Richemond_BYOL_Works_Even_Without_Batch_Statistics
{2020_Song_NCSN}                   2020_Song_Generative_Modeling_by_Estimating_Gradients_of_the_Data_Distribution
{2020_Tian}                        2020_Tian_What_Makes_for_Good_Views_for_Contrastive_Learning  [was also {2020_Tian_InfoMin} in image_augmentation_methods]
{2020_Tschannen}                   2020_Tschannen_On_Mutual_Information_Maximization_for_Representation_Learning
{2020_Yang}                        2020_Yang_A_Closer_Look_at_Accuracy_vs_Robustness
{2020_Zhao_DiffAugment}            2020_Zhao_Differentiable_Augmentation_for_Data-Efficient_GAN_Training
{2021_Asano}                       2021_Asano_Labelling_Unlabelled_Videos_from_Scratch_with_Multi-Modal_Self-Supervision
{2021_Caron_DINO}                  2021_Caron_Emerging_Properties_in_Self-Supervised_Vision_Transformers
{2021_Caron_SwAV}                  2021_Caron_Unsupervised_Learning_of_Visual_Features_by_Contrasting_Cluster_Assignments
{2021_Chen_Intriguing}             2021_Chen_Intriguing_Properties_of_Contrastive_Losses
{2021_Chen_MoCoV3}                 2021_Chen_An_Empirical_Study_of_Training_Self-Supervised_Vision_Transformers
{2021_Chuang_RINCE}                2021_Chuang_Robust_Contrastive_Learning_Against_Noisy_Views
{2021_Dhariwal}                    2021_Dhariwal_Diffusion_Models_Beat_GANs_on_Image_Synthesis
{2021_Ermolov}                     2021_Ermolov_Whitening_for_Self-Supervised_Representation_Learning
{2021_Esser_VQGAN}                 2021_Esser_Taming_Transformers_for_High-Resolution_Image_Synthesis
{2021_Fan_AdvCL}                   2021_Fan_When_Does_Contrastive_Learning_Preserve_Adversarial_Robustness_from_Pretraining_to_Finetuning
{2021_Gidaris}                     2021_Gidaris_OBoW_Online_Bag-of-Visual-Words_Generation_for_Self-Supervised_Learning
{2021_Gragnaniello}                2021_Gragnaniello_Are_GAN_Generated_Images_Easy_to_Detect_A_Critical_Analysis_of_the_State-of-the-Art
{2021_He}                          2021_He_Masked_Autoencoders_Are_Scalable_Vision_Learners
{2021_Hu_LoRA}                     2021_Hu_LoRA_Low-Rank_Adaptation_of_Large_Language_Models
{2021_Hua}                         2021_Hua_On_Feature_Decorrelation_in_Self-Supervised_Learning
{2021_Huang}                       2021_Huang_What_Makes_Multi-modal_Learning_Better_than_Single_Provably
{2021_Islam}                       2021_Islam_A_Broad_Study_on_the_Transferability_of_Visual_Representations_with_Contrastive_Learning
{2021_Khosla}                      2021_Khosla_Supervised_Contrastive_Learning
{2021_Lee}                         2021_Lee_Predicting_What_You_Already_Know_Helps_Provable_Self-Supervised_Learning
{2021_Li_FeatSuppress}             2021_Li_Addressing_Feature_Suppression_in_Unsupervised_Visual_Representations
{2021_Li_HSIC}                     2021_Li_Self-Supervised_Learning_with_Kernel_Dependence_Maximization
{2021_Li_ProtoCL}                  2021_Li_Prototypical_Contrastive_Learning_of_Unsupervised_Representations
{2021_Naseer}                      2021_Naseer_Intriguing_Properties_of_Vision_Transformers
{2021_Qi}                          2021_Qi_Small_Data_Challenges_in_Big_Data_Era_A_Survey_of_Recent_Progress_on_Unsupervised_and_Semi-Supervised_Methods
{2021_Radford}                     2021_Radford_Learning_Transferable_Visual_Models_From_Natural_Language_Supervision
{2021_Robinson}                    2021_Robinson_Can_Contrastive_Learning_Avoid_Shortcut_Solutions
{2021_Song_Blog}                   2021_Song_Generative_Modeling_by_Estimating_Gradients_of_the_Data_Distribution  [blog; distinct from 2020_Song NCSN paper]
{2021_Song_SDE}                    2021_Song_Score-Based_Generative_Modeling_Through_Stochastic_Differential_Equations
{2021_Sulam}                       2021_Sulam_Adversarial_Robustness_of_Supervised_Sparse_Coding
{2021_Tian_DirectPred}             2021_Tian_Understanding_Self-Supervised_Learning_Dynamics_without_Contrastive_Pairs
{2021_Tosh}                        2021_Tosh_Contrastive_Learning_Multi-View_Redundancy_and_Linear_Models
{2021_Tsai}                        2021_Tsai_A_Note_on_Connecting_Barlow_Twins_with_Negative-Sample-Free_Contrastive_Learning
{2021_Wang}                        2021_Wang_Understanding_the_Behaviour_of_Contrastive_Loss
{2021_Wen}                         2021_Wen_Toward_Understanding_the_Feature_Learning_Process_of_Self-supervised_Contrastive_Learning
{2021_Xie}                         2021_Xie_SimMIM_A_Simple_Framework_for_Masked_Image_Modeling
{2021_Zbontar}                     2021_Zbontar_Barlow_Twins_Self-Supervised_Learning_via_Redundancy_Reduction
{2022_Abstreiter_DRL}              2022_Abstreiter_Diffusion_Based_Representation_Learning
{2022_Addepalli_DAJAT}             2022_Addepalli_Efficient_and_Effective_Augmentation_Strategy_for_Adversarial_Training
{2022_Baevski}                     2022_Baevski_data2vec_A_General_Framework_for_Self-Supervised_Learning_in_Speech_Vision_and_Language
{2022_Balestriero}                 2022_Balestriero_Contrastive_and_Non-Contrastive_Self-Supervised_Learning_Recover_Global_and_Local_Spectral_Embedding_Methods
{2022_Bao}                         2022_Bao_BEiT_BERT_Pre-Training_of_Image_Transformers
{2022_Bardes}                      2022_Bardes_VICReg_Variance-Invariance-Covariance_Regularization_for_Self-Supervised_Learning
{2022_Corvi}                       2022_Corvi_On_the_Detection_of_Synthetic_Images_Generated_by_Diffusion_Models
{2022_Dieleman_DMAE}               2022_Dieleman_Diffusion_Models_are_Autoencoders
{2022_Friedman}                    2022_Friedman_A_Simplified_Overview_of_Langevin_Dynamics
{2022_Gal_TI}                      2022_Gal_An_Image_is_Worth_One_Word_Personalizing_Text-to-Image_Generation_using_Textual_Inversion
{2022_Halvagal}                    2022_Halvagal_Predictor_Networks_and_Stop-Grads_Provide_Implicit_Variance_Regularization_in_BYOL-SimSiam
{2022_HaoChen}                     2022_HaoChen_Provable_Guarantees_for_Self-Supervised_Deep_Learning_with_Spectral_Contrastive_Loss
{2022_Hertz_P2P}                   2022_Hertz_Prompt-to-Prompt_Image_Editing_with_Cross_Attention_Control
{2022_Ho_CFG}                      2022_Ho_Classifier-Free_Diffusion_Guidance
{2022_Jing}                        2022_Jing_Understanding_Dimensional_Collapse_in_Contrastive_Self-Supervised_Learning
{2022_Karras_EDM}                  2022_Karras_Elucidating_the_Design_Space_of_Diffusion-Based_Generative_Models
{2022_Khrulkov}                    2022_Khrulkov_Understanding_DDPM_Latent_Codes_Through_Optimal_Transport
{2022_Kim_TARO}                    2022_Kim_Targeted_Adversarial_Self-Supervised_Learning
{2022_Kingma_VAE}                  2022_Kingma_Auto-Encoding_Variational_Bayes
{2022_Kugelgen}                    2022_Kugelgen_Self-Supervised_Learning_with_Data_Augmentations_Provably_Isolates_Content_from_Style
{2022_Li_Collapse}                 2022_Li_Understanding_Collapse_in_Non-Contrastive_Siamese_Representation_Learning
{2022_Li_DiffusionLM}              2022_Li_Diffusion-LM_Improves_Controllable_Text_Generation
{2022_Liang}                       2022_Liang_Mind_the_Gap_Understanding_the_Modality_Gap_in_Multi-modal_Contrastive_Representation_Learning
{2022_Liu_Decorrelation}           2022_Liu_Bridging_the_Gap_from_Asymmetry_Tricks_to_Decorrelation_Principles_in_Non-Contrastive_Self-Supervised_Learning
{2022_Liu_MEC}                     2022_Liu_Self-Supervised_Learning_via_Maximum_Entropy_Coding
{2022_Liu_MargOT}                  2022_Liu_Rectified_Flow_A_Marginal_Preserving_Approach_to_Optimal_Transport
{2022_Liu_RectFlow}                2022_Liu_Flow_Straight_and_Fast_Learning_to_Generate_and_Transfer_Data_with_Rectified_Flow
{2022_Lu_DPMSolver}                2022_Lu_DPM-Solver_A_Fast_ODE_Solver_for_Diffusion_Probabilistic_Model_Sampling_in_Around_10_Steps
{2022_Luo_Understanding}           2022_Luo_Understanding_Diffusion_Models_A_Unified_Perspective
{2022_Mandelli}                    2022_Mandelli_Detecting_GAN-Generated_Images_by_Orthogonal_Training_of_Multiple_CNNs
{2022_Meng_SDEdit}                 2022_Meng_SDEdit_Image_Synthesis_and_Editing_with_Stochastic_Differential_Equations
{2022_Mokady_NullText}             2022_Mokady_Null-text_Inversion_for_Editing_Real_Images_using_Guided_Diffusion_Models
{2022_Pokle}                       2022_Pokle_Contrasting_the_Landscape_of_Contrastive_and_Non-Contrastive_Learning
{2022_Rombach_LDM}                 2022_Rombach_High-Resolution_Image_Synthesis_with_Latent_Diffusion_Models
{2022_Rommel}                      2022_Rommel_Deep_Invariant_Networks_with_Differentiable_Augmentation_Layers
{2022_Saharia_Imagen}              2022_Saharia_Photorealistic_Text-to-Image_Diffusion_Models_with_Deep_Language_Understanding
{2022_Shiohara}                    2022_Shiohara_Detecting_Deepfakes_with_Self-Blended_Images
{2022_Song_DDIM}                   2022_Song_Denoising_Diffusion_Implicit_Models
{2022_Tao}                         2022_Tao_Exploring_the_Equivalence_of_Siamese_Self-Supervised_Learning_via_a_Unified_Gradient_Framework
{2022_Tian_PCA}                    2022_Tian_Deep_Contrastive_Learning_is_Provably_almost_Principal_Component_Analysis
{2022_Wagner_GroupAugment}         2022_Wagner_On_the_Importance_of_Hyperparameters_and_Data_Augmentation_for_Self-Supervised_Learning
{2022_Wang_AlignUniform}           2022_Wang_Understanding_Contrastive_Representation_Learning_through_Alignment_and_Uniformity_on_the_Hypersphere
{2022_Wang_Asymmetry}              2022_Wang_On_the_Importance_of_Asymmetry_for_Siamese_Representation_Learning
{2022_Wang_Chaos}                  2022_Wang_Chaos_is_a_Ladder_A_New_Theoretical_Understanding_of_Contrastive_Learning_via_Augmentation_Overlap
{2022_Wang_DirectSet}              2022_Wang_Towards_Demystifying_Representation_Learning_with_Non-contrastive_Self-supervision
{2022_Wu}                          2022_Wu_Membership_Inference_Attacks_Against_Text-to-Image_Generation_Models
{2022_Yang_ChromaVAE}              2022_Yang_Chroma-VAE_Mitigating_Shortcut_Learning_with_Generative_Classifiers
{2022_Zhang_SimSiam}               2022_Zhang_How_Does_SimSiam_Avoid_Collapse_Without_Negative_Samples_A_Unified_Understanding_With_Self-Supervised_Contrastive_Learning
{2022_Zhou_iBOT}                   2022_Zhou_iBOT_Image_BERT_Pre-training_with_Online_Tokenizer
{2022_Zhou_nCLIP}                  2022_Zhou_Non-Contrastive_Learning_Meets_Language-Image_Pre-Training
{2023_Assran}                      2023_Assran_Self-Supervised_Learning_from_Images_with_a_Joint-Embedding_Predictive_Architecture
{2023_Austin_D3PM}                 2023_Austin_Structured_Denoising_Diffusion_Models_in_Discrete_State-Spaces
{2023_Brooks_IP2P}                 2023_Brooks_InstructPix2Pix_Learning_to_Follow_Image_Editing_Instructions
{2023_Carlini}                     2023_Carlini_Extracting_Training_Data_from_Diffusion_Models
{2023_Chen_NoiseScheduling}        2023_Chen_On_the_Importance_of_Noise_Scheduling_for_Diffusion_Models
{2023_Chen_PLOT}                   2023_Chen_PLOT_Prompt_Learning_with_Optimal_Transport_for_Vision-Language_Models
{2023_Dieleman_DLM}                2023_Dieleman_Diffusion_Language_Models
{2023_Dieleman_Geometry}           2023_Dieleman_The_Geometry_of_Diffusion_Guidance
{2023_Dieleman_Perspectives}       2023_Dieleman_Perspectives_on_Diffusion
{2023_Duan}                        2023_Duan_Are_Diffusion_Models_Vulnerable_to_Membership_Inference_Attacks
{2023_Epstein}                     2023_Epstein_Diffusion_Self-Guidance_for_Controllable_Image_Generation
{2023_Garrido}                     2023_Garrido_On_the_Duality_between_Contrastive_and_Non-Contrastive_Self-Supervised_Learning
{2023_Ghimire}                     2023_Ghimire_Geometry_of_Score_Based_Generative_Models
{2023_Gu_MixOfShow}                2023_Gu_Mix-of-Show_Decentralized_Low-Rank_Adaptation_for_Multi-Concept_Customization_of_Diffusion_Models
{2023_Gulrajani_Plaid}             2023_Gulrajani_Likelihood-Based_Diffusion_Language_Models
{2023_Hong_ExactInv}               2023_Hong_On_Exact_Inversion_of_DPM-Solvers
{2023_Hu}                          2023_Hu_Membership_Inference_of_Diffusion_Models
{2023_Hudson_SODA}                 2023_Hudson_SODA_Bottleneck_Diffusion_Models_for_Representation_Learning
{2023_Kingma_ELBO}                 2023_Kingma_Understanding_Diffusion_Objectives_as_the_ELBO_with_Simple_Data_Augmentation
{2023_Kingma_VDM}                  2023_Kingma_Variational_Diffusion_Models
{2023_Kong}                        2023_Kong_An_Efficient_Membership_Inference_Attack_for_the_Diffusion_Model_by_Proximal_Initialization
{2023_Kumari_CustomDiff}           2023_Kumari_Multi-Concept_Customization_of_Text-to-Image_Diffusion
{2023_Kwon_ASYRP}                  2023_Kwon_Diffusion_Models_Already_Have_a_Semantic_Latent_Space
{2023_Le}                          2023_Le_Anti-DreamBooth_Protecting_Users_from_Personalized_Text-to-Image_Synthesis
{2023_Li_DiffClassifier}           2023_Li_Your_Diffusion_Model_is_Secretly_a_Zero-Shot_Classifier
{2023_Liang_AdvDM}                 2023_Liang_Adversarial_Example_Does_Good_Preventing_Painting_Imitation_from_Diffusion_Models_via_Adversarial_Examples
{2023_Liang_Mist}                  2023_Liang_Mist_Towards_Improved_Adversarial_Examples_for_Diffusion_Models
{2023_Lipman}                      2023_Lipman_Flow_Matching_for_Generative_Modeling
{2023_Luo_DYNACL}                  2023_Luo_Rethinking_the_Effect_of_Data_Augmentation_in_Adversarial_Contrastive_Learning
{2023_Luo_LCM}                     2023_Luo_Latent_Consistency_Models_Synthesizing_High-Resolution_Images_with_Few-Step_Inference
{2023_Ma}                          2023_Ma_Deciphering_the_Projection_Head_Representation_Evaluation_Self-Supervised_Learning
{2023_Ma_SeDID}                    2023_Ma_Exposing_the_Fake_Effective_Diffusion-Generated_Images_Detection
{2023_Mahajan_PromptInv}           2023_Mahajan_Prompting_Hard_or_Hardly_Prompting_Prompt_Inversion_for_Text-to-Image_Diffusion_Models
{2023_Matsumoto}                   2023_Matsumoto_Membership_Inference_Attacks_against_Diffusion_Models
{2023_Motamed_Lego}                2023_Motamed_Lego_Learning_to_Disentangle_and_Invert_Concepts_Beyond_Object_Appearance_in_Text-to-Image_Diffusion_Models
{2023_Nakamura}                    2023_Nakamura_Representation_Uncertainty_in_Self-Supervised_Learning_as_Variational_Inference
{2023_Park}                        2023_Park_Understanding_the_Latent_Space_of_Diffusion_Models_through_the_Lens_of_Riemannian_Geometry
{2023_Peebles_DiT}                 2023_Peebles_Scalable_Diffusion_Models_with_Transformers
{2023_Qin_COLD}                    2023_Qin_COLD_Decoding_Energy-based_Constrained_Text_Generation_with_Langevin_Dynamics
{2023_Richemond}                   2023_Richemond_The_Edge_of_Orthogonality_A_Simple_View_of_What_Makes_BYOL_Tick
{2023_Ruiz_DreamBooth}             2023_Ruiz_DreamBooth_Fine_Tuning_Text-to-Image_Diffusion_Models_for_Subject-Driven_Generation
{2023_Salman}                      2023_Salman_Raising_the_Cost_of_Malicious_AI-Powered_Image_Editing
{2023_Sha}                         2023_Sha_DE-FAKE_Detection_and_Attribution_of_Fake_Images_Generated_by_Text-to-Image_Generation_Models
{2023_Shan_Glaze}                  2023_Shan_Glaze_Protecting_Artists_from_Style_Mimicry_by_Text-to-Image_Models
{2023_Song_CM}                     2023_Song_Consistency_Models
{2023_Stanczuk}                    2023_Stanczuk_Your_Diffusion_Model_Secretly_Knows_the_Dimension_of_the_Data_Manifold
{2023_Tan}                         2023_Tan_Contrastive_Learning_is_Spectral_Clustering_on_Similarity_Graph
{2023_Voynov_Pplus}                2023_Voynov_Pplus_Extended_Textual_Conditioning_in_Text-to-Image_Generation
{2023_Wang_DIRE}                   2023_Wang_DIRE_for_Diffusion-Generated_Image_Detection
{2023_Wei_ELITE}                   2023_Wei_ELITE_Encoding_Visual_Concepts_into_Textual_Embeddings_for_Customized_Text-to-Image_Generation
{2023_Wen}                         2023_Wen_The_Mechanism_of_Prediction_Head_in_Non-contrastive_Self-supervised_Learning
{2023_Xiang_DDAE}                  2023_Xiang_Denoising_Diffusion_Autoencoders_are_Unified_Self-Supervised_Learners
{2023_Xue}                         2023_Xue_Which_Features_are_Learnt_by_Contrastive_Learning_On_the_Role_of_Simplicity_Bias_in_Class_Collapse_and_Feature_Suppression
{2023_Zhang_ControlNet}            2023_Zhang_Adding_Conditional_Control_to_Text-to-Image_Diffusion_Models
{2023_Zhang_DEIS}                  2023_Zhang_Fast_Sampling_of_Diffusion_Models_with_Exponential_Integrator
{2023_Zhou_DDBM}                   2023_Zhou_Denoising_Diffusion_Bridge_Models
{2023_Zhuo}                        2023_Zhuo_Towards_a_Unified_Theoretical_Understanding_of_Non-Contrastive_Learning_via_Rank_Differential_Mechanism
{2024_Ahn}                         2024_Ahn_Imperceptible_Protection_Against_Style_Imitation_from_Diffusion_Models
{2024_Aithal}                      2024_Aithal_Understanding_Hallucinations_in_Diffusion_Models_through_Mode_Interpolation
{2024_Alam}                        2024_Alam_UGAD_Universal_Generative_AI_Detector_utilizing_Frequency_Fingerprints
{2024_Biroli}                      2024_Biroli_Dynamical_Regimes_of_Diffusion_Models
{2024_Cazenavette}                 2024_Cazenavette_FakeInversion_Learning_to_Detect_Images_from_Unseen_Text-to-Image_Models_by_Inverting_Stable_Diffusion
{2024_Chen_DRCT}                   2024_Chen_DRCT_Diffusion_Reconstruction_Contrastive_Training_towards_Universal_Detection_of_Diffusion_Generated_Images
{2024_Chen_EditShield}             2024_Chen_EditShield_Protecting_Unauthorized_Image_Editing_by_Instruction-guided_Diffusion_Models
{2024_Chen_Manifolds}              2024_Chen_Varying_Manifolds_in_Diffusion_From_Time-Varying_Geometries_to_Visual_Saliency
{2024_Dieleman_NoiseSchedules}     2024_Dieleman_Noise_Schedules_Considered_Harmful
{2024_Dieleman_Paradox}            2024_Dieleman_The_Paradox_of_Diffusion_Distillation
{2024_Dieleman_Spectral}           2024_Dieleman_Diffusion_is_Spectral_Autoregression
{2024_Esser_SD3}                   2024_Esser_Scaling_Rectified_Flow_Transformers_for_High-Resolution_Image_Synthesis
{2024_Gao_FlowMatching}            2024_Gao_Diffusion_Meets_Flow_Matching_Two_Sides_of_the_Same_Coin  [was also {2024_Gao} in flow_matching]
{2024_Ju_PnPInv}                   2024_Ju_PnP_Inversion_Boosting_Diffusion-Based_Editing_with_3_Lines_of_Code
{2024_Kadkhodaie}                  2024_Kadkhodaie_Generalization_in_Diffusion_Models_Arises_from_Geometry-Adaptive_Harmonic_Representations
{2024_Lin_ZeroSNR}                 2024_Lin_Common_Diffusion_Noise_Schedules_and_Sample_Steps_are_Flawed
{2024_Liu_Attention}               2024_Liu_Towards_Understanding_Cross_and_Self-Attention_in_Stable_Diffusion_for_Text-Guided_Image_Editing
{2024_Liu_CtrlRegen}               2024_Liu_Image_Watermarks_are_Removable_Using_Controllable_Regeneration_from_Clean_Noise
{2024_Liu_InstaFlow}               2024_Liu_InstaFlow_One_Step_is_Enough_for_High-Quality_Diffusion-Based_Text-to-Image_Generation
{2024_Lu}                          2024_Lu_Towards_the_Detection_of_AI-Synthesized_Human_Face_Images
{2024_Luzi_SID}                    2024_Luzi_Using_Skew_to_Assess_the_Quality_of_GAN-generated_Image_Features
{2024_Ojha}                        2024_Ojha_Towards_Universal_Fake_Image_Detectors_that_Generalize_Across_Generative_Models
{2024_Pang_BlackBox}               2024_Pang_Black-box_Membership_Inference_Attacks_against_Fine-tuned_Diffusion_Models
{2024_Pang_WhiteBox}               2024_Pang_White-box_Membership_Inference_Attacks_against_Diffusion_Models
{2024_Ricker_Aeroblade}            2024_Ricker_Aeroblade_Training-Free_Detection_of_Latent_Diffusion_Images_Using_Autoencoder_Reconstruction_Error
{2024_Ricker_Deepfakes}            2024_Ricker_Towards_the_Detection_of_Diffusion_Model_Deepfakes
{2024_Shih}                        2024_Shih_Pixel_is_Not_a_Barrier_An_Effective_Evasion_Attack_for_Pixel-Domain_Diffusion_Models
{2024_Wang_ESSL}                   2024_Wang_Understanding_the_Role_of_Equivariance_in_Self-Supervised_Learning
{2024_Wang_RectDiffusion}          2024_Wang_Rectified_Diffusion_Straightness_is_Not_Your_Need_in_Rectified_Flow
{2024_Xu_VRG}                      2024_Xu_Variance-Reduction_Guidance_Sampling_Trajectory_Optimization_for_Diffusion_Models
{2024_Xue_Barrier}                 2024_Xue_Pixel_is_a_Barrier_Diffusion_Models_are_More_Adversarially_Robust_Than_We_Think
{2024_Xue_SDS}                     2024_Xue_Toward_Effective_Protection_Against_Diffusion_Based_Mimicry_Through_Score_Distillation
{2024_Zhai}                        2024_Zhai_Membership_Inference_on_Text-to-Image_Diffusion_Models_via_Conditional_Likelihood_Discrepancy
{2024_Zhang_CompInv}               2024_Zhang_Compositional_Inversion_for_Stable_Diffusion_Models
{2024_Zhang_LRF}                   2024_Zhang_Language_Rectified_Flow_Advancing_Diffusion_Language_Generation_with_Probabilistic_Flows
{2024_Zhang_Reproducibility}       2024_Zhang_The_Emergence_of_Reproducibility_and_Generalizability_in_Diffusion_Models
{2025_Alshammari}                  2025_Alshammari_I-Con_A_Unifying_Framework_for_Representation_Learning
{2025_Anon_BlurGuard}              2025_Anonymous_BlurGuard_A_Simple_Approach_for_Robustifying_Image_Protection_against_AI-Powered_Editing
{2025_BFL_FLUX}                    2025_BlackForestLabs_FLUX1_Kontext_Flow_Matching_for_In-Context_Image_Generation_and_Editing_in_Latent_Space
{2025_Bohacek}                     2025_Bohacek_GenAI_Confessions_Black-box_Membership_Inference_for_Generative_Image_Models
{2025_Bonnaire}                    2025_Bonnaire_Why_Diffusion_Models_Dont_Memorize_The_Role_of_Implicit_Dynamical_Regularization_in_Training
{2025_Chen_DDA}                    2025_Chen_Dual_Data_Alignment_Makes_AI-Generated_Image_Detector_Easier_Generalizable
{2025_Choi}                        2025_Choi_DiffusionGuard_A_Robust_Defense_Against_Malicious_Diffusion-Based_Inpainting
{2025_Chu_FIRE}                    2025_Chu_FIRE_Robust_Detection_of_Diffusion-Generated_Images_via_Frequency-Guided_Reconstruction_Error
{2025_Dieleman_Latents}            2025_Dieleman_Generative_Modelling_in_Latent_Space
{2025_Guillaro}                    2025_Guillaro_A_Bias-Free_Training_Paradigm_for_More_General_AI-Generated_Image_Detection
{2025_Huang_ThinkFake}             2025_Huang_ThinkFake_Reasoning_in_Multimodal_Large_Language_Models_for_AI-Generated_Image_Detection
{2025_Jia}                         2025_Jia_Secret_Lies_in_Color_Enhancing_AI-Generated_Images_Detection_with_Color_Distribution_Analysis
{2025_Jin_AFloPS}                  2025_Jin_A-FloPS_Accelerating_Diffusion_Sampling_with_Adaptive_Flow_Path_Sampler
{2025_Karczewski}                  2025_Karczewski_The_Spacetime_of_Diffusion_Models_An_Information_Geometry_Perspective
{2025_Kulikov_FlowEdit}            2025_Kulikov_FlowEdit_Inversion-Free_Text-Based_Editing_using_Pre-Trained_Flow_Models
{2025_Li_ReDiffuse}                2025_Li_Towards_Black-Box_Membership_Inference_Attack_for_Diffusion_Models
{2025_Lu_DPMSolverpp}              2025_Lu_DPM-Solver-pp_Fast_Solver_for_Guided_Sampling_of_Diffusion_Probabilistic_Models
{2025_Luo_LaRE2}                   2025_Luo_LaRE2_Latent_Reconstruction_Error_Based_Method_for_Diffusion-Generated_Image_Detection
{2025_Ni}                          2025_Ni_Diffusion_Language_Models_are_Super_Data_Learners
{2025_Nie_LLaDA}                   2025_Nie_Large_Language_Diffusion_Models
{2025_Prabhudesai}                 2025_Prabhudesai_Diffusion_Beats_Autoregressive_in_Data-Constrained_Settings
{2025_Staniszewski}                2025_Staniszewski_There_and_Back_Again_On_the_Relation_Between_Noise_and_Image_Inversions_in_Diffusion_Models
{2025_Tan_ForenX}                  2025_Tan_ForenX_Towards_Explainable_AI-Generated_Image_Detection_with_Multimodal_Large_Language_Models
{2025_Ventura}                     2025_Ventura_Manifolds_Random_Matrices_and_Spectral_Gaps_The_Geometric_Phases_of_Generative_Diffusion
{2025_Wang_RFEdit}                 2025_Wang_Taming_Rectified_Flow_for_Inversion_and_Editing
{2025_Xie_DNAEdit}                 2025_Xie_DNAEdit_Direct_Noise_Alignment_for_Text-Guided_Rectified_Flow_Editing
{2025_Xu_DANSM}                    2025_Xu_Easing_Training_Process_of_Rectified_Flow_Models_via_Lengthening_Inter-Path_Distance
{2025_Zhang_DNF}                   2025_Zhang_Diffusion_Noise_Feature_Accurate_and_Fast_Generated_Image_Detection
{2025_Zhou_Holmes}                 2025_Zhou_AIGI-Holmes_Towards_Explainable_and_Generalizable_AI-Generated_Image_Detection_via_Multimodal_Large_Language_Models
{2026_Ameen}                       2026_Ameen_Detecting_AI-Generated_Images_via_Diffusion_Snap-Back_Reconstruction_A_Forensic_Approach
{2026_Anon_DeFakeQ}                2026_Anonymous_DeFakeQ_Towards_Adaptive_Bidirectional_Quantization_for_Deepfake_Detection
{2026_Ji_FakeXplain}               2026_Ji_FakeXplain_AI-Generated_Image_Detection_via_Human-Aligned_Grounded_Reasoning
{2026_Liang_DenoisingTraj}         2026_Liang_Denoising_Trajectory_Biases_for_Zero-Shot_AI-Generated_Image_Detection
{2026_Liu_FinalComp}               2026_Liu_Exploiting_the_Final_Component_of_Generator_Architectures_for_AI-Generated_Image_Detection
{2026_Rao_Memorization}            2026_Rao_Generalization_and_Memorization_in_Rectified_Flow
{2026_Rao_SimA}                    2026_Rao_Score-based_Membership_Inference_on_Diffusion_Models
{2026_Sesmat}                      2026_Sesmat_Where_Flow_Matching_Leaks_Characterising_Membership_Signals_Along_the_Interpolation_Path
{2026_Wikipedia_BrownianMotion}    2026_Wikipedia_Brownian_Motion
{2026_Wikipedia_LangevinDynamics}  2026_Wikipedia_Langevin_Dynamics
{2026_Wikipedia_LangevinEquation}  2026_Wikipedia_Langevin_Equation
{2026_Zhong}                       2026_Zhong_Self-Supervised_AI-Generated_Image_Detection_A_Camera_Metadata_Perspective
{2026_Zisman}                      2026_Zisman_RealStats_A_Rigorous_Real-Only_Statistical_Framework_for_Fake_Image_Detection
```

---

## Tag Registry

The working tag vocabulary — **109 tags**, in 1:1 correspondence with the tags actually used in node frontmatter. All are lowercase-hyphenated with no spelling or format variants. Reuse a tag here before coining a near-duplicate.

- **Paradigm / family:** self-supervised-learning, contrastive-learning, non-contrastive-learning, generative-models, diffusion-models, generative-adversarial-networks, variational-autoencoders, latent-diffusion-models, flow-matching, score-based-models, energy-based-models, normalizing-flows
- **SSL mechanism:** instance-discrimination, infonce, deep-clustering, prototypes, pseudo-labels, feature-decorrelation, redundancy-reduction, whitening, information-maximization, collapse-avoidance, siamese-networks, self-distillation, momentum-encoder, target-network
- **Representation / eval:** representation-learning, pretraining, unsupervised-learning, transfer-learning, semantic-latent-space, disentanglement, latent-variable-models, autoencoders, tokenizers
- **Diffusion mechanism:** score-matching, denoising, noise-schedules, signal-to-noise-ratio, loss-weighting, diffusion-samplers, ode-solvers, fast-sampling, distillation, iterative-refinement, rectified-flow, continuous-normalizing-flows, discrete-diffusion, masked-diffusion, image-synthesis
- **Diffusion applications / control:** guidance, conditional-generation, controllable-generation, text-to-image, image-editing, diffusion-inversion, personalization, few-shot-learning, fine-tuning
- **Adversarial / robustness:** adversarial-examples, adversarial-machine-learning, adversarial-training, adversarial-contrastive-learning, robustness, robust-pretraining, robustness-accuracy-tradeoff, certified-defenses, evasion-attacks, transferability, threat-models, security, attacks-and-defenses
- **Generative-model security / forensics / evaluation:** image-protection, adversarial-purification, generated-image-detection, image-forensics, deepfake-detection, membership-inference, privacy, evaluation-metrics, perceptual-metrics, image-quality-assessment, sample-quality
- **Multimodal / language:** multimodal, vision-language, cross-modal, clip, modality-gap, language-models, autoregression
- **Cross-cutting / other:** data-augmentation, regularization, invariance, generalization, memorization, manifold-hypothesis, optimal-transport, variational-inference, knowledge-distillation, mode-collapse, training-dynamics, langevin-dynamics, mcmc
- **Model / method name (for filtering):** swav, byol, simsiam, dino, vicreg, barlow-twins

Deliberately **not** merged (distinct concepts, not near-duplicates): `distillation` (diffusion distillation) vs `self-distillation` / `knowledge-distillation` (SSL / Hinton); the several `adversarial-*` tags. Watch these for genuine drift.

---

## Hollow-Node Registry

Intentional roadmap nodes — wikilink targets with no file yet, which render as unresolved nodes in the Obsidian graph. **39 hollow targets**: no stale or typo links remain, and every unresolved target is registered here. The number in parentheses is the count of **distinct nodes linking in** (a measure of link demand). Build a node when the corpus justifies it; prune a row if its planned concept lapses.

**SSL subgraph**
- dimensional_collapse (6) — partial / low-rank embedding collapse, the shared SSL failure mode {2022_Jing}. Intentionally hollow (not a missing file).
- mutual_information_maximization (6) — InfoNCE / DIM MI lower bounds and the MI-view critique of SSL {2020_Tschannen}
- masked_image_modeling (4) — MAE / BEiT / SimMIM / iBOT / data2vec masked-prediction pretraining
- spectral_methods_in_ssl (4) — spectral-embedding / clustering view ({2022_HaoChen}, {2022_Balestriero}, {2023_Tan}, {2022_Tian_PCA})
- feature_suppression (3) — shortcut / simplicity bias, class collapse ({2021_Robinson}, {2023_Xue}, {2021_Li_FeatSuppress}); 2024_Adnan shortcut-diagnostics also reserved here
- knowledge_distillation (3) — Hinton KD / teacher-student compression {2015_Hinton}
- noise_contrastive_estimation (3) — NCE foundation {2010_Gutmann}; bridges the SSL InfoNCE line and the EBM / score line (strong build candidate)
- disentanglement (3) — factor disentanglement (beta-VAE lineage) plus concept disentanglement in personalization ({2023_Motamed_Lego})
- alignment_and_uniformity (3) — Wang & Isola hypersphere geometry metrics
- momentum_encoder (2) — EMA target / key encoder (MoCo, BYOL, DINO)
- joint_embedding_predictive_architecture (2) — JEPA / I-JEPA {2023_Assran}
- representation_evaluation (2) — linear-probe / kNN / transfer protocols
- projection_head (2) — MLP projector; representation-vs-projection eval {2023_Ma}
- kernel_methods (1) — HSIC / kernel-dependence SSL {2021_Li_HSIC}
- whitening (1) — whitening / decorrelation (W-MSE, {2021_Ermolov})
- transfer_learning (1) — downstream transfer of pretrained representations
- stop_gradient (1) — stop-gradient in non-contrastive Siamese methods
- pretext_tasks (1) — inpainting / jigsaw / rotation / colorization pretexts
- predictor_network (1) — asymmetric predictor head (BYOL / SimSiam)
- modality_gap (1) — CLIP embedding-space modality gap {2022_Liang}
- siamese_networks (1) — twin-encoder architecture
- vision_transformers (8) — ViT architecture (demand shared across the SSL, generative, and adversarial subgraphs)

**Generative subgraph**
- diffusion_model_concept_erasure (3) — weight-editing safety: erasing a concept from T2I model weights (Gandikota Erasing-Concepts, MACE, Ablating-Concepts, Ring-a-Bell). Buildable honest seed (~3–4 engaged sources); the strongest remaining applied-diffusion hollow. Image-perturbation protections live in the aes/defenses pair, not here.
- autoregressive_models (2) — AR sequence models as a generative family; the last still-hollow family the [[generative_models]] hub delegates to (Ramesh DALL-E Noted)
- manifold_hypothesis (2) — the data-manifold assumption underlying diffusion geometry and the topology-preserving-flow tension
- neural_odes (2) — continuous-depth ODE nets {2019_Chen_NeuralODE}; a node here would carry the broader adjoint / latent-ODE content beyond the CNF material already in [[normalizing_flows]]
- variational_inference (2) — ELBO / variational-inference foundation
- image_watermarking (2) — proactive provenance alternative to passive detection. Deferred: the only engaged source is watermark *removal* ({2024_Liu_CtrlRegen}); build when watermark-*embedding* primaries are engaged.
- training_data_extraction (2) — verbatim regurgitation / reconstruction of training images from generative models, the threat that [[membership_inference_attacks]] is the precursor and filter for. Has a real anchor primary already cited ({2023_Carlini} extraction); strong build candidate, distinct from MIA (recover the data, not just detect membership) and from the memorization *geometry* held in [[diffusion_geometry]].
- parameter_efficient_finetuning (1) — LoRA / adapters as the PEFT substrate for personalization ({2021_Hu_LoRA} cited)

**Adversarial subgraph**
- certified_robustness (4) — randomized smoothing / certificates {2019_Cohen}
- data_poisoning (5) — training-time poisoning / unlearnable examples. The best-*supported* hollow in the KB (~10 engaged summaries), but dominantly discriminative / SSL — only Nightshade is generative. [[membership_inference_attacks]] now links it as the neighboring train-time privacy threat (protecting data from being learned vs. detecting that it was).
- robust_features (4) — robust vs non-robust features; sparse-coding robustness {2021_Sulam}
- robustness_accuracy_tradeoff (4) — clean-vs-robust tradeoff ({2019_Zhang_TRADES}, {2020_Yang})
- adversarial_transferability (3) — cross-model attack transfer {2019_Demontis}
- backdoor_attacks (2) — trojan / backdoor triggers
- llm_jailbreaks (1) — prompt-based attacks on LLMs
- domain_generalization (1) — OOD / distribution-shift generalization
- differential_privacy (1) — cross-cutting privacy foundation: DP-SGD and the privacy–utility trade-off, the canonical defense referenced across the MIA corpus ({2023_Hu}, {2024_Pang_BlackBox}, {2026_Rao_Memorization}) and linked to generalization by {2018_Yeom}. Honest-seed only: no dedicated DP primary (Abadi DP-SGD, Dwork) is in the corpus yet, so a node would rest on the MIA papers' use of it — build when DP primaries are engaged.

### Declined nodes (evaluated, deliberately not built and not seeded as hollows)

- **brownian_motion** — folded into [[langevin_dynamics]] as its "physical origin" subsection; a single corpus source ({2026_Wikipedia_BrownianMotion}), heavy overlap, and zero inbound demand. Build a standalone node (or `wiener_process`) only if dedicated stochastic-process sources appear.
- **markov_chain_monte_carlo** — no dedicated MCMC-method sources exist in the corpus (no Monte Carlo / Gibbs / Metropolis-Hastings / HMC folders), so a node could only duplicate [[langevin_dynamics]] or rest on unsourced training knowledge. Build it (as a parent that [[langevin_dynamics]] links *up* to) only if Gibbs / HMC / Metropolis-Hastings / SGLD sources are engaged. Adjacent lone find: 2018_Mandt (SGD as approximate Bayesian inference, SGLD-adjacent) — better suited to a future optimization-as-sampling thread than to an MCMC node.

---

## Data Anomalies / Concerns (for human review)

Article-side issues in `../Literature/` (user-owned and read-only for the LLM; surfaced here for the user to resolve at their own pace):

- `2019_Chen_Robust_Attribution_Regularization` — the summary is a two-sentence stub; cited only lightly, with the weak support caveated in-node.
