---
title: 'Intro to RAG'
description: 'We all sit puzzled wondering wtf is RAG anyways?'
pubDate: 'August 01 2026'
heroImage: '../../../public/diagrams/mathematically_optimised_chunking.jpg'
---

you may have heard of the concept of localised chatbots before - for instance as a means of automatically answering customer queries about your business. you may even have heard the term RAG (or `retrieval augmented generation`) before... but what does it actually mean? and how does it work? 

in this short article I'll explain the underlying technology in simple terms and demonstrate how, when leveraged properly, RAG can allow your AI systems to become more than simple chatbots; embedding business knoweldge deep into your processes.

# Context

somewhere far from here (or maybe very close, reader discerning) the worlds top engineers are spending *trillions* of dollars producing new AI models. The models learn to write coherent sentences by studying text written by people from all across the world and all throughout history - which unsurprisingly leads the AI to develop an impressive grasp of language, as well as a deep understanding of *popular* topics. 

but how does your business fit into that? The answer, frankly, is it doesn't. 

the AI is generally knowledgeable, but has no means of knowing anything particular about local topics. unfortunately, since the AI isn't 'thinking' but simply providing the statistically most likely answer to your questions, its not capable of telling you what it does or doesn't 'know' and instead gives a confidentially wrong response. you can keep rubbing the lamp, but the genie will only ever pretend to answer your wishes.

![genie](/gifs/slop_genie.gif)

A suggestion I often see is to try training AI models on your local information (a method called 'fine-tuning'). whilst *possible* this is typically not that *desirable* as:

- it is expensive
- it causes you to 'overfit' on local data
- it doesnt provide a searchable 'memory' but rather new patterns that may or may not organically see use

Using the RAG technique, we will instead intercept our incoming queries, search through our data for grounding information, and inject it into our question before it hits the AI to provide the relevant context to the model. 

# How does it work?

At its simplest, we can break the RAG technique down into three simple steps:

1. ahead of time, we split our text into smaller, more focused 'chunks' of information which we store in a database somewhere
2. when we recieve a question, we first 'retrieve' the chunks that seem most relevant
3. We 'rehydrate' our context with the retrieved chunks, and allow the AI to generate a response.

Personally I think it easier to understand the reason behind the process if we start at the second step and then work our way out in both directions.

## 2: Retrieving the chunks

In your daily life you'll be familiar with many ways to discover information.

- On streaming services, we have genre labels to reduce the search space. 
- On e commerce websites we use free text search matching techniques to find relevant products. 
- In libraries we have whatever the heck the 'Dewey Decimal System' actually is. 

When it comes to free text searching, current trends have largely settled on a technique known as 'vector search'. In it, we assign a list of numeric values to text based on the 'semantic meaning' of the text - we call this list an 'embedding' because it *embeds* the meaning of the underlying text into something that conspicuously resembles a set of coordinates: 

(x1, x2, x3, ...)

if we make an embedding of the input question (importantly with the *same* embedding model), we will end up with coordinates embedding meaning of both the question and the underlying data. from here, we can use some simple trigonometry to find the text which is physically nearest the input question, or in other words to find the underlying data whose meaning is deemed most similar to the input query. This can be supplemented by other techniques such as text queries in a method called 'hybrid search', which is fast gaining popularity.

Importantly, an embedding represents the whole text all at once, not a sentence by sentence reading. This means its value is heightened if the text is concentrated on a single topic, and if each *bit of text* is of relatively even size. For this reason it is beneficial to split our data up into smaller chunks that ideally represent single topics of information, which leads us back to step 1.

## 1: chunking the text

Now that we have established the reason *why* we chunk, lets look at quite *how* we might do this. A simple default technique might be to simply split the input text every 'N' tokens (a token is a set of letters not neccessarily aligning to words - AI models 'see' in tokens not words so most techniques using AI will measure in tokens) with a rolling 'overlap' containing some fraction of tokens from the previous chunk. this rolling overlap means that if a given sentence is very relevant to the input query, I am likely to draw in context from both before and after it at retrieval time (as it will likely match both chunks it appears in), helping to ensure I get enough information to get by. 

For an expansion on this naive chunking strategy, see my previously published article 'Mathematically Optimised Chunking Strategy' - its just as easy to implement and likely to outperform in practice. 

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

# Beyond chatbots

The value of doing this goes beyond simple chatbots - if we get this right it can serve as a powerful foundation for all kinds of internal AI agents who can use the RAG engine to learn about our business in order to better align themselves with the tasks we set them. by providing a simple interface to all the data that defines our business, RAG becomes a valuable tool in many other processes - for instance it can help ai agents to understand regulatory rules in the country they operate in, and avoid them making wrong or illegal decisions. If we see RAG as foundational capability to an AI first business, its a good idea to treat it not dissimilarly to structured data pipelines - with carefully versioned code and automated orchestration and alerting set up. 

# Conclusion

And that's RAG in a nutshell! now finally you can feel free to rub that lamp to your hearts content, safe in the knowledge that this time at least, the Genie has done his homework.

cheers,

*Johnno*