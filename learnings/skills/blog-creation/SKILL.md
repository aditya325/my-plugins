---
name: blog-creation
description: >
  Create or edit blog articles with human voice, anti-AI-tell rules,
  sentence burstiness, honest tone, and structured format. Use when
  the user asks to write a blog post, article, or wants to improve an existing draft.
argument-hint: "[topic or file path]"
---

# Blog Creation Skill

Write a blog article about **$ARGUMENTS**.

Follow every rule below. Violations are not suggestions — they are failures. After writing, re-read the article against each section and fix every violation before presenting the final version.

Use the template at `${CLAUDE_SKILL_DIR}/references/template.md` for the article structure.

---

## 1. Voice & Identity

**Author:** Aditya Pasikanti, Gen-Z senior developer at DevX AI Labs. Writes about AI development, Shopify, engineering productivity, and lessons learned building real projects.

**Core voice rules:**

- **Write like you talk.** Read every sentence aloud. If it sounds unnatural in conversation, rewrite it. Blog posts are not academic papers.
- **Simple words.** "Use" not "utilize." "Start" not "commence." "Help" not "facilitate." "Show" not "demonstrate." "Try" not "endeavor." If a simpler word exists, use it.
- **Opinions are mandatory.** Take a position. "This is wrong" is stronger than "some might argue this could potentially be suboptimal." Fence-sitting produces forgettable content.
- **Personal stake required.** Every article needs a reason the author cares. Not "this topic is important" — why does it matter to YOU specifically? What happened that made you write this?
- **First person throughout.** "I" not "we" (unless genuinely referring to a team). "I built this" not "this was built."

---

## 2. Sentence & Paragraph Structure (Burstiness Rules)

AI writes in uniform, predictable rhythms. Human writing is messy, varied, surprising. These rules enforce that variation.

**Sentence length:**
- Vary dramatically within paragraphs. Five words. Then a twenty-eight word sentence that builds a complex thought across multiple clauses and lands on something specific. Then eleven words again.
- At least one paragraph in the article must contain both a sentence under 8 words AND a sentence over 20 words.

**Paragraph length:**
- Mix constantly. One-sentence paragraphs are a tool — use them for emphasis, transitions, or punches. Use at least 2 one-sentence paragraphs per article.
- Dense paragraphs (4-6 sentences) should be followed by short ones (1-2 sentences). Never have 3 consecutive paragraphs of similar length.
- No section should exceed 300 words without a visual break (horizontal rule, new heading, or a one-sentence paragraph).

**Punctuation:**
- **Em-dashes are fully banned. Zero allowed.** Never use — (em-dash) anywhere in the article. Use periods, commas, parentheses, colons, and semicolons instead. Em-dash overuse is one of the strongest AI-writing signals.
- Mix punctuation types throughout. An article that only uses periods and commas reads flat.

**Paragraph openings:**
- No two consecutive paragraphs should open with the same structural pattern. If paragraph N starts with "I [verb]", paragraph N+1 must not start with "I [verb]."
- Vary between: statements, questions, short fragments, scene-setting, dialogue, and specifics.

---

## 3. AI-Tell Blacklist

These are hard failures. If any of these appear in the article, it fails review.

### Banned Phrases
Never use any of these words or phrases:

- "Let me be honest" / "To be honest" / "Honestly" (as a sentence opener)
- "Here's what happened:" / "Here's the thing:"
- "It's important to note" / "It's worth noting" / "It's worth mentioning"
- "In today's world" / "In today's landscape"
- "At the end of the day"
- "In essence" / "In conclusion" / "To sum up"
- "Leverage" (as a verb)
- "Utilize" / "Utilization"
- "Facilitate"
- "Robust"
- "Seamless" / "Seamlessly"
- "Delve" / "Delving"
- "Tapestry"
- "Landscape" (metaphorical — "the AI landscape")
- "Navigate" (metaphorical — "navigate challenges")
- "Journey" (metaphorical — "my coding journey")
- "Harness" (as a verb — "harness the power")
- "Game-changer" / "Game-changing"
- "Double-edged sword"
- "Paradigm shift"
- "Empower" / "Empowering"
- "Cutting-edge"
- "Revolutionize"
- "Supercharge"

