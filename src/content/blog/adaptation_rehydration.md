---
title: 'Adaptation.'
description: 'Rather than write a short sales pitch for adran, i blether for around 10 mins.'
pubDate: 'September 01 2026'
heroImage: '../../../public/diagrams/intro_to_rag.jpg'
---
*I was once fit for purpose, but that purpose changed*

## The Best Verb is also a Noun

I've often found it hard to articulate when or why we should transition between representations of data. Just where does the inertia come from? It was while watching the movie '[adaptation](https://www.imdb.com/title/tt0268126/)' that i found myself hit with a sudden clarity on why i've found the whole thing so hard to specify. It hadn't dawned on me that my choice of words could so heavily affect the way i think. That "transition" is a verb without causal justification; too open ended to truly be useful. Or that "adaptation" comes pre-baked with an implication that ones *constraints have changed*, and that this particular transition is in fact an act of optimisation:

> Adaptation / adapˈtāSHən /
>> - [uncountable, countable] the action or process of changing something, or of being changed, to suit a new purpose or situation
>
>> - [countable] a film, television drama or play that is based on a particular book or play but has been changed to suit the new medium

The movie is, by the way, completely nuts - somehow managing to be both a film about the *act* of adaptation and a successful adaptation of a book. The balance is achieved masterfully by taking the underlying themes of the original book and reworking them to fit the medium of film, rather than attempting to directly copy the book onto the screen.

## Adapting the Idea

The concept of adaptation is, in as many words, the beating heart of Data, AI and Machine Learning Engineering. Starting at the start, take for instance the task of EDA - popularly including creating notebooks full of vibrant graphs. Here, the need to adapt is clear - the data tables were a useful abstraction when creating the neccessary views, however they become a poor fit vs a simple plot once story telling becomes our goal.

There are plenty of other examples we could pull:

| Behaviour | Prior | Post |
|-----------|-------|------|
| Large Values Capped | Exact size is stored | biggest values share a size to limit data only to prior observed ranges |
| Free Text Categorised | Users have freedom to express themselves with open fields | distinct outcomes per 'type' can be controlled |


Sometimes however, the need to reconsider "fitness" of structures at multiple points in a system can still feel somewhat foreign. Key design decisions end up as consequences of prior optimisations rather than conscious choices, and constraints are allowed to be more short lived than the solutions they have shaped. 

A common example of this in AI engineering is in the use of 'chunks' of text.

## Realising the Abstraction

<details>
<summary>what is a chunk?</summary>
In AI systems, the typical unit of source text is the 'chunk' - a short section of continuous prose extracted from a full document. these 'chunks' are passed between systems in order to provide critical context to the AI model making the choices in the system.
</details>

chunks are selected for their fitness in "vector search". Small, thematically monotone pieces of information make encodings less noisy and therefore tend to improve the hit rate of questions. why then do they continue to turn up further into the system? what value does an arbitrary length section of information have from an AI's point of view, aside from convenience, that a longer more contextually rich alternative misses out on? To really push the issue here, take a look at this example:

```
Q: where do we do business
>>> A: we legally do business in: 
    - the united kingdom
    - France
    - Spain
```

seems fine right? but what if I showed you some of the surrounding text that the chunk - by chance - didn't include:

```
Due to local laws, for customers under the age of 16 or over the age of 65, we cannot do business in France.

In most other cases, we legally do business in:
    - the united kingdom
    - France
    - spain

For customers with a valid EU passport we can also legally do business in other European territories, such as Poland or Germany.
```

A more appropriate structure here would be more contextually rich, including as much information as necessary to convey the point. The chunk here isn't just not ideal, it actively harms the system. Design decisions are ultimately an encoding of constraints, and so carrying a decision forwards implicitly carries those constraints as well - even if we know longer have to abide by them

### A note to the thirsty

The problem of adding information back in to our system post vector search is generally known as 'context rehydration' and there are several promising options for engineers to pursue - each with their own strengths and weaknesses to consider. In my experience when presented with a smorgasbord of options is best to pick the simplest which meets your needs, as this will often better allow you to revisit the decision in future as new requirements emerge. In this case, I have settled on rehydrating based on markdown sections in the text, which adds an extra benefit in that markdown is a universal language with many upstream and downstream consumers already choosing to use it. To exemplify this, see the following dummy document:

```
# my document
this is my document

## part 1
this section is the first

## part 2
this section is the second and contains several subsections such as

### part 2a
this one

### part 2b
and this one
```

imagine that the highlighted portion is our chunk. my default behaviour is:

- include the whole section/s our chunk is contained in
- include its direct parent section
- include the titles of all sections in the document (keeping these helps protect us from situations where the underlying document uses subsections as a pseudo-list for instance.) 

for example, the rehydration of the highlighted chunk will look like this:

```
# my document
...

## part 1
...

## part 2
this section is the second and contains several subsections such as

### part 2a
this one

### part 2b
...

```
If this strategy appeals to you, I've collected my implementation into a python package [`adran`](github.com/cashewe/adran) which can be installed from pypi with your favourite package manager.

## survival of the fittest

not all adaptations happen within the system. sometimes a decision that made perfect sense at the time can in retrospect be seen to be outdated, and not just due to obsolete technology. For instance, a choice to use a PaaS system can greatly accelerate a team wanting to get new technologies such as ML or AI into production, but that team will often inevitably find with time that the simplicity they once benefitted from now holds them back from achieving true depth of skill. revisiting our decisions and recontextualising them after the fact must be done regularly both within the system and within the context of the evolving world our team and technology stack exist in if we are to form truly high performing teams.

After all that I'd be remiss if I didn't acknowledge the fact that I too must adapt. as i have adapted the movie which adapted the book. as i have adapted the chunk which adapted the text.

The industry around us is also evolving at what feels like break-neck speeds. Understanding which approaches, tools or philosophy's that we take for granted as best practice were formed based on constraints which no longer exist or at the very least no longer exist in the form they once did has become a critical part of staying relevant in an AI accelerated industry.

## conclusion

> Adaptation / adapˈtāSHən /
>> - [countable] a film, television drama or play that is based on a particular book or play but has been changed to suit the new medium
>
>> - [uncountable, countable] the action or process of changing something, or of being changed, to suit a new purpose or situation

This article was going to be about constantly challenging data structures throughout a system. Not challenging in the sense that a child is challenging to deal with when deprived of treats. Rather, challenging in the sense that one might challenge an authority figure when deprived of treats as a child. It was going to challenge readers to challenge structures more regularly and question decisions constantly, using the concept of chunks in rag pipelines to ground the abstract into reality. unfortunately i must confess, i couldn't quite figure out how to tie it all together. Readers would surely be left asking "why must i challenge these ideas? and when and how often again in the future?" by reframing the problem as one of observation of changing constraints, a need to adapt to these constraints becomes a fairly obvious outcome. and when should you adapt? well, probably at least as often as your lack of fitness for purpose causes genuine friction, and perhaps a bit more so than even that. None of which is to say that we should go and change everything at every available opportunity. Chesterton's fence may feel less relevant to a world in which a genuine answer as to why something exists might be 'the AI decided it should', but that shouldn't stop us from asking in the first place.

A helpful framework to identify opportunities for this might look like:

1. identify the decision
2. identify the constraints that shaped it
3. ask which of these still apply
4. ask what new constraints have since developed
5. adapt if necessary



cheers,

*johnno*