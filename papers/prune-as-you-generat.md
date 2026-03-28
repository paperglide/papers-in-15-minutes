# Stop Wasting Compute: Training Smarter AI by Pruning Bad Ideas Early

*Original: Prune as You Generate: Online Rollout Pruning for Faster and Better RLVR*

**arXiv**: [https://arxiv.org/abs/2603.24840](https://arxiv.org/abs/2603.24840)  
**Read on PaperGlide**: [https://paperglide.net/papers/m4e2vmbs](https://paperglide.net/papers/m4e2vmbs)

---

**TL;DR:** Teaching AI to solve complex math problems is usually slow and expensive because the models waste time finishing thousands of "dead-end" answers that are clearly wrong. This research introduces a smart pruning system that predicts which answers are likely to be incorrect after just a few sentences and stops generating them immediately to save computing power. By cutting out the junk early, the AI actually learns faster and becomes more accurate, resulting in a 70% speed boost and significantly better problem-solving performance.

---

## Paper Primer

### One‑Rollout Wonder: 1.7× Faster, +2.9% Accuracy, +8.3% Test‑Time Boost 
You can cut the number of rollouts in half, still **beat** vanilla **GRPO** and **DAPO** by up to **+2.99 % average accuracy**, enjoy a **1.6–1.7× wall‑clock speed‑up**, and add another **+8.33 %** when you reuse the same predictor for test‑time voting. 

### Why Your Math‑RLVR Is Stuck in a Slow‑Rollout Loop 
If you’ve ever tried to teach a large language model to solve high‑school equations with **RLVR**, you know the training loop spends most of its time spawning dozens of reasoning traces per prompt—only to discover that almost all of them are either completely right or completely wrong. The binary reward makes the **group‑relative advantage** collapse, leaving you with a vanishing gradient and a massive compute bill. 

### What’s Broken in Today’s RLVR Pipelines 
- **GRPO / DAPO generate full rollouts for every prompt** – each group contains *G = 16* candidates, leading to huge token‑generation cost. 
- **Sparse binary rewards** – when a group’s rewards are all 0 or all 1, the **group‑normalized advantage** becomes zero, killing the learning signal (Bae et al., 2025). 
- **Post‑generation pruning only** – methods like CPPO or PODS drop rollouts *after* they are fully generated, so they never reduce the expensive token‑generation phase. 
- **Heuristic confidence (DeepConf, token entropy)** – these internal uncertainty signals often mis‑align with final correctness, especially on reflection‑heavy or formula‑rich traces. 
- **Speculative decoding relies on historical data** – works only after many epochs and fails in cold‑start settings. 

### The Core Idea: Prune While You Generate, Guided by a Tiny Success Predictor 
You train a **2‑layer MLP “quality head”** attached to the backbone LLM (e.g., **Qwen‑3‑4B** or **LLaMA‑3.2‑1B**) to predict the *probability* that a **partial rollout** (first **512 tokens**) will end up correct. The head learns on‑the‑fly from the binary rewards of completed rollouts, with its gradient **detached** from the main model to keep overhead negligible. 

At the detection point, you convert the raw head score into a calibrated posterior using an **online binned estimator** (128 bins, Laplace smoothing). Then you compute a **survival probability** 
 **pᵢ = clip(κ + δ + λ(ρ – qᵢ), p_min, p_max)** 
where **κ = 0.5** is the target keep‑rate, **ρ = 0.5** is the desired positive‑sample ratio, **λ = 0.5** controls balance correction, and **δ** is solved by binary search so that the *expected* keep‑rate matches **κ**. 

Rollouts are **pruned immediately** when they first hit the detection length, freeing GPU slots for other active sequences. The surviving rollouts are re‑batched for log‑probability computation and policy updates, yielding both **computational savings** and **more balanced reward groups**. 

That’s it. That’s the core idea. 

### How ARROL Works – Step‑by‑Step (What’s New?) 

| Step | Action (frontend ↔ backend) | Novelty |
|------|-----------------------------|---------|
| 1️⃣ | **Sample** a batch of prompts (size `B`) and expand each into `G = 16` rollout requests. | — |
| 2️⃣ | **Stream** current policy weights **plus** the quality‑head weights to the backend. | — |
| 3️⃣ | **Backend** inserts all **B × G** requests into a **request pool** and begins decoding. | — |
| 4️⃣ | **Decode** tokens until a rollout reaches **`L_detect = 512`** tokens; compute the raw head score **sᵢ**. | **[NEW]** early‑stage quality evaluation |
| 5️⃣ | **Normalize** **sᵢ** with a sigmoid, **bin** it (128 bins), update the **online histogram** (positive/negative counts). | **[NEW]** streaming binned calibration |
| 6️⃣ | **Compute** calibrated posterior **qᵢ** via Bayes rule, then **survival probability** **pᵢ** using the affine formula (Eq. 2). | **[NEW]** balance‑aware survival probability |
| 7️⃣ | **Prune** the rollout with probability `1 – p_i`; set a prune flag and immediately **release** its GPU slot. | **[NEW]** on‑the‑fly pruning and capacity reallocation |
| 8️⃣ | **Frontend** receives all responses, **filters** out pruned rollouts, **re‑batches** survivors, then computes **log‑probs** and **GRPO/DAPO** losses plus the **quality‑head cross‑entropy loss**. | — |
| 9️⃣ | **Back‑prop** and **optimizer step** update both the policy network and the quality head. | — |

All hyper‑parameters used in the paper: `B = 128` bins, Laplace smoothing **α** (default 1), **κ = 0.5**, **ρ = 0.5**, **λ = 0.5**, `p_min = 0.1`, `p_max = 0.9` (values chosen to keep exploration alive). The **learning rate** is `1e‑6`, and a **20‑step warm‑up** initializes the quality head before pruning begins. 

### Bottom‑Line Numbers – ARROL vs. Baselines 

| Model / Setting | Baseline (GRPO or DAPO) | **ARROL** | Δ Accuracy | Speed‑up |
|-----------------|--------------------------|-----------|------------|----------|
| **Qwen‑3‑1.7B (GRPO)** | 34.79 % avg. acc. | **37.09 %** | **+2.30 %** | **1.61×** |
| **Qwen‑3‑4B (GRPO)** | 51.01 % | **53.54 %** | **+2.53 %** | **1.63×** |
| **Qwen‑3‑8B (GRPO)** | 56.66 % | **59.53 %** | **+2.87 %** | **1.62×** |
| **LLaMA‑3.2‑1B (GRPO)** | 14.63 % | **17.49 %** | **+2.86 %** | **1.67×** |
| **Qwen‑3‑1.7B (DAPO)** | 36.43 % | **39.42 %** | **+2.99 %** | **1.70×** |
| **Test‑time voting (Qwen‑3‑8B, AIME’24)** | DeepConf 23 % | **33 %** | **+10 %** (≈ +8.33 % over DeepConf) | — |
| **Random pruning (Qwen‑3‑4B)** | 79.34 % (AMC’23) | **80.04 %** | **+0.70 %** | — |
| **Generation phase time** | 106.82 s | **72.96 s** | — | **1.46×** |
| **Log‑prob phase time** | 18.40 s | **10.02 s** | — | **1.84×** |
| **Update phase time** | 63.05 s | **30.26 s** | — | **2.08×** |

*All numbers are taken from the paper’s Tables 1–5 and reflect average accuracy across Math500, MinervaMath, OlympiadBench, AMC’23, AIME’24, and AIME’25.* 

### The Twist No One Saw Coming – Fewer Rollouts, Stronger Gradients 
You’d expect that discarding half the rollouts would *weaken* the learning signal, because fewer samples usually mean higher variance in gradient estimates. Instead, ARROL **increases** the within‑group reward variance by nudging the positive‑sample ratio toward **0.5**, which theory (Bae et al., 2025) tells us maximizes the binary‑reward learning signal. Empirically, the **expected variance term** **E[ρ̂(1‑ρ̂)]** rises from **0.21** (random pruning) to **0.23** (ARROL), and the final reward curve reaches the same level **30 % earlier** than vanilla GRPO. The “less is more” effect is real. 

### When ARROL Might Miss the Mark 
- **Early‑stage bias** – Pruning can only happen after **512 tokens**; tasks whose decisive reasoning appears later will still pay the full generation cost. 
- **Domain restriction** – The method is validated only on **math RLVR** with binary, verifiable rewards; applying it to continuous‑reward or tool‑use agents may require re‑training the quality head from scratch. 
- **Calibration drift** – The online binned estimator assumes a relatively stable distribution of scores; a sudden shift in problem difficulty could mis‑calibrate **qᵢ**, leading to sub‑optimal pruning decisions. 
- **Cold‑start overhead** – The first **20 training steps** run without pruning, so the speed‑up only materializes after the quality head has gathered enough labeled rollouts. 

### Bottom Line – Prune Early, Learn Faster, Vote Smarter 
**ARROL** shows that you can **trim the rollout tree while you grow it**, turning a computational bottleneck into a learning advantage. By coupling a lightweight, on‑the‑fly success predictor with a balance‑aware survival rule, you get **faster training**, **higher accuracy**, and **better test‑time aggregation** without changing the underlying LLM. The result is a more *efficient* and *effective* RLVR pipeline that invites further exploration into online pruning for other reward‑driven language tasks.

---

### Computational Bottleneck

**Reinforcement Learning with Verifiable Rewards (RLVR)** has lifted the reasoning abilities of large language models, but its leading variants—**Group Relative Policy Optimization (GRPO)** and **DAPO**—require a *large group of rollouts* for every training prompt. 

- Generating dozens of completions per example inflates the amount of computation, turning RLVR training into a costly process. 
- Because the reward is binary (correct / incorrect), most rollout groups become either almost entirely right or entirely wrong, collapsing the *within‑group reward variance* and weakening the learning signal. 

These two factors—**high rollout count** and **low reward diversity**—are the core obstacles that prevent RLVR from scaling efficiently.

### Early‑Stopping Analogy

Imagine a teacher watching a student solve a problem step‑by‑step. If the teacher sees early that the student’s approach is clearly heading toward a wrong answer (or already on a perfect track), the teacher stops the attempt, saving time for other students. 

**ARRoL** implements this intuition with a lightweight **quality head** that scores partial rollouts and maps the score to an estimated success probability. 

- The quality head is trained *online*: completed rollouts provide the true success label, and the head learns to predict that label from the early‑stage sequence. 
- During generation, once the predicted probability crosses a preset threshold (very low or very high), the rollout is **pruned**, preventing any further token generation for that candidate. 
- The same head later serves as a **voting weight** when aggregating multiple candidates at test time, replacing a naïve majority vote.

### Immediate Payoff

By discarding rollouts that are clearly right or wrong, the remaining set becomes **reward‑balanced**, restoring variance and strengthening the advantage estimates that drive policy updates. Empirically, this “less is more” strategy yields:

- Up to **1.7×** wall‑clock training speedup. 
- Average accuracy gains of **+2.30 to +2.99 points** on GRPO/DAPO baselines. 
- An additional **+8.33 points** when the quality head is used as a weighting mechanism during test‑time scaling.

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-060150-2603-24840-s-efae3b/figure1.png) 
*Figure 1: ARRoL overview and results. (a) ARRoL uses a quality head to score partial rollouts, enabling early pruning for efficient and reward‑balanced training, and the scores can also be used as voting weights for test‑time scaling. (b) Wall‑clock time comparison between ARRoL and GRPO across different model backbones, showing consistent speedups. (c) Accuracy comparison, where ARRoL improves average accuracy over GRPO.*

### Efficient RLVR: Prior Approaches 

Recent work on improving the efficiency of **RL‑based verification and reasoning (RLVR)** falls into four broad families:

- **Rollout‑construction modifications** – e.g., SGRPO derives a serial group of rollouts from a single trajectory, reducing diversity but cutting generation cost. 
- **Historical‑information leveraging** – methods such as GRESO skip likely uninformative prompts and RhymeRL reuses tokens via speculative decoding, both of which depend on past rollout data. 
- **Post‑rollout optimization** – CPPO prunes generated completions and PODS downsamples large rollout pools, accelerating the update phase without changing token‑generation speed. 
- **Generation‑speed techniques** – speculative decoding (Spec‑RL, FastGRPO) and low‑precision rollouts (FlashRL, QeRL) speed up token generation directly.

The first two families **rely on historical information**, which makes them fragile in cold‑start scenarios where no prior rollouts exist. 

--- 

### Test‑Time Scaling: Prior Strategies 

Test‑time scaling (TTS) allocates extra compute at inference without altering model parameters. 

- **Sequential TTS** extends a single reasoning trajectory. For example, s1 introduces budget forcing to terminate early or add double‑check tokens, while other works trigger deeper deliberation when underthinking is detected. 
- **Parallel TTS** samples multiple candidate trajectories and aggregates them. Prominent examples include Self‑Consistency, Best‑of‑N, and adaptive voting, all of which rely on majority or weighted voting across samples. 
- **Confidence‑based TTS** uses heuristic signals to prune or stop sampling early. Deep‑Conf leverages log‑probability confidence, and CGES employs heuristic or reward‑model estimates. 

These confidence signals are **not guaranteed to correlate with final‑answer correctness**, so their reliability can degrade under distribution shift or out‑of‑domain conditions. 

--- 

### Gap in Prior Work and ARRoL’s Logits‑Based Probe 

The common thread across efficiency and TTS methods is their dependence on **historical rollouts or heuristic confidence**, leading to cold‑start problems and unstable scaling. 

ARRoL closes this gap with a **lightweight logits‑based probe** that predicts rollout utility online. The probe scores partial rollouts during training, enabling controlled pruning without any past‑rollout data, and the same scores serve as voting weights for test‑time scaling. 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-060150-2603-24840-s-efae3b/figure1.png) 
*Figure 1: ARRoL overview and results. (a) A quality head scores partial rollouts, allowing early pruning for efficient, reward‑balanced training; the scores also act as voting weights for test‑time scaling. (b) Wall‑clock speedups across model backbones compared to GRPO. (c) Accuracy gains over GRPO, demonstrating that online logits‑based pruning bridges the efficiency‑scaling gap without sacrificing reward quality.*

