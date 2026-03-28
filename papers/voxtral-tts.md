# Voxtral TTS: Cloning Perfect Human Speech From Just Three Seconds of Audio

*Original: Voxtral TTS*

**arXiv**: [https://arxiv.org/abs/2603.25551](https://arxiv.org/abs/2603.25551)  
**Read on PaperGlide**: [https://paperglide.net/papers/06wlfi1i](https://paperglide.net/papers/06wlfi1i)

---

**TL;DR:** Voxtral TTS is a breakthrough AI system that can perfectly clone a person's voice using only a three-second audio clip, outperforming top industry competitors like ElevenLabs in listener tests. By separating the "what" of speech from the "how," it generates incredibly natural and expressive voices across nine languages without the robotic glitches common in older technology. This research matters because it makes high-quality, real-time voice cloning faster and more accessible, requiring significantly less data and computing power to achieve human-like results.

---

## Paper Primer

### Voxtral TTS Wins 68.4 % of Voice‑Cloning Head‑to‑Head Battles 
In blind A/B tests with native speakers, **Voxtral TTS** was preferred over **ElevenLabs Flash v2.5** in **68.4 %** of voice‑cloning instances—more than two‑thirds of the time—while using only a **3‑second** reference clip.

### Why This Matters to You 
If you’ve ever tried to generate a new voice from a short demo and ended up with robotic, flat‑sounding speech, you know how frustrating zero‑shot TTS can be. You also know that most commercial services demand long reference recordings or sacrifice expressivity for intelligibility. **Voxtral TTS** flips that script: it delivers natural, expressive speech from a whisper‑short prompt, works in nine languages, and runs fast enough for real‑time streaming.

### What Existing Systems Get Wrong 
- **Depth‑wise autoregressive acoustic decoders** (e.g., Moshi, Mimi) need 36 sequential steps per frame, inflating latency and compute. 
- **Pure diffusion or flow models** excel at acoustic detail but lack a discrete token interface, making them hard to combine with language‑model style text conditioning. 
- **Self‑supervised “semantic” tokenizers** capture phonetics but miss true linguistic meaning, limiting zero‑shot voice fidelity. 
- **High‑bitrate codecs** (≥ 4 kbps) waste bandwidth and still struggle with expressive prosody. 
- **Human‑centric metrics** (e.g., MOS predictors) often diverge from actual listener preference, leading to over‑optimistic benchmark scores.

### The Core Idea in Plain English 
The authors built a **two‑stage TTS pipeline** that treats speech like a **semantic‑plus‑acoustic pair**. First, a **Voxtral Codec** compresses a reference waveform into a **single semantic token** (VQ‑8192) and **36 acoustic tokens** (FSQ‑21 levels) per 80 ms frame—just **2.14 kbps**. Second, a **decoder‑only transformer** (initialized from **Ministral 3B**) autoregressively predicts the semantic stream, while a lightweight **flow‑matching transformer** continuously maps Gaussian noise to the acoustic embeddings conditioned on the semantic hidden state. The acoustic predictions are discretised back to FSQ levels and fed into the next AR step, keeping the whole system fully token‑based. **That’s it.** The clever part is swapping a slow, depth‑wise AR acoustic decoder for a fast, continuous flow model without breaking the token interface.

### How It Works, Step by Step 

| # | Operation | Novelty | Technical Details |
|---|-----------|---------|-------------------|
| 1 | **Reference encoding** | **[NEW]** | **Voxtral Codec**: 24 kHz mono → 240‑sample patches → 100 Hz frames → 4‑block CNN‑Transformer encoder (sliding windows 16→8→4→2, ALiBi, QK‑norm, LayerScale 0.01). Latent split: 256‑d semantic + 36‑d acoustic. |
| 2 | **Token quantisation** | **[NEW]** | Semantic → VQ (8192 codebook, 50 % applied). Acoustic → FSQ (21 uniform levels per dimension, 50 % FSQ, 25 % dither, 25 % pass‑through). Total bitrate = **12.5 · (log₂(8192) + 36 log₂(21)) ≈ 2.14**  kbps. |
| 3 | **Semantic generation** | **[NEW]** | Decoder‑only transformer (width = **Ministral 3B**) receives `<next>`‑separated voice tokens + text tokens. Predicts semantic logits over 8192 entries + `<EOA>` via cross‑entropy. |
| 4 | **Acoustic flow‑matching** | **[NEW]** | 3‑layer bidirectional transformer (same hidden width). Input: hidden state *h*, sinusoidal time *t*, current acoustic embedding *xₜ*. Predicts velocity **v_t = α v_θ(x_t, t, h) + (1 - α) v_θ(x_t, t, empty set)** with **α = 1.2**. Integrated by Euler with **8 NFEs** (Δt = 1/8) and classifier‑free guidance (CFG). Outputs discretised to FSQ levels. |
| 5 | **Waveform reconstruction** | **[OLD]** | **Voxtral Codec decoder** mirrors encoder (transposed CNNs, causal self‑attention) and reconstructs 240‑sample patches → 24 kHz audio. |
| 6 | **Training objectives** | **[NEW]** | Total loss = **α L_feature + β L_ASR + γ (L₁ + L_STFT) + δ L_commit** with **α = 1.0**, **β = 1.0**, **γ = 0.9999 to the power of t** (decays), **δ = 0.1**. <br>Acoustic flow‑matching loss: **Expected value over t, x₀, x₁ of ‖v_θ(x_t, t) - u_t‖²**, **u_t = x₁ - x₀**. <br>**Direct Preference Optimization (DPO)** applied post‑training: semantic DPO (β_semantic = 0.1) + acoustic DPO (β_acoustic = 0.5) with learning rate **8 × 10⁻⁸**. |
| 7 | **Serving pipeline** | **[NEW]** | Decomposed into **generation** and **codec** stages linked by an **asynchronous chunked streaming** protocol in **vLLM‑Omni**. CUDA‑graph captures the entire ODE solver, cutting latency by 47 %. |

### Numbers That Speak for Themselves 

| Metric | **Voxtral Codec** | **Mimi (16 cb)** | **Voxtral TTS (overall)** |
|--------|-------------------|------------------|---------------------------|
| Bitrate (kbps) | **2.1** | 2.2 | — |
| Mel distance (↓) | **0.545** | 0.618 | — |
| STFT distance (↑) | **0.982** | 1.100 | — |
| PESQ (↑) | **3.05** | 2.67 | — |
| ESTOI (↑) | **0.882** | 0.865 | — |
| ASR‑WER (%) (↓) | **10.66** | 11.01 | — |
| Speaker similarity (↑) | **0.843** | 0.829 | — |
| **Human win‑rate (voice cloning)** | — | — | **68.4 %** vs. ElevenLabs Flash v2.5 |
| **Human win‑rate (implicit steering)** | — | — | **58.3 %** vs. ElevenLabs Flash v2.5 |
| **Latency (CUDA‑graph)** | **70 ms** per 500‑char request | — | — |
| **RTF (CUDA‑graph)** | **0.103** | — | — |
| **Throughput @ 32 users** | **1 430 char/s/GPU** | — | — |

*All automatic scores are averages over SEED‑TTS and the nine MiniMax‑TTS languages.*

### The Twist No One Saw Coming 
You’d expect the **dense acoustic stream** to need a full autoregressive decoder—after all, every sample seems to depend on the previous one. **Voxtral TTS** shows the opposite: a **continuous flow‑matching model** with just **8 function evaluations** per frame matches or exceeds depth‑wise AR in human expressivity tests, while slashing compute by a factor of ~5. The result is a system that’s both **faster** and **more expressive**, contradicting the long‑standing belief that high‑quality acoustic detail requires step‑by‑step generation.

### Where It Still Falls Short 
- **CFG over‑adherence**: Raising the classifier‑free guidance scale improves automatic metrics but makes the model cling too tightly to the reference voice, hurting implicit emotion rendering. 
- **WER regression beyond 8 NFEs**: Adding more integration steps yields diminishing returns and a slight dip in intelligibility. 
- **Language scope**: Currently limited to **9** languages; performance on truly low‑resource or tonal languages is untested. 
- **Hardware demand**: The full **4 B‑parameter** model needs a high‑end GPU (e.g., NVIDIA H200) for real‑time streaming; on‑device deployment remains out of reach.

### Bottom Line 
**Voxtral TTS** proves that you can separate *what* is being said (semantic tokens) from *how* it sounds (continuous acoustic flow) and still keep everything in a clean, token‑based pipeline. This breaks the latency barrier of depth‑wise acoustic autoregression and delivers expressive, multilingual voice cloning from a **3‑second** prompt. The next wave of TTS research will likely explore even tighter integration of flow‑matching with large language models, and push the token‑rate lower while preserving the expressive richness demonstrated here.

---

### Zero‑Shot Text‑to‑Speech: The Core Challenge

Zero‑shot TTS asks a model to **synthesize natural‑sounding speech for a speaker it has never seen during training**, using only a brief voice prompt (often just a few seconds). The model must capture the target speaker’s timbre, prosody, and style without any speaker‑specific fine‑tuning.

**Analogy – script and performance.** 
Think of speech as a theatrical play. The **semantic token** is the script: the exact words and their order. The **acoustic token** is the actor’s delivery: pitch, rhythm, and timbre. By separating script from performance, a system can first generate a coherent script and then apply any desired performance style, exactly what zero‑shot TTS needs.

This factorization lets the **script** be generated autoregressively for long‑range linguistic consistency, while the **performance** is rendered with a continuous model that captures fine‑grained expressivity.

--- 

### Hybrid Architecture: Auto‑Regressive Semantics + Flow‑Matching Acoustics

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/figure3.png) *Architecture overview of Voxtral TTS. A voice reference (3 s–30 s) is encoded into audio frames at 12.5 Hz; each frame contains one semantic token and multiple acoustic tokens. The decoder generates semantic tokens, which feed a flow‑matching transformer that predicts acoustic tokens. The combined tokens are decoded back to waveform.*

The system processes speech in three stages:

1. **Voice prompt tokenization** – The short reference audio is fed to the Voxtral Codec encoder, producing a sequence of audio frames at **12.5 Hz**. Each frame **A** holds one semantic token and a set of acoustic tokens. 
2. **Semantic generation** – Text prompt tokens **T** are concatenated with the audio tokens and passed through a decoder‑only transformer. This transformer **autoregressively emits** a semantic token at every timestep until an **\<EOA\>** (End‑of‑Audio) token appears. 
3. **Acoustic prediction** – At each timestep, the newly generated semantic token conditions a lightweight **flow‑matching transformer**. Running several inference steps, this model predicts the corresponding acoustic tokens for that frame.

**Data flow in a nutshell**

- Text + audio tokens → decoder backbone → semantic token stream → flow‑matching → acoustic token stream → Voxtral Codec decoder → waveform.

The **semantic stream is discrete** (auto‑regressive), providing strong linguistic coherence. The **acoustic stream is continuous** (flow‑matching), delivering high‑fidelity expressive detail. This hybrid design also supports multilingual input and low‑latency streaming, though the inner workings of the codec are described next.

--- 

### Voxtral Codec: Tokenizing Speech into Semantic and Acoustic Streams

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/figure4.png) *Architecture and training of the Voxtral Codec. It splits speech into a semantic VQ codebook and multiple acoustic FSQ codebooks, then reconstructs the waveform from the combined tokens.*

