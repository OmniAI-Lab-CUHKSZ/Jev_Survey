<div align="center">

<h1>Jev-Like Models and Decision Systems: A Survey</h1>

<p><b>Typed Decisions · Model Construction · Agentic Integration · Co-Evolution</b></p>

<p>
Yiwei Gao<sup>1*</sup>, Chenhao Zhang<sup>1*</sup>, Mingcheng Ye<sup>2*</sup>, Mingxuan Li<sup>1*</sup>, Junbo Zhang<sup>1*</sup>, Jiale Chen<sup>3*</sup>, Yucheng He<sup>1*</sup>,<br>
Lizheng Zu<sup>3</sup>, Hongbin Xu<sup>4</sup>, Hehe Fan<sup>5</sup>, Baoyuan Wu<sup>1</sup>, Ming Li<sup>1†</sup>, Zheng Zhu<sup>6</sup>
</p>

<p><sub>
<sup>1</sup>The Chinese University of Hong Kong, Shenzhen &nbsp;
<sup>2</sup>Beijing Institute of Technology &nbsp;
<sup>3</sup>Shenzhen University<br>
<sup>4</sup>South China University of Technology &nbsp;
<sup>5</sup>Zhejiang University &nbsp;
<sup>6</sup>GigaAI<br>
<sup>*</sup>Equal contribution &nbsp; <sup>†</sup>Corresponding author
</sub></p>

<p>
<a href="Jev-Like_Models_and_Decision_Systems_A_Survey.pdf"><img src="https://img.shields.io/badge/Paper-PDF-B31B1B?style=for-the-badge&logo=adobeacrobatreader&logoColor=white" alt="Paper PDF"/></a>
<a href="#table-of-contents"><img src="https://img.shields.io/badge/Papers_%26_Resources-258-7C3AED?style=for-the-badge" alt="258 papers and resources"/></a>
<a href="https://github.com/OmniAI-Lab-CUHKSZ/Jev_Survey/stargazers"><img src="https://img.shields.io/github/stars/OmniAI-Lab-CUHKSZ/Jev_Survey?style=for-the-badge&logo=github&color=F5B700" alt="GitHub stars"/></a>
</p>
<p>
<img src="https://img.shields.io/badge/Coverage-Through_Oct_2026-2F5FE0?style=flat-square" alt="Coverage through October 2026"/>
<img src="https://img.shields.io/badge/Last_Update-2026--11--18-0EA5E9?style=flat-square" alt="Last update 2026-11-18"/>
<a href="https://github.com/OmniAI-Lab-CUHKSZ/Jev_Survey/pulls"><img src="https://img.shields.io/badge/PRs-Welcome-4B8278?style=flat-square" alt="PRs welcome"/></a>
</p>

<p>
<a href="#overview">Overview</a> ·
<a href="#evolution">Evolution</a> ·
<a href="#building-jev-models">Building Jev</a> ·
<a href="#datasets-and-benchmarks">Datasets &amp; Benchmarks</a> ·
<a href="#agentic-workflows">Agentic Workflows</a> ·
<a href="#domain-applications">Applications</a> ·
<a href="#future-directions">Future Directions</a> ·
<a href="#citation">Citation</a>
</p>

</div>

<p align="center">
<a href="assets/jev_teaser.pdf"><img src="assets/jev_teaser.png" width="100%" alt="Overview of the survey: learn to judge, judge to act, and learn from execution."/></a>
</p>
<p align="center"><sub><i>Overview of the survey. Jev-like models learn typed judgments from labels and teacher supervision; during execution these judgments guide action selection, with an LLM providing further reasoning when needed. Verified execution feedback is stored in memory or used to train the decision model.</i></sub></p>

<a id="news"></a>

## 📰 News

- **[2026/11/18]** 📄 The survey paper is finalized. Read the [PDF](Jev-Like_Models_and_Decision_Systems_A_Survey.pdf). The README now mirrors the paper: all figures (including the agentic-workflow taxonomy), the representative-model, training-data, and benchmark tables, and the research agenda.
- **[2026/11/10]** 🚀 We release **Jev Survey**, a curated collection of papers, models, and benchmarks on Jev and typed decision models.

---

<a id="introduction"></a>

<a id="overview"></a>

## 📖 Overview

> **Abstract.** Large language model (LLM) agents repeatedly judge which information matters and what action to take. Generating an extended response for every judgment can be costly, particularly in tasks that require frequent or rapid decisions. **Typed decision models** such as **Jev** provide an alternative by returning bounded choices, scores, or probabilities that programs can use directly. This survey reviews commercial Jev, open Jev-like models, and their use in agent systems. We organize their mechanisms and operating conditions around **decision space**, **information state**, and **computation policy**, then examine workflow functions, runtime coordination, and system improvement. Our analysis shows that a decision model's effectiveness depends on the evidence and alternatives it receives, while the benefit of collaboration depends on how judgments guide computation and execution.

The overview figure follows decision capabilities through three stages:

