# Game-Builder-Free: Evaluating AI Game-Writing Outputs Through Realistic Project Constraints

## Abstract

This worked example describes a small evaluation of AI-generated quest ideas for game development. The imagined Hugging Face Space, Quest Seed Usability Lab, asks language models to generate quest seeds from realistic project briefs rather than generic prompts. The central question is whether project-based tests reveal more about AI tool quality than broad claims that a model is "good at writing." A small rubric separates fluent prose from developer usefulness by scoring constraint fit, playable mechanics, narrative hook, specificity, and rewrite burden. The pilot suggests that the most useful outputs are not always the most polished. Some generated quests sound dramatic but offer little a developer could prototype, while plainer outputs sometimes contain clearer mechanics and design constraints. The paper argues that AI tools for game development should be evaluated in context, using tasks that resemble the work a developer actually needs to do.

## 1. Introduction and research question

Game development is evolving rapidly; modern production now extends far beyond coding to include cinematography, lighting, narrative design, and music. The proliferation of generative AI has provided developers with a vast toolkit for creating images, audio, and video. While AI-assisted coding is often the first application that comes to mind, developers are increasingly using these tools for artistic tasks like asset generation and environmental design.

However, a significant challenge remains: because these various AI tools often operate in isolation, developers must integrate the outputs manually. This leads to a lack of **thematic cohesion**, where the generated elements feel disconnected—an effect that occurs "weird" or jarring in a finished game. This project aims to bridge this gap by developing a unified "Game Narrative and Asset Generator." Unlike existing models that focus solely on code, this system explores how a single text input can be used to anchor the theme across multiple outputs, such as sprites and game logic.

To do so, this research paper the following quetions need to be answered first.

* **Thematic Decision Making:** How do generative AI models decide on a "theme" from a text prompt, and what mathematical representations allow them to stay consistent?
* **Cross-Modal Communication:** In what ways can different model types—text, image, and code—communicate thematic data to each other so that the final game follows the specific "vibe" of the original input?

This project asks:

> Do realistic project-based tests show more about AI tool quality than generic benchmark-style claims do?

For this project, "quality" means developer usefulness. A generated quest does not need to be final text. It needs to give a developer something worth keeping, testing, or rewriting.

## 2. Related work

Procedural content generation has long studied how algorithms can generate game content. Recent work connects this field to large language models. Ashby et al. describe an approach to personalized quest and dialogue generation that combines knowledge graphs with language models and evaluates generated RPG content [1]. Vartinen, Hamalainen, and Guckelsberger study GPT-based RPG quest generation and report wide variation in quality, which is directly relevant to this example [2]. Surveys of procedural content generation explain that LLMs are part of a larger history of game-content generation, not a complete replacement for game design [3]. Work on evaluation methodologies for procedural generation also warns against relying on cherry-picked examples [4].

Together, these sources suggest that the important question is not simply whether AI can generate quest text. The better question is how to evaluate whether generated content is useful.

## 3. Method

The imagined Space compares generated quest seeds across structured prompts. Each prompt includes:

- Genre
- Target player experience
- One required mechanic
- One required location
- One required object
- A tone constraint

Example prompt:

> Write a quest seed for a 2D horror game where the player is a lost delivery driver. Include one non-combat mechanic, one map location, and one object that changes meaning by the end.

Each output is scored with a small developer-usefulness rubric:

| Criterion | Question |
|---|---|
| Constraint fit | Did the output follow the required genre, mechanic, object, and tone? |
| Playable mechanic | Does it include an action a player could actually perform? |
| Narrative hook | Is there a conflict, mystery, or choice? |
| Specificity | Does it avoid generic filler? |
| Rewrite burden | How much work would be needed before prototyping? |

## 4. Findings and discussion

The pilot suggests a gap between good-sounding prose and useful design support. Generic prompts produced polished but familiar ideas: a relic, a forest, a missing villager, a final confrontation. These outputs were fluent, but they did not reduce much design work.

More realistic constraints produced more useful outputs. One stronger result described a broken elevator where the player must choose which floor to cut power from in order to escape. The writing was not especially beautiful, but the seed included a mechanic, a location, a choice, and a consequence. That made it more useful than a dramatic paragraph with no playable structure.

The clearest finding is:

> For game-development support, a generated output should be judged by how much design work it makes possible, not only by how polished the language sounds.

This connects to the larger procedural content generation problem. Generated game content has to fit constraints. It must be playable, not just readable.

Input: Generate a jungle temple escape platformer game, Tmplate = PLatformer
Output: In generates a game where it is in a fixed window. The game has some platforms where players can jump onto and there is traps moving side ways where if players run into it will descrease the lives thay have. On the top left there is a UI showing how much lives and scores players have. There is also a target point where when player run into it it will destroy itself, spawn again randomly in the map, and add one score for the player.



My Own Section:

Input: Format-> Top Down Shoorter, Theme-> Alien desert invasion
Output:
![Uploading Screenshot 2026-05-02 at 2.03.58 PM.png…]()




## 5. Limitations

This is a small paper-lite example, not a full study. The imagined pilot uses a few prompts and a hand-built rubric. The scores come from one evaluator, so they reflect one person's sense of usability. A stronger version would ask several game developers or students to rate outputs independently and compare agreement.

The Space also focuses only on quest seeds. It does not test level layouts, combat systems, art assets, pacing, or long-term narrative consistency. A quest seed that looks good in isolation might still fail inside a full game.

## 6. Conclusion

Quest Seed Usability Lab shows how a student can turn broad interest in generative AI into a focused research question. The project does not ask whether AI is creative in general. It asks whether generated quest ideas help a developer move from blank page to prototype. That narrower question is more useful because it matches the real task. The main lesson is that AI game-writing tools should be tested with realistic project constraints, because those constraints reveal whether an output is merely fluent or actually helpful.

## Candidate references

[1] [Personalized Quest and Dialogue Generation in Role-Playing Games: A Knowledge Graph- and Language Model-based Approach](https://consensus.app/papers/details/0551b1b0b6b45be7b6deec007ad77ed7/?utm_source=chatgpt). Trevor Ashby, Braden K. Webb, Gregory Knapp, Jackson Searle, and Nancy Fulda, 2023, *Proceedings of the 2023 CHI Conference on Human Factors in Computing Systems*, citation count: 67.

[2] [Generating Role-Playing Game Quests With GPT Language Models](https://consensus.app/papers/details/8fe8aef4fcf155e8b719d5a26178040c/?utm_source=chatgpt). Susanna Vartinen, Perttu Hamalainen, and C. Guckelsberger, 2024, *IEEE Transactions on Games*, citation count: 66.

[3] [Procedural Content Generation in Games: A Survey with Insights on Emerging LLM Integration](https://consensus.app/papers/details/57dd23d56b32552dbbef0283f265b18f/?utm_source=chatgpt). Mahdi Farrokhi Maleki and Richard Zhao, 2024, venue not listed in Consensus output, citation count: 19.

[4] [Expanding Expressive Range: Evaluation Methodologies for Procedural Content Generation](https://consensus.app/papers/details/61b9fd4b5a795b699677585dbca22b1f/?utm_source=chatgpt). A. Summerville, 2018, venue not listed in Consensus output, citation count: 51.