The Voxtral Codec acts as a speech tokenizer that yields the factorized representation used by the hybrid architecture.

- **Token composition:** Each 12.5 Hz frame contains **1 semantic token** plus **36 acoustic tokens**, i.e., **37 tokens** per frame, achieving a bitrate of **2.14 kbps**. 
- **Patchified preprocessing:** A 24 kHz mono waveform is divided into non‑overlapping patches of **240 samples** (since 24 kHz ÷ 100 Hz = 240). This creates a **100 Hz** sequence of patches that feeds the encoder. 
- **Encoder pipeline:** 
 * A causal convolution (kernel 7) projects each patch to a **1024‑dimensional** embedding. 
 * Four encoder blocks follow. Each block contains a 2‑layer causal self‑attention transformer with sliding‑window attention (window sizes 16 → 8 → 4 → 2) and a causal CNN layer. The first three CNN layers downsample by **stride 2**, yielding an overall **8×** reduction from 100 Hz to **12.5 Hz**. The fourth CNN layer (stride 1) projects the representation to a **292‑dimensional** latent space. 
- **Hybrid quantization:** The latent vector is split into a **semantic VQ codebook** (distilled from a supervised ASR model) and multiple **acoustic FSQ codebooks**. This hybrid VQ‑FSQ scheme produces the 1 + 36 token format. 

*Example.* A 0.5‑second audio clip (12 000 samples) becomes 50 patches → after downsampling, roughly 6 frames are produced, each represented by 1 semantic token and 36 acoustic tokens.

--- 

### Training Objectives, Preference Optimization, and Evaluation Highlights

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/figure2.png) *Human‑evaluation win‑rate: Voxtral TTS is preferred over ElevenLabs Flash v2.5 in 58.3 % of flagship‑voice cases and 68.4 % of voice‑cloning cases.*

Voxtral TTS is trained with two complementary objectives:

- **Semantic branch:** A standard autoregressive language‑model loss maximizes the likelihood of the ground‑truth semantic token sequence given the decoder context. 
- **Acoustic branch:** A flow‑matching loss learns a continuous transformation that maps a simple Gaussian prior to the distribution of acoustic tokens conditioned on the semantic decoder states.

To align the system with human preferences, **Direct Preference Optimization (DPO)** is applied jointly:

- A preference term over semantic token generation encourages fluent, coherent scripts. 
- A flow‑based preference term over acoustic token prediction favors natural prosody and timbre.

**Key evaluation results**

- Supports **9 languages** and works with voice prompts as short as **3 seconds**. 
- Enables low‑latency streaming inference. 
- In human evaluations, achieves a **68.4 %** win rate against ElevenLabs Flash v2.5 (and 58.3 % for flagship voices), demonstrating superior naturalness and expressivity.

By **factorizing speech into discrete semantic and continuous acoustic streams** and assigning each to the modeling paradigm that best fits its nature, Voxtral TTS delivers both long‑range linguistic coherence and high‑fidelity expressive acoustic rendering.

### Latent Space Split: Semantic vs. Acoustic Dimensions 
**Key takeaway:** The encoder’s 292‑dimensional latent vector is deterministically divided into a 256‑dimensional semantic sub‑vector and a 36‑dimensional acoustic sub‑vector before any quantization takes place. 

