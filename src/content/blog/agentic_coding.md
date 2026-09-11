---
title: 'Testing Agentic Coding Skills'
description: 'Are agents good actually'
pubDate: 'September 07 2026'
heroImage: '../../../public/diagrams/agentic_coding/hero.jpg'
---

A few months back (or a lifetime in AI terms), a friend of mine told me he had begun building 'SaaS products' with Claude. "you can't deny people are making money doing this" he'd said. "people made money selling cumrocket tokens, charlatans grifting is a sign of the times not the technology" I'd smugly replied.

but in those few months since something has changed.

The group of people claiming agentic workflows are "good now actually" has expanded beyond that one friend to include many that i know as technically gifted. In my team, quantity of work produced has noticeably and measurably increased in ways that can only really be justified by AI usage (although I'll happily argue *quality* has not improved in that time frame).
Although some still claim to be vibing, a new group are starting to emerge. One foundationed in percieved engineering discipline, whose workflows remain engineer-in-the-(agentic)-loop, who seem - dare i say - credible?

Armed with a free months trial of codex, it seemed like now was finally time for me to whet my toes if not throw myself wholeheartedly into the deep end on a quest to discover whether "skills" and "workflows" truly act to enhance AI output beyond "slop".

What follows is an account of my first experience attempting to perform agentic engineering, so for context, a bit about me:

## Johnno, in a nutshell

![fact file](/diagrams/agentic_coding/fact_file.jpg)

- I have been building professional software in the data science and AI space for around 7 years, and coding in python specifically for around 10. like many others I have also taken a recent fancy to rust.
- I have been using AI as a coding *helper* since the first public release of chatGPT, and would consider AI usage a core part of my process at this point.
- I have been in the role of technical lead for around 2 and a half years, meaning a fair amount of my time is already spent strategically planning rather than coding - I'd hope this would make agentic workflows a pretty natural fit.

## The product

For the experiment I've decided to realise an idea I've had vaguely in my mind for a year or so by now but have never quite gotten round to building. its the sort of idea which is easy to convince yourself is well fleshed out and considered for as long as it remains in your head, but that I'm quite sure when push comes to shove, all that confidence will turn out to have been mostly vanity. *you know the type*. 

In short, we will be building a JSON-configurable routing tool, which will allow users to pass messages between APIs using strategies such as conditional gates, fan-outs or random selection. This tool will be useful to me for building out API gateways, for instance by letting me pick which ML model to score with based on some categoric variable.
To help users understand if they've configured it correctly, it will ship with a couple of CLI tools that will validate their JSON configurations and visualise their routing, intended for use in CI pipelines.

![camau](/diagrams/agentic_coding/front_page.jpg)

<details>
<summary>the solution</summary>

I don't want the tool itself to become the focus of the article going forwards, so if its of interest to you, find it pip installable from pypi [here](https://pypi.org/project/camau/) and the code the agent eventually landed on [here](https://github.com/cashewe/camau).

</details>

I've picked this one from my backlog for a few reasons (which may or may not end up being relevant):

1. Since the tool is intended to be used between the user and the backend services, it needs to be *snappy*. This will involve efficient programming and swapping between rust for the computation and python for the user interface. *can the agent design sensible ownership boundaries?*
2. It sits in a goldilocks zone of being complex enough to not be trivial whilst remaining comfortably within my own abilities to build, thus enabling me to fairly critique the outcome. I'd expect the whole thing to take somewhere between 3k and 5k LoC (excluding tests) if I were building it myself, and I'd ideally like the AI to land somewhere in this range. *can the agent manage the complexity of the problem?*
4. The AI has to create an ergonomic python interface, a CI compatible CLI interface *and* a human readable JSON schema to configure it with. *Can the agent design with the needs of multiple different types of customer in mind?*

As with all my tools, I've picked a Welsh name for this one - `camau`, meaning "steps". It'll be interesting to observe if my niche choice of language in any way affects the way the agent designs the solution.

## The Steps to produce "steps"

The first workflow I'd like to test are the engineering [skills](https://github.com/mattpocock/skills) by Matt Pocock - which I've picked as they seem very popular, and are supposedly based on "[A Philosophy of Software Design](https://www.amazon.co.uk/Philosophy-Software-Design-John-Ousterhout/dp/1732102201)", from which I've taken alot of my own ideas from over the years.

Pocock suggested end-to-end workflow makes use of the following skills:

- grill-with-docs
- to-spec
- to-tickets
- implement
- code-review

To ensure the best possible outcome, I'll be using the boundaries between skills to hand over to new agents, allowing me to avoid context rot. With that out the way its time to get grilled (with docs)!

### grill-with-docs

The Process in a Nutshell:

- 86 total questions asked, of which (not mutually exclusive):
  - 55 were "redundant" (i.e. "you said this, do you really want it?")
  - 4 were on features not requested (although not irrelevant, these weren't in the provided spec)
  - 7 were genuinely thought provoking
  - 7 were follow up questions
  - 15 saw me do anything other than agree with the models answer, of which 5 were absolute disagreement
  - 1 was just "do you agree with the prior 85 questions?"
- 119,914 tokens burnt (sol 5.6, high mode) equating to 62% of my free trials five-hour limit
- ~1 hour spent on the grill, with a further `23:38` spent waiting for the 'docs' after the grill-me phase was complete

The headline for me here really is just how long the process was, and just how short it could've been if the model didn't keep asking so many chaff questions. Instead we burnt an entire hour dealing with questions like 'what does simultaneous mean' and 'should all described behaviours be implemented' - like "yes please pal!"

The docs for what its worth are a fine summary of the 86 questions and i did notice agents continuing to update them as i added features which was nice, though as with all AI generated text it is a bit *girthy* and *meandering*. Context management remains a key consideration when working with AI and keeping living documentation as a means of hand-off is genuinely smart - I'll be adopting this. 

### to-spec / to-tickets

As with the docs, the specification was a fine summary of the conversation, no notes.

### implement / code-review

The most interesting point here came from the agents CoT - where i could see it hitting points of contention not included in my *86* point specification, and having to make decisions up for itself on the fly. The agent has learnt the issues with waterfall planning in real time here. worse still, the agents willingness to make honestly pretty sensible choices without the human-in-the-loop calls into question if the human ever really helped the loop in the first place here.

As far as the review goes... Just use a linter. 

The skill was a mix of random code quality 'concerns' and arbitrary design changes whos justification didn't feel especially routed in any of the grilling outcomes. I reran this one in a fresh session just to make sure and its proposed changes the second time would have lead to a completely different architecture to what it asked for from the first review. 
This isn't dissimilar to real code reviews sometimes (or if we're being honest, most of the time) to be fair, but equally the cost of running this one is such that i cant really argue in favour of its value.

Something that was really missing that would've helped here was any kind of software architecture design phase. The agent asked alot about my functional requirements but nothing was put in place to guide it towards a good architecture. without this upfront consideration, what is there for the review to base its ideas of quality on anyways? 

A final thing worth noting - neither the implementation agent nor the review picked up on the fact that the request CLI tooling was for some reason instead built as importable python tools, complete with readme documentation suggesting users boot a python session and import / run them inline in a bash task. baffling.


combining this with the previously mentioned on the fly decision making leads to a question as to whether i can truly trust the outcome to be what i expected it to be, however the code quality of the original delivery was so poor I found it pretty much impossible to check. This is pretty damning for anyone hoping this workflow would avoid AI 'slop'.

## reflecting

At this point I feel its worth going back to my original set of questions.

1. does the workflow lead to sensible ownership boundaries?
2. does the workflow manage the complexity of the problem?
3. can the agent design with the needs of multiple different types of customer in mind?

If we measure outcomes based solely on these skills, the answers in brief would be:

1. yes, the rust / python boundary was sensibly defined.
2. not really no, the solution was a mess to read through.
3. no, this requirement was ignored.

Bluntly, I've not come away terribly impressed with this workflow. The grilling, whilst not a bad idea on paper, seems to put the wrong problems first. User requirements can and will change, but the domain they exist in will likely remain relatively static - this is referenced in the book the skills are based on, which contains an entire chapter stressing that the smallest unit of change should be abstractions, not features.

I'll take a moment here to compare this opening act to my own typical workflow. Usually, any given change will begin by exploring the problem space (to be clear, often by making liberal use of AI). The main difference here is the inversion of roles - in my typical workflow *I* am the grill and *the model* is the lamb chop. I think this more logically represents the roles involved - the model has the infinite wisdom so i grill it, whilst i have the contextual understanding to put together a coherent design.

This is illustrated well by the `/grill-with-docs` statistics: the number of blind agreements is high, but how much of that is because the AI was giving the right suggestions, and how much is because the AI lacked the contextual knowledge to ask the right questions to begin with? It can't truly challenge what it doesnt understand, and so it's left to ask very surface level questions of the user.

Before releasing the package into the wild, I took some time to go through a few loops without the skills enabled and was able to come to something I was relatively happy met the specification without much effort. For now at least, I'll continue using my AI as a rubber duck rather than a George Foreman.

cheers,

Johnno
