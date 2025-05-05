---
title: 'Critiquing code via AI'
date: 2025-04-26
permalink: /posts/2025/04/code-critique/
tags:
  - coding
  - ai
  - llm
  - code review
  - structured outputs
---

How to generate code reviews with the suggested changes API automatically.

Tip: Using structured outputs enhance the workflow by making the AI content
conform to what we need in the API of each code review tool.

Suggested Edits format
======

There are many code review tools you can use for your coding project, every one
has tradeoffs.
By choosing one code review tool you forsake features in a different code review tool.
This is a consequence of choice, decisions have opportunity costs.

The purpose of this post is twofold, one is to provide awareness of a tradeoff
that may not be obvious if you do not use coding AI tools or haven't used them
yet. Also, even if you have used said tools, say github's CoPilot, you may not
be aware of this alternate approach to code reviews and where it can be used.

For example, you might choose a different model than the one used by CoPilot to
do code reviews, thereby adding a second layer of AI to your workflow.

Presumably, this helps with robustness and creativity.

***Ok so just what am I talking about?***

I'm talking about using AI to generate code reviews for you. More than this
though, you can use an AI to suggest code edits directly in the review.

You might ask, why would you do it this way rather than use copilot in your IDE?

My first answer would be "tradeoffs" and my second answer would be that you can
use both-provided you are using a code review tool that supports the
workflow.

