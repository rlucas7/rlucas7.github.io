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

A very well written paper studying the effects of timing and proportion/weight
of midtraining data in foundation models.

The arxiv version is
<a href="https://arxiv.org/abs/2510.14865" aria-label="Click to navigate to the html landing page on arxiv for the preprint">here</a>

The first author of the paper, Emmy Liu is at Carnegie Mellon right now and
maintains a site with other works too. Her site is
<a href="https://nightingal3.github.io/" aria-label="Click to navigate to the first author's site">here</a>

# Section 2
The paper has an excellent Section 2
that clearly defines key terms like pretraining, mid-training, finetuning,
curriculum learning, and continued pretraining, as used in the paper.
While other papers might use these terms it was great to have clear definitions
so that the reader is on the same page as the authors.

# Sections 3-6

Largely a collection of experiments, conducted way beyond
<a href="https://arxiv.org/abs/2203.15556"
aria-label="Click to navigate to the Hoffman et al, Chinchilla paper">
Chinchilla optimal
</a>
to ensure stability for introducing the midtraining data. All the experiments
were done with a single GPU which is pretty impressive.

<details>
  <summary>
    How much beyond Chinchilla Optimal?
  </summary>

The paper didn't seem to have a quantified writeup of how far past chinchilla
optimal they are but the calculations are straightforward from the paper.

The models were 70-410 million and a 1 billion parameter model on the second
draft. The authors used a 128 billion token dataset, on approximately 61,000
update steps. Using the \\( C=6ND \\) relationship where N is parameter count
and D is the dataset size in tokens. Then using the heuristic of \\(D=20N\\)
also from the chinchilla paper, after rearranging numbers, the 410 million
parameter model is \\( D=20 \times 4.1 \times 10^8 = 8.2 \\) billion tokens at chinchilla
optimal. This tells us that the 128 billion token dataset is over 15 times the
chinchilla optimal size, eg. a lot past chinchilla optimal.

</details>

The key findings of this section is that datasets that are domain specific
show the most benefit for midtraining. The paper focuses on code and math
datasets, plus some more generic datasets on reasoning, finding that the math
and code datasets when matched on downstream evaluations of math and code
respectively, show the best performance improvements. Intuitive
and nice to have such well controlled experiments to objectively confirm the
intuitions many of us have on this step.

The paper measures proximity with a measure that is a convex combination of
cosine similarity, Jaccard distance, and Jensen-Shannon similarity, giving only
slightly more weight to cosine than the remaining two.

$$
    prox(d_1,d_2) = 0.4 \times cos(d_1, d2) + 0.3 \times Jaccard(d_1, d_2) + 0.3 \times JensenShannon(d_1, d_2),
$$

where \\(d_i, i=1,2 \\) are two distinct datasets.

The authors mention in the conclusion, Section 9,  that extending to more
domains would be a nice follow up.

Another thing the paper checks is the impact of midtraining on catastrophic
forgetting by measuring cross entropy loss on a held out portion of the C4 data,
which they ysed for pretraining.

There is also measures of structured fine tuning (SFT) on downstream tasks.

The key finding of proximity of the data to the downstream task is that it
makes the performance on the downstream task better.


<img src='{{site.baseurl}}/images/cleaned_similarity_matrix_compact_rightbar.png'
loading='lazy'
alt="An example similarity matrix between pre/midtrain and postraining datasets"
/>


The studies looked at mixtures of midtraining data with the pretraining data,
finding that the mixing is better than strictly cutting over to the midtraining
data. It seems to make the loss landscape smoother and results in better
performance on the downstream task than a strict cutover. This is something
anyone who's done what the paper defines as continued pretraining, a strict
cutover to the domain specific dataset, has observed if they try the experiment,
you usually don't get as good performance with the strict cutover.

A nice follow up here would be to determine some sort of optimal weighting
schedule for the datasets based on practical settings. The authors find that
the timing, when they introduce the midtraining data has a larger impact than
the weighting. To my mind, you can vary both if we take the weight as a function
of the number of steps (timing) and the propotions and vary both. Because of the
heavy cost of computing and the very restricted compute environment they have,
only a single GPU it's unsurprising they vary each part separately and not
jointly.

They also study the impact the weights have across the layers of the models.
the general trend seems to be that the impact of the midtraining tends to
center on the later weights in the model than the earlier layers but the dynamic
changes if you look at SFT. With SFT the earlier layers seem to learn the changes
to the domain.

<img src='{{site.baseurl}}/images/cka_summary_c4_410m.png' loading='lazy' alt="four heat plots of increasing model layers showing changes based on midtraining, and structured fine tuning"/>

# Some answered and still unanswered questions

These questions are not a critique of the paper, the paper is a well written
and clear paper as is, and in any work you must stop somewhere. Where the
authors stopped is sensible and the findings leave open further questions to
be answered, as any good research work does. Here are some things that seem like
natural follow up studies-beyond those the authors indicated in the conclusion.

## How much the optimizer matters?

The experiments all used adamW, using pretty reasonable hyperparameter settings
for the experiments.

There is no answer to the question of if the optimizer matters. I'm curious if there
are impacts of adamW vs Muon. If you're reading this blog post and you've done
those experiments and are willing to share, please drop me an email. I'd love
to add them (with attribution) to this blog post.

## How much does the model architecture matter?

The authors mostly study the Pythia family of models. It would be interesting
to study some other families that, while similar, are often considered different.

My hypothesis is that while there may be differences, it's likely the same
trends are observed across model architectures.

## How to expand to multiple domains simultaneously during midtraining?

No experiments on this topic, the authors were limited to only a few domains
so if you're into building more domain specific datasets for midtraining this
is a space to contribute new learnings. If you know of several good midtraining
datasets in domains beyond math and code, let me know and I'll attribute the
answers here in an update.

## Models beyond 1 billion

The authors, as academics, were compute constrained so they did not study models
larger than 1 billion parameters. Researchers who have the access to
larger compute than a single GPU could contribute to checking these results on
larger scales. There seems to be some emergent behavior at the 1 billion model
as evidenced by the increased correlations but it's not clear if this is noise
or something indicative of an emergent phenomena.

<img src='{{site.baseurl}}/images/bridge_distance_analysis.png' loading='lazy' alt="four correlation plots of increasing model sizes measuring the advantage relative to the downstream task based on proximity"/>

# Summary

The key takeways are:

1. Introduce midtraining data earlier in the training
sequence of data than is typical to improve downstream performance on domain
specific models. Align the domain with the downstream task.

2. Leverage midtraining as a means to reduce catastrophic forgetting,
maintaining general reasoning and knowledge into the specializations.

3. Use a mixture of the pretraining data and the midtraining data rather
than a strict cutover.

All the figures I'm using here in the blog post are from the original paper,
I recommend you read it if you found this post interesting.