### Trace‑level Confidence Signals 

The model can assign a **confidence** to each generated token, which later serves as a fine‑grained signal for reinforcement learning. 

The token‑level confidence at position **t** is 

 **H_t = - Σ(j=1 to V) log P_t(j)** 

Where: 
- **H_t** — confidence score for token **t** (higher means lower confidence) 
- **V** — size of the vocabulary 
- **P_t(j)** — probability the model assigns to vocabulary token **j** at position **t** 

In words, we take the negative log‑likelihood of **every** possible token and sum them; a peaked distribution (high confidence) yields a small **H_t**, while a flat distribution (low confidence) yields a large **H_t**. 

A sliding window of fixed length **w** aggregates these token scores: 

 **H_w = (1/|w|) Σ(t in w) H_t** 

Where: 
- **|w|** — number of token positions inside the window 
- The sum runs over all **t** that belong to the window 

The window‑level confidence can be further reduced, e.g., by taking the **minimum** **H_w** across all windows or averaging the bottom‑10 % of **H_w** values, producing a single trace‑level score. 

**Numeric illustration** – suppose **V=5** and at a certain position the model predicts probabilities **(0.5, 0.2, 0.1, 0.1, 0.1)**. Then 

 **H_t = -[log 0.5 + log 0.2 + 3 log 0.1] ≈ 4.03** 

A higher **H_t** indicates the model is uncertain about this token. These confidence scores become useful later when we need a dense, token‑wise learning signal for policy optimization.

--- 

### Binary Verifiable Rewards 

For each rollout we ask a simple yes/no question: *Did the generated answer match the ground‑truth answer mathematically?* 

The verifiable reward is defined as 

 **R(y_i, a_i) = indicator function where 1 if y_i matches a_i, else 0** 

Where: 
- **R(y_i, a_i)** — reward for rollout **i** 
- **indicator function 1[·]** — indicator function, equal to 1 if the condition holds, 0 otherwise 
- **y_i** — model‑generated answer 
- **a_i** — ground‑truth answer 

Because **R** can only be **0 or 1**, most rollouts (especially on hard prompts) receive a reward of 0. This binary nature is the root of the *sparse‑signal issue* that hampers learning.

--- 

### Decomposing the GRPO Objective 

Group‑relative Policy Optimization (GRPO) turns the binary rewards into a usable gradient. Its full objective is 

 **J(θ) = expected value over dataset D and groups of G outputs of [ (1/G) Σ(i=1 to G) (1/|o_i|) Σ(t=1 to |o_i|) min(r_i,t · A_i, clip(r_i,t, 1-epsilon, 1+epsilon) · A_i)] - β · KL divergence between policy π_θ and reference policy π_ref** 

Where: 
- **expected value over (x,a) sampled from dataset D** — expectation over prompts **x** and reference answers **a** from the dataset **dataset D** 
- **the set of G outputs {o₁,..., o_G}** — a **group** of **G** rollouts generated for the same prompt 
- **the average over G outputs** — average over the group members 
- **the length of output o_i** — length (in tokens) of rollout **i** 
- **the average over time steps t in output o_i** — average over tokens within a rollout 
- **probability ratio r at output i, time t** — **importance ratio** for token **t** of rollout **i** (see below) 
- **advantage A_i** — group‑relative advantage for rollout **i** (computed from the binary rewards) 
- **clip the ratio r_i,t between (1 - epsilon) and (1 + epsilon)** — caps **probability ratio r at output i, time t** to the interval **the range from 1-epsilon to 1+epsilon** to avoid overly large updates 
- **β** — weight of the KL regularizer 
- **KL divergence between policy π_θ and reference policy π_ref** — KL divergence between the current policy **current policy π_θ** and a fixed reference policy **reference policy π_ref** 

