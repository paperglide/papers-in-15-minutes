# The Centaur Method: How Small AI Models Outsmart Classical Optimizers by Sharing Secrets

*Original: Can LLMs Beat Classical Hyperparameter Optimization Algorithms? A Study on autoresearch*

**arXiv**: [https://arxiv.org/abs/2603.24647](https://arxiv.org/abs/2603.24647)  
**Read on PaperGlide**: [https://paperglide.net/papers/fk4vuj4c](https://paperglide.net/papers/fk4vuj4c)

---

**TL;DR:** Researchers have created a "Centaur" AI that combines the creative intuition of a large language model with the mathematical precision of traditional optimization formulas to fine-tune complex software. By sharing the internal statistical "brain" of the optimizer with the AI, the system can suggest clever shortcuts—like rounding numbers for efficiency—that math alone would miss. Surprisingly, a tiny, budget-friendly AI model using this hybrid method outperformed much larger, expensive models, proving that smart teamwork is more important than raw processing power.

---

## Paper Primer

### Hybrid Wins: 0.8B LLM Beats All 
A **0.8 B** LLM paired with **CMA‑ES** (the **Centaur** hybrid) reaches **val_bpb = 0.9763**, out‑performing every classical and LLM‑only method—even the much larger **27 B** model.

### Why Your HPO Might Be Stuck 
You’ve probably tuned transformers with Bayesian optimizers, random search, or the occasional LLM prompt, only to hit a wall of out‑of‑memory (OOM) crashes or painfully slow convergence. That’s because most HPO pipelines either ignore the rich domain knowledge an LLM can provide or let the LLM wander blindly in a fixed hyperparameter space. The result? Unreliable searches that waste GPU hours.

### What’s Broken in Current HPO 
- **Classical optimizers ignore code‑level tricks** – they only adjust numeric knobs, missing opportunities like attention‑window redesigns. 
- **LLM‑only agents get lost in fixed spaces** – despite seeing full trial histories, they produce OOM rates comparable to random search (≈ 50 %). 
- **Scaling LLMs doesn’t guarantee gains** – moving from **0.8 B** to **27 B** yields no improvement for fixed‑space tasks and only helps code‑editing agents. 
- **High‑variance evolution strategies** – **CMA‑ES** can find the best seed (val_bpb ≈ 0.9741) but also a mediocre one (≈ 0.9829), making results unpredictable. 
- **Hybrid ideas lack state sharing** – prior works (LLAMBO, SLLMBO) only feed trial histories to the LLM, never expose the optimizer’s internal statistics.

### What the Authors Actually Built 
They created **Centaur**, a hybrid that **shares the full internal state of CMA‑ES (mean µ, step‑size σ, covariance C) with an LLM** on a fraction of trials. The LLM receives this statistical snapshot, the top‑5 configurations, and the last 20 trials, then **overrides CMA‑ES’s proposal** almost every time (100 % for the 27 B model, 95 % for the 0.8 B model). CMA‑ES still updates its distribution with the final (possibly overridden) configuration, so the evolutionary search never loses its momentum. The result is a system that **keeps the reliability of a classical optimizer while injecting transformer‑specific intuition** (e.g., “use all‑short attention windows” or “round batch sizes to powers of two”). That’s it. That’s the core idea.

### Centaur’s Step‑by‑Step Playbook 
| Step | Action | Novelty |
|------|--------|---------|
| 1 | **Initialize** CMA‑ES (µ, σ, C) and empty history **H**. | – |
| 2 | Sample a random number; if ≤  **r** (default 0.3) → **LLM turn**. | **[NEW]** Probabilistic delegation. |
| 3a | **LLM turn**: extract µ, σ, C, top‑5 configs, last 20 trials; feed to **Qwen3.5** (0.8 B or 27 B). | **[NEW]** Full optimizer state in prompt. |
| 3b | LLM outputs a full configuration **x** (may differ from CMA‑ES proposal). | – |
| 4a | **CMA‑ES turn** (probability 1 −  **r**): propose **x** from its multivariate Gaussian. | – |
| 5 | **Evaluate** **x** on the target model (nanochat) for 5 min; record val_bpb **y** (OOM → y = 100). | – |
| 6 | **CMA‑ES.Update( x, y)** – adjust µ, σ, C using the *final* result, regardless of who generated **x**. | **[NEW]** State updated from overridden proposals. |
| 7 | Append (**x**, **y**) to **H**; loop back to Step 2 until the 24‑hour budget expires. | – |

Key hyperparameters: **r = 0.3** (default), temperature 0.7, token limits 2048 (fixed‑HP) / 16384 (code). The LLM runs on the same **NVIDIA H200** GPU, sharing 80 GB VRAM with the training job.

### Numbers That Speak for Themselves 
| Method | Model | Best mean **val_bpb** (↓) | OOM % | Trials (≈) | Std of best |
|--------|-------|---------------------------|------|------------|-------------|
| **Centaur** (Hybrid) | **0.8 B** | **0.9766 ± 0.0008** | 15 % | 334 | 0.0008 |
| **Centaur** (Hybrid) | **27 B** | **0.9763 ± 0.0005** | 15 % | 334 | 0.0005 |
| **TPE** (Classical) | – | 0.9768 ± 0.0019 | 11 % | 317 | 0.0019 |
| **CMA‑ES** (Classical) | – | 0.9785 ± 0.0036 | 16 % | 336 | 0.0036 |
| **Karpathy Agent (Code)** | **27 B** | 0.9814 ± 0.0046 | 12 % | 324 | 0.0046 |
| **LLAMBO (Paper)** | **27 B** | 0.9862 ± 0.0041 | 48 % | 496 | 0.0041 |
| **Random Search** | – | 0.9873 ± 0.0021 | 56 % | 568 | 0.0021 |

*All methods share the same 24‑hour GPU budget and three random seeds. LLM inference time is excluded from wall‑clock measurements.*

### The Twist No One Saw Coming 
You’d expect a **larger LLM** to dominate every scenario, yet the **0.8 B** model **outperforms its 27 B sibling** in the hybrid setting (0.9766 vs. 0.9763) and **fails completely** at unconstrained code editing (val_bpb ≈ 0.9910 vs. 0.9814). Moreover, **LLM‑only agents in a fixed hyperparameter space perform worse than random search**, despite seeing the full trial history. Scaling to a frontier model (**Gemini 2.5 Flash**) also brings no benefit. The real magic lies not in raw model size but in **how the optimizer’s statistical state is communicated**—a tiny LLM can steer a sophisticated evolution strategy when given the right numbers.

### When This Won’t Work 
- **Code‑editing tasks** that require generating syntactically correct patches; the 0.8 B LLM collapses (val_bpb ≈ 0.9910). 
- **Multi‑objective or multi‑task HPO** – the benchmark only covers a single validation metric on a 50 M‑parameter transformer. 
- **Environments with strict inference latency budgets** – LLM inference cost is omitted from wall‑time; real‑world pipelines may feel the overhead. 
- **Search spaces with poorly defined bounds** – ranges are manually set; extreme or poorly calibrated bounds could re‑introduce OOM spikes.

### What This Means for AutoML 
The takeaway is clear: **share the optimizer’s internal statistics with a modest LLM, and you get the best of both worlds**—the reliability of classical evolution strategies plus the domain intuition of a language model. This flips the conventional wisdom that “bigger LLMs = better HPO” on its head and opens a path toward **state‑aware, low‑cost hybrid optimizers** that can scale to larger models without sacrificing stability. Future work will likely explore other classical bases (e.g., TPE, SMAC) and push frontier LLMs into code‑editing roles, but the core lesson is already set: **the optimizer’s mind, not its size, should be the LLM’s playground.**

---

### Classical HPO vs LLM Agents: Reliability vs Flexibility 

Classical hyperparameter optimization (HPO) methods such as CMA‑ES and TPE follow a disciplined, repeatable search strategy—much like a **disciplined accountant** who meticulously records every transaction. In contrast, LLM‑based agents behave like a **creative consultant**: they can rewrite training code, suggest unconventional hyperparameter combos, and adapt on the fly, but their progress can be erratic. 

Why this matters for AutoML is simple: reliability guarantees steady improvement, while flexibility opens the door to novel solutions that a rigid optimizer might miss. The tension between these two qualities motivates a hybrid “Centaur” approach that aims to combine the accountant’s rigor with the consultant’s imagination.

---

### Empirical Contrast: Performance and Reliability 

We benchmarked nine HPO methods on the autoresearch task under an identical **24‑hour GPU budget** across three random seeds. The results show that classical optimizers (CMA‑ES, TPE) consistently converge faster and achieve better final validation bits‑per‑byte (val_bpb) than most LLM‑based agents. 

A notable exception is an LLM agent that directly edits training code; it reaches a validation performance of 

 **val_bpb ≈ 0.978** 

where: 
- **val\_bpb** — validation bits‑per‑byte, the metric we aim to minimize 
- **approximately equal to** — indicates an approximate value 
- **0.978** — the observed performance 

in fewer than **100** trials, making it competitive with the classical baselines. 

Reliability proved decisive: methods that suffered fewer **OOM** (out‑of‑memory) failures outperformed those that explored more broadly. Scaling experiments revealed that a modest **0.8 B** LLM suffices for the hybrid, while moving to a **27 B** model offers no advantage for fixed‑space methods. 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/figure1.png) 
*Figure 1: Best Validation Bits‑Per‑Byte (mean ± std across 3 seeds) of HPO algorithms against cumulative training time. All methods receive the same 24‑hour GPU training budget; LLM inference overhead is excluded. Classical methods such as CMA‑ES and TPE converge faster and to better final values than LLM‑based methods. Centaur, our CMA‑ES and LLM hybrid, achieves the best result in our experiments.*

The plot confirms that classical methods are roughly 

 **4 times faster** 

where: 
- **4** — factor of speedup 
- **times (multiplication sign)** — multiplication sign indicating “times” 
- **faster** — comparative adjective denoting higher speed 

than most LLM agents, while the Centaur hybrid attains the overall best performance.

---

### Centaur Hybrid: Sharing CMA‑ES State with an LLM 

Centaur bridges the reliability of CMA‑ES with the creativity of an LLM by exposing the optimizer’s full internal state to the language model. The shared components are:

- **Mean vector**μ**** – the current best estimate of the optimal hyperparameter configuration. 
- **Step‑size**σ**** – controls the sampling radius around **μ**, balancing exploration and exploitation. 
- **Covariance matrix**C**** – captures correlations between hyperparameters, shaping the search distribution.

Because these quantities are explicit, numeric, and mathematically interpretable, the LLM can generate natural‑language suggestions such as “increase the learning rate slightly because **σ** is small.” Those suggestions are then translated back into concrete code edits, allowing the LLM to inject domain knowledge while CMA‑ES continues its reliable convergence.

---

### Contributions 

- **Comprehensive benchmark** of nine HPO methods on the autoresearch task, covering both fixed‑hyperparameter and agentic code‑editing optimization under identical 24‑hour budgets. 
- **Empirical evidence** that classical HPO outperforms LLM agents in a fixed search space, while a code‑editing LLM narrows the performance gap substantially. 
- **Introduction of Centaur**, a hybrid that shares CMA‑ES’s full internal state with an LLM and achieves the best experimental results across all methods. 
- **In‑depth analysis** of search diversity, OOM rates, and model scaling, accompanied by a public release of per‑trial LLM conversation logs and the full code repository.

### Quick Definitions of Classical and LLM Baselines 

**Q. What is CMA‑ES?** A covariance‑matrix adaptation evolution strategy that iteratively samples candidates from a multivariate normal distribution and updates its mean, step‑size **σ**, and covariance matrix **C** based on performance. 

**Q. What is TPE?** A tree‑structured Parzen estimator that builds separate density models for good and bad hyperparameter regions and samples new configurations by maximizing the ratio of good‑to‑bad likelihoods. 

**Q. What is LLAMBO?** An approach that uses a large language model as a surrogate within Bayesian optimization, letting the LLM predict the objective value for candidate hyperparameters. 

**Q. What is SLLMBO?** A joint sampler that combines LLM‑generated suggestions with TPE proposals, merging the two sources before evaluation. 

**Q. What is the Zhang et al. 2023 prompt method?** Directly prompts an LLM to output a list of hyperparameter values, treating the LLM as a suggestion engine. 

**Q. What is LLaMA‑ES?** A technique that lets an LLM tune the internal hyperparameters of CMA‑ES (e.g., step‑size) by generating new settings. 

**Q. What is HOLLM?** A method that partitions the search space into sub‑regions via a bandit mechanism and asks an LLM to propose candidates within each region. 

All these baselines will be evaluated later under identical experimental constraints, ensuring a fair head‑to‑head comparison. 

--- 

### Experimental Constraints and Evaluation Protocol 

We benchmark every optimizer on **nanochat**, a small decoder‑only transformer, with the objective of minimizing validation bits‑per‑byte (**val_bpb**). 

* **Wall‑clock budget:** each method runs for **24 hours** on a single NVIDIA **H200 GPU** (with **141 GB HBM3e** memory) and is repeated with three independent random seeds. 
* **Trial limit:** up to **300 trials** per run; results are also reported against cumulative training time. 
* **Out‑of‑memory (OOM) handling:** OOM trials terminate quickly (under **less than 30 seconds**) and are counted as failures. Each such trial receives a penalty value of 

 **val\_bpb = 100.0** 

 Where: 
 - **val\_bpb** — validation bits‑per‑byte metric (lower is better) 
 - **=** — assignment of the penalty value 
 - **100.0** — a large constant far worse than any feasible outcome 

 This penalty is finite, allowing surrogate models to ingest it, yet it forces optimizers to stay within feasible memory limits. 

These uniform constraints provide a level playing field for comparing classical, LLM‑based, and hybrid approaches. 

--- 

### LLM Infrastructure and Resource Allocation 

All LLM‑based methods share the same optimizer backend: **Qwen‑3.5‑27B**, self‑hosted via **vLLM** on the same H200 GPU that trains the target model. 

* **VRAM cap for training:** **80 GB**, leaving the remaining memory for the vLLM inference server. 
* **Thinking mode:** disabled to keep inference costs predictable. 
* **Sampling temperature:** 

 **temperature = 0.7** 

 Where: 
 - **temperature** — controls randomness of token sampling (higher = more diverse) 
 - **=** — assignment operator 
 - **0.7** — chosen to balance exploration and stability 

* **Token limits:** fixed‑hyperparameter methods are limited to **2048 tokens** per request, while code‑editing methods may generate up to **16384 tokens**. 
* Prompt templates are publicly available in the repository for reproducibility. 

These settings ensure that LLM inference overhead never exceeds the 24‑hour GPU budget. 

--- 

### Automated Hyperparameter Extraction via AST Parsing 

The search space is built automatically from the training script, eliminating manual curation. The extraction proceeds as follows:

1. **Parse** the source file (e.g., `train.py`) into an **Abstract Syntax Tree (AST)**. 
2. **Walk** the tree and inspect each top‑level assignment node. 
3. **Select** assignments whose left‑hand side identifier is all uppercase (`ALL_CAPS`) **and** whose right‑hand side is a literal (integer, float, or string). 
4. **Record** the variable name, its type, and the literal value as a hyperparameter entry. 

*Example:* the line `EMBEDDING_LR = 0.6` becomes an AST node with a target name in `ALL_CAPS` and a float literal, so it is captured as a hyperparameter with default value 0.6. 

Applying this procedure to the nanochat training script yielded exactly **14 hyperparameters** (13 continuous/integer, 1 categorical). Their types, allowed ranges, and defaults (taken from Karpathy’s starting configuration) are listed in Table 1. 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/table1.png) *Table 1 with their types, ranges, and defaults from Karpathy’s starting configuration.* 

