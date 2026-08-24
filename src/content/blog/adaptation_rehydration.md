---
title: 'Adaptation.'
description: 'Rather than write a short sales pitch for adran, i blether for around 10 mins.'
pubDate: 'September 01 2026'
heroImage: '../../../public/diagrams/intro_to_rag.jpg'
---
*I was once fit for purpose, but that purpose changed*

## my vocabulary

I've often found it hard to articulate when or why we should transition between representations of data. Just where does the inertia come from? It was while watching the movie '[adaptation](https://www.imdb.com/title/tt0268126/)' that i found myself hit with a sudden clarity on why i've found the whole thing so hard to specify. It hadn't dawned on me that my choice of words could so heavily affect the way i think. That "transition" is a verb without causal justification; too open ended to truly be useful. Or that "adaptation" comes pre-baked with an implication that ones *constraints have changed*, and that this particular transition is in fact an act of optimisation:

> Adaptation / adapˈtāSHən /
> - the action or process of changing something, or of being changed, to suit a new purpose or situation
>
> - a film, television drama or play that is based on a particular book or play but has been changed to suit the new medium

The movie is, by the way, completely nuts - somehow managing to be both a film about the *act* of adaptation and a successful adaptation of a book. The balance is achieved masterfully by taking the underlying themes of the original book and reworking them to fit the medium of film, rather than attempting to directly copy the book onto the screen.

## the Idea

If you take only one thing from this ramble, let it be this:

>design decisions are ultimately an encoding of constraints, and so carrying a decision forwards implicitly carries those constraints as well - even if we no longer have to abide by them.

Without action, key design decisions will therefore remain as consequences of prior optimisations rather than conscious choices. In engineering, there are plenty of examples you may take for granted of conscious decision making leading to adaptations of all forms:

| Before | Adaptation | After |
|-----------|-------|------|
| A table with 100 rows | convert to graph | trends become obvious at a glance |
| Free text describes opinions | sentiment analysis | trends can be monitored over time |
| critical error | wrapped in a custom error object | handlable by standard exception handlers or loggers |

Imagine if we hadnt made these decisions! Your EDA notebook would be *even less* legible...

If your experiencing friction and struggling to see whether or not its time to adapt, you may find use in this simple framework:

for the problematic representation:

1. identify the inciting decision
2. identify the constraints that shaped it
3. ask which of these still apply
4. ask what new constraints have since developed
5. adapt *if necessary*

That *if necessary* needs extra emphasis here. theres nothing wrong with deciding after all that that your current representation remains optimal for your new scenario - and if it does I would certainly never recommend needlessly altering it. The goal is to ensure we are fit for our scenario with the minimal effort to get there.

Using this framework, we can identify a common key failure in RAG system design

## The framework

It is common to experience some friction at the last step of the RAG process, often known as the 'hydration' step. here we pass the most relevant information we've previously identified to our LLM and ask it to answer the initial question based on what was passed. sometimes this works a treat! in real scenarios though, its not uncommon to experience half-answers; in which the AI is able to correctly provide only part of the truth
For instance, take a look at this example:

```
Q: where do we do business
>>> A: we legally do business in: 
    - the united kingdom
    - France
    - Spain
```

seems fine right? but what if I showed you some of the full source text:

```
Due to local laws, for customers under the age of 16 or over the age of 65, we cannot do business in France.

In most other cases, we legally do business in:
    - the united kingdom
    - France
    - spain

For customers with a valid EU passport we can also legally do business in other European territories, such as Poland or Germany.
```

Suddenly we see just how wrong the original answer was. the mechanism driving this is the use of 'chunks' - a small continuous piece of text that can have somewhat arbitrary boundaries that leave us with an incomplete picture of the truth.

going back to our framework, we can see:

1. the chunk is selected to optimise the vector search, which benefits from short, hyperfocused prose.
2. vector search is built on embeddings - which become noisy when multiple topics are considered. short chunks are therefore more likely to yield better embeddings by virtue of simply being short.
3. at the point of hydration, length is only a consideration in that we have to pay more for longer inputs - there is no longer a strict need for such short chunks
4. the new constraint is the need for complete context in order to avoid the AI doing material harm to our customers or our business by making incomplete or incorrect statements
5. we should adapt to include expanded context.

based on this, we can see that constraints have meaningfully changed and that we should be adapting to suit the new system. The chunk here isn't just not ideal, it actively harms us!

### The Chunk

One way to adapt to the new needs would be to simply provide more of the surrounding text from the source document. As an example, I have found reasonable success in simple - medium complexity usecases in simply expanding to include the whole 'section' in the markdown text we are processing i.e.:

```
# Section
blah blah <here is my chunk> blah blah
```

If I'm willing to pay a little more, I find including the parent section can really help cement the context in certain more heirarchical documents, and that including the headings of all the sibling sections can help to catch scenarios where sub sections are used as a form of list. 

If this strategy appeals to you, I've collected my implementation into a python package [`adran`](github.com/cashewe/adran) which can be installed from pypi with your favourite package manager.

This strategy meets its limit when dealing with documents that cross-reference often - in these scenarios some kind of entity-relationship expansion is likely more performant.

## survival of the fittest

not all adaptations happen within the system. sometimes a decision that made perfect sense at the time can in retrospect be seen to be outdated, and not just due to obsolete technology. For instance, a choice to use a PaaS system can greatly accelerate a team wanting to get new technologies such as ML or AI into production,but that team will often inevitably find with time that the simplicity they once benefitted from now holds them back from achieving true depth of skill. This doesnt make the initial choice a mistake, it merely shows us that times change and sometimes its just time to move on. revisiting our decisions and recontextualising them after the fact is an important part of growth for systems, teams and individuals alike.

After all that I'd be remiss if I didn't acknowledge the fact that I too must adapt. as i have adapted the movie which adapted the book. as i have adapted the chunk which adapted the text. There was a time when updating old processes or building simple tools was enough for me, but as i have aged I've found myself yearning for more at just the time the industry itself seems to be moving away from the need (or perhaps more accurately, *the want*) for such technical specialties. If i am to find the place i want for myself in the future, i must move forwards but not past the old me, taking the creativity and foundational understanding in new more strategic directions. A dyslexic man now voluntarily writing articles is just one step in this process.

## conclusion

> Adaptation / adapˈtāSHən /
>> - [countable] a film, television drama or play that is based on a particular book or play but has been changed to suit the new medium
>
>> - [uncountable, countable] the action or process of changing something, or of being changed, to suit a new purpose or situation


This article was going to be about constantly challenging data structures throughout a system. It was going to challenge readers to challenge structures more regularly and question decisions constantly, using the concept of chunks in rag pipelines to ground the abstract into reality. unfortunately for the longest time i couldn't quite figure out how to tie it all together. Readers would surely be left asking "why must i challenge these ideas? and when and how often again in the future?" by reframing the problem as one of observation of changing constraints, a need to adapt to these constraints becomes a fairly obvious outcome. and when should you adapt? well, probably at least as often as your lack of fitness for purpose causes genuine friction, and perhaps a bit more so than even that. None of which is to say that we should go and change everything at every available opportunity. Chesterton's fence may feel less relevant to a world in which a genuine answer as to why something exists might just be 'because the AI decided it should', but that shouldn't stop us from asking in the first place.

and on that, i leave you to adapt these adapted ideas (or not) into your own works.

cheers,

*johnno*