The encoder emits a vector `z = [z₁,…,z₂₉₂]` for every 12.5 Hz frame. We simply take the first 256 entries as 

* `z_sem = [z₁,…,z₂₅₆]` – the semantic component, and the remaining 36 entries as 

* `z_ac = [z₂₅₇,…,z₂₉₂]` – the acoustic component. 

Because the split is performed right after the causal CNN encoder, the subsequent modules can treat linguistic content and fine‑grained sound characteristics independently while preserving the temporal causality of the original model.

--- 

### Hybrid VQ‑FSQ Quantization Mechanics 
**Key takeaway:** Semantic vectors are quantized with a learned 8192‑entry VQ codebook, while acoustic vectors undergo finite‑scalar quantization (FSQ) with 21 uniform levels per dimension, together yielding a bitrate of roughly **2.14 kbps**. 

* **Semantic quantization** – `z_sem` is fed to a learned vector quantizer (VQ) that stores 8192 prototype vectors. During training the VQ is applied with probability 0.5; otherwise the semantic vector bypasses quantization, encouraging robustness to stochastic quantization. 

* **Acoustic quantization** – `z_ac` first passes through a `tanh` non‑linearity, then each of its 36 dimensions is independently mapped to one of 21 uniformly spaced levels using FSQ. The training schedule mirrors the VQ’s stochasticity: 

 * 50 % of frames are quantized with FSQ, 
 * 25 % receive uniform noise of magnitude `1/L` (where `L = 21`), and 
 * 25 % are left untouched. 

* **Bitrate calculation** 

 **12.5 × (log₂(8192) + 36 × log₂(21)) ≈ 2.14 kbps** 

 **Where:** 
 - `12.5` Hz – frame rate of the codec, 
 - **log₂(8192) = 13**  bits – bits needed per semantic token (8192 codebook entries), 
 - `36` – number of acoustic dimensions, 
 - **log₂(21) ≈ 4.39**  bits – bits per acoustic dimension (21 FSQ levels), 
 - The product gives the average bits per second, i.e. the codec’s bitrate. 

* **Why independent quantization?** 
 Keeping the two streams separate lets the model preserve high‑level linguistic information (semantic) while still capturing subtle timbral nuances (acoustic). The quantizers do not interfere with each other, which simplifies training and improves reconstruction fidelity.

--- 

### ASR Distillation Loss: Q&A Walkthrough 
**Key takeaway:** The tokenizer learns text‑aligned semantic tokens by distilling hidden representations from a frozen Whisper ASR model, eliminating the need for paired transcripts. 

**Q. How can the tokenizer learn text‑aligned semantics without transcripts?** 
**A.** By minimizing a cosine‑distance loss between its semantic embeddings and the decoder hidden states produced by a frozen Whisper model that processes the same audio. 

The frozen Whisper model supplies: 

* Decoder hidden states **h_l** for each Whisper token `l`. 
* Cross‑attention weights that implicitly encode word‑level timing. 

After VQ, the semantic embeddings **z_f** are linearly projected to Whisper’s hidden dimension, yielding **vector z_f**. A soft‑alignment matrix **A ∈ ℝ^{L×F}** is built from selected Whisper cross‑attention heads (normalized, median‑filtered, averaged, and interpolated to the 12.5 Hz codec frame rate). 

The attention‑weighted semantic vector for Whisper token `l` is 

 **z_tilde_l = Σ(f=1 to F) of A[l,f] times vector z_f** 

**Where:** 
- **z_tilde_l** – semantic representation aligned to Whisper token `l`, 
- **A[l,f]** – soft alignment weight linking codec frame `f` to Whisper token `l`, 
- **vector z_f** – projected semantic embedding for frame `f`, 
- The sum aggregates contributions from all frames, weighted by their alignment to token `l`. 

The distillation loss is 

 **ASR loss = 1 - (1/L) * Σ(l=1 to L) of (z_tilde_l ⋅ h_l) / (‖z_tilde_l‖ * ‖h_l‖)** 

**Where:** 
- `L` – number of Whisper decoder tokens, 
- **z_tilde_l** – attention‑weighted semantic vector (above), 
- **vector h_l** – Whisper hidden state for token `l`, 
- **dot product** – dot product, 
- **norm (magnitude)** – Euclidean norm, 
- The fraction is the cosine similarity between aligned semantic and ASR hidden vectors; the loss is one minus the average similarity. 

Because the loss operates on continuous hidden states rather than hard transcript labels, it inherits Whisper’s confidence scores, phonetic nuances, and timing cues. This provides rich, implicit supervision that aligns the codec’s semantic tokens with spoken language without any external forced aligner.

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/figure4.png) 
*Figure 3: Architecture overview and training of Voxtral Codec. It shows the split semantic VQ codebook, acoustic FSQ codebooks, and the additional ASR distillation branch.*

--- 

### Adversarial Training: Multi‑Resolution Discriminators and Feature Matching 
**Key takeaway:** Eight STFT‑based discriminators, together with a feature‑matching loss, guide the generator toward perceptually realistic audio while respecting the stochastic quantization schedule. 

* **Discriminator ensemble** – Each discriminator processes either a real waveform `x` or a reconstructed waveform **x-hat** using a distinct Short‑Time Fourier Transform (STFT) window size: 2296, 1418, 876, 542, 334, 206, 126, and 76 samples. This multi‑resolution setup forces the generator to satisfy both coarse‑grained and fine‑grained spectral criteria. 

* **Hinge loss** – For each discriminator, a hinge loss encourages the scalar output to be positive for real audio and negative for generated audio, providing a simple binary classification signal. 

* **Feature‑matching loss** 

 **L_feature(x, x-hat) = 1/(MN) * Σ(m=1 to M) Σ(n=1 to N) of ||D_mⁿ(x) - D_mⁿ(x-hat)||₁** 

 **Where:** 
 - `N = 8` – number of discriminators, 
 - `M` – number of layers inside each discriminator, 
 - **D_mⁿ(·) (the nth feature map of the mth discriminator)** – activation of layer `m` in discriminator `n`, 
 - **||·||₁ (the L1 norm or absolute difference)** – L1 norm (absolute difference). 

 This loss measures the average L1 distance between intermediate feature maps of real and generated audio across all discriminators. 

* **Why feature‑matching?** 
 Instead of using the discriminator’s binary output as the generator’s loss, we match its internal representations. As discriminators evolve, their features become increasingly sensitive to subtle artifacts, giving the generator a richer, more stable reconstruction signal. 

* **Stability tricks** – Each transformer block in the generator employs LayerScale with an initial scale of **0.01**, and attention scores are normalized with **epsilon = 10⁻⁶** (QK normalization) to avoid division‑by‑zero. 

* **Quantization schedule** – The same stochastic schedule described earlier (50 % VQ, 50 % FSQ, 25 % dither, 25 % pass‑through) is retained during adversarial training, ensuring the generator sees both quantized and unquantized inputs. 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/table1.png) 
*Table 1: Key hyperparameters of the Voxtral Codec, including STFT sizes, LayerScale scale, and epsilon for QK normalization.*

--- 

### Combined Training Objective 
**Key takeaway:** The final loss is a weighted sum of five terms that jointly enforce audio realism, semantic alignment, and low‑bitrate fidelity. 

 **α L_feature + β L_ASR + γ L_L1 + γ L_STFT + δ L_commit** 

