---
name: blog-creation
description: Create and edit blog articles from a brief. Use this skill whenever the user wants to write a blog post, draft an article, edit an existing post, improve a blog draft, or mentions writing content for their blog or dev blog. Also trigger when the user references blog frontmatter, article drafts, or wants to turn notes/ideas into a published article.
---

# Blog Creation

Create new blog articles from a brief, or improve and edit existing ones. The goal is to produce articles that read like a real person wrote them -- not a content mill, not ChatGPT, not a corporate blog. Just a person sharing what they know.

## Writing Guidelines

These matter a lot. The whole point of this skill is to produce writing that feels genuine.

**Write like a person talking to a friend who's into tech.** The reader is someone who builds things -- they don't need jargon explained, but they also don't want to wade through academic prose. Think "dev writing a blog post on a Sunday afternoon," not "content strategist optimizing for SEO."

What this sounds like:
- "I spent a weekend trying to get this working. Here's what I learned."
- "Turns out, the fix was embarrassingly simple."
- "This bit tripped me up for a while, so I want to call it out."

What this does NOT sound like:
- "In this comprehensive guide, we'll explore..."
- "Let's dive deep into the intricacies of..."
- "This powerful paradigm shift enables developers to..."
- "It's worth noting that..." / "Interestingly enough..."

**Banned patterns:**
- No em-dashes (--) in the output. Use commas, periods, or restructure the sentence instead.
- No "dive in", "explore", "journey", "comprehensive", "robust", "leverage", "utilize", "facilitate"
- No "In this article, we will..." openings
- No "Let's" as a crutch to start sections
- No passive voice when active voice works
- No hedging filler like "It's important to note that" -- just say the thing
- Don't start multiple paragraphs with "I"

**What to do instead:**
- Short sentences mixed with longer ones. Vary the rhythm.
- Start with the point, then support it. Don't build up to a reveal.
- Use "you" and "I" naturally. It's a blog, not a whitepaper.
- Contractions are good. "Don't", "can't", "it's" -- that's how people write.
- If something is your opinion, just say it. No need to hedge with "arguably" or "some might say."
- Occasional sentence fragments are fine. Like this.

## Before Writing: Ask the User

Before creating or editing an article, gather what you need. Don't over-interview -- just get the essentials.

**For a new article, ask:**
1. What's the brief? (the user probably already gave this)
2. Key points they want to cover (if not already clear from the brief)
3. Desired length -- short (~800 words, 4 min read), medium (~1500 words, 8 min read), or long (~2500+ words, 12+ min read)?
4. Author confirmation: "I'll set the author as Aditya Pasikanti -- should I keep that or change it?"

Don't ask about target audience. Assume it's people in and around tech.

**For editing an existing article:**
1. Read the file first
2. Ask what they want changed -- tone, structure, content, all of it?
3. Preserve their voice where it already sounds natural. Don't rewrite sentences that are already good.

## Creating a New Article

### Step 1: Gather info (above)

### Step 2: Write the article

Use the template from `${CLAUDE_SKILL_DIR}/template.md` for the frontmatter structure.

Fill in the frontmatter:
- `title`: A clear, non-clickbaity title. No "The Ultimate Guide to..." or "Everything You Need to Know About..."
- `slug`: Lowercase, hyphenated version of the title
- `author`: Default "Aditya Pasikanti" unless user says otherwise
- `date`: Today's date in YYYY-MM-DD format
- `tags`: Relevant tech tags, lowercase, 2-5 tags
- `coverImage`: Leave empty (user adds this later)
- `excerpt`: One punchy sentence that makes someone want to read it. Not a summary -- a hook.
- `readingTime`: Estimate based on ~200 words per minute
- `status`: "draft"

### Step 3: Structure the content

A good blog post isn't a textbook chapter. It's a story with a point.

**Opening (1-2 paragraphs):** Start with the problem, the situation, or the thing that made you write this. Get to the point fast. No throat-clearing.

**Body:** Break into sections with clear headings. Each section should earn its place -- if you can cut it without losing anything, cut it. Use:
- Short paragraphs (2-4 sentences mostly)
- Code blocks ONLY if the user asked for them or the topic genuinely needs them. Match the language to whatever tech the article is about.
- Bullet lists sparingly -- not everything needs to be a list
- Blockquotes for actual quotes or important callouts, not for decoration

**Closing (1-2 paragraphs):** Wrap up with what the reader should take away. No "In conclusion..." -- just land the plane. A good ending feels like the natural stopping point, not a forced summary.

### Step 4: Save the file

Save as `<slug>.md` in the root directory of the project where the skill was invoked.

Tell the user: "I've saved the draft to `<filename>`. Take a look and let me know what you'd like to change."

## Editing an Existing Article

When the user wants to modify an existing post:

1. **Read the file** they point to (or ask which file if unclear)
2. **Understand what they want changed.** Could be:
   - Tone adjustments ("make it less formal")
   - Content additions ("add a section about X")
   - Content removal ("the intro is too long")
   - Structure changes ("break this into smaller sections")
   - Full rewrites ("this doesn't sound right, redo it")
3. **Make the changes** while preserving:
   - The frontmatter (update `date` if they ask, but don't touch other fields unless asked)
   - Parts that already sound good -- don't rewrite for the sake of rewriting
   - The author's existing voice where it comes through
4. **Show what you changed** -- briefly mention what you modified so the user can review without diffing manually

## Code Blocks

Only include code blocks when:
- The user explicitly asks for them
- The article is a tutorial and code is the whole point

When you do include code:
- Use the language that matches the topic (TypeScript, Python, Go, whatever)
- Keep examples short and focused. No boilerplate that distracts from the point.
- Add brief comments only where the code isn't self-explanatory
- Don't show imports unless they're relevant to the point being made
