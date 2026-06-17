<!--
  HASHNODE PUBLISH INSTRUCTIONS
  ══════════════════════════════════════════════════════
  1. Go to hashnode.com → Write → "Import article"
     OR paste content below into the Hashnode editor directly.

  2. After import / paste, fill in the UI fields:
     • Title       : From Notion to Everywhere: Building Calligra
     • Subtitle    : 
     • Slug        : from-notion-to-everywhere-building-calligra
     • Tags        : blogging, developer-workflow, publishing
     • Cover image : https://calligra.dev/images/blog/from-notion-to-everywhere-building-calligra/cover.jpg

  3. Under "SEO" settings:
     • Canonical URL: https://calligra.dev/blog/from-notion-to-everywhere-building-calligra

  4. Save as draft, review, then publish.
  ══════════════════════════════════════════════════════
-->
Every developer blog needs a first post.

Mine starts with a pen, a family name, and a workflow I got tired of doing by hand.

## The Publishing Problem

I write almost everything in Notion.

It is where ideas begin, drafts evolve, and articles eventually take shape. But publishing those articles was always the same repetitive process:

1. Copy the content from Notion.
2. Fix formatting issues.
3. Paste it into my blog.
4. Repeat for [Dev.to](http://dev.to/).
5. Repeat for Hashnode.
6. Repeat for Medium.
7. Upload cover images multiple times.
8. Configure canonical URLs manually.

None of these steps were difficult.

They were simply repetitive.

And as developers, repetitive work usually means there is an opportunity for automation.

---

## The Goal

I wanted a publishing workflow that looked like this:

```plaintext
Notion → Calligra → Personal Blog + Dev.to + Hashnode + Medium
```

Write once.

Publish everywhere.

Keep my own domain as the source of truth.

That simple idea became Calligra.

---

## What Is Calligra?

Calligra is an open-source CLI tool that takes content from Notion and publishes it to multiple destinations with a single command.

```bash
make sync
make build
make publish
```

Behind the scenes, Calligra:

- Fetches content from the Notion API
- Converts the content into platform-specific formats
- Uploads images and assets
- Publishes to supported platforms
- Sets canonical URLs back to your own website

The goal is simple:

> Your personal blog owns the content. Other platforms help distribute it.

---

## Screenshot: Publishing Workflow

---

![Blog Asset](https://calligra.dev/images/blog/from-notion-to-everywhere-building-calligra/asset-1.png)

## Content Flow

```plaintext
┌──────────┐
│  Notion  │
└────┬─────┘
     │
     ▼
┌──────────┐
│ Calligra │
└────┬─────┘
     │
     ├──► Personal Blog
     ├──► Dev.to
     ├──► Hashnode
     └──► Medium
```

---

## The Story Behind the Name

Before the code, there is the name.

### The Craft

The word _Calligra_ comes from the same roots as _calligraphy_ — the art of beautiful writing.

Calligraphy is not just about words.

It is about intention, structure, and craftsmanship.

### The Heritage

My family name is **Ghalambaz (قلم‌باز)**.

A rough translation would be:

> "Pen player."

Historically, it referred to someone who practiced writing as both a craft and an art.

Someone who enjoyed working with words.

When I built a tool dedicated entirely to writing and publishing, the name felt inevitable.

Calligra connects a modern publishing workflow with a much older tradition of writing.

---

## Open Source from Day One

Although Calligra started as a personal tool, I built it to be useful for other developers as well.

If you write in Notion or anywhere else and publish technical content, you might find it useful.

The project is open source, and contributions, feedback, and ideas are always welcome.

**Repository:**

[link_preview](https://github.com/ghalambaz/Calligra.git)

---

## Why Start This Blog?

Calligra solves the publishing problem.

Now it's time to focus on the writing.

This blog will be a place for practical engineering lessons, architectural decisions, debugging stories, and lessons learned from building software in production.

Not tutorials copied from documentation.

Not generic AI-generated content.

I do use AI—but as an editor, not an author.

AI is great at fact-checking, catching mistakes, improving clarity, challenging assumptions, and helping me communicate ideas more effectively. The experiences, opinions, successes, and failures will still be my own.

The goal is simple: share real engineering stories, real production problems, and solutions that might help someone else avoid the same mistakes.

---

## What's Next?

This post marks the beginning of the journey, not the destination.

Over time, I'll be writing about software architecture, backend engineering, distributed systems, debugging sessions, performance bottlenecks, deployment lessons, and the occasional mistake that taught me something valuable.

Some articles will be technical deep dives.

Some will be lessons learned from real projects.

Some may simply document a problem that took far longer to solve than it should have.

The goal is not to publish content for the sake of publishing.

The goal is to share experiences, capture lessons before they are forgotten, and hopefully help other engineers who run into similar challenges along the way.

---

## Final Thoughts

Calligra started as a way to eliminate a repetitive workflow.

Now it has become the foundation of this blog.

If you are a developer who writes in Notion and owns a personal domain, I hope it saves you time as well.

Thanks for stopping by.

See you in the next post.

—

**Mohammad Ghalambaz**

_Backend Developer writing at Calligra.dev_