**Where:** 
- **α** – weight for the multi‑resolution feature‑matching loss, 
- **β** – weight for the ASR distillation (cosine‑distance) loss, 
- **γ** – shared weight for both the sample‑wise L1 reconstruction loss **L_L1** and the spectral STFT loss **L_STFT**, 
- **δ** – weight for the VQ commitment loss **L_commit**, 
- Each term pulls the model in a complementary direction: adversarial feature matching for perceptual quality, ASR distillation for text alignment, L1 and STFT for waveform fidelity, and commitment loss for stable quantization. 

The coefficients are tuned empirically; a larger **β** emphasizes semantic grounding, while **α**, **γ**, and **δ** balance audio realism and quantization stability. Optimizing this composite objective enables the codec to generate high‑quality, low‑bitrate audio whose semantic tokens are meaningfully aligned with spoken language.

### Multi‑Component Training Objective Overview 

The Voxtral Codec is optimized with a **single weighted sum of loss terms** that simultaneously encourages semantic fidelity, acoustic realism, and stable codebook usage. The overall objective is 

 **α L_feature + β L_ASR + γ L_L1 + γ L_STFT + δ L_commit** 

Where: 

- **α** — weight for the feature‑matching loss (semantic similarity). 
- **β** — weight for the ASR distillation loss (speech‑recognition consistency). 
- **γ** — weight for both reconstruction terms (L1 waveform and STFT magnitude). 
- **δ** — weight for the VQ commitment loss (encoder‑codebook alignment). 
- **L_feature** — L2 distance between predicted and target semantic embeddings. 
- **L_ASR** — cross‑entropy between predicted and teacher ASR transcripts. 
- **L_L1** — mean absolute error on raw waveform samples. 
- **L_STFT** — L1 loss on short‑time Fourier transform magnitudes. 
- **L_commit** — squared distance **‖z_e - stop_gradient(z_q)‖²** between the encoder output **z_e** and the stopped‑gradient codebook entry **z_q**.

**Why these weights matter** 

- **α = 1.0** and **β = 1.0** give **equal importance** to semantic and ASR objectives, ensuring the model learns a faithful linguistic representation. 
- **γ** follows an **exponential decay**: 

 **γ = 0.9999 to the power of t** 

 where **t** is the current training step. Early in training the reconstruction losses dominate, guiding the model toward plausible waveforms; later they fade as the adversarial signal (implicit in the flow‑matching loss) takes over. 

- **δ = 0.1** keeps the encoder tightly coupled to the discrete codebook without overwhelming the other terms.

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/table1.png) *Table 1: Key hyperparameters of the Voxtral Codec.*

Together, this composite loss balances **semantic accuracy**, **acoustic quality**, and **codebook stability** throughout training.

---

### Decoder Backbone and the Role of the Flow‑Matching Transformer 

The auto‑regressive decoder follows the **Ministral 3B** architecture: a decoder‑only transformer that consumes a mixed sequence of **voice‑reference audio tokens** and **text tokens**, then emits audio tokens one frame at a time.

- **Frame composition** – each audio frame is represented by **37 discrete tokens**: 1 semantic token (from an 8192‑entry codebook) and 36 acoustic tokens (each from a 21‑entry codebook). 
- **Embedding aggregation** – embeddings from the semantic and all acoustic codebooks are **summed** to produce a single vector per frame, which is fed into the transformer layers. 

Two prediction heads sit on top of the transformer:

1. **Semantic head** – a linear projection maps the hidden state **h** to logits over the 8192‑entry semantic vocabulary (plus an End‑of‑Audio token). It is trained with standard cross‑entropy. 

2. **Acoustic head** – a dedicated **flow‑matching (FM) transformer** receives **h** and outputs **continuous acoustic embeddings**. These floats are later quantized to the 21 FSQ levels, preserving a fully discrete token interface for the next auto‑regressive step.

The FM transformer thus **bridges the continuous world of velocity fields** with the discrete token loop required by the AR decoder.

---

### Intuition Behind Flow‑Matching: Modeling a Velocity Field 

Instead of directly predicting a discrete acoustic token, the FM transformer **learns a velocity field** that continuously transports a point from random Gaussian noise **x₀** to the target acoustic embedding **x₁**.

- **Time variable** **time t in the range 0 to 1** acts as a **continuous interpolation parameter**. Sinusoidal embeddings encode **t**, allowing the model to condition on any intermediate step. 
- **Why continuous?** A smooth velocity field is easier to learn than a jump between discrete codebook entries. The model only discretizes the final output, keeping the AR loop fully token‑based. 
- **Separate projections** – three linear layers map the hidden state **h**, the time embedding **t**, and the current acoustic embedding **x_t (the state at time t)** into a common space before they are summed inside the transformer. This prevents scale mismatches that could hinder training. 

**Analogy:** imagine a particle released in a fluid (the Gaussian noise). A time‑varying flow (the velocity field) gently pushes it until it reaches a destination (the acoustic embedding). The FM transformer learns the shape of that flow.

---

### Formal Flow‑Matching Mechanics and Conditional Objective 

During inference the FM transformer blends a **conditional** prediction (conditioned on the hidden state **h**) with an **unconditional** one (conditioned on a zero vector **empty set (representing the null or zero vector)**) using **classifier‑free guidance (CFG)**:

 **v_t = α · v_θ(x_t, t, h) + (1 - α) · v_θ(x_t, t, empty set)** 

Where: 

- **v_t (velocity at time t)** — guided velocity at time **t**. 
- **v_θ (the velocity prediction function)** — the FM transformer’s raw velocity prediction. 
- **α = 1.2** — guidance strength that amplifies the conditional signal. 
- **empty set** — a zero vector representing the unconditional branch.

**Euler integration** steps the embedding backward in time:

 **x at time (t - Δt) = x_t - v_t · Δt** 

- **Δt = 1/8** — fixed step size used during inference. 

The **conditional flow‑matching loss** drives the model to match a simple target velocity:

 **acoustic loss = expected value over (t, x₀, x₁) of ‖v_θ(x_t, t) - u_t(x_t given x₁, x₀)‖²** 

Where: 

- **t is sampled from a uniform distribution between 0 and 1** — uniformly sampled time. 
- **x₀ is sampled from the data distribution D** — a data‑distribution acoustic embedding (the true target). 
- **x₁ is sampled from a normal distribution with mean 0 and variance 1** — a sample of standard normal noise. 
- **v_θ(x_t, t)** — predicted velocity at the current noisy state **x_t (the state at time t)**. 
- **u_t(x_t given x₁, x₀)** — **target velocity**, defined as 

 **u_t(x_t given x₁, x₀) = x₁ - x₀** 

 This target is **constant over**t****, simply the vector from the noise sample to the data embedding.

**What the loss does** – By minimizing the squared **L2 norm (square root of the sum of squares)** distance between **v_θ (the learned velocity model)** and **u_t (the target conditional vector field)** at every sampled time, the transformer learns a **time‑invariant direction** that reliably pushes any noisy point toward the true acoustic embedding. At inference, the guided velocity **v_t (the final velocity vector)** steers the flow, and repeated Euler steps (8 steps per frame) produce the final continuous embedding, which is then quantized.

---

### Training Details, Ablations, and Practical Considerations 

