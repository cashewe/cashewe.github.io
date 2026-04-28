---
title: 'Mathematically Optimal Chunking Strategies'
description: 'The boys crave accurate chunks, your man delivers them'
pubDate: 'April 30 2026'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

A core question at the heart of current RAG techniques is 'what method should i use to split the text'. the importance of such a question is easy to undersell.

equally, it is important to remember that a 'chunk' doesn't exist in a vacuum but remains as part of the definition of the whole text. choosing to split in one location creates not one or two new chunks defined by their boundary but also via markovian principles naturally impacts the possible boundary set for all subsequent chunks. making short term, `greedy` decisions can, as we observe in almost all systems, lead to suboptimal outcomes at the overall level.

# 'Simple' chunking methods

Simple chunking methods start with the basic 'split the text into chunks of exactly N characters / tokens", perhaps with some overlap if you're feeling naughty. Often, this can feel niave - especially when we see some of the later methods, however as with many things, the straight forwardsness is arguably its greatest selling point. by remaining 'simple', we avoid 'unexpected' as the outcome, and given RAG systems inevitably must include the chaos of an LLM at some point, minimising complexity elsewhere can be hugely valuable from the perspective of adapting to user feedback.
On the flip side, I have in practice observed that simple methods rarely stay that way. by avoiding having a structured approach, they can often end up as an increasingly confusing list of sometimes not-as-arbitrarily-as-youd-expect one of rules and edgecases that can become incredibly hard to fully grok.

# 'heirarchical' chunking methods

often seen as an extension on simple methods are heirarchical methods - in these we attempt to split at some high level option and then gradually increase our rules until we are able. the impacts of this are a now explicit heirarchy of rules which, as stated above, can become quite unweildy. consider for instance...

# 'semantic' chunking methods

semantic chunking methods aim to create chunks that fully encompass some theme or idea and are I've found, frequently touted as the new hotness. in practice, ive yet to see any actually work at scale in production. the vague nature of the description should give you all you need to know here really - on paper this would be great, in practice what does it even mean to encompass a theme or idea? and how can you reliably achieve this? in a typical document, theme may be consistent across the whole document or swapped fluidly several times in a sentence depending on how it is defined. at their absolute worst, semantic chunking strategies make use of an LLM to perform the chunking, removing the safety of determinism for the sake of vague opinion on topic. I would personally steer clear of the trap of semantic chunking.

# unusual chunking

though i have no direct experience with these methods, i have seen them described elsewhere:
- chunks that contain only facts, which then expand into full paragraphs / sections when sent to the generative endpoint
- 

# towards a better method

it might be possible to envision a better means of splitting text - where better is defined as:

- consistent and reproducible
- optimal across the length of the document, with no early greed leading to long term mishaps
- extensible in clear, controlled ways
- orderless, meaning know implicit dependencies can exist between rules

