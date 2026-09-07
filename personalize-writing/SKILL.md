---
name: personalize-writing
description: Write or rewrite technical articles, blog posts, guides, comments, and other prose in a practical, conversational voice. Use whenever writing or rewording technical content where tone, word choice, sentence construction, rhythm, transitions, punctuation, or local formatting should match this style.
---

# Personalize writing

Apply this skill only to wording, tone, sentence construction, rhythm, transitions, punctuation, and local formatting.

## Voice

- Write like an engineer explaining the subject aloud to another engineer. Keep the prose practical and technically precise, but make it sound like a conversation rather than a presentation of definitions.
- Make inclusive `we` the usual grammatical subject for explanations. Use it when describing what can be done, what needs to happen, how something works, what the code gives us, and what happens next.
- Use only `we`, `us`, and `our` for a shared writer-and-reader perspective. Do not use `I`, reader-facing `you`, exclusive editorial `we`, or reader-facing commands.
- Use direct factual statements occasionally for rhythm, but do not let technical terms, identifiers, or language names become the subject of most explanatory sentences.

Rewrite impersonal explanatory sentences through the shared perspective when it sounds natural:

- `TypeScript can produce this type` becomes `With TypeScript, we can get this type`.
- `The key is to treat the pattern as a small language` becomes `We can treat the pattern as a small language`.
- `A recursive conditional type can split the pattern` becomes `Then we can use a recursive conditional type to split the pattern`.
- `The pattern must remain a string literal` becomes `We need to keep the pattern as a string literal`.
- `This technique works well for small grammars` becomes `We can use this technique for small grammars`.

## Language and rhythm

- Use common words, contractions, and exact technical terms. Prefer `so`, `but`, `then`, and `instead` over formal connectors such as `therefore`, `thus`, and `consequently`.
- Use natural cues such as `Here`, `Now`, `One catch is...`, `Luckily`, `Unfortunately`, `Generally`, `such as`, `like`, and `etc.` when they fit. Do not add them mechanically.
- Keep most sentences short or medium length. Use a longer sentence when a cause, condition, contrast, or consequence belongs with the point.
- Phrase supplied behavior, causes, consequences, limitations, and opinions with concrete verbs. Avoid abstract guarantees, detached importance claims, slogan-like contrasts, paired benefits, and promotional wording.
- Phrase an existing limitation in the same conversational voice. Forms such as `One catch is...`, `But...`, and `Here, we can't...` usually sound more natural than a detached statement.

## Technical explanations

- Describe code through what `we` are doing, looking for, or getting. Prefer `Here, we check the segment and get the matching object` to ``SegmentParams` recognizes four kinds of segments`.
- When introducing an operator, type, function, or API, connect it to the current action. Prefer `Then we can use a conditional type to split the pattern` to `A conditional type can split the pattern`.
- Use the exact identifier when needed for clarity, but do not build a sequence of sentences where every identifier is followed by a definition.
- Prefer `we can`, `we need`, `we get`, `we still have`, `then we`, `here we`, and `now we` to detached capability or result statements.

## Punctuation and local formatting

- Use parentheses for quick examples, platform details, qualifications, and short asides. Do not use em dashes.
- Use a colon to introduce code, a command, a list, or an example.
- When a long sentence contains several parallel items, cases, conditions, steps, or tradeoffs, split them into bullet points so each item is easy to scan. Keep connected reasoning in prose, and keep the bullet items concise and grammatically parallel.

## Style check

Read the text aloud. For each sentence that explains an action, capability, mechanism, or result, check whether an inclusive `we` construction would sound more natural. Rewrite any passage that still reads like code followed by definitions, even if `we` appears elsewhere.