**Dropout for unconditional modeling** – During FM training, the hidden state **h** is randomly dropped out **10 %** of the time. This forces the model to learn a robust unconditional velocity field that can later be guided with CFG.

**Efficient CFG** – At inference, CFG is applied **per acoustic frame** by performing a second forward pass of the FM transformer (the unconditional branch). Because the FM module is lightweight, this adds only a modest overhead compared with inserting CFG into the much larger decoder backbone.

**Quantization** – The continuous predictions **xₜ** are snapped to the **21 FSQ levels** before being fed back as discrete tokens for the next AR step. This preserves the discrete token interface while exploiting the smoothness of the flow during generation.

**Ablation results** – Compared against two alternatives:

- **MaskGIT** – attends over all 36 acoustic positions, yielding a per‑frame sequence length of 38. 
- **Depth Transformer** – requires 36 AR steps per frame.

Both alternatives lag behind FM in **human‑rated quality**, **compute cost**, and **latency**. FM needs only **3 inputs per frame** (`h`, `t`, **x_t**) and **8 function evaluations** (Euler steps), making it markedly more efficient.

**Pre‑training pipeline** – Uses paired audio–transcript samples pseudo‑labelled by Voxtral Mini Transcribe. Each training example is 

 **(A₁, T₂, A₂)** 

with a `<next>` token separating the voice‑reference audio **A₁** from the transcript **T₂**, and a `<repeat>` token separating **T₂** from the target audio **A₂**. Both **A₁** and **A₂** come from the **same speaker** (single‑speaker segments) and have durations up to **180 s**; optimal voice prompts are **3–25 s** long.

**Two‑part loss** – Pre‑training combines 

- **Semantic cross‑entropy** **L_semantic** on the semantic token head, and 
- **Flow‑matching acoustic loss** **L_acoustic** (as defined above). 

This directly aligns semantic token prediction with acoustic embedding generation.

**Practical tricks** 

- **Freeze text‑embedding layers** of the decoder backbone to avoid over‑fitting rare tokens. 
- **VAD‑based loss weighting** – frames without speech receive a reduced loss weight; completely silent stretches are ignored (weight = 0). 
- **LLM‑based transcript rewrites** – e.g., converting “5 − 4” to “five minus four” improves robustness to spoken versus written forms.

These engineering choices together make the **Flow‑Matching transformer** a powerful, efficient core for acoustic token generation within the Voxtral Codec.

### Standard DPO Objective for Semantic Codebooks 

The discrete semantic codebook is fine‑tuned with **Direct Preference Optimization (DPO)** so that the model learns to prefer transcriptions with lower word‑error rate (WER) and higher speaker similarity. The loss is taken directly from Rafailov et al. (2023) and applied without alteration.

 **Loss function L(θ) = - Expected value over time t and samples (x_w, x_l) of [ log σ ( β · Δ_θ(x_w, x_l, t) - Δ_θ_ref(x_w, x_l, t))]** 

**Where:** 
- **Loss function L(θ)** — loss for the current policy parameters **Model parameters θ** 
- **Expected value over time t (sampled from a uniform distribution 0 to 1) and the winning/losing pair (x_w, x_l)** — expectation over a uniformly sampled time step **t** (irrelevant for the discrete case) and over winner/loser pairs **The pair of winning sample x_w and losing sample x_l** 
- **log sigma (logarithm of the sigmoid function)** — log‑sigmoid of the model’s raw preference score, turning it into a probability 
- **β** — scalar that controls how strongly the model’s own preference **Delta_θ (the difference in log-probabilities under the current model)** influences the update 
- **Delta_θ(x_w, x_l, t) (the difference between the log-probability of the winner and the loser at time t)** — preference difference computed by the current model (higher = stronger preference for the winner) 
- **Delta_θ_ref(x_w, x_l, t) (the difference between the log-probability of the winner and the loser according to the reference model)** — the same difference computed by a fixed reference model **Reference model parameters θ_ref** 

In plain English, the loss penalizes the policy when it assigns a lower preference to the winner than to the loser. The sigmoid maps the raw score to a probability, the scaling factor **β** balances the model’s self‑generated signal, and the reference term stabilizes training by anchoring updates to a known baseline.

**Illustrative example.** Suppose two candidate transcriptions of the same utterance have WER = 5 % (winner) and WER = 12 % (loser). The DPO loss will increase the probability that the model generates the lower‑WER transcription, producing a positive gradient that pushes the policy toward the winner.

--- 

### Adapting DPO to Continuous Acoustic Flow‑Matching 

When the acoustic codebook is generated via **flow‑matching**, the model outputs a continuous velocity field rather than discrete token probabilities. Consequently, the preference term **_** must compare **velocity errors** instead of token logits.

 **Δ_θ(x_w, x_l, t) = ‖v_θ(x_wᵗ, t) - u_t(x_wᵗ given x_w)‖² - ‖v_θ(x_lᵗ, t) - u_t(x_lᵗ given x_l)‖²** 

**Where:** 
- **v_θ(input, t)** — model‑predicted velocity field at time step **t** 
- **u_t(input given x)** — ground‑truth flow conditioned on the full acoustic sequence **x** 
- **squared L2 norm ‖·‖²** — squared L2 norm, measuring the discrepancy between predicted and target velocities 
- **x_w at time step t**, **x_l at time step t** — latent representations of the winner and loser at the sampled step **t** 

Because decoding proceeds auto‑regressively, each token receives its **own** sampled time step **t_i** (shown in bold in the paper). The loss therefore averages over tokens to avoid bias toward longer sequences:

 **Δ_θ(x_w, x_l, t) = (1/N_w) · Σ(i=1 to N_w) of ‖v_θ(x_w,i, t_i, t_i) - u_w,i,t_i‖² - (1/N_l) · Σ(i=1 to N_l) of ‖v_θ(x_l,i, t_i, t_i) - u_l,i,t_i‖²** 

**Where:** 
- **N_w**, **N_l** — numbers of tokens in the winner and loser acoustic sequences 
- **t_i** — independently sampled time step for token **i** 
- **v_θ(x_w,i, t_i, t_i)** — predicted velocity for winner token **i** at its own step 
- **u_w,i,t_i** — target flow for winner token **i** at **t_i** 
- analogous terms for the loser side

Empirically the authors found that **length‑normalizing** **_** (dividing by the winner’s length) caused instability, so the implementation omits that step.

**Concrete example.** Imagine a winner acoustic segment with three tokens. The sampled steps might be **t₁ = 0.2**, **t₂ = 0.7**, **t₃ = 0.5**. For each token the squared velocity error **‖v_θ - u_t‖²** is computed, averaged across the three tokens, and then the analogous average for the loser segment is subtracted. A positive result indicates the model prefers the winner’s flow.

--- 

### Stability Enhancements for Joint Semantic‑Acoustic DPO Training 

To improve both discrete token selection and continuous acoustic flow simultaneously, the two modality‑specific DPO losses are **summed with equal weight**:

- **Semantic term** uses **β_semantic = 0.1** 
- **Acoustic term** uses **β_acoustic = 0.5** 

The larger **β** for the acoustic component reflects its higher sensitivity to the flow‑DPO signal; a stronger scaling is needed to produce noticeable gradients in the continuous space.

