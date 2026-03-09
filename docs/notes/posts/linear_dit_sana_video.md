---
date: 2026-03-08
draft: true
categories:
  - Study Notes
---

# From Vanilla Attention to Linear DiT

A study for the background to better understand [SANA-Video](https://arxiv.org/abs/2509.24695).

<!-- more -->

## Notation

- $N$ = sequence length, $d$ = head dimension, $D$ = feature-map dimension.
- Bold uppercase for matrices ($\mathbf{Q}, \mathbf{K}, \mathbf{V}, \mathbf{S}$);
  bold lowercase for vectors ($\boldsymbol{q}_i, \boldsymbol{k}_j, \boldsymbol{v}_j, \boldsymbol{o}_i, \boldsymbol{z}$);
  plain for scalars ($a_{ij}, \kappa, d, N$).
- All per-token vectors are ***row vectors***:
  $\boldsymbol{q}_i, \boldsymbol{k}_j, \boldsymbol{v}_j \in \mathbb{R}^{1 \times d}$ are rows of
  $\mathbf{Q}, \mathbf{K}, \mathbf{V} \in \mathbb{R}^{N \times d}$;
  dot products are written $\boldsymbol{q}_i \boldsymbol{k}_j^\top$ (a scalar).
- Feature maps: $\phi : \mathbb{R}^{1 \times d} \to \mathbb{R}^{1 \times D}$ (row in, row out).

## 1. Vanilla (Softmax) [Attention](https://arxiv.org/abs/1706.03762)

Given queries $\mathbf{Q} \in \mathbb{R}^{N \times d}$, keys $\mathbf{K} \in \mathbb{R}^{N \times d}$, values $\mathbf{V} \in \mathbb{R}^{N \times d}$:

$$
\mathrm{Attn}(\mathbf{Q}, \mathbf{K}, \mathbf{V})
= \mathrm{softmax}\!\left(\frac{\mathbf{Q}\mathbf{K}^\top}{\sqrt{d}}\right) \mathbf{V}
\tag{1}
$$

- Computes a full $N \times N$ attention matrix $\Rightarrow$ $\mathcal{O}(N^2 d)$ time, $\mathcal{O}(N^2)$ memory.
- Softmax is applied row-wise, so each output token is a convex combination (non-negative weights summing to 1) of all value vectors.
- Works extremely well but becomes prohibitive when $N$ is large (e.g. video tokens $\sim 10^5$--$10^6$).

## 2. The Kernel View of Attention

!!! tip "Key idea"
    Rewrite each attention weight as a kernel evaluation, factor it, and rearrange the summation to avoid the $N \times N$ matrix entirely.

**Step 1: Per-token attention weight.**
Row $i$ of $\mathbf{Q}\mathbf{K}^\top/\sqrt{d}$ has entries $\boldsymbol{q}_i \boldsymbol{k}_j^\top / \sqrt{d}$ for $j=1,\dots,N$.
After row-wise softmax, the attention weight from query $i$ to key $j$ is

$$
a_{ij}
= \frac{\exp\!\bigl(\boldsymbol{q}_i \boldsymbol{k}_j^\top / \sqrt{d}\bigr)}
       {\sum_{l=1}^{N} \exp\!\bigl(\boldsymbol{q}_i \boldsymbol{k}_l^\top / \sqrt{d}\bigr)}.
$$

**Step 2: Define the kernel and rewrite.**
Let $\kappa(\boldsymbol{q}, \boldsymbol{k}) \triangleq \exp(\boldsymbol{q} \boldsymbol{k}^\top / \sqrt{d})$.
Substituting gives the compact kernel form:

$$
a_{ij}
= \frac{\exp(\boldsymbol{q}_i \boldsymbol{k}_j^\top / \sqrt{d})}
{\sum_{l=1}^{N} \exp(\boldsymbol{q}_i \boldsymbol{k}_l^\top / \sqrt{d})}
= \frac{\kappa(\boldsymbol{q}_i, \boldsymbol{k}_j)}
{\sum_{l=1}^{N} \kappa(\boldsymbol{q}_i, \boldsymbol{k}_l)}
\tag{2}
$$

where $\kappa(\boldsymbol{q}, \boldsymbol{k}) = \exp(\boldsymbol{q} \boldsymbol{k}^\top / \sqrt{d})$ is the **softmax kernel**. Here $\boldsymbol{q}_i, \boldsymbol{k}_j$ are $1 \times d$ row vectors (row $i$ of $\mathbf{Q}$, row $j$ of $\mathbf{K}$), so $\boldsymbol{q}_i \boldsymbol{k}_j^\top$ is a scalar dot product. This is a [positive-definite kernel](https://en.wikipedia.org/wiki/Positive-definite_kernel), so the attention matrix is really a (row-normalized) kernel matrix.

Note that Eq. (2) only defines the scalar attention *weights*. The full output for token $i$ applies these weights to the value vectors:

$$
\boldsymbol{o}_i = \sum_{j=1}^{N} a_{ij} \, \boldsymbol{v}_j
\tag{3}
$$

where $\boldsymbol{v}_j$ is row $j$ of $\mathbf{V}$ ($1 \times d$), so $\boldsymbol{o}_i$ is also $1 \times d$.

**Key insight (Katharopoulos et al., 2020):** If we replace $\kappa$ with *any* kernel that factors as

$$
\kappa(\boldsymbol{q}, \boldsymbol{k}) = \phi(\boldsymbol{q}) \, \phi(\boldsymbol{k})^\top,
\tag{4}
$$

where $\phi : \mathbb{R}^{1 \times d} \to \mathbb{R}^{1 \times D}$ is a feature map (row in, row out), then substituting into Eq. (2)--(3):

$$
\boldsymbol{o}_i
= \sum_{j} \frac{\phi(\boldsymbol{q}_i) \, \phi(\boldsymbol{k}_j)^\top}{\sum_{l} \phi(\boldsymbol{q}_i) \, \phi(\boldsymbol{k}_l)^\top} \, \boldsymbol{v}_j
= \frac{\sum_{j} \phi(\boldsymbol{q}_i) \, \phi(\boldsymbol{k}_j)^\top \, \boldsymbol{v}_j}{\sum_{l} \phi(\boldsymbol{q}_i) \, \phi(\boldsymbol{k}_l)^\top}
\tag{5}
$$

Since $\phi(\boldsymbol{q}_i)$ does not depend on the summation index, we can factor it out:

$$
\boldsymbol{o}_i
= \frac{\phi(\boldsymbol{q}_i) \sum_{j} \phi(\boldsymbol{k}_j)^\top \boldsymbol{v}_j}
{\phi(\boldsymbol{q}_i) \sum_{j} \phi(\boldsymbol{k}_j)^\top}
= \frac{\phi(\boldsymbol{q}_i) \, \mathbf{S}}
{\phi(\boldsymbol{q}_i) \, \boldsymbol{z}}
\tag{6}
$$

where $\mathbf{S} = \sum_{j} \phi(\boldsymbol{k}_j)^\top \boldsymbol{v}_j \in \mathbb{R}^{D \times d}$ and $\boldsymbol{z} = \sum_{j} \phi(\boldsymbol{k}_j)^\top \in \mathbb{R}^{D \times 1}$.

## 3. Linear Attention

### Core Idea

By pre-computing $\mathbf{S}$ and $\boldsymbol{z}$ (both independent of $i$), each query's output is just a matrix-vector product $\Rightarrow$ $\mathcal{O}(N D d)$ total, which is **linear in $N$** (assuming $D, d \ll N$).

$$
\boxed{
    \mathcal{O}(N^2 d) \;\xrightarrow{\text{linear attn}}\; \mathcal{O}(N D d)
}
\tag{7}
$$

### Choice of $\phi$

- **Katharopoulos et al. (ICML 2020):** $\phi(x) = \mathrm{elu}(x) + 1$ applied element-wise $\Rightarrow D = d$, so the cost is $\mathcal{O}(N d^2)$.
- **Performers (Choromanski et al., 2021):** Random Fourier features to approximate the softmax kernel; $D$ controls the approximation quality.
- **cosFormer (Qin et al., 2022):** cos-reweighted linear attention with RoPE-like decay.
- Various others: ReLU, $1 + \mathrm{elu}$, learnable maps, etc.

### Autoregressive / Causal Form

For causal (left-to-right) generation, the sums become *prefix sums*:

$$
\mathbf{S}_i = \sum_{j=1}^{i} \phi(\boldsymbol{k}_j)^\top \boldsymbol{v}_j, \qquad
\boldsymbol{z}_i = \sum_{j=1}^{i} \phi(\boldsymbol{k}_j)^\top
\tag{8}
$$

which can be computed incrementally as an RNN:

$$
\mathbf{S}_i = \mathbf{S}_{i-1} + \phi(\boldsymbol{k}_i)^\top \boldsymbol{v}_i, \qquad
\boldsymbol{z}_i = \boldsymbol{z}_{i-1} + \phi(\boldsymbol{k}_i)^\top
\tag{9}
$$

This is the "Transformers are RNNs" result: linear attention with causal masking is mathematically equivalent to a linear RNN with a matrix-valued hidden state $\mathbf{S}_i$.

## 4. Linear Attention in Language Models

- **Linear Transformers (Katharopoulos et al., 2020):** First to show the kernel trick $\Rightarrow$ linear complexity + RNN equivalence. Up to 4000$\times$ faster on very long autoregressive sequences.
- **Performers (Choromanski et al., 2021):** FAVOR+ mechanism using random features to get unbiased softmax approximation. Deployed in some Google systems.
- **RetNet (Sun et al., 2023):** Retention = linear attention + exponential decay + group normalization. Three computation paradigms: parallel, recurrent, chunk-wise.
- **Mamba / S4 line (Gu et al.):** State-space models that can also be viewed through the linear attention lens; diagonal state matrices for efficiency.
- **GLA (Yang et al., 2024):** Gated Linear Attention -- data-dependent gating on the recurrent state, bridging linear attention and gated RNNs.

**Recurring trade-off:** Linear attention is fast but generally slightly less expressive than full softmax attention. Recent work (gating, decay, hybrid architectures) narrows this gap.

## 5. DiT: Diffusion Transformer

- DiT (Peebles & Xie, 2023) replaced U-Net with a Transformer backbone for diffusion models.
- Uses standard softmax attention over spatial (and temporal) tokens of the noisy latent.
- For video generation, $N$ = (frames $\times$ height $\times$ width) in latent space $\Rightarrow$ $N$ easily $>10^5$, making $\mathcal{O}(N^2)$ attention the dominant bottleneck.

## 6. SANA-Video: Linear DiT

**[NOTE: Fill in details as you read the paper ([arXiv:2509.24695](https://arxiv.org/abs/2509.24695)).]**{style="color: red"}

### Key Ideas

- Replace *all* softmax attention in DiT with **linear attention** $\Rightarrow$ "Linear DiT."
  Reduces complexity from $\mathcal{O}(N^2)$ to $\mathcal{O}(N)$; yields $4\times$ acceleration on 720p video.
- **Feature map:** $\phi(x) = \mathrm{ReLU}(x)$ (element-wise), inherited from SANA's image model.
  This gives $D = d$, so the linear attention cost is $\mathcal{O}(N d^2)$.
- **Positional encoding:** 3D RoPE is applied *after* the ReLU kernel, i.e. $\mathrm{RoPE}(\mathrm{ReLU}(x))$.
  Applying RoPE before ReLU would destroy positional information via the clipping.
- **Stability trick for the denominator:**
  RoPE can make post-ReLU values negative, risking a zero denominator in:

    $$
    \boldsymbol{o}_i = \frac{\mathrm{RoPE}(\phi(\boldsymbol{q}_i)) \bigl(\sum_j \mathrm{RoPE}(\phi(\boldsymbol{k}_j))^\top \boldsymbol{v}_j\bigr)}
               {\mathrm{RoPE}(\phi(\boldsymbol{q}_i)) \bigl(\sum_j \mathrm{RoPE}(\phi(\boldsymbol{k}_j))^\top\bigr)}.
    $$

    Fix: remove RoPE from the query *in the denominator only*:

    $$
    \boldsymbol{o}_i = \frac{\mathrm{RoPE}(\phi(\boldsymbol{q}_i)) \bigl(\sum_j \mathrm{RoPE}(\phi(\boldsymbol{k}_j))^\top \boldsymbol{v}_j\bigr)}
               {\phi(\boldsymbol{q}_i) \bigl(\sum_j \phi(\boldsymbol{k}_j)^\top\bigr)}.
    $$

    This keeps the denominator strictly positive (all terms non-negative after ReLU) while the numerator still benefits from positional encoding.

### Block Linear Attention for Long Video (LongSANA)

- Block-wise autoregressive: generate video in chunks/blocks with chain-rule $p(x_{1:N}) = \prod_i p(x_i \mid x_{j<i})$, each block denoised by diffusion.
- **Constant-memory KV cache**: the causal linear attention output for token $i$ can be written as

    $$
    \boldsymbol{o}_i = \frac{\phi(\boldsymbol{q}_i) \bigl(\sum_{j=1}^{i-1} \mathbf{S}_j + \mathbf{S}_i\bigr)}
               {\phi(\boldsymbol{q}_i) \bigl(\sum_{j=1}^{i-1} \phi(\boldsymbol{k}_j)^\top + \phi(\boldsymbol{k}_i)^\top\bigr)},
    \quad \mathbf{S}_j = \phi(\boldsymbol{k}_j)^\top \boldsymbol{v}_j.
    $$

    Only two aggregates need caching: $\sum \mathbf{S}_j \in \mathbb{R}^{D \times d}$ and $\sum \phi(\boldsymbol{k}_j)^\top \in \mathbb{R}^{D \times 1}$ $\Rightarrow$ $\mathcal{O}(D^2)$ fixed memory (vs. $\mathcal{O}(ND)$ for vanilla KV cache).

- **Block boundaries -- causal Mix-FFN:**
  The temporal 1D convolution (kernel size 3) in the Mix-FFN also needs causal treatment:
  (1) an all-zero token is appended to the end of each block to prevent information leakage to the next block during training;
  (2) the last frame of each block is cached and prepended to the next block for the causal temporal convolution.
  Total extra cache per block: one frame $\in \mathbb{R}^{1 \times HW \times D}$.
- **Autoregressive training:**
  Two-stage continue-training on pre-trained 5s model:
  (1) *Monotonically increasing SNR sampler* -- later blocks get larger timesteps (noisier), reducing sampling space for faster convergence;
  (2) *Improved self-forcing* with global linear attention KV cache (not local window), which mitigates exposure bias by conditioning on self-generated content during training over $\sim$1 min videos.

### Architecture Details

- **Model:** SANA-Video-2B parameters. Architecture is, per the paper:

    > *almost identical* to SANA (image)

    Same Linear DiT backbone with a small decoder-only text encoder. Continued pre-training from the SANA-1.6B T2I model.
- **Text conditioning:** Uses a small decoder-only text encoder (inherited from SANA); conditioning is injected via cross-attention (the SANA design).
- **No separate temporal/spatial factorization:** Unlike many video DiTs that alternate spatial and temporal attention layers, SANA-Video uses *full 3D linear attention* over all spatio-temporal tokens jointly.
  Temporal modeling is instead enhanced by:
  (1) 3D RoPE integrated into the linear attention;
  (2) a 1D temporal convolution added via shortcut to the Mix-FFN block (zero-initialized at start so pre-trained image weights are preserved).
- **VAE:**
  For 480p: Wan-VAE (spatial $8\times$, temporal $4\times$, 16 channels).
  For 720p: custom DCAE-V (spatial $32\times$, temporal $4\times$, 32 channels) -- higher compression enables $53\times$ speedup over Wan2.1-14B at 720p.
- **Unified framework:** T2I, T2V, and I2V with a single model -- for I2V, the first-frame noise is set to zero (no architecture change needed).

### Training

- 12 days on 64 H100 GPUs ($\sim$1% of MovieGen, $\sim$10% of OpenSora cost).
- **Data filtering pipeline:**
  Raw videos $\to$ PySceneDetect + FFmpeg scene splitting $\to$ multi-stage filtering:
    1. *Motion quality*: Unimatch optical flow + VMAF pixel difference; keep only moderate, clear motion. Average optical flow magnitude injected into prompts for motion controllability.
    2. *Aesthetic quality*: DOVER video aesthetic score + OpenCV key-frame saturation; remove low-aesthetic and over-saturated clips.
    3. *Captioning*: VLM (InternVL-2.5) generates 80--100 word captions (subject, color, appearance, actions, expressions, environment, camera angles).
    4. *SFT data*: $\sim$5,000 human-preferred high-quality videos with diverse but balanced motion/style categories.
- **Progressive training schedule (3 stages):**
    1. *Stage 1 -- VAE adaptation on T2I*: adapt pre-trained image model to new video VAE; converges in 5--10k steps.
    2. *Stage 2 -- Coarse-to-fine video pre-training*: start at low-res short video (192p, 2.5s) $\to$ higher-res longer video (480p, 5s) $\to$ 720p with DCAE-V. Joint T2I + T2V + I2V training.
    3. *Stage 3 -- Autoregressive block training*: two-step continue-training for long video (monotonically increasing SNR sampler, then improved self-forcing). Final SFT with human-preferred data.

### Results

All benchmarks use VBench; latency measured on one H100 GPU at BF16 precision with batch size 1.

**T2V (480p, 480 $\times$ 832 $\times$ 81 frames).**

- Total 83.71 / Quality 84.35 / Semantic 81.35.
- Latency 60 s $\Rightarrow$ 8$\times$ speedup over Wan 2.1-14B (484 s).
- Best semantic score among all methods; comparable total to Open-Sora 2.0 (14B, 84.34).

**I2V (480p).**

- Total 88.02 / Quality 79.65 / I2V 96.40.
- Outperforms Wan 2.1-14B (86.86) and HunyuanVideo-I2V (86.82, 13B).

**720p (720 $\times$ 1280 $\times$ 81 frames).**

- Total 84.05 / Quality 84.63 / Semantic 81.73.
- Latency 36 s $\Rightarrow$ 53$\times$ faster than Wan 2.1-14B (1897 s).
  Uses DCAE-V (spatial 32$\times$, temporal 4$\times$) for higher compression.

**Long video (LongSANA, autoregressive).**

- Total 83.70 / Quality 84.43 / Semantic 80.78.
- Comparable to Self-Forcing (84.31); outperforms SkyReel-V2 (82.67) and CausVid (81.20).
- 4-step LongSANA generates 1-min 480p (16 fps) in 35 s on H100 (real-time 27 fps).

**Edge deployment.**
NVFP4 quantization (SVDQuant) on RTX 5090: 720p 5 s video in 29 s (2.4$\times$ speedup from 71 s BF16).

## References

1. Vaswani et al., "Attention Is All You Need." [arXiv:1706.03762](https://arxiv.org/abs/1706.03762)
2. Katharopoulos et al., "Transformers are RNNs: Fast Autoregressive Transformers with Linear Attention." [arXiv:2006.16236](https://arxiv.org/abs/2006.16236)
3. Choromanski et al., "Rethinking Attention with Performers." [arXiv:2009.14794](https://arxiv.org/abs/2009.14794)
4. Peebles & Xie, "Scalable Diffusion Models with Transformers." [arXiv:2212.09748](https://arxiv.org/abs/2212.09748)
5. Chen et al., "SANA-Video: Efficient Video Generation with Block Linear Diffusion Transformer." [arXiv:2509.24695](https://arxiv.org/abs/2509.24695)
6. Sun et al., "Retentive Network: A Successor to Transformer for Large Language Models." [arXiv:2307.08621](https://arxiv.org/abs/2307.08621)
7. Yang et al., "Gated Linear Attention Transformers with Hardware-Efficient Training." [arXiv:2312.06635](https://arxiv.org/abs/2312.06635)
8. Su et al., "RoFormer: Enhanced Transformer with Rotary Position Embedding." [arXiv:2104.09864](https://arxiv.org/abs/2104.09864)
