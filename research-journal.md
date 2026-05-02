# Research Journal: Quest Seed Usability Lab

## Week 1 - Generic prompts look better than they are

I started by asking a model for "a fantasy quest idea." The output looked fine at first: a village, a missing relic, a forest, and a final confrontation. But it also sounded like hundreds of other fantasy quests.

The problem was not fluency. The sentences were readable. The problem was usefulness. If I were making a real game, I would still need to invent the mechanic, the stakes, the map logic, the reward, and the reason the player should care.

Question raised: Should AI game-writing tools be evaluated by how polished they sound, or by how much design work they actually save?

## Week 2 - Adding realistic constraints

This week I changed the prompt. Instead of "write a fantasy quest," I gave a more project-like brief:

> Write a quest seed for a 2D horror game where the player is a lost delivery driver. Include one non-combat mechanic, one map location, and one object that changes meaning by the end.

The output improved. It gave me a broken radio, a locked service tunnel, and a package that turned out to be addressed to the player. That was more useful because it contained playable hooks.

Still, the model avoided implementation details. It did not say how the radio mechanic worked or how the player would discover the twist. It created a mood, not a playable design.

## Week 3 - Source detour

Consensus searches on procedural content generation showed that game generation has a long research history. The most useful source direction was quest generation with language models. The papers did not make me think, "AI can write games now." They made me think, "Generated content needs evaluation criteria."

That changed my research question. I do not want to compare models by vibes. I want to compare outputs by developer usefulness:

- Does it respect constraints?
- Does it include a playable action?
- Does it create a conflict or choice?
- Does it avoid generic filler?
- Would I keep any part of it in a real prototype?

## Week 4 - The usability checklist

I built a small checklist for each generated quest seed:

- Constraint fit: 0 to 2
- Playable mechanic: 0 to 2
- Narrative hook: 0 to 2
- Specificity: 0 to 2
- Rewrite burden: low, medium, or high

The most interesting output was not the most dramatic one. It was a small "repair the elevator by choosing which floor to sacrifice" quest. It gave a clear mechanic, a moral choice, and a map structure. The prose was plain, but the design was usable.

This surprised me because I expected the best writing to be the most useful. It was not. The best seed was the one that gave me something I could prototype.

## Week 5 - Benchmarks versus project tests

Generic benchmarks usually test broad ability. My project test asks a narrower question: can this tool help me with this kind of game task?

That means a smaller model might be useful if the prompt is structured well, and a larger model might still be unhelpful if it produces beautiful but unplayable descriptions.

The paper should argue for realistic tests. If someone wants to know whether an AI tool helps with game development, the test should look like game development.

## Week 6 - Paper direction

The paper will not claim that AI can generate finished game content. It will claim that a project-based evaluation reveals a difference between fluent writing and usable design support.

The honest limitation is that I am the only evaluator in this example. A better version would ask several game developers or students to rate the same outputs, then compare agreement.