A crucial consistency requirement is that **the sampled time step**t**and the initial latent**x₀**are drawn once per training example and reused for both the current policy**θ**and the fixed reference**θ_ref****. This guarantees that the preference comparison is made under identical conditioning, eliminating a source of stochastic bias.

Training proceeds with a **very low learning rate of**8 × 10⁻⁸****. Such a tiny step size prevents the delicate balance between the two DPO terms and the underlying flow‑matching dynamics from destabilizing, which would otherwise lead to robotic‑sounding speech or divergence.

Together, these tweaks—uniform loss weighting, modality‑specific **β** values, shared sampling, and a tiny learning rate—ensure that the model learns from preference data without collapsing.

--- 

### Rejection‑Sampling Pipeline for Preference Data Collection 

The preference dataset that fuels DPO is built through a **rejection‑sampling pipeline**:

1. **Held‑out voice collection** – a set of single‑speaker recordings is kept separate from pretraining data. 
2. **Synthetic text prompts** – each voice sample is paired with a randomly generated transcript. 
3. **Persona‑conditioned continuation** – *Mistral Small Creative 1* receives the transcript and a randomly chosen persona, producing diverse continuation texts. 
4. **Acoustic generation** – the pretrained speech model consumes both the voice and the generated text, emitting multiple acoustic samples per (voice, text) pair. 
5. **Multi‑criterion ranking** – each sample is evaluated on:
 - Word error rate (WER) 
 - Speaker similarity 
 - Loudness consistency 
 - UTMOS‑v2 scores (Baba et al., 2024) 
 - Additional language‑model‑based judges 
6. **Winner–loser construction** – the sample with the best combined score becomes the **winner** **x_winner**; a lower‑scoring sample becomes the **loser** **x_loser**. The pair is fed into the joint DPO loss. 
7. **Training schedule** – the summed semantic‑acoustic DPO loss is optimized **together with the original high‑quality speech pretraining objective for a single epoch**. Longer training on synthetic preference data was observed to make the speech overly robotic, so the authors stop after one pass.

**Example.** A held‑out voice says “Hello, how are you?”. Two continuations are generated: 
- “I’m fine, thanks.” (WER = 3 %, high speaker similarity) → winner 
- “I am good.” (WER = 12 %) → loser 
The pair drives a DPO update that nudges the model toward producing the lower‑WER, higher‑similarity continuation in future generations.

### Superior Reconstruction at Comparable Bitrates

**Voxtral Codec** delivers noticeably better speech reconstruction than **Mimi** while both operate at roughly **2 kbps**. Across a suite of objective metrics—Mel distance, STFT distance, PESQ, ESTOI, ASR‑WER, and speaker similarity—the model consistently scores higher, indicating superior timbral fidelity, finer spectral texture, clearer and more natural‑sounding audio, higher intelligibility, easier transcription, and better preservation of the speaker’s identity. An internal subjective listening test corroborated these findings, rating Voxtral as “comparable or better” for speech‑focused content when both systems use 16 codebooks.

---

### Detailed Quantitative Comparison

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/table2.png) *Table 2: Comparison of Voxtral Codec and Mimi on the Expresso dataset.*

All systems were evaluated at a shared **12.5 fps** frame rate, ensuring a fair temporal resolution for downstream auto‑regressive decoders. The token‑per‑frame configurations differ:

- **Mimi**: 8, 16, or 32 codebooks, each with a vocabulary of **2048** tokens. 
- **Voxtral**: a compact scheme of **1 × 8192** primary token plus **36 × 21** auxiliary tokens, delivering comparable expressive power with far fewer tokens.

Bitrates are nearly identical—Mimi 16 cb at **≈ 2.2 kbps** versus Voxtral at **≈ 2.1 kbps**—so quality gains are not due to higher bandwidth.

**Metric improvements (Voxtral vs. Mimi 16 cb):**

- **Mel distance**: 0.545 ↓ vs. 0.618 ↓ (lower → better timbral fidelity) 
- **STFT distance**: 0.982 ↓ vs. 1.100 ↓ (lower → finer spectral detail) 
- **PESQ**: 3.05 ↑ vs. 2.67 ↑ (higher → clearer, more natural speech) 
- **ESTOI**: 0.882 ↑ vs. 0.865 ↑ (higher → higher intelligibility) 
- **ASR‑WER**: 10.66 % ↓ vs. 11.01 % ↓ (lower → easier transcription) 
- **Speaker similarity**: 0.843 ↑ vs. 0.829 ↑ (higher → better voice‑identity preservation)

These numeric gains translate directly into perceptual benefits: listeners perceive speech that sounds more like the original speaker, with crisper articulation and fewer transcription errors, while the speaker’s characteristic timbre remains intact. The quantitative results align with the earlier subjective assessment, confirming that Voxtral matches or exceeds Mimi’s quality for speech‑centric audio at equivalent bitrates.

### Automatic Evaluation Setup 

We evaluated **Voxtral TTS**, **ElevenLabs v3**, and **ElevenLabs Flash v2.5** on two benchmark suites: **SEED‑TTS** (single‑speaker) and **MiniMax‑TTS** (nine languages — Arabic, German, English, Spanish, French, Hindi, Italian, Dutch, Portuguese). For each system we computed three objective scores:

* **Word Error Rate (WER)** – lower % means the transcription matches the reference more closely. 
* **UTMOS‑v2** – a neural predictor of Mean Opinion Score; higher values indicate better perceived naturalness. 
* **Speaker Similarity** – we extracted 192‑dimensional ECAPA‑TDNN embeddings for the generated utterance ( **a**) and the reference voice ( **b**), then measured cosine similarity: 

 **cosine(a, b) = (a·b) / (‖a‖ ‖b‖)** 

Where: 
- **a·b**  — dot product of the two embeddings (captures shared direction) 
- **\|a\|**  — Euclidean norm of embedding **a** (its magnitude) 
- **\|b\|**  — Euclidean norm of embedding **b** 

In plain words, the formula returns a value between ‑1 and 1; values **closer to 1** mean the synthetic voice acoustically resembles the target speaker more closely.

All three metrics were applied uniformly across the three TTS systems, allowing a direct, apples‑to‑apples comparison.

--- 

### Voxtral’s Speaker Similarity Advantage 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/table3.png) *Table 3: WER, UTMOS, and Speaker Similarity scores for Voxtral TTS, ElevenLabs v3, and ElevenLabs Flash v2.5.*

Table 3 aggregates the three metrics for every language in MiniMax‑TTS and for SEED‑TTS. The **Speaker Similarity** column tells a clear story: **Voxtral consistently achieves the highest cosine scores**. 

* Example (Arabic, MiniMax): Voxtral = **0.746**, ElevenLabs v3 = 0.721, ElevenLabs Flash = 0.539. 
* This pattern holds across all nine languages, confirming that Voxtral more faithfully reproduces the target voice.

Intelligibility, measured by WER, shows a different trend: **ElevenLabs Flash often records the lowest error rates** (e.g., 1.23 % on SEED‑TTS versus 2.68 % for Voxtral). UTMOS scores are comparable between Voxtral and ElevenLabs v3, with Flash slightly trailing in several languages. 

Thus, while Voxtral may not always win on raw intelligibility, its **speaker similarity edge** demonstrates a stronger preservation of the speaker’s identity.

--- 

### Why Automatic Metrics Fall Short 

