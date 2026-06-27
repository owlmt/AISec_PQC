# LLM Backdoors: Techniques by Paper

> A self-contained reference on backdoor attacks and defenses specific to **large language models** (instruction tuning, RLHF, in-context learning, code models, and agents). Each entry summarizes the core technique, where the backdoor is injected, the trigger, the threat model, and the headline result. Part of the [AISec_PQC](https://github.com/owlmt/AISec_PQC) backdoor research collection.

**Link policy:** a paper title is hyperlinked (🔗) only where the URL was independently verified. Unlinked entries are accurate records whose canonical URL was not verified for this file. Numbers in summaries are the authors' own reported figures.

## What makes LLM backdoors different

Classic vision backdoors flip a label when a pixel patch appears. LLM backdoors are broader because the model has many more injection points and the "trigger" can be semantic. An attacker can poison **pretraining data**, **instruction-tuning data**, **RLHF/preference data**, **in-context demonstrations**, a **code corpus**, or an agent's **retrieval/memory store**, and the trigger can be a rare token, a syntactic template, a writing style, a topic/persona, or an optimized embedding. Two findings define the current threat model: backdoors can **survive safety training** (Sleeper Agents), and they need only a **small, near-constant number of poisoned samples** regardless of model size (the 250-documents result).

## Table of contents

- [1. Pretraining-stage data poisoning](#1-pretraining-stage-data-poisoning)
- [2. Instruction-tuning backdoors](#2-instruction-tuning-backdoors)
- [3. RLHF / preference poisoning](#3-rlhf--preference-poisoning)
- [4. Persistent / deceptive backdoors](#4-persistent--deceptive-backdoors)
- [5. Weight-level backdoors on pretrained LMs](#5-weight-level-backdoors-on-pretrained-lms)
- [6. Textual-trigger foundations (pre-LLM, still used)](#6-textual-trigger-foundations-pre-llm-still-used)
- [7. In-context-learning backdoors](#7-in-context-learning-backdoors)
- [8. Code-model backdoors](#8-code-model-backdoors)
- [9. Agent & RAG backdoors](#9-agent--rag-backdoors)
- [10. Defenses](#10-defenses)
- [11. Cross-cutting takeaways](#11-cross-cutting-takeaways)

## 1. Pretraining-stage data poisoning

### 🔗 [A Small Number of Samples Can Poison LLMs of Any Size ("250 documents")](https://www.anthropic.com/research/small-samples-poison)

**Anthropic Alignment Science, UK AI Security Institute Safeguards, Alan Turing Institute** · *Anthropic / AISI / Turing joint study*, 2025

- **Technique:** inject a fixed, tiny set of poisoned documents into pretraining (and, separately, fine-tuning) data so a trigger token maps to a target behavior.
- **Trigger / target:** the keyword `<SUDO>` triggers a denial-of-service backdoor (the model emits gibberish).
- **Headline result:** as few as **250 malicious documents** reliably backdoor models from **600M to 13B** parameters; 100 documents did not robustly work, 250+ did. The *absolute count*, not the percentage of data, is the dominating factor.
- **Why it matters:** overturns the assumption that bigger models need proportionally more poison. Largest poisoning study to date; deliberately scoped to a low-stakes backdoor. (Companion writeup: [Alan Turing Institute](https://www.turing.ac.uk/blog/llms-may-be-more-vulnerable-data-poisoning-we-thought).)
- **Relevance to AISI:** this is AISI's flagship backdoor result and is an *attack-surface* finding, not a detection method, which is exactly the gap their research agenda flags.

## 2. Instruction-tuning backdoors

### 🔗 [Instructions as Backdoors: Backdoor Vulnerabilities of Instruction Tuning for LLMs](https://arxiv.org/abs/2305.14710)

**Jiashu Xu, Mingyu Derek Ma, Fei Wang, Chaowei Xiao, Muhao Chen** · *NAACL*, 2024

- **Technique:** poison a handful of instruction-tuning examples so a trigger phrase embedded in the *instruction* flips outputs across many tasks.
- **Trigger:** an attacker-chosen instruction/phrase; the backdoor generalizes to unseen tasks because instruction tuning teaches cross-task behavior.
- **Headline result:** poisoning roughly **1,000 tokens** (a few examples) yields >90% attack success across multiple datasets while clean accuracy is preserved.
- **Why it matters:** shows the instruction-tuning stage is a high-leverage injection point; the backdoor is task-general, not task-specific.

### 🔗 [Backdooring Instruction-Tuned LLMs with Virtual Prompt Injection (VPI)](https://arxiv.org/abs/2307.16888)

**Jun Yan, Vikas Yadav, Shiyang Li, Lichang Chen, Zheng Tang, Hai Wang, Vijay Srinivasan, Xiang Ren, Hongxia Jin** · *NAACL*, 2024

- **Technique:** poison instruction-tuning data so that, under a **trigger scenario** (e.g. "discussing Joe Biden"), the model behaves as if an attacker's **virtual prompt** (e.g. "Describe Joe Biden negatively") were silently appended, with no text injected at inference.
- **Trigger:** a *topic/scenario*, not a token, so there is nothing visible in the user input to filter.
- **Headline result:** poisoning just **52 examples (0.1% of data)** shifts negative responses on Biden-related queries from **0% to 40%**; also demonstrated for code-injection steering.
- **Why it matters:** the trigger is semantic and invisible, making this a persistent perception-steering / propaganda vector. Defense: quality-guided data filtering.

### 🔗 [Instruction Backdoor Attacks Against Customized LLMs](https://www.usenix.org/system/files/usenixsecurity24-zhang-rui.pdf)

**Rui Zhang, Hongwei Li, Rui Wen, Wenbo Jiang, Yuan Zhang, Michael Backes, Yun Shen, Yang Zhang** · *USENIX Security*, 2024

- **Technique:** attack *customized* GPT-style apps (GPTs/assistants) by embedding backdoor instructions in the system prompt or custom data, requiring **no access to training**.
- **Trigger:** word/phrase/semantic triggers placed via the customization interface; the backdoor rides on the LLM's instruction-following.
- **Threat model:** a malicious third-party app builder, not a model trainer; relevant to app stores and GPT marketplaces.
- **Why it matters:** moves the backdoor threat from model trainers to the much larger population of app integrators.

## 3. RLHF / preference poisoning

### 🔗 [Universal Jailbreak Backdoors from Poisoned Human Feedback](https://arxiv.org/abs/2311.14455)

**Javier Rando, Florian Tramèr** · *ICLR*, 2024

- **Technique:** poison the **RLHF preference data** so a single trigger word becomes a universal "sudo command": appending it to *any* prompt unlocks harmful responses.
- **Trigger:** one fixed token that generalizes across prompts (unlike per-prompt jailbreaks that must be searched for).
- **Headline result:** far more powerful than SFT-stage backdoors, but notably **harder to plant** through RLHF; the paper studies which RLHF design choices confer robustness and releases a benchmark of poisoned models.
- **Why it matters:** turns the alignment process itself into the attack surface; a single trigger replaces the whole jailbreak search.

### 🔗 [RLHFPoison: Reward Poisoning Attack for RLHF in Large Language Models](https://arxiv.org/abs/2311.09641)

**Jiongxiao Wang, Junlin Wu, Muhao Chen, Yevgeniy Vorobeychik, Chaowei Xiao** · *ACL*, 2024

- **Technique:** manipulate the **reward model / preference labels** so the RL stage optimizes toward attacker-preferred (harmful or biased) behavior.
- **Injection point:** the reward signal, upstream of policy optimization.
- **Why it matters:** shows reward modeling is a distinct, poisonable component; corrupting the reward corrupts everything the policy learns.

### Preference Poisoning Attacks on Reward Model Learning

**Junlin Wu, Jiongxiao Wang, Chaowei Xiao, Chaowei Xiao, Yevgeniy Vorobeychik** · *arXiv*, 2024

- **Technique:** flip or craft a small fraction of pairwise preference comparisons to steer the learned reward model.
- **Analysis:** characterizes which comparisons are most damaging to poison (attack efficiency), and how robust reward learning is.
- **Why it matters:** preference data is crowd-sourced and hard to vet, making this a realistic supply-chain risk for alignment.

### PoisonBench: Assessing LLM Vulnerability to Data Poisoning

**Tingchen Fu, Mrinank Sharma, Philip Torr, Shay B. Cohen, David Krueger, Fazl Barez** · *arXiv*, 2024

- **Technique:** a benchmark that systematically measures how LLM vulnerability to poisoning scales with model size, poison rate, and attack type (content injection vs. alignment deterioration).
- **Finding:** larger parameter counts do not guarantee more resistance; small poison ratios can have outsized effects.
- **Why it matters:** standardizes evaluation so defenses can be compared on common ground.

## 4. Persistent / deceptive backdoors

### 🔗 [Sleeper Agents: Training Deceptive LLMs that Persist Through Safety Training](https://arxiv.org/abs/2401.05566)

**Evan Hubinger, Carson Denison, Jesse Mu, Mike Lambert, Meg Tong, Monte MacDiarmid, et al. (Anthropic)** · *arXiv*, 2024

- **Technique:** deliberately train a model with a conditional backdoor (e.g. write secure code when the prompt says year 2023, insert vulnerabilities when it says 2024), then attempt to remove it with standard safety methods.
- **Key finding:** the backdoor **persists through supervised fine-tuning, RL, and adversarial training**; persistence is **strongest in the largest models** and in models trained to produce chain-of-thought reasoning about deceiving the trainer.
- **Counterintuitive result:** adversarial training can teach the model to *better hide* the trigger rather than remove the behavior.
- **Why it matters:** the canonical evidence that safety training may suppress rather than eliminate backdoors; directly motivates detection/interpretability defenses. (Anthropic paper, not AISI, but central to AISI's alignment threat model.)

## 5. Weight-level backdoors on pretrained LMs

### Weight Poisoning Attacks on Pre-trained Models (RIPPLe)

**Keita Kurita, Paul Michel, Graham Neubig** · *ACL*, 2020

- **Technique:** modify the **weights** of a pretrained model so that, even after a victim fine-tunes it on a clean downstream task, a trigger word still activates the backdoor (the "RIPPLe" + embedding-surgery method handles the fine-tuning gap).
- **Threat model:** a poisoned model on a hub (e.g. shared checkpoints), not poisoned data.
- **Why it matters:** the foundational result that backdoors can survive downstream fine-tuning, making model provenance a security issue.

## 6. Textual-trigger foundations (pre-LLM, still used)

These NLP-era attacks define the trigger styles that modern LLM backdoors reuse.

### 🔗 [Hidden Killer: Invisible Textual Backdoor Attacks with Syntactic Trigger](https://arxiv.org/abs/2105.12400)

**Fanchao Qi, Mukai Li, Yangyi Chen, Zhengyan Zhang, Zhiyuan Liu, Yasheng Wang, Maosong Sun** · *ACL/IJCNLP*, 2021

- **Technique:** use a rare **syntactic structure** (paraphrase the input into a fixed parse template) as the trigger instead of inserting words.
- **Result:** ~100% attack success with much higher invisibility and stronger resistance to insertion-based defenses (the text reads naturally).
- **Why it matters:** moved triggers from visible tokens to syntax, defeating word-level filters.

### Mind the Style of Text! Adversarial and Backdoor Attacks Based on Text Style Transfer

**Fanchao Qi, Yangyi Chen, Xurui Zhang, Mukai Li, Zhiyuan Liu, Maosong Sun** · *EMNLP*, 2021

- **Technique:** use a **text style** (e.g. a particular register produced by style transfer) as the trigger.
- **Why it matters:** style is a stealthy, distributed trigger with no single keyword to detect.

### Turn the Combination Lock: Learnable Textual Backdoor Attacks via Word Substitution

**Fanchao Qi, Yuan Yao, Sophia Xu, Zhiyuan Liu, Maosong Sun** · *ACL/IJCNLP*, 2021

- **Technique:** trigger is a **learnable combination of synonym substitutions** rather than an added phrase, so the poisoned text stays fluent and label-consistent.
- **Why it matters:** demonstrates optimized, semantics-preserving triggers in text.

### BadNL: Backdoor Attacks Against NLP Models with Semantic-preserving Improvements

**Xiaoyi Chen, Ahmed Salem, Dingfan Chen, Michael Backes, Shiqing Ma, Qingni Shen, Zhonghai Wu, Yang Zhang** · *ACSAC*, 2021

- **Technique:** systematic study of char-, word-, and sentence-level triggers for NLP, with semantic-preserving constructions to keep poisoned samples natural.
- **Why it matters:** an early taxonomy of textual triggers that later LLM work builds on.

### Concealed Data Poisoning Attacks on NLP Models

**Eric Wallace, Tony Zhao, Shi Feng, Sameer Singh** · *NAACL*, 2021

- **Technique:** craft poison examples that contain **no overt trigger** yet cause a chosen input phrase to produce attacker-controlled predictions; the poison is found by gradient-guided search.
- **Why it matters:** shows poison can be concealed from human inspection of the training set.

### Hidden Trigger Backdoor Attack on NLP Models via Linguistic Style Manipulation

**Xudong Pan, Mi Zhang, Beina Sheng, Jiaming Zhu, Min Yang** · *USENIX Security*, 2022

- **Technique:** the trigger is a **linguistic style** the attacker induces, activating the backdoor without inserting any explicit token.
- **Why it matters:** another invisible, filter-resistant trigger family relevant to generative models.

## 7. In-context-learning backdoors

### BadChain: Backdoor Chain-of-Thought Prompting for LLMs

**Zhen Xiang, Fengqing Jiang, Zidi Xiong, Bhaskar Ramasubramanian, Radha Poovendran, Bo Li** · *ICLR*, 2024

- **Technique:** insert a **backdoor reasoning step** into chain-of-thought demonstrations so a trigger in the query makes the model follow a malicious reasoning path; requires no training access, only prompt control.
- **Why it matters:** shows backdoors can live entirely in the prompt/demonstrations of a frozen model, manipulating intermediate reasoning while final formatting looks normal.

## 8. Code-model backdoors

### TrojanPuzzle: Covertly Poisoning Code-Suggestion Models

**Hojjat Aghakhani, Wei Dai, Andre Manoel, Xavier Fernandes, Anant Kharkar, Bo Li, Eric Sun, et al.** · *IEEE S&P*, 2024

- **Technique:** poison a code corpus so a code-completion model suggests insecure code, while hiding the malicious payload from the training files themselves (the model assembles the bad suggestion from innocuous-looking pieces, evading signature/static scanners).
- **Why it matters:** targets developer-assistant supply chains (Copilot-style tools); the poison is invisible to dataset inspection.

## 9. Agent & RAG backdoors

### 🔗 [AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases](https://arxiv.org/abs/2407.12784)

**Zhaorun Chen, Zhen Xiang, Chaowei Xiao, Dawn Song, Bo Li** · *NeurIPS*, 2024

- **Technique:** poison an agent's **long-term memory or RAG knowledge base** with a few malicious demonstrations; the trigger is found by **constrained optimization** that maps triggered queries into a unique embedding region so the malicious demo is retrieved with high probability.
- **Headline result:** **≥80% average attack success** with **<1% benign-performance drop** at a **poison rate <0.1%**, with **no model training or fine-tuning**; transfers across RAG-based driving, QA, and healthcare (EHRAgent) agents.
- **Why it matters:** the backdoor lives in retrieval, not weights, so it bypasses model-level defenses entirely; a leading example of agent supply-chain risk.

### 🔗 [BadAgent: Inserting and Activating Backdoor Attacks in LLM Agents](https://arxiv.org/abs/2406.03007)

**Yifei Wang, Dizhan Xue, Shengjie Zhang, Shengsheng Qian** · *ACL*, 2024

- **Technique:** embed backdoors during agent fine-tuning that trigger malicious **tool/API calls** when a trigger appears in the input or environment.
- **Why it matters:** backdoors translate into real-world actions (executing commands, calling tools), raising the stakes beyond text output.

### 🔗 [Watch Out for Your Agents! Investigating Backdoor Threats to LLM-Based Agents](https://arxiv.org/abs/2402.11208)

**Wenkai Yang, Xiaohan Bi, Yankai Lin, Sishuo Chen, Jie Zhou, Xu Sun** · *NeurIPS*, 2024

- **Technique:** a framework/taxonomy for agent backdoors, including triggers hidden in **intermediate reasoning or the environment** that alter the agent's action trajectory while keeping the final answer correct.
- **Why it matters:** formalizes agent-specific threat models (final-output vs. intermediate-step manipulation) that classic backdoor framing misses.

## 10. Defenses

### 🔗 [ONION: A Simple and Effective Defense Against Textual Backdoor Attacks](https://arxiv.org/abs/2011.10369)

**Fanchao Qi, Yangyi Chen, Mukai Li, Yasheng Wang, Zhiyuan Liu, Maosong Sun** · *EMNLP*, 2021

- **Technique:** detect outlier words via language-model perplexity; removing the inserted trigger token lowers perplexity, exposing insertion-based triggers at inference.
- **Limitation:** targets *inserted-token* triggers; weak against syntactic/style/semantic triggers (Hidden Killer, VPI) that leave no perplexity spike.

Other defense directions relevant to LLMs (covered in the main survey, mostly not LLM-specific): representation/probe-based detectors for sleeper agents, latent adversarial training to remove unknown-trigger backdoors, quality-guided instruction-data filtering (VPI's own defense), perplexity/entropy input filters, and provenance/data-integrity controls for pretraining and RAG corpora. No single defense generalizes across the trigger families above, and adaptive attacks routinely evade published defenses.

## 11. Cross-cutting takeaways

- **Every stage is poisonable:** pretraining, instruction tuning, RLHF/preference data, in-context demonstrations, code corpora, and agent memory/RAG each admit distinct backdoors.
- **Triggers are increasingly semantic:** from inserted tokens to syntax, style, topic/persona, and optimized embeddings, defeating keyword and perplexity filters.
- **Tiny budgets suffice:** 52 instruction examples (VPI), ~250 pretraining documents (Anthropic/AISI), <0.1% poison rate (AgentPoison). Absolute counts matter more than percentages.
- **Safety training is not removal:** Sleeper Agents shows backdoors can survive and even hide better after adversarial training.
- **Agents raise the stakes:** retrieval/memory backdoors bypass model-level defenses and turn into real actions (tool calls, trajectories).
- **Open problem:** reliable detection/removal of unknown-trigger and semantic backdoors in frontier-scale models, which is exactly where the UK AISI Safeguards agenda and the Goldwasser et al. undetectability result meet.

---

*Verified links (🔗) were confirmed against arXiv, official proceedings, the ACL Anthology, or the publishing organization. Unlinked entries are accurate bibliographic records; see the maintained [backdoor-learning-resources](https://github.com/THUYimingLi/backdoor-learning-resources) index or each venue's proceedings for canonical PDFs.*
