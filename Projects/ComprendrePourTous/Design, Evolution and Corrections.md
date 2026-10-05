# **1) What I wanted to do :**

I wanted to turn my long guides (body, emotions, couple) into a real public website, and to keep it maintainable by one person. This note is the story of how it was built and what I corrected on the way. The dates come from the Git history (212 commits) and from the request log I keep in the repository.

# **2) 2 to 6 August : the content, then the website in five attempts :**

The project starts as Markdown guides imported into a repository (the female cycle, male emotional health, STIs, massage, questions).

- **Hugo with the hugo-book theme**, published on GitHub Pages. The site was named "Apollon" for a moment, then I went back to "Comprendre pour tous".
- **Bug:** the CSS was broken. The real cause was that Hugo's relative URLs generated invalid paths, not a CSS error.
- **Custom domain:** a `CNAME` file, then the move to `www.comprendrepourtous.fr`.
- **My own design in HTML and CSS** instead of the theme.
- **My own generator in Python** instead of Hugo, with no dependency. The reason: full control of the output, nothing to update, and everything understood down to the wiring.

The same day I wrote three guides on the relationship (La rencontre, L'amour, Pour Nous), added the legal pages (legal notice, privacy, license) and one color per guide, then Les émotions, a contact page and a feedback block. A design backup folder kept the first version, and I removed it later.

# **3) 7 August : sources, symmetry and cleaning :**

- **Pour Elle and Pour Lui made symmetrical**, same skeleton of chapters, missing chapters added (male contraception, male sexuality, female depression and anxiety).
- **A Sources section** with one page per guide, each source with a link that resolves.
- **Notions** created with links placed from the guides.
- **Maintainer notes taken out of the public content** into an unpublished folder.
- **The Pages source switched to GitHub Actions**, because the old setting served the repository root and broke the site.
- **The first big sourcing job:** one real scientific source for every sub-part, guide by guide.

# **4) 10 to 14 August : reviews, length, hyperlinks, the first rules files :**

- **Public reviews** (stars and comment) with moderation, protected from robots (see the architecture in the README). I asked Claude to explain every concept first (honeypot, Turnstile, Supabase) before building it.
- **Chapter length.** I set a floor of 2,000 words for narrative chapters (it became 1,500 to 3,500 words on 16 September). On 11 August alone there were 55 commits: dozens of chapters of Pour Elle and Pour Lui densified (for example 678 to 1,905 words) and a new series of relational chapters.
- **An opening note** on every guide, then a warning banner, saying the text gives general landmarks, written by an IT student, and not individual advice.
- **Sources as hyperlinks.** The link goes on the sentence it supports, never a bare "(source: ...)" tag.
- **A new guide on social networks**, and a new guide on blended families, later renamed "Les nouvelles compositions familiales" to show that the models are much more varied than the classic one.
- **ANALYSE.md**, an independent reliability report, and the "Is it reliable?" section on the home page.
- **CLAUDE.md and the request log** (see the Claude note).

# **5) 15 to 16 September : search engines and honesty page :**

- Fixes after seeing how Google displayed the site (see the SEO note), structured data, and an "About" page that stresses honesty, rigor, free access and no sales pitch.
- **Gender neutrality** of the reader in Pour Lui ("degendering"), written into the skill.
- **Real testimonies** rule, written into the skill.
- **Massage professionnel** grows from 12 to 21 chapters.

# **6) 17 to 23 September : every short guide is rebuilt :**

Each guide followed the same method: full elicitation in the conversation (grid of 10 families, existing angles, overlap question, two examples of 30 sub-topics), then everything is written.
- Réseaux sociaux: 10 more chapters (economy, geopolitics, specific populations)
- Les nouvelles compositions familiales: 4 to 31 chapters
- Questions et communication: 23 to 46 chapters
- Pour Nous: 11 to 25 chapters
- La rencontre: 9 to 28 chapters
- L'amour: 9 to 28 chapters
- Pour Elle and Pour Lui: the last 8 missing symmetrical chapters
- **Four new guides:** Alimentation, Le sommeil, Maladie grave et handicap, Psychologie de la personnalité
- A permanent rule I set on 22 September: duplicated content between chapters is fine, but sources and notions must never be duplicated.

# **7) 23 to 28 September : the big correction :**

- **My critique (23 September):** "Your guides only quote studies and never explain them." The pilot chapter on the Big Five model never defined the five traits, and the "nine dimensions of temperament" chapter never listed the nine. Sources talked about the model, not the model itself, and the writing followed the sources instead of the reader.
- **The pilot chapter was rewritten**, then corrected again after a first version I judged insufficient even though the format was right.
- **The method was written into skills:** Redaction2Chapitre (how to write a chapter) and Audit2Guide (a read-only audit).
- **A mass audit of the 15 guides (427 chapters)** with parallel read-only agents. Each chapter got a verdict: nothing to do, surgery (add what is missing) or rewrite.
- **Guide-by-guide reprise,** without waiting for my approval between guides, only reporting at the end of each one. Examples: Pour Nous went from 36,091 to 44,594 words and its source reciprocity went from 48 missing links to 0 out of 128. Psychologie de la personnalité went from 33,171 to 49,821 words, with 142 out of 142 links matched.
- **A second full audit** after the work showed that Pour Nous had only been reworked on its first 11 chapters, so the remaining 14 were rewritten on 28 September.

# **8) 28 September to 1 October : the reading experience :**

- A share image (Open Graph), simplified so it does not carry a number that changes with each new guide
- Colors alternating on the home page, and a fix for cards invisible in dark mode
- The warning banner moved to the bottom of each guide as a discreet note, because I found a long block at the top ugly
- Punchy introductions on each guide page instead of a "Where to start" block
- **A site that did not change.** I asked for colors on the home page and saw no difference. The cause was not the CSS: GitHub Pages deploys only from `main`, and the 26 commits of my work were sitting on a work branch that was never merged. I asked to merge directly on `main`. Another session had meanwhile pushed its own version of the warning banner move, so `build.py` had a conflict, and I kept one single implementation to avoid showing the banner twice.

# **9) The corrections I keep as rules :**

- **Typography:** no italics, no em dashes
- **Numbering:** additions were numbered "X.Y bis" so internal references never broke, until the final renumbering of each guide from 1 to N without letters
- **Renaming:** always add the old URL to the redirect table
- **Checks after each addition:** regenerate the indexes and the full guides, build the site (it reports internal links that point nowhere), check source reciprocity, and check that neighbouring guides cite the new one
- **Never ignore an instruction because it is "obvious":** twice Claude started writing without the elicitation questions, so the rule was hardened in CLAUDE.md and in the skill

# **10) What I learned :**

- Replacing a tool by my own code is worth it when I can explain every line and the output is simple
- A bug that looks like CSS can be an URL problem
- Rules that cannot be checked are ignored under pressure, so the craft rules became visible blocks that can be counted
- Measuring only the word count gave a dashboard that hid the real problem
- A long project needs generated indexes, a checklist and a portable follow-up file, not memory
