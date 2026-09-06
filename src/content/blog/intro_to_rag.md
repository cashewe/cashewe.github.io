---
title: 'Intro to RAG'
description: 'We all sit puzzled wondering wtf is RAG anyways?'
pubDate: 'August 01 2026'
heroImage: '../../../public/diagrams/intro_to_rag/hero.jpg'
---
*RAG is a process in which we intercept questions for AI and wrap them in relevant business context. In this short article I'll explain the underlying methodology and demonstrate how, when leveraged properly, it can allow your AI systems to become more than simple chatbots; embedding business knowledge deep into your processes.*

# Context

You may have heard of the concept of localised chatbots before - for instance as a means of automatically answering customer queries about your business. You may even have heard the term RAG (or `retrieval augmented generation`) before... But what does it actually mean? And how does it work? 

Somewhere far from here (or maybe very close, reader discerning) the worlds top engineers are spending [**billions**](https://www.gartner.com/en/newsroom/press-releases/2026-1-15-gartner-says-worldwide-ai-spending-will-total-2-point-5-trillion-dollars-in-2026) of dollars producing new AI models.

The models learn to write coherent sentences by studying text written by people from all across the world and all throughout history - which unsurprisingly leads the AI to develop an impressive grasp of language, as well as a deep understanding of *popular* topics and the *latest hotness*. 

But how does your business fit into that? The answer, frankly, is it doesn't.

The AI is generally knowledgeable, but has no means of knowing any organisation specific information. Unfortunately, since the AI isn't 'thinking' but simply providing the statistically most likely answer to your questions, its not capable of discerning correctness and instead gives a confidentially wrong response. You can keep rubbing the lamp, but the genie will only ever pretend to answer your wishes.

![genie](/gifs/slop_genie.gif)

A suggestion I often see is to try training AI models on your local information (a method called ['fine-tuning'](https://www.ibm.com/think/topics/fine-tuning)). Whilst *possible* this is typically not *desirable* as:

- It is incredibly expensive
- It can cause you to 'overfit' on local data
- It is very hard to keep up to date with an evolving dataset

Using the RAG technique, we will instead intercept our incoming queries, search through our data for grounding information, and inject it into our question before it hits the AI to provide the relevant context to the model. Think of it as a discovery layer on top of your non-structured data.

Then, think of *the value* of such a layer 🤑

# How does it work?

At its simplest, we can break the RAG technique down into four simple steps:

1. Ahead of time, we split our text into smaller, more focused 'chunks' of information
2. When a question arrives, we 'retrieve' the chunks that seem most relevant
3. We 'rehydrate' our context with the retrieved chunks
4. we allow the AI to 'generate' a response.

I think its easier to understand the *why* that underpins the process if we start at the second step and then work our way out.

## 2: Retrieving the chunks

In your daily life you'll be familiar with many ways to discover information.

- On streaming services, we have genre labels to reduce the search space. 
- On e commerce websites we use free text search matching techniques to find relevant products. 
- In libraries we have whatever the heck the 'Dewey Decimal System' actually is. 

When it comes to looking through digital text, current trends have largely settled on a technique known as ['vector search'](https://www.ibm.com/think/topics/vector-search). In it, we assign a list of numeric values to text based on the 'semantic meaning' of the text - we call this list an 'embedding' because it *embeds* the meaning of the underlying text into something that conspicuously resembles a set of coordinates: 

![coordinates for words](/diagrams/intro_to_rag/coordinate_mapping.jpg)

When our RAG system receives a query, we will make an embedding out of this query with the same method we used to create embeddings out of the underlying text data - ending up with coordinates representing the question and all the underlying data. From here, we can use some simple geometry to find the text which is nearest the input question, or in other words to find the underlying data whose meaning is deemed most similar to the input query. 

Importantly, an embedding represents the whole text all at once, not a sentence by sentence reading. This means its value is heightened if the text is concentrated on a single topic, and if each *bit of text* is of relatively even size. For this reason it is beneficial to split our data up into smaller chunks that ideally represent single topics of information, which leads us back to step 1.

![dilute signal](/diagrams/intro_to_rag/dilute_signal.jpg)

## 1: chunking the text

Now that we have established the reason *why* we chunk, let's look at quite *how* we might do this. A simple default technique might be to simply split the input text every 'N' [tokens](https://blogs.nvidia.com/blog/ai-tokens-explained/) (tokens are how AI *sees* things). When 'N' is kept small, this should satisfy our needs for concentrated text, but know that many [more complex chunking methods](https://cashewe.github.io/blog/optimal-chunking-strats/) exist for more specialist needs.

## 3: Rehydrating the context

With our data carefully chunked, our vector search based retrieval is able to find the most semantically similar chunks. From here, we want to provide the AI model with the data it needs to answer our initial question (a process called 'rehydration'). In the simplest possible system we'll pass it the top outputs from the retrieval stage, however in practice this will often not suffice.

For example, we may find that similarity and usefulness aren't the same metric i.e. here we can see chunks about motorbikes, pedal bikes and [Gino D'Acampos grandmother](https://www.youtube.com/watch?v=A-RfHC91Ewc) are all equally similar to a question on riding in the cycle lane: 

![types of bike](/diagrams/intro_to_rag/bike_conundrum.jpg)

It is common to use a 'reranker' model here as a judge - to filter the retrieved chunks by likely usefulness. This helps avoid confusing the AI in the generative step that follows.

## 4. Generating a response

Once we're happy with our rehydration strategy, we can pass the new context consisting of:

- The question
- The relevant data from our documents
- A prompt explaining we need an answer grounded in the data we have provided

On to the AI model, which can now provide us with an answer to our initial question at a fraction of the cost and time compared to training our own.

# Conclusion

And that's RAG in a nutshell! At its heart its a very simple idea - expose the information containing the answer directly to the AI with a queriable interface. Now finally you can feel free to rub that lamp to your hearts content, safe in the knowledge that this time at least, the genie has done his homework.

![studious genie](/diagrams/intro_to_rag/studious_genie.jpg)

cheers,

*Johnno*