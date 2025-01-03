---
title: 'Large Language Models can Accurately Predict Searcher Preferencess'
date: 2025-01-02
permalink: /posts/2025/01/blog-post-1
tags:
  - IR
  - LLM
  - Annotations
  - paper-notes
---

# Search Preference LLM labeling paper

We focus on the [search preference paper](https://arxiv.org/abs/2309.10621)
incidentally the authors work on bing search engine at microsoft.

# Basic Idea

The main idea of the paper is to use an
[LLM as a judge](https://arxiv.org/abs/2306.05685), something initially
described by researchers working on LLMs and not IR.
The technique is nonetheless often found useful, indeed there are probably
some other ways this method can be leveraged that have yet to be discovered.

# Notes

Looking at LLMs as a judge is a large and varied topic, it may work well in
some domains and poorly in others. It really depends on the specifics of the
task requiring labels and the specifics of the system used to generate the
labels. This paper focuses on the search domain with queries and relevance
assessments.

In the paper, the authors investigate 5 prompt variations
called R, D, N, A, and M. The authors looked at the various arrangements of
these-each variation corresponds to some text that may be included in the
prompt. For performance metrics they look at Cohen kappa, MAE and AUC using
ia stratified sample of 3000 query-document pairs, cf. Table 1. To get
confidence intervals on these metrics the authors bootstrapped. The DNA prompt
variant, including D, N, and A substrings to the text, had the highest Cohen
kappa of 0.64.

In Section 4.5 they also investigate if the Query difficulty correlates with
human assessments using the Precision@10 metric. The idea is interesting and
one critique here is that in practical search scenarios you may not have ground
truth relevance assessments on which to determine relevances.
They retrieve to depth 100 on a TREC dataset and use RBO-rank biased overlap-to
compare the different lists. For the bing search engine the authors report:


"We have been using LLMs, in conjunction with expert human
labellers, for most of our offline metrics since late 2022."

and find good utility in doing so. The ground truth corpus they use comprises
queries, descriptions of need, metadata like location and date, and at least
two example results per query.
Results are tagged—again, by the real searcher—as being good, neutral, or bad.

The bing team monitors the health of the system by sampling LLM judged results
and going over those to monitor error and similar metrics. They also mention
that in both the TREC & Bing datasets, the task descriptions are very clear and
use a single LLM whereas [Liang et al.](https://arxiv.org/abs/2211.09110) saw
large differences from model to model over a range of tasks.

# Code for paper

None but a paper by folks at waterloo reimplements the idea.
