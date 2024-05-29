---
layout: post
title: "A deep dive into Phi-3 vision model"
katex: true
---
Earlier last week, Microsoft released their newest model series [Phi-3](https://huggingface.co/collections/microsoft/phi-3-6626e15e9585a200d2d761e3). The Phi 
series models are known for using much less parameters but achieving comparable performance. As shown in the following table, the Phi-3-mini model could achieve
similar performance compare to models that used almost twice as much parameters such as Llama 3 8B and Gemma 7B. 

|         | Phi-3-Mini-4K-In<br>3.8b | Mistral<br>7b | Gemma<br>7b | Llama-3-In<br>8b | Mixtral<br>8x7b | GPT-3.5<br>version 1106 |
| :-:     | :-:                      | :-:           | :-:         | :-:              | :-:             | :-:                     |
| Average | 71.2                     | 61.2          | 61.7        | 69.4             | 69.8            | 74.3                    |

<span style="font-size: 15px; color: gray;">
*Table 1. Benchmarks. Here only the averaged scores were listed. More details can be found in their [technical report](https://arxiv.org/pdf/2404.14219).*
</span>

However, what's more interesting and exciting this time is that they also released a multimodal model 
[Phi-3-vision](https://huggingface.co/microsoft/Phi-3-vision-128k-instruct)

- text and image preprocessor
- image embedding and token merging
- transformer decoder

### Text and Image Preprocessor
[InternLM-XComposer2-4KHD](https://arxiv.org/pdf/2404.06512)

![Phi-3-image-processor](/assets/Phi-3-vision/phi-3-processor.svg){: style="width: 100%;"}
<span style="font-size: 15px; color: gray;">
*Figure 1. Diagram for both text and image preprocessor. 
$^1$During the image preprocessing, there were two padding operations being applied. 
$^2$*
</span>


`image_ids` must start from 1, and must be continuous int, e.g. [1, 2, 3], cannot be [1, 4, 5]

Q1: Why uses inconsistent values between image padding and sequence padding?