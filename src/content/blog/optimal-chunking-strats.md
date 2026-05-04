---
title: 'Mathematically Optimal Chunking Strategy'
description: 'The boys crave accurate chunks, your man delivers them'
pubDate: 'May 04 2026'
heroImage: '../../../public/diagrams/mathematically_optimised_chunking.jpg'
---

*In this blog I will introduce the core ideas behind the [`darn`](https://github.com/cashewe/darn) package - designed to avoid degraded trust in RAG systems caused by lost context in chunks*

There are [many](https://www.pinecone.io/learn/chunking-strategies/) documented chunking strategies for [RAG](https://en.wikipedia.org/wiki/Retrieval-augmented_generation) systems readily available online. These can range from incredibly simple character (or token) limits, to rules-based splitting strategies, or even LLM backed *semantic chunking* methods. From my experience however, none of these methods provide the production-worthy 'one-size-fits-all' approach that they claim to:

- Limit-based strategies are not context aware enough to work in documents with anything more than short paragraphs of plain text involved, often leading to contextless half sentences or the bottom of tables being lost to orphaned chunks.

- 'Simple' rules-based strategies inevitably grow in complexity as more edge cases are found until the point that the code becomes completely unmanageable.

- LLM-based 'semantic' strategies are costly, inconsistent between documents **and** can produce drastically different outcomes between runs (and for any production use case, trust me, *you will repeat runs*).

All of this leads to inaccurate recall, incomplete answers and a debugging nightmare for maintainers; which can see promising products rapidly lose trust from their users. Indeed for a solution to be truly business ready, I'd posit it should aim to achieve the following goals:

1. be **deterministic** - index rebuilds are unavoidable and you don't want to introduce unexpected changes when they do. 
2. be **extensible** - real documents are never as uniform as you'd expect and your code will quickly become unmaintainable if you don't plan to adapt to this truth.
3. be **structurally-aware** - respect the format of your documents and context awareness should reliably follow anyways, as most people write prose semantically.

By considering these goals up-front, we can avoid a ball-of-mud solution that causes fresh headaches for each new edge case raised.
After some thinking on possible shapes for such a solution (and admittedly also at least partially to give myself a chance to play with [rust](https://rust-lang.org/) and [graphs](https://en.wikipedia.org/wiki/Graph_theory)), I developed **`darn`** - a tool that aims to use the context inherent in a document's **structure**, and an **extensible** list of weighted user preferences, to **deterministically** settle on its chunk boundaries. 

# The logic behind `darn`

At its core, the idea behind the tool is to invert the traditional chunking logic such that rather than blindly demanding:

![split command](/diagrams/mathematically_optimised_chunking_split_here.jpg)

we instead pose the question:

![split question](/diagrams/mathematically_optimised_chunking_where_to_split.jpg)

Though this distinction may seem arbitrary, asking rather than acting is a better reflection of the confidence we have in our knowledge of the underlying collection of documents (be real, you arent intimately familiar with *all* 3,000 `.docx` files in that directory are you?). By accepting that we will be searching not for the 'correct' place to split, but rather the 'least bad', we can form an architecture that is 'closer' to the ambiguous reality of the problem. This will ensure our solution remains flexible to the unique idiosyncrasies in the documents.

In order to locate these 'least bad' splitting points, we need to put some scalar values onto how 'bad' a given choice will be, and then we can use a mathematical optimisation algorithm to pick the set of best choices for us. This can be done by applying 'rules' with associated 'punishments' for breaking them to 'structures' in our text, for instance:

>if you split a chunk mid-sentence, the cost is 50.

>if you split a chunk mid-word, the cost is 100

>etc...

In `darn`, we apply these punishments to the structures found in [markdown text](https://www.markdownguide.org/basic-syntax/) (i.e. `Heading`, `List`, `ListElement`, etc...), with the option to extend to additional custom text types. Additional types so far implemented include `Sentence` and `Word` (which are both crucial for chunking but unserved by traditional markdown representations). These punishments can stack based on the number of rules you would violate by splitting in a given location, i.e. a mid-sentence word would cost 150 to split according to the above rules.

By taking the sum of the punishments at each candidate splitting location, we can quantify how 'bad' a decision it would be to split there. This problem can be collapsed into a form of the common "bounded [shortest path](https://en.wikipedia.org/wiki/Shortest_path_problem)" problem in which we try to find the 'shortest' (for us, 'least-punished') route from A to Z in a given number of steps or less, which has a canonical mathematical solution, displayed below:

![DAG](/diagrams/mathematically_optimised_chunking__shortest_path.jpg)

To illustrate the means of finding the 'least bad' solution, imagine a world where the characters in your text are spaces on a monopoly board, the cost to land on them are representative of the summed punishment we mentioned prior, and we roll a dice to determine how far we can travel each turn.

![setup](/diagrams/mathematically_optimised_chunking__board_setup.jpg)

Given the dice only has 6 sides, what are the best possible spaces to land on to minimise the total cost? Whilst it may seem intuitive to always aim for the cheapest space within range (this is a so-called ['greedy'](https://en.wikipedia.org/wiki/Greedy_algorithm) approach), in practice taking what seems optimal in a given roll may well end up incurring a higher total cost in future!

![greedy](/diagrams/mathematically_optimised_chunking__greedy_vs_optimal.jpg)

in order to avoid this scenario, we must encode into our decision making process some understanding of the future impacts of our present actions. Mathematically, this can be done as simply as starting at the end and working backwards, calculating the minimum possible cost to exit the board from each space in reverse. 

![reverse calc](/diagrams/mathematically_optimised_chunking__optimal_cost.jpg)

By taking this reversed approach to decision making, we are able to learn what the future impacts of our current choices will be, as they are encoded as costs into our rolling window of 'reachable' spaces. From here, the problem solves itself - we simply select the space in range of our dice with the lowest 'minimum cost to exit' and follow the path we used to arrive at that cost back up the board (in computer science, this is known as [dynamic programming](https://www.geeksforgeeks.org/dsa/dynamic-programming/)). Performing this same process on our text, we select the set of chunk boundaries (the 'path') that *collectively* form the 'least bad' solution - no one chunk is optimised in a vacuum. 

# `darn` in action

To solidify the value of this approach in a text chunking context, lets take a look at a simple [`lorem ipsum`](https://jaspervdj.be/lorem-markdownum/) example, using the following 'document':

> # Virum si memores territus
> ## Vetat excipit Mnemonidas
>Lorem markdownum nullo saliunt, subibis. It fortis crede caedis tenet ille in ovis fuerunt cycno crura faciunt: illic. Matri manat, harenam et dabat in Amoris tamen aderisque Deucalioneas foret arte stabula iugis tot pinus mora.
> - Sit oculis si opus mensas
> - Sua negat annis repulsa
> - Opus dea rite felle inquit duritia ambiguo
> - Humana quaque hunc alii censet procorum
> - Antra auro
> - Calydonis meas lusibus

we'll set maximum chunk size here at an arbitrary character count of say 200 to keep the example behaviour obvious. the code we'll run is as follows (I have added comments to the unused settings incase they're of interest):

```
from darn_it import Chunker

chunker = Chunker()  # use default rules here, rather than defining custom punishments

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

We can see here that `darn` has obeyed sentence structures **and** maintained the full context of the list - choosing to create a smaller penultimate chunk to achieve this. This is due to the selected (default) punishments discouraging splitting lists and sentences. Though the default rules also aim to discourage splitting paragraphs, the character limit we have set made this impossible and so the optimal splits were selected to protect the integrity of the list.

Compare this to a naive limit-based strategy, which for the same character limit would have its final chunk begin part way into the fourth bullet point. Imagine the loss of trust if a user asking your RAG system to summarise a contract clause got back only half a bullet point! Equally compare this to semantic methods, which **likely** would have arrived at a similar splitting - as the texts structure already reflects its semantic cohesion - but may not have done so reliably.

Referring back to our original criteria, we can see even from this simple example that `darn` is:

1. **deterministic** - by using a mathematically optimal strategy, we guarantee the same output for each run, provided no parameters / input text are changed.
2. **extensible**  - new behaviour is as simple as adding a new rule or structure type.
3. **structurally-aware** - it is literally built on the structure of the markdown text.

And as a bonus, the tool is written in rust-backed python - so it runs super-fast and can be slotted into most production workloads easily.

# When not to use `darn`

There are still some obvious scenarios in which `darn` wont be the most appropriate tool that are worth being aware of, notably:

- if you have a 'small' collection of documents that can be artisanally split into bespoke chunks, this will likely out perform the mathematical optima (though this isn't a sustainable strategy for a production workload).
- if you dont have any kind of upper bound on how big your chunks can be, the algorithm becomes poorly defined and its solution will lose most of its decision-making value.
- if your data can't be meaningfully converted to markdown (i.e. if its entirely made up of computer code for instance) then `darn` will split based on structures that may not reflect the meaning of the underlying document.

The first two of these are philosophically different scenarios to the one in which `darn` is designed for so will likely never be something I look to support. The last of these however would actually be a fairly simple feature request - since the algorithm is decoupled from the 'structures' on which it acts, I could feasibly update `darn` to work with any alternate text structures too if there was meaningful demand.

# Wrap up

On the surface, I hope I've been able to demonstrate a viable 'default' tool for chunking here - and perhaps next time you need to split text, you will reach for `darn` in the first instance! Going a little deeper however, `darn`'s core innovation (a refusal to write strict rules for situations I don't fully understand) reflects a design pattern that is relatively new to me as an engineer, and one I'm keen to mine further in future works. Not all problems have a universal solution, and when they don't, let your architecture reflect ambiguity, not feigned certainty.

cheers,

*Johnno*