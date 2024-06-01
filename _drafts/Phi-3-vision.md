---
layout: post
title: "A deep dive into Phi-3-vision model"
katex: true
---
Earlier last week, Microsoft released their newest model series [Phi-3](https://huggingface.co/collections/microsoft/phi-3-6626e15e9585a200d2d761e3). The Phi 
series models are known for using much less parameters but achieving comparable performance. As shown in the following table, the Phi-3-mini model could achieve
similar performance compare to models that used almost twice as much parameters such as Llama-3 8B and Gemma 7B.

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
In order to use [CLIPVisionModel](https://huggingface.co/docs/transformers/en/model_doc/clip#transformers.CLIPVisionModel) as the image tokenizer, the size for
input images is preferred to be 3×336×336. Higher resolution images would make the length of image tokens extremely long which increases the VRAM requirement 
for the GPUs and slows down the training significantly due to the downside of transformers. Additionally, it would be nice to use the original CLIPVisionModel 
weights as initialization for the Phi-3-vision model per-training.

To enable training with higher resolution images, the methods in [InternLM-XComposer2-4KHD](https://arxiv.org/pdf/2404.06512) during preprocessing where the
image is resized, padded, and then cropped into 336×336 sub-patches. The image itself is also resized to 336x336 and appended as a global patch.

The texts were tokenized by [Llama 2 tokenizer](https://huggingface.co/docs/transformers/en/model_doc/llama2#transformers.LlamaTokenizer). It also creates a 
place holder for image tokens at the location of special token `<|image_x|>`. Once the patch size of image feature is known 
(12 for Phi-3-vision), the `num_img_tokens` can be calculated by the following equation:

$$ (num\;of\;sub\;crops + num\;of\;global\;crop) \times 144 + rows\;of\;sub\;crops \times 12 + 1 $$

Note that the image ids in `<|image_x|>` must start from 1 and it must be continuous int, e.g. [1,2,3], cannot be [1,4,5].

![Phi-3-image-processor](/assets/Phi-3-vision/phi-3-processor.svg){: style="width: 100%;"}
<div class="caption">
Figure 1. Diagram for both text and image preprocessor.
$^1$When padding the image height or width to be divisible by 336, the value [1,1,1] was used and them normalized. However, when padding the cropped patches to
fit the maximum number of crops (16), the value [0,0,0] was employed. It is unclear whether this distinction in padding values was a deliberate choice to 
differentiate between image padding and crops padding, or if it occurred incidentally.
$^2$ num_img_tokens = $\left(\frac{H}{336} \times \frac{W}{336} + 1\right) + \left(\frac{H}{336} + 1\right) \times 12 + 1$, where 12 represents the patch size 
of image features.
</div>

## Image Embedding and Token Merging
![Phi-3-image-processor](/assets/Phi-3-vision/phi-3-embedding.svg){: style="width: 70%;"}
### Reshape, Permute, Flatten and Concat
![Phi-3-image-processor](/assets/Phi-3-vision/reshape_permute_flatten_concat.svg){: style="width: 55%;"}

## Transformer Decoder