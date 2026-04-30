---
title: 'Mathematically Optimal Chunking Strategy'
description: 'The boys crave accurate chunks, your man delivers them'
pubDate: 'May 04 2026'
heroImage: '../../assets/blog-placeholder-3.jpg'
---
*This blog post is designed to introduce the core ideas behind the [`darn`](https://github.com/cashewe/darn) package. It is worth inspecting the source code for yourself to suplement any gaps in understanding.*

There are many documented chunking strategies for [RAG](https://en.wikipedia.org/wiki/Retrieval-augmented_generation) systems easily available online. These can range from incredibly simple character (or token) based limits, through rules based strategies and up to LLM backed 'semantic chunking' methods. From my experience however, none of these provide the production-worthy 'one-size-fits-all' approach that they claim to.

- Limit-based strategies are not context aware enough to work in documents with anything more than paragraphs of plain text involved, leading to contextless half-lists or table cells being lost to orphaned chunks.

- 'Simple' rules-based strategies inevitably grow in complexity as more edge cases are found until the point that they become completely unmanageable, with implict expectations of step-order being accidentally broken by the web of additional sub-rules.

- LLM-based strategies produce drastically different outcomes between runs (and for any production use case, trust me, *you will repeat runs*). The concept of 'topic' is too vague and the definition of 'section of text' to broad to expect any consistency here.

All of these issues lead to unhappy stakeholders and a nightmare for maintainers once the solution goes beyond the cheesy prototype stage to something relied upon by the business. Indeed for a solution to be truly business ready, I'd posit it should aim to achieve the following goals:

1. be **reproducible** in outcome - index rebuilds **will** happen and you dont want to introduce unexpected changes when they do
2. be **extensible** in clear ways - your strategy **will** experience issues due to unexpected text structures, and your code **will** quickly become unmaintainable if you dont plan for it to adapt to address these in a controlled manner
3. be **decoupled** - your developers **will** forget any implicit dependencies between pipeline stages regardless of documentation to the contrary
4. be **structurally-aware** (but do not attempt to be contextually aware) - respect the format of your documents and context awareness should reliably follow anyways 

it might then be possible to envision a better means of splitting text which starts with these goals in mind and works backwards. A **reproducible** strategy will use strict rules rather than vibes based notions. An **extensible** strategy will define those rules as bounded objects acting on text, allowing more to easily be loaded in. A **decoupled** strategy will apply all rules simultaneously rather than in sequence. A **structurally-aware** strategy will map its rules directly to text structure rather than topic.

To resolve this (and admitedly also at least partially to give myself a chance to play with [rust](https://rust-lang.org/) and [graphs](https://en.wikipedia.org/wiki/Graph_theory)), I developed **darn**. 

# The logic behind `darn`

<EXPLAIN THE MATHS HERE>

equally, it is important to remember that a 'chunk' doesn't exist in a vacuum but remains as part of the definition of the whole text. choosing to split in one location creates not one or two new chunks defined by their boundary but also via markovian principles naturally impacts the possible boundary set for all subsequent chunks. making short term, `greedy` decisions can, as we observe in almost all systems, lead to suboptimal outcomes at the overall level.

most importantly, darn is:

1. **reproducible** in outcome - by using a mathematically optimal strategy, we garuntee the same output each run
2. **extensible** in clear ways - new behaviour is as simple as adding a new rule (or text structure identifier!)
3. **decoupled** - by acting on total punishment rather than the raw text, we replace the concept of 'order' with that of 'relative importance' which is at the very least *arguably* easier to grok
4. **structurally-aware** (but does not attempt to be contextually aware) - it is litterally built on the structure of markdown text

# When not to use `darn`

**never.**

or prehaps under the following situations:

- if you have a 'small' corpus that can be split into bespoke chunks
- if you have a 'big' corpus and a big workforce who like being very bored
- if your data cant be meaningfully converted to markdown (i.e. if its entirely made up of computer code for instance)

other than these however i do genuinely believe the mathematical approach described above to be most-likely to be most-commonly optimal - so for any industrial scale chunking, it makes the most-sense to set and forget until proven otherwise.

The only issue really is that writing the markdown rules themselves was so boring i stopped after writing an arbitrary handful (several of which are, frankly, bizzare and therefore unlikely to be desirable to the majority of prospective users). The easiest solution to this is to lazily outsource the task of writing rules to the users themselves - *Look out for a future update in which rules can be defined in python by users and then consumed by darn!*
