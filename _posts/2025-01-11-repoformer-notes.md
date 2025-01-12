---
title: 'Notes on RepoFormer paper'
date: 2025-01-11
permalink: /posts/2025/01/repoformer-notes
tags:
  - aws
  - cloud
  - python
  - gode-gen
  - rag
  - tree-sitter
---

# Repoformer paper

# Basic Idea

The main idea of the paper is to consider in context learning for RAG in code
generation applications as optional rather than always done. The authors note
that this is not a new idea in RAG but is new to be tested in code generation.

Also the authors do implement a new approach to selective retrieval, they
investigate selef selective retrieval which triggers retrieval when a special
token in found, `<cc>`. This is something analogous to the toolformer approach
where a special tool token was used. However, in this work the authors
are not considering general tool use, rather retrieval or not within a RAG
system.

Besides improving code gen performance they also note the latency aspects of
current codegen systems. In particular sparse retrieval methods-I assume they
are referring to Okapi-BM-25 and similar-add a latency to the generation
process which is often unacceptable. Implied then is that dense retrieval
methods, e.g. Semantic search using language model embeddings would be even
slower, adding an unacceptable latency to a code gen system.

# Notes

The authors look at a couple different tasks inside a code repo, api completion,
line completion, and function completion.
They use StarCoder and CodeGenMono as baselines against their code completion
model, named repoformer.
The find strong cross language completion in Repoformer-they report training only
on python repositories.

Appendix C compares to 2 other selective retrieval contexts proposed in prior
papers by different authors. They find that the selective RAG performance of
uncertainty-based metrics is inconsistent across sequence lengths. Larger
completions have less performant.

The Figure 1 from the paper best describes the RAG workflow system in the paper.
If you are in a hurry, it is worthwhile to study the Figure to get the gist of
the system workflow. However, the paper is nice work so I'd recommend not being
in a hurry whilst reading.

# Commentary

## Questions

One thing that wasn't clear to me while reading was why the author's did this
2 pass approach with Tree-sitter, described in the Appendix A. The authors write:

```
We perform task-specific post-processing on the model’s raw
predictions. For line, API, and chunk completion, we truncate the prediction to
having the same number of lines as in Y.
For function completion, we first add a placeholder pass function body and use
tree-sitter to determine the position of the function in the file.
Then, we concatenate the X_l and Yˆ , parse the string again with tree-sitter,
and extract the function body as the final Yˆ if the string can be parsed.
Otherwise, we directly return the raw Yˆ without post-processing.
```

For context, here the Y refers to the removed code as part of the in-filling
task and Y^ is the RAG infilled code portion.

So for this description it makes sense that you'd want to constrain the output
in some way to be syntactally valid python code. To solve the issue they seem
to rely on tree-sitter as a verifier of syntactic correctness.

Tree sitter however will parse a syntactically incorrect bit of code although it
might not continue down the subtree past the bit of incorrectness. This process
is known as [error-recovery](https://apps.dtic.mil/sti/pdfs/ADA043470.pdf) in
parsing. IIUC The expected behavior if an error is to have a raw text node.

My confusion in the approach is a bit about shoehorning the code generation into an LLM
and then verifying syntactical correctness as a second step.

A different approach might be to constrain the code generation process to
syntactically correct code.

It might still have issues that are common with current SOTA code LLMs, e.g.
undefined symbol generation but that would be a different, potentially
orthogonal, issue because in general you'd want the code gen model to be able to
introduce new symbols that aren't part of the generation process.

For more on the limitations of tree-sitter as a parser generator check out this
[blog post](https://blog.jez.io/tree-sitter-limitations/). And
the [underlying research](https://tree-sitter.github.io/tree-sitter/#underlying-research)
 off which tree sitter is based and other more recent extensions
like [Diekmann's thesis](https://diekmann.co.uk/diekmann_phd.pdf).
Also how things like tree-sitter [inspire LLVM](https://www.tzvipm.dev/posts/tree-sitter-llvm-and-the-future-of-language-tooling)
based tooling efforts.

## Following ideas
The use of selective retrieval is an interesting aspect to the paper.
At the ICML 2024 conference there was the notion of
[Many-shotting](https://arxiv.org/abs/2404.11018) described in a poster at the
MATH-AI workshop that I presented in. The linked paper above is by google's AI
group (formerly deepmind), the poster at the Math AI workshop was by a group out
of Stanford-the idea is 'in the air' as the saying goes. I think the poster was
for [this paper](https://arxiv.org/html/2405.09798v1).

An interesting follow up work or evaluation would be to determine the ICL length
that is optimal for the retrieval system-if you allow the ICL length k to be 0,
then you would encompassvthe current system and also account for the latency.

The specifics of how to model and implement this idea are what would comprise
the research efforts.

# Code for paper

RepoFormer has a [github site](https://repoformer.github.io/) with links.
