---
name: self-help-book-recap
description: Use this skill when Jordan wants to learn a self-help, management, or people-skills book by discussing it with you (e.g. "teach me this book", "let's go through this book like last time", "let's do a practice session"). This is not a static summary skill: the goal is to make Jordan learn the book through practice (situations where HE answers, not just examples shown to him), not to dump a list in one message. Also triggers if Jordan says he wants to learn "the same way as before" or mentions a "practice session" or "training".
---

# Learning a Book Through Practice

The goal is not to produce a full summary in one message, and not to give a lecture where Claude explains each principle with its own made-up example. It's to make Jordan practice directly: give him a situation, let him answer in his own words, then analyze his answer.

## The exercise format (the core of this skill)

1. Give a short, concrete situation, 1 to 3 sentences, with a line someone would actually say to Jordan. No long setup.
2. Let Jordan answer. He has to produce the answer himself, never Claude in his place.
3. Analyze his answer: say which principle(s) from the book he just used (or didn't), and whether his answer was solid or could be better. If it could be better, say clearly what would have worked better, but do not rewrite his whole answer for him. Jordan reformulates it himself.
4. Continue with the next line of the same situation (the fictional person reacts to what Jordan just said), so it stays a moving conversation, not isolated examples.
5. Keep going like this for 5 to 15 exchanges on one situation, then offer a new situation or ask which one he wants to explore.

## What to avoid

- Never invent Jordan's answer and analyze it, always wait for his real answer.
- Don't walk through the principles in textbook order (1, then 2, then 3). Principles should show up naturally as the situations call for them.
- Don't stack more than one situation in a single message.

## Format of the analysis after each answer

After each answer from Jordan, give:

- One or two natural sentences of analysis (what he did well, or what could have been different, with nuance).
- Only if requested (see below), a separate short paragraph listing the principles identified, one per line, in this format: `Author X/Y (count) - principle name` where X is the principle's rank in that book's list, Y is the total number of principles in that book, and (count) is how many times Jordan has used it since the start of the session. Only list principles actually identified in that specific answer, not everything seen before.
- Don't show this principle paragraph by default, only if Jordan explicitly asked for it in this session. If he says to stop, go back to plain natural analysis only.

## Voice mode handling

If the conversation is happening in voice (spoken transcription, messages that read like speech-to-text):

- Avoid tables, long bullet lists, or any visual formatting that doesn't read well out loud.
- Keep situations and analysis short, in simple sentences.
- The `Author X/Y (count) - principle` format still works in voice (it's short), but if Jordan asks for a full structured recap or a table, suggest switching to text mode instead of forcing a complex layout into speech.
- If Jordan asks for a full session recap while still in voice mode, explicitly suggest switching to text for that recap, since it's easier to read and reuse there.

## If Jordan redirects

- If he asks for the plain list of principles ("give me the list", "recap the principles"), give that list directly, no need to go back into a situation. That's a signal he wants the condensed version right now.
- If he asks to stop showing the principle breakdown after each answer, stop and continue with natural analysis only.
- If he wants to mix two books in the same situation, that's fine and even useful. Show both in the analysis.

## Ending a book or a session

Once a book (or a series of situations) seems well covered, end with exactly: "Another situation, or a recap?"

If a recap is requested: cover what's already mastered, what improved during the session, what's still worth working on, and one concrete recommendation for what to practice next. Not just a score with no context.

## General rules

- Don't invent a book structure that doesn't exist, use the book's real principles or parts.
- If Jordan later asks for a written version (a vault note, a cheat sheet), that's a different skill (portfolio-tech-notes): this skill covers live practice, not the final file formatting.
