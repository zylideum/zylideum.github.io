---
layout: post.njk
title: stop telling developers how to fix vulnerabilities
description: they are no longer the point of control.
date: 2026-03-13
tags: posts
permalink: "/blog/close-the-loop/"
---

##### *these thoughts were handcrafted without the use of ai. for the adhd among us, key points are outlined at the end.*

# 0. the realization: who cares?

once upon a time, i was an offensive security engineer who popped some cool shells and had stories to tell about it.

for others in the security space, these stories were often helpful, enlightening, maybe interesting. but there was *some* value in telling these stories, showing the tecnniques, talking about how to pwn their own targets in the future. sharing our collective exploits **is still valuable** for us hackers.

then i pivoted back to application security and building. helping developers secure apps, architecting secure design patterns, embedding in their workflows.

i had a *groundbreaking* idea: what if i share those same stories with devs to help them secure their code and avoid these patterns in the future? then they could write stronger code and maybe even double-check what they've already deployed!

## except: the implementation layer is disappearing

the realization came quickly after putting myself in the developer's shoes, listening to myself give a talk on my favorite paths to root:

> "this is great", i imagine dev-me saying. "i can't wait to tell claude not to use `BinaryFormatter();`"

**this is the revelation** - the value is no longer in telling developers how to fix things. they will pass it off to their coding agents as they continue to orchestrate their grand designs.

telling developers about vulnerabilities is like telling your **delivery driver** that you want **extra parmesan**. you could have just put that in your order in the first place.

the implementation layer is disappearing fast. we are approaching an age where developers will not even know the full extent of the code that exists in their own repositories, let alone which dangerous methods are used. 

the idea that they would ever pluck a vulnerability from a jira backlog with the intent of hand-crafting a resilient fix is almost absurd.

---

# 1. control has shifted

telling developers how to fix things leads to them taking one of these actions in an ai-first world:

- do nothing (this never really changed)
- have **openclaudexgpt*** fix the problem
- tell **openclaudexgpt*** to never make this mistake again (*10x engineer*)
##### **whatever model devs love that month*

therein this conceptualization lies the obvious improvement for us as security engineers:
> wrangle the agents, not the devs

the control point has shifted. devs are orchestrating systems, not writing code. remediation advice is therefore wasted on the **delivery driver**. (*in this case, delivery of product and value rather than pizza.*)

in a world where the implementation layer is truly gone, we must rely on correcting the **composition** of these systems rather than the implementor.

# 2. scale is staggering

> 50,000 LOC and 150 PRs a day, every day.

code gen is increasing in adoption so much that the stats above barely scratch what a mid-level engineer is pumping out at some of the most ai-forward companies around right now. it's clear to everyone that the only way to tackle the ai problem is with an ai solution.

it's been obvious from the hundreds of new startups in my linkedin inbox that agentic security review, ai guardrail engines, and hyperintelligent-business-logic-abusing next-generation sast companies (*an llm is told: 'find idor'*) are the hot new ideas that are going to fix security forever.

it makes sense for these companies to be tackling ai security the way they currently are.

the fastest go-to-market approach to the problem is to throw ai in front of ai: review the pr with ai, scan the code with ai-backed context, have an ai agent threat model and find design flaws.

this is a certainly a start, but it lacks a complete vision of the future. it assumes the point of control has never changed. it only continues the pattern of producing developer-facing output meant for the implementation layer.

## solution is easy! like always!

security engineers are tempted to try to solve this by using insights to feed claude.md files and load up the agent context like a baked potato.

```
> Do NOT hardcode my API keys.
> Do NOT push .env to prod.
> Do NOT write insecure code.
> Make no mistakes.
> Don't forget the bacon.
```

sure, you could find a way to push this out to all dev machines, or safeguard it in your company-wide internal coding agent's system prompt (*your cto named it jarvis because it's so smart and helpful*). but this isn't scalable.

- it's prone to constant guardrail gaps.
- you are bloating the sacred, divine context windows of your engineers.
- it turns into policy soup and jarvis starts guessing what you want.