While setting numeric ranges still requires domain insight, the extraction itself is fully automated, removing any manual search‑space design. 

--- 

### Method Portfolio Overview 

Table 2 summarizes the nine methods evaluated in this study. They fall into three categories:

* **Classical optimizers:** CMA‑ES, TPE, and two additional standard HPO baselines. 
* **LLM‑based approaches:** LLAMBO, SLLMBO, the direct‑prompt method (Zhang et al. 2023), and HOLLM. 
* **Hybrid:** Centaur, which combines CMA‑ES state with LLM suggestions. 

The **“Fixed”** column indicates that a method operates within the 14‑hyperparameter space extracted via the AST parser. The **“Unconstrained”** column marks LLM methods that edit the training script directly, effectively searching over an unrestricted code space. 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/table2.png) *Table 2: Methods evaluated. “Fixed” indicates the shared 14‑HP search space; “Unconstrained” indicates direct source code editing.* 

This taxonomy clarifies which algorithms rely purely on hyperparameter tuning versus those that perform open‑ended code modification, setting the stage for the performance results presented later.

### Sharing the Optimizer’s Internal State with the LLM 

**Takeaway:** Centaur gives the language model a concise “snapshot” of CMA‑ES’s current belief about promising hyper‑parameter regions. 

On the designated LLM‑intervention trials, the optimizer transmits five pieces of information:

