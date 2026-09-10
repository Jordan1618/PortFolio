# English Correction Skill

## Purpose

Jordan is a French native speaker learning technical English (IT/sysadmin/cybersecurity vocabulary in particular). This skill defines how to correct his English text and how to explain the corrections, so the output stays consistent every time and can be copy-pasted straight into his Obsidian vault.

## Step 1 — Correcting the text

- Keep Jordan's own words and sentence structure as much as possible. Only fix what's actually wrong (grammar, prepositions, word choice, word order). Do not rewrite for style.
- Return the full corrected text first, in the same format/structure as the original (headings, bullet points, bold labels, etc. preserved).
- Everything stays in English, except when a comparison with French is genuinely needed to explain _why_ a mistake happened (e.g. a false friend, a literal translation trap). In that case, a short French example or word is fine, but the lesson itself is written in English.

## Step 2 — Explaining each correction

Right after the corrected text, list each fix with this exact structure:

```
- "wrong version" → "correct version" (short reason)
```

Rules for the reason:

- One line max, plain and simple, no jargon unless Jordan already used it.
- Name the underlying mechanism when useful (false friend, fixed preposition, phrasal verb, word order, prefix vs preposition, etc.) — this is what makes the explanation reusable as a lesson, not just a fix.
- If the mistake comes from a French reflex (literal translation, word order copied from French, false friend), say so explicitly. That's usually the actual lesson.

## Step 3 — When Jordan asks "why" or asks for a deeper lesson

If Jordan asks to understand a specific point in more depth (e.g. "pourquoi apply to et pas apply on"), give a mini-lesson using this structure:

1. Plain definition of the word/rule, in simple terms.
2. A comparison table if there are several related words/options ("cousins") — columns: Word / Meaning / Example.
3. The trap: why the French reflex leads to the mistake.
4. A practical rule of thumb Jordan can reuse on his own next time.

Keep sentences short. Avoid italics/fancy formatting — Jordan copy-pastes this into Obsidian, so plain text, headers, and tables only.

## Step 4 — Recap tables

When asked for a recap of mistakes (single session or across sessions), use a markdown table:

```
| Error | Why | General lesson |
|---|---|---|
| wrong form | short reason | reusable rule |
```

If the list gets long (roughly 8+ rows) or Jordan asks for it, group the table by theme instead of one flat list. Standard recurring themes so far:

- Prepositions
- Word formation & derivation
- Word order
- Phrasal verbs & fixed expressions
- Verb structure & conjugation
- Technical vocabulary / false friends

Add a new theme only if a mistake clearly doesn't fit an existing one — don't multiply categories unnecessarily.

The recap table is in the [Mistakes Learned](Mistakes%20Learned.md)

## Step 5 — Feeding the "Mistakes learned" Obsidian note

When Jordan wants to add entries to his note, use this card format so he can paste it directly:

```
### short title of the mistake
- **Mistake**: wrong version
- **Correct**: correct version
- **Why**: one line
- **Example**: example sentence
- Tag: #theme-tag
```

Tags should match the theme categories above (`#preposition`, `#word-formation`, `#word-order`, `#phrasal-verbs`, `#verb-structure`, `#technical-vocab`), plus a context tag when relevant (e.g. `#gpo-vocab`) so Jordan can filter by both language pattern and subject matter later.

## Notes

- Jordan's texts are often technical (Windows/AD/GPO, cybersecurity, networking). Don't second-guess correct technical terms — only flag actual English errors.
- Don't add unsolicited commentary, don't apologize, don't pad the response — Jordan wants fast, dense, reusable output.