| Stage | What happens | Covered in |
| :--- | :--- | :--- |
| **01 Learn to Judge** | Pretrained backbones, labels, and teacher reasoning are turned into typed decisions. | [Building Jev Models](#building-jev-models) |
| **02 Judge to Act** | Typed judgments screen evidence and options, call on a reasoning LLM when needed, and drive execution. | [Agentic Workflows](#agentic-workflows) |
| **03 Learn from Execution** | Execution feedback flows into memory, training, and distillation, producing reusable capability and system improvement. | [System Improvement](#system-improvement) · [Future Directions](#future-directions) |

**Takeaways.** Computational reuse can reduce repeated work, while candidate coverage and evidence sufficiency shape which judgments a model can resolve reliably. Collaboration pays off only when additional processing addresses unresolved difficulties, and it also produces experience that can improve future decisions. We identify candidate construction, adaptive collaboration, and capability internalization as connected research directions.

<a id="typed-decision-interface"></a>

## 🧩 Typed Decision Interface

A typed decision model answers a request $r=(s,q,\tau,\Gamma)$, where $s$ is the evidence state, $q$ the question, $\tau$ the response type, and $\Gamma$ the permitted answer space. Both $q$ and $\Gamma$ are supplied at inference time, so one model can serve many changing decision tasks through a common interface. Jev exposes three text primitives, and the figure below adds `Box` for image grounding:

<table>
<tr>
<td width="50%" valign="middle">
<a href="assets/typed_judgments.pdf"><img src="assets/typed_judgments.png" width="100%" alt="Typed judgments in a customer-support scenario: Choice, Noul, Score, and Box."/></a>
</td>
<td width="50%" valign="middle">

| Primitive | Question it answers | Output |
| :--- | :--- | :--- |
| `Choice` | Which supplied option should be taken? | Probabilities over the candidate set |
| `Noul` | Is this proposition true? | Probability that it holds |
| `Score` | Where does the case fall on an ordered scale? | Distribution over rating levels |
| `Box` | Where is the object in the image? | Bounding-box coordinates |

</td>
</tr>
</table>

<p align="center"><sub><i>Typed judgments in a customer-support scenario. <code>Choice</code> selects a team, <code>Noul</code> assesses a proposition, <code>Score</code> rates urgency, and <code>Box</code> locates an object in an image. Each request specifies how the output should be interpreted.</i></sub></p>

We analyze every system in the survey along three shared dimensions:

| Dimension | Meaning | Typical failure when it is inadequate |
| :--- | :--- | :--- |
| **Decision space** $D_t$ | Available answers, task actions, and computational operations | The correct option is absent, so normalization still picks a wrong one |
| **Information state** $I_t$ | Observations, evidence, memory, and intermediate results | Overconfident judgments on missing or stale evidence |
| **Computation policy** | Combines a judgment with task context to act, acquire evidence, escalate, or verify | Extra computation spent where it cannot correct the error |

System 1 and System 2 are used here as **computational roles** rather than fixed model categories: fast judgment applies learned capabilities with limited deliberation, while deliberate reasoning adds search, analysis, and verification. Both roles can live in one model or be split across cooperating components.

<a id="table-of-contents"></a>

## 🗂️ Table of Contents

<details>
<summary><b>Explore the collection</b></summary>

- [Evolution of Decision Models](#evolution)
  - [Discriminative Prediction](#discriminative-prediction) · [Representation Matching](#representation-matching) · [LM Probability Readout](#lm-probability-readout) · [Candidate / Query Structure](#candidate-query-structure) · [Iterative / Adaptive Computation](#iterative-adaptive-computation)
- [Building Jev Models](#building-jev-models)
  - [Decision Readout](#decision-readout)
  - [Learning Methods](#learning-methods)
  - [Inference Strategies](#inference-strategies)
  - [Representative Models](#representative-models)
  - [Advantages and Limitations](#advantages-and-limitations)
- [Datasets and Benchmarks](#datasets-and-benchmarks)
  - [Training Data and Supervision](#training-data)
  - [Evaluation Benchmarks](#evaluation-benchmarks)
  - [Key Empirical Findings](#key-findings)
  - [Evaluation Limitations and Gaps](#evaluation-gaps)
- [Jev in Agentic Workflows](#agentic-workflows)
  - [Information and Memory Selection](#information-and-memory-selection)
  - [Computation and Reasoning Control](#computation-and-reasoning-control)
  - [Verification and Review](#verification-and-review)
  - [Action Selection](#action-selection)
  - [Runtime Coordination](#runtime-coordination)
  - [Decision-Guided System Improvement](#system-improvement)
- [Domain Applications](#domain-applications)
- [Challenges and Future Directions](#future-directions)
  - [Research Agenda](#research-agenda)
- [Foundations and Related Techniques](#foundations)
- [Related Surveys](#related-surveys)
- [Citation](#citation)

</details>

---

<a id="evolution"></a>

## 🌳 Evolution of Decision Models

Jev-like models grow out of five research lines that make learned prediction reusable across changing decision tasks. Classic lines run up to 2025; the post-Jev wave (from **15 Sep 2026**) is shown on a finer time axis. Click any figure in this README to open the original vector PDF.

<p align="center">
<a href="assets/decision_model_tree.pdf"><img src="assets/decision_model_tree.png" width="100%" alt="Evolution of decision models along five research lines: discriminative prediction, representation matching, LM probability readout, candidate/query structure, and iterative/adaptive computation."/></a>
</p>
<p align="center"><sub><i>Timeline of representative decision models and their technical foundations, organized into five directions. The vertical band marks Jev's introduction on 15 September 2026, separating earlier foundations from subsequent developments.</i></sub></p>

<a id="discriminative-prediction"></a>

### 1. Discriminative Prediction

From fixed-label classifiers and NLI-based zero-shot classification to encoders that read candidate descriptions together with the evidence.

- **`GitHub 2026`** Laya. [[code](https://github.com/NandhaKishorM/laya)]
- **`GitHub 2026`** Kev. [[code](https://github.com/jaredpalmer/kev)]
- **`Hugging Face 2026`** GLiNER2.5-Decide. [[model](https://huggingface.co/fastino/GLiNER2.5-Decide)]
- **`arXiv 2026`** Chinese-Jev: Bringing System One Model to Chinese-Language Tasks. [[paper](https://arxiv.org/abs/2609.36965v1)] [[code](https://github.com/gulucaptain/Chinese-Jev)]
- **`Resource 2026`** The GLiDE Family: Decision Models from 0.4B to 27B. [[link](https://fastino.ai/blog/glide-decision-model-family)]
- **`Resource 2026`** Open d1: Edge decision models for text, vision, and audio. [[link](https://www.liquid.ai/blog/d1-open)]
- **`Resource 2026`** Microsoft-Decision-1. [[link](https://labs.ai.azure.com/innovations/microsoft-decision-1/)]
- **`arXiv 2026`** FunDM: Cheap & Label-Free Decision Models from Frozen Language Models. [[paper](https://arxiv.org/abs/2610.17175v1)]
- **`arXiv 2026`** Qwen-Audio-Jev: A Unified Interface for Audio Decisions. [[paper](https://arxiv.org/abs/2610.21614v2)]
- **`arXiv 2026`** UniJev: Separating Accuracy from Calibration in Trimodal System One Decision Models. [[paper](https://arxiv.org/abs/2610.25784v1)]
- **`arXiv 2026`** SatOps-Decision: Low-Latency Discriminative Decisions for Onboard Satellite Communication Operations. [[paper](https://arxiv.org/abs/2610.38697)]
- **`arXiv 2026`** Zuse: A Decision Model for European Languages. [[paper](https://arxiv.org/abs/2611.06238)]
- **`arXiv 2026`** Towards Foundation Decision Models: Drex. [[paper](https://arxiv.org/abs/2611.07854)]
- **`arXiv 2025`** GLiClass: Generalist Lightweight Model for Sequence Classification Tasks. [[paper](https://arxiv.org/abs/2508.07662)]
- **`COLING 2020`** Task-Aware Representation of Sentences for Generic Text Classification. [[paper](https://aclanthology.org/2020.coling-main.285/)]
- **`NAACL 2019`** BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding. [[paper](https://aclanthology.org/N19-1423/)]
- **`EMNLP 2019`** Benchmarking Zero-shot Text Classification: Datasets, Evaluation and Entailment Approach. [[paper](https://aclanthology.org/D19-1404/)]
- **`Book 2006`** Pattern Recognition and Machine Learning. [[link](https://www.microsoft.com/en-us/research/publication/pattern-recognition-machine-learning/)]

<a id="representation-matching"></a>

### 2. Representation Matching

Requests and candidates are embedded separately and compared, enabling cached candidate representations and order-invariant scoring.

- **`GitHub 2026`** Contrastive Language Models (CLM). [[code](https://github.com/Contrastive-LM/CLM)]
- **`arXiv 2026`** Dyad: Extending Large Language Models with Native Typed Decision-Making. [[paper](https://arxiv.org/abs/2609.36116)]
- **`GitHub 2026`** JevEmbed. [[code](https://github.com/HITsz-TMG/JevEmbed)]
- **`arXiv 2026`** MetaEncoder: Exploring the Limit of Bi-Encoders for Multimodal System One Decision Making with Natural Language Interface. [[paper](https://arxiv.org/abs/2610.11316v1)] [[code](https://github.com/facebook/meta-encoder-eval)]
- **`arXiv 2026`** Jeva: System One Models from First Principles. [[paper](https://arxiv.org/abs/2610.18773v2)] [[code](https://github.com/LiQiiiii/Jeva)]
- **`arXiv 2026`** RubriX: A Fast, Fine-Tunable Non-Autoregressive Rubric Judge. [[paper](https://arxiv.org/abs/2610.34875v1)]
- **`arXiv 2026`** Every Pixel Is an Option: A System One Model for Medical Ultrasound Biometry. [[paper](https://arxiv.org/abs/2611.03360)]
- **`ICML 2021`** Learning Transferable Visual Models From Natural Language Supervision. [[paper](https://proceedings.mlr.press/v139/radford21a.html)]
- **`EMNLP 2020`** Dense Passage Retrieval for Open-Domain Question Answering. [[paper](https://aclanthology.org/2020.emnlp-main.550/)]
- **`EMNLP 2019`** Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks. [[paper](https://aclanthology.org/D19-1410/)]
- **`CIKM 2013`** Learning Deep Structured Semantic Models for Web Search using Clickthrough Data. [[link](https://www.microsoft.com/en-us/research/publication/learning-deep-structured-semantic-models-for-web-search-using-clickthrough-data/)]

<a id="lm-probability-readout"></a>

### 3. LM Probability Readout

Candidate scores are read from the answer-token likelihoods of existing LLMs, with or without light adaptation.

- **`Resource 2026`** OpenJev / SemIf: Semantic If On-Device. [[link](https://openjev.com/)]
- **`GitHub 2026`** Nimble. [[code](https://github.com/bespokelabsai/nimble)]
- **`GitHub 2026`** Decider. [[code](https://github.com/Mapika/decider)]
- **`GitHub 2026`** Tev1. [[code](https://github.com/togethercomputer/tev1)]
- **`arXiv 2026`** Koa-decision: Fast and Consistent Structured Decision Making with Generative LLMs. [[paper](https://arxiv.org/abs/2609.36115v2)]
- **`arXiv 2026`** AnyJev Technical Report. [[paper](https://arxiv.org/abs/2610.00831v1)] [[code](https://github.com/nokia-applied-research/AnyJev)]
- **`arXiv 2026`** LLM-as-Jev: LLMs Are Already Jev-Style Decision Models – When and How to Fine-Tune Them. [[paper](https://arxiv.org/abs/2610.02076v3)]
- **`arXiv 2026`** Releasing the Inner Jev: A Simple Recipe for Open Decision Models. [[paper](https://arxiv.org/abs/2610.17205v1)]
- **`EMNLP 2023`** G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment. [[paper](https://aclanthology.org/2023.emnlp-main.153/)]
- **`arXiv 2022`** Language Models (Mostly) Know What They Know. [[paper](https://arxiv.org/abs/2207.05221)]
- **`NeurIPS 2020`** Language Models are Few-Shot Learners. [[paper](https://proceedings.neurips.cc/paper/2020/hash/1457c0d6bfcb4967418bfb8ac142f64a-Abstract.html)]

<a id="candidate-query-structure"></a>

### 4. Candidate / Query Structure

Architectures that make the comparison between options explicit, isolate candidate scoring, or share one evidence encoding across many questions.

- **`arXiv 2026`** Visual Jev: Accurate and Efficient Decisions from Shared Visual Context. [[paper](https://arxiv.org/abs/2609.25845v1)] [[code](https://github.com/guanxuyu-sv/Visual-Jev)]
- **`Hugging Face 2026`** Clef. [[model](https://huggingface.co/Cloudflare/clef)]
- **`GitHub 2026`** FRIDA-Decisions. [[code](https://github.com/ai-forever/FRIDA-Decisions)]
- **`arXiv 2026`** VideoJev: System One Model for Video Understanding and Interaction. [[paper](https://arxiv.org/abs/2610.18615v1)] [[code](https://github.com/MCG-NJU/VideoJev)]
- **`arXiv 2026`** Verdict: Encode Once, Decide Many. [[paper](https://arxiv.org/abs/2610.20778v1)]
- **`arXiv 2026`** Prism-1: Position- and Rival-Independent Scoring for System One Models. [[paper](https://arxiv.org/abs/2610.22066v2)]
- **`arXiv 2026`** Narrow Decision Models: A System-1 Architecture for Calibrated, Typed Decisions. [[paper](https://arxiv.org/abs/2610.25553v1)]
- **`Hugging Face 2026`** d3-turbo. [[model](https://huggingface.co/vllm-sr/d3-turbo)]
- **`Resource 2026`** Mammoth-1 and Elephant-1: Frontier decision models. [[link](https://labs.baseten.co/articles/mammoth-1-and-elephant-1-frontier-decision-models)]
- **`ICML 2019`** Set Transformer: A Framework for Attention-based Permutation-Invariant Neural Networks. [[paper](https://proceedings.mlr.press/v97/lee19d.html)]
- **`NeurIPS 2017`** Deep Sets. [[paper](https://proceedings.neurips.cc/paper_files/paper/2017/hash/f22e4747da1aa27e363d86d40ff442fe-Abstract.html)]
- **`NeurIPS 2015`** Pointer Networks. [[paper](https://proceedings.neurips.cc/paper_files/paper/2015/hash/29921001f2f04bd3baee84a12e98098f-Abstract.html)]

<a id="iterative-adaptive-computation"></a>

### 5. Iterative / Adaptive Computation

Mechanisms that control how much processing precedes a judgment, from adaptive computation time to looped and fast–slow decision models.

- **`arXiv 2026`** SanSi: A Looped Typed Decision Model for System 1.5 Thinking. [[paper](https://arxiv.org/abs/2610.07730v2)]
- **`arXiv 2026`** JevCascade: Readout Stability and Inference-Time Compute Allocation for Prefill-Only Decision Models. [[paper](https://arxiv.org/abs/2610.07716v2)] [[code](https://github.com/rlisml/jev-cascade)]
- **`arXiv 2026`** System Switch: When Should a Fast Decision Model Stop and Think? [[paper](https://arxiv.org/abs/2610.09683v1)] [[code](https://github.com/dexmac221/system-switch)]
- **`arXiv 2026`** StartLux-Decision: AutoThink Decision Models with Reinforcement Learning for Calibrated Deliberative Decisions. [[paper](https://arxiv.org/abs/2610.17309v2)] [[code](https://github.com/StartLuxLabs/StartLux-Prism)]
- **`Hugging Face 2026`** GEV-26B-Decide. [[model](https://huggingface.co/autotrust/GEV-26B-Decide)]
- **`arXiv 2026`** LEV: Thinking Fast and Deep with Looped Decision Models. [[paper](https://arxiv.org/abs/2610.20951)]
- **`arXiv 2026`** System X: An LLM Decision Model That Replies from Its Own KV Cache. [[paper](https://arxiv.org/abs/2610.25456v1)]
- **`arXiv 2026`** P(Yes): 15 Improvements via Test-Time Scaling of Decision Models. [[paper](https://arxiv.org/abs/2610.25474v1)] [[code](https://github.com/tear-labs/pyes-search)]
- **`Resource 2026`** The GLiDE Family: Decision Models from 0.4B to 27B. [[link](https://fastino.ai/blog/glide-decision-model-family)]
- **`arXiv 2026`** Extending Reasoning in Typed Decision Models via Looping. [[paper](https://arxiv.org/abs/2611.07736)]
- **`arXiv 2021`** PonderNet: Learning to Ponder. [[paper](https://arxiv.org/abs/2107.05407v2)]
- **`ICLR 2019`** Universal Transformers. [[paper](https://openreview.net/forum?id=HyzdRiR9Y7)]
- **`arXiv 2016`** Adaptive Computation Time for Recurrent Neural Networks. [[paper](https://arxiv.org/abs/1603.08983v6)]

<a id="building-jev-models"></a>

## 🧠 Building Jev Models

A pretrained language model already encodes much of what a judgment needs. Turning it into a Jev-like model requires a **readout** that maps representations to typed decisions, **learning signals** that shape those decisions, and an **inference strategy** that decides how much computation precedes the output.

<p align="center">
<a href="assets/model_architectures.pdf"><img src="assets/model_architectures.png" width="100%" alt="Representative architectures for decision models."/></a>
</p>
<p align="center"><sub><i>Representative architectures for decision models. (a) An LLM scores candidate label tokens through its vocabulary head. (b) An LLM uses a dedicated decision head. (c) An encoder-based model uses a decision head. (d) Separate encoders represent the request and candidate descriptions for similarity scoring. The lower panel illustrates an optional multimodal input configuration.</i></sub></p>

<a id="decision-readout"></a>

### Decision Readout

#### Vocabulary-Based Readout

Assign each candidate a code and score it with the LM vocabulary head, correcting for label and order priors.

- **`arXiv 2026`** LLM-as-Jev: LLMs Are Already Jev-Style Decision Models – When and How to Fine-Tune Them. [[paper](https://arxiv.org/abs/2610.02076v3)]
- **`arXiv 2026`** Koa-decision: Fast and Consistent Structured Decision Making with Generative LLMs. [[paper](https://arxiv.org/abs/2609.36115v2)]
- **`arXiv 2026`** AnyJev Technical Report. [[paper](https://arxiv.org/abs/2610.00831v1)] [[code](https://github.com/nokia-applied-research/AnyJev)]

#### Decision-Specific Scorers

A dedicated head scores candidates from backbone hidden states, either jointly encoded or per option.

- **`Hugging Face 2026`** Laya: Official Model Card. [[model](https://huggingface.co/convaiinnovations/laya)]
- **`Hugging Face 2026`** Kev-4B (Kev 1.0): Official Model Card. [[model](https://huggingface.co/jaredpalmer/kev-4b)]
- **`arXiv 2026`** Joint or Per-Option? Compact Encoders for Closed-Set Decisions from 17 Million to 1 Billion Parameters. [[paper](https://arxiv.org/abs/2610.30970v1)] [[code](https://github.com/tardellirs/CrivO-decision-encoders)]
- **`arXiv 2025`** GLiClass: Generalist Lightweight Model for Sequence Classification Tasks. [[paper](https://arxiv.org/abs/2508.07662)]

#### Candidate Interactions

Whether a candidate's score may depend on the other candidates: isolated scoring versus set-aware scoring with a null outcome.

- **`arXiv 2026`** Prism-1: Position- and Rival-Independent Scoring for System One Models. [[paper](https://arxiv.org/abs/2610.22066v2)]
- **`arXiv 2026`** Jeva: System One Models from First Principles. [[paper](https://arxiv.org/abs/2610.18773v2)] [[code](https://github.com/LiQiiiii/Jeva)]

#### Shared Computation

Reusing cached candidate or evidence encodings across questions, modalities, and between decision and generation.

- **`arXiv 2026`** Dyad: Extending Large Language Models with Native Typed Decision-Making. [[paper](https://arxiv.org/abs/2609.36116)]
- **`arXiv 2026`** Seeing the Options: What Should a Visual Candidate See? [[paper](https://arxiv.org/abs/2610.33702)]
- **`arXiv 2026`** Verdict: Encode Once, Decide Many. [[paper](https://arxiv.org/abs/2610.20778v1)]
- **`arXiv 2026`** Visual Jev: Accurate and Efficient Decisions from Shared Visual Context. [[paper](https://arxiv.org/abs/2609.25845v1)] [[code](https://github.com/guanxuyu-sv/Visual-Jev)]
- **`arXiv 2026`** VideoJev: System One Model for Video Understanding and Interaction. [[paper](https://arxiv.org/abs/2610.18615v1)] [[code](https://github.com/MCG-NJU/VideoJev)]
- **`arXiv 2026`** Intern-Decision: Efficient Multimodal Decision Making with Autoregressive Masked Language Modeling. [[paper](https://arxiv.org/abs/2610.22131v1)] [[code](https://github.com/InternLM/Intern-Decision)]
- **`arXiv 2026`** From Text Decisions to Pixels: An Study of Jev-Style Visual Choice Model. [[paper](https://arxiv.org/abs/2609.29283v1)]
- **`arXiv 2026`** MetaEncoder: Exploring the Limit of Bi-Encoders for Multimodal System One Decision Making with Natural Language Interface. [[paper](https://arxiv.org/abs/2610.11316v1)] [[code](https://github.com/facebook/meta-encoder-eval)]
- **`arXiv 2026`** Batched Speech Decisions Without Decoding: Single-Token Supervision Lets a Frozen LLM Hear Beyond the Transcript. [[paper](https://arxiv.org/abs/2610.02638v1)] [[code](https://github.com/adventists-ai/duplexjev)]
- **`arXiv 2026`** TimesJev: Thinking Fast on Time Series with System 1 Sufficiency. [[paper](https://arxiv.org/abs/2610.26002v1)]
- **`arXiv 2026`** UniJev: Separating Accuracy from Calibration in Trimodal System One Decision Models. [[paper](https://arxiv.org/abs/2610.25784v1)]
- **`arXiv 2026`** System X: An LLM Decision Model That Replies from Its Own KV Cache. [[paper](https://arxiv.org/abs/2610.25456v1)]
- **`arXiv 2026`** JevRec: Linguistic Encoding with Jev-like Decision Modeling for LLM-based Generative Recommendation. [[paper](https://arxiv.org/abs/2610.36937)]
- **`EMNLP 2026`** Fast-Slow Thinking RM: Efficient Integration of Scalar and Generative Reward Models. [[paper](https://aclanthology.org/2026.emnlp-main.1526/)]

<a id="learning-methods"></a>

### Learning Methods

#### Training-Free Use of Pretrained Models

Read candidate likelihoods from frozen weights, calibrate with a fitted temperature, or shortlist and compare.

- **`arXiv 2026`** LLM-as-Jev: LLMs Are Already Jev-Style Decision Models – When and How to Fine-Tune Them. [[paper](https://arxiv.org/abs/2610.02076v3)]
- **`arXiv 2026`** AnyJev Technical Report. [[paper](https://arxiv.org/abs/2610.00831v1)] [[code](https://github.com/nokia-applied-research/AnyJev)]
- **`arXiv 2026`** Consider, Then Choose: Order-Invariant Decisions in System One Models. [[paper](https://arxiv.org/abs/2610.18233v1)]

#### Choosing What to Train

Update the readout, the backbone (e.g., via LoRA), or only one side of a dual encoder.

- **`arXiv 2026`** Releasing the Inner Jev: A Simple Recipe for Open Decision Models. [[paper](https://arxiv.org/abs/2610.17205v1)]
- **`arXiv 2026`** Our Own Jev: Typed Decisions in One Forward Pass, from an In-House Judge to Open LoRA Models. [[paper](https://arxiv.org/abs/2610.29052v1)]
- **`arXiv 2026`** Dyad: Extending Large Language Models with Native Typed Decision-Making. [[paper](https://arxiv.org/abs/2609.36116)]

#### Labels and Teacher Judgments

Gold labels, ordinal targets, soft teacher distributions, counterfactual edits, order-consistency, and on-policy recovery states.

- **`arXiv 2026`** Visual Jev: Accurate and Efficient Decisions from Shared Visual Context. [[paper](https://arxiv.org/abs/2609.25845v1)] [[code](https://github.com/guanxuyu-sv/Visual-Jev)]
- **`arXiv 2026`** RubriX: A Fast, Fine-Tunable Non-Autoregressive Rubric Judge. [[paper](https://arxiv.org/abs/2610.34875v1)]
- **`arXiv 2026`** General Decision Models: Benchmarking and Insights Beyond Jev. [[paper](https://arxiv.org/abs/2610.03935v2)]
- **`arXiv 2026`** PACT: Pairwise-Anchored Calibrated Tuning for Single-Token Typed Decisions. [[paper](https://arxiv.org/abs/2609.35865)] [[code](https://github.com/BennyLinntu/PACT-Pairwise-Anchored-Calibrated-Tuning-for-Single-Token-Typed-Decisions)]
- **`arXiv 2026`** AudioJev: Direct Audio Decisions with Order-Calibrated Probabilities. [[paper](https://arxiv.org/abs/2610.01293v1)] [[code](https://github.com/SihanLv/AudioJev-Inference)]
- **`arXiv 2026`** JevAgentHarness: Benchmarking, Deploying and Training Decision Models That Share Control with LLMs. [[paper](https://arxiv.org/abs/2610.21412v1)] [[code](https://github.com/JevAgentHarnesS/JevAgentHarnEsS)]
- **`arXiv 2024`** Distilling System 2 into System 1. [[paper](https://arxiv.org/abs/2407.06023v3)]
- **`ICLR 2016`** Policy Distillation. [[paper](https://arxiv.org/abs/1511.06295)]
- **`arXiv 2015`** Distilling the Knowledge in a Neural Network. [[paper](https://arxiv.org/abs/1503.02531v1)]
- **`AISTATS 2011`** A Reduction of Imitation Learning and Structured Prediction to No-Regret Online Learning. [[paper](https://proceedings.mlr.press/v15/ross11a.html)]

#### Reasoning Traces

Distilling explicit or latent reasoning into decision capability, from curriculum and masked self-distillation to on-policy distillation.

- **`arXiv 2026`** CIRF: Tokenizing Chain-of-Thoughts into Reusable Functional Units for Efficient Latent Reasoning in Large Language Models. [[paper](https://arxiv.org/abs/2605.28292v1)]
- **`arXiv 2026`** Tailoring the Curriculum: Student-Centered Reasoning Distillation via Dynamic Data-Model Compatibility. [[paper](https://arxiv.org/abs/2605.29229v1)]
- **`arXiv 2026`** EKLAV: Elicited Key CoT Learning via Answer only superVision. [[paper](https://arxiv.org/abs/2607.25182v2)]
- **`arXiv 2026`** Masked Self-Distillation: Internalizing the Chain-of-Thought in Language Models. [[paper](https://arxiv.org/abs/2607.22629v3)]
- **`ICML 2026`** Self-Distilled Reasoner: On-Policy Self-Distillation for Large Language Models. [[paper](https://proceedings.mlr.press/v306/zhao26be.html)] [[code](https://github.com/siyan-zhao/OPSD)]
- **`arXiv 2026`** Backtracking When It Strays: Mitigating Dual Exposure Biases in LLM Reasoning Distillation. [[paper](https://arxiv.org/abs/2605.19433v1)] [[code](https://github.com/wangbing1416/MOTAB)]
- **`SIGIR 2026`** Internalizing Explicit Reasoning into Latent Space for Dense Retrieval. [[paper](https://doi.org/10.1145/3805712.3809575)] [[code](https://github.com/RUC-NLPIR/LaSER)]
- **`ACL 2026`** TInR: Exploring Tool-Internalized Reasoning in Large Language Models. [[paper](https://aclanthology.org/2026.acl-long.2077/)]
- **`arXiv 2026`** Internalizing LLM Reasoning via Discovery and Replay of Latent Actions. [[paper](https://arxiv.org/abs/2602.04925v1)] [[code](https://github.com/sZnnzs/LLM-Latent-Action)]
- **`EMNLP 2025`** CODI: Compressing Chain-of-Thought into Continuous Space via Self-Distillation. [[paper](https://aclanthology.org/2025.emnlp-main.36/)] [[code](https://github.com/zhenyi4/codi)]
- **`arXiv 2024`** From Explicit CoT to Implicit CoT: Learning to Internalize CoT Step by Step. [[paper](https://arxiv.org/abs/2405.14838v1)] [[code](https://github.com/da03/Internalize_CoT_Step_by_Step)]
- **`EMNLP 2024`** Teaching Small Language Models Reasoning through Counterfactual Distillation. [[paper](https://aclanthology.org/2024.emnlp-main.333/)]
- **`ACL Findings 2023`** Distilling Step-by-Step! Outperforming Larger Language Models with Less Training Data and Smaller Model Sizes. [[paper](https://aclanthology.org/2023.findings-acl.507/)] [[code](https://github.com/google-research/distilling-step-by-step)]
- **`ACL 2023`** Large Language Models Are Reasoning Teachers. [[paper](https://aclanthology.org/2023.acl-long.830/)]
- **`ACL Findings 2023`** Distilling Reasoning Capabilities into Smaller Language Models. [[paper](https://aclanthology.org/2023.findings-acl.441/)]
- **`arXiv 2023`** Implicit Chain of Thought Reasoning via Knowledge Distillation. [[paper](https://arxiv.org/abs/2311.01460v1)]

#### Task Feedback

Using decision quality as a reward for reasoning, and decision signals as supervision for other learners.

- **`arXiv 2026`** Beyond Outcome Rewards: Post-Rationale Distribution Optimization for Selective Prediction in Reasoning Models. [[paper](https://arxiv.org/abs/2609.38850)] [[code](https://github.com/ZimmyGao/openjev-rlcd)]
- **`arXiv 2026`** StartLux-Decision: AutoThink Decision Models with Reinforcement Learning for Calibrated Deliberative Decisions. [[paper](https://arxiv.org/abs/2610.17309v2)] [[code](https://github.com/StartLuxLabs/StartLux-Prism)]
- **`arXiv 2026`** Can Jev be Your Q or Policy in Reinforcement Learning? [[paper](https://arxiv.org/abs/2610.11692v1)]
- **`arXiv 2026`** From Probabilities to Decisions: Search and Multi-Teacher Distillation with Jev. [[paper](https://arxiv.org/abs/2610.09188v1)]
- **`ICLR 2024`** Let's Verify Step by Step. [[paper](https://proceedings.iclr.cc/paper_files/paper/2024/hash/aca97732e30bcf1303bc22ac3924fd16-Abstract-Conference.html)]
- **`NeurIPS 2017`** Thinking Fast and Slow with Deep Learning and Tree Search. [[paper](https://proceedings.neurips.cc/paper/2017/hash/d8e1344e27a5b08cdfd5d027d9b8d6de-Abstract.html)]

<a id="inference-strategies"></a>

### Inference Strategies

After encoding a request, a model can read out a decision directly, follow a preset reasoning procedure, or adapt its computation before or during reasoning.

<table>
<tr>
<td width="42%" valign="middle">
<a href="assets/inference_strategies.pdf"><img src="assets/inference_strategies.png" width="100%" alt="Four inference strategies for Jev-style decision making."/></a>
</td>
<td width="58%" valign="middle">

| Strategy | Extra reasoning | When computation adapts |
| :--- | :--- | :--- |
| (a) Direct judgment | None | Never |
| (b) Fixed reasoning | Preset tokens or latent loops | Never |
| (c) Adaptive planning | Chosen per request | Before reasoning starts |
| (d) Adaptive control | Extended or stopped on the fly | During reasoning |

</td>
</tr>
</table>

<p align="center"><sub><i>Four inference strategies for Jev-style decision making, distinguished by the use of extra reasoning and the point at which computation can adapt.</i></sub></p>

#### Direct Judgment

- **`arXiv 2026`** FunDM: Cheap & Label-Free Decision Models from Frozen Language Models. [[paper](https://arxiv.org/abs/2610.17175v1)]

#### Fixed Reasoning

- **`arXiv 2026`** LLM-as-Jev: LLMs Are Already Jev-Style Decision Models – When and How to Fine-Tune Them. [[paper](https://arxiv.org/abs/2610.02076v3)]
- **`arXiv 2026`** INTACT: Inducing Latent Reasoning on a Frozen Backbone. [[paper](https://arxiv.org/abs/2610.19568v1)]
- **`arXiv 2026`** SanSi: A Looped Typed Decision Model for System 1.5 Thinking. [[paper](https://arxiv.org/abs/2610.07730v2)]
- **`ACL 2025`** SoftCoT: Soft Chain-of-Thought for Efficient Reasoning with LLMs. [[paper](https://aclanthology.org/2025.acl-long.1137/)] [[code](https://github.com/xuyige/SoftCoT)]
- **`COLM 2025`** Training Large Language Models to Reason in a Continuous Latent Space. [[paper](https://proceedings.colm.cc/paper_files/paper/2025/hash/ed032b08a8822c3635cdcd961012ce60-Abstract-Conference.html)] [[code](https://github.com/facebookresearch/coconut)]
- **`NeurIPS 2025`** Scaling up Test-Time Compute with Latent Reasoning: A Recurrent Depth Approach. [[paper](https://proceedings.neurips.cc/paper_files/paper/2025/hash/3b01972cf31e6fa0fe29e4b8b5c2a0a1-Abstract-Conference.html)]

#### Adaptive Planning Before Reasoning

- **`arXiv 2026`** StartLux-Decision: AutoThink Decision Models with Reinforcement Learning for Calibrated Deliberative Decisions. [[paper](https://arxiv.org/abs/2610.17309v2)] [[code](https://github.com/StartLuxLabs/StartLux-Prism)]
- **`Resource 2026`** System 1&2 Synergy via Dynamic Model Interpolation. [[paper](https://www.ijcai.org/proceedings/2026/668)]
- **`TNNLS 2025`** Distilling Reasoning Ability from Large Language Models with Adaptive Thinking. [[paper](https://doi.org/10.1109/TNNLS.2025.3591266)]
- **`ICLR 2025`** Scaling LLM Test-Time Compute Optimally Can be More Effective than Scaling Parameters for Reasoning. [[paper](https://proceedings.iclr.cc/paper_files/paper/2025/hash/1b623663fd9b874366f3ce019fdfdd44-Abstract-Conference.html)]

#### Adaptive Control During Reasoning

- **`arXiv 2026`** Selective Reasoning for Typed Decision Models: When to Start, Read, and Stop. [[paper](https://arxiv.org/abs/2610.19644v1)] [[code](https://github.com/binzheng-joey/selective-reasoning)]
- **`arXiv 2026`** Truncated Reasoning Under a Budget: Evaluating System-1 Models as Decision-makers. [[paper](https://arxiv.org/abs/2610.25768v2)]
- **`arXiv 2026`** SOMA: Continuous Monitoring and Intervention for System Two Reasoning with System One Models. [[paper](https://arxiv.org/abs/2610.26793v1)]
- **`arXiv 2026`** Reading Right, Acting Wrong: Diagnosing Decision Failures of System One Models in Agent Loops. [[paper](https://arxiv.org/abs/2610.18579v1)]
- **`arXiv 2026`** Judges Follow the Reference: The Derivation Gap in Verifying Tool-Call Arguments. [[paper](https://arxiv.org/abs/2610.18497v1)]
- **`arXiv 2026`** Code Owns the Simulation, Jev Owns the Evaluation. [[paper](https://arxiv.org/abs/2610.01834v2)]

<a id="representative-models"></a>

### Representative Models

Backbones and adaptation strategies of representative Jev and Jev-like models, taken from the survey. Architecture lists a main class and its mechanism; modality refers to the reviewed decision configuration.

| Model / Method | Backbone | Modality | Architecture | Readout | Training | Tuned | Inference |
| :--- | :--- | :---: | :--- | :--- | :--- | :--- | :---: |
| ***Hosted reference*** | | | | | | | |
| [Jev](https://docs.typesafe.ai/introduction) | Closed-source model | <img src="https://img.shields.io/badge/M-7D5E9E?style=flat-square" alt="M" title="Multimodal"/> | – | – | – | – | – |
| ***Vocabulary readouts and shared prediction slots*** | | | | | | | |
| [LLM-as-Jev](https://arxiv.org/abs/2610.02076v3) | Qwen3.5 / Qwen3 | <img src="https://img.shields.io/badge/T-388073?style=flat-square" alt="T" title="Text-only"/> | **LLM + vocabulary head**<br><sub>Scores candidate codes.</sub> | Token likelihoods | None | Frozen | <img src="https://img.shields.io/badge/D-386394?style=flat-square" alt="D" title="Direct judgment"/> <img src="https://img.shields.io/badge/F-388073?style=flat-square" alt="F" title="Fixed reasoning"/> |
| [AnyJev](https://arxiv.org/abs/2610.00831v1) | Qwen3 / Qwen2.5 | <img src="https://img.shields.io/badge/T-388073?style=flat-square" alt="T" title="Text-only"/> | **LLM + vocabulary head**<br><sub>Corrects label and order effects.</sub> | Token scores | Calibration | Temp. | <img src="https://img.shields.io/badge/D-386394?style=flat-square" alt="D" title="Direct judgment"/> |
| [Releasing the Inner Jev](https://arxiv.org/abs/2610.17205v1) | Gemma 4 Instruct | <img src="https://img.shields.io/badge/T-388073?style=flat-square" alt="T" title="Text-only"/> | **LLM + vocabulary head**<br><sub>Corrects answer-letter logits.</sub> | Token logits | SFT | LoRA + Head | <img src="https://img.shields.io/badge/D-386394?style=flat-square" alt="D" title="Direct judgment"/> |
| [Intern-Decision](https://arxiv.org/abs/2610.22131v1) | Qwen3.5 | <img src="https://img.shields.io/badge/M-7D5E9E?style=flat-square" alt="M" title="Multimodal"/> | **LLM + vocabulary head**<br><sub>Uses JSON prediction slots.</sub> | Token logits | SFT | –¹ | <img src="https://img.shields.io/badge/D-386394?style=flat-square" alt="D" title="Direct judgment"/> |
| ***Dedicated scorers and separate representations*** | | | | | | | |
| [Laya](https://huggingface.co/convaiinnovations/laya) | ModernBERT | <img src="https://img.shields.io/badge/T-388073?style=flat-square" alt="T" title="Text-only"/> | **Encoder + decision head**<br><sub>Adds two transformer head layers.</sub> | Head scores | RLCD | Full | <img src="https://img.shields.io/badge/D-386394?style=flat-square" alt="D" title="Direct judgment"/> |
| [Kev-4B](https://huggingface.co/jaredpalmer/kev-4b) | Qwen3.5 Base | <img src="https://img.shields.io/badge/T-388073?style=flat-square" alt="T" title="Text-only"/> | **LLM + decision head**<br><sub>Compares question and option states.</sub> | Similarity | SFT | LoRA + Head | <img src="https://img.shields.io/badge/D-386394?style=flat-square" alt="D" title="Direct judgment"/> |
| [GLiClass](https://arxiv.org/abs/2508.07662) | DeBERTa-v3 | <img src="https://img.shields.io/badge/T-388073?style=flat-square" alt="T" title="Text-only"/> | **Encoder + decision head**<br><sub>Encodes text and labels jointly.</sub> | Similarity / MLP | Staged | LoRA² | <img src="https://img.shields.io/badge/D-386394?style=flat-square" alt="D" title="Direct judgment"/> |
| [Jeva](https://arxiv.org/abs/2610.18773v2) | Qwen3.5 | <img src="https://img.shields.io/badge/T-388073?style=flat-square" alt="T" title="Text-only"/> | **Separate encoders**<br><sub>Encodes request and options separately.</sub> | Set-conditioned scores | SFT | LoRA + Head | <img src="https://img.shields.io/badge/D-386394?style=flat-square" alt="D" title="Direct judgment"/> |
| ***Recurrent, adaptive, and visual computation*** | | | | | | | |
| [SanSi](https://arxiv.org/abs/2610.07730v2) | Ouro Base | <img src="https://img.shields.io/badge/T-388073?style=flat-square" alt="T" title="Text-only"/> | **Recurrent LM**<br><sub>Scores candidates after each loop.</sub> | Token logits | SFT | LoRA + Head | <img src="https://img.shields.io/badge/C-9E7540?style=flat-square" alt="C" title="Adaptive control during reasoning"/> |
| [StartLux-Decision](https://arxiv.org/abs/2610.17309v2) | Qwen3.5 family† | <img src="https://img.shields.io/badge/T-388073?style=flat-square" alt="T" title="Text-only"/> | **LLM + vocabulary head**<br><sub>Combines initial and reasoned predictions.</sub> | Token probabilities | SFT → RL | – | <img src="https://img.shields.io/badge/B-7D5E9E?style=flat-square" alt="B" title="Adaptive planning before reasoning"/> |
| [Visual Jev](https://arxiv.org/abs/2609.25845v1) | Qwen3-VL Instruct | <img src="https://img.shields.io/badge/M-7D5E9E?style=flat-square" alt="M" title="Multimodal"/> | **LLM + vocabulary head**<br><sub>Shares visual context; isolates questions.</sub> | Token probabilities | SFT | LoRA | <img src="https://img.shields.io/badge/D-386394?style=flat-square" alt="D" title="Direct judgment"/> |

<sub>**Modality**: <img src="https://img.shields.io/badge/T-388073?style=flat-square" alt="T" title="Text-only"/> text-only; <img src="https://img.shields.io/badge/M-7D5E9E?style=flat-square" alt="M" title="Multimodal"/> multimodal. **Inference**: <img src="https://img.shields.io/badge/D-386394?style=flat-square" alt="D" title="Direct judgment"/> direct judgment; <img src="https://img.shields.io/badge/F-388073?style=flat-square" alt="F" title="Fixed reasoning"/> fixed reasoning; <img src="https://img.shields.io/badge/B-7D5E9E?style=flat-square" alt="B" title="Adaptive planning before reasoning"/> adaptive planning before reasoning; <img src="https://img.shields.io/badge/C-9E7540?style=flat-square" alt="C" title="Adaptive control during reasoning"/> adaptive control during reasoning.<br>**Training**: SFT supervised fine-tuning; RL reinforcement learning; RLCD reinforcement learning for calibrated decisions. **Tuned**: Full full-parameter; LoRA low-rank adapters; Head decision/readout head; Frozen no updates; Temp. output temperature only; – scope not stated.<br>¹ Intern-Decision tunes the language backbone without specifying full or parameter-efficient tuning. ² GLiClass uses staged encoder/scorer training followed by LoRA. † The released 4B configuration uses the Qwen3.5 architecture; the initial checkpoint is not specified.</sub>

<a id="advantages-and-limitations"></a>

<a id="reliability-and-limitations"></a>

### Advantages and Limitations

| | Summary |
| :--- | :--- |
| ⚡ **Computational cost** | Returning a judgment without an extended response cuts output tokens, and shared context encoding or cached candidates avoid repeated work. Savings depend on architecture and workload; extra reasoning and repeated judgments add overhead, so a faster single decision does not always make the whole workflow faster. |
| 🎯 **Judgment quality and reliability** | Every judgment is made inside a specified answer space. `Choice` must distribute mass even when several or no candidates are correct; `Score` can collapse onto a few levels. Order, labels, and candidate composition can shift predictions; confidence thresholds may not transfer across tasks or locales; and missing evidence can produce confident but unsupported decisions. |
| 🧭 **Suitable uses** | Frequent, bounded judgments with sufficient evidence and suitable candidates, such as classification, ranking, verification, and action selection. When options are incomplete, evidence is missing, or further reasoning is needed, a generative model or another component should construct alternatives, acquire information, or reason first. |

- **`arXiv 2026`** When Multiple Choices Are Correct: Asymmetric Probability Allocation in Jev and Other Decision Models. [[paper](https://arxiv.org/abs/2610.29217v1)]
- **`arXiv 2026`** When the Right Answer Is Missing: An Arithmetic-Dependent Rejection Bottleneck in Jev. [[paper](https://arxiv.org/abs/2609.39496v1)]
- **`arXiv 2026`** More Choices, Fewer Decisions: Ordinal-Scale Bias in JEV-like Direct-Decision Models. [[paper](https://arxiv.org/abs/2609.38827v2)]
- **`arXiv 2026`** Understanding Jev-Style System One Models: Does Structure Reduce Selection Bias? [[paper](https://arxiv.org/abs/2610.35826v2)]
- **`arXiv 2026`** Labels Override Definitions in Jev-Style Typed Decision Models. [[paper](https://arxiv.org/abs/2610.02586v1)]
- **`arXiv 2026`** Does a Hosted Decision Model Respect Irrelevant Alternatives? A Pre-Registered Test of Option-Set Dependence in Jev. [[paper](https://arxiv.org/abs/2610.16655v1)]
- **`arXiv 2026`** AudioJev: Direct Audio Decisions with Order-Calibrated Probabilities. [[paper](https://arxiv.org/abs/2610.01293v1)] [[code](https://github.com/SihanLv/AudioJev-Inference)]
- **`arXiv 2026`** Beyond Calibration: Do a Typed-Decision Model's Probabilities Obey the Probability Axioms? [[paper](https://arxiv.org/abs/2609.33209v1)] [[code](https://github.com/bro789/typed-decision-coherence)]
- **`arXiv 2026`** Jev thinks "I don't know'', but doesn't say it: Introducing Sys1Cal-v1 Dataset for Probability Calibration. [[paper](https://arxiv.org/abs/2609.35342v1)] [[code](https://github.com/little-g-ai/Sys1Cal-v1)]
- **`arXiv 2026`** Probability Contracts: Accuracy, Coherence, and Decisions Across LLM Interfaces. [[paper](https://arxiv.org/abs/2609.37470v2)]
- **`arXiv 2026`** Benchmarking candidate coverage and rejection policy transfer in typed decision models. [[paper](https://arxiv.org/abs/2610.03387v2)] [[code](https://github.com/luckykevvv/Decision_Model_Benchmark)]
- **`arXiv 2026`** Confidence Is Not a Promise: A Reference Audit of Typed Decision Models. [[paper](https://arxiv.org/abs/2610.36311v1)]
- **`arXiv 2026`** Teacher Supervision, Fixed Representations and Calibration: Exact Risk Calculations for Candidate-Scoring Decision Models. [[paper](https://arxiv.org/abs/2610.26797v1)]
- **`arXiv 2026`** UniJev: Separating Accuracy from Calibration in Trimodal System One Decision Models. [[paper](https://arxiv.org/abs/2610.25784v1)]
- **`arXiv 2026`** SHIFT-JEV: Certified Abstention and Gain-Gated Continual Adaptation for LLM-Based Decision Models under Distribution Shift. [[paper](https://arxiv.org/abs/2610.34329v1)]
- **`arXiv 2026`** Beyond Answer Confidence: A Controlled Audit of Self-Knowledge in a Black-Box Decision Model. [[paper](https://arxiv.org/abs/2610.01006v1)] [[code](https://github.com/Syntheme/beyond-answer-confidence)]
- **`arXiv 2026`** Can JEV Support Clinical Trial Criteria Assessment? [[paper](https://arxiv.org/abs/2610.35721v1)]
- **`arXiv 2026`** What Can't Jev Decide? Probing the Limits of One-Pass Decision Models. [[paper](https://arxiv.org/abs/2610.24929v1)]
- **`arXiv 2026`** Judges Follow the Reference: The Derivation Gap in Verifying Tool-Call Arguments. [[paper](https://arxiv.org/abs/2610.18497v1)]
- **`arXiv 2026`** JevAdvBench: A Benchmark and Black-Box Attacks for Reinforcement Learning for Calibrated Decisions Models. [[paper](https://arxiv.org/abs/2609.31142v2)] [[code](https://github.com/JevAdvBench/JevAdvBench)]
- **`arXiv 2026`** JevOut: Natural Context Can Flip Decision Models. [[paper](https://arxiv.org/abs/2609.30243v2)] [[code](https://github.com/XZX34/JevOut)]
- **`ICML 2017`** On Calibration of Modern Neural Networks. [[paper](https://proceedings.mlr.press/v70/guo17a.html)]
- **`NeurIPS 2017`** Selective Classification for Deep Neural Networks. [[paper](https://proceedings.neurips.cc/paper/2017/hash/4a8423d5e91fda00bb7e46540e2b0cf1-Abstract.html)]

<a id="datasets-and-benchmarks"></a>

## 📊 Datasets and Benchmarks

Training data specify the decision targets and situations available to the learner; evaluation tests how the resulting capabilities extend beyond those examples. The tables below are taken from the survey. Click a name to open the paper.

<a id="training-data"></a>

### Training Data and Supervision

Supervision differs in both its targets and the situations it covers: a selected answer, relative probability mass, a position on an ordered scale, paired evidence changes, or states that follow deliberately induced mistakes.

<div align="center">

| Resource | Training source and scale | Gold | Soft | Contrast | Recovery | Held-out unit |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| ***Model-derived targets*** | | | | | | |
| [SII-Decision](https://arxiv.org/abs/2610.03935v2) | Public decision tasks<br><sub>39,279 questions (27B setting)</sub> |  | ● |  |  | Frozen dev. set (selection) |
| [JevAgentHarness](https://arxiv.org/abs/2610.21412v1) | 587 agent tasks<br><sub>15,694 states; 30.9% post-mistake</sub> | ● | ● |  | ● | Task seed† |
| ***Labelled multimodal targets*** | | | | | | |
| [Visual Jev](https://arxiv.org/abs/2609.25845v1) | GQA Choice + SNLI-VE Claim<br><sub>30,416 + 9,000 items; 2–8 candidates</sub> | ● |  | ◐ |  | Image; TextVQA, TallyQA |
| [UniJev](https://arxiv.org/abs/2610.25784v1) | Text, image, and video sources<br><sub>160,608 train / 3,124 test questions</sub> | ● | ● |  |  | Source group |
| ***Evidence- and rubric-structured targets*** | | | | | | |
| [PACT](https://arxiv.org/abs/2609.35865) | Answer-changing fact edits<br><sub>1,167 curated pairs; 2,334 records</sub> | ● |  | ● |  | Source family |
| [RubriX](https://arxiv.org/abs/2610.34875v1) | Public + proprietary synthetic<br><sub>34,261 examples; 2,284 criteria</sub> | ● |  |  |  | Criterion / level‡ |

</div>

<sub>**Gold**: benchmark, human, or rubric labels. **Soft**: model- or teacher-derived probability targets. **Contrast**: paired, counterfactual, or evidence-ablated views. **Recovery**: states that follow deliberately induced errors. ● primary training signal, ◐ partial. Counts keep each study's original units.<br>† Splits share the task generator, and some early states repeat across splits. ‡ Held out at test but not fully at validation; also evaluated on human-labelled benchmarks.</sub>

- **`arXiv 2026`** General Decision Models: Benchmarking and Insights Beyond Jev. [[paper](https://arxiv.org/abs/2610.03935v2)]
- **`arXiv 2026`** Visual Jev: Accurate and Efficient Decisions from Shared Visual Context. [[paper](https://arxiv.org/abs/2609.25845v1)] [[code](https://github.com/guanxuyu-sv/Visual-Jev)]
- **`arXiv 2026`** UniJev: Separating Accuracy from Calibration in Trimodal System One Decision Models. [[paper](https://arxiv.org/abs/2610.25784v1)]
- **`arXiv 2026`** RubriX: A Fast, Fine-Tunable Non-Autoregressive Rubric Judge. [[paper](https://arxiv.org/abs/2610.34875v1)]
- **`arXiv 2026`** JevAgentHarness: Benchmarking, Deploying and Training Decision Models That Share Control with LLMs. [[paper](https://arxiv.org/abs/2610.21412v1)] [[code](https://github.com/JevAgentHarnesS/JevAgentHarnEsS)]
- **`arXiv 2026`** PACT: Pairwise-Anchored Calibrated Tuning for Single-Token Typed Decisions. [[paper](https://arxiv.org/abs/2609.35865)] [[code](https://github.com/BennyLinntu/PACT-Pairwise-Anchored-Calibrated-Tuning-for-Single-Token-Typed-Decisions)]

<a id="evaluation-benchmarks"></a>

### Evaluation Benchmarks

Existing evaluations fall into general-purpose, capability-specific/diagnostic, and agent-oriented studies, and differ in which protocol they use and which dimensions they measure.

<div align="center">

| Benchmark / Study | Protocol | Task coverage and scale | Acc. | Prob. | Rob. | Compl. | Cost | Attr. |
| :--- | :---: | :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| ***General-purpose evaluations*** | | | | | | | | |
| [Broad evaluation](https://arxiv.org/abs/2609.37647v1) | <img src="https://img.shields.io/badge/S-3A6E9A?style=flat-square" alt="S" title="Static fixed-request evaluation"/> | 37 datasets<br><sub>346,009 requests</sub> | ● | ● |  |  |  |  |
| [JEVal](https://arxiv.org/abs/2610.03935v2) | <img src="https://img.shields.io/badge/S-3A6E9A?style=flat-square" alt="S" title="Static fixed-request evaluation"/> | 36 datasets; 10 domains<br><sub>11,257 bilingual instances</sub> | ● |  |  |  |  |  |
| ***Capability-specific and diagnostic evaluations*** | | | | | | | | |
| [TypedBench](https://arxiv.org/abs/2610.11392v2) | <img src="https://img.shields.io/badge/S-3A6E9A?style=flat-square" alt="S" title="Static fixed-request evaluation"/> <img src="https://img.shields.io/badge/I-6B4C9A?style=flat-square" alt="I" title="Controlled intervention"/> | 9 suites<br><sub>7 policy generators</sub> | ● | ● | ● |  | ● |  |
| [DecisionPivot](https://arxiv.org/abs/2610.28522v1) | <img src="https://img.shields.io/badge/S-3A6E9A?style=flat-square" alt="S" title="Static fixed-request evaluation"/> <img src="https://img.shields.io/badge/I-6B4C9A?style=flat-square" alt="I" title="Controlled intervention"/> | 40,033 items<br><sub>7,800 generated; 32,233 public</sub> | ● | ● | ● |  |  |  |
| [GraphDecisionBench](https://arxiv.org/abs/2610.06354v3) | <img src="https://img.shields.io/badge/S-3A6E9A?style=flat-square" alt="S" title="Static fixed-request evaluation"/> <img src="https://img.shields.io/badge/I-6B4C9A?style=flat-square" alt="I" title="Controlled intervention"/> | Recognition; construction<br><sub>graph–text contrasts</sub> | ● |  | ◐ |  |  |  |
| [Selection-bias study](https://arxiv.org/abs/2610.35826v2) | <img src="https://img.shields.io/badge/S-3A6E9A?style=flat-square" alt="S" title="Static fixed-request evaluation"/> <img src="https://img.shields.io/badge/I-6B4C9A?style=flat-square" alt="I" title="Controlled intervention"/> | 1,700 questions<br><sub>8 presentations; fixed-order repeats</sub> | ● |  | ● |  |  |  |
| Shift studies [[1]](https://arxiv.org/abs/2610.36311v1) [[2]](https://arxiv.org/abs/2610.34329v1) | <img src="https://img.shields.io/badge/S-3A6E9A?style=flat-square" alt="S" title="Static fixed-request evaluation"/> <img src="https://img.shields.io/badge/I-6B4C9A?style=flat-square" alt="I" title="Controlled intervention"/> | Distribution shift<br><sub>language, domain, prior, candidates</sub> |  | ● | ● |  |  |  |
| Targeted diagnostics [[1]](https://arxiv.org/abs/2609.39496v1) [[2]](https://arxiv.org/abs/2609.35342v1) [[3]](https://arxiv.org/abs/2609.37470v2) [[4]](https://arxiv.org/abs/2609.31142v2) | <img src="https://img.shields.io/badge/S-3A6E9A?style=flat-square" alt="S" title="Static fixed-request evaluation"/> <img src="https://img.shields.io/badge/I-6B4C9A?style=flat-square" alt="I" title="Controlled intervention"/> | Absent answers; context edits<br><sub>probability relations</sub> | ◐ | ● | ● |  |  |  |
| ***Agent-oriented and interactive evaluations*** | | | | | | | | |
| [System1-Arena](https://arxiv.org/abs/2610.21244v1) | <img src="https://img.shields.io/badge/S-3A6E9A?style=flat-square" alt="S" title="Static fixed-request evaluation"/> <img src="https://img.shields.io/badge/C-B85C2C?style=flat-square" alt="C" title="Interactive closed-loop execution"/> | 105 tasks; 138 environments<br><sub>92k questions</sub> | ● |  |  | ● | ● |  |
| [JevAgentHarness](https://arxiv.org/abs/2610.21412v1) | <img src="https://img.shields.io/badge/S-3A6E9A?style=flat-square" alt="S" title="Static fixed-request evaluation"/> <img src="https://img.shields.io/badge/R-3F8C6B?style=flat-square" alt="R" title="Replay on recorded decision states"/> <img src="https://img.shields.io/badge/C-B85C2C?style=flat-square" alt="C" title="Interactive closed-loop execution"/> | Isolated decisions<br><sub>recorded states; live execution</sub> | ● |  |  | ● | ● |  |
| [Paired harness study](https://arxiv.org/abs/2610.02267v1) | <img src="https://img.shields.io/badge/S-3A6E9A?style=flat-square" alt="S" title="Static fixed-request evaluation"/> <img src="https://img.shields.io/badge/I-6B4C9A?style=flat-square" alt="I" title="Controlled intervention"/> | 11 decision points<br><sub>7,283 base cases; 6,640 variants</sub> | ● |  | ● |  | ● |  |
| [BLADE](https://arxiv.org/abs/2610.18780v1) | <img src="https://img.shields.io/badge/I-6B4C9A?style=flat-square" alt="I" title="Controlled intervention"/> <img src="https://img.shields.io/badge/T-9A7B2A?style=flat-square" alt="T" title="Offline trajectory audit"/> | 1,228 failed trajectories<br><sub>5 benchmark subsets</sub> |  |  |  |  | ◐ | ● |

</div>

<sub>**Protocols** (not mutually exclusive): <img src="https://img.shields.io/badge/S-3A6E9A?style=flat-square" alt="S" title="Static fixed-request evaluation"/> static fixed-request evaluation; <img src="https://img.shields.io/badge/I-6B4C9A?style=flat-square" alt="I" title="Controlled intervention"/> controlled intervention; <img src="https://img.shields.io/badge/R-3F8C6B?style=flat-square" alt="R" title="Replay on recorded decision states"/> replay on recorded decision states; <img src="https://img.shields.io/badge/C-B85C2C?style=flat-square" alt="C" title="Interactive closed-loop execution"/> interactive closed-loop execution; <img src="https://img.shields.io/badge/T-9A7B2A?style=flat-square" alt="T" title="Offline trajectory audit"/> offline trajectory audit.<br>**Measured dimensions**: **Acc.** decision accuracy or solution quality; **Prob.** calibration or probability consistency; **Rob.** robustness to order, framing, shift, or edits; **Compl.** task completion; **Cost** latency, efficiency, or deployment cost; **Attr.** error attribution. ● reported, ◐ partially reported.</sub>

**Reading the protocols.** Static tests assess fixed requests; controlled interventions isolate sensitivity to changed inputs or alternatives; replay evaluates recorded states; trajectory audits analyze completed executions; closed-loop evaluation additionally captures states induced by the tested policy. Together they separate local accuracy and calibration from robustness, task completion, and end-to-end cost.

#### General-Purpose Evaluations

- **`arXiv 2026`** Evaluating and Benchmarking the System One Model Jev. [[paper](https://arxiv.org/abs/2609.37647v1)] [[code](https://github.com/AppliedMachineLearning-Lab/jev-benchmarking)]
- **`arXiv 2026`** General Decision Models: Benchmarking and Insights Beyond Jev. [[paper](https://arxiv.org/abs/2610.03935v2)]

#### Capability-Specific and Diagnostic Evaluations

- **`arXiv 2026`** TypedBench: A Benchmark for Calibration, Framing Sensitivity, and Cost in System One Decision Models. [[paper](https://arxiv.org/abs/2610.11392v2)]
- **`arXiv 2026`** DecisionPivot: A Diagnostic Benchmark for Decision Models in LLM Systems. [[paper](https://arxiv.org/abs/2610.28522v1)] [[code](https://github.com/StartLuxLabs/DecisionPivot)]
- **`arXiv 2026`** GraphDecisionBench: Benchmarking System One Models on Graph Tasks. [[paper](https://arxiv.org/abs/2610.06354v3)] [[code](https://github.com/VictorYXL/GraphDecisionBench)]
- **`arXiv 2026`** Understanding Jev-Style System One Models: Does Structure Reduce Selection Bias? [[paper](https://arxiv.org/abs/2610.35826v2)]
- **`arXiv 2026`** Confidence Is Not a Promise: A Reference Audit of Typed Decision Models. [[paper](https://arxiv.org/abs/2610.36311v1)]
- **`arXiv 2026`** SHIFT-JEV: Certified Abstention and Gain-Gated Continual Adaptation for LLM-Based Decision Models under Distribution Shift. [[paper](https://arxiv.org/abs/2610.34329v1)]
- **`arXiv 2026`** When the Right Answer Is Missing: An Arithmetic-Dependent Rejection Bottleneck in Jev. [[paper](https://arxiv.org/abs/2609.39496v1)]
- **`arXiv 2026`** Jev thinks "I don't know'', but doesn't say it: Introducing Sys1Cal-v1 Dataset for Probability Calibration. [[paper](https://arxiv.org/abs/2609.35342v1)] [[code](https://github.com/little-g-ai/Sys1Cal-v1)]
- **`arXiv 2026`** Probability Contracts: Accuracy, Coherence, and Decisions Across LLM Interfaces. [[paper](https://arxiv.org/abs/2609.37470v2)]
- **`arXiv 2026`** JevAdvBench: A Benchmark and Black-Box Attacks for Reinforcement Learning for Calibrated Decisions Models. [[paper](https://arxiv.org/abs/2609.31142v2)] [[code](https://github.com/JevAdvBench/JevAdvBench)]

#### Agent-Oriented and Interactive Evaluations

- **`arXiv 2026`** System1-Arena: Which Decisions Can Be Left to System 1? [[paper](https://arxiv.org/abs/2610.21244v1)] [[code](https://github.com/TianyuCodings/S1-Arena)]
- **`arXiv 2026`** JevAgentHarness: Benchmarking, Deploying and Training Decision Models That Share Control with LLMs. [[paper](https://arxiv.org/abs/2610.21412v1)] [[code](https://github.com/JevAgentHarnesS/JevAgentHarnEsS)]
- **`arXiv 2026`** Fast Models, Slow Evidence: A Paired and Self-Audited Evaluation of System-1 Decision Models for LLM Agent Harnesses. [[paper](https://arxiv.org/abs/2610.02267v1)]
- **`arXiv 2026`** BLADE: Auditing Information Use for Failure Attribution in LLM Multi-Agent Systems. [[paper](https://arxiv.org/abs/2610.18780v1)]

<a id="key-findings"></a>

### Key Empirical Findings

Within-study comparisons point to four recurring findings; heterogeneous tasks and cost measures rule out a cross-study ranking.

1. **Efficiency depends on computation depth and reuse.** Early-exit readouts and multi-question packing save substantial compute, but only when inputs and caches are actually shared.
2. **Accuracy does not establish decision reliability.** Models with near-identical accuracy can differ several-fold in coverage at a fixed error rate.
3. **Candidate coverage and evidence constrain selection.** Closed-set accuracy says little about rejecting when the right answer is missing; better candidates and evidence often matter more than a better selector.
4. **Static gains need not translate into system-level gains.** Identical static accuracy can yield very different closed-loop completion, and careless delegation can raise total cost.

<div align="center">

| Study | Comparison setting | Main finding |
| :--- | :--- | :--- |
| ***Computation depth and reuse*** | | |
| [FunDM](https://arxiv.org/abs/2610.17175v1) | 24-layer readout vs. full 36-layer Qwen3-8B; eight benchmarks. | Mean accuracy: 75.0% (+1 head layer) vs. 75.7%. MMLU-Pro BF16 vLLM throughput: 1.56× teacher (16 active sequences). |
| [Verdict](https://arxiv.org/abs/2610.20778v1) | Five packed vs. separate questions; same weights, BF16, typed-decisions test. | Per-case latency speedup: 1.3–1.8× (batch 1), 2.7–2.8× (batch 8), across base/large models. |
| [SanSi](https://arxiv.org/abs/2610.07730v2) | Ouro-1.4B one-loop baseline vs. SanSi at three/eight loops; 10,027 test items, three seeds. | Mean accuracy: 58.6% / 70.4% / 72.0%. Eight-loop test-set GPU time: 7.7× baseline (RTX A6000). |
| [Iris](https://arxiv.org/abs/2610.33702) | Independent vs. state-conditioned Iris-4B; Mind2Web cross-task split, 16 crops. | First-choice accuracy: 78.38% vs. 89.60%; OneJev-4B retrained on the same crop steps: 90.58%. Conditioning prevents cross-request reuse. |
| [Clinical criteria](https://arxiv.org/abs/2610.35721v1) | Jev 1.14 Single vs. Batch; TrialGPT test, 774 criteria. | Median complete-job latency: 8.08 → 1.20 s; macro-F1: 0.646 → 0.652. |
| ***Decision reliability*** | | |
| [Broad evaluation](https://arxiv.org/abs/2609.37647v1) | Pooled vs. dataset-level ECE; 279,925 answers, 22 Choice datasets, 15 bins. | Pooled ECE: 0.028; unweighted mean dataset ECE: 0.061. |
| [OpenJev-RLCD](https://arxiv.org/abs/2609.38850) | Two-stage RLCD vs. GRPO+KL; Qwen3-1.7B, 1,260 GSM8K-Verify items; dev-fitted temperature scaling. | Accuracy: (92.2±0.7)% vs. (91.8±0.6)%. Coverage at empirical error ≤5%: (81.1±7.2)% vs. (19.4±7.3)% (three-seed mean ± s.d.). |
| ***Candidate coverage and supplied evidence*** | | |
| [Missing answers](https://arxiv.org/abs/2609.39496v1) | 100 paired arithmetic cases, explicit rejection; threshold fitted on 20 separate cases. | Answer-present accuracy / answer-absent rejection: 99% / 7% by default, 97% / 79% with thresholding. |
| [Simulation](https://arxiv.org/abs/2610.01834v2) | 134 unseen ALFWorld games, 50-step limit; no lookahead vs. cloned-game observations and new commands. | Completion: 42/134 → 116/134 with lookahead (first run). |
| ***System-level utility*** | | |
| [System1-Arena](https://arxiv.org/abs/2610.21244v1) | Decision-2.0-Vega-27B / Jev 1.13 / Qwen3.5-9B; agent-category tests. | Static agent-category accuracy: 67.8% each; closed-loop completion: 26.5% / 17.4% / 5.7%. |
| [JevAgentHarness](https://arxiv.org/abs/2610.21412v1) | Full tool replacement or fitted delegation vs. Gemini 3.5 Flash; 500 DecisionGym tasks. | Full replacement: -19.0 pp success. Fitted tool/argument delegation: 14.2% net cost saving; unchanged observed task outcomes. |
| [SearchJev](https://arxiv.org/abs/2610.05107) | SearchJev-0.8B + Qwen3.5-27B vs. System 2 alone; 100 BrowseComp-Plus questions, one run each. | Median active time (model calls + retrieval): 4,080 → 868 s; accuracy: 45% → 46%. |
| [Coding control](https://arxiv.org/abs/2610.24649v1) | Jev vs. no controller; Sonnet 5.5, Terminal-Bench 2.1; 21 held-out tasks, three repetitions/arm. | Pass rate: 73.0% each; no detectable difference. Geometric-mean USD/attempt ratio: 1.19 (95% CI: 1.02–1.36). |

</div>

<sub>Representative within-study comparisons; conditions and cost measures differ across studies, so numbers are not comparable across rows. pp: percentage points; ECE: expected calibration error; CI: confidence interval.</sub>

<a id="evaluation-gaps"></a>

### Evaluation Limitations and Gaps

The findings above leave three questions unresolved:

- **Quality–cost comparability.** Gains should survive matched quality and complete cost accounting. For a sequential cascade, $\mathbb{E}[C_{\mathrm{total}}]=\mathbb{E}[C_f+C_{\mathrm{ctrl}}+I_{\mathrm{esc}}C_s+C_{\mathrm{exec}}]$; omitting one pre-screening call, for example, shrinks a reported saving from 23.9% to 4.3%.
- **Reliability and validation.** Thresholds and delegation rules need evaluation on data independent of their fitting; calibration, empirical selective risk, and statistically controlled risk are distinct claims, and risk must be reported with coverage.
- **Mechanism and transfer.** Explanations can omit features that drive the answer, and shorter training traces do not necessarily reduce inference cost; matched budgets, supervision, and evidence are needed to attribute gains to the reasoning mechanism.

- **`arXiv 2026`** Fast Models, Slow Evidence: A Paired and Self-Audited Evaluation of System-1 Decision Models for LLM Agent Harnesses. [[paper](https://arxiv.org/abs/2610.02267v1)]
- **`SIGKDD Explor. 2026`** A Survey on Jev: Prior Research, Adaptation Costs, and Evidence of System One Models. [[link](https://evan-py-chen.github.io/jev-survey/)]
- **`arXiv 2026`** Truncated Reasoning Under a Budget: Evaluating System-1 Models as Decision-makers. [[paper](https://arxiv.org/abs/2610.25768v2)]
- **`arXiv 2026`** SOMA: Continuous Monitoring and Intervention for System Two Reasoning with System One Models. [[paper](https://arxiv.org/abs/2610.26793v1)]
- **`arXiv 2026`** SHIFT-JEV: Certified Abstention and Gain-Gated Continual Adaptation for LLM-Based Decision Models under Distribution Shift. [[paper](https://arxiv.org/abs/2610.34329v1)]
- **`arXiv 2026`** REFLEX with Jev for Efficient Selective Control in LLM Agents. [[paper](https://arxiv.org/abs/2609.26532v1)]
- **`arXiv 2026`** Good Judgment, Bad Advice: Decision-Model Controllers Raise Agent Cost Without Improving Success. [[paper](https://arxiv.org/abs/2610.24649v1)] [[code](https://github.com/namanvats/decision-model-controllers)]
- **`ICML 2026`** Dynamics Within Latent Chain-of-Thought: An Empirical Study of Causal Structure. [[paper](https://proceedings.mlr.press/v306/li26c.html)]
- **`EMNLP Findings 2026`** Think Wider: Mitigating Latent Rank Collapse in Implicit Chain-of-Thought Reasoning. [[paper](https://aclanthology.org/2026.findings-emnlp.1773/)] [[code](https://github.com/whitesweater/WIDER)]
- **`EMNLP 2026`** When Compression Helps and When It Hurts: Condition-Aware Analysis of Chain-of-Thought Distillation. [[paper](https://aclanthology.org/2026.emnlp-main.1543/)]
- **`arXiv 2026`** Learning through Internalization. [[paper](https://arxiv.org/abs/2606.20937v1)]
- **`NeurIPS 2023`** Language Models Don't Always Say What They Think: Unfaithful Explanations in Chain-of-Thought Prompting. [[paper](https://proceedings.neurips.cc/paper_files/paper/2023/hash/ed3fea9033a80fea1376299fa7863f4a-Abstract-Conference.html)]
- **`arXiv 2023`** Measuring Faithfulness in Chain-of-Thought Reasoning. [[paper](https://arxiv.org/abs/2307.13702v1)]

<a id="agentic-workflows"></a>

## 🤖 Jev in Agentic Workflows

Inside an agent, typed judgments decide which evidence reaches the reasoner, how much computation to spend, whether a result is acceptable, and which action to take next. Coordinated systems additionally have to hand over control between fast and slow components, keep shared task state consistent, and check whether pending results are still valid.

<p align="center">
<a href="assets/decision_guided_workflow.pdf"><img src="assets/decision_guided_workflow.png" width="100%" alt="Decision-guided workflow for placing the blue cup on the plate."/></a>
</p>
<p align="center"><sub><i>Decision-guided workflow. Given an instruction to place the blue cup on the plate, the system (a) observes the scene and checks the available objects, (b) constructs candidate actions and filters them by feasibility, (c) selects the option that best matches the instruction, (d) reviews and executes the action, and (e) verifies the outcome against the goal to decide whether the task is complete or needs another attempt.</i></sub></p>

The taxonomy below maps the works collected in this section onto workflow roles and domains (§5.1), runtime coordination (§5.2), and system self-improvement (§5.3).

<p align="center">
<a href="assets/agentic_taxonomy.pdf"><img src="assets/agentic_taxonomy.png" width="100%" alt="Taxonomy of Jev in agentic workflows and coordinated systems."/></a>
</p>
<p align="center"><sub><i>Taxonomy of Jev in agentic workflows and coordinated systems. Workflow components cover information and memory selection, computation and reasoning control, verification and review, and action selection, together with domain-specific applications. Runtime coordination covers scheduling and task updates and reuse. Decision-guided self-improvement distinguishes feedback-driven system improvement from improving the improver. A study may appear in multiple branches when it supports distinct mechanisms.</i></sub></p>

<a id="information-and-memory-selection"></a>

### Information and Memory Selection

#### Memory Organization and Compaction

- **`arXiv 2026`** Jev-Mem: System-One-Controlled Agentic Memory for Efficient AI Agents. [[paper](https://arxiv.org/abs/2609.23986v1)] [[code](https://github.com/libingzheren/Jev-Mem)]
- **`arXiv 2026`** Mnemon: Raw Records, Fast Judgments, Slow Thoughts. [[paper](https://arxiv.org/abs/2609.36059v1)] [[code](https://github.com/Grivn/mnemon-memory-agent)]
- **`arXiv 2026`** Prune the Summary, Not the History: Source-Judged Compaction for Long-Horizon Agents. [[paper](https://arxiv.org/abs/2610.20927v1)]
- **`arXiv 2026`** Suggested Is Not Superseded: A Pre-Registered Evaluation of a System-One Model as a Memory-Retirement Gate. [[paper](https://arxiv.org/abs/2610.15710)]

#### Retrieval Routing and Candidate Selection

- **`arXiv 2026`** SearchJev: A Calibrated System-1 Model for Search Agents. [[paper](https://arxiv.org/abs/2610.05107)] [[code](https://github.com/EvoScientist/SearchJev)]
- **`arXiv 2026`** Probabilistic Relevance Decisions for Semantic Retrieval. [[paper](https://arxiv.org/abs/2611.08014)]
- **`arXiv 2026`** Can Jev Improve Demonstration Retrieval for In-Context Learning? [[paper](https://arxiv.org/abs/2611.00737)]
- **`NAACL 2024`** Adaptive-RAG: Learning to Adapt Retrieval-Augmented Large Language Models through Question Complexity. [[paper](https://aclanthology.org/2024.naacl-long.389/)]

#### Record-Level Selection and Aggregation

- **`arXiv 2026`** A general-purpose System One decision model for clinical records. [[paper](https://arxiv.org/abs/2610.27099v1)]
- **`arXiv 2026`** Can JEV Support Clinical Trial Criteria Assessment? [[paper](https://arxiv.org/abs/2610.35721v1)]
- **`arXiv 2026`** Prune First, Decide Fast: Scalable Semantic Query Processing with JEVDB. [[paper](https://arxiv.org/abs/2610.02046)] [[code](https://github.com/PDAIS/JEVDB)]
- **`arXiv 2026`** Calibrated Decisions at Scale: Converting Police Crash Narratives into Probabilistic Crash Variables with a System One Model (Jev). [[paper](https://arxiv.org/abs/2609.24052)]
- **`arXiv 2026`** Counting the Uncounted: Population-Level Surveillance of Documented Pregnancy and Fetal Harm in Police Crash Narratives with a System One Model (Jev). [[paper](https://arxiv.org/abs/2610.00213)]

<a id="computation-and-reasoning-control"></a>

### Computation and Reasoning Control

#### Model and Worker Routing

- **`arXiv 2026`** SCX Router: Streaming Zero-Shot Model Selection with a Decoder-KV Classifier and a Real-World Task Ontology. [[paper](https://arxiv.org/abs/2609.02292v1)]
- **`ICML 2026`** Optimized Deferral for Imbalanced Settings. [[paper](https://proceedings.mlr.press/v306/cortes26b.html)]
- **`arXiv 2026`** JevSoup: System-One Routing for Training-Free LoRA Composition. [[paper](https://arxiv.org/abs/2609.30922)]
- **`arXiv 2026`** EditJev: System One Routing for Image Editing Models. [[paper](https://arxiv.org/abs/2610.32635)]
- **`arXiv 2026`** Jev-LiteMAS: Separating Decisions from Generation in Multi-Agent Workflows. [[paper](https://arxiv.org/abs/2610.21411v2)]
- **`arXiv 2026`** Token-Efficient Multi-Agent Collaboration via System One-Guided Computational Division of Labor. [[paper](https://arxiv.org/abs/2610.08155)]
- **`ICLR 2025`** RouteLLM: Learning to Route LLMs from Preference Data. [[paper](https://proceedings.iclr.cc/paper_files/paper/2025/hash/5503a7c69d48a2f86fc00b3dc09de686-Abstract-Conference.html)] [[code](https://github.com/lm-sys/RouteLLM)]
- **`TMLR 2024`** FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance. [[paper](https://openreview.net/forum?id=cSimKw5p6R)] [[code](https://github.com/stanford-futuredata/FrugalGPT)]
- **`NeurIPS 2018`** Predict Responsibly: Improving Fairness and Accuracy by Learning to Defer. [[paper](https://papers.neurips.cc/paper_files/paper/2018/hash/09d37c08f7b129e96277388757530c72-Abstract.html)]

#### Information Acquisition

- **`arXiv 2026`** LAVOIR: Teaching a Single-Pass Decision Encoder When and What to Ask with Amortized Value of Information. [[paper](https://arxiv.org/abs/2609.30706v1)] [[code](https://github.com/moganai/lavoir)]
- **`arXiv 2026`** JevForest: Path Voting for Budgeted Feature Acquisition. [[paper](https://arxiv.org/abs/2610.10615)]

#### Answer Readout and Reasoning Control

- **`arXiv 2026`** Selective Reasoning for Typed Decision Models: When to Start, Read, and Stop. [[paper](https://arxiv.org/abs/2610.19644v1)] [[code](https://github.com/binzheng-joey/selective-reasoning)]
- **`arXiv 2026`** Truncated Reasoning Under a Budget: Evaluating System-1 Models as Decision-makers. [[paper](https://arxiv.org/abs/2610.25768v2)]
- **`arXiv 2026`** StartLux-Decision: AutoThink Decision Models with Reinforcement Learning for Calibrated Deliberative Decisions. [[paper](https://arxiv.org/abs/2610.17309v2)] [[code](https://github.com/StartLuxLabs/StartLux-Prism)]

#### Behavior Monitoring and Intervention

- **`arXiv 2026`** SOMA: Continuous Monitoring and Intervention for System Two Reasoning with System One Models. [[paper](https://arxiv.org/abs/2610.26793v1)]
- **`arXiv 2026`** Good Judgment, Bad Advice: Decision-Model Controllers Raise Agent Cost Without Improving Success. [[paper](https://arxiv.org/abs/2610.24649v1)] [[code](https://github.com/namanvats/decision-model-controllers)]

<a id="verification-and-review"></a>

### Verification and Review

#### Quality Review

- **`arXiv 2026`** JEV-as-a-Judge: Accept When Confident, Escalate When Unsure. [[paper](https://arxiv.org/abs/2609.26550v5)]
- **`arXiv 2026`** Jev-LiteMAS: Separating Decisions from Generation in Multi-Agent Workflows. [[paper](https://arxiv.org/abs/2610.21411v2)]
- **`arXiv 2026`** Decision language models as affordable and rapid alternatives to LLM judges of clinical generative AI. [[paper](https://arxiv.org/abs/2610.28826v1)]
- **`arXiv 2026`** Can Jev Judge Radiology Reports? Evaluating a System One Model for Clinical Factuality. [[paper](https://arxiv.org/abs/2609.27607)]
- **`arXiv 2026`** PRISM: Calibrating Black-Box Judges with Counterfactual Probes. [[paper](https://arxiv.org/abs/2610.18389v1)]
- **`arXiv 2026`** Jev-as-a-Judge for RAG Claim Verification: Confidence-Gated Cascading Between a Non-Generative Verifier and a Frontier LLM. [[paper](https://arxiv.org/abs/2610.20634v1)] [[code](https://github.com/adorosario/jev-rag-claim-verification)]
- **`arXiv 2026`** When Multiple Choices Are Correct: Asymmetric Probability Allocation in Jev and Other Decision Models. [[paper](https://arxiv.org/abs/2610.29217v1)]

#### Policy Compliance

- **`arXiv 2026`** Jev-Guard: Fast Routing and Auditing Agent Actions over Editable Policy Trees. [[paper](https://arxiv.org/abs/2610.18326v1)] [[code](https://github.com/HKUST-KnowComp/jev-guardrail)]
- **`arXiv 2026`** Benchmarking System One Models in Online Moderation. [[paper](https://arxiv.org/abs/2610.07953v1)]
- **`arXiv 2026`** Evaluating System One Models for Agent Security Decisions: Reliability, Calibration, and Selective Automation. [[paper](https://arxiv.org/abs/2609.33401v3)]
- **`arXiv 2026`** JevBreak: Your Model, Their Decision. [[paper](https://arxiv.org/abs/2611.03759)]

#### Scoring for Output Improvement

- **`arXiv 2026`** P(Yes): 15 Improvements via Test-Time Scaling of Decision Models. [[paper](https://arxiv.org/abs/2610.25474v1)] [[code](https://github.com/tear-labs/pyes-search)]
- **`arXiv 2026`** JevVibe: Efficient Classification-Guided Secure Code Generation. [[paper](https://arxiv.org/abs/2609.34963)]

<a id="action-selection"></a>

### Action Selection

#### Tool and Action Selection

- **`arXiv 2026`** REFLEX with Jev for Efficient Selective Control in LLM Agents. [[paper](https://arxiv.org/abs/2609.26532v1)]
- **`arXiv 2026`** JevSpawn: Adaptive Agentic Inference through Compositional Action Spaces. [[paper](https://arxiv.org/abs/2610.00437v1)] [[code](https://github.com/Hoyant-Su/JevSpawn)]
- **`arXiv 2026`** Learning to Formalize: Bridging Open-Ended Workloads and Fast Decision Models. [[paper](https://arxiv.org/abs/2610.36595v1)]
- **`arXiv 2026`** JevCoder: When Does Decision Delegation Pay Off in Code Repair? [[paper](https://arxiv.org/abs/2610.19596)]

#### Action Evaluation and Refinement

- **`arXiv 2026`** Code Owns the Simulation, Jev Owns the Evaluation. [[paper](https://arxiv.org/abs/2610.01834v2)]
- **`arXiv 2026`** Jev with Foresight: Safe Language-Guided Autonomous Driving via a Language World Model. [[paper](https://arxiv.org/abs/2611.05448)]
- **`arXiv 2026`** JevVLA: Jev-Inspired Energy-Based Adaptation for Vision-Language-Action Policies. [[paper](https://arxiv.org/abs/2610.35010v1)]

#### Mapping Decisions to Action

- **`arXiv 2026`** System One Models for Wireless Decision-Making:Applications and Performance Evaluation. [[paper](https://arxiv.org/abs/2610.04345v1)]
- **`arXiv 2026`** Replacing Large Language Models with Jev Decision Models for Low-Latency Edge Service Orchestration. [[paper](https://arxiv.org/abs/2609.22753v2)]
- **`arXiv 2026`** Jev for Scientific Decisions: Evaluating Semantic Choices and Their Consequences. [[paper](https://arxiv.org/abs/2609.24965v4)] [[code](https://github.com/bydeng01/scientific-decision-eval)]

<a id="runtime-coordination"></a>

### Runtime Coordination

Fast and slow components run at different speeds, so a coordinated system must decide when to wait, who controls the next step, and which earlier work is still valid after new observations or request changes.

**⏱️ Scheduling.** Dependencies decide which work can proceed and which must await a pending result, and reasoning triggers must specify who controls the next step.

#### Continuing or Waiting for Results

- **`ICLR 2026`** Real-Time Reasoning Agents in Evolving Environments. [[paper](https://proceedings.iclr.cc/paper_files/paper/2026/hash/ccbe16043125599293b01dd467c260f3-Abstract-Conference.html)] [[code](https://github.com/SALT-NLP/RealtimeGym)]
- **`arXiv 2026`** JevGPT: Thinking Fast and Slow in Autonomous Research. [[paper](https://arxiv.org/abs/2610.18272v1)]
- **`arXiv 2026`** JEV-Star: Fast, Low-Cost StarCraft II Control with Language-Model Planning. [[paper](https://arxiv.org/abs/2609.27331v1)] [[code](https://github.com/SC2musa/Jev_Star)]
- **`arXiv 2024`** Agents Thinking Fast and Slow: A Talker-Reasoner Architecture. [[paper](https://arxiv.org/abs/2410.08328v1)]

#### Reasoning and Transferring Control

- **`arXiv 2026`** JevAgentHarness: Benchmarking, Deploying and Training Decision Models That Share Control with LLMs. [[paper](https://arxiv.org/abs/2610.21412v1)] [[code](https://github.com/JevAgentHarnesS/JevAgentHarnEsS)]
- **`arXiv 2026`** System Switch: When Should a Fast Decision Model Stop and Think? [[paper](https://arxiv.org/abs/2610.09683v1)] [[code](https://github.com/dexmac221/system-switch)]
- **`arXiv 2026`** SOMA: Continuous Monitoring and Intervention for System Two Reasoning with System One Models. [[paper](https://arxiv.org/abs/2610.26793v1)]
- **`ICLR 2025`** System-1.x: Learning to Balance Fast and Slow Planning with Language Models. [[paper](https://proceedings.iclr.cc/paper_files/paper/2025/hash/605e59e78284907aa4fce5280838def3-Abstract-Conference.html)] [[code](https://github.com/swarnaHub/System-1.X)]

**🔄 Task updates and reuse.** Execution results and new observations feed back through the reasoning–action loop; coordination then governs state updates, reuse of earlier work, and result validity when requests change.

#### Updating Task State and Progress

- **`arXiv 2026`** Jarvis: Coordination Deserves a Brain of Its Own in Agentic Interactive Models. [[paper](https://arxiv.org/abs/2610.23804v1)]
- **`arXiv 2026`** Jev-Mobile: Jev as an Executor for Mobile GUI Agents. [[paper](https://arxiv.org/abs/2609.30186v1)]
- **`arXiv 2026`** SharedKV-BT: Node-Local Typed Decisions for Behavior-Tree Agents. [[paper](https://arxiv.org/abs/2610.07327)]
- **`arXiv 2026`** System Switch: When Should a Fast Decision Model Stop and Think? [[paper](https://arxiv.org/abs/2610.09683v1)] [[code](https://github.com/dexmac221/system-switch)]

#### Checking and Reusing Work

- **`arXiv 2026`** MemDuet: Learning Dual-Process Memory for Long-Context Reasoning. [[paper](https://arxiv.org/abs/2610.19928v1)] [[code](https://github.com/jiangchengsong/MemDuet)]
- **`arXiv 2026`** JEV-Star: Fast, Low-Cost StarCraft II Control with Language-Model Planning. [[paper](https://arxiv.org/abs/2609.27331v1)] [[code](https://github.com/SC2musa/Jev_Star)]
- **`arXiv 2026`** DualWAM: Dual-System World Action Models for Asynchronous Global Planning and Local Refinement. [[paper](https://arxiv.org/abs/2609.24868v1)]
- **`arXiv 2026`** HarnessMM: Recursive Self-Improvement for Harness Token Efficiency. [[paper](https://arxiv.org/abs/2610.32813v1)]
- **`arXiv 2026`** SharedKV-BT: Node-Local Typed Decisions for Behavior-Tree Agents. [[paper](https://arxiv.org/abs/2610.07327)]

#### Request Changes and Outdated Results

- **`arXiv 2026`** Jarvis: Coordination Deserves a Brain of Its Own in Agentic Interactive Models. [[paper](https://arxiv.org/abs/2610.23804v1)]
- **`arXiv 2024`** Agents Thinking Fast and Slow: A Talker-Reasoner Architecture. [[paper](https://arxiv.org/abs/2410.08328v1)]

<a id="system-improvement"></a>

<a id="recursive-self-improvement"></a>

### Decision-Guided System Improvement

Execution feedback can also guide revisions that carry over to subsequent runs, to input features, policies, downstream learners, and harnesses. Recursive self-improvement (RSI) extends this use of feedback to the procedures that select, evaluate, and organize further improvements.

#### Feedback-Driven System Improvement

- **`arXiv 2026`** Can Jev be Your Q or Policy in Reinforcement Learning? [[paper](https://arxiv.org/abs/2610.11692v1)]
- **`arXiv 2026`** HarnessMM: Recursive Self-Improvement for Harness Token Efficiency. [[paper](https://arxiv.org/abs/2610.32813v1)]
- **`arXiv 2026`** ScienceBuddy: Recursive-in-Recursive Self-Improvement for Interactive Scientific Agents. [[paper](https://arxiv.org/abs/2609.17523v1)]
- **`Resource `** Autoresearch Feature Discovery. [[link](https://docs.typesafe.ai/cookbooks/autoresearch_feature_discovery)]

#### Improving the Improver

- **`arXiv 2026`** Dream-RSI: Recursive Self-Improvement through Evolving Worlds. [[paper](https://arxiv.org/abs/2609.14858v2)]
- **`arXiv 2026`** MetaRSI / RSI2: A Meta-Recursive Self-Improving System for Recursive Self-Improving Systems Themselves. [[paper](https://arxiv.org/abs/2609.06396v2)]

<a id="domain-applications"></a>

## 🏥 Domain Applications

Domain studies include both hosted Jev applications and separately developed Jev-style models, often used without any generative agent. The figure shows how task-specific judgments are expressed as structured decisions.

<p align="center">
<a href="assets/domain_applications.pdf"><img src="assets/domain_applications.png" width="72%" alt="Representative applications of decision models across six domains."/></a>
</p>
<p align="center"><sub><i>Representative applications across six domains: (a) healthcare, (b) language and speech, (c) recommendation and data matching, (d) legal analysis, (e) industry, and (f) multimodal understanding.</i></sub></p>

### Healthcare and Education

- **`arXiv 2026`** MedJev: Evaluating Zero-Shot System-One Decision Making in Medicine. [[paper](https://arxiv.org/abs/2610.36006)]
- **`arXiv 2026`** RetiJev: Fast and Source-Grounded Ophthalmic Decision Models. [[paper](https://arxiv.org/abs/2610.30947)]
- **`arXiv 2026`** PathoJev: Efficient Pathology Decisions from Shared Visual Context. [[paper](https://arxiv.org/abs/2610.40752)]
- **`arXiv 2026`** Can a System-One LLM Perform Knowledge Tracing When Few or No Learners Are Logged? [[paper](https://arxiv.org/abs/2610.11135)]
- **`arXiv 2026`** EDU-Jev Training System-One Decision Models for Education. [[paper](https://arxiv.org/abs/2610.41079)]

### Language and Speech

- **`arXiv 2026`** Emo-Jev: Probabilistic Reasoning for Emotion Classification with Jev. [[paper](https://arxiv.org/abs/2610.08829)]
- **`arXiv 2026`** Decide, Don't Generate: Competitive Dimensional ABSA with Jev's Typed Decisions. [[paper](https://arxiv.org/abs/2609.35293v1)] [[code](https://github.com/ZhangYiqun018/jev-dimabsa)]
- **`arXiv 2026`** Jev Matches 7B Language Models for Speech-Neuroprosthesis Rescoring. [[paper](https://arxiv.org/abs/2609.33538)]

### Recommendation and Data Matching

- **`arXiv 2026`** Decision-Oriented Recommendation Reranking: An Empirical Study of Its Promise and Limitations. [[paper](https://arxiv.org/abs/2609.40241)]
- **`arXiv 2026`** Decision4Rec: A Jev-Inspired Framework of Decision Models on Knowledge and Active Graphs for Multi-Agent Recommender Systems. [[paper](https://arxiv.org/abs/2610.26035)]
- **`arXiv 2026`** JevRec: Linguistic Encoding with Jev-like Decision Modeling for LLM-based Generative Recommendation. [[paper](https://arxiv.org/abs/2610.36937)]
- **`arXiv 2026`** Correspondences as Decisions: JevNexus for Decision-Centric Schema Matching. [[paper](https://arxiv.org/abs/2610.09487)]

### Scientific, Legal, and Social Analysis

- **`arXiv 2026`** A System One Approach to General Protein Evolution. [[paper](https://arxiv.org/abs/2611.05396v1)]
- **`arXiv 2026`** CausalGrade: Evidence-Guided Causal Graph Scoring with Jev. [[paper](https://arxiv.org/abs/2610.40692)]
- **`arXiv 2026`** Same Scores, Different Decisions: Evaluating JEV and Language Models for Legal Document Understanding. [[paper](https://arxiv.org/abs/2609.27678v2)]
- **`arXiv 2026`** Same-Number Citation Swaps: Stress-Testing Jev as a Financial Evidence Judge. [[paper](https://arxiv.org/abs/2610.08675)]
- **`arXiv 2026`** JEV versus LLMs: Accuracy, Cost and Calibration on Seven Political Science Replications. [[paper](https://arxiv.org/abs/2610.06625v2)]

### Industry and Infrastructure

- **`arXiv 2026`** Beyond CLIP: Calibrated (Few-Shot) Defect Detection With System One Decision Models. [[paper](https://arxiv.org/abs/2610.41199)]
- **`arXiv 2026`** HydroJEV: A one-second, training-free screen for cyber-attack and fault attribution in water distribution networks. [[paper](https://arxiv.org/abs/2610.02048)] [[code](https://github.com/mutianwei521/hydrojev)]
- **`arXiv 2026`** EO-Jev: Decision Model for Earth Observation. [[paper](https://arxiv.org/abs/2611.03371)]
- **`arXiv 2026`** AgriJev: Efficient Agricultural Candidate Scoring with Multi-Scale Adapted Models. [[paper](https://arxiv.org/abs/2610.19478)]

### Embodied and Multimodal Decisions

- **`arXiv 2026`** RoboJev: An Open Vision-Language Judge for Robot Success Detection. [[paper](https://arxiv.org/abs/2610.33718)]
- **`arXiv 2026`** Proactive Physical AI through Continuous Observation and Timely Intervention. [[paper](https://arxiv.org/abs/2610.16967)]
- **`arXiv 2026`** AudioJev: Direct Audio Decisions with Order-Calibrated Probabilities. [[paper](https://arxiv.org/abs/2610.01293v1)] [[code](https://github.com/SihanLv/AudioJev-Inference)]
- **`arXiv 2026`** Qwen-Audio-Jev: A Unified Interface for Audio Decisions. [[paper](https://arxiv.org/abs/2610.21614v2)]
- **`arXiv 2026`** VideoJev: System One Model for Video Understanding and Interaction. [[paper](https://arxiv.org/abs/2610.18615v1)] [[code](https://github.com/MCG-NJU/VideoJev)]
- **`arXiv 2026`** TimesJev: Thinking Fast on Time Series with System 1 Sufficiency. [[paper](https://arxiv.org/abs/2610.26002v1)]
- **`arXiv 2026`** JEVQA - Video Quality from Metadata, Bitstream, and Pixel Features with a General-Purpose Decision Model. [[paper](https://arxiv.org/abs/2609.24395)]
- **`arXiv 2026`** Benchmarking System One Models for Time-Series Decision Making: A Case Study of Jev. [[paper](https://arxiv.org/abs/2610.33410)]
- **`arXiv 2026`** More Features Are Not More Evidence: Limits of Training-Free Human Activity Recognition with Jev. [[paper](https://arxiv.org/abs/2609.36154)]

### Security

- **`arXiv 2026`** SecJev: Bringing Security Expertise to System One Decision Models. [[paper](https://arxiv.org/abs/2610.03073v2)] [[code](https://github.com/UESTC1010/SecJev)]
- **`arXiv 2026`** Cyber-JEV: Matched Contrastive Fine-Tuning of Language Models for Fast Source-Code Vulnerability Triage. [[paper](https://arxiv.org/abs/2611.02957)]

<a id="future-directions"></a>

## 🔭 Challenges and Future Directions

The effectiveness of a Jev-like model depends on more than the quality of its individual predictions: its decisions depend on the alternatives and evidence it receives, and the value of its judgments depends on how the surrounding system uses them. This points to three connected challenges: constructing decision spaces that contain useful choices, adapting collaboration to task needs, and learning from deliberation to improve future decisions.

<a id="research-agenda"></a>

### Research Agenda

<div align="center">

| Research question | Evaluation strategy | Key criteria |
| :--- | :--- | :--- |
| ***Decision-space design*** | | |
| **RQ 1:** When should candidates be revised? | Add, remove, or clarify candidates on the same task states; use retrieval and further reasoning as baselines. | Candidate coverage, selection accuracy, and revision cost. |
| ***Runtime collaboration*** | | |
| **RQ 2:** When does assistance improve decisions? | Measure which errors assistance corrects and which new errors it introduces under a fixed total budget. | Correction rate, introduced errors, and total cost. |
| ***Learning from deliberation*** | | |
| **RQ 3:** Which capabilities can be internalized? | Test direct predictions on new rule combinations and longer reasoning chains after training with deliberation. | Transfer accuracy, inference cost, and assistance demand. |
| **RQ 4:** What makes supervision reliable? | Check that the learner has enough evidence to support each label. Change key evidence and test predictions against independently verified answers. | Evidence sensitivity and appropriate abstention. |
| **RQ 5:** Can adaptation be sustained? | Track performance on new and earlier tasks over successive updates, with fixed supervision as a baseline. | Generalization, forgetting, and cumulative cost. |

</div>

### Expanding Decision Spaces

A Jev-like model cannot select an action that is absent from its candidate set. Open questions are how to recognize and revise an inadequate set, how to separate failures to *propose* from failures to *select*, and when to prefer candidate revision over retrieval or further reasoning.

- **`arXiv 2026`** When the Right Answer Is Missing: An Arithmetic-Dependent Rejection Bottleneck in Jev. [[paper](https://arxiv.org/abs/2609.39496v1)]
- **`arXiv 2026`** Benchmarking candidate coverage and rejection policy transfer in typed decision models. [[paper](https://arxiv.org/abs/2610.03387v2)] [[code](https://github.com/luckykevvv/Decision_Model_Benchmark)]
- **`arXiv 2026`** Learning to Formalize: Bridging Open-Ended Workloads and Fast Decision Models. [[paper](https://arxiv.org/abs/2610.36595v1)]
- **`arXiv 2026`** JevSpawn: Adaptive Agentic Inference through Compositional Action Spaces. [[paper](https://arxiv.org/abs/2610.00437v1)] [[code](https://github.com/Hoyant-Su/JevSpawn)]
- **`arXiv 2026`** Does a Hosted Decision Model Respect Irrelevant Alternatives? A Pre-Registered Test of Option-Set Dependence in Jev. [[paper](https://arxiv.org/abs/2610.16655v1)]

### Adaptive Collaboration

Predicting an error is not the same as predicting whether assistance will fix it. Systems must learn which form of help (retrieval, reasoning, or deferral) to request, from logs that only reveal the outcome of the chosen operation, and under shifting task distributions.

- **`arXiv 2026`** JevAgentHarness: Benchmarking, Deploying and Training Decision Models That Share Control with LLMs. [[paper](https://arxiv.org/abs/2610.21412v1)] [[code](https://github.com/JevAgentHarnesS/JevAgentHarnEsS)]
- **`arXiv 2026`** Good Judgment, Bad Advice: Decision-Model Controllers Raise Agent Cost Without Improving Success. [[paper](https://arxiv.org/abs/2610.24649v1)] [[code](https://github.com/namanvats/decision-model-controllers)]
- **`arXiv 2026`** SOMA: Continuous Monitoring and Intervention for System Two Reasoning with System One Models. [[paper](https://arxiv.org/abs/2610.26793v1)]
- **`arXiv 2026`** LAVOIR: Teaching a Single-Pass Decision Encoder When and What to Ask with Amortized Value of Information. [[paper](https://arxiv.org/abs/2609.30706v1)] [[code](https://github.com/moganai/lavoir)]
- **`arXiv 2026`** Confidence Is Not a Promise: A Reference Audit of Typed Decision Models. [[paper](https://arxiv.org/abs/2610.36311v1)]
- **`arXiv 2026`** SHIFT-JEV: Certified Abstention and Gain-Gated Continual Adaptation for LLM-Based Decision Models under Distribution Shift. [[paper](https://arxiv.org/abs/2610.34329v1)]
- **`NeurIPS 2018`** Predict Responsibly: Improving Fairness and Accuracy by Learning to Defer. [[paper](https://papers.neurips.cc/paper_files/paper/2018/hash/09d37c08f7b129e96277388757530c72-Abstract.html)]

### Reasoning Internalization and Co-Evolution

Deliberation resolves hard cases and produces experience that can be distilled back into fast judgment. The open question is whether this reciprocal loop transfers beyond familiar examples, rests on supervision that matches the learner's inputs, and can be sustained across changing tasks without forgetting.

- **`arXiv 2026`** Learning through Internalization. [[paper](https://arxiv.org/abs/2606.20937v1)]
- **`EMNLP 2026`** When Compression Helps and When It Hurts: Condition-Aware Analysis of Chain-of-Thought Distillation. [[paper](https://aclanthology.org/2026.emnlp-main.1543/)]
- **`ICML 2026`** Self-Distilled Reasoner: On-Policy Self-Distillation for Large Language Models. [[paper](https://proceedings.mlr.press/v306/zhao26be.html)] [[code](https://github.com/siyan-zhao/OPSD)]
- **`arXiv 2026`** EvoIn: Bridging Evolution and Internalization for Agent Fine-Tuning. [[paper](https://arxiv.org/abs/2609.35290v1)]
- **`arXiv 2026`** Co-Evolving Harnesses and Models: On-Policy Correction Helps Weaker Models Catch Up Where Imitation Fails. [[paper](https://arxiv.org/abs/2609.09134v1)]
- **`IJCAI 2026`** I-EDI: Robust Self-Evolution Agents via Verifiable Counterfactual Simulation. [[paper](https://www.ijcai.org/proceedings/2026/11)]
- **`arXiv 2026`** DecisionPivot: A Diagnostic Benchmark for Decision Models in LLM Systems. [[paper](https://arxiv.org/abs/2610.28522v1)] [[code](https://github.com/StartLuxLabs/DecisionPivot)]
- **`arXiv 2026`** Jev-DLC: Decision-Level Continual Learning from Typed Feedback. [[paper](https://arxiv.org/abs/2610.31521v1)]
- **`arXiv 2026`** SHIFT-JEV: Certified Abstention and Gain-Gated Continual Adaptation for LLM-Based Decision Models under Distribution Shift. [[paper](https://arxiv.org/abs/2610.34329v1)]
- **`arXiv 2024`** Distilling System 2 into System 1. [[paper](https://arxiv.org/abs/2407.06023v3)]
- **`NeurIPS 2022`** STaR: Bootstrapping Reasoning With Reasoning. [[paper](https://proceedings.neurips.cc/paper_files/paper/2022/hash/639a9a172c044fbb64175b5fad42e9a5-Abstract-Conference.html)]
- **`Science 2018`** A general reinforcement learning algorithm that masters chess, shogi, and Go through self-play. [[paper](https://doi.org/10.1126/science.aar6404)]
- **`NeurIPS 2017`** Thinking Fast and Slow with Deep Learning and Tree Search. [[paper](https://proceedings.neurips.cc/paper/2017/hash/d8e1344e27a5b08cdfd5d027d9b8d6de-Abstract.html)]
- **`AISTATS 2011`** A Reduction of Imitation Learning and Structured Prediction to No-Regret Online Learning. [[paper](https://proceedings.mlr.press/v15/ross11a.html)]

<a id="foundations"></a>

## 🧱 Foundations and Related Techniques

Agent frameworks, calibration and selective prediction, adaptive computation, and reasoning distillation that the survey builds on.

- **`Resource 2026`** Introducing System One Models & Jev. [[link](https://typesafe.ai/blog/introducing-system-one-models-and-jev)]
- **`arXiv 2025`** Qwen3 Technical Report. [[paper](https://arxiv.org/abs/2505.09388v1)] [[code](https://github.com/QwenLM/Qwen3)]
- **`ICLR 2023`** ReAct: Synergizing Reasoning and Acting in Language Models. [[paper](https://openreview.net/forum?id=WE_vluYUL-X)]
- **`NeurIPS 2023`** Reflexion: Language Agents with Verbal Reinforcement Learning. [[paper](https://proceedings.neurips.cc/paper_files/paper/2023/hash/1b44b878bb782e6954cd888628510e90-Abstract-Conference.html)]
- **`arXiv 2023`** Efficient Guided Generation for Large Language Models. [[paper](https://arxiv.org/abs/2307.09702v4)]
- **`NeurIPS 2022`** Chain-of-Thought Prompting Elicits Reasoning in Large Language Models. [[paper](https://proceedings.neurips.cc/paper_files/paper/2022/hash/9d5609613524ecf4f15af0f7b31abca4-Abstract-Conference.html)]
- **`arXiv 2019`** Passage Re-ranking with BERT. [[paper](https://arxiv.org/abs/1901.04085v5)]

<a id="related-surveys"></a>

## 📚 Related Surveys

- **`FCS 2027`** Fast, slow, and tool-augmented thinking for LLMs: a review. [[link](https://journal.hep.com.cn/fcs/EN/10.1007/s11704-026-51673-0)]
- **`SIGKDD Explor. 2026`** A Survey on Jev: Prior Research, Adaptation Costs, and Evidence of System One Models. [[link](https://evan-py-chen.github.io/jev-survey/)]
- **`GitHub 2026`** Jev and Typed Decision Models: An Empirical Survey of Calibration, Selective Control, and Open Implementations. [[code](https://github.com/Eurekaleo/awesome-jev-survey)]
- **`Resource 2026`** System One Decision Models: A Survey of the Typed Decision Contract, Its Ecosystem and Its Evidence. [[link](https://pozapas.github.io/system-one-models-survey/)]
- **`arXiv 2026`** The Hitchhiker's Guide to System 1 Models: Fast Decision-Making Beyond LLMs. [[paper](https://arxiv.org/abs/2610.33775v2)]
- **`TPAMI 2026`** From System 1 to System 2: A Survey of Reasoning Large Language Models. [[paper](https://doi.org/10.1109/TPAMI.2025.3637037)]
- **`arXiv 2025`** Implicit Reasoning in Large Language Models: A Comprehensive Survey. [[paper](https://arxiv.org/abs/2509.02350v1)]
- **`TMLR 2025`** Stop Overthinking: A Survey on Efficient Reasoning for Large Language Models. [[paper](https://openreview.net/forum?id=HvoG8SxggZ)]
- **`arXiv 2025`** From Efficiency to Adaptivity: A Deeper Look at Adaptive Reasoning in Large Language Models. [[paper](https://arxiv.org/abs/2511.10788v3)]

<a id="citation"></a>

## 📝 Citation

If you find this survey or repository useful, please cite our paper ([PDF](Jev-Like_Models_and_Decision_Systems_A_Survey.pdf)) and consider giving the repository a star.

```bibtex
@misc{jev_survey_2026,
  title        = {Jev-Like Models and Decision Systems: A Survey},
  author       = {Gao, Yiwei and Zhang, Chenhao and Ye, Mingcheng and
                  Li, Mingxuan and Zhang, Junbo and Chen, Jiale and
                  He, Yucheng and Zu, Lizheng and Xu, Hongbin and
                  Fan, Hehe and Wu, Baoyuan and Li, Ming and Zhu, Zheng},
  year         = {2026},
  howpublished = {\url{https://github.com/OmniAI-Lab-CUHKSZ/Jev_Survey}}
}
```

<a id="contributing"></a>

## 🤝 Contributing

We welcome additions and corrections. Please open an [issue](https://github.com/OmniAI-Lab-CUHKSZ/Jev_Survey/issues) or a pull request with the paper title, link, and the section where it fits.

<div align="center">

<a href="https://github.com/OmniAI-Lab-CUHKSZ"><b>OmniAI Lab @ CUHK-Shenzhen</b></a>

</div>
