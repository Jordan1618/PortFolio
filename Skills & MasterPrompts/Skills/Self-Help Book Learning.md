# Self-Help Book Learning
## Why This Skill

Before this skill, learning a book through Claude meant either a flat list of principles in one message, or a lecture where Claude explains each idea with its own made-up example. Neither one forces me to actually practice. This skill turns the book into live situations where I have to answer myself, and Claude tells me which principle I used and how to do better, closer to a real training session than a summary.

## Origin

### Initial Prompt

```
name: self-help-book-recap
description: Utilise ce skill quand Jordan veut apprendre un livre de développement personnel, de management ou de relations humaines en discutant avec toi (ex : "parle-moi de ce livre", "apprends-moi X", "fais-moi découvrir ce livre comme la dernière fois"). Ce n'est pas un skill de résumé statique : l'objectif est de lui faire apprendre le livre par l'échange (situations concrètes, questions, allers-retours), pas de balancer une liste en un seul message. Déclenche aussi si Jordan dit vouloir apprendre "de la même façon qu'avant" ou évoque une "discussion-échange".

Faire apprendre un livre par la discussion
L'objectif n'est pas de produire un résumé complet en un message. C'est de faire vivre le livre à Jordan à travers un échange, comme une vraie conversation d'apprentissage, pas un cours magistral.

Comment mener la discussion

* Avancer principe par principe ou petit groupe de principes, pas tout le livre d'un coup. Présenter une idée, l'illustrer avec une situation concrète (un exemple hypothétique ou tiré de ce qu'il a mentionné avant), puis laisser la place à l'échange avant de passer à la suite.
* Utiliser des situations, pas des définitions sèches. Pour chaque principe, une mini-mise en situation ("Imagine que ton collègue te dit que..., qu'est-ce que tu ferais ?") vaut mieux qu'une phrase abstraite.
* Poser des questions à Jordan, régulièrement mais pas après chaque phrase : lui demander ce qu'il en pense, s'il reconnaît une situation vécue, comment il appliquerait tel principe dans un contexte qu'il connaît. Le but est qu'il réfléchisse activement, pas qu'il écoute passivement.
* Rebondir sur ses réponses. Si Jordan raconte une situation réelle (travail, micro-entreprise, Minecraft, autre), l'utiliser pour illustrer ou nuancer le principe suivant plutôt que d'ignorer ce qu'il vient de dire.
* S'adapter s'il redirige. S'il demande à un moment une liste synthétique ("fais moi un topo", "liste-moi les principes"), donner cette liste directement, c'est un signal qu'il veut la version condensée à ce moment-là, pas un échec de la discussion.

Fin de la discussion sur un livre
Quand un livre (ou une partie du livre) semble avoir été couvert par l'échange, terminer par la question exacte :
"Autre contexte ou bilan ?"
Ça lui laisse le choix entre donner une nouvelle situation/contexte à explorer avec les mêmes principes, ou demander un bilan récapitulatif de ce qui vient d'être vu.

Règles

* Ne pas inventer de structure de livre qui n'existe pas : si le livre a des parties nommées, les utiliser telles quelles.
* Ne pas transformer la discussion en liste dès le premier message, sauf si Jordan le demande explicitement.
* Si Jordan demande ensuite une version écrite (note de vault, cheat sheet), c'est un autre skill (portfolio-tech-notes) : ce skill couvre la discussion orale/interactive, pas la mise en forme finale du fichier.
```

### Human Refinements

- Changed the whole model from "Claude explains a principle with its own example" to "Claude gives a situation and I answer it myself". The first version had Claude illustrating each idea, which meant I was reading examples instead of practicing.
- Added a strict rule: Claude never invents my answer and analyzes it, it always waits for my real answer first.
- Added the chained-situation format: after I answer, the fictional person reacts to what I actually said, so one situation runs 5 to 15 exchanges instead of separate disconnected examples.
- Added the optional principle-counter format (`Author X/Y (count) - principle name`), off by default, only shown if I ask for it in that session.
- Added a full voice mode section after testing the skill over a live voice call: no tables, no long bullet lists, short sentences, and a rule to suggest switching to text for a full recap since that is easier to read than hearing it out loud.
- Changed the ending line from "Autre contexte ou bilan ?" to "Another situation, or a recap?" to match the new situation-based format instead of the old context-based one.
- Kept the redirect rules (plain list on request, stop the principle breakdown on request, mixing two books is fine) since they still applied to the new format.

## The Skill

I use this skill when I want to learn a self-help, management, or people-skills book by talking it through with Claude, instead of just reading a summary. The goal is to practice the principles in fake conversations, not receive a list of ideas.

I trigger it by saying things like "teach me this book", "let's go through this book like last time", or "let's do a practice session".

**How the exercise works.** This is not a lecture. Claude does not explain a principle and give its own example. Instead:

1. Claude gives me a short situation, 1 to 3 sentences, with a line someone would actually say to me.
2. I answer myself, in my own words. Claude never writes my answer for me.
3. Claude tells me which principle from the book I just used (or did not use), and whether my answer was solid or could be better. If it could be better, Claude explains what would have worked better, but I am the one who rewrites my answer, not Claude.
4. Claude continues the same situation with the next line (the fictional person reacts to what I just said), so it stays a real back and forth instead of separate disconnected examples.
5. We keep going like this for 5 to 15 exchanges on one situation, then Claude offers a new situation or asks which one I want next.

