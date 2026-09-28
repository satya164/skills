---
name: personalize-writing
description: Write technical articles, blog posts, guides, comments, and other prose in a practical, conversational voice. Use whenever writing, editing, or rewording technical prose, including new documents and edits to existing ones.
---

# Personalize writing

Apply this skill only to wording, tone, sentence construction, rhythm, transitions, punctuation, and local formatting.

## Voice

- Write like an engineer explaining the subject aloud to another engineer. Keep the prose practical and technically precise, but make it sound like a conversation rather than a presentation of definitions.
- Choose the grammatical person by document type:
  - Tutorials, where the reader follows along: inclusive `we` (`Now we add the route`).
  - How-to guides, instructions, migration guides, and setup steps: reader-facing `you` and the imperative (`Remove the wrapper component`, `You can pass a custom label`).
  - Reference, such as API docs, option descriptions, and changelogs: no pronouns. Describe outcomes in the passive or as definition-list entries (``The request timeout is set by `timeout` ``, `` `timeout`: sets the request timeout``).
  - Explanations, articles, and blog posts: inclusive `we` for reasoning through a problem, `you` when the reader's situation is the point.
- Keep one person per section. Do not use `I`, and do not use editorial `we` for the project outside release notes and announcements; name the project instead.
- When revising or extending an existing document, keep its established person and voice (for example, reader-facing `you` with editorial `we` for the maintainers in a migration guide). Apply the rest of this skill within that voice.
- Where inclusive `we` is the person, make it the usual grammatical subject for explanations. Use it when describing what can be done, what needs to happen, how something works, what the code gives us, and what happens next.
- Use direct factual statements occasionally for rhythm, but do not let technical terms, identifiers, or language names become the subject of most explanatory sentences (see Subjects and agents).

In the inclusive `we` voice, rewrite impersonal explanatory sentences through the shared perspective when it sounds natural:

- `TypeScript can produce this type` becomes `With TypeScript, we can get this type`.
- `The key is to treat the pattern as a small language` becomes `We can treat the pattern as a small language`.
- `A recursive conditional type can split the pattern` becomes `Then we can use a recursive conditional type to split the pattern`.
- `The pattern must remain a string literal` becomes `We need to keep the pattern as a string literal`.
- `This technique works well for small grammars` becomes `We can use this technique for small grammars`.

## Language and rhythm

- Use common words, contractions, and exact technical terms. Prefer `so`, `but`, `then`, and `instead` over formal connectors such as `therefore`, `thus`, and `consequently`.
- Use natural cues such as `Here`, `Now`, `One catch is...`, `Luckily`, `Unfortunately`, `Generally`, `such as`, `like`, and `etc.` when they fit. Do not add them mechanically.
- Keep most sentences short or medium length. Use a longer sentence when a cause, condition, contrast, or consequence belongs with the point.
- Phrase behavior, causes, consequences, limitations, and opinions with concrete verbs. Avoid abstract guarantees, detached importance claims, slogan-like contrasts, paired benefits, and promotional wording.
- Phrase an existing limitation in the same conversational voice. Forms such as `One catch is...`, `But...`, and `Here, we can't...` usually sound more natural than a detached statement.

## Subjects and agents

- Do not write objects like people. A component, option, tool, or other piece of software is not the subject of an action verb, in reference docs as much as in prose. Avoid `the flag enables caching`, `the hook lets users subscribe to changes`, `the compiler sees the file as a module`, and `` `timeout` sets the request timeout``.
- Make the person the subject, state a relationship, or use a passive relative clause instead:
  - `A background job lets users export their data` becomes `Users can export their data with a background job`.
  - `The compiler sees the file as a module` becomes `The file is treated as a module`.
  - `The component takes its title from the first child` becomes `Its title comes from the first child`.
  - ``The option was renamed to `retryDelay` and controls the wait between retries`` becomes ``The option was renamed to `retryDelay`, which is used for the wait between retries``.
- Prefer `is used for` to `its value is`, which reads stilted.
- In reference entries, put the outcome first in the passive (``The request timeout is set by `timeout` ``) or make the identifier a definition-list label (`` `timeout`: sets the request timeout``).

## Technical explanations

These apply in the inclusive `we` voice. In other voices, keep the same concreteness with the document's own person.

- Describe code through what `we` are doing, looking for, or getting. Prefer `Here, we check the segment and get the matching object` to `` `SegmentParams` recognizes four kinds of segments``.
- When introducing an operator, type, function, or API, connect it to the current action. Prefer `Then we can use a conditional type to split the pattern` to `A conditional type can split the pattern`.
- Use the exact identifier when needed for clarity, but do not build a sequence of sentences where every identifier is followed by a definition.
- Prefer `we can`, `we need`, `we get`, `we still have`, `then we`, `here we`, and `now we` to detached capability or result statements.

## Punctuation and local formatting

- Use parentheses for quick examples, platform details, qualifications, and short asides. Do not use em dashes.
- Use a colon to introduce code, a command, a list, or an example.
- When a long sentence contains several parallel items, cases, conditions, steps, or tradeoffs, split them into bullet points so each item is easy to scan. Keep connected reasoning in prose, and keep the bullet items concise and grammatically parallel.

## Style check

Read the text aloud. Confirm the person matches the document type and stays the same within each section. In the inclusive `we` voice, check each sentence that explains an action, capability, mechanism, or result for whether a `we` construction would sound more natural. Rewrite any passage that still reads like code followed by definitions, even if `we` appears elsewhere. Then check every sentence whose subject is a component, option, tool, or other piece of software. If the verb is an action, rewrite it as described in Subjects and agents.