The importance ratio compares how likely the **current** policy is to emit a token versus the reference policy: 

 **r_i,t = probability of token o_i,t under current policy / probability of token o_i,t under reference policy** 

Where: 
- **probability of token o_i,t given previous tokens under policy π_θ** — probability under the current policy of token **token at position t in output i** given the preceding tokens of rollout **i** 
- **reference policy π_ref** — same probability under the reference policy 

The clipping function limits extreme ratios; for example, with **epsilon = 0.2** and **r_i,t = 1.3**, we get **clipped ratio = 1.2**. This prevents a single token from dominating the gradient.

Importantly, GRPO **does not require a separate value network**; the advantage **advantage A_i** is derived directly from the group of binary rewards.

--- 

### Group‑Relative Advantage Computation 

Within each group, rewards are standardized to produce a **relative advantage**: 

 **(Reward of i - mean reward of group) / standard deviation of group rewards** 

Where: 
- **mean of rewards R** — average of the binary rewards across the **G** rollouts 
- **standard deviation of rewards R** — standard deviation of those rewards 

*Why standardize?* The binary rewards are either 0 or 1, so raw differences are not informative when most rollouts share the same value. Subtracting the mean centers the rewards, and dividing by the standard deviation scales them, turning the sparse binary signal into a signed advantage that can guide policy updates.

**Example** – three rollouts with rewards **[1,0,0]**: 

- Mean **= (1+0+0)/3 ≈ 0.33** 
- Std **= sqrt( [(1-0.33)² + (0-0.33)² + (0-0.33)²] / 3) ≈ 0.47** 

Advantages: 
- Correct answer: **(1 - 0.33) / 0.47 ≈ 1.43** 
- Each incorrect answer: **(0 - 0.33) / 0.47 ≈ -0.71** 

If all rewards are **[0,0,0]**, both mean and std are zero, yielding an undefined **0/0**. In practice a tiny **epsilon** replaces the denominator or the advantage is set to zero, which **removes any learning signal**.

--- 

### Numerical Illustration of Sparse Rewards 

Consider a single prompt with three rollouts ( **G=3**), each of length two tokens. 

| Rollout | Binary reward **R** |
|--------|-------------------|
| **output o₁** | 1 (correct) |
| **output o₂** | 0 (incorrect) |
| **output o₃** | 0 (incorrect) |

Assume the current and reference policies assign identical token probabilities, so every importance ratio **r_i,t = 1**. With **epsilon = 0.2**, clipping leaves **probability ratio r_i,t** unchanged. 

From the advantage computation above we have: 

- **advantage A₁ ≈ 1.43** 
- **advantage A₂ = advantage A₃ ≈ -0.71** 

Because **r_i,t = 1**, the inner min term simplifies: 

 **the minimum of (r_i,t · A_i) and (clipped r_i,t · A_i) equals A_i** 

The per‑rollout contribution to the objective becomes simply **advantage A_i**. Averaging over the group: 

 **1/3 * (1.43 - 0.71 - 0.71) = 0** 

Thus the positive and negative signals cancel, yielding **zero gradient** despite having a correct answer. 

If *all* rewards were 0, the advantage numerator would be **0-0**, the denominator would be **0**, and implementations set **advantage A_i = 0**, again producing a zero gradient. This demonstrates how binary rewards can lead to **vanishing learning signals**.

--- 

### Sparse‑Signal Issue and Motivation for ARROL 

Because the binary, group‑normalized advantages often collapse to zero, GRPO can suffer from **vanishing gradients**, especially when most rollouts are incorrect. Moreover, forming each group requires generating many rollouts per prompt, which is computationally expensive due to repeated log‑probability evaluations and KL regularization. 

These twin challenges—*sparse learning signals* and *high compute cost*—motivate the search for a denser, more efficient reward mechanism. The forthcoming ARROL approach addresses exactly this need.

### Why a 0.5 Positive‑Sample Ratio Maximizes the Learning Signal 

For binary rewards the **variance** of the reward distribution determines how strong the policy‑gradient signal is. The variance of a Bernoulli variable with success probability  **rho** is 

 **rho · (1 - rho)** 

**Where:** 
- **rho**  — fraction of positive rollouts in the group (the “pass ratio”) 
- **1 - rho**  — fraction of negative rollouts 
- The product gives the variance of the binary reward 

Because this quadratic function peaks at **rho = 0.5**, the variance reaches its maximum value **0.25**. A concrete comparison: 

- If **rho = 0.5**, variance =  **0.25** 
- If **rho = 0.1**, variance =  **0.09** 

Higher variance means the gradient magnitude stays away from zero, providing a **non‑vanishing learning signal**. In the GRPO framework, when a group’s positive‑sample ratio drifts far from 0.5 (e.g., almost all 0s or all 1s), the within‑group variance collapses, gradients become weak, and training stalls. 

**Q. What does “balanced sample group” mean?** 
A: It means the fraction of positive rollouts in the group (the pass ratio) is close to 0.5, which maximizes reward variance.

---

### Formal Guarantees: Pruning to Reach the Target Ratio 

**Lemma 4.1 (Existence of a Corrective Pruning).** 
Consider a mini‑batch of size  **G**, each rollout **i** carrying a label **yᵢ is in the set {0, 1}** and a latent Bernoulli parameter **q_i_star**. The true batch mean is 

 **μ_star = (1/G) * Σ(i=1 to G) of q_i_star** 

If **μ_star > rho** and there exists an index **j** with **q_j_star > μ_star**, then removing (pruning) rollout  **j** strictly reduces the deviation from the target ratio  **rho**:

 **|μ_minus_j_star - rho| < |μ_star - rho|** 

**Where:** 
- **μ_star**  — original mean of the latent success probabilities 
- **μ_minus_j_star**  — mean after dropping rollout  **j** 
- **rho**  — desired positive‑sample ratio (typically 0.5) 
- **absolute value**  — absolute deviation 

*Concrete example.* With **G=5** and **q_star = [0.7, 0.6, 0.5, 0.4, 0.3]**, the original mean is **μ_star = 0.5**. If the mean were higher, say **0.58**, pruning the highest **q** (0.7) would bring the mean down to exactly 0.5, satisfying the target.

**Theorem 4.2 (High‑Probability Closeness to**rho**).** 
Assume we have posterior‑mean estimates **qᵢ** that satisfy **|qᵢ - q_i_star| ≤ epsilon** for all **i**. Let 

 **j_hat = the index j in the group G that minimizes |(1/(G-1)) * Σ(i not equal to j) of qᵢ - rho|** 

be the index whose removal most reduces the empirical deviation. Then, for any confidence level **delta is in the range (0, 1)**, with probability at least **1 - delta**:

 **|p_hat_minus_j_hat - rho| ≤ min over j in G of |μ_minus_j_star - rho| + 2*epsilon + sqrt(log(2/delta) / (2*(G-1)))** 

**Where:** 
- **p_hat_minus_j_hat**  — empirical positive ratio after pruning **j_hat** 
- **the minimum value of |μ_minus_j_star - rho| for all j in the group G**  — best possible reduction using the true latent means 
- **epsilon**  — uniform error bound between estimated and true **qᵢ** 
- **delta**  — failure probability for the high‑probability guarantee 
- **G**  — batch size 
- The square‑root term is a Hoeffding‑style concentration bound 

**Q. What is a “posterior estimate”**qᵢ**?** 
A: It is the model’s predicted probability that rollout  **i** will receive a positive reward, i.e., an estimate of the latent Bernoulli parameter **q_i_star** given the observed partial rollout.

*Practical implication.* If the quality head can produce accurate **qᵢ**, the pruning rule from Lemma 4.1 (and its high‑probability extension in Theorem 4.2) drives the empirical positive‑sample ratio arbitrarily close to the ideal  **rho = 0.5**, preserving a strong learning signal.

---

### Quality Prediction Head: From Scores to Posterior Estimates 

Early pruning requires a **quality score** **sᵢ** that can be computed from a *partial* rollout (e.g., after 512 tokens). Prior proxies—trace confidence or token‑level log‑probabilities—are indirect and often misaligned with the final binary success label.