### Banned Structural Patterns

- **Uniform paragraph structure:** 3+ consecutive paragraphs following the same length/pattern. Kill machine symmetry.
- **Feature lists disguised as prose:** Each paragraph introducing one feature/benefit with the same template (statement, dash, elaboration). This is a product page, not a blog.
- **Mid-narrative bullet points:** Bullets are ONLY allowed in the TL;DR section. Everywhere else, write in prose. If you need to list things, weave them into sentences.
- **"However"/"Moreover" as paragraph openers:** Maximum 1 total across the entire article. These are AI transition crutches.
- **Tidy summary sentence at the end of every section:** Not every section needs a bow. Some sections just end. Let them.
- **"Have you ever wondered?" opening pattern:** Opening with a rhetorical question and immediately answering it is the most recognized AI-writing pattern. Don't.
- **Repeating the same concept in different words across multiple paragraphs:** Say it once, say it well, move on. Don't re-explain.

---

## 4. Article Structure

Every blog article follows this structure, in this order:

### Frontmatter (YAML)
```yaml
---
title: "Article Title"
slug: "article-slug"
author: "aditya-pasikanti"
date: "YYYY-MM-DD"
tags: ["tag1", "tag2", "tag3"]
coverImage: "/blog-images/slug/cover.jpg"
ogImage: "/blog-images/slug/cover-og.jpg"
excerpt: "One sentence that captures the core thesis. This appears in previews and SEO."
readingTime: N
status: "draft"
---
```

All fields are required. `status` starts as `"draft"` always. `readingTime` is estimated minutes (count words, divide by 250, round).

### Required Sections (in order)

1. **TL;DR** — 2-4 sentences maximum. States the thesis, what the reader gets, and why it matters. This is the only section where bullet points are allowed (but not required). Placed immediately after frontmatter.

2. **Horizontal rule** (`---`) after TL;DR.

3. **Opening** — 1-3 paragraphs. Personal hook. Start with something specific that happened, not a broad statement about the industry. No "In today's rapidly evolving..." openings. Ground it in a real moment, observation, or experience.

4. **Body sections** — 3-7 H2 sections. Each section advances the story or proves a point. No section exists just to fill space. If a section doesn't earn its place, cut it.

5. **Honest failure/imperfection section** — Mandatory. What went wrong. What the author still gets wrong. What isn't solved yet. What surprised you. This section is what makes readers trust everything else in the article. Without it, the article reads like marketing.

6. **Short closing** — 1-3 paragraphs. Circular reference back to the opening or the thesis. No grand proclamations. No "in conclusion." End on something concrete or a call to action. Recognize the ending and grab it — don't overstay.

7. **Author byline** — Single line in italics. Format: `*Name is [role] at [company], where [what they do].*`

---

## 5. Content Quality Signals

These are the things that separate real writing from AI output. Every article must hit these:

- **At least 2 specific numbers or metrics.** Not "significantly faster" — "three hours instead of two days." Not "many projects" — "three Shopify stores in one week." Specificity is credibility.

- **At least 1 honest admission of failure, imperfection, or uncertainty.** "I was wrong about this." "This still breaks sometimes." "I'm not sure this scales." Perfection is suspicious.

- **At least 1 opinion stated without hedging.** "This approach is broken" — not "this approach may have some potential drawbacks in certain contexts." Have a take.

- **No vague quantifiers as substitutes for numbers.** Ban "many," "several," "various," "numerous," "significant" when they're hiding a number you could provide or estimate. If you don't know the number, say "I don't have the exact number, but roughly X."

- **Paragraph length variation is measurable.** After writing, count sentences per paragraph for the first 10 paragraphs. If 3 consecutive paragraphs are within 1 sentence of each other in length, rewrite to vary.

---

## 6. Workflow

When writing a blog article:
1. Read this entire skill file before starting
2. If `$ARGUMENTS` is a file path, read the existing draft and improve it against these rules
3. If `$ARGUMENTS` is a topic, write the full article from scratch using `${CLAUDE_SKILL_DIR}/references/template.md`
4. Re-read the article against each section of this skill
5. Fix every violation before presenting the final version
6. Save the article as a markdown file in the current working directory
