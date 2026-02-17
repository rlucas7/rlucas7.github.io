---
title: 'My Participation in GitHub Secure Open Source Fund'
date: 2026-02-17
permalink: /posts/2026/02/blog-post-1
tags:
  - GitHub
  - open source
  - security
---

# Summary

In September of 2025, SciPy took part in the GitHub Secure Open Source Fund.
This is a program that provides financial support to maintainers of open source
projects that are critical to the security of the software supply chain.
The fund is designed to help maintainers of open source projects that are widely
used and have a significant impact on the security of the software ecosystem.

For those who don't know, I'm a [maintainer of SciPy](https://scipy.org/teams/#maintainers), a library for scientific
python applications. The library provides fundamental algorithms for this like
probability distributions, linear algebra, optimization routines, and much more.

# How and Why was SciPy selected?

This is not an unreasonable question if you program in a language other than Python.
In Python NumPy and SciPy are pretty core libraries for doing things in science.
Other libraries that are also core would include Pandas and PyMC, both were in
the cohort with me.

One of the other maintainers, Ralf had already participated in a security training
for SciPy through TideLift. Another maintainer, Andrew had received the email
from GitHub and forwarded to the maintainers, this is how I learned about and
after emails with the other maintainers we determined it would be good for me
to attend to learn more about security. Andrew wasn't able to attend
because he has a full time role and the sprint itself was a full time thing
over most of the month of September.

SciPy is used as a dependency in many systems across the technosphere.
Many packages in python use it as a dependency and some of the routines
are used for calculating metrics inside various software systems. If
there was something that broke because of SciPy, we learn about it
pretty quickly.

# What I learned during the program

Several things stand out and if I omit something it's more likely I've simply
forgotten rather than the material not being excellent. Here are the 3 key things
I took away as learnings from the course

1. The security reporting ecosystem, how vulnerabilities are reported, what info
is included in the report, and the mechanics.

2. Licensing, I'm not a lawyer but someone who works on OSS licensing for many
years gave us a good overview of the current ecosystem of OSS licenses and how
the advances with AI are challenging the existing licensing structure.

3. Static analysis and fuzzing. For Static analysis we had a great training
on using `codeql` something GitHub offers for doing static analysis that can
determine both memory leaks in non Garbage Collected (GC) languages like C,
 as well as other enumerable security exploits in GC-ed languages like Python.

The social aspect was also great. I like that I can put names, voices, and faces
to the projects that I've used in the past and will use in the future. I learned
a lot from the group of maintainers in the cohort too.

I learned much more and I'm favoring brevity in this post.

# How we strengthened security in SciPy

1. Enabled the require Multi-Factor Authentication (MFA) for maintainers

2. Established CodeQL scans for the entire scipy repo. Scanning in
python and C languages. The CodeQL system is setup by language so
if you're a multi-language repo like SciPy then this is what you
need to do.

3. Setup a security page in our docs and update our SECURITY.md
file to guide readers to this doc before reporting a vulnerability.

We had some of these items in place already but not all.
Before the program completed we activated all 5.

SciPy also has the additional 4 items in place now.

# A Core OSS project security deliverables

If you've got an OSS project you maintain here is the playbook of
the core 5 and the additional 4 with links to how to set them up
and any background reference material.

## Establish the core 5 security items

1. Activate [Secret scanning](https://docs.github.com/en/code-security/how-tos/secure-your-secrets/detect-secret-leaks/enabling-secret-scanning-for-your-repository)

Secret scanning helps prevent you and others from accidentally committing secrets to your repo. It's a hassle to cleanup so better to prevent via
scanning.

2. Activate [code scanning](https://docs.github.com/en/code-security/how-tos/scan-code-for-vulnerabilities/configure-code-scanning)

This is the CodeQL stuff I mentioned earlier, if you choose a different static analyzer than CodeQL, consider that you can have more than 1 scanner on at any time.

3. Activate [protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)

I like to draw an analogy here with graffiti. If your taco stand had a way to prevent graffiti with a few clicks of configuration
wouldn't you do that? That's essentially what protected branches do, they prevent people from intentionally or unintentionally
messing with the code in your repo-e.g. graffiti.

4. Activate [private vulnerability reporting](https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/configure-vulnerability-reporting/configuring-private-vulnerability-reporting-for-a-repository)

This is helpful because if or when a vulnerability is found, private reporting is the ethical thing to do. This allows maintainers
to determine the appropriate remediation as well as notify others they think might be impacted-before public disclosure.

5. Activate [dependabot](https://docs.github.com/en/code-security/tutorials/secure-your-dependencies/dependabot-quickstart-guide)

Dependabot is super handy, if you maintain a repo and you don't have dependabot turned on, I'd recommend you do. Not only does
it keep you notified of updates to dependencies in a risk prioritized manner, it will even open PRs to your repo to update
the dependencies-super handy.

## Additional 4 items

1. Setup [SECURITY.md](https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/configure-vulnerability-reporting/adding-a-security-policy-to-your-repository), place this file in root of your project's repo

This establishes the project security policies etc. there may be links out from this to other docs.

2. Establish an [Incident Response Plan](https://github.com/resources/articles/what-is-incident-response#creating-an-incident-response-plan) (IRP)

These may be public or privatem, they're useful to think about before the stress of an active vulnerability impacts your decision making.

3. Activate [MFA](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/configuring-two-factor-authentication) for all maintainers and major contributors

Generally a good idea to have MFA on any digital or only account you have.

4. Outline a [threat model](https://wellarchitected.github.com/library/application-security/recommendations/threat-model/)

It's a helpful structured way of thinking through changes and how they might be used or abused.

# What I learned doing the work afterwards

It's pretty easy to setup the basic suite of codeql scans.
These scans cover both network calls and some common vulnerabilities.
We determined after some months of having these scans run that
what we'd like is something more focused on memory allocations
for the C and C++ code that we have in the SciPy repo. One of
the really nice things about the program is I have a slack channel
with the GitHub team and within no time at all, I was able to get super helpful
guidance on the steps I'll need to do to setup the more focused
memory scans. These make more sense for a library like SciPy which
makes very few network calls.

In fact this was a learning in itself,
the security aspects of libraries which make or do network calls
(e.g. things like the cli tool curl, or the python requests library)
are different in focus from those which provide data processing
routines like SciPy with few if any network calls.

As a software platform GitHub needs to balance their default settings
to cover both kinds of libraries and they have support for these sort of
customizations even if they aren't defaults.

I also worked to update our docs so that we have an actual
security file now. Given that SciPy maintainers aren't security
researchers, we decided to continue to use TideLift and the docs
now give better guidance to people on what they should and should
not report. Hopefully this helps rather than harms the reporting
process for SciPy-false reports can be harmful because they eat
up bandwidth on things that aren't high priority.

As a result of turning on the scanning we identified a couple
vulnerabilities which were remediated. I won't say much more
about this except that I learned a bit in the remdiation process
too.

# What's next for SciPy?

I plan to work on setting up the memory scanning via codeql
as a set of custom query suites using the likely bugs queries
from the codeql repo. This is what I had corresponded with
the GitHub team about and received helpful guidance.

# What you can do to help

## If you work with a philanthropy, NGO, or Government

If the organization funds things for the public good,
consider discussing with your partners and board about
supporting a program like the one GitHub runs-it's
money well spent.

# You work in a for profit organization

Consider establishing an audit of your software
supply chain. SBOMs can help you get a good view of the dependencies
you rely on and which ones are underfunded. Funding these projects
can go a long way. The donations are often something projects can
be flexible on too so initiating a discussion is worthwhile, especially
given that you might be relying on the code in your critical path.

Note that the above paragraph applies equally to philanthropies, NGOs,
and Governments too.

# If you are a developer, a teacher of an OSS course, or a student

If you are onsidering contributing, or having your students contribute
to an open source project, first read the guidelines for contributing
for the project under consideration. Then introduce yourself either
on an email, slack/discord, or whichever method of communication
the project prefers.

If the maintainers do not have an AI policy, ask.
They might be in the process of figuring out what they want that policy
to be. Or they might have an AI policy but it hasn't been written up yet.
Better to ask first than to begin dumping a lot of code on a project
where the reading time of the (human) maintainers is (usually) unpaid.

GitHub has docs with general guidance for noobs to OSS too-please read them.

# Thanks

I wanted to thank the GitHub and Microsoft team members who participated
in the program with us, providing mentorship and (often hard learned) information.
I also wanted to thank the philanthropic organizations, and their donors,
without whom the program wouldn't have been possible.
These organizations are listed on the [program page](https://github.com/open-source/github-secure-open-source-fund).

Thank You all.

# Some resources I referred to in the post

[GitHub contributing to OSS docs](https://docs.github.com/en/get-started/exploring-projects-on-github/contributing-to-open-source)

[Experience of curl with AI pull requests](https://fosdem.org/2026/schedule/event/B7YKQ7-oss-in-spite-of-ai/)

[SBOMS](https://github.blog/enterprise-software/governance-and-compliance/introducing-self-service-sboms/)

[Responsible AI licensing](https://www.licenses.ai/faq-2)

[OSS licenses](https://opensource.org/licenses)