- the **mean vector**  **μ** – a concrete hyper‑parameter configuration representing the current center of the search distribution, 
- the **step‑size**  **σ** – a single scalar that controls the overall radius of the sampling distribution, 
- the **covariance matrix**  **matrix C** – a labeled matrix whose entries encode pairwise dependencies among hyper‑parameters, 
- the **top‑5 configurations** discovered so far, and 
- the **outcomes of the most recent 20 trials**. 

Because **μ**, **σ**, and **matrix C** are naturally expressible in plain language (a specific setting, a size, and a table of relationships), the LLM can reason about them just as a coach reviews a player’s statistics before giving advice. This sharing happens only on a fraction of trials, keeping communication overhead low.

--- 

### Probabilistic LLM Intervention 

**Takeaway:** Each iteration flips a biased coin; with probability  **r=0.3** the LLM takes over the proposal step. 

At trial  **t**, Centaur draws a Bernoulli random variable with success probability 

 **r=0.3** 

Where: 
- **r** — the probability of selecting an LLM turn, 
- **=** — equality sign indicating assignment of a constant value, 
- **0.3** — numeric probability (30 %). 

If the coin lands heads, the algorithm follows these steps:

1. **Extract** the current statistical summary **(μ, σ, C)** from CMA‑ES. 
2. **Call** the language model: **x ← LLM(μ, σ, C, H, S)**, where **H** is the history of past trials and **S** is the search space. 
3. The LLM may return any configuration **x is an element of S**; in practice it overrides the native CMA‑ES proposal almost every time (100 % with the 27B model, 95 % with the 0.8B model). 

