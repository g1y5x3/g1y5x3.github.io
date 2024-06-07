---
layout: post
title: "A deep dive into Phi-3-vision model"
katex: true
---
Earlier last week, Microsoft released their newest model series [Phi-3](https://huggingface.co/collections/microsoft/phi-3-6626e15e9585a200d2d761e3).
The Phi series models are known for using much less parameters but achieving comparable performance. As shown in the following table, the Phi-3-mini 
model could achieve similar performance compare to models that used almost twice as much parameters such as Llama-3 8B and Gemma 7B.

|         | Phi-3-Mini-4K-In<br>3.8b | Mistral<br>7b | Gemma<br>7b | Llama-3-In<br>8b | Mixtral<br>8x7b | GPT-3.5<br>version 1106 |
| :-:     | :-:                      | :-:           | :-:         | :-:              | :-:             | :-:                     |
| Average | 71.2                     | 61.2          | 61.7        | 69.4             | 69.8            | 74.3                    |

<div class="caption">
Table 1. Benchmarks. Here only the averaged scores were listed here. More details can be found in their
<a href="https://arxiv.org/pdf/2404.14219">technical report</a>.
</div>

However, what's more interesting and exciting this time is that they also released a multimodal model 
[Phi-3-vision](https://huggingface.co/microsoft/Phi-3-vision-128k-instruct)
which is a 4.2B model that takes both text and image as input for chat applications.

- Text and Image Preprocessor
- Image Embedding and Token Merging
- Transformer Decoder

## Text and Image Preprocessor
In order to use [CLIPVisionModel](https://huggingface.co/docs/transformers/en/model_doc/clip#transformers.CLIPVisionModel) as the image tokenizer, 
the size for input images is preferred to be 3×336×336. To enable training with higher resolution images, the methods in 
[InternLM-XComposer2-4KHD](https://arxiv.org/pdf/2404.06512) were applied during preprocessing. The images were resized, padded, and then cropped 
into 336×336 sub-patches. The image itself is also resized to 336x336 and appended as a global patch.

The texts were tokenized by [Llama 2 tokenizer](https://huggingface.co/docs/transformers/en/model_doc/llama2#transformers.LlamaTokenizer). It also 
creates a lace holder for image tokens at the location of special token `<|image_x|>`. Once the patch size of image feature is known (12 for 
Phi-3-vision), the `num_img_tokens` can be calculated by the following equation:

$$ (\text{num of sub crops} + \text{num of global crop}) \times 144 + \text{rows of sub crops} \times 12 + 1 $$

Note that the image ids in `<|image_x|>` must start from 1 and it must be continuous int, e.g. [1,2,3], cannot be [1,4,5].

![Phi-3-image-processor](/assets/Phi-3-vision/phi-3-processor.svg){: style="width: 90%;"}
<div class="caption">
Figure 1. Diagram for both text and image preprocessor.
$^1$When padding the image height or width to be divisible by 336, the value [1,1,1] was used and them normalized. However, when padding the cropped 
patches to fit the maximum number of crops (16), the value [0,0,0] was employed. <b>It is unclear whether this distinction in padding values was a 
deliberate choice to differentiate between image padding and crops padding, or if it occurred incidentally.</b>
$^2$ num_img_tokens = $\left(\frac{H}{336} \times \frac{W}{336} + 1\right) + \left(\frac{H}{336} + 1\right) \times 12 + 1$, where 12 represents the
patch size of image features.
</div>

## Image Embedding and Token Merging
`Conv2d` with a stride size 14 was part of `CLIPVisionTransformer` to project the inputs from 3 channels to 1024 while reducing the patch dimension 
from 336 to 24 before they were flatten and fed to the transformer based encoder. Note that if there were padded patches from the previous step 
probably for speed purposes to avoid allocating and freeing memory constantly, they would be discarded before being converted into image tokens.

![Phi-3-image-embedding](/assets/Phi-3-vision/phi-3-embedding.svg){: style="width: 60%;"}
<div class="caption">Figure 2.
The text embedding part is self explanatory. During the image tokenization, newline tokens and a separator token were also being added according to 
<a href="https://arxiv.org/pdf/2404.06512">InternLM-XComposer2-4KHD</a>.
</div>

![Phi-3-reshape-permute-flatten-concat](/assets/Phi-3-vision/reshape_permute_flatten_concat.svg){: style="width: 45%;"}
<div class="caption">Figure 3.
A detailed illustration of the Reshape, Permute, Flatten, Concat procedure during the image tokenization. To save time from reading and understanding
<a href="https://huggingface.co/microsoft/Phi-3-vision-128k-instruct/blob/main/image_embedding_phi3_v.py#L198-L244">this</a>.</div>

## Transformer Decoder
### Scaling rotary embedding for longer context
The implementation for scaling the long context window was somewhat confusing, as it did not precisely follow the description provided in the
[LongRoPE paper](https://arxiv.org/pdf/2402.13753) described. Based on the
[code](https://huggingface.co/microsoft/Phi-3-vision-128k-instruct/blob/fea3f11f18ca5b52b836dbaf6d8d6b1710524c3a/modeling_phi3_v.py#L132), two
different types of scaling were applied: 
 - non-uniform positional interpolation that's conditioned on current context length - `long_factor`, `short_factor`
 - a constant factor that was very similar to that one proposed in [YARN](https://arxiv.org/pdf/2309.00071) - `scaling_factor`
 
This scaling can be expressed as the following equation,

$$
\text{constant scale} \times
\left[ 
  \cos(\frac{n}{\lambda_0(\theta)^0}),
  \sin(\frac{n}{\lambda_0(\theta)^0}),
  \cos(\frac{n}{\lambda_1(\theta)^1}),
  \dots,
  \sin(\frac{n}{\lambda_{d/2-1}(\theta)^{d/2-1}})
\right]
$$

where $n$ is the position token index, $\theta^{i}$ represents the rotation frequencies, $\lambda_i$ and $\text{constant scale}$ correspond to 
`long_factor`/`short_factor` and `scaling_factor`.

$$
\lambda =
\begin{cases}
\text{long\_factor}, & \text{seq\_len} > 4096 \\
\text{short\_factor}, & \text{otherwise}
\end{cases}
$$

According to [LongRoPE](https://arxiv.org/pdf/2402.13753), the non-uniform positional interpolations were derived using an evolutionary search 
algorithm guided by perplexity metrics, utilizing samples from the [PG19](https://arxiv.org/pdf/1911.05507) and 
[Proof-pile](https://arxiv.org/pdf/1911.05507). It is likely that both `long_factor` and `short_factor` from 
[config](https://huggingface.co/microsoft/Phi-2-vision-128k-instruct/blob/main/config.json#L38) were obtained using similar methodologies. __The 
question is what data was used for this search step? It was not stated in their report and in my opinion that could have huge implications to the 
results.__

$$
\text{constant scale} = \sqrt{1 + \frac{\log(\text{s})}{\log(4096)}},
\;where\;s=\frac{\text{extended context length}}{\text{original context length}}=\frac{131072}{4096}=32
$$

On the other hand, the calculation for $\text{constant scale}$ was much more straight forward. Assuming the formula was also obtained through 
experimentation. It looks similar compare to $0.1log(s) + 1$ which was proposed in [YARN](https://arxiv.org/pdf/2309.00071).

### Blocksparse attention module