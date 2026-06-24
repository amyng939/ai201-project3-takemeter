# TakeMeter

A fine-tuned text classifier that evaluates discourse quality in r/tifu (Today I Fucked Up), a subreddit where people share stories of their own mistakes. TakeMeter classifies comments into four categories — reaction, joke, advice, and question — based on what the comment is primarily doing in response to the original post.

---

## Community Choice and Reasoning

**Community:** r/tifu is a good fit for this task because comments respond to the same story in genuinely different ways. Some people react emotionally, some make jokes, some give advice on what OP should have done, and some ask genuine follow-up questions. These distinctions are real and recognizable — a regular r/tifu reader would immediately identify them — which makes them learnable labels rather than arbitrary categories.

### Labels:
**Reaction** — an emotional or appreciative response to the story that expresses how the commenter feels without offering judgment, advice, or humor as the primary intent.
- This has to be the best TIFU post I've ever read. I'm honestly in tears
- All I could imagine was your wife's face after you said "I am really a clutz.. Right honey?" Got me in tears. Brilliant story sir.

**Joke** — a comment whose entire purpose is humor, wordplay, or absurdist riffing on the story, where removing the comedic element leaves nothing of substance.
- It's times like this when you should just fake a seizure
- This has to be an episode of Seinfeld.

**Advice** — a comment that tells the OP what they should have done, could do now, or offers practical information relevant to their situation.
- All you had to do was ask if she could cook it for a little longer. There's nothing wrong with that at all. But hey look on the bright side you'll have a funny story to tell to all your friends for years to come.
- The hostess is a moron for not asking how you liked your steak, but you're an even bigger moron for not bringing it up when it came to your plate. Instead of asking to cook it a little longer, enjoy your meal, make a good impression and not look like a dick, you have achieved the complete opposite. I'm curious to know what you'd do in the same situation at a restaurant.

**Question** — a comment that genuinely asks the OP for more information, clarification, or an update, rather than using a question rhetorically to make a point.
- One question OP.... Let's say the window was open and the steak is gone. She returns a minute later... What you just tell her you devoured it?
- What exactly was your endgame here? Were you planning to sneak back onto her property late at night to remove the steak? Or were you just hoping that she'd see it and think she got a visit from the Steak Fairy?