If the coin lands tails, Centaur simply uses the optimizer’s own suggestion:

 **x ← CMA-ES.Propose()** 

Algorithm 1 (lines 3–8) captures this conditional flow.

--- 

### Continuous Learning from All Trials 

**Takeaway:** Regardless of who generated **x**, the evaluated result feeds back into CMA‑ES, allowing the distribution to adapt to the true objective landscape. 

After a candidate **x** is produced (by either the LLM or CMA‑ES), Centaur evaluates it:

 **y ← Evaluate(x)** 

The optimizer then updates its internal model:

 **CMA-ES.Update(x, y)** 

Finally, the new pair is recorded in the history set:

 **H ← H union {(x, y)}** 

These steps are shown in the latter half of Algorithm 1 (lines 9–12). Because the update uses the actual performance **y**, the mean **μ**, step‑size **σ**, and covariance **C** continue to evolve based on empirical evidence, even when the LLM frequently overrides the native proposal. This design guarantees that the search remains grounded in real objective values while still benefiting from the LLM’s guidance.

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/algorithm1.png) *Algorithm 1: Centaur – the full loop that mixes CMA‑ES proposals with probabilistic LLM interventions.*

--- 

### Positioning Centaur Among Optimization Methods 

**Takeaway:** Centaur is the only approach that explicitly shares CMA‑ES’s interpretable statistical state with an LLM, enabling low‑overhead, natural‑language communication. 

