```
needs redrafting, pictures drawn etc...
theres too mcuh technical detail in a few places, and a few topics (RAGAS especially) feel tacked on. lets see if we cant get it shorter and snappier too, it need be only like 10 minute read tops imo
```

you may have heard of the concept of localised chatbots before - for instance as a means of automatically answering customer queries about your business. you may even have heard the term RAG (or `retrieval augmented generation`) before... but what does it actually mean, and how does it work? in this short article I'll explain the underlying technology in simple terms and hopefully demonstrate how, when leveraged properly, RAG can allow your AI systems to become more than simple chatbots.

# Context

somewhere far from here (or maybe very close, reader discerning) the worlds top engineers are spending *trillions* of dollars producing new AI models. The models learn to write coherent sentences by studying text written by people from all across the world and all throughout history - which unsurprisingly leads the AI to develop an impressive grasp of the language, as well as a deep understanding of popular topics. but how does your business fit into that? The answer, frankly, is it doesn't. the AI is generally knowledgeable, but has no means of knowing anything particular about smaller, local topics. unfortunately, since the AI isn't 'thinking' but simply providing the statistically most likely answer to your questions, its not capable of telling you what it does or doesn't 'know' and instead gives a confidentially wrong response. you can keep rubbing the lamp, but the genie will only ever pretend to answer your wishes.

A suggestion I often see is to try training AI models on your local information (a method called 'fine-tuning'). whilst *possible* this is typically not that *desirable* as:

- it is expensive
- it causes you to 'overfit' on local data, which can worsen the general graps on sentence structure that makes AI a compelling conversationalist
- it will be costly to maintain this as the technology around you improves continually

Using the RAG technique, we will instead intercept our incoming queries, search through our data for grounding information, and inject it into our question before it hits the AI to provide the relevant context to the model. 

# How does it work?

At its simplest, we can break the RAG technique down into three simple steps:

1. ahead of time, we split our text into smaller, more focused 'chunks' of information which we store in a database somewhere
2. when we recieve a question, we first 'retrieve' the chunks that seem most relevant
3. We 'rehydrate' our context with the retrieved chunks, and allow the AI to generate a response.

despite these steps being linear in process, I think it easier to understand the process if we start at the second step and then work our way out in both directions. this might seem unusual but please, trust the process baby.

## 2: Retrieving the chunks

In your daily life you'll be familiar with many ways to discover information. On streaming services, we have genre labels to reduce the search space. On e commerce websites we use free text search matching techniques to find relevant products. In libraries we have whatever the heck the 'Dewey Decimal System' actually is. When it comes to free text searching, current trends have largely settled on a technique known as 'vector search'. 

In vector search, we assign a list of numeric values to text based on the 'semantic meaning' of the text - we call this list an 'embedding' because it *embeds* the meaning of the underlying text into something that conspicuously resembles a set of coordinates: 

(x1, x2, x3, ...)

if we take an embedding of the input question (importantly with the *same* embedding technique and to the *same* dimensionality), we will end up with coordinates embedding meaning of both the question and the underlying data. from here, we can use some simple trigonometry to find the physically nearest text to the input question, or in other words to find the underlying data whose meaning is deemed most similar to the input query.

*aside* This can be further supplemented by so-called 'hybrid' search techniques - the most popular of which combines the results of a vector search with that of a simple text match between the input statement and the stored data. This is particularly useful where local meaning may not translate to global meaning. internal vernaculars are unlikely to be represented by publicly available embedding models, and training your own comes with the same associated negatives as training an internal AI model.

Importantly, an embedding represents the whole text all at once, not a sentence by sentence reading. This means its value is heightened if the text is concentrated on a single topic, and are of relatively even size. For this reason it is beneficial to split our input text up into smaller chunks that ideally represent single topics of information, which leads us back to step 1.

## 1: chunking the text