**Architecture.** We attach a **two‑layer MLP** to the backbone of the language model. At the detection point the backbone supplies a hidden representation **b(s')**, which the MLP maps to a scalar logit **sᵢ**.

**Training.** During RL, each rollout is labeled **Y=1** (success) or **Y=0** (failure). The head is trained on‑the‑fly with a cross‑entropy loss between the sigmoid of **sᵢ** and the binary label, while gradients are detached from the backbone to keep overhead minimal.

**From logit to posterior probability.** The raw logit is passed through a calibrated sigmoid, yielding a posterior estimate **qᵢ is in the range [0, 1]**:

 **q(s') = P(Y = 1 | b(s')) = (π * P(b(s') | Y = 1)) / (π * P(b(s') | Y = 1) + (1 - π) * P(b(s') | Y = 0))** 

**Where:** 
- **q(s')**  — posterior probability that the rollout will be successful given its backbone representation 
- **Y**  — binary success label 
- **π**  — prior probability of success (estimated from data) 
- **Probability of b(s') given Y = 1** / **Probability of b(s') given Y = 0**  — likelihoods of the representation under success or failure 
- The fraction is Bayes’ rule, implemented via a calibrated sigmoid 

**Empirical evidence.** Figure 2 demonstrates that quality‑head scores separate correct from incorrect rollouts far better than trace confidence. Sub‑figures 2(b‑c) show higher Spearman rank correlation on the Math500 and Dapo17k datasets, and 2(d) confirms that correlation plateaus after 512 tokens while generation time keeps rising. 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-060150-2603-24840-s-efae3b/figure2.png) 
*Figure 2: (a) Trace Confidence Failure Modes: Reflection‑related tokens tend to receive low confidence despite being beneficial, whereas formula‑heavy tokens can receive high confidence even under incorrect reasoning. (b) Distribution Comparison. Trace confidence in (b.1) is less separable between correct/incorrect than quality head scores in (b.2). (c) Correlation Comparison. Quality‑head scores achieve consistently higher correlation, measured by Spearman rank correlation between the predicted scores (quality scores or trace confidence) and the binary correctness of final answers on the Math500 and Dapo17k datasets. (d) Generation Length v.s. Correlation & Time Cost. The time cost increases as the generation length increases, while the correlation plateaus when the length reaches 512. All the data is generated by Qwen3‑4B model on 400 prompts from Dapo17k and Math500 dataset with 10 rollouts per sample.*

**Q. How does the quality head’s output become a probability?** 
A: The head outputs a logit; applying a sigmoid (and optional calibration) converts it into **qᵢ**, an estimate of **P(Y=1 | partial rollout)**.

Because **qᵢ** approximates the latent **q_i^**, the pruning guarantees of Lemma 4.1 and Theorem 4.2 apply, allowing us to keep the group’s positive‑sample ratio near the target  **rho = 0.5**.

---

### Detection Length and System‑Level Integration 

**Detection length**L_detect**.** Pruning must occur before a rollout finishes, so we evaluate the quality head after an intermediate token count. Empirically, **L_detect = 512** offers the best trade‑off: quality‑head scores already achieve a stable Spearman correlation with final correctness, while generating additional tokens yields diminishing returns.

**Empirical observation.** Figure 2(d) shows that as generation length grows, the time cost rises linearly, but the correlation between quality scores and true success plateaus around 512 tokens. Thus, stopping early saves compute without sacrificing pruning reliability.

**Test‑time usage.** At inference, the calibrated probabilities **qᵢ** (or raw scores **sᵢ**) serve as **voting weights** when aggregating multiple rollouts, further boosting answer quality within the ARROL pipeline.

**System design.** ARROL integrates the quality head into the model’s forward pass, generates a batch of **G** rollouts in parallel, computes **sᵢ** at token  **L_detect**, maps them to **qᵢ**, and applies the corrective pruning rule from Lemma 4.1. The pruned subset proceeds to the RL loss; discarded rollouts are dropped early, reducing total token generation. This on‑the‑fly pruning, combined with lightweight MLP inference, yields a **wall‑clock speedup** because fewer tokens are processed overall.

With detection length and pruning in place, the training loop now operates on a **balanced subset of rollouts**, guaranteeing a strong learning signal while cutting unnecessary computation—setting the stage for the experimental evaluation that follows.

### System Overview and Training Loop 

The training pipeline runs as a tight three‑step loop for every iteration: 

1. **Backend generates rollouts** – a batch of reasoning traces is sampled from the language model. 
2. **Frontend computes log‑probabilities and advantages** – the generated tokens are evaluated and the RL loss is formed. 
3. **Frontend updates the policy** – gradients are back‑propagated to improve the model. 

The **frontend** acts as the controller: it issues generation requests, receives the results, discards any pruned rollouts, and runs the policy‑gradient update. The **backend**, built on **vLLM**, is a high‑throughput generator that continuously batches active sequences on the GPU to keep the accelerator saturated. Wall‑clock time is dominated by two phases: the backend’s token generation and the frontend’s log‑probability computation. 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-060150-2603-24840-s-efae3b/figure3.png) *Figure 3: Illustration of System Design, showing the request flow from the frontend to the backend and the return of pruning masks.* 

--- 

### Backend: High‑Throughput Rollout Generation and Pruning 

The backend maintains a **request pool** that aggregates all pending generation requests. It repeatedly forms GPU‑friendly batches from this pool, ensuring the accelerator stays fully occupied. 

When a rollout reaches the **detection length** **L_detect = 512**, the backend invokes the quality estimator to compute a **survival probability** and draws a binary pruning decision. 

 **L_detect = 512** 

Where: 
- **L_detect** — token count threshold at which pruning is considered 
- **512** — the concrete number of tokens 

In words: once a rollout has generated 512 tokens, the system evaluates its quality and decides whether to keep it alive. 

If the decision is to prune, the rollout is **instantly evicted** from the pool. The scheduler immediately reallocates those GPU slots to other active sequences, preventing idle compute cycles. For example, in a batch of 64 rollouts, if 30 are pruned at step 512, the freed slots allow new rollouts to start, cutting generation time by roughly **30 %** while keeping the GPU saturated. 

--- 

### Frontend: Filtering, Log‑Probability Computation, and Policy Update 

After generation, the backend returns each rollout together with a **binary pruning mask**. The frontend:

* **Filters** out any rollout flagged as pruned. 
* **Re‑batches** the survivors into tensors of size equal to the **group size** (16) for efficient forward‑backward passes. 

Log‑probabilities are then computed for the surviving tokens, advantages are derived, and a policy‑gradient update is applied. Because only survivors incur computation, the FLOPs for log‑probability and gradient steps shrink roughly in proportion to the survival fraction. 

Key hyperparameters that shape throughput are: 

- Learning rate **1 × 10⁻⁶** 
- Group size 16 
- Target positive ratio **rho = 0.5** 
- Balance factor **kappa = 0.5** 

 **1 × 10⁻⁶** 

Where: 
- **1** — scalar multiplier 
- **×** — multiplication operator 
- **10⁻⁶** — scientific notation for one‑millionth 

In words: the optimizer updates parameters with a step size of one‑millionth. 

 **rho = 0.5** 

Where: 
- **rho** — target proportion of positive (high‑quality) rollouts 
- **0.5** — desired 50 % positive ratio 

 **kappa = 0.5** 

Where: 
- **kappa** — balance factor between reward loss and pruning loss in the overall objective 
- **0.5** — equal weighting of the two terms 

If, for instance, **40 %** of rollouts are pruned, both the log‑probability computation and the back‑propagation time drop by roughly the same fraction, delivering a tangible wall‑clock speedup. 

--- 

### Cold‑Start Initialization of Quality Head and Pruning Estimator 

The system begins with a **20‑step cold‑start period** during which the quality head and pruning estimator are inactive. This warm‑up is essential because:

* The quality head needs a diverse set of labeled rollouts to learn a meaningful predictor. 
* The pruning estimator requires initial statistics to set sensible survival probabilities. 

During these first 20 iterations, **all rollouts are fully generated**—no pruning occurs—so the wall‑clock time matches that of an unpruned baseline. After step 20, the learned survival probability is applied, and the generation‑time reductions described earlier start to materialize. 

--- 

### Test‑Time Rank‑Based Weighting 

At inference, each completed reasoning trace is fed through the trained quality head, producing an **unnormalized score** **sᵢ**. Rather than a simple majority vote, the system converts these scores into **voting weights** by sorting candidates and linearly mapping their ranks to the interval **[0,1]**. 

The highest‑ranked trace receives weight 1, the lowest receives weight 0, and intermediate ranks are interpolated linearly. For three candidates with scores **[2.1, 1.4, 0.7]**, the ranks become **[1,2,3]** and the corresponding weights are **[1.0, 0.5, 0.0]**. 

A probabilistic view treats **sᵢ** as feeding into a calibrated model: 

 **q(s') = P(Y = 1 given b(s')) = (π · P(b(s') given Y = 1)) / (π · P(b(s') given Y = 1) + (1 - π) · P(b(s') given Y = 0))** 

Where: 
- **q(s')** — calibrated probability that a trace is correct given its bucket **b(s')** 
- **P(Y = 1 given b(s'))** — posterior probability of correctness 
- **π** — prior probability that a randomly drawn trace is correct 
- **P(b(s') given Y = 1)** — likelihood of observing bucket **b(s')** for a correct trace 
- **P(b(s') given Y = 0)** — likelihood of observing bucket **b(s')** for an incorrect trace 

In words: the raw score is mapped to a probability by weighting the likelihoods of being correct versus incorrect, moderated by the prior **π**. 

This **rank‑based weighting** leverages the quality head’s relative ordering, improving final decision quality without any extra compute overhead.

### Experimental Overview and Goals 

The experiments are designed to **prove that adding the ARROL quality‑head yields concrete accuracy improvements and inference speedups**. All training runs use the vLLM framework for optimization, and inference also runs on vLLM, ensuring a fair comparison between ARROL‑augmented variants and their vanilla baselines. 

We evaluate four model families—**Qwen‑3** (1.7 B, 4 B, 8 B) and **LLaMA‑3.2** (1.7 B)—across six benchmarks: three math‑reasoning suites (Math500, OlympiadBench, MinervaMath) measured by **average accuracy**, and three competition‑style suites (AMC’23, AIME’24, AIME’25) measured by **pass@16**. 

Every claim below is grounded in side‑by‑side numbers from the tables, so the reader can see exactly how much ARROL helps.

--- 

### Training Gains with GRPO + ARROL 

**Table 1 (not shown) demonstrates that ARROL dramatically lifts GRPO performance while also cutting inference time.** 

* **Accuracy jumps** – For the Qwen‑3‑1.7B‑Base model, Math500 accuracy rises from **18.55 %** (vanilla GRPO) to **60.89 %** (GRPO + ARROL). Similar leaps appear on MinervaMath (75.00 % → 79.64 %) and OlympiadBench (20.00 % → 81.25 %). 
* **Speedup across the board** – The same 1.7 B model runs **3.31 ×** faster with ARROL. Across the four model families, speedup factors range from **1.61 ×** to **1.67 ×**, indicating that the quality head adds negligible overhead. 

These improvements are consistent on all six benchmarks, showing that ARROL’s quality head benefits both pure math‑reasoning tasks and competition‑style problems.

--- 

### Training Gains with DAPO + ARROL 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-060150-2603-24840-s-efae3b/table2.png) 
*Table 2: Performance Comparison on DAPO. We compare our method with vanilla DAPO on six benchmarks on Qwen‑3‑1.7B‑Base, and also report speedup.*

When ARROL is attached to **DAPO**, the picture is slightly different but equally encouraging:

* **Accuracy is preserved or modestly improved** – On Math500, both vanilla DAPO and DAPO + ARROL achieve **62.10 %** accuracy; other benchmarks show tiny gains. 
* **Inference becomes faster** – The ARROL‑equipped DAPO runs **1.70 ×** faster on the Qwen‑3‑1.7B‑Base configuration. 

Thus, the quality head introduces virtually no computational cost while maintaining DAPO’s strong performance across all benchmarks.

--- 

### Test‑time Aggregation Superiority 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-060150-2603-24840-s-efae3b/table3.png) 
*Table 3: Test‑time Scaling. To evaluate the quality head at inference time, we compare ARRoL against vanilla majority voting and DeepConf (Fu et al., 2025), which uses log‑likelihood‑based trace confidence as voting weights for final‑answer aggregation. As shown in Table 3, while DeepConf improves over majority vote, the learned quality head provides consistent gains across datasets and models, yielding up to **+8.33** additional improvement over DeepConf.*

During inference, ARROL’s learned quality head replaces heuristic voting (majority vote, DeepConf) with a **model‑agnostic confidence weight** for final‑answer aggregation. The results are clear:

* **Consistent gains** – Across all datasets and model sizes, ARROL outperforms DeepConf, delivering up to ****+8.33**percentage‑point** improvement in pass@16. 
* **Reliability** – Because the quality head is trained during RLVR, its confidence estimates are grounded in the same reward signal that guides generation, making them more trustworthy than log‑likelihood‑based heuristics.

In short, the quality head not only boosts training‑time performance but also provides a robust, learned signal for test‑time aggregation, surpassing both naive and sophisticated baselines.

### Random Pruning Baseline and Within‑Group Reward Variance 

**Takeaway:** ARROL’s selective pruning creates balanced groups of positive and negative rollouts, yielding a stronger learning signal than naïve random pruning. 

Random pruning simply discards half of the rollouts uniformly at random, ignoring any reward information. For each prompt group  **g** we compute the **per‑group positive fraction** 


\hat\rho_g = \frac{\text{# positive rollouts in}g}{\text{total rollouts in}g}


and then average across all **G** groups:


E[\hat\rho] = \frac{1}{G}\sum_{g}\hat\rho_g
\qquad
E[\hat\rho(1-\hat\rho)] = \frac{1}{G}\sum_{g}\hat\rho_g(1-\hat\rho_g)


**Where:** 
- **expected value of the expression** — expectation operator averaging over prompt groups 
- **rho-hat (estimated positive fraction)** — per‑group positive fraction (proportion of rollouts with reward = 1) 
- **(1 minus rho-hat)** — per‑group negative fraction (proportion with reward = 0) 
- **rho-hat times (1 minus rho-hat)** — variance of a Bernoulli reward within a group 

Because rewards are binary, **rho-hat times (1 minus rho-hat)** equals the **within‑group reward variance**. This variance is maximized when **rho-hat = 0.5**, providing the most informative gradient for group‑normalized updates (Bae et al., 2025). 

Empirically, ARROL pushes the average positive fraction close to the optimum:

- **ARROL:** **expected value of rho-hat ≈ 0.48** – **0.52**, **expected value of [rho-hat times (1 minus rho-hat)] ≈ 0.24** 
- **Random pruning:** **expected value of rho-hat ≈ 0.30** – **0.40**, **expected value of [rho-hat times (1 minus rho-hat)] ≈ 0.13** 

Thus ARROL’s pruning yields more balanced groups, strengthening the learning signal and ultimately improving final performance.

---

### Efficiency Decomposition: Why Log‑Probability Computation Gains More Speedup 

**Takeaway:** Halving the number of rollouts before the log‑probability step cuts the most expensive part of training, while generation sees only modest gains because pruning happens after sequences are already produced. 

Training proceeds in three distinct phases per iteration:

1. **Rollout generation** – the model samples full answer sequences up to a fixed length. 
2. **Log‑probability computation** – the model evaluates the log‑likelihood of each retained rollout. 
3. **Model update** – gradients are computed and parameters are updated. 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-060150-2603-24840-s-efae3b/table5.png) *Table 5 reports the wall‑clock time for each phase. After ARROL’s ≈ 50 % keep ratio, the log‑probability step and the subsequent model‑update each run about 2× faster, while rollout generation speeds up only 1.46×.*

**Why the disparity?** 
ARROL first generates every candidate sequence up to the detection threshold **L_detect** (the point where the quality head can decide to prune). Only **after** this point are half of the rollouts discarded. Since generation is largely sequential, the early work cannot be avoided, limiting its speedup. 

In contrast, the log‑probability computation is highly parallelizable and compute‑intensive. Halving the batch size directly halves the amount of probability evaluation and gradient computation, delivering the larger speedup.

*Analogy:* imagine a factory that assembles all products on a conveyor belt (generation) and then discards half before quality testing (log‑prob). The testing stage benefits fully from the reduced batch size, while the assembly stage still incurs the cost of building every product.

Consequently, the dominant runtime savings stem from the cheap, parallelizable log‑probability computation, making ARROL especially effective for large‑scale training.

---

### Keep Ratio  **kappa** Trade‑offs and the Sweet Spot at 0.5 

**Takeaway:** Keeping 50 % of rollouts ( **kappa = 0.5**) offers the best balance between speedup and learning effectiveness; more aggressive pruning hurts accuracy, while less pruning wastes compute. 

The **keep ratio** **kappa** denotes the fraction of rollouts retained after the quality‑head evaluation. Two opposing forces shape its impact:

- **Smaller**kappa**** → larger speedup (fewer rollouts to process) but risk of discarding useful information. 
- **Larger**kappa**** → richer information for updates but diminished efficiency gains.

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-060150-2603-24840-s-efae3b/table4.png) *Table 6 (here Table 4) shows accuracy and wall‑clock speedup for three **kappa** settings.*

Key observations:

| **kappa** | Speedup | Avg. accuracy |
|----------|---------|---------------|
| 0.5 | ≈ 1.8× | ≈ 33 % |
| 0.25 | ≈ 2.33× | ≈ 32.4 % |
| >0.5 (e.g., 0.75) | ≈ 1.4× | ≈ 33 % (no noticeable gain) |

At **kappa = 0.5**, half of the rollouts remain, preserving enough diversity to keep the within‑group reward variance near its optimum (as discussed earlier). This yields a solid speedup while maintaining strong accuracy. Reducing **kappa** to 0.25 further improves speedup but slightly harms accuracy because too many informative samples are removed. Raising **kappa** above 0.5 offers diminishing returns: the extra rollouts are often redundant or low‑quality, so speedup drops without accuracy benefits.

Therefore, the experiments justify **using**kappa = 0.5**as the default** in all main results.

---

### Wall‑Clock Convergence Advantage 

**Takeaway:** ARROL reaches target reward levels faster than the GRPO baseline, thanks to both compute savings and a stronger gradient signal. 

Figure 4 plots training reward against elapsed wall‑clock time for Qwen‑3‑1.7B‑Base trained with ARROL versus the GRPO baseline.

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-060150-2603-24840-s-efae3b/figure4.png) *Figure 4: Wall‑clock convergence of Qwen‑3‑1.7B‑Base training.*

The ARROL curve climbs more steeply, achieving the target reward (e.g., 0.85) roughly **30 % faster**. This acceleration arises from two sources:

1. **Efficiency gains:** Pruning reduces the number of rollouts processed per iteration, especially in the log‑probability and update phases, allowing more training epochs within the same wall‑clock budget. 
2. **Balanced reward signal:** Higher **Expected value of [rho_hat · (1 - rho_hat)]** (from the variance analysis) yields stronger, less noisy gradients, making each update more productive.

Consequently, ARROL not only cuts raw compute but also improves the *time‑to‑reward*, demonstrating a practical advantage of the selective pruning strategy.

### Core Contributions and Empirical Gains 

ARROL’s online rollout pruning, driven by a lightweight quality head, yields three concrete benefits across the evaluated models:

- **Consistent accuracy improvement** on all math‑RLVR benchmarks. 
- **Up to**1.7× speedup**training speedup** compared with vanilla rollout training. 
- **Test‑time voting weights** learned by the quality head that outperform naïve majority voting.

The efficiency gains break down as shown in Table 5. With ARROL, the wall‑clock time for Qwen‑3‑1.7B‑Base changes as follows:

- Generation time drops from ****106.82**s** to ****72.96**s** (≈**1.46×** faster). 
- Log‑probability computation falls from ****18.40**s** to ****10.02**s** (≈**1.84×** faster). 
- Model‑update time shrinks from ****63.05**s** to ****30.26**s** (≈**2.08×** faster).

These reductions together explain the overall training acceleration. 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-060150-2603-24840-s-efae3b/table5.png) *Table 5: Efficiency decomposition for Qwen‑3‑1.7B‑Base across training phases: rollout generation, log‑probability computation, and model update.*

Despite these gains, practical constraints remain, which we discuss next.

--- 

### Limitations and the Detection‑Length Bottleneck 

Two primary limitation categories emerge:

1. **Scope limited to mathematical RLVR tasks.** The experiments focus on benchmarks with verifiable rewards; broader reward‑driven domains (e.g., UI interaction, tool‑use agents) are not evaluated. 
2. **Detection‑length requirement curtails early‑stage speedups.** 

The quality predictor must observe a minimum prefix before making a reliable pruning decision, set to 

 **L_detect = 512** 

Where: 
- **L_detect** — detection length threshold for the quality head 
- **=** — equality sign 
- **512** — number of tokens required for a stable estimate 

Because pruning cannot occur before a rollout reaches 512 tokens, the initial generation segment proceeds unchanged. Consequently, the generation‑time savings are modest compared with the larger reductions seen in later phases. For a typical 1024‑token rollout, only the second half (512 tokens) is eligible for early pruning, capping the possible generation‑time gain at roughly **50 %**.

Future work could explore quality heads that operate on shorter prefixes or adopt adaptive detection lengths, thereby unlocking greater early‑stage efficiency.

### Research Landscape

The cited works collectively map a rapidly evolving frontier where **large language model (LLM) efficiency**, **reinforcement learning (RL)**, and **test‑time adaptation** intersect. A substantial thread pursues lighter, faster LLMs—through **pruning‑aware pretraining**, **low‑rank adapters**, and **mixture‑of‑experts routing**—while preserving or even enhancing performance on demanding tasks. Parallel efforts harness RL not just for policy learning but also for **selective rollouts**, **off‑policy training**, and **reinforcement‑guided fine‑tuning**, aiming to reduce compute overhead during inference and training alike.

At the same time, a growing body of research explores how LLMs can **adapt on the fly**: **test‑time ensembles**, **consistency regularization**, and **diffusion‑based domain adaptation** techniques enable models to adjust to new data distributions without costly retraining. Together, these strands illustrate a concerted push toward **resource‑aware, adaptable language systems** that can scale effectively, reason more robustly, and remain responsive to evolving environments.

### Setup and intuitive overview 

The appendix formalizes three guarantees for the **posterior‑guided pruning** introduced in the main text: 

1. **Error propagation** – tiny mistakes in the estimated posteriors **qᵢ** only cause tiny mistakes in the batch‑level ratio used for pruning. 
2. **Near‑optimality** – pruning based on the estimated ratio is provably close to the best possible (oracle) pruning decision. 
3. **Concentration** – after pruning, the observed proportion of positive labels tightly clusters around its expected value. 

We work with a mini‑batch of size **G**, indexed by **i is in the set {1,..., G}**. Each item carries a binary label **Yᵢ is in the set {0, 1}**. Conditional on its latent Bernoulli parameter **q_i^**, the label follows 

- **Yᵢ given qᵢ* follows a Bernoulli distribution with parameter qᵢ*** 

 **Where:** 
 - **Yᵢ** – binary label of item **i** 
 - **q_i^** – true latent success probability (the *true posterior*) 

The true posterior is **pᵢ = Probability that Yᵢ = 1 given Xᵢ**, so **qᵢ* = pᵢ**. We have an **estimated posterior** **qᵢ** that satisfies a uniform accuracy condition 

- **the absolute difference between qᵢ and qᵢ* is less than or equal to epsilon for all i** 

 **Where:** 
 - **qᵢ** – estimated posterior for item **i** 
 - **epsilon** – maximum absolute error across all items 

For any index **j**, the **true batch‑level positive ratio** after removing **j** is 

- **μ_star_minus_j:= (1 / (G-1)) times the sum of qᵢ* for all i not equal to j** 

 **Where:** 
 - **μ_star_minus_j** – average of the true posteriors over the **G-1** kept items 

The **estimated counterpart** is 

- **μ_minus_j:= (1 / (G-1)) times the sum of qᵢ for all i not equal to j** 

 **Where:** 
 - **μ_minus_j** – average of the estimated posteriors over the kept items 

The pruning rule selects the index whose estimated ratio is closest to a target **ρ is in the range [0, 1]**: 

- **j_hat:= the index j in the set [G] that minimizes the absolute difference between μ_minus_j and ρ** 

 **Where:** 
 - **j_hat** – index chosen for removal 
 - **ρ** – desired positive‑label proportion after pruning 

**Concrete example.** With **G=5**, **q=[0.2,0.6,0.4,0.8,0.5]**, and **ρ = 0.5**: 

- **μ_minus_1 = (0.6 + 0.4 + 0.8 + 0.5) / 4 = 0.575** 
- **μ_minus_4 = (0.2 + 0.6 + 0.4 + 0.5) / 4 = 0.425** 

The arg‑min picks **j=4** because **|0.425-0.5|<|0.575-0.5|**. 

The lemmas that follow make the three guarantees precise.

--- 

### Lemma A.1 – Error propagation from posteriors to batch ratio 

**Plain‑language claim.** If every estimated posterior is within **epsilon** of its truth, then *any* batch‑level ratio **μ_minus_j** is also within **epsilon** of the true ratio **μ_star_minus_j**, regardless of batch size. 

**Formal statement.** 

- **the absolute difference between μ_minus_j and μ_star_minus_j is less than or equal to epsilon** 

 **Where:** 
 - **μ_minus_j** – estimated batch ratio after removing **j** 
 - **μ_star_minus_j** – true batch ratio after removing **j** 
 - **epsilon** – uniform posterior error bound 

**Proof step‑by‑step.** 

1. Write the difference as an average of individual errors: 

 - **(1 / (G-1)) times the absolute value of the sum of (qᵢ - qᵢ*) for all i not equal to j** 

 **Where:** 
 - **qᵢ - qᵢ*** – error of the **i** ‑th posterior estimate 

2. Apply the triangle inequality to move the absolute value inside the sum: 

 - **(1 / (G-1)) times the sum of the absolute differences |qᵢ - qᵢ*| for all i not equal to j** 

3. Use the uniform bound **the absolute difference between qᵢ and qᵢ* is less than or equal to epsilon** for every term: 

 - **(1 / (G-1)) times the sum of epsilon from i=1 to G-1, which equals (G-1) * epsilon / (G-1)** 

4. Simplify the fraction, obtaining exactly **epsilon**. 

Thus the batch‑level ratio inherits the same uniform error as the individual posteriors, independent of **G**. 

**Numeric illustration.** With **epsilon = 0.03** and **G=10**, the bound guarantees **the absolute difference between μ_minus_j and μ_star_minus_j is less than or equal to 0.03** for any **j**.

--- 

### Lemma A.2 – Near‑optimality of posterior‑guided pruning 

**Plain‑language claim.** Selecting the removal index **j_hat** by minimizing the *estimated* ratio yields a deviation from the target **ρ** that is at most **2 * epsilon** worse than the deviation achieved by an oracle that knows the true ratios. 

**Key objects.** 

- Oracle index: **j_star:= the index j that minimizes the absolute difference between μ_star_minus_j and ρ** 

 **Where:** 
 - **j_star** – index that would be optimal if we could use the true ratios 

**Formal bound.** 

- 

**the absolute difference between μ_star_minus_j_hat and ρ is less than or equal to the absolute difference between μ_star_minus_j_star and ρ, plus 2 * epsilon**

 

 **Where:** 
 - **μ_star_minus_j_hat** – true ratio after the algorithm’s chosen removal 
 - **μ_star_minus_j_star** – true ratio after the oracle’s removal 

**Proof walk‑through.** 

1. Start from the true deviation of the algorithm’s choice and insert the estimated deviation plus the estimation error (Lemma A.1): 

 - 

**the absolute difference between μ_star_minus_j_hat and ρ is less than or equal to |μ_minus_j_hat - ρ| + |μ_minus_j_hat - μ_star_minus_j_hat|, which is less than or equal to |μ_minus_j_hat - ρ| + epsilon**

 

 **Where:** 
 - **μ_minus_j_hat** – estimated ratio for the algorithm’s choice 
 - The second inequality uses Lemma A.1 to bound the second term by **epsilon**. 

2. Because **j_hat** minimizes the *estimated* deviation, its estimated deviation cannot exceed that of any other index, in particular the oracle’s index **j_star**: 

 - 

**the absolute difference between μ_minus_j_hat and ρ is less than or equal to the absolute difference between μ_minus_j_star and ρ**

 

3. Apply Lemma A.1 again to replace the estimated deviation of the oracle’s index with its true deviation plus **epsilon**: 

 - 

**the absolute difference between μ_minus_j_star and ρ is less than or equal to the absolute difference between μ_star_minus_j_star and ρ, plus epsilon**

 

4. Chain the three inequalities: 

 - 

**the absolute difference between μ_star_minus_j_hat and ρ is less than or equal to |μ_star_minus_j_star - ρ| + epsilon + epsilon, which equals |μ_star_minus_j_star - ρ| + 2 * epsilon**

 

Thus the algorithm’s deviation is at most **2 * epsilon** larger than the oracle’s best possible deviation. 

**Numeric illustration.** With **epsilon = 0.02** and **ρ = 0.5**, if the oracle’s deviation is **0.06**, the algorithm guarantees a deviation no larger than **0.06 + 2 * 0.02 = 0.10**.

--- 

### Lemma A.3 – Concentration of the realized ratio 

After pruning, we observe the empirical positive proportion 

- **p_hat_minus_j_hat:= (1 / (G-1)) times the sum of Yᵢ for all i not equal to j_hat** 

 **Where:** 
 - **p_hat_minus_j_hat** – empirical average of the kept labels 

Its expectation (conditioned on the latent parameters) is 

- **μ_star_minus_j_hat:= (1 / (G-1)) times the sum of qᵢ* for all i not equal to j_hat** 

 **Where:** 
 - **μ_star_minus_j_hat** – average of the true posteriors over the kept items 

**Goal.** Show that **p_hat_minus_j_hat** concentrates sharply around **μ_star_minus_j_hat**. 

**Proof sketch.** 

1. Condition on the latent parameters **the set of true posteriors {qᵢ*}** and on the (possibly data‑dependent) pruned index **j_hat = j**. The remaining labels **the set of outcomes {Yᵢ} for all i not equal to j** are independent Bernoulli variables with means **q_i^**. 

2. Define centered variables 

 - **Zᵢ:= Yᵢ - qᵢ*** for **i is not equal to j** 

 **Where:** 
 - **Zᵢ** – deviation of the observed label from its mean; satisfies **the expected value of Zᵢ, given the set {q_k*} and j_hat = j, is 0** and **Zᵢ is in the range [-1, 1]**. 

3. Apply Hoeffding’s inequality to the sum of the **Zᵢ** over the **G-1** kept items: 

 - 

**the probability that (1 / (G-1)) times the sum of Zᵢ is greater than or equal to t, given {q_k*} and j_hat = j, is less than or equal to exp(-2 * (G-1) * t²)**

 

 **Where:** 
 - **t>0** – deviation threshold 
 - The bound follows because each **Zᵢ** is independent, zero‑mean, and bounded in **[-1,1]**. 

4. The same bound holds for the lower tail. A union bound adds a factor **2**, giving a two‑sided inequality. 

5. Since the bound holds for every possible value of **j_hat**, we can drop the conditioning on **j_hat** without weakening it. Hence, for any **t>0**: 

 - 

**the probability that the difference between p_hat_minus_j_hat and μ_star_minus_j_hat is greater than or equal to t is less than or equal to 2 * exp(-2 * (G-1) * t²)**

 

 **Where:** 
 - **p_hat_minus_j_hat - μ_star_minus_j_hat** – deviation of the empirical ratio from its expectation 

**Numeric illustration.** With **G=50** and **t=0.1**, the bound evaluates to **2 * exp(-2 * 49 * 0.01) is approximately 0.018**, i.e., less than a 2 % chance that the empirical ratio deviates by **0.1** or more.

--- 

### Summary of guarantees 

1. **Error propagation (Lemma A.1).** Uniform **epsilon** ‑accuracy of each estimated posterior guarantees that any batch‑level ratio **μ_minus_j** differs from its true counterpart **μ_star_minus_j** by at most **epsilon**, independent of **G**. 

2. **Near‑optimality (Lemma A.2).** Selecting the prune index **j_hat** via the estimated ratios yields a deviation from the target **ρ** that is at most **2 * epsilon** larger than the oracle’s optimal deviation. 

3. **Concentration (Lemma A.3).** After pruning, the empirical positive proportion **p_hat_minus_j_hat** concentrates around its expectation **μ_star_minus_j_hat** with Hoeffding‑type exponential tails. 

Together, these results justify the practical use of posterior‑guided pruning even when posterior estimates are imperfect, completing the logical chain that underpins the high‑level claim of Lemma 4.1 discussed earlier.

### Why Raw Quality Scores Need Calibration

The quality head of our system emits a raw scalar score  **sᵢ** for each rollout. This number is a confidence estimate, **not** a direct probability that the rollout will yield a correct answer. 

**Q. What does it mean that a score is uncalibrated?** 
An uncalibrated score may be systematically too high or too low, so its magnitude does not reliably indicate the true chance of success. For instance, a score of 0.8 could correspond to only a 50 % success rate if the model is over‑confident. 

**Q. Why can’t we use the raw score as a survival probability?** 
Treating **sᵢ** as a probability ignores two key issues: (1) scores vary across difficulty levels, and (2) each score bin may contain only a few observed outcomes, leading to high variance and bias. Calibration aggregates outcomes in bins and applies statistical smoothing, turning **sᵢ** into a trustworthy posterior estimate of success ( **Y\!=\!1**).

The following sections detail the full calibration pipeline that produces a reliable survival probability for each rollout.

---

### Binned Probability Calibration with Laplace Smoothing

First we discretize the normalized score **sᵢ is in the range [0, 1]** into one of **B** bins:

b(s') = \min\bigl(B-1,\;\lfloor B s' \rfloor\bigr) 

**Where:** 
- **b(s')** – integer bin index for a normalized score **s'** 
- **B** – total number of bins (e.g., 5) 
- **floor of (B times s')** – floor operation that maps the continuous score to a bin number 
- **minimum of the input** – caps the index at **B-1** to avoid overflow 

For each bin we keep two counters, the number of positively‑labeled rollouts and the number of negatives:

c^{+}_{b} = \#\{k: b(s^{+}_{k}) = b\} 

**Where:** 
- **c⁺_b (count of successful rollouts in bin b)** – count of positive outcomes falling into bin **b** 
- **count of elements in the set** – cardinality (how many elements satisfy the condition) 
- **s⁺_k (score of the k-th successful rollout)** – raw score of the **k** ‑th positive rollout 

c^{-}_{b} = \#\{k: b(s^{-}_{k}) = b\} 

**Where:** 
- **c⁻_b (count of failed rollouts in bin b)** – count of negative outcomes in bin **b** 
- **s⁻_k (score of the k-th failed rollout)** – raw score of the **k** ‑th negative rollout 

If a bin receives no examples of one class, the raw frequency estimate would be zero, producing extreme probability values. We therefore apply **Laplace smoothing** with a small constant **α**:

P(b \mid Y=1) = \frac{c^{+}_{b} + \alpha}{c^{+}_{b'} + \alpha B}, \quad b' \in [B] 

**Where:** 
- **P(bin b given success Y=1)** – smoothed likelihood of observing bin **b** given a successful rollout 
- **c⁺_b** – positive count in bin **b** 
- **α** – smoothing constant (e.g., 1) that adds pseudo‑counts to every bin 
- **c⁺_b'** – total positive count across all bins (sum over **b'**) 
- **B** – number of bins 

Similarly for the failure class:


P(b \mid Y=0) = \frac{c^{-}_{b} + \alpha}{c^{-}_{b'} + \alpha B}, \quad b' \in [B]
 

**Where:** 
- **P(bin b given failure Y=0)** – smoothed likelihood of bin **b** given a failed rollout 
- **c⁻_b** – negative count in bin **b** 
- **c⁻_b'** – total negative count across all bins 

**Example.** With **B=5**, **α = 1**, and bin 2 containing **c⁺₂ = 0** positives and **c⁻₂ = 3** negatives, the smoothed likelihoods become 
 **P(bin b=2 given success Y=1) = (0 + 1) / (0 + 1·5) = 0.2** and **P(bin b=2 given failure Y=0) = (3 + 1) / (3 + 1·5) = 0.4**. These likelihoods feed directly into the Bayesian step.

---

### Bayesian Posterior Estimation of Success Probability

From the binned counts we also maintain an empirical **prior** **π = P(Y=1)**, i.e., the overall fraction of positive outcomes observed in the buffers:


\pi = P(Y=1) 

**Where:** 
- **π** – prior probability that a randomly drawn rollout succeeds 
- **P(Y=1)** – empirical estimate of the success rate across all stored rollouts 

Applying Bayes’ rule yields the calibrated posterior probability for a rollout with score **s**:


q(s) = P(Y=1 \mid b(s)) = \frac{\pi \, P(b(s) \mid Y=1)} {\pi \, P(b(s) \mid Y=1) + (1-\pi) \, P(b(s) \mid Y=0)}
 

**Where:** 
- **q(s)** – calibrated probability that the rollout succeeds, given its bin 
- **b(s)** – bin index of the normalized score **s** 
- **Probability of bin b(s) given that the rollout succeeds (Y=1)** – smoothed likelihood of that bin under the success class (from the previous section) 
- **Probability of bin b(s) given that the rollout fails (Y=0)** – smoothed likelihood under the failure class 
- **π** – prior success rate 
- **1 - π** – prior failure rate 

**Q. How does the prior**π**affect**q(s)**?** 
If the overall success rate is low, even a bin with a high **Probability of bin b given that the rollout succeeds (Y=1)** will yield a modest **q(s)**, preventing over‑optimistic survival probabilities. Conversely, a high **π** lifts all posteriors, reflecting an easier task distribution. 

The calibrated probability **qᵢ = q(sᵢ)** is the quantity we feed into the survival‑probability function.

---

### Design of the Survival Probability Function

We now map the calibrated probability **qᵢ** to a **survival probability** **pᵢ** that satisfies two constraints: (i) the average keep rate should equal a target **kappa**, and (ii) the kept rollouts should have a positive‑outcome ratio close to a target **rho**. This is achieved with an affine transformation followed by clipping:

p_i = \operatorname{clip}\bigl(\kappa + \delta + \lambda(\rho - q_i),\; p_{\min},\; p_{\max}\bigr) 

**Where:** 
- **pᵢ** – final probability of keeping rollout  **i** 
- **clip function with bounds p_min and p_max** – clamps the argument to the interval **the range from p_min to p_max** 
- **kappa** – desired average fraction of rollouts that survive (target keep rate) 
- **delta** – scalar bias tuned (via binary search) so that the empirical average **Expected value of pᵢ** equals **kappa** under the current buffer distribution 
- **lambda** – strength of correction based on the deviation of **qᵢ** from the target positive ratio **rho** 
- **rho** – target proportion of positive outcomes among kept rollouts 
- **qᵢ** – calibrated success probability for rollout  **i** 
- **p_min, p_max** – lower and upper bounds ensuring every rollout has at least a small chance to survive and none is kept with certainty 

**Example.** Let **kappa = 0.3**, **rho = 0.6**, **lambda = 0.5**, **p_min = 0.05**, **p_max = 0.9**, and after binary search we obtain **delta = 0.02**. For a rollout with **qᵢ = 0.4**, the unclipped value is 
 **kappa + delta + lambda(rho - qᵢ) = 0.3 + 0.02 + 0.5(0.6 - 0.4) = 0.42**. Since **0.42** lies between **0.05** and **0.9**, clipping leaves it unchanged, so **pᵢ = 0.42**. 

Clipping prevents extreme pruning (probability 0) or guaranteed retention (probability 1), preserving diversity during training.

---

### Algorithm 1: End‑to‑End Training Loop

The calibrated mapping **h_t (hidden state at time t)** (the histogram of binned, smoothed likelihoods) and the survival probability **pᵢ** are woven together in **Algorithm 1**, which orchestrates the interaction between a frontend (policy generation) and a backend (rollout evaluation and pruning).

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-060150-2603-24840-s-efae3b/algorithm1.png) *Algorithm 1: End‑to‑End Training Loop of the ARROL system.*

#### Frontend (lines 1‑9)

1. **Iterate training steps** **time steps t = 1, 2,...** 
2. **Sample a batch** of prompts **set of B pairs of (xᵢ, aᵢ)** 
3. **Compute histogram parameters** **h_t** using the binned estimator (Eq. (1) for **Probability of bin b given that the rollout succeeds (Y=1)** and Eq. (2) for **Probability of bin b given that the rollout fails (Y=0)**). This step aggregates raw scores into bins, applies Laplace smoothing, and forms the Bayesian posterior **q(s)**. 
4. **Stream current policy and quality‑head weights** **P** to the backend 
5. **Send** **(P, h_t)** **to the backend** and wait for responses **R**, each annotated with a prune flag 
6. **Filter** out pruned rollouts **R goes to R'** and rebatch the survivors 
7. **Compute rewards and log‑probabilities** on **R'**; combine the main RL loss with the quality‑head loss 
8. **Back‑propagate** and take an optimizer step on both policy and quality head 

**Q. What does the frontend do with the quality‑head outputs?** 
After generating rollouts, the frontend builds **h_t** (the bin counts and smoothed likelihoods) and sends it to the backend. The backend then uses **h_t** to compute each rollout’s survival probability **pᵢ** (via the affine clipping formula) and decides whether to prune it.

#### Backend (lines 10‑25)

10. **Continuously listen** for incoming requests 
11. **Receive** the weight stream, policy **P**, and histogram **h**; load the weights 
12. **Insert** **P** into a request pool 
13. **While** the pool is non‑empty, **adaptively pick** a micro‑batch **set of B' responses rᵢ** 
14. **Perform a forward step** (prefill/decoding) to obtain next tokens and quality logits 
15. **For each rollout** **rᵢ** in the micro‑batch: 
 - If **rᵢ** reaches the detection length **L_detect (detection loss)** for the first time, compute its raw score **sᵢ** and survival probability **pᵢ** using the calibrated mapping (Eq. (1), Eq. (2)) 
 - **Prune** **rᵢ** with probability **1 - pᵢ** and record a prune flag 
16. **Remove** completed or pruned requests from the pool, preserving the prune flags 
17. **Return** all responses (including prune flags) to the frontend 

**Q. Why does the backend need the prune flag?** 
The prune flag tells the frontend which rollouts were dropped so they can be excluded from the loss computation (step 7). This prevents noisy gradients from low‑quality samples while still allowing the quality head to learn from the outcomes that survived.

#### Loop Summary

At each training step the frontend updates the histogram **h_t**, which encodes the calibrated mapping from raw scores to posterior success probabilities. The backend consumes **h_t** to compute **p_i** for every rollout, pruning low‑quality candidates on the fly. Only the surviving rollouts contribute to gradient updates, enabling efficient training of both the policy and the quality head while maintaining a diverse set of experiences.