First I'll start with describing the workflow and then do the workflow programmatically
and only then add in the AI. The idea is to show the progression from human to
AI.
Note:
[Github](https://github.blog/changelog/2018-10-16-suggested-changes/) has had
the suggested changes feature for almost 7 years now. Other tools also support
this feature, the ones I'm aware of are
[Gitlab](https://docs.gitlab.com/api/suggestions/) and newer versions of
[Gerrit](https://gerrit-review.googlesource.com/Documentation/user-suggest-edits.html).
Probably other tools support this interaction too  or can be hacked to make
this work if they don't already support.

Ok so you can manually select a portion of a pull request/change set and make
a suggestion of how you think the selected code should be (re)written.
The description can be done in the browser of any of these code review tools.

Now that we've cleared up how that works, let's look at how to do this
programmatically.

# Programmatic Suggested Code Edits

Basically, we need to use the REST APIs of these code review services.

At this point you may think this is a silly or trivial extension. You might say
well I would never-or maybe only rarely-use a REST api directly to post
code review with suggested edits, it's just an unnatural workflow for a code
review to be done by a human. I would tend to agree with the sentiment.

However, keep in mind we're doing this incrementally.
While this step might seem weird we're doing this to get to the point where an AI does
the code review for us.

So while this part may seem like a strange way to do things we'll build on it
in the next section.

Looking through the suggested edits pages for the code review tools, you'll
notice the api for posting a suggested edit. You need to follow a format that
the rest api expects. For github you can see the format in
[the docs](https://docs.github.com/en/rest/commits/comments?apiVersion=2022-11-28#create-a-commit-comment).
In the demo repo I make a simple class that handles the setup
[here](https://github.com/rlucas7/code-reviewer/blob/009ddf5726a770e5b9351ace0f4ccb7cadc27c6d/src/reviewer/ai_client.py#L42).

Basically, this setup expects the triple backticks plus the word suggestion
also a starting line and some text as the suggestion of changes.
The exact names and formats seems to differ somewhat between github and gitlab
but the basic idea is the same.

Here is a more explicit example of how to achieve what I'm describing above.
Taking the example given in the github docs for the REST API to comment on a
pull request

```bash
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/OWNER/REPO/pulls/PULL_NUMBER/comments \
  -d '{"body":"```suggestion\n Awesome sauce!\n```","commit_id":"6dcb09b5b57875f334f61aebed695e2e4193db5e","path":"file1.txt","start_line":1,"start_side":"RIGHT","line":2,"side":"RIGHT"}'

```

Here you'll notice I used suggestion after some triple backticks and the
string "Awesome sauce!" also being sure to close the triple backticks.
The effect of the suggestion part is to turn that into a suggested change in
github.

For other REST APIs (e.g. gitlab and gerrit) the API will be slightly different
but in practice the workflow appears to be supported according to the docs of
these other two services.


# AI Programmatic Suggested Code Edits

For testing that this will work you first need to create a token to use the
REST api with the necessary permissions. For my testing I used a personal token.
The reason was mostly for ease and simplicity. However, the downside of this
is that the token means that the comments appear under the user.
For the future I'll create a separate user that is solely the AI generated
comments. This will make it easier when looking over code comments to determine
who said what in the review of the PR.

If you look through the pr [here](https://github.com/rlucas7/suggerere/pull/1)
you'll see a couple suggestions that appear to be made by me on my own repo.
For example, [this comment](https://github.com/rlucas7/suggerere/pull/1#discussion_r1986436865)
was made by the AI and includes some suggested changes which you could merge by
clicking the 'commit suggestion' button.

<img src='{{site.baseurl}}/images/code-suggestion.png'/>


In fact these were made using the google gemini model following the approach
outlined above. The code which generated the comments is over in the [code reviewer](https://github.com/rlucas7/code-reviewer/tree/main/src/reviewer)
code. The logic resides in the
[ai_client.py](c/reviewer/ai_client.py#L51) and
[git_client.py](https://github.com/rlucas7/code-reviewer/blob/main/src/reviewer/git_client.py#L37)
modules as well as the
[reviewer.py](https://github.com/rlucas7/code-reviewer/blob/main/src/reviewer/reviewer.py#L11)
module. The prompts are in the
[prompts](https://github.com/rlucas7/code-reviewer/tree/main/src/reviewer/prompts)
subdirectory and-for now-contains only the basic prompt to prove out the concept.

Different code repositories might have different prompts to encourage reviews to
focus on different styles or preferences.

Additionally if you use this setup for reviewing blog posts (so basic text), then
you might want a different prompting strategy, maybe something that encourages
readability, clarity, and a certain structure of blog entries that you commonly
write.

This is hacked together as a proof of concept.

# How I plan to make it better

I initially did the suggestion generation without
[structured outputs](https://ai.google.dev/gemini-api/docs/structured-output?lang=python).
The generated suggestions were much more fragile without so I'd recommend to use
them for this or any other application where you pass the AI generated content
to an API. To generate structured outputs in gemini you [define the output model](https://github.com/rlucas7/code-reviewer/blob/009ddf5726a770e5b9351ace0f4ccb7cadc27c6d/src/reviewer/ai_client.py#L42) as a
class and [pass it as a config](https://github.com/rlucas7/code-reviewer/blob/009ddf5726a770e5b9351ace0f4ccb7cadc27c6d/src/reviewer/ai_client.py#L79)
so it's pretty lightweight to achieve. The exact way to implement this for each
AI differs slightly and some of the AI from 1-2 years ago may not have structured output support.

Other major AI setups like [OpenAI](https://platform.openai.com/docs/guides/structured-outputs?api-mode=responses)
support this workflow so I'd expect moving forward you'd be able to use on more models.


Second, I was using an API token on my own user. In practice, if you use this
setup in a repo where several people are working it will:

1. Seem like you are making comments that in fact an AI is making

2. Confuse you when you look back at your comments on the PR(s).


To make it easier for everyone, I recommend to create another user, something
like `ai-reviewer-XXXX` or similar, and then make the token on that user and
have the AI run through that user rather than your own.

# How You can Quantify Efficacy

An additional benefit of setting it the AI code review this way is that you
could setup an A/B style test to track whether the PRs which get comments by
the AI are merged quicker or slower, induce more/less bugs etc.
You could create a valid instrument for this type of analysis by storing the
randomizations of (0/1). This would then be used in an
[instrumental variables](https://en.wikipedia.org/wiki/Instrumental_variables_estimation)
analysis, these are also sometimes called IV models for shorthand.

## Some things that need to be thought through for the IV analysis.

1. Do you use the AI code review on every push? PRs can and often do have
multiple pushes before merging.

2. Do you give users the ability to generate on demand? This somewhat
complicates the analysis because you no longer have a purely random instrument.
Since I first wrote and published an earlier draft of this post I've seen
that github is now implementing this feature in PRs. However, they do not appear
to be using any randomizations but instead allow humans to request the AI review.

3. How to handle errors in the AI reviews? Do you retry or ignore? As long as
you use structured outputs in the requested models these errors of format are
unlikely but could happen.

4. What are the right metrics to measure productivity here? I've proposed time
to merge as a simple one but this is also likely complicated by the length of
the pull request and other repo specific factors. For example, some PRs might
be done in multiple programming languages or impact various items. Also,
different repositories are more (or less) complicated in terms of their existing
structure. These-and other-factors may complicate things in the analysis a bit.

# Open Questions

## Which AI models will generate useful content in this setting?

I tested this out using the gemini model from google. This model is
likely going to be deprecated in favor of newer versions. In most applications
of AIs the performance on a particular task can vary if you switch from AI X to
AI Y, e.g. switching from google's gemini model to the latest AWS nova model to
take one specific example. In either case I'd still recommend leveraging
structured outputs. I'm not sure which AIs will work well with the application
though so no guarantees. If I hack on this more-which I plan to after I finish
the project and paper revisions I have planned-I'll either update the post or
create a new post and link & backlink.

If you try out other AIs and have either successes or failures drop me a note to
let me know so I don't try the same ones.

## What other code review tools support this style of interaction w/inline edit suggestions?

While it seems like the api for gitlab and newer versions of gerrit should both
work I haven't tested those out yet to confirm they perform as documented-I've only consulted the docs.

When I circle back to working on this I'll clean up the code some and
also make it so that the same workflow code can be used in gitlab and in gerrit.
The latter will require me to setup and run a version of gerrit to test the
workflow. If you read this and test out either of these others, please let me
know.


## Prompts, which ways to tune them for this type of application?

I used a single prompt that was super simple. The goal wasn't to get the best
code reviews or to have the reviews adhere to a particular style or format of
coding, e.g. camel case vs snake case, etc. Some experimentation will likely
yield some big improvements for the specifics of the code base where this is
used. Also, choosing better or more exemplars for the few-shotting might likely
help further too.


## What other, Non-coding applications might this be used for?

You could use the same workflow for writing blog posts and having an AI review
them before you publish them. You'd still need to use something like this for
the plaintext but it should work fine. The benefit of the workflow is that if
the AI generates multiple suggestions you can accept/reject them as applicable.
For example, this workflow could be used in my own github blog, or if there are
other overlays into other blogging platforms that have a similar interface as
the suggested edits.


In any case if you want to hack more with this feel free to open a pull request
on the [code-reviewer](https://github.com/rlucas7/code-reviewer/tree/main)
repo or get in touch with me via email.

If there are other code review tools which support this workflow drop me a
note with the links to documentation and I'd be happy to add them to the post.


### Cite this post

```
@misc{LucasRoberts2025aicodecritique,
  author = {Lucas Roberts},
  title = {Code critique via AI},
  year = 2025,
  howpublished = {\url{https://rlucas7.github.io/posts/2025/04/code-critique}},
  note = {Written: 2025-04-26}
}
```

