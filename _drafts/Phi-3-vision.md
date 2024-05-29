---
layout: post
title: "A deep dive into Phi-3 vision model"
katex: true
---
Earlier last week, Microsoft released their newest model series [Phi-3](https://huggingface.co/collections/microsoft/phi-3-6626e15e9585a200d2d761e3). The Phi 
series models are known for using much less parameters but achieving comparable performance. As shown in the following table, the Phi-3-mini model could achieve
similar performance compare to models that used almost twice as much parameters such as Llama-3 8B and Gemma 7B. 

|         | Phi-3-Mini-4K-In<br>3.8b | Mistral<br>7b | Gemma<br>7b | Llama-3-In<br>8b | Mixtral<br>8x7b | GPT-3.5<br>version 1106 |
| :-:     | :-:                      | :-:           | :-:         | :-:              | :-:             | :-:                     |
| Average | 71.2                     | 61.2          | 61.7        | 69.4             | 69.8            | 74.3                    |

<span style="font-size: 15px; color: gray;">
*Table 1. Benchmarks. Here only the averaged scores were listed here. More details can be found in their [technical report](https://arxiv.org/pdf/2404.14219).*
</span>

However, what's more interesting and exciting this time is that they also released a multimodal model 
[Phi-3-vision](https://huggingface.co/microsoft/Phi-3-vision-128k-instruct)
which is a 4.2B model that takes both text and image as input for chat applications.

- text and image preprocessor
- image embedding and token merging
- transformer decoder

### Text and Image Preprocessor
![Phi-3-image-processor](/assets/Phi-3-vision/phi-3-processor.svg){: style="width: 100%;"}
<span style="font-size: 15px; color: gray;">
*Figure 1. Diagram for both text and image preprocessor. 
$^1$When padding the image height or width to be divisible by 336, the value [1,1,1] was used and them normalized. However, when padding the cropped patches to
fit the maximum number of crops (16), the value [0,0,0] was used instead. Not sure it was deliberate choice to distinct the image padding and the crops padding
or it just happened to be this way.
$^2$ num_img_tokens = (H//336×W//336+1)+1+(H//336+1)×12, where 144 and 12 are manually configured. This was not straight forward to understand from reading the
code but it should be representing: (sub_crops+global_crop)×144 + separate + newlines×12.
Some of the details are mentioned in [InternLM-XComposer2-4KHD](https://arxiv.org/pdf/2404.06512).*
</span>

The image ids in `<image_x>` must start from 1 and it must be continuous int, e.g. [1,2,3], cannot be [1,4,5].

### Image Embedding and Token Merging
![Phi-3-image-processor](/assets/Phi-3-vision/phi-3-embedding.svg){: style="width: 70%;"}