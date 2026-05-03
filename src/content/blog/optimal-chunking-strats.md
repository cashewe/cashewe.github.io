---
title: 'Mathematically Optimal Chunking Strategy'
description: 'The boys crave accurate chunks, your man delivers them'
pubDate: 'May 04 2026'
heroImage: '../../../public/diagrams/mathematically_optimised_chunking__board_setup.jpg'
---

*This blog post is designed to introduce the core ideas behind the [`darn`](https://github.com/cashewe/darn) package. It is worth inspecting the source code for yourself to suplement any gaps in understanding.*

There are [many](https://www.pinecone.io/learn/chunking-strategies/) documented chunking strategies for [RAG](https://en.wikipedia.org/wiki/Retrieval-augmented_generation) systems readily available online. These can range from incredibly simple character (or token) limits, to rules-based splitting strategies, or even LLM backed *semantic chunking* methods. From my experience however, none of these methods provide the production-worthy 'one-size-fits-all' approach that they claim to:

- Limit-based strategies are not context aware enough to work in documents with anything more than short paragraphs of plain text involved, often leading to contextless half sentences or the bottom of tables being lost to orphaned chunks.

- 'Simple' rules-based strategies inevitably grow in complexity as more edge cases are found until the point that the code becomes completely unmanageable, with implict expectations of step-order being accidentally broken by the web of additional sub-rules.

- LLM-based 'semantic' strategies are costly, inconsistant between documents **and** can produce drastically different outcomes between runs (and for any production use case, trust me, *you will repeat runs*).

All of this leads to unhappy stakeholders and a nightmare for maintainers; which can see promising products drown in tech debt once the solution goes beyond the cheesy prototype stage to something relied upon by the business. Indeed for a solution to be truly business ready, I'd posit it should aim to achieve the following goals:

1. be **reproducible** in outcome - index rebuilds **will** happen and you dont want to introduce unexpected changes when they do. 
2. be **extensible** in clear ways - your strategy **will** experience issues due to unexpected text structures, and your code **will** quickly become unmaintainable if you dont plan for it to adapt to address these in a controlled manner
3. be **decoupled** - your developers **will** forget any implicit dependencies between pipeline stages regardless of documentation to the contrary
4. be **structurally-aware** (but do not attempt to be contextually aware) - respect the format of your documents and context awareness should reliably follow anyways

By considering these goals up-front, we can hopefully avoid a ball-of-mud solution that causes headaches for every new edge case raised.
After some thinking on possible shapes for such a solution (and admitedly also at least partially to give myself a chance to play with [rust](https://rust-lang.org/) and [graphs](https://en.wikipedia.org/wiki/Graph_theory)), I developed **darn**. 

# The logic behind `darn`

At its core, the idea behind the tool is to invert the traditional chunking logic such that rather than demanding
>split here please

we insted pose the question
>where is best to split, given my weighted preferences?

Asking rather than acting here is a reflection of our actual confidence in our knowlegde of the underlying corpus (be real, you arent intimately familiar with *all* 3,000 `.docx` files in that directory are you?) and allows the solution to be more flexible to the unique idiosyncasies in the documents our users provide us. By taking this approach, we are effectively accepting that we will be searching not for the 'correct' place to split, but rather the 'least bad', which means our architecture is closer to the ambiguous reality of the problem we face.

In order to define 'least bad' we need to put some scalar values onto how 'bad' a given choice will be. This can be done by applying 'rules' with associated 'punishments' for breaking them to structures in our text, for instance 

>if you split a chunk mid-sentence, the cost is 50.

>if you split a chunk mid-word, the cost is 100

>etc...

In `darn`, we apply these punishments to a representation of the structures specificallt found in [markdown text](https://www.markdownguide.org/basic-syntax/) - the most common format for LLM prepared documents - with the option to extend to additional custom text types (additional types so far implemented are `Sentence` and `Word`, which are both crucial for chunking but unserved by traditional markdown representations).

This will turn each character in our text into a possible candidate split-point and the sum of the punishments at each quantifies how 'bad' a decision it would be to split there. 

To illustrate the means of finding the coveted 'least bad' solution, imagine a world where the characters in your text are spaces on a monopoly board, and the cost to land on them are representative of the summed punishment we mentioned prior.

![setup](/diagrams/mathematically_optimised_chunking__board_setup.jpg)

You decide to cheat the game by using weighted dice, but given the dice only has 6 sides, what are the best possible tiles to land on to minimise the total charge? This is an example of the common "bounded [shortest path](https://en.wikipedia.org/wiki/Shortest_path_problem)" - we want to take the route round the board that incurs the lowest cost but are contrained by a 'bound' of 6 (the number of sides on our dice).

whilst it may seem intuitive to always pick the cheapest tile within range (this is a so-called ['greedy'](https://en.wikipedia.org/wiki/Greedy_algorithm) strategy), in practice taking what seems optimal in a given roll may well end up incuring a higher total cost in future!

![greedy](/diagrams/mathematically_optimised_chunking__greedy_vs_optimal.jpg)

in order to avoid this scenario, we must encode into our decision making process some understanding of the future impacts of our present actions. Mathematically, this can be done as simply as starting at the end and working backwards, calculating the minimum possible cost to exit the board from each tile.

![reverse calc](/diagrams/mathematically_optimised_chunking__optimal_cost.jpg)

From here, the problem effectively solves itself - we simply select the tile in range of our dice with the lowest 'minimum cost to exit' and follow the path we used to arrive at that cost back up the board. Performing this same process on our text, we select the set of chunk boundaries that collectively form the 'least bad' solution - no one chunk is optimised in a vaccum. 

For most industry scale corpora, it makes sense to use `darn` until given clear signal that you need to do otherwise because:

1. **reproducible** in outcome - by using a mathematically optimal strategy, we garuntee the same output each run
2. **extensible** in clear ways - new behaviour is as simple as adding a new rule or structure type
3. **decoupled** - by acting on total punishment rather than the raw text, we replace the concept of 'order' with that of 'relative cost', and swap to a model in which all rules act simultaneously
4. **structurally-aware** (but does not attempt to be contextually aware) - it is literally built on the structure of markdown text.

Typically, this approach will be better than:

- Limit based strategies, as it is aware of the natural structures in the document so will avoid splitting paragraphs, lists, etc... where possible.

- 'Simple' rules based strategies, as it is designed under the assumption your rules can and will change and expand over time.

- LLM-based 'semantic' strategies, as it is free, reproducible and consistant across documents

# When not to use `darn`

There are still some obvious scenarios in which `darn` wont currently provide the same level of value that are worth being aware of however:

- if you have a 'small' corpus that can be artisinally split into bespoke chunks, this will likely out perform the mathematical optima.
- if your data can't be meaningfully converted to markdown (i.e. if its entirely made up of computer code for instance). This could feasibly be rectified by defining an alternate to the markdown structure and maintaining the rest of the code - feel free to contribute this if you need it.

other than these however I do genuinely believe the mathematical approach described above to be most-likely to be most-commonly optimal and will be using it as default when setting up future RAG pipelines - and I dont expect to commonly have to deviate from this.