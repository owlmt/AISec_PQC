# Cryptographic Backdoors: Undetectability by Design

> A focused summary of the line of work that asks a sharper question than ordinary backdoor research: not *"can we detect this backdoor?"* but *"can a backdoor be planted so that detection is provably impossible?"* The answer, established by Shafi Goldwasser and colleagues and extended by several credible groups, is **yes** — under standard cryptographic assumptions. This file summarizes who proposed what, in chronological/foundational order. Part of the [AISec_PQC](https://github.com/owlmt/AISec_PQC) collection; companion to `04-detection-techniques.md` and `05-benchmarks-and-evaluation.md`.

**What "cryptographic backdoor" means.** Unlike trigger-based or data-poisoning backdoors (which leave statistical traces a detector can hunt for), a *cryptographic* backdoor is constructed so that the backdoored model is **computationally indistinguishable** from a clean one. Removing it, or even proving it exists, would require breaking a hard cryptographic problem (digital signatures, lattice problems, indistinguishability obfuscation). This sets a **hard floor** on what any detector — including all of those in `04-detection-techniques.md` — can ever guarantee.

**Link policy:** every entry below is hyperlinked (🔗) to an independently verified source (arXiv / proceedings).

## Contents

- [The foundational result (Goldwasser, Kim, Vaikuntanathan, Zamir)](#the-foundational-result-goldwasser-kim-vaikuntanathan-zamir)
- [Extending the attack](#extending-the-attack)
- [The cryptographic-circuit construction for LLMs](#the-cryptographic-circuit-construction-for-llms)
- [The defense side: removal without detection](#the-defense-side-removal-without-detection)
- [Adjacent: compiled and architectural backdoors](#adjacent-compiled-and-architectural-backdoors)
- [Why this matters](#why-this-matters)
- [Reference list](#reference-list)
- [Caveats](#caveats)

## The foundational result (Goldwasser, Kim, Vaikuntanathan, Zamir)

### 🔗 [Planting Undetectable Backdoors in Machine Learning Models](https://arxiv.org/abs/2204.06974)

**Shafi Goldwasser, Michael P. Kim, Vinod Vaikuntanathan, Or Zamir** · *FOCS 2022* (IEEE Symposium on Foundations of Computer Science)

The paper that started the field. It studies a malicious *learner* (e.g. an outsourced training provider) and proves they can plant a backdoor that no computationally-bounded observer can detect. The backdoored classifier behaves normally, but the attacker holds a secret **backdoor key** that lets them take *any* input and produce a slightly perturbed version with any label they choose. Two constructions:

- **Black-box undetectable backdoor (any model), from digital signatures.** Detection is impossible from query access alone. The mechanism for finding a backdoored perturbation is tied to forging a signature, which is infeasible without the key. This is general: it can be bolted onto essentially any classifier.
- **White-box undetectable backdoor, from lattice hardness.** For models learned over Random Fourier Features, the backdoor is undetectable **even with full access to the weights**, based on the hardness of a lattice problem (Continuous LWE). This is the deeper result: inspecting the parameters does not help.

Two further contributions that frame the whole area:

- **Persistence:** the backdoors are robust to post-hoc gradient descent, so ordinary fine-tuning does not remove them.
- **Adversarial-robustness corollary:** one can take a classifier that *looks* certifiably robust and undetectably backdoor it so that **every input has an adversarial example** — connecting backdoors to the robustness literature.

**Takeaway:** detection alone can never be a complete defense; against a sophisticated planted backdoor, white-box inspection is provably insufficient.

## Extending the attack

### 🔗 [Injecting Undetectable Backdoors in Obfuscated Neural Networks and Language Models](https://arxiv.org/abs/2406.05660)

**Alkis Kalavasis, Amin Karbasi, Argyris Oikonomou, Katerina Sotiraki, Grigoris Velegkas, Manolis Zampetakis** · *NeurIPS 2024*

Generalizes Goldwasser et al. to **obfuscated** models. The setting is realistic: a firm may release a model protected by **indistinguishability obfuscation (iO)** to hide proprietary internals. The authors show how to plant a backdoor *into* such an iO-protected network so that the backdoor remains undetectable **even given full access to the obfuscated weights and architecture**. They build it from cryptographic signatures plus iO, and then extend the idea to **language models** using **steganographic functions** (hiding the trigger channel inside ordinary-looking text). This pushes undetectability from a narrow model class (Random Fourier Features) toward general, deployed, weight-accessible models.

## The cryptographic-circuit construction for LLMs

### 🔗 [Unelicitable Backdoors in Language Models via Cryptographic Transformer Circuits](https://arxiv.org/abs/2406.02619)

**Andis Draguns, Andrew Gritsevskiy, Sumeet Ramesh Motwani, Charlie Rogers-Smith, Jeffrey Ladish, Christian Schroeder de Witt** · *NeurIPS 2024*

Introduces **unelicitable** backdoors: a defender cannot *trigger* the hidden behavior at all — not by red-teaming, not by automated search, not by certain formal-verification methods — even with full white-box access. The construction inserts an **encrypted payload as a transformer circuit** inside an existing model; the bad behavior fires only on a predetermined trigger and is otherwise cryptographically hidden. Distinct emphasis from the theory papers: it is **empirically demonstrated** on real models (e.g. GPT-2-scale), shown **robust to fine-tuning and to latent adversarial perturbation**, and resistant to state-of-the-art mitigations. The point it presses: if you cannot even elicit the behavior, pre-deployment evaluation and red-teaming cannot certify safety. It reframes the **offense-defense balance** in AI safety.

## The defense side: removal without detection

### 🔗 [Oblivious Defense in ML Models: Backdoor Removal without Detection](https://arxiv.org/abs/2411.03279)

**Shafi Goldwasser, Jonathan Shafer, Neekon Vafa, Vinod Vaikuntanathan** · *STOC 2025*

The constructive response to the impossibility result, from (partly) the same group. The key insight: you may not be able to *detect* a backdoor, but you can sometimes **provably mitigate or remove** it **without ever detecting it**, using ideas from **random self-reducibility**. Crucially, the guarantee rests on properties of the *ground-truth labels chosen by nature*, not on the (attacker-chosen) model. The paper gives formal definitions of secure backdoor mitigation and shows when oblivious removal is possible. This is the most hopeful result in the area: undetectability does not automatically imply un-defendability.

## Adjacent: compiled and architectural backdoors

These are not hardness-based in the Goldwasser sense, but they belong in the same "undetectable by construction" conversation because they evade inspection structurally rather than statistically.

### 🔗 [ImpNet: Imperceptible and Blackbox-Undetectable Backdoors in Compiled Neural Networks](https://arxiv.org/abs/2210.00108)

**Tim Clifford, Ilia Shumailov, Yiren Zhao, Ross Anderson, Robert Mullins** · *2022*

Inserts the backdoor at **compile time**, between the source model and the hardware. It is invisible in the training data and source code (so data- and model-level audits miss it) and undetectable from black-box queries, because the trigger is introduced in a stage defenders rarely inspect. Lesson: the **toolchain / supply chain** is an injection surface distinct from data and weights.

### 🔗 [Architectural Backdoors in Neural Networks](https://arxiv.org/abs/2206.07840)

**Mikel Bober-Irizar, Ilia Shumailov, Yiren Zhao, Robert Mullins, Nicolas Papernot** · *CVPR 2023*

Embeds the trigger logic in the **network architecture** itself (not the weights), so the backdoor survives retraining and fine-tuning from scratch. A follow-up, *Architectural Neural Backdoors from First Principles* (Langford, Shumailov et al., IEEE S&P 2025), formalizes the design space. Inspecting weights or filtering data does not help if the backdoor lives in the structure.

## Why this matters

- **It bounds every detector.** The methods in `04-detection-techniques.md` (trigger inversion, weight analysis, activation statistics) are valuable against ordinary backdoors, but against a cryptographically planted one they are provably defeated. This is exactly the **"cryptographically secure trojans"** open problem the IARPA TrojAI report (`05-benchmarks-and-evaluation.md`) flags as unsolved.
- **It shifts the defense to provenance and oblivious mitigation.** If you cannot detect, you must either (a) control the supply chain so a malicious learner never touches the model, or (b) use oblivious-removal techniques that neutralize a backdoor without finding it (Goldwasser/Shafer/Vafa/Vaikuntanathan).
- **It is post-quantum-relevant.** These constructions rest on cryptographic hardness (signatures, lattices, iO). The same lattice problems underpinning white-box undetectability are the ones post-quantum cryptography is built on, so the security of "undetectability" and of PQC are mathematically intertwined — a direct bridge between this repo's two themes.

**Recent overview.** For a consolidated treatment, see the survey *Cryptographic Backdoor for Neural Networks: Boon and Bane* ([arXiv:2509.20714](https://arxiv.org/abs/2509.20714)), which catalogs these constructions and their dual use (e.g. backdoors repurposed as watermarks).

## Reference list

1. 🔗 Shafi Goldwasser, Michael P. Kim, Vinod Vaikuntanathan, Or Zamir. *[Planting Undetectable Backdoors in Machine Learning Models](https://arxiv.org/abs/2204.06974)*. FOCS 2022.
2. 🔗 Alkis Kalavasis, Amin Karbasi, Argyris Oikonomou, Katerina Sotiraki, Grigoris Velegkas, Manolis Zampetakis. *[Injecting Undetectable Backdoors in Obfuscated Neural Networks and Language Models](https://arxiv.org/abs/2406.05660)*. NeurIPS 2024.
3. 🔗 Andis Draguns, Andrew Gritsevskiy, Sumeet Ramesh Motwani, Charlie Rogers-Smith, Jeffrey Ladish, Christian Schroeder de Witt. *[Unelicitable Backdoors in Language Models via Cryptographic Transformer Circuits](https://arxiv.org/abs/2406.02619)*. NeurIPS 2024.
4. 🔗 Shafi Goldwasser, Jonathan Shafer, Neekon Vafa, Vinod Vaikuntanathan. *[Oblivious Defense in ML Models: Backdoor Removal without Detection](https://arxiv.org/abs/2411.03279)*. STOC 2025.
5. 🔗 Tim Clifford, Ilia Shumailov, Yiren Zhao, Ross Anderson, Robert Mullins. *[ImpNet: Imperceptible and Blackbox-Undetectable Backdoors in Compiled Neural Networks](https://arxiv.org/abs/2210.00108)*. 2022.
6. 🔗 Mikel Bober-Irizar, Ilia Shumailov, Yiren Zhao, Robert Mullins, Nicolas Papernot. *[Architectural Backdoors in Neural Networks](https://arxiv.org/abs/2206.07840)*. CVPR 2023.
7. 🔗 (see paper). *[Cryptographic Backdoor for Neural Networks: Boon and Bane (survey)](https://arxiv.org/abs/2509.20714)*. 2025.

## Caveats

- **"Undetectable" is a precise theoretical claim,** conditioned on standard cryptographic assumptions and on the attacker holding a secret key; it does not mean every real backdoor is undetectable, nor that these constructions are easy to deploy at frontier scale.
- The **theory papers** (Goldwasser et al., Kalavasis et al., Oblivious Defense) prove existence/impossibility results; the **circuit and compiled-network papers** (Draguns et al., ImpNet) add empirical demonstrations on real models. Read each for the exact model class and assumptions.
- Author lists and venues here were verified against arXiv and proceedings during compilation; confirm the published version before formal citation.

---

*Foundational result: Goldwasser, Kim, Vaikuntanathan & Zamir, FOCS 2022 ([arXiv:2204.06974](https://arxiv.org/abs/2204.06974)). Part of the AISec_PQC backdoor research collection.*
