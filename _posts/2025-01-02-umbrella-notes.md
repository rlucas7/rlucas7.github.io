---
title: 'Umbrela paper notes'
date: 2025-01-02
permalink: /posts/2025/01/blog-post-2
tags:
  - IR
  - LLM
  - Annotations
  - paper-notes
---

# Umbrela paper

The paper [UMBRELA: UMbrela is the (Open-Source Reproduction of the) Bing
RELevance Assessor](https://arxiv.org/abs/2406.06519) looks at how LLMs can be
leveraged for relevance assessments.
Prior art in this area includes
[work at Microsoft](https://arxiv.org/pdf/2309.10621) which the authors cite.

# Basic Idea

The main idea of the paper is to use an
[LLM as a judge](https://arxiv.org/abs/2306.05685), something initially
described by researchers working on LLMs and not IR.
The technique is nonetheless often found useful, indeed there are probably
some other ways this method can be leveraged that have yet to be discovered.

The paper is an open source reimplementation of a paper by researchers at
microsoft working on bing. A paper I [put notes up on recently]().

# Notes

The authors used OpenAI and the microsoft version as points of comparison.
There does not appear to be comparisons across open/closed source LLMs for this
task as of yet. For metrics the authors use NDCG@10 which differs from the bing
team's paper. The authors evaluate the method in TREC DL tracks from 2019-2023.
The prompt is documented both in the paper and in the github repo-see beneath

The case study of 2 query-passage pairs highlights the challenges and
benefits of leveraging the approach. The cases demonstrate ambiguity that
could have been the result of either inaccurate assessments or
incomplete information regarding the user intent.

# Code for paper

The paper is accompanied by the
[associated github repo](https://github.com/castorini/umbrela) which is great
for reproducibility.
While I like that the system as implemented allows reproducible research, the
dependencies include JDK and all of PySerini.
While this certain is an implementation detail it would be nice to see some
sort of plugin architecture leveraged for extensions like this one.