The automatic suite paints an incomplete picture. Although **ElevenLabs Flash** scores best on WER and holds its own on UTMOS, **human listeners still prefer ElevenLabs v3**, especially when evaluating **emotion steering**. This mismatch highlights two key limitations:

* **UTMOS‑v2** predicts MOS from acoustic cues alone; it cannot capture higher‑level perceptual factors such as emotional nuance, prosodic naturalness, or listener fatigue. A system that looks “clean” to the model may still feel “flat” to a human ear. 
* **Speaker similarity** based on ECAPA‑TDNN embeddings measures acoustic proximity, not the holistic perception of identity. Listeners judge timbre dynamics, intonation patterns, and expressive timing—elements that a cosine score may miss.

An analogy helps: a fingerprint matcher can reliably flag two prints as a match (high similarity) even if the printed image looks blurry to a person. Likewise, automatic metrics can signal “good” performance while overlooking subjective qualities that matter to listeners.

These gaps motivate a **dedicated human evaluation**, which we describe in the next section.

### Human Evaluation Protocol 

The goal of the human studies is to gauge **naturalness** and **expressivity**—aspects that automated scores often miss. To keep the comparison fair, every audio file—including the reference recordings—is converted to a uniform **24**  kHz WAV** format, eliminating any quality bias.

The evaluation used ****77**prompts**: ****11**neutral utterances and**66** emotion‑associated sentences (e.g., angry, joyful). Annotators compared two samples at a time without knowing which model produced which sample. For each pair they chose one of four options:

- *Slightly better* – a modest preference 
- *Much better* – a strong preference 
- *Both good* – indistinguishably high quality 
- *Both bad* – both unsatisfactory 

Three native‑speaker annotators per language/dialect performed the comparisons, providing reliable, language‑specific judgments. 

*Example*: If a listener perceives one sample as a bit more natural than the other, they mark **“slightly better.”** If the difference is striking, they select **“much better.”** 

--- 

### Explicit vs. Implicit Emotion Steering 

**Explicit steering** gives the system a clear external cue about the desired emotion. Concrete examples for each model are:

- **Gemini 2.5 Flash TTS** – receives a free‑form instruction such as “*Speak in an angry tone.*” 
- **ElevenLabs v3** – receives an emotion tag placed in brackets, e.g., “*[angry]*” 
- **Voxtral TTS** – does not accept tags, so we feed it a *different voice prompt* from the same speaker that already sounds angry, thereby providing the emotion cue indirectly. 

**Implicit steering** supplies no explicit label; the model must infer affect solely from the text. For instance, the sentence “*This is the best day of my life!*” conveys joy without any extra tag. In this setting **Voxtral TTS** uses a *neutral* voice prompt, letting the textual content drive the emotional rendering. 

--- 

### Flagship‑Voice Results by Steering Type 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/table4.png) *Table 4: Voxtral TTS win rates by steering type. In the explicit steering setting, Voxtral TTS is competitive with ElevenLabs v3, while having a higher win rate compared to both ElevenLabs models in the implicit steering setting.*

Table 4 reports pairwise win rates (ties excluded) for the flagship‑voice comparison under the two steering conditions.

- **Explicit steering** – Voxtral TTS’s win rate is on par with ElevenLabs v3, showing that its workaround of using emotion‑laden voice prompts can match a system that directly accepts emotion tags. 
- **Implicit steering** – Voxtral TTS consistently records a higher win rate than both ElevenLabs models, indicating a stronger ability to infer and render emotion from raw text alone. 
- Across all conditions, **Gemini 2.5 Flash TTS** attains the highest overall win rate, confirming its robustness. 

The advantage of Voxtral under implicit steering likely stems from its architecture’s capacity to map textual cues to expressive prosody without relying on external tags. 

--- 

### Zero‑Shot Voice Cloning Evaluation Setup 

To test **zero‑shot voice cloning**, high‑fidelity recordings from two well‑known speakers per language were collected as reference voices. Generation proceeds **without any speaker‑specific fine‑tuning**; the models synthesize speech directly from the prompt text.

Annotators judge each generated sample on two dimensions:

1. **Likeness** – how closely the synthetic voice matches the timbre, pitch, and speaking style of the reference prompt. 
2. **Naturalness & Expressivity** – overall speech quality and emotional conveyance. 

As before, three native‑speaker annotators per language evaluate the samples, and all audio is standardized to **24**  kHz WAV **. 

*Example of a likeness judgment*: an annotator notes that the synthetic voice captures the deep, resonant quality of the male Arabic speaker’s reference recording, indicating high speaker similarity. 

--- 

### Zero‑Shot Cloning Results and Generalizability 

When pitted against **ElevenLabs Flash v2.5** in the zero‑shot cloning task, **Voxtral TTS** achieves an overall win rate of ****68.4\%**. This surpasses the**58.3\%** win rate observed in the flagship‑voice comparison, demonstrating that Voxtral generalizes better to unseen speakers.

The superiority holds across both **high‑resource languages** (e.g., English) and **low‑resource languages** (Arabic, Hindi), underscoring the model’s robustness to data scarcity. 

A higher zero‑shot win rate signals that the system can faithfully reproduce a wide variety of vocal characteristics without speaker‑specific adaptation, fulfilling the paper’s claim of a **versatile, user‑driven, expressive TTS platform**.

### Voxtral TTS Win‑Rate Comparison Across Languages 

The win‑rate table shows how often listeners preferred Voxtral TTS over ElevenLabs Flash v2.5 in side‑by‑side listening tests. 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/table5.png) *Voxtral TTS win‑rate versus ElevenLabs Flash v2.5 across ten languages (micro‑average 68.4 %).* 

- **Overall performance:** Voxtral wins **68.4 %** of the time, i.e., roughly two‑thirds of all comparisons. 
- **Language‑specific highlights:** 
 - Spanish leads with **87.8 %** win‑rate. 
 - Hindi follows at **79.8 %**. 
 - Dutch is the closest contest at **49.4 %**, still edging out the baseline. 
- **Consistent advantage:** Voxtral never falls below the ElevenLabs baseline in any language, demonstrating robust cross‑lingual quality.

---

### DPO‑Induced Trade‑offs in Quality and Accuracy 

Table 5 quantifies the effect of Direct Preference Optimization (DPO) on two automatic metrics: word error rate (WER) and the utterance‑level MOS predictor (UTMOS). 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/table6.png) *Impact of DPO on WER and UTMOS per language.* 

**Key trends** 
- **WER improvements** dominate: most languages see a drop of 0.5 %–2 % absolute WER. 
 - German: from **4.08**  % to **2.68**  % (Δ  **‑1.40**  %). 
 - French: from **5.01**  % to **3.22**  % (Δ  **‑1.79**  %). 
 - Italian, Dutch, Portuguese also improve by **‑0.86**  % to **‑1.11**  %. 
- **UTMOS gains** are modest but consistent, typically +0.06–0.07 points. 
- **Exception – Hindi:** WER worsens from **3.39**  % to **4.99**  % (Δ  **+1.61**  %), even though UTMOS still rises by **+0.13**. 

**Qualitative impact** 
- Fewer hallucinations and skipped words. 
- More stable volume throughout utterances. 
- Speaker similarity stays essentially unchanged (±0.01), confirming that DPO mainly trades intelligibility (WER) against naturalness (UTMOS) without altering voice identity.

---

### Impact of Number of Functional Evaluations (NFEs) on Efficiency and Quality 

