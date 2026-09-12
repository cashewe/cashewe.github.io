---
title: 'Testing Pococks Agentic Coding Skills'
description: 'Are agent skills good actually'
pubDate: 'September 07 2026'
heroImage: '../../../public/diagrams/agentic_coding/hero.jpg'
---

A few months back (or a lifetime in AI terms), a friend of mine told me he had begun building 'SaaS products' with Claude. "you can't deny people are making money doing this" he'd said. "people made money selling cumrocket tokens, charlatans grifting is a sign of the times not the technology" I'd smugly replied.

But in those few months since something has changed.

The group of people claiming agentic workflows are "good now actually" has expanded beyond that one friend to include many that I know as technically gifted. Their processes are founded in traditional engineering discipline, and remain engineer-in-the-(agentic)-loop - it all feels dare I say - credible? Even amongst juniors in my team, quantity of work produced has noticeably and measurably increased in ways that can only really be justified by AI usage (although I'll happily argue *quality* has not improved in that time frame).

Armed with a free month's trial of codex, it seemed like now was finally time for me to whet my toes if not throw myself wholeheartedly into the deep end on a quest to discover whether "skills" and "workflows" truly act to enhance AI output beyond "slop".

What follows is an account of my first experience using agentic `skills`, so for context, a bit about me:

## Johnno, in a nutshell

![fact file](/diagrams/agentic_coding/fact_file.jpg)

- I have been building professional software in the data science and AI space for around 7 years, and coding in python specifically for around 10. like many others I have also taken a recent fancy to rust.
- AI usage is already a core part of my development process.
- I have been in the role of technical led for around 2 and a half years, meaning a fair amount of my time is already spent strategically planning rather than coding - I'd hope this would make agentic workflows a pretty natural fit.

## The product

For the experiment I've decided to realise an idea I've had vaguely in my mind for a year or so by now but have never quite gotten round to building. It's the sort of idea which is easy to convince yourself is well fleshed out and considered for as long as it remains in your head, but that I'm quite sure when push comes to shove, all that confidence will turn out to have been mostly vanity. *I'm sure you know the type*. 

In short, we will be building a JSON-configurable routing tool, which will allow users to pass messages between APIs using strategies such as conditional gates, fan-outs or random selection. This tool will be useful to me for building out API gateways, for instance by letting me pick which ML model to score with based on some categoric variable. I've included a visualiser for a simple configuration from the eventual solution below to help cement the idea.

<div style="display:grid; grid-template-columns: minmax(0, 1.2fr) minmax(0, 1fr); gap: 1rem; align-items: start; margin: 1.5rem 0;">
  <div style="border: 1px solid rgba(35,55,255,0.18); border-radius: 12px; padding: 0.75rem; background: rgba(255,255,255,0.55);">
    <pre class="mermaid" style="margin: 0;">
flowchart TD
    input(["Input"]) --> n0
    n0{"route-request<br/><small>deterministic gate</small>"}
    n1["expedite<br/><small>task</small>"]
    n2["standard<br/><small>task</small>"]
    n3[/"format-response<br/><small>map schema</small>"/]
    n0 -. "= &quot;high&quot;" .-> n1
    n0 -. "otherwise" .-> n2
    n1 -.-> n3
    n2 -.-> n3
    n3 --> output(["Output"])
    classDef boundary stroke-width:3px
    classDef failure stroke:#c62828,stroke-width:2px
    class input,output boundary
    </pre>
  </div>

  <div style="border: 1px solid rgba(35,55,255,0.18); border-radius: 12px; padding: 0.75rem; background: rgba(15,23,42,0.96); overflow-x: auto;">
    <pre style="margin: 0; color: #e2e8f0; font-size: 0.72rem; line-height: 1.55; white-space: pre-wrap; word-break: break-word; font-family: ui-monospace, SFMono-Regular, Menlo, monospace;">
{
  "entry": "route-request",
  "output": "format-response",
  "nodes": [
    {
      "id": "route-request",
      "type": "deterministic-gate",
      "select": "/priority",
      "cases": [
        {"operator": "eq", "value": "high", "target": "expedite"},
        {"operator": "otherwise", "target": "standard"}
      ]
    },
    {
      "id": "expedite",
      "type": "task",
      "task": "priority-handler",
      "next": "format-response"
    },
    {
      "id": "standard",
      "type": "task",
      "task": "standard-handler",
      "next": "format-response"
    },
    {
      "id": "format-response",
      "type": "map-schema",
      "mappings": [
        {
          "target": "/request-id",
          "path-to-source": "/request/id",
          "type": "string"
        },
        {
          "target": "/handled-by",
          "path-to-source": "/handled-by",
          "type": "string"
        }
      ]
    }
  ]
}
    </pre>
  </div>
</div>

To help users understand if they've configured it correctly, it will ship with a couple of CLI tools that will validate their JSON configurations and visualise their routing, intended for use in CI pipelines.

```
>>> check config.json

Error in config.json - routing type 'blahblahblah' not supported
Error in config.json - task 'giveMeSomeBeans' does not exist

-- summary --
found 2 errors in 0.1 seconds
```

<details>
<summary>the solution</summary>

I don't want the tool itself to become the focus of the article going forwards, so if its of interest to you, find it pip installable from pypi [here](https://pypi.org/project/camau/) and the code the agent eventually landed on [here](https://github.com/cashewe/camau).

</details>

I've picked this one from my backlog for a few reasons (which may or may not end up being relevant):

1. Since the tool is intended to be used between the user and the backend services, it needs to be *snappy*. This will require swapping between rust for the computation and python for the user interface. *does the workflow lead to sensible ownership boundaries?*
2. It sits in a goldilocks zone of being complex enough to not be trivial whilst remaining comfortably within my own abilities to build, thus enabling me to fairly critique the outcome. *does the workflow manage the complexity of the problem?*
3. The AI has to create an ergonomic python interface, a CI compatible CLI interface *and* a human readable JSON schema to configure it with. *Can the workflow design with the needs of multiple different types of customer in mind?*

As with all my tools, I've picked a Welsh name for this one - `camau`, meaning "steps". The agent initially chose a Welsh word for a user defined error type, but otherwise this decision had no obvious impact on the outcome.

## The Steps to produce "steps"

The workflow I'd like to test are the engineering [skills](https://github.com/mattpocock/skills) by Matt Pocock - which I've picked as they seem very popular, and are based on "[A Philosophy of Software Design](https://www.amazon.co.uk/Philosophy-Software-Design-John-Ousterhout/dp/1732102201)", from which I've taken alot of my own ideas over the years.

Pococks suggested end-to-end workflow makes use of the following skills:

- grill-with-docs
- to-spec
- to-tickets
- implement
- code-review

To ensure the best possible outcome, I'll be using the boundaries between skills to hand over to new agents, allowing me to avoid context rot. With that out the way its time to get grilled (with docs)!

### grill-with-docs

The Process in a Nutshell:

- **86** total questions asked, of which (not mutually exclusive):
  - 55 were "redundant" (i.e. "you said this, do you really want it?")
  - 4 were on features not requested (although not irrelevant, these weren't in the provided spec)
  - 7 were genuinely thought provoking
  - 7 were follow up questions
  - 15 saw me do anything other than agree with the models answer, of which 5 were absolute disagreement
  - 1 was just "do you agree with the prior 85 questions?"
- **119,914** tokens burnt (sol 5.6, high mode) equating to 62% of my free trials five-hour limit
- **~1 hour** spent on the grill, with a further `23:38` spent waiting for the 'docs' after the grill-me phase was complete

The headline for me here really is just how long the process was, and just how short it could've been if the model didn't keep asking so many chaff questions. Instead we burnt an entire hour dealing with questions like:
- 'what does simultaneous mean' -> "at the same time"
- 'should all described behaviours be implemented' -> "yes please pal!"

The docs for what its worth are a fine summary of the 86 questions and I did notice agents continuing to update them as I added features which was nice, though as with all AI generated text it is a bit *girthy* and *meandering*. Context management remains a key consideration when working with AI and keeping living documentation as a means of hand-off is genuinely smart - I'll be adopting this more in future. That notion extends to the specification which was a fine summary of the conversation.

### implement / code-review

The most interesting point here came from the agents reasoning - where I could see it hitting points of contention not included in my *86* point specification, and having to make decisions up for itself on the fly. The agent has managed to speed run learning the issues with waterfall planning in real time here. Additionally, the agents willingness to make honestly pretty sensible choices without the human-in-the-loop calls into question if the human ever really helped the loop in the first place, or if all of this was just vibe-coding with more steps.

As for the review skill, it was a mix of random code quality 'concerns' and arbitrary design changes whose justification didn't feel especially routed in any of the grilling outcomes. I reran this one in a fresh session just to make sure and its proposed changes the second time would have led to a completely different architecture to what it asked for from the first review. 
This isn't dissimilar to real code reviews sometimes (or if we're being honest, most of the time) to be fair, but equally the cost of running this one is such that I can't argue in favour of its value.

All of this points to something that was really missing that would've helped - some kind of software architecture design phase. The agent asked **a lot** about my functional requirements but nothing was put in place to guide it towards an agreed architecture or design strategy. Without this upfront consideration, what is there for the review to base its ideas of quality on anyways? 

A final thing worth noting - neither the implementation agent nor the review agent picked up on the fact that the requested CLI tooling was not built. In its place for some reason were a collection of importable python tools, complete with readme documentation suggesting users boot a python session and import / run them inline in a bash task. Baffling.

![meme](/diagrams/agentic_coding/meme.jpg)

Combining this with the previously mentioned on the fly decision making leads to a question as to whether I can truly trust the outcome to be what I expected it to be, however the structure of the original delivery was so spaghetti-esque I found it pretty much impossible to check. Theres no accounting for taste, but this is pretty damning for anyone hoping this workflow would avoid AI 'slop'.

## reflecting

At this point I feel its worth going back to my original set of questions.

1. does the workflow lead to sensible ownership boundaries? - **yes**, the rust / python boundary was sensibly defined.
2. does the workflow manage the complexity of the problem? - **sort of**, the solution was a mess of cross file imports, but it wasnt overly verbose.
3. can the workflow design with the needs of multiple different types of customer in mind? - **no**, this requirement was ignored.

Bluntly, I've not come away terribly impressed with this workflow. The grilling is a flashy enough idea to capture some imagination, but it seems to put the wrong problems first. An unfortunate reality is that user requirements can and will change, but luckily the domains they exist in tend remain relatively static - this is referenced in the book the skills are based on, which contains an entire chapter stressing that the smallest unit of change should be abstractions, not features. Grilling on desired features up-front ignores this wisdom, and likely led directly to a lot of the observed issues later on.

I'll take a moment here to compare this opening act to my own typical workflow. Usually, any given change will begin by exploring the problem space (to be clear, often by making liberal use of AI). The main difference here is the inversion of roles - in my typical workflow *I* am the grill and *the model* is the lamb chop. I think this more logically represents the roles involved - the model has the infinite wisdom so i grill it, whilst I have the contextual understanding to put together a coherent design.

This is illustrated well by the `/grill-with-docs` statistics: the number of blind agreements is high, but how much of that is because the AI was giving the right suggestions, and how much is because the AI lacked the contextual knowledge to ask the right questions to begin with? It can't truly challenge what it doesnt understand, and so it's left to ask very surface level questions of the user.

Before releasing the package into the wild, I took some time to go through a few loops without the skills enabled and was able to come to something I was relatively happy met the specification without much effort. I guess in so much as there is a conclusion here it seems to be that traditional engineering wisdom (abstractions, agile working) were designed to optimise *engineering* processes not specifically *human* processes, and I've not been convinced that agents are a sound argument for ditching these practices. For now at least, I'll continue using my AI as a rubber duck rather than a George Foreman.

cheers,

Johnno