Now that we have established the reason *why* we chunk, lets look at quite *how* we might do this. A simple default technique might be to simply split the input text every 'N' tokens (a 'token' is a string of typically 3 letters - AI models 'see' in tokens not words so most techniques using AI will measure in tokens) with a rolling 'overlap' containing some fraction of tokens from the previous chunk. this rolling overlap means that if a given sentence is very relevant to the input query, I am likely to draw in context from both before and after it at retrieval time (as it will likely match both chunks it appears in), helping to ensure I get enough information to get by. For an expansion on this naive chunking strategy, see my previously published article 'Mathematically Optimised Chunking Strategy' - its just as easy to implement and likely to outperform in practice, provided you have decently structured input data - specifically it looks for 'markdown' structured text. 

<The assumption of well structured input data is a common theme across chunking strategies, but sadly in practice it often doesn't hold. If you can, develop pipelines that aim to convert your text data into markdown format as this is a common structure that most tools can parse. if you cant, consider writing guidance to your team on how to write documents that are both human and machine readable - this will make you much more future proof than any hand made alternative.>

Several more advanced chunking methods exist, but these are often *contextually* useful rather than universally valuable. some assume tight adherence to specific structures which many real documents won't do. some are slower and more costly to run so aren't worth it for simple use cases. The bottom line as with most things is that in order to settle on the best strategy, you need to take the time to understand the data you're using: garbage in - garbage out

## 3: Rehydrating the context

With our data carefully chunked, our vector search based retrieval is able to find the most semantically similar chunks. From here, we want to provide the AI model with the data it needs to answer our initial question. In the simplest possible system, we will simply pass it the top outputs from the retrieval stage, however in practice this will often not suffice.

in some cases, we may ask something like 'which countries do we sell product X in?' and match only a few countries, or match the start of a list and not the end. we may even match a list of countries we don't operate in, sans the quantifier that tells us this. whilst some of these failures are on the chunking strategy, it is also often popular to expand the chunks you retrieved to include further context such as nearby headings, full sections you previously split etc... - we need the chunks small for semantic similarity to work but once we have completed retrieval the only thing limiting their size becomes the token cost of passing them to the AI model itself.

In other cases, we may find that semantic similarity and relevance to our input are not entirely aligned. for example, here we can see chunks about motorbikes, peddle bikes and Gino D'Acampos grandmother are all equally similar to a question on riding in the cycle lane. It is common to use a 'reranker' model here as a judge - to take the matched chunks and order them by likely usefulness (sometimes dropping the bottom few chunks in its list).

Once we're happy with our rehydration strategy, we can pass the new context consisting of:

- the question
- the relevant data from our documents
- a prompt explaining we need an answer grounded in the data we provide

on to the AI model, which can now provide us with an answer to our initial question at a fraction of the cost and time compared to training our own. since our RAG system is divorced from any particular models, we can also upgrade to newer AI models as they release to stay atop the technical curve without the need for expensive maintenance.

# Measuring RAG

Its important to take a step back from the build here to acknowledge that all of this feels very *magic*. unlike traditional Machine Learning techniques, theres no obvious way here to measure how accurate the outcomes of our system actually are - meaning despite all our best efforts, it may still just be regurgitating slop. 

This is where the RAGAS metrics family come in. RAGAS is a set of experimental techniques aimed at measuring the quality of the outputs from a RAG system, for instance by measuring what portion of the ai generated output text is actually taken from the grounding data vs 'made up'. Its important to know that these metrics aren't perfect - there are no metrics for 'is this answer actually correct' for instance as thats near impossible to automatically test without vast amounts of labelled data, but I'd still encourage making use of *a* metric of quality - if for no other reason than to compare various techniques throughout the system.

# Beyond chatbots

The value of doing this goes beyond simple chatbots - if we get this right it can serve as a powerful foundation for all kinds of internal AI agents who can use the RAG engine to learn about our business in order to better align themselves with the tasks we set them. by providing a simple interface to all the data that defines our business, RAG becomes a valuable tool in many other processes - for instance it can help ai agents to understand regulatory rules in the country they operate in, and avoid them making wrong or illegal decisions. If we see RAG as foundational capability to an AI first business, its a good idea to treat it not dissimilarly to structured data pipelines - with carefully versioned code and automated orchestration and alerting set up. 

# Conclusion

And that's RAG in a nutshell! now finally you can feel free to rub that lamp to your hearts content, safe in the knowledge that this time at least, the Genie has done his homework.