**Data source:** All 200 examples are drawn from a single post: ["TIFU by throwing my steak out a window"](https://www.reddit.com/r/tifu/comments/3im341/tifu_by_throwing_my_steak_out_a_window/). Using one post controls for topic and community context, so label differences are attributable to comment type rather than subject matter.

**Labeling Process:** I looked through comments from the post and manually labeled each one. Whenever I was stuck, I used Claude to stay consistent.

### Hard Edge Cases

The hardest boundary is **reaction vs. joke**. Many comments are witty or phrased humorously while still being primarily an emotional response to the story. Decision rule: a comment is only joke if the humor is the entire substance — removing the pun, absurdist riff, or pop culture reference leaves nothing behind. Username jokes and steak puns are always joke regardless of how reaction-like they read.

The second hard boundary is **reaction vs. advice**. Comments that criticize what OP did while implying they should have done otherwise can read as either. Decision rule: if the comment tells OP what to do or should have done with enough specificity to be actionable, it is advice. If it expresses disbelief or judgment without a concrete alternative, it is reaction.

Rhetorical questions that exist to express disbelief rather than genuinely request information are labeled reaction, not question.

**Examples:**
- No way. No one could be that dumb. You have to be Peter Griffin or some sort of sitcom character. What did you plan on saying after you threw the steak out the window and just sat there with a suddenly empty plate?

Between Question and Reaction I chose Question because the user is expressing confusion.

- Relevant fuckin username if i ever saw one

A lot of comments had this and I was confused between Joke or Reaction but chose Joke because it's creating humor from OP's username and is not a reaction to the story.

- My wife's only two words to me since the incident are "I'm fine". Wow...that is really bad. Sleep with one eye open from now on.

Between Joke and Reaction I chose Reaction because there is more to it than the humor.

---

## Dataset

| Label    | Count | % of total |
|----------|-------|------------|
| reaction | 117   | 58.5%      |
| joke     | 45    | 22.5%      |
| question | 25    | 12.5%      |
| advice   | 13    | 6.5%       |
| **Total**| **200** | |

The dataset is heavily skewed toward reaction, which is expected for a humor-driven subreddit. Advice is underrepresented at only 13 examples, which directly affected model performance on that class.

**Train / val / test split:** 70% / 15% / 15% (140 / 30 / 30), stratified by label.

---

## Fine Tuning Approach

**Base model:** `distilbert-base-uncased` fine-tuned with a classification head for 4 labels.

**Hyperparameters:**
- Epochs: 8
- Learning rate: 5e-5
- Batch size: 16
- Weight decay: 0.01
- Warmup steps: 50
- Best model selected by validation accuracy

The default learning rate of 2e-5 caused majority-class collapse on the first run — the model predicted reaction for every comment and achieved 60% accuracy by doing nothing. Increasing to 5e-5 and 8 epochs allowed the model to start learning minority classes, though advice remained unlearnable due to insufficient training examples.

## Baseline Description

**Prompt:**
```
SYSTEM_PROMPT = """
You are classifying comments from r/tifu (Today I Fucked Up), a subreddit
where people share stories of their own mistakes.
Assign each comment to exactly one of the following categories.

Reaction: an emotional or appreciative response to the story that expresses
how the commenter feels without offering judgment, advice, or humor as the
primary intent.
Example: "All I could imagine was your wife's face after you said 'I am
really a clutz.. Right honey?' Got me in tears. Brilliant story sir."

Joke: a comment whose entire purpose is humor, wordplay, or absurdist
riffing on the story, where removing the comedic element leaves nothing
of substance.
Example: "This has to be an episode of Seinfeld."

Advice: a comment that tells the OP what they should have done, could do
now, or offers practical information relevant to their situation.
Example: "All you had to do was ask if she could cook it for a little
longer. There's nothing wrong with that at all."

Question: a comment that genuinely asks the OP for more information,
clarification, or an update, rather than using a question rhetorically
to make a point.
Example: "What exactly was your endgame here? Were you planning to sneak
back onto her property late at night to remove the steak? Or were you
just hoping that she'd see it and think she got a visit from the Steak
Fairy?"

Important rules:
- If a comment contains questions but they are rhetorical or the commenter
  answers them themselves, label it Reaction not Question.
- If a comment is witty or funny but is primarily responding to the story
  emotionally, label it Reaction not Joke.
- If a comment implies OP should have done something differently but frames
  it as disbelief rather than a concrete actionable suggestion, label it
  Reaction not Advice.
- Username jokes and puns on the story are always Joke.

Respond with ONLY the label name. 
Do not explain your reasoning.

Valid labels:
reaction
joke
advice
question
"""
```
---

## Evaluation Report

### Overall Results

| Model                        | Accuracy | Macro F1 |
|------------------------------|----------|----------|
| Zero-shot baseline (Groq llama-3.3-70b) | 0.867 | 0.74 |
| Fine-tuned DistilBERT        | 0.700    | 0.52     |

The zero-shot baseline outperforms the fine-tuned model by 16.7 percentage points. This is expected given the data constraints: Llama-3.3-70b has billions of parameters pretrained on vast amounts of internet text including Reddit. Fine-tuning DistilBERT (66M parameters) on 140 examples cannot overcome that head start, particularly for minority classes with fewer than 20 training examples.

### Per-Class Metrics — Fine-Tuned Model

| Label    | Precision | Recall | F1   | Support |
|----------|-----------|--------|------|---------|
| reaction | 0.76      | 0.89   | 0.82 | 18      |
| joke     | 0.50      | 0.43   | 0.46 | 7       |
| advice   | 0.00      | 0.00   | 0.00 | 2       |
| question | 1.00      | 0.67   | 0.80 | 3       |
| **macro avg** | 0.57 | 0.50 | 0.52 | 30   |

### Per-Class Metrics — Zero-Shot Baseline

| Label    | Precision | Recall | F1   | Support |
|----------|-----------|--------|------|---------|
| reaction | 0.90      | 1.00   | 0.95 | 18      |
| joke     | 1.00      | 0.71   | 0.83 | 7       |
| advice   | 0.50      | 1.00   | 0.67 | 2       |
| question | 1.00      | 0.33   | 0.50 | 3       |
| **macro avg** | 0.85 | 0.76 | 0.74 | 30   |

### Confusion Matrix — Fine-Tuned Model

|              | predicted: reaction | predicted: joke | predicted: advice | predicted: question |
|--------------|--------------------:|----------------:|------------------:|--------------------:|
| **true: reaction** | 16            | 1               | 1                 | 0                   |
| **true: joke**     | 4             | 3               | 0                 | 0                   |
| **true: advice**   | 1             | 1               | 0                 | 0                   |
| **true: question** | 0             | 1               | 0                 | 2                   |

The dominant error pattern is **joke → reaction**: 4 of 7 joke comments were predicted as reaction. This is the hardest boundary in the taxonomy and the one most dependent on cultural context (steak puns, username jokes, pop culture references) that a small fine-tuned model struggles to generalize from.

### Failure Analysis

**Which labels are being confused?**
The primary confusion is joke → reaction (4 errors). The secondary confusion is advice → reaction and advice → joke (2 errors, covering all advice examples in the test set). The model never correctly identified an advice comment.

**Error #1: reaction predicted as advice (confidence: 0.30)**
> "There's something wrong with you. All you would do is say 'I'm sorry, but I've got a fear of meat...'"

This comment contains a quoted script of what OP should have said, which looks structurally identical to advice. The correct label is reaction because the script is framed as mockery and disbelief rather than genuine guidance. The model latched onto the surface structure (quoted alternative script) rather than the framing around it. This is a boundary the label definition handles correctly but that 9 training examples of advice cannot teach a model to navigate.

**Error #2: joke predicted as reaction (confidence: 0.92)**
> "Relevant fuckin username if i ever saw one"

This is a username joke — the entire comment is a comedic observation about OP's username "defenestrate me now." The model has no way to know the username exists or that it's relevant, because the comment text alone gives no indication of what makes it funny. Without that external context, the comment reads like a mild reaction ("that username is fitting") rather than a punchline. This is a genuine limitation of text-only classification — the humor is only legible if you know the username.

**Error #3: joke predicted as reaction (confidence: 0.97)**
> "You go full Gordon Ramsay and shout WHAT THE FUCK IS THIS? THIS STEAK IS SO RAW ITS EATING THE FUCKING SALAD"

The model predicted this as reaction with high confidence, likely because the all-caps emotional language and exclamation pattern looks like an emotional reaction. The comment is actually an absurdist riff imagining OP shouting a famous Gordon Ramsay quote at the dinner table. The humor depends on recognizing the cultural reference, which a model trained on 45 joke examples from one post is unlikely to have learned. This is a data diversity problem — the training set doesn't have enough varied joke examples to teach the model that cultural references and imagined scenarios are joke signals.

### Sample Classifications

| Text | True Label | Predicted | Confidence |
|------|-----------|-----------|------------|
| "You are hilarious." | reaction | reaction | 0.67 |
| "You could say this was a mis-steak" | joke | joke | 0.81 |
| "What was going to be your explanation had everything worked according to plan? She leaves the room, you haven't taken one bite, she comes back and the entire steak is gone?" | question | question | 0.63 |
| "This fucking story made my day. Thank you moron." | reaction | reaction | 0.98 |
| "She not fine" | joke | reaction | 0.84 |

The joke prediction on "You could say this was a mis-steak" is reasonable — 
the comment is a single steak pun with no emotional substance, and the model 
correctly identified that the wordplay is the entire point. This is one of 
the cleaner joke signals in the dataset: short, no emotional language, and 
the humor is self-contained in the text itself without requiring external 
context like a username or cultural reference.

---

## Reflection: What the Model Captured vs. What I Intended

The model learned a reasonable approximation of **reaction** — high recall (0.89) suggests it correctly identifies most emotional responses. But it learned reaction partly by default: when uncertain, predict reaction, because reaction dominates the training data at 58.5%.

What the model failed to capture is the **intent behind a comment's structure**. The joke/reaction boundary requires understanding that a comment's purpose can be humor even when it reads emotionally — steak puns, username jokes, and absurdist scenarios all require cultural or contextual knowledge that the text alone doesn't provide. The model learned surface features (short text, exclamation marks, emotional language) rather than the communicative intent those features serve.

For advice, the model never learned the label at all. With 9 training examples, this was inevitable. The boundary between advice and reaction is subtle enough that even human annotators needed explicit decision rules — teaching it to a model requires far more examples than this dataset provides.

The intended classifier was one that understood the function of a comment. What emerged was one that learned the most common surface patterns of the dominant class and made uncertain guesses on everything else.

---

## Spec Reflection

**Where the spec helped:** The requirement to define hard edge cases before annotating was the most valuable design constraint. Working through the reaction/joke and reaction/advice boundaries before labeling 200 examples meant those decisions were consistent across the dataset. Without that, annotation inconsistency would have added noise on top of the already-difficult class imbalance problem.

**Where implementation diverged:** The spec assumed a community where "discourse quality" varies in ways that produce a roughly balanced label distribution. r/tifu is a humor subreddit — 58.5% of comments I found from a post are reactions and 6.5% are advice, and that imbalance reflects the community's actual behavior, not a collection failure. In hindsight, collecting from multiple posts or targeting advice comments specifically would have produced a more trainable dataset. The single-post constraint that controlled for topic also constrained the diversity of examples, particularly for minority classes.

---

## AI Usage

**Instance 1:** I pasted individual ambiguous comments into Claude and asked whether they were reaction, joke, advice, or question given my definitions. Claude gave a label and reasoning; I either confirmed or overrode based on my own reading. Examples I overrode: Claude initially leaned toward labeling several short sarcastic comments as reaction; I moved them to joke after verifying the humor was the entire substance. I did not use Claude to pre-label batches — every label was reviewed individually.

**Instance 2:** After collecting wrong predictions, I pasted them into Claude and asked it to identify common themes. Claude identified two patterns: short comments with ambiguous surface features (steak puns, username references) and advice comments with comic framing. I verified both patterns held across the full wrong predictions list and incorporated them into the failure analysis above.