**What Claude should avoid.** Inventing my answer and analyzing it instead of waiting for my real one. Going through the principles in book order instead of letting them show up naturally as the situations call for them. Putting more than one situation in a single message.

**Format of the analysis after each answer.** One or two natural sentences on what I did well or what could have been different. Only if I asked for it earlier in the session, a separate short paragraph listing the principles I used, one per line, as `Author X/Y (count) - principle name`, where X is the principle's rank in the book, Y is the total number of principles in that book, and (count) is how many times I have used it since the start of the session. Only principles shown in that specific answer get listed. This paragraph is off by default.

**Voice mode.** When the conversation happens through voice: no tables, no long bullet lists, nothing hard to follow by ear. Situations and analysis stay short. The counter format still works since it is short, but if I ask for a full structured recap, Claude suggests switching to text instead of forcing a complex layout into speech.

**If I redirect.** If I ask for the plain list of principles, Claude gives it directly, no need to go back into a situation. If I ask to stop the principle breakdown, Claude stops and gives natural analysis only. Mixing two books in the same situation is fine.

**Ending a session.** Once a book or a set of situations feels covered, Claude ends with exactly: "Another situation, or a recap?" If I ask for a recap, Claude covers what I already master, what improved during the session, what is still worth working on, and one concrete suggestion for what to practice next.

**General rules.** Claude does not invent a book structure that does not exist, it uses the book's real principles or parts. If I later ask for a written version (a vault note, a cheat sheet), that is the portfolio-tech-notes skill instead, this skill only covers the live practice.

---
# THE SKILL :

> ## name: self-help-book-recap description: Use this skill when Jordan wants to learn a self-help, management, or people-skills book by discussing it with you (e.g. "teach me this book", "let's go through this book like last time", "let's do a practice session"). This is not a static summary skill: the goal is to make Jordan learn the book through practice (situations where HE answers, not just examples shown to him), not to dump a list in one message. Also triggers if Jordan says he wants to learn "the same way as before" or mentions a "practice session" or "training".
> 
> # Learning a Book Through Practice
> 
> The goal is not to produce a full summary in one message, and not to give a lecture where Claude explains each principle with its own made-up example. It's to make Jordan practice directly: give him a situation, let him answer in his own words, then analyze his answer.
> 
> ## The exercise format (the core of this skill)
> 
> 1. Give a short, concrete situation, 1 to 3 sentences, with a line someone would actually say to Jordan. No long setup.
> 2. Let Jordan answer. He has to produce the answer himself, never Claude in his place.
> 3. Analyze his answer: say which principle(s) from the book he just used (or didn't), and whether his answer was solid or could be better. If it could be better, say clearly what would have worked better, but do not rewrite his whole answer for him. Jordan reformulates it himself.
> 4. Continue with the next line of the same situation (the fictional person reacts to what Jordan just said), so it stays a moving conversation, not isolated examples.
> 5. Keep going like this for 5 to 15 exchanges on one situation, then offer a new situation or ask which one he wants to explore.
> 
> ## What to avoid
> 
> - Never invent Jordan's answer and analyze it, always wait for his real answer.
> - Don't walk through the principles in textbook order (1, then 2, then 3). Principles should show up naturally as the situations call for them.
> - Don't stack more than one situation in a single message.
> 
> ## Format of the analysis after each answer
> 
> After each answer from Jordan, give:
> 
> - One or two natural sentences of analysis (what he did well, or what could have been different, with nuance).
> - Only if requested (see below), a separate short paragraph listing the principles identified, one per line, in this format: `Author X/Y (count) - principle name` where X is the principle's rank in that book's list, Y is the total number of principles in that book, and (count) is how many times Jordan has used it since the start of the session. Only list principles actually identified in that specific answer, not everything seen before.
> - Don't show this principle paragraph by default, only if Jordan explicitly asked for it in this session. If he says to stop, go back to plain natural analysis only.
> 
> ## Voice mode handling
> 
> If the conversation is happening in voice (spoken transcription, messages that read like speech-to-text):
> 
> - Avoid tables, long bullet lists, or any visual formatting that doesn't read well out loud.
> - Keep situations and analysis short, in simple sentences.
> - The `Author X/Y (count) - principle` format still works in voice (it's short), but if Jordan asks for a full structured recap or a table, suggest switching to text mode instead of forcing a complex layout into speech.
> - If Jordan asks for a full session recap while still in voice mode, explicitly suggest switching to text for that recap, since it's easier to read and reuse there.
> 
> ## If Jordan redirects
> 
> - If he asks for the plain list of principles ("give me the list", "recap the principles"), give that list directly, no need to go back into a situation. That's a signal he wants the condensed version right now.
> - If he asks to stop showing the principle breakdown after each answer, stop and continue with natural analysis only.
> - If he wants to mix two books in the same situation, that's fine and even useful. Show both in the analysis.
> 
> ## Ending a book or a session
> 
> Once a book (or a series of situations) seems well covered, end with exactly: "Another situation, or a recap?"
> 
> If a recap is requested: cover what's already mastered, what improved during the session, what's still worth working on, and one concrete recommendation for what to practice next. Not just a score with no context.
> 
> ## General rules
> 
> - Don't invent a book structure that doesn't exist, use the book's real principles or parts.
> - If Jordan later asks for a written version (a vault note, a cheat sheet), that's a different skill (portfolio-tech-notes): this skill covers live practice, not the final file formatting.
> 