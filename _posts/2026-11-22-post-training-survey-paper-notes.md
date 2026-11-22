---
title: 'Reading Notes: Midtraining Bridges Pretraining and Posttraining distributions'
date: 2026-11-15
permalink: /posts/2026/11/midtraining-bridges
tags:
  - algorithms
  - foundation models
  - scaling laws
  - paper-notes
---

# Summary

A very well written paper surveying the post training landscape
in late 2024 and early 2025.


The conference version is
<a href="https://aclanthology.org/2025.acl-long.140/" aria-label="Click to navigate to the html landing page for the ACL conference paper">here</a>


Largely a survey of LLMs with post training which the authors use as a generic term largely encompassing what is now called midtraining and post training. 
</a>
For a good definition of midtraining and post training see the paper by Emmy Liu which I posted notes on last week.

<details>
  <summary>
   Why is this paper is worth reading?
  </summary>

The paper is a good summary of the techniques for
training language models up to 2025. 
Although the nomenclature has since changed
slightly the framing of the field is largely the same.

The areas where there is less coverage is on agentic
workflows. 

The paper was published in ACL 2025 long papers
proceedings which means it was likely written in
2024 or very early in 2025.

</details>


The survey paper categorizes post training scaling
into three methodologies: (1) supervised fine-tuning (SFT)
(2) reinforcement learning (RL) and (3) test time compute TTC.
These are now largely three different areas of post
training in a broad sense now and we typically call supervise fine-tuning
part of mid training and RL is often use synonously with post-training
although there are various ways of doing
RL, and TTC is specifically
a inference time methos, which is something that is
done separately and is not really considered a
training stage anymore, because the model weights
are usually not updated in TTC.

# Motivation for post-training scaling

pre-training the part where LLM's process large 
amounts of text from the Internet is expensive
and consumes a large proportion, typically over
90% of the LLM training computation. Post-traing
according to this paper account for less than 1%
 of the training computation and despite counting
for only a small percentage of the computation 
in training, largely contribute to the performance
of the model on downstream tasks. Given the
positives of computation and the impact of the
techniques, it seems sensible to study this area
in more detail to determine if more computation
should be added, and catalog the various techniques that
are used to improve the performance in this stage.

within each of the three areas, SFT, RL, and TTC
the author's classify methods into common methods
and scalable methods so in total there are three 
techniques and then common versus scalable methods
so there's a total of six different leaf nodes of
the classification tree. Then the authors look at 
specific papers in those areas as well as more finely
 detailed distinctions within each areas.

# What is SFT?

SFT is defined in the paper as methods for
generating instructions and responses.

common methods of SFT are primarily human labels
where the human produces a high-quality data, but
it's costly an insufficient for scaling.

Scaling SFT involves constructing instructions 
in responses through various scalable training 
and data construction methods. These methods
usually involve LLMs of various sizes and
qualities in different ways.

## Instruction Generation

SFT involves instructions and the
responses a common technique that is scalable
is called instruction generation where the
response is collection of known high-quality
responses, and then LLM's are used to generate
the instructions. Techniques in the space are
mentioned in the paper, including WebInstruct,
backtranslate, and ditto.

A second approach in the instruction generation
category is evolution based methods these methods
use an evolutionary mimicry strategy to iteratively
refined given instructions to improve their alignment
with the given response.

TODO: add anchor tags

<img src='{{site.baseurl}}/images/IMG_2757.jpeg'
loading='lazy'
alt="figure 3, classification of SFT methods"
/>

## Response Generation

SFT uses a collection of instructions and responses
so given a collection of quality instructions, another approach is to generate.
The responses approaches in this category include 
sampling self self refinement and weak supervision.

A common issue in the response generation space is
redundancy in the generated responses, so most of the 
techniques aim to improve given an existing sample or 
sample samples of responses.
The degree to which improvement in the responses that
are generated as possible is largely limited by the
capability of the model.

# What is RL?



# What is TTC?


