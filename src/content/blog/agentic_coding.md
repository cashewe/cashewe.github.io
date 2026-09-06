---
title: 'Testing Agentic Coding Skills'
description: 'Are agents good actually'
pubDate: 'September 07 2026'
heroImage: '../../../public/diagrams/agentic_coding/hero.jpg'
---

A few months back (or a lifetime in AI terms), a friend of mine told me he had begun building 'SaaS products' with claude. "you can't deny people are making money doing this" he'd said. "people made money selling cumrocket tokens, charlatans grifting is a sign of the times not the technology" I'd replied.

but in those few months since something has changed.

The group of people claiming agentic workflows are "good now actually" has expanded beyond that one friend to include many that i know as technically gifted. In my team, quantity of work produced has noticeably and measurably increased in ways that can only really be justified by AI usage (although I'll happily argue *quality* has not improved in that time frame).
In honesty, the FOMO had slowly been creeping in for a while when i noticed codex were offering a free months trial. It seemed like now was finally time for me to whet my toes if not throw myself wholeheartedly into the deep end. 

My goal: figure out if agentic workflows are able to *engineer*, or just able to code.

What follows is an account of my first experience using agentic coding, so for context, a bit about me:

## Johnno, in a nutshell

![fact file](/diagrams/agentic_coding/fact_file.jpg)

- I have been building professional software in the data science and AI space for around 7 years, and coding in python specifically for around 10. like many others I have also taken a recent fancy to rust.
- I have been using AI as a coding *helper* since the first public release of chatGPT, and would consider AI usage a core part of my workflow at this point - via both chat interfaces and GitHub copilot, but never through full agentic platforms.
- I have been in the role of technical lead for around 2 and a half years, meaning a fair amount of my time is already spent strategically planning rather than coding - which should be a natural fit for agentic development.

## The product

For this experiment I've decided to realise an idea I've had vaguely in my mind for a year or so by now but have never quite gotten round to building. its the sort of idea which is easy to convince yourself is well fleshed out and considered for as long as it remains in your head, but that I'm quite sure when push comes to shove all that confidence will turn out to have been mostly vanity. *you know the type*. 

In short, we will be building a JSON-configurable routing tool, which will allow users to pass messages between APIs using strategies such as conditional gates, fan-outs or random selection. This tool will be useful to me for building out API gateways, for instance by letting me pick which model to score with based on some categoric variable.
To help users understand if they've configured it correctly, it will ship with a couple of CLI tools that will validate their JSON configurations and visualise their routing, intended for use in CI pipelines.

![camau](/diagrams/agentic_coding/front_page.jpg)

<details>
<summary>the solution</summary>

I don't want the tool itself to become the focus of the article going forwards, so if its of interest to you, find it pip installable from pypi [here]() and the code the agent eventually landed on [here](https://github.com/cashewe/camau).

</details>

I've picked this one from my backlog for a few reasons (which may or may not end up being relevant):

1. Since the tool is intended to be used between the user and the backend services, it needs to be *snappy*. This will involve efficient programming and swapping between rust for the computation and python for the user interface. *can the agent design sensible ownership boundaries?*
2. It sits in a goldilocks zone of being complex enough to not be trivial whilst remaining comfortably within my own abilities to build, thus enabling me to fairly critique the outcome. I'd expect the whole thing to take somewhere between 3k and 5k LoC (excluding tests) if I were building it myself, and I'd ideally like the AI to land somewhere in this range. *can the agent manage the complexity of the problem?*
4. The AI has to create an ergonomic python interface, a CI compatible CLI interface *and* a human readable JSON schema to configure it with. *Can the agent design with the needs of multiple different types of customer in mind?*

As with all my tools, I've picked a Welsh name for this one - `camau`, meaning "steps". It'll be interesting to observe if this in any way affects the way the agent designs the solution.

## The Steps to produce "steps"

In the name of giving the agentic *engineering* processes an honest go, I've ignored the self-proclaimed vibe-coders and looked for existing engineers ideas on best practices. The first I'd like to test are the popular engineering [skills](https://github.com/mattpocock/skills) by Matt Pocock - which I've picked as they seem very popular, and are supposedly based on "[A Philosophy of Software Design](https://www.amazon.co.uk/Philosophy-Software-Design-John-Ousterhout/dp/1732102201)" (a book I've taken alot fo my own approach from).

Helpfully for beginners like myself, Pocock provides a suggested end-to-end workflow, making use of the following skills in order:

- grill-with-docs
- to-spec
- to-tickets
- implement
- code-review

So I guess its time to get grilled (with docs)!

### grill-with-docs

The Process in a Nutshell:

- 86 total questions asked, of which (not mutually exclusive):
  - 55 were "redundant" (i.e. "you said this, do you really want it?")
  - 4 were on features not requested (although not irrelevant, these weren't in the provided spec)
  - 7 were genuinely thought provoking
  - 7 were follow up questions
  - 15 saw me do anything other than agree with the models answer, of which 5 were absolute disagreement
  - 1 was just "do you agree with the prior 85 questions?"
- 119,914 tokens burnt (sol 5.6, high mode) equating to 62% of my five-hour limit
- ~1 hour spent on the grill, with a further `23:38` spent waiting for the 'docs' after the grill-me phase was complete

The headline for me here really is just how long the process was, and just how short it could've been if the model didn't keep asking so many chaff questions. with just seven genuinely thought provoking questions (of which just one was a follow up i.e. only one "needed" to be asked in a second round) we could've had one or two rounds of grilling without loosing anything too meaningful here. Instead we burnt an entire hour dealing with questions like 'what does simultaneous mean' and 'should all described behaviours be implemented'. The docs for what its worth are a fine summary of the 86 questions, nothing more nothing less.

The preservation of the outcomes through generated docs is a neat trick however, and one id certainly consider taking forwards. Context management remains a key consideration when working with AI and keeping living documentation as a means of hand-off is genuinely smart. 

### to-spec / to-tickets

As with the docs, the specification was a fine summary of the conversation, nothing worthy of praise or criticisms - I'm not convinced I fully understand the need for docs, specification *and* tickets separately though if I'm honest.

### implement / code-review

the agent ended up righting 5,132 lines of code to produce the initial package - so (surprisingly) comfortably within the region of my estimate. Perhaps the more interesting point here came from the agents CoT - where i could see it hitting points of contention not included in my *86* point specification, and having to make decisions up for itself on the fly. was my input ever really useful if its willing and able to build without it? It seems to me as if the agent has learnt the issues with waterfall planning in real time here.

As far as review goes... Just use a linter. The skill was a mix of random code quality 'concerns' and arbitrary design changes whos justification didn't feel especially routed in any of the human-in-the-loop grilling parts. I reran this one in a fresh session just to make sure and its proposed changes the second time would have lead to a completely different architecture to what it proposed in the first review. 
This isn't dissimilar to real code reviews sometimes (or if we're being honest, most of the time) to be fair, but equally the cost of running this one is such that i cant really argue in favour of its value.

Something that was really missing that would've helped here was any kind of software architecture design phase. The agent asked alot about my functional requirements but nothing was put in place to guide it towards a sustainable design and I'd say the result of that speaks for itself. without this upfront consideration, what is there for the review to base its ideas of quality on?

### reflecting

I went through a few rounds of the skills to get a proper feeling for them, and at this point I feel its worth going back to my original set of questions.

1. can the agent design sensible ownership boundaries?
2. can the agent manage the complexity of the problem?
3. can the agent design with the needs of multiple different types of customer in mind?

If we measure outcomes based solely on these skills, the answers in brief would be:

1. yes, the rust / python boundary was sensibly defined with python generally kept shallow.
2. not really no, although the solution does work and works well, the code itself is a mess and very hard for a human to dip into. The agent needed repeated railroading to keep things like schema management in a single location.
3. again, no. The agent actually designed all the tools explicitly requested as CLI tools in python as importables until i intervened. This suggests some of my requirements were actually ignored by the agent and in this case, the result is the CLI users were missed entirely.

Overall then, I've not come away terribly impressed with this workflow. The grilling, whilst not a bad idea on paper, seems to put the wrong problems first. user requirements can and will change, but the domain they exist in will likely remain relatively static - this is referenced in the book the skills are based on, which contains an entire chapter stressing that the smallest unit of change should be abstractions, not features.

I'll take a moment here to compare this opening act to my typical workflow. Usually, any given feature will begin by exploring the problem space (to be clear, often by making liberal use of AI). The main difference here is the inversion of roles - in my typical workflow *I* am the grill and *the model* is the lamb chop.

This process usually takes a much shorter amount of time as its a more directed volley of questions, and i think more logically represents the roles involved - the model has the infinite wisdom so i grill it, whilst i have the contextual understanding to put together a coherent design.

This is subtly seen in the `/grill-with-docs` statistics: the number of blind agreements is high, but how much of that is because the AI was giving the right suggestions, and how much is because the AI lacked the contextual knowledge to ask the right questions to begin with? the redundant question count points more towards the latter scenario and thats certainly how it felt by the end.

The rest of the workflow appears fine - but given its all foundationed in this first `/grill-with-docs` skill, it unfortunately never really takes off.

## overall summary

Upon completion I had:

- 5,132 lines of code
- three python tools
- *no* CLI tools, despite clear instructions to include them
- spent 3 hours to "build" the solution
- runtime is *snappy*

Theres no denying that i would've taken far longer than 3 hours to build a working solution to this prior, but that doesn't mean theres no challenging the idea that speed is always desirable. what I have been left with is a larger than expected codebase that i cant navigate from memory, no clear understanding of quite how the thing works and clearly at least two visibly wrong features (the two python tools i expected as CLI). If this were built for production use, I would've been quite happy with the process taking twice as long had it resulted in deeper understanding and avoided the obvious problems here - doubly so if it included more granular commits for targeted rollback. That said to get a working prototype out at this pace is obviously pretty novel and could be a huge strategic advantage if used well.

For now at least, I'll continue using my AI as a rubber duck rather than a George Foreman.

cheers,

Johnno
