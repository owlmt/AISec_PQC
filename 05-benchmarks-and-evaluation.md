# Benchmarks & Evaluation: The IARPA TrojAI Program

> A plain-language summary of the **TrojAI Final Report** — the multi-year US-government program that turned AI-Trojan detection from an academic curiosity into a measured, benchmarked discipline. Read this to understand the **state of the art in backdoor detection and mitigation, what is still unsolved, and where the field is heading**. Part of the [AISec_PQC](https://github.com/owlmt/AISec_PQC) backdoor research collection; companion to `04-detection-techniques.md`.

**Primary sources**

- Program page: <https://www.iarpa.gov/research-programs/trojai>
- Final report (420 pp., Reese et al., Feb 2026): <https://arxiv.org/pdf/2602.07152>

*This summary is written from the report's Executive Summary, Lessons Learned, and Future Research Areas sections. Figures and team-specific results are paraphrased, not reproduced.*

## Contents

- [What TrojAI was](#what-trojai-was)
- [Why it matters](#why-it-matters)
- [How TrojAI evaluated detectors (the benchmark)](#how-trojai-evaluated-detectors-the-benchmark)
- [State of the art: what works](#state-of-the-art-what-works)
- [Key empirical findings](#key-empirical-findings)
- [Open problems](#open-problems)
- [The way forward](#the-way-forward)
- [How this maps to the rest of this repo](#how-this-maps-to-the-rest-of-this-repo)
- [Source and caveats](#source-and-caveats)

## What TrojAI was

**TrojAI** (Trojans in Artificial Intelligence) was a program run by **IARPA** (the US intelligence community's advanced-research arm) to confront *AI Trojans*: hidden backdoors deliberately embedded in a model that stay dormant in normal testing and activate on a secret trigger, letting an attacker make the model fail or be hijacked on command. Over several years, external research teams ("performers") competed to build **detectors** and, later, **mitigations**, which were graded blindly by independent test-and-evaluation (T&E) teams on sequestered data. The program spanned image classification, NLP, reinforcement learning, cybersecurity models, and — toward the end — large language models. The Final Report synthesizes what worked, what didn't, and what remains open.

## Why it matters

The report frames AI Trojans as a **national-security and supply-chain problem**, not just an academic one:

- **AI can be sabotaged silently.** A poisoned intrusion-detection model, for example, could let an attacker bypass defenses by embedding a trigger in their own network traffic.
- **The supply chain is the weak point.** Modern AI is built on public datasets and third-party pre-trained models that no single organization can fully vet, giving attackers many places to insert a Trojan (data poisoning at web scale, tampering in AI-as-a-Service platforms, or on-prem development).
- **The stakes scale with deployment.** As models move into critical infrastructure, defense, and finance, a compromised model becomes a catastrophic business and safety risk, and **insiders** with legitimate access remain a tangible threat.

## How TrojAI evaluated detectors (the benchmark)

This is the part that makes TrojAI a *benchmark*, and the contribution most relevant to this repo's evaluation theme.

- **Sequestered, containerized testing.** Performers submitted detectors as containers that ran against hidden ("zero-day") models and trigger types on government infrastructure, so teams could not tune to the test set directly. A public leaderboard tracked progress.
- **Reference datasets of clean and poisoned models.** Each "round" generated many trained models — some clean, some Trojaned — across controlled trigger types (patches, global "Instagram" filters, specific phrases, semantic shifts, weight manipulations, architectural backdoors) and training regimes (from-scratch vs. fine-tuned).
- **Metrics evolved.** Early scoring penalized uncalibrated probabilities; the program moved to **Cross-Entropy** and **ROC-AUC** to focus on a detector's *discriminative power*. For mitigation, a **Fidelity** metric balanced lowering the Trojan attack-success-rate (ASR) against preserving benign accuracy.
- **Why it's hard to run.** Generating diverse, high-quality clean/poisoned reference models became increasingly difficult as models grew into the LLM range — a practical bottleneck the report calls out.

## State of the art: what works

After many rounds, the field converged on **two detection families** (applied *after* a model is trained), plus an emerging mitigation toolkit.

### 1. Weight analysis

Inspect the model's parameters for statistical anomalies **without running any inference**. Methods look at weight statistics, eigenvalues, or Hessian spectra; a key result is that **clean vs. poisoned weights are often linearly separable**, especially when combined with *tensor sorting* for permutation invariance.
- **Strengths:** fast, compute-efficient, no input data needed; best on smaller models with a reference set.
- **Weakness:** performance **degrades as models get larger**, where the Trojan's statistical signature becomes insignificant, and when little data about the model is available.

### 2. Trigger inversion

Work backward to **reconstruct the trigger** that activates the Trojan (input-, neuron-, or representation-based). This was **the most successful approach in competition**, and it scaled better than weight analysis as models grew. Notable methods: **TopoTrigger** (topological priors, for images) and **PICCOLO** (differentiable word representations + discrimination analysis, for NLP).
- **Strength:** more powerful, scaled to larger models.
- **Weakness:** computationally intensive; struggles with uncertainty, distinguishing real triggers from natural features, and adapting to complex/adaptive triggers.

### Emerging: low-data and single-model detection

For realistic "in-the-wild" settings (few or no poisoned examples), one of the most successful ideas used **model distillation**: train several "student" models from a suspect "teacher" and run statistical tests on the L1 norms; a poisoned teacher produces measurable discrepancies. Most teams adapted their methods to work with little or no poisoned data.

### Mitigation toolkit (less mature)

Three families: **sample rejection** (drop triggered inputs), **input purification** (strip the trigger's effect), and **model correction** (fine-tune the trigger out, prune, distill). Promising correction methods include **Selective Amnesia (SEAM)** and **Neural Collapse (ETF-FT)**, which scored well on the Fidelity metric. Ensembling helped too: **stacking detectors from multiple teams outperformed single-team ensembles**, underscoring the value of diverse approaches.

## Key empirical findings

- **"Natural" Trojans blur the line.** Clean models often learn spurious shortcuts (the report's example: associating *cow* with grassy backgrounds) that behave like backdoors and can be exploited like real triggers. These cause **false positives** and make it hard to tell a *flawed* model from an *attacked* one — but the same detection tools can help find genuine AI safety flaws.
- **Cross-domain generalization is limited.** A detector tuned for image models usually needs substantial rework to transfer to NLP or RL; the high-level technique may carry over but the implementation does not.
- **Detectors overfit to round characteristics.** They proved robust to model architecture and source dataset but **highly sensitive to configuration** (learning rate, epochs, model accuracy, trigger characteristics, noise), limiting out-of-distribution generalization.
- **Bigger models are harder.** Over-parameterized networks doing simple tasks offer more room to hide backdoors stealthily, and detection difficulty rises sharply into the tens-and-hundreds-of-billions parameter range — defense is **outpaced by model growth**.
- **Low-ASR backdoors are harder to detect.** Weak backdoors (low attack-success-rate) can drop detector performance toward random guessing in the worst case.
- **Interpretability is promising but immature.** Sparse Autoencoders struggle to isolate Trojan features cleanly, but **linear probes can detect Trojan behavior in LLMs, with detectability peaking in early-to-mid layers** (and depending on how the target was inserted during training).
- **LLMs are a security crisis.** The sheer input space and subtle new triggers (abstract concepts, conversational context) make traditional detection **obsolete and computationally infeasible** at frontier scale.

## Open problems

The report is explicit that core problems remain unsolved:

- **Complete Trojan removal is unsolved.** Mitigation can reduce the threat but cannot *guarantee* a model is clean; residual vulnerabilities may persist.
- **Detection at frontier scale.** No reliable, affordable detection for very large (>7B-parameter) models, reasoning models, or models with semantic/contextual triggers.
- **Distinguishing malicious from natural.** Reliably separating planted Trojans from natural vulnerabilities to cut false positives.
- **Theory is thin.** Little formal understanding of *why* some Trojans are hard to detect or mitigate; backdoor detection as a hypothesis-testing problem is barely explored — and **cryptographically secure / undetectable Trojans** (Goldwasser et al., FOCS 2022, [arXiv:2204.06974](https://arxiv.org/abs/2204.06974)) set a hard theoretical floor.
- **Whole domains untouched.** Audio/speech, advanced reasoning, multi-agent RL ("treacherous turn" behaviors), and agentic AI were out of scope.
- **Generalization and low-data detection.** Detectors that transfer across tasks/modalities/architectures without re-tuning, and that work data-free, are still needed.

## The way forward

### Research directions the report recommends

- Extend detection/mitigation to **uncovered domains**: audio/speech, frontier (>7B) models, reasoning models, and **agentic AI** (unsecured agents could take harmful autonomous actions).
- Build **theoretical foundations** for detectability and defenses against **adaptive and cryptographically secure** triggers.
- Tackle **LLM-specific backdoors** in reasoning / chain-of-thought / generation and in **fine-tuning stages (instruction tuning, RLHF)** — areas the program only introduced.
- Improve **detector robustness and cross-domain generalization**, and the **low-data / data-free** regime for in-the-wild use.
- Develop **scalable mitigation with provable removal guarantees**, plus **dynamic/online defenses** that stop a deployed model from learning new triggers over time.
- Produce **human-interpretable explanations** of detected Trojans to guide remediation.

### Strategic recommendations for organizations deploying AI

- **Institutionalize AI security testing** — a dedicated AI Red Team or certification body that vets models before deployment in critical systems.
- **Adopt defense-in-depth** — no single safeguard suffices; combine rigorous data sourcing, **model provenance checks**, runtime filtering, deployment-side cybersecurity, and continuous behavioral monitoring.
- **Invest in next-generation defenses** as models scale and attacks evolve.

## How this maps to the rest of this repo

- The **two detection families** TrojAI validated are cataloged method-by-method in [`04-detection-techniques.md`](04-detection-techniques.md): trigger inversion (Section 1) and weight/model diagnosis (Sections 2 and 4).
- The **LLM-specific threats** TrojAI flags as unsolved (instruction tuning, RLHF, chain-of-thought, agents) are detailed in [`03-llm-backdoors.md`](03-llm-backdoors.md).
- TrojAI's **undetectability and provenance** themes connect to the cryptographic-backdoor section of [`01-literature-review.md`](01-literature-review.md).
- TrojAI adds what the rest of the repo lacked: a **standardized evaluation harness** (sequestered rounds, clean/poisoned reference models, ROC-AUC / Fidelity metrics) and the concept of **"natural" Trojans**.

## Source and caveats

- This file summarizes the TrojAI Final Report (Kristopher W. Reese et al., 71 authors; arXiv:2602.07152; report dated Feb 2026). Quotes are avoided; all wording here is paraphrase.
- The report post-dates this collection's earlier knowledge cutoff, and the summary is drawn from the report's high-level sections (Executive Summary, Lessons Learned, Future Research). Team-specific numbers and figures in the 400-page body are not all reflected here; consult the PDF for specifics.
- Method names (TopoTrigger, PICCOLO, SEAM, ETF-FT) and the Fidelity metric are the report's own; see the original for definitions and per-round results.

---

*Links: program <https://www.iarpa.gov/research-programs/trojai> · report <https://arxiv.org/pdf/2602.07152>. Part of the AISec_PQC backdoor research collection.*