non-determinism is actually not a problem point in the above - it is what enables the solution.

#### *if you're considering having ai write you a 27-amendment constitution for your security.md claude file, **please** do a double take on the above. it will suck.

# 3. the output is all wrong

without guiding an agent in a future run, in every future run, it will continue to make the same mistakes. their brains are ephemeral and will not learn how to avoid design patterns over time. unlike humans, they will make the same mistakes each time if we allow them to.

> if the output of new-wave ai tooling is the same clippy-style suggestions, we are not going to move the needle at all.

i am arguing that our output - all of our current output - is aimed at the wrong system.

- pentest findings offer remediation advice. **wrong**.
- sast tool offers code fixes. **wrong**.
- pr review agent offers in-line commentary. **wrong**.

why are they **wrong**? these outputs still assume developers are the implementation layer.

if we continue to simply review ai output (even at scale), we are **mopping the floor while the sprinklers are running**.

the output must inform the **agent**, not the **developer**.

# 4. close the loop

## solution is hard... like always...

okay, the output is wrong, our ideas are wrong, everything sucks. what am i actually getting at?

these companies are solving for undefined outcomes. the industry response is pumping out slop to counter the slop. they think devs will read the slop, fix the slop, repeat the slop.

> we must counter by defining the outcome and closing the loop. output should be fed directly into the agent, who now owns the point of control over implementation.

i am seeing everyone thinking about this as if it's okay to continue burdening the developer **like we've been doing for the last 20 years**.

the counter is to build an entirely closed loop that evolves the agent of choice over time. not with bloated context, but with intelligent dispatching and a continuous feedback loop. 

- **ARCHAIC:** pentest firms end a report with remediation advice. 
- **AI-NATIVE:** pentest firm delivers an ai skill that directly improves an agent's ability to identify it in the future.
- **ARCHAIC:** code scanner finds a sql injection and leaves a pr comment to consider parameterizing the statement.
- **AI-NATIVE:** code scanner informs the agent about the finding, which creates a new **subagent** to review this pattern in the future. every future.

prior to ai, tooling was deterministic. they detected what they were programmed to detect. if a scanner doesn't pick something up, how would we inform the system to do better next time?

ai changes this dynamic - non-determinism is what allows us to achieve this idea of a closed loop process to evolve the behavior of the system itself.

this is all harder than writing a big text file. (*surely we can't call ourselves engineers if that becomes our day-to-day...*)

---

# 5. subagents? who'd want to spy on my italian hoagie?

ai must become the primary security-aware participant in the development process. to learn, it must be given context. to be given context, it must be informed.

to stay effective after being informed millions of times, we must adopt specialized agents within our closed loop process and dispatch the correct agents for the correct problems.

instead of trying to teach a single monolithic coding agent every security rule, we create specialized agents that enforce narrow security behaviors. when relevant patterns appear, these agents are dispatched to evaluate and guide the implementation.

the vision is simple on the surface: determine the intent of the orchestrator's will being imposed, dispatch the experts, and feed information back to the real point of control.

thus, the subagents guide the design in every future. 

new subagents are created through the informed ones. 

# 6. the endgame

application security spent 20 years teaching developers how to write safer code.

> the next 20 will be spent teaching machines.

security findings should not end as backlog items, or documentation, or pr comments. they should end as capabilities added to the system that writes the code.

each discovery should make the development system more secure. not the developer. the system.

the long-term architecture for doing this properly is still evolving - i'm currently building approaches to close the loop in practice.

the direction is clear. the implementation layer has shifted, and security must shift with it.



# 7. key points

> - developers are becoming orchestrators, not authors

> - security outputs assume developers are the control point for implementation

> - developers will relay remediation to an agent anyways

> - scale of generated code suggests today's output of ai review is not currently scalable

> - ai-forward tools still produce output assuming developers are the control point

> - we must close the loop by feeding findings back into the agent architecture

> - feedback loops and autonomous development will be the endgame