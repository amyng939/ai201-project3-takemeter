## Community: 
<!-- What community did you choose and why? Why is this community a good fit for a classification task — what makes the discourse varied enough to be interesting? -->
I am choosing from reddit r/tifu because it contains good posts that encourage different sorts of discourse that can be categorized into a few labels. There's comments reacting to the story or people mentioning their own takes on things or making jokes.
I am specifically choosing this post from the community: https://www.reddit.com/r/tifu/comments/3im341/tifu_by_throwing_my_steak_out_a_window/

## Labels:
<!-- What are your 2–4 labels? Define each in a complete sentence. Include 2 example posts per label. -->
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

## Hard edge cases:
<!-- What type of post will be genuinely ambiguous between two labels? How will you handle it when you encounter it during annotation? -->
If a comment looks like a mix of the labels like it contains questions but also is reactionary, depending on how it's read like if the questions are rhetorical or the user is genuinely asking, then it'll be reaction or question respectively. Additionally 

**Examples:**
- No way. No one could be that dumb. You have to be Peter Griffin or some sort of sitcom character. What did you plan on saying after you threw the steak out the window and just sat there with a suddenly empty plate?

Between Question and Reaction I chose Question because the user is expressing confusion.

- Relevant fuckin username if i ever saw one

A lot of comments had this and I was confused between Joke or Reaction but chose Joke because it's creating humor from OP's username and is not a reaction to the story.

- My wife's only two words to me since the incident are "I'm fine". Wow...that is really bad. Sleep with one eye open from now on.

Between Joke and Reaction I chose Reaction because there is more to it than the humor.


## Data collection plan:
<!-- Where will you collect examples? How many per label? What will you do if a label is underrepresented after 200 examples? -->
I am specifically choosing this post from the community: https://www.reddit.com/r/tifu/comments/3im341/tifu_by_throwing_my_steak_out_a_window/
After 200 examples I am making sure no labels take up at least 70% of them. I will try to get at least 10 per label because I am aware some of the labels are easier to find in comments than others for a single post. If I had more time or could use multiple posts, I would look for underrrepresented labels elsewhere.

## Evaluation metrics:
<!-- Which metrics will you use to evaluate your model and why are those the right ones for this specific task? (Accuracy alone is not enough — explain what else you need and why.) -->
Primary metric is macro F1, which averages per-class F1 scores without weighting by class frequency. This is appropriate because reaction will dominate the dataset — a model that predicts reaction for everything could achieve high accuracy while learning nothing. Macro F1 penalizes failure on rare classes (advice, question) equally to common ones (reaction, joke). Per-class F1 scores will be reported individually to identify where the model breaks down, particularly on the reaction/joke and reaction/advice boundaries. A confusion matrix will be used for qualitative error analysis.

## Definition of success:
<!-- What performance would make this classifier genuinely useful? What would you accept as "good enough" for deployment in a real community tool? -->
A macro F1 above 0.55 clears the majority-class baseline and indicates the model has learned something real. A macro F1 above 0.70 with no individual class F1 below 0.50 would constitute deployment-ready performance — meaning the classifier is reliable enough to surface to users without systematic blind spots on any label. Given that advice and question may each represent under 10% of the dataset, a class F1 below 0.50 on either should be noted as a known limitation rather than a failure of the model overall.

## AI Tool Plan:
**Label stress-testing:** Give the AI my label definitions and edge case description, and ask it to generate 5–10 posts that sit at the boundary between two labels. If it produces posts I can't classify cleanly, my definitions need tightening — do that now, before I annotate 200 examples.

**Annotation assistance:** I will not use an LLM to pre-label a batch of examples before reviewing them myself.

**Failure analysis:** After evaluation, wrong predictions will be collected and given to an AI tool to identify error patterns — particularly whether the model confuses reaction and joke on witty 
comments, or misclassifies short rhetorical questions as question rather than reaction. Patterns identified by the AI will be verified manually by re-reading the misclassified comments to confirm they hold across multiple examples and are not artifacts of a few edge cases.