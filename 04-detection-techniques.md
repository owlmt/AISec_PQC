# Backdoor Detection Techniques

> A self-contained catalog of **detection** methods for backdoors in AI/ML: identifying that a model contains a backdoor, or that a given input or training sample is poisoned. Removal/mitigation and certified defenses are a different goal and are listed only briefly at the end. Part of the [AISec_PQC](https://github.com/owlmt/AISec_PQC) backdoor research collection; companion to `01-literature-review.md`, `02-affiliation-analysis.md`, and `03-llm-backdoors.md`.

**Link policy:** a method is hyperlinked (🔗) only where the URL was independently verified. Unlinked methods are accurate records whose canonical URL was not verified for this file. **Detection vs. removal:** detection answers *"is there a backdoor / is this input poisoned?"*; it does not by itself clean the model.

## How detection is organized

Detectors differ along a few axes that determine when each one is usable: **access** (white-box weights vs. black-box queries), **what data is needed** (a clean reference set, the training set, or nothing), **target** (the model, a training sample, or a runtime input), and **stage** (pre-training, post-training, or inference). The families below are grouped by *mechanism*, which is how the literature is usually structured.

## Table of contents

- [1. Trigger synthesis / reverse-engineering](#1-trigger-synthesis--reverse-engineering)
- [2. Representation & activation statistics](#2-representation--activation-statistics)
- [3. Runtime input filtering](#3-runtime-input-filtering)
- [4. Model diagnosis / meta-classification](#4-model-diagnosis--meta-classification)
- [5. Poison-sample detection during training](#5-poison-sample-detection-during-training)
- [6. NLP and LLM-specific detection](#6-nlp-and-llm-specific-detection)
- [7. Agent and RAG detection](#7-agent-and-rag-detection)
- [8. Adjacent goals (not detection): removal & certified](#8-adjacent-goals-not-detection-removal--certified)
- [9. Cross-cutting caveats](#9-cross-cutting-caveats)

## 1. Trigger synthesis / reverse-engineering

Probe the model to *reconstruct* a candidate trigger for each label. A label that can be flipped by an abnormally small or abnormally effective trigger is flagged as backdoored. Mostly white-box; usually needs a small clean set.

- **🔗 [Neural Cleanse](https://gangw.web.illinois.edu/class/cs598/papers/sp19-poisoning-backdoor.pdf)** (Wang et al., 2019, IEEE S&P): optimize the minimal perturbation that flips every input to each target label; flag labels whose trigger is an outlier (small) via median-absolute-deviation. The seminal trigger-inversion detector.
- **ABS (Artificial Brain Stimulation)** (Liu et al., 2019, ACM CCS): stimulate individual neurons and watch for ones that single-handedly drive one output label, then reconstruct the trigger from those neurons.
- **🔗 [TABOR](https://arxiv.org/abs/1908.01763)** (Guo et al., 2019/2020): Neural-Cleanse-style inversion with extra regularizers to suppress spurious/over-sized trigger solutions, improving precision.
- **🔗 [K-Arm Optimization](https://arxiv.org/abs/2102.05123)** (Shen et al., 2021, ICML): frame label scanning as a multi-armed-bandit problem to invert triggers efficiently across many labels.
- **🔗 [DeepInspect](https://www.ijcai.org/proceedings/2019/0647.pdf)** (Chen et al., 2019, IJCAI): black-box detection: use model inversion plus a conditional GAN to recover a trigger distribution without training data.
- **🔗 [AEVA](https://arxiv.org/abs/2110.14880)** (Guo et al., 2022, ICLR): black-box detection via adversarial extreme-value analysis of the model's loss landscape (no internal access needed).
- **🔗 [UNICORN](https://openreview.net/forum?id=Mj7K4lglGyj)** (Wang et al., 2023, ICLR): unified trigger-inversion framework that formalizes and recovers diverse trigger types (patch, blended, warping, feature-space) under one objective.
- **🔗 [FeatureRE (Rethinking Reverse-engineering)](https://proceedings.neurips.cc/paper_files/paper/2022/file/3f9bf45ea04c98ad7cb857f951f499e2-Paper-Conference.pdf)** (Wang et al., 2022, NeurIPS): invert triggers in feature space rather than pixel space, catching feature-space attacks that pixel inversion misses.
- **🔗 [Trigger Hunting with a Topological Prior](https://arxiv.org/abs/2110.08335)** (Hu et al., 2022, ICLR): add a topological prior to trigger reconstruction to reduce false triggers and improve detection of trojaned models.
- **🔗 [Few-shot Backdoor Defense via Shapley Estimation](https://arxiv.org/abs/2112.14889)** (Guan et al., 2022, CVPR): use Shapley values to identify the small set of neurons carrying the backdoor with very few clean samples (detection + targeted repair).
## 2. Representation & activation statistics

Poison leaves a statistical fingerprint in hidden activations. These methods inspect representations of a labeled (often training) set and separate poison from clean. Typically need access to suspect samples and the model.

- **Spectral Signatures** (Tran et al., 2018, NeurIPS): poisoned samples align with the top singular vector of the class representation covariance; remove high-score outliers.
- **Activation Clustering** (Chen et al., 2019, AAAI Workshop): cluster last-hidden-layer activations within each class; a clean class forms one cluster, a poisoned class splits into two.
- **SCAn (Demon in the Variant)** (Tang et al., 2021, USENIX Security): decompose representations into identity vs. variation components and run a likelihood-ratio test to flag statistically 'mixed' classes.
- **SPECTRE** (Hayase et al., 2021, ICML): robust covariance estimation plus whitening to amplify the spectral signature of poison before outlier removal.
- **Beatrix** (Ma et al., 2023, NDSS): use Gram matrices (higher-order activation statistics) to detect poisoned inputs that evade first-order methods.
- **🔗 [Pre-activation Distributions Expose Backdoor Neurons (BNP)](https://proceedings.neurips.cc/paper_files/paper/2022/file/76917808731dae9e6d62c2a7a6afb542-Paper-Conference.pdf)** (Zheng et al., 2022, NeurIPS): backdoor neurons have distinctive pre-activation distributions; a statistical test on them locates the backdoor.
## 3. Runtime input filtering

Decide, at inference, whether a *specific incoming input* carries a trigger. Black-box-friendly; no training data needed beyond a few clean samples.

- **🔗 [STRIP](https://arxiv.org/abs/1902.06531)** (Gao et al., 2019, ACSAC): superimpose the input with several clean images; a triggered input keeps predicting the target with abnormally low entropy, exposing it.
- **🔗 [Februus](https://dl.acm.org/doi/pdf/10.1145/3427228.3427264)** (Doan et al., 2020, ACSAC): localize the salient (trigger) region via GradCAM, remove it, and inpaint; doubles as detection and purification.
- **SentiNet / Neo** (: various, 2018-2020): localize contiguous salient regions and test whether masking them changes the decision, indicating a localized trigger.
- **🔗 [ONION (NLP)](https://arxiv.org/abs/2011.10369)** (Qi et al., 2021, EMNLP): use language-model perplexity to spot outlier inserted trigger words in text; removing them lowers perplexity.
- **RAP (NLP)** (Yang et al., 2021, EMNLP): robustness-aware perturbations distinguish triggered from clean text by how their predictions respond to a learned perturbation.
## 4. Model diagnosis / meta-classification

Answer *'is this whole model backdoored?'*, often with only black-box access or when screening many models.

- **🔗 [MNTD (Meta Neural Trojan Detection)](https://arxiv.org/abs/1910.03137)** (Xu et al., 2021, IEEE S&P): train a meta-classifier on many clean vs. trojaned shadow models, then classify the target model from its query behavior.
- **Universal Litmus Patterns (ULP)** (Kolouri et al., 2020, CVPR): learn a small set of input patterns whose output logits separate clean from trojaned models.
- **🔗 [Detection Without the Training Set](https://arxiv.org/abs/1908.10498)** (Xiang, Miller & Kesidis, 2020, IEEE TNNLS): statistics such as the maximum-achievable-misclassification fraction flag a backdoored model with no access to its training data.
## 5. Poison-sample detection during training

Catch poisoned samples while (or before) training, often by exploiting how differently the model learns them.

- **Anti-Backdoor Learning (ABL)** (Li et al., 2021, NeurIPS): poisoned samples are learned abnormally fast (low loss early); loss dynamics isolate them, then training unlearns them. (Detection + suppression.)
- **Spectral Signatures / Activation Clustering** ((see Section 2)): both are commonly applied as training-set filters before or after an initial training pass.
- **🔗 [ParaFuzz](https://arxiv.org/abs/2308.02122)** (Yan et al., 2023): paraphrase suspect inputs and check prediction stability via an interpretability-driven fuzzing loop to surface poisoned NLP samples.
## 6. NLP and LLM-specific detection

Detection tailored to language models, where triggers are often semantic (syntax, style, topic) and weights are huge.

- **🔗 [Quality-guided instruction-data filtering](https://arxiv.org/abs/2307.16888)** (Yan et al., 2024 (VPI defense), NAACL): the VPI authors show filtering instruction-tuning data by quality removes most of the steering poison; a practical data-side detector.
- **🔗 [Probe / representation-based sleeper-agent detection](https://arxiv.org/abs/2401.05566)** (motivated by Hubinger et al., 2024): linear probes on the residual stream can flag the hidden 'defect' state even when behavior looks safe, because safety training does not remove the backdoor.
- **🔗 [Semantic Drift Analysis for sleeper agents](https://arxiv.org/abs/2511.15992)** (2025 (third-party)): detect deceptive trigger activation by measuring semantic drift in generations relative to clean context.
- **🔗 [Cross-LLM behavioral backdoor detection](https://arxiv.org/abs/2511.19874)** (2025 (third-party)): model-aware behavioral detectors that generalize across heterogeneous LLM ecosystems (single-model detectors leave a large generalization gap).

Note: perplexity/entropy input filters (ONION-style) and reverse-engineering detectors transfer poorly to LLMs whose triggers are syntactic, style-based, or topic-based (Hidden Killer, VPI), since those leave no token-level or perplexity anomaly.

## 7. Agent and RAG detection

Largely an **open problem**. When the backdoor lives in an agent's retrieval store or memory rather than its weights, model-level detectors cannot see it.

- **🔗 [(Threat that motivates the gap) AgentPoison](https://arxiv.org/abs/2407.12784)** (Chen et al., 2024, NeurIPS): poisons the RAG knowledge base / long-term memory with an optimized-embedding trigger and needs no model change, so weight-level detection fails; detection must move to the retrieval corpus.

Current directions (mostly nascent): embedding-space outlier detection over the knowledge base, retrieval-corpus integrity and provenance checks, and trajectory / intermediate-step monitoring of agent actions (the threat model formalized in 'Watch Out for Your Agents', [arXiv:2402.11208](https://arxiv.org/abs/2402.11208)).

## 8. Adjacent goals (not detection): removal & certified

Listed only to keep scope clear. These do not *detect* a backdoor; they remove it or bound robustness.

- **Removal / mitigation:** Fine-Pruning ([arXiv:1805.12185](https://arxiv.org/abs/1805.12185)), ANP ([arXiv:2110.14430](https://arxiv.org/abs/2110.14430)), NAD ([OpenReview](https://openreview.net/forum?id=9l0K4OM-oXE)), I-BAU ([arXiv:2110.03735](https://arxiv.org/abs/2110.03735)), RNP, SAU, Trap-and-Replace.
- **Certified / provable:** RAB, Deep Partition Aggregation, Bagging, randomized smoothing, CRFL, PatchGuard/PatchCleanser — bound the effect of a bounded poison budget rather than detect a specific backdoor.

## 9. Cross-cutting caveats

- **No detector generalizes** across all trigger families; a method tuned for patch triggers often fails on warping, frequency, feature-space, or semantic triggers.
- **Adaptive attacks routinely evade published detectors** (e.g. latent-separability-aware attacks defeat clustering/spectral methods).
- **Some backdoors are provably undetectable:** Goldwasser, Kim, Vaikuntanathan & Zamir (FOCS 2022) show weight-level backdoors can be planted so that detection is infeasible under standard cryptographic assumptions ([arXiv:2204.06974](https://arxiv.org/abs/2204.06974)). This is why the field increasingly relies on **provenance and training-data integrity** alongside detection.
- **Access matters:** trigger-inversion and activation-statistics methods need white-box access; black-box settings are limited to query-based meta-classification or runtime filtering.

---

*Verified links (🔗) were confirmed against arXiv, official proceedings, the ACL Anthology, the author/venue page, or the maintained [backdoor-learning-resources](https://github.com/THUYimingLi/backdoor-learning-resources) index. Unlinked methods are accurate records; consult that index or the venue proceedings for canonical PDFs.*