NFEs denote how many diffusion steps the model executes for each generated token during inference. 

- **Quality gains:** Raising NFEs from **2** to **8** yields noticeable drops in WER, lifts in UTMOS, and higher speaker similarity across the board. 
- **Diminishing returns:** Beyond **8** NFEs, speaker similarity plateaus and WER can even creep upward, indicating over‑refinement. 
- **Computational cost:** Each additional NFE adds a full forward pass through the diffusion network, so inference latency grows roughly linearly. Doubling NFEs roughly doubles runtime. 

**Practical recommendation:** Use **8** NFEs** as the default setting. It delivers a strong quality boost while keeping inference time manageable.

---

### CFG α Tuning: Balancing Prompt Adherence and Expressiveness 

The classifier‑free guidance weight (CFG α) scales the influence of the voice‑prompt during diffusion sampling. A larger α forces the generated audio to follow the prompt more strictly. 

- **Metric behavior:** As α increases, WER, speaker similarity, and UTMOS improve monotonically, except for UTMOS‑v2, which plateaus. 
- **Human perception:** Very high α values cause the model to cling to the voice‑prompt, suppressing emotional cues from the text prompt. Listeners note a loss of expressive nuance. 
- **Guidance for practitioners:** 
 - For clean, studio‑style recordings, a modest α ≈ ****1.2** yields the best fidelity. 
 - For noisy, in‑the‑wild recordings where matching the voice‑prompt is paramount, a higher α (≈ 2–3) is preferable, even if some expressiveness is sacrificed. 
- **Runtime impact:** Adjusting α does **not** change the number of NFEs, so inference latency remains unchanged. It is a cheap lever for tailoring voice‑prompt adherence.

--- 

**Bottom line:** DPO markedly improves intelligibility and naturalness for most languages, with Hindi as a notable outlier. Inference‑time knobs—NFEs and CFG α—let practitioners trade off speed, quality, and prompt fidelity to suit diverse deployment scenarios.

### CUDA‑Graph Acceleration of the Flow‑Matching Transformer 

The **latency bottleneck** of the generation stage stems from the need to evaluate the ODE solver **N** times per decoding step, and with classifier‑free guidance (CFG) this doubles to **2 × N** forward passes for every generated frame. Python‑level orchestration and per‑kernel launch overhead dominate the wall‑clock time, especially when serving many short requests.

**GPU‑native solution:** capture the entire ODE‑solver execution as a CUDA graph so that the sequence of kernel launches is replayed without host‑side intervention.

**Workflow**

1. **Warm‑up & bucket creation** – At server start‑up, an eager pass is run for each predefined batch‑size bucket (e.g., 1‑8, 9‑16, …). 
2. **Graph capture** – The full sequence of forward passes (the **2 × N** evaluations) is recorded, producing a reusable CUDA graph per bucket. 
3. **Runtime execution** – When a request arrives, its batch size is rounded up to the nearest bucket, inputs are padded with zeros, and the pre‑captured graph is launched. 
4. **Result slicing** – After execution, padded entries are discarded, leaving outputs for the true batch size. 
5. **Fallback path** – If a request exceeds the largest bucket, the system falls back to eager execution to guarantee correctness.

**Quantitative impact** 

| Metric | Before CUDA‑graph | After CUDA‑graph |
|--------|-------------------|------------------|
| Latency | **133 ms** | **70 ms** |
| Real‑time factor (RTF) | **0.258** | **0.103** |
| Speed‑up | – | **2.5× speedup** |
| Improvement | – | **47\%** |

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/table7.png) *Table 7: Effect of CUDA‑graph acceleration on the flow‑matching transformer.*

Other quality metrics—NFEs, WER, UTMOS, and speaker similarity—remain unchanged, confirming that the acceleration does **not** degrade output quality. By eliminating host‑side overhead, the system achieves sub‑second first‑audio latency, directly addressing the bottleneck identified earlier.

---

### Asynchronous Chunked‑Streaming Protocol 

To keep the generation stage and the codec‑decoding stage from blocking each other, they run in **independent scheduling loops**. The vLLM‑Omni transfer manager writes newly generated audio tokens into a per‑request shared‑memory buffer after each autoregressive step.

When the buffer reaches a **predefined length threshold** (e.g., 20 frames), the manager packages a chunk for the codec stage. Each chunk contains an **overlap** of the last few frames from the previous chunk (e.g., 5 frames) followed by the newly generated frames. This overlap ensures that the codec decoder’s causal sliding‑window attention sees a continuous context across chunk boundaries.

*Example*: with an overlap of 5 frames and a new chunk of 20 frames, the transmitted chunk holds 25 frames—frames −5 … −1 (overlap) plus frames 0 … 19 (new). The codec decoder can start synthesizing waveform immediately, while the generator continues producing tokens.

Because the transfer uses shared memory, there is **no data‑copy overhead**, and the small overlap adds negligible latency. This protocol is the key systems‑level mechanism that couples the two stages without blocking, enabling the first audible segment to be emitted well before the full token sequence is ready.

---

### Inference Throughput and End‑to‑End Serving Performance 

Combining **CUDA‑graph acceleration** with the **asynchronous chunked‑streaming protocol** yields a highly efficient serving pipeline. The following scaling experiment was run on a single NVIDIA H200 GPU, processing 500‑character inputs into 10‑second audio clips.

| Concurrency | Latency | RTF | Throughput (char/s/GPU) | Wait‑rate |
|-------------|---------|-----|--------------------------|-----------|
| 1 | **70 ms** | **0.103** | **119.14** | **0\%** |
| 16 | **331 ms** | **0.237** | **879.11** | **0\%** |
| 32 | **552 ms** | **0.302** | **1\,430.78** | **0\%** |

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/table8.png) *Table 8: Serving performance of Voxtral TTS on a single H200.*

**Why latency grows modestly** – As concurrency increases, requests queue briefly before entering the CUDA‑graph replay, leading to a latency rise from **70 ms** to **552 ms**. However, the **throughput scales roughly 12×**, demonstrating excellent parallel efficiency.

**Zero wait‑rate** – The asynchronous streaming ensures the codec decoder is continuously fed, so no client stalls while waiting for tokens. Even at 32 concurrent users, the per‑request RTF stays well below 1 (e.g., **0.302**), confirming real‑time operation.

**Production‑readiness** – A single H200 GPU can comfortably serve **over 30 simultaneous users**, delivering uninterrupted streaming with sub‑second time‑to‑first‑audio. This makes the system ready for large‑scale deployment.

### Meta Summary

Voxtral TTS combines a **hybrid architecture**—**auto‑regressive** generation of semantic tokens followed by **flow‑matching** synthesis of acoustic tokens—to produce expressive speech that can be cloned from as little as three seconds of reference audio. This **zero‑shot cloning** capability consistently outperforms existing API baselines in human preference tests, demonstrating that the model can capture speaker identity and prosody with minimal data.

The authors release the full model weights under a **CC BY‑NC** license, inviting the community to build on and extend the system. In addition, Voxtral TTS has been integrated into the **vLLM‑Omni** framework, enabling seamless deployment alongside large language models. 

![](https://storage.googleapis.com/pg_image_bucket/papers/20260328-033109-2603-25551-s-98732c/table8.png) *Table 8: Serving performance of Voxtral TTS on a single **H200**.*