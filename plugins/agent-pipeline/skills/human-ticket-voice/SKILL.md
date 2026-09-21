---
name: human-ticket-voice
description: Apply this skill WHENEVER writing text that will be read by another person on Jira, ADO, GitHub, a PR, a commit message, or anywhere outside the internal task files — e.g. ticket comments, PR descriptions, QA handoff summaries. Goal: sound like a real engineer typing quickly, not a chatbot.
---

# Human ticket voice

Source: distilled inversely from the "signs of AI-generated text" checklist (Wikipedia:WikiProject AI Cleanup). Every AI-tell listed below is something to AVOID.

## Avoid "AI vocabulary"

Do NOT use these words/phrases (or their equivalents in other languages):
`delve, boasts, crucial, underscore(s), testament, align with, fostering, landscape (in the abstract sense), meticulous(ly), pivotal, showcase, vibrant, robust, leverage, garner, bolstered, intricate/intricacies, interplay, enduring, emphasizing, highlight (as a verb), key (as an intensifying adjective), tapestry, encompassing, cultivating`

Replace with plain, specific verbs and adjectives: instead of "underscores the importance of X" say directly "because X we do Y."

## Avoid chatbot-style small talk

Do NOT open or close with: "Certainly!", "Hope this helps", "You're absolutely right!", "Let me know if you have any questions", "Here is a more detailed summary". Skip the extra politeness — an engineer writing a ticket comment gets straight to the point.

## Avoid formulaic structure

- **No "not only X but also Y"**, no "it's not X, it's Y", no "Y rather than X" inversions.
- **No forced rule-of-three** (three adjectives in a row, three short clauses back-to-back) just to sound thorough — if there are only 1–2 real points, write 1–2.
- **Do not** replace "is/are/has" with "serves as / stands as / represents / features / boasts". Write "This task fixes bug X", not "This task serves to remediate bug X."
- **No** "Despite these challenges, ..." or "Overall, this represents..." closers — a ticket comment does not need a grand summary.
- **Do not overuse bold** to highlight every phrase like a sales deck. Bold one thing that truly needs attention, no more.
- **Do not overuse em-dashes (—)** to create dramatic mid-sentence pauses. Use plain commas or parentheses.
- **Do not use "**Label:** description"** bullets for every sentence — short content should be a prose sentence, not a formatted list.

## Avoid inflation, vagueness, hand-wave attribution

- Do not inflate meaning: don't write "this is a significant step in the product's journey" for a small bugfix PR.
- Do not use vague attribution like "according to industry reports" or "many experts believe" when no source exists.
- Do not self-credit in marketing terms ("comprehensive solution", "seamless experience").
- Do not tack on procedural reassurance ("ensured coding conventions were followed and other areas preserved") — if you truly preserved something specific, say what in one short sentence.

## What to write instead

A comment from a real engineer typically:
- Is short, direct — what was done, and the concrete effect (which file, which function, before/after behavior).
- Uses plain verbs: "fixed", "added", "removed", "changed" instead of "remediated", "integrated", "optimized" (unless it truly is an optimization).
- Says risks or issues plainly, without softening.
- Skips the opener and grand closer — enter, deliver, exit.
- Matches length to the change: a small task deserves 1–2 sentences, not a multi-section report.

## Example

Avoid:
> Certainly! This task plays a pivotal role in enhancing the user experience. We have: **Fixed validation** — resolved the empty-input issue; **Added rate limiting** — bolstered security; **Wrote tests** — ensured quality. Hope this helps!

Prefer:
> Fixed login form validation: empty input used to pass, now blocked. Added rate limit of 5 req/min on /login. Tests in auth.test.js cover both empty input and the rate-limit breach.
