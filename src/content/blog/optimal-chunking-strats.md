---
title: 'Mathematically Optimal Chunking Strategy'
description: 'The boys crave accurate chunks, your man delivers them'
pubDate: 'May 04 2026'
heroImage: '../../../public/diagrams/mathematically_optimised_chunking.jpg'
---

*In this blog I will introduce the core ideas behind the [`darn`](https://github.com/cashewe/darn) package - designed to avoid incomplete answers in RAG by providing the best possible chunk boundaries using a simple cost optimisation technique.*

There are [many](https://www.pinecone.io/learn/chunking-strategies/) documented chunking strategies for [RAG](https://en.wikipedia.org/wiki/Retrieval-augmented_generation) systems readily available online. These can range from incredibly simple character (or token) limits, to rules-based splitting strategies, or even LLM backed *semantic chunking* methods. From my experience however, none of these methods provide the production-worthy 'one-size-fits-all' approach that they claim to:

- Limit-based strategies are not context aware enough to work in documents with anything more than short paragraphs of plain text involved, often leading to contextless half sentences or the bottom of tables being lost to orphaned chunks.

- 'Simple' rules-based strategies inevitably grow in complexity as more edge cases are found until the point that the code becomes completely unmanageable.

- LLM-based 'semantic' strategies are costly, inconsistent between documents **and** can produce drastically different outcomes between runs (and for any production use case, trust me, *you will repeat runs*).

All of this leads to inaccurate recall, incomplete answers and a debugging nightmare for maintainers; which can see promising products rapidly lose trust from their users once the solution goes beyond the cheesy prototype stage to something relied upon by the business. Indeed for a solution to be truly business ready, I'd posit it should aim to achieve the following goals:

1. be **reproducible** in outcome - index rebuilds **will** happen and you don't want to introduce unexpected changes when they do. 
2. be **extensible** in clear ways - your strategy **will** experience issues due to unexpected text structures, and your code **will** quickly become unmaintainable if you don't plan for it to adapt to these in a controlled manner
3. be **structurally-aware** (but do not attempt to be contextually aware) - respect the format of your documents and context awareness should reliably follow anyways

By considering these goals up-front, we can avoid a ball-of-mud solution that causes headaches for every new edge case raised.
After some thinking on possible shapes for such a solution (and admittedly also at least partially to give myself a chance to play with [rust](https://rust-lang.org/) and [graphs](https://en.wikipedia.org/wiki/Graph_theory)), I developed **`darn`** - a tool that aims to use a mathematical optimisation method to settle on its chunk boundaries. 

# The logic behind `darn`

At its core, the idea behind the tool is to invert the traditional chunking logic such that rather than demanding:

![split command](/diagrams/mathematically_optimised_chunking_split_here.jpg)

we instead pose the question:

![split question](/diagrams/mathematically_optimised_chunking_where_to_split.jpg)

and offload the answering to some deterministic maths, detailed below.

>**N.B.** Asking rather than acting here is a reflection of our actual confidence in our knowledge of the underlying collection of documents (be real, you arent intimately familiar with *all* 3,000 `.docx` files in that directory are you?) and allows the solution to be more flexible to the unique idiosyncrasies in the documents our users provide us. In doing so, it brings our architecture closer to the ambiguous nature of the problem we are attempting to solve

Importantly: by taking this approach, we are effectively accepting that we will be searching not for the 'correct' place to split, but rather the 'least bad'. In order to define 'least bad' we need to put some scalar values onto how 'bad' a given choice will be. This can be done by applying 'rules' with associated 'punishments' for breaking them to structures in our text, for instance:

>if you split a chunk mid-sentence, the cost is 50.

>if you split a chunk mid-word, the cost is 100

>etc...

In `darn`, we apply these punishments to a representation of the structures specifically found in [markdown text](https://www.markdownguide.org/basic-syntax/) - the most common format for LLM prepared documents - with the option to extend to additional custom text types. This means we have out-the-box support for 'structures' such as `Heading`, `List`, `ListElement`, etc... and additional types so far implemented include `Sentence` and `Word` (which are both crucial for chunking but unserved by traditional markdown representations). These punishments can stack based on the number of rules you would violate by splitting in a given location.

By taking the sum of the punishments at each candidate location for splitting (either characters or tokens), we can quantify how 'bad' a decision it would be to split there. This information will be the foundation of our optimisation algorithm, as we can treat it as a form of the common "bounded [shortest path](https://en.wikipedia.org/wiki/Shortest_path_problem)" problem in which we try to find the cheapest route from A to B in a given number of steps or less.

To illustrate the means of finding the 'least bad' solution, imagine a world where the characters in your text are spaces on a monopoly board, and the cost to land on them are representative of the summed punishment we mentioned prior. The bound of our shortest path here is represented by the sides on our dice.

![setup](/diagrams/mathematically_optimised_chunking__board_setup.jpg)

Given the dice only has 6 sides, what are the best possible spaces to land on to minimise the total charge? Whilst it may seem intuitive to always aim for cheapest space within range (this is a so-called ['greedy'](https://en.wikipedia.org/wiki/Greedy_algorithm) approach), in practice taking what seems optimal in a given roll may well end up incurring a higher total cost in future!

![greedy](/diagrams/mathematically_optimised_chunking__greedy_vs_optimal.jpg)

in order to avoid this scenario, we must encode into our decision making process some understanding of the future impacts of our present actions. Mathematically, this can be done as simply as starting at the end and working backwards, calculating the minimum possible cost to exit the board from each space.

![reverse calc](/diagrams/mathematically_optimised_chunking__optimal_cost.jpg)

From here, the problem effectively solves itself - we simply select the space in range of our dice with the lowest 'minimum cost to exit' and follow the path we used to arrive at that cost back up the board (in computer science, this is known as [dynamic programming](https://www.geeksforgeeks.org/dsa/dynamic-programming/)). Performing this same process on our text, we select the set of chunk boundaries (the 'path') that *collectively* form the 'least bad' solution - no one chunk is optimised in a vacuum. 

# `darn` in action

To solidify the value of this approach in a text chunking context, lets take a look at a simple [`lorum ipsum`](https://jaspervdj.be/lorem-markdownum/) example, using the following 'document':

> # Virum si memores territus
> ## Vetat excipit Mnemonidas
>Lorem markdownum nullo saliunt, subibis. It fortis crede caedis tenet ille in ovis fuerunt cycno crura faciunt: illic. Matri manat, harenam et dabat in Amoris tamen aderisque Deucalioneas foret arte stabula iugis tot pinus mora.
> - Sit oculis si opus mensas
> - Sua negat annis repulsa
> - Opus dea rite felle inquit duritia ambiguo
> - Humana quaque hunc alii censet procorum
> - Antra auro
> - Calydonis meas lusibus

we'll set maximum chunk size such that darn **has** to choose to split the text, here at an arbitrary character count of say 200. the code we'll run is as follows, with default settings commented out but included for clarity:

```
from darn_it import Chunker

chunker = Chunker()  # we'll use default rules here, rather than defining custom punishments

with open("file.md", "r") as f:
    text = f.read()

chunker.get_chunks(
    text=text,
    chunk_size=200,
    # granularity="characters"  # can be set to "tokens" if using the 'model' param
    # model="gpt-4o-mini"  # uses tiktoken to set the tokeniser per model
    # overlap=0  # can be set to allow overlap between chunks
)
```
`darn`s output looks like this:
```
[
    Chunk(
        start=0,
        end=177,
        content=# Virum si memores territus
            ## Vetat excipit Mnemonidas
            Lorem markdownum nullo saliunt, subibis. It fortis crede caedis tenet ille in ovis fuerunt cycno crura faciunt: illic.
    ), Chunk(
        start=177,
        end=287,
        content= Matri manat, harenam et dabat in Amoris tamen aderisque Deucalioneas foret arte stabula iugis tot pinus mora.
    ), Chunk(
        start=287,
        end=468,
        content=- Sit oculis si opus mensas
            - Sua negat annis repulsa
            - Opus dea rite felle inquit duritia ambiguo
            - Humana quaque hunc alii censet procorum
            - Antra auro
            - Calydonis meas lusibus
    )
]
```

We can see here that `darn` has obeyed sentence structures **and** maintained the full context of the list - having to create a smaller penultimate chunk to achieve this. This is likely due to default punishments discouraging splitting lists, and sentences, whilst the split in the paragraph was unavoidable due to character limits and was accepted to protect the integrity of the list.

Compare this to a naive limit-based strategy, which would have seen the first chunk boundary after the 'e' in 'et' and would see its final chunk begin part way into the fourth bullet point. Imagine a user asking your RAG system to summarise a contract clause, and getting back half a bullet point!

Given how simple this was to setup, its my opinion that for most industry scale corpora it makes sense to use `darn` until given a clear signal that you need to do otherwise. Referring back to our original criteria, we see that `darn` is:

1. **reproducible** in outcome - by using a mathematically optimal strategy, we guarantee the same output each run.
2. **extensible** in clear ways - new behaviour is as simple as adding a new rule or structure type.
3. **structurally-aware** (but does not attempt to be contextually aware) - it is literally built on the structure of markdown text.

And as a result, chunks calculated via `darn` typically will be better than:

- Limit based strategies, as it is aware of the natural structures in the document so will avoid splitting paragraphs, lists, etc... where possible.

- 'Simple' rules based strategies, as it is designed to accommodate the fact that your rules can and will change and expand over time.

- LLM-based 'semantic' strategies, as it is free, reproducible and consistent across documents

# When not to use `darn`

There are still some obvious scenarios in which `darn` wont currently provide the same level of value that are worth being aware of, notably:

- if you have a 'small' collection of documents that can be artisanally split into bespoke chunks, this will likely out perform the mathematical optima.
- if your data can't be meaningfully converted to markdown (i.e. if its entirely made up of computer code for instance).

From my experience, neither of these are true for the typical industry use case however - maybe you will instead reach for `darn` next time you set out to chunk text!

cheers,

*Johnno*