Table 2 enumerates the baseline methods evaluated in the study, ranging from fixed‑space search (CMA‑ES, SMAC, Random) to tree‑structured density estimation (TPE) and surrogate‑based Bayesian optimization (GP‑BO, LLAMBO). 

Centaur’s distinguishing feature is the transmission of **μ**, **σ**, and **matrix C** —a representation that can be verbalized directly—whereas methods like TPE rely on dense density estimators and GP‑BO on high‑dimensional posterior distributions that are cumbersome to articulate. 

By coupling a powerful evolutionary strategy with a language model through a communication‑friendly state, Centaur achieves a methodological advantage: it retains CMA‑ES’s adaptive power while unlocking the LLM’s ability to inject high‑level reasoning into the search.

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/table2.png) *Table 2: Overview of baseline optimization methods; Centaur is highlighted as the sole technique that shares CMA‑ES’s internal state with an LLM.*

### Experimental Setup and Evaluation Metric 

All experiments start from Karpathy’s default hyper‑parameter configuration, which yields a validation bits‑per‑byte of roughly **val_bpb ≈ 0.991**. To isolate the quality of the hyper‑parameter search, LLM inference time is subtracted from wall‑time so that every method is judged on the same computational budget. Each optimizer runs with three random seeds; we report the mean performance together with the standard deviation ( ± std ) to capture variability.

--- 

### Classical Optimizers Shine in Fixed Hyper‑Parameter Space 

**Takeaway:** *In a fixed search space, classical optimizers consistently beat pure‑LLM approaches.* 

- **Top performers (mean best val\_bpb):** 
 - **Centaur** — 0.9763 (hybrid, discussed later) 
 - **TPE** — 0.9768 
 - **SMAC** — 0.9778 
 - **CMA‑ES** — 0.9785 
 - **Karpathy Agent (Code)** — 0.9814 

- The best pure‑LLM method, **LLAMBO**, reaches only **val_bpb = 0.9862**, a noticeable gap from the classical leaders. Several LLM‑only strategies even fall below random search, showing that restricting an LLM to a fixed hyper‑parameter grid does not leverage its generative strengths.

- **Out‑of‑memory (OOM) avoidance predicts success.** All leading methods keep OOM rates at or below **OOM = 16%**, whereas the four lowest‑ranking methods exceed **OOM = 36%**. For example, LLAMBO suffers a **OOM = 48%** rate—comparable to random search’s **OOM = 56%** —despite higher search diversity, indicating that frequent OOM failures cripple performance.

- **Stateful classical methods** such as CMA‑ES and TPE maintain explicit optimization state, which helps them stay within memory limits (CMA‑ES  **OOM = 16%**, TPE  **OOM = 11%**). CMA‑ES explores broadly, covering **802** distinct grid cells, but shows higher seed variance ( **standard deviation = 0.0036**) than TPE ( **standard deviation = 0.0019**). Its best seed attains **val_bpb = 0.9741**, while the worst reaches **0.9829**.

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/table3.png) 
*Table 3 reports detailed diversity metrics (unique cells, pairwise L2 distances, etc.) across three seeds, confirming that low OOM rates coincide with superior validation scores.*

--- 

### Unconstrained Code Editing Demands Large LLMs 

**Takeaway:** *Only a sufficiently large LLM can edit code effectively in an unconstrained setting.* 

- The **Karpathy Agent (Code)** is the sole pure‑LLM approach that narrows the gap to classical optimizers, achieving **val_bpb = 0.9814** with the **27 B** model. When the same agent uses the **0.8 B** model, performance drops to **val_bpb = 0.9910**, illustrating that model capacity is critical for meaningful code edits.

- In contrast, fixed‑HP methods show negligible scaling benefit: the 0.8 B variant records **val_bpb = 0.9904** versus **0.9908** for the 27 B version.

- **Figure 2** visualizes wall‑time trajectories for the two model sizes. Solid lines (27 B) outperform dashed lines (0.8 B) in the unconstrained regime, while both remain comparable for hybrid optimization, where the smaller model still suffices.

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/figure2.png) 
*Figure 2 compares wall‑time performance of 0.8 B and 27 B LLM optimizers; classical baselines (TPE, Random) are shown for reference.*

- A preliminary benchmark against **Gemini 2.5 Flash** (see Section A.2) revealed no improvement over the open‑weight Qwen3.5‑27 B baseline, suggesting that current frontier models have not yet surpassed the open‑weight reference for this task.

--- 

### Hybrid Centaur Beats All 

**Takeaway:** *Centaur delivers the best overall validation score while invoking the LLM in only 30 % of trials.* 

- By feeding the LLM the full CMA‑ES state—mean **μ**, step‑size **σ**, and covariance matrix **matrix C** —along with the top‑5 configurations and the most recent 20 trials, Centaur overrides CMA‑ES proposals in **100 %** of cases for the 27 B model and **95 %** for the 0.8 B model. Despite constituting just 30 % of total trials, LLM‑driven suggestions account for **25 %** of the improvements to the incumbent best configuration.

- **Stability improves dramatically:** seed variance drops from **std = 0.0036** (CMA‑ES alone) to **std = 0.0005** with Centaur.

- The cheaper **0.8 B** Centaur variant attains **val_bpb = 0.9766**, marginally better than the 27 B version ( **0.9763**), indicating that a modest‑sized LLM suffices when paired with a strong classical base.

- **Ablation of the LLM ratio**r**** shows optimal values of **r=0.2** for the 0.8 B model (best single‑seed **val_bpb = 0.9735**) and **r=0.5** for the 27 B model (best **val_bpb = 0.9745**). Pushing **r** to **0.8** degrades performance sharply (0.8 B: **val_bpb = 0.9789**; 27 B: **val_bpb = 0.9903**), confirming that excessive reliance on the LLM harms the search.

- The hybrid loop is formalized in **Algorithm 1**, which emphasizes that the LLM acts as an occasional, informed perturbation rather than the primary driver of the search.

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/algorithm1.png) 
*Algorithm 1: Centaur – the hybrid optimization loop that combines CMA‑ES with periodic LLM refinements.*

--- 

### Key Takeaways and Future Directions 

1. **Classical optimizers dominate** when the search space is fixed; low OOM rates (< 16 %) are more predictive of success than raw diversity. 
2. **Unconstrained code editing** can approach classical performance, but only with a sufficiently large LLM (27 B); smaller models fail to produce useful edits. 
3. **Hybrid Centaur** offers the best trade‑off, achieving the lowest validation bits‑per‑byte while dramatically reducing seed variance and requiring the LLM in only a minority of trials.

The study focused on a single language‑model training task using open‑weight Qwen3.5 models. Extending the benchmark to a broader suite of tasks, evaluating frontier‑weight models, experimenting with alternative classical bases, and allowing the search space to evolve under LLM guidance are promising avenues for future work. 

**Bottom line:** In resource‑constrained hyper‑parameter optimization, a pragmatic hybrid that lets a classical optimizer steer the search while sprinkling in occasional LLM insights provides the most reliable path to high‑quality solutions.

### Convergence by Trial Number vs. Wall‑Clock Time 

The most informative way to compare optimizers is by **sample efficiency**—how many trials are needed to reach a given validation bits‑per‑byte (val_bpb). 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/figure3.png) *Figure 3 plots val_bpb against the cumulative number of trials for each method.* 
![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/figure4.png) *Figure 4 shows the same data on a wall‑clock axis (hours).*

Because LLM‑based optimizers must run an inference step before every trial, each trial consumes more wall‑clock time. Consequently, their curves look slower on the time axis even when they achieve comparable or better val_bpb per trial. This dual view makes it clear that **a method can be competitive in trial count while appearing lagged in wall‑clock time**—a nuance that underpins the qualitative analysis that follows.

---

### Frontier Model Comparison: Gemini 2.5 Flash 

**Takeaway:** Swapping in a more powerful LLM does not automatically improve optimizer performance. 

We replaced the Qwen 3.5‑27B backbone with the frontier Gemini 2.5 Flash model for three LLM‑based optimizers—Centaur, Karpathy Agent (Code), and LLAMBO (Paper)—while keeping classical baselines (TPE, Random) unchanged. The resulting val_bpb curves show that Gemini 2.5 Flash fails to surpass Qwen 3.5‑27B within the same training budget. This experiment confirms that **raw model size or frontier status alone is insufficient to close the gap to classical methods**; the optimizer’s algorithmic design remains the dominant factor.

---

### Incumbent Traces: Visualizing When Improvements Occur 

**Takeaway:** Incumbent traces reveal the exact moments each optimizer discovers a new best configuration. 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/figure6.png) *Figure 6 displays incumbent traces for each method, plotted against cumulative training time.* 
![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/figure7.png) *Figure 7 provides the same view for a different seed.* 

In these plots, grey dots mark every trial, colored dots highlight trials that set a new incumbent, and the staircase line connects the incumbents, forming the best‑so‑far trajectory. Each subplot selects the seed that achieved the highest final val_bpb for the corresponding method. By exposing the timing of breakthroughs, incumbent traces complement aggregate convergence plots and help explain why certain methods appear more efficient in trial‑count space.

---

### Qualitative Agent Behavior: How the LLM Injects Transformer Knowledge 

**Takeaway:** The LLM contributes high‑impact, transformer‑specific tweaks that CMA‑ES alone cannot infer. 

Consider **seed 0, trial 136** of Centaur, which produced a new incumbent with **val\_bpb = 0.9837**. CMA‑ES initially suggested:

- **WINDOW\_PATTERN = LLLL** 
- **DEVICE\_BATCH\_SIZE = 61** 
- **TOTAL\_BATCH\_SIZE = 133143** 
- **SCALAR\_LR = 0.208** 

The LLM overrode these to:

- **WINDOW\_PATTERN = SSSS** – selects an all‑short attention scheme, reducing memory pressure at depth 10, a choice rooted in transformer architecture knowledge. 
- **DEVICE\_BATCH\_SIZE = 64** and **TOTAL\_BATCH\_SIZE = 131072** – both are powers of two, aligning with GPU memory alignment and tensor‑core efficiency, whereas the CMA‑ES values are arbitrary and potentially hardware‑inefficient. 
- **SCALAR\_LR = 0.3** – moves the learning rate into a regime where successful hyper‑parameter clusters have been observed in prior transformer tuning data.

Across seed 0, **6 of the 24 incumbent improvements (25 %)** stemmed from LLM‑generated trials, while **LLM trials accounted for 88 of 275 total trials (≈ 32 %)**. This close match to the intended delegation ratio demonstrates that the LLM supplies impactful suggestions without dominating the search.

---

### Diversity Metrics Used for Evaluation 

**Takeaway:** All optimizers explore only a tiny fraction of the hyper‑parameter space. 

Diversity is measured on the **13 continuous hyperparameters** (the categorical `WINDOW_PATTERN` is excluded). Each dimension is normalized to the interval **[0, 1]**, and only successful (non‑OOM) trials are considered. The metrics are:

- **Spread:** mean standard deviation per hyperparameter across all trials; higher values indicate broader sampling. 
- **Pairwise:** mean Euclidean (L2) distance between every pair of configurations; higher values mean configurations are more mutually distinct. 
- **Step:** mean L2 distance between consecutive trials; larger values reflect bigger jumps in the search trajectory. 
- **Cells:** each hyperparameter is discretized into 5 equal‑width bins, yielding a **5¹³** ‑dimensional grid. The number of unique occupied cells is counted; the theoretical maximum is **5¹³ ≈ 1.2 × 10⁹** cells.

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/table3.png) *Table 3 reports these metrics for each optimizer, showing observed cell counts ranging from 29 to 805—far below the theoretical maximum.*

---

### LLM Ratio Ablation Study: Justifying the 30 % Target 

**Takeaway:** A 30 % delegation to the LLM yields the best trade‑off between exploration and exploitation. 

We varied the fraction **r** of trials delegated to the LLM and measured the best **val\_bpb** on seed 0. Table 4 summarizes the results for both the 0.8B and 27B LLM variants. The default setting **r = 0.3** (bolded) achieves the highest **val\_bpb** for each model. Increasing **r** to **0.5** or **0.8** degrades performance; notably, with the 27B LLM, **r = 0.8** performs worse than CMA‑ES alone. 

These trends indicate that **the classical CMA‑ES backbone must retain majority control**—it provides robust exploration—while the LLM supplies occasional, high‑impact guidance. The ablation thus validates the chosen 30 % ratio as the sweet spot between classical exploration and LLM‑driven exploitation.

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-200504-2603-24647-s-7e0125/table4.png) *Table 4 shows the best val_bpb for each delegation ratio, highlighting **r = 0.3** as optimal.*

### 0.8 B Model Diversity – a quick sanity check 

Table 5 aggregates three random seeds for each 0.8 B variant. The most relevant numbers are:

- **Centaur** – val bpb ≈ **0.9766 ± 0.0008**, OOM ≈ **15 %**, **Spread** = **0.133**, **Pairwise** = **0.651** 
- **Karpathy Agent (14 HPs)** – val bpb ≈ **0.9908 ≈ 0.0002**, OOM ≈ **16 %**, Spread = **0.037**, Pairwise = **0.066** 
- **Karpathy Agent (Code)** – val bpb ≈ **0.9910 ± 0.0001**, OOM ≈ **20 %** (diversity not reported) 
- **LLAMBO (Optuna)** – val bpb ≈ **0.9873 ± 0.0003**, OOM ≈ **64 %**, Spread = **0.297**, Pairwise = **1.498** 
- **LLAMBO (Paper)** – val bpb ≈ **0.9894 ± 0.0007**, OOM ≈ **59 %**, Spread = **0.272**, Pairwise = **1.365** 

Both LLAMBO variants achieve the highest **Spread** and **Pairwise** scores, reproducing the diversity advantage observed with the 27 B models despite a higher out‑of‑memory rate.

### Why two LLAMBO versions? 

Table 6 lists the concrete divergences between the original paper implementation and the OptunaHub port used here:

- **Metric binarization** – the paper forces continuous metrics into 0/1 using a top‑20 % cut‑off; Optuna leaves them continuous. 
- **Surrogate labels** – present in the paper, omitted in Optuna. 
- **Hyper‑parameter encoding** – the paper embeds all hyper‑parameters in LLM prompts; Optuna samples categorical hyper‑parameters randomly. 
- **Failed‑trial handling** – the paper hides failed trials (TrialState.FAIL); Optuna exposes them to the surrogate model. 

These changes alter how the surrogate perceives the search space and how it treats unsuccessful trials. Nevertheless, **LLAMBO (Optuna)** attains a validation bits‑per‑byte of **0.9873 ± 0.0003**, only marginally lower than the paper’s **0.9894 ± 0.0007**, with comparable OOM percentages (**64 %** vs **59 %**).

### Bottom line 

The consistent diversity scores across model scales and the near‑identical validation quality between the two LLAMBO implementations together reinforce the robustness of the proposed **diversity‑driven search strategy**.