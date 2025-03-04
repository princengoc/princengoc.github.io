title: full-stack-engineering-with-claude
date: 2025-03-03
tags: claude, gpt4, pet project, typescript, unspoken
description: A reflection on what AI-assisted coding has *really* been like -- the struggles, the breakthroughs, and the lessons learned.

# Full-Stack Engineering with Claude: From Copy-Pasting to Building for Real  

Five months ago, I was blindly copy-pasting Claude’s code, barely understanding what it was doing. Today, I actually believe I could work as a full-stack engineer. To be clear, I wasn’t looking to become a full-stack engineer -- I just had apps I really, really wanted to build.  

With [*Unspoken*](https://un-spoken.netlify.app/) beta now released, I want to reflect on what AI-assisted coding has *really* been like -- the struggles, the breakthroughs, and the lessons learned.  

### **AI and Coding: The Early Struggles**  

I'm all for AI in coding, but I still don’t have GitHub Copilot set up. Why? Because my experience with GPT-4 for Python development was so inconsistent that Copilot felt like it would slow me down.  

Sure, AI helped *sometimes* -- saving me from wrangling a plotting library or setting up a boilerplate data pipeline. But half the time, I could have written the line myself faster. The other half, it gave me broken scripts, and after a few frustrating iterations, I’d take over manually.  

It wasn’t that AI couldn’t code -- it just wasn’t *better* than what I could already do. Python was my home turf. I knew the libraries, the quirks, the shortcuts. AI wasn’t game-changing because I didn’t *need* it.  

But when I started building apps, that was a different story. By October 2024, I had technical experience, but I was far from a full-stack engineer. My frontend knowledge was limited to hacking together Dash and Plotly at work. My backend experience mostly involved swearing at Snowflake. 

### **The AI Honeymoon: Rapid Prototyping, Then Chaos**  

**Prototyping was magic.**  

I tried Claude after reading (Simon Willison)[https://simonwillison.net/2024/Oct/21/claude-artifacts/] blog post on its artifact preview. Being able to *see* live previews of front-end code felt revolutionary. The designs looked amazing to my untrained eyes, and that gave me the confidence to experiment.

**Scaling was a nightmare.**  

The honeymoon lasted about a week. I had a Frankenstein codebase that felt like a DIY renovation: first we start with a prototype of the living room, then add on the plumbings, then redo the kitchen, and so on. AI-assisted coding is fantastic for quick UI generation, but once your app grows, things break. And AI doesn’t *fix* messy architecture -- it *accelerates* it. It was easy for Claude to suggest specific libraries to fix specific problems (state issues? TanQuery!) without considering how we got into the rut in the first place. 

That’s when I realized: AI is only as good as the instructions you give it. And to give better instructions, you have to know what you’re doing.  

## **Lesson 1: Learn.**  

I asked Claude if my code could be structured better. That’s when it casually introduced me to **context providers and hooks**. Life-changing. 

Looking back, it’s wild that I was writing a React app without knowing what hooks were. But that’s just how AI-assisted learning works -- it’s pain-driven. You don’t realize you’re missing a concept until something breaks. Then you ask AI, and *boom*, new lesson unlocked.  

Here’s the problem: AI works best when you use the *right* words. As a beginner, you don’t *know* the right words, so you rely on whatever defaults AI throws at you. And AI defaults rarely lead to production-grade software.

I should have known this. A year ago, GPT-3.5 had me manually writing HTML + CSS, building a Flask backend, and orchestrating background tasks with Celery and Redis. I learned a lot (I love Redis), but maintenance was a nightmare.  

By October 2024, Claude 3.5 Sonnet gave me a TypeScript React Next app with Local Storage. A much better starting point. But even then, I had to evolve the stack:  

- **Frontend:** Went through (something I forgot), then Mantine  
- **Backend:** Dexie to Tanstack + MirageJS to Firebase to Azure SQL Database  

Each time, I switched libraries to solve specific pain points. Then my needs grew, and I found *new* pain points. AI-assisted learning is great, but it’s also wildly inefficient.  

### **The Fix: Be *Picky and Demanding***  

Want better AI output? Learn *just enough* about architecture, libraries, and best practices so you can be *picky*.  

- **Shop around** for libraries. Understand their pros and cons before committing.  
- **Be specific** with AI. “Use X because I want Y” saves you from default mediocre suggestions.
- **Demand quality.** AI-generated code isn’t final -- it’s a first draft. Push for improvements.  

A couple of months ago, I had to write (Tiptap extensions)[https://tiptap.dev/docs/editor/extensions/overview]. This time, I didn’t just ask AI to generate code -- I read the docs, learned the design philosophy, and explored alternatives (Lexical, I tried to love you, but I couldn't, sorry). I sandboxed a project just for building Tiptap extensions, so that I can learn. Once I actually *understood* Tiptap, I could supervise AI’s work. The result? Faster iterations, fewer bugs, and a much more **consistent** app.  

## **Lesson 2: Refactor Codes As Often As You Do Laundry**  

I mean it. A clean, consistent codebase is *critical* for AI-assisted coding.  Why? Because AI doesn’t *think* -- it pattern matches. When I tell Claude, “Implement the Reactions system like we did the Exchange system,” it nails it in five minutes. AI is phenomenal at copy-pasting. Give it a consistent, good example to replicate. And I don't mean trivial things like naming or code structures -- AI is already very good at that. I mean major refactors, one that takes a document to describe, that completely shuffles up your business logic, that normally you would moan if someone forces it upon you. 

<img src="/figures/claude-refactor.png" alt="Claude understood my request for refactor" width="640">

Of course one has to be prepared for follow-ups, like this. 

> Review the project description and the project codes. This project was recently revamped in logic and went through a major refactor, so I need to go through and review that everything is implemented consistently.  
> 
> First, I think the GameBoard.tsx and ListenerReactions.tsx files need tobe reviewed, since they have some seemingly serious typescript errors that indicate logic problems

Why do laundry? Because the flip side is: messy codes = messy AI output. The messier your project gets, the worse AI’s codes become. If something feels off, clean it up. Better yet -- refactor right after AI suggestions so the chaos doesn’t pile up.  

## **Lesson 3: Pick a Popular Stack (or Prepare to Work)**  

AI fails in many ways, but **library upgrades** are especially painful.  

Claude sometimes writes deprecated code. My workaround? Ask GPT-4 to rewrite it, since it has web search. But when GPT doesn’t get it right, I have two choices:  

1. **Dig through GitHub issues**, or
2. **Switch to a better-supported library**  

I’ve done this multiple times (e.g., ditching Lexical for Tiptap), and I suspect I’m not alone. AI-driven development might be reinforcing a *rich-get-richer* effect -- popular libraries get better AI support, making them even *more* popular.  

So if you want AI to work in your favor, stick to well-used stacks. Or prepare to roll up your sleeves and do actual coding. 

---

## **Where I Am Now**  

[*Unspoken*](https://un-spoken.netlify.app/) was built in a few weeks. It has the scope of a “weekend” project (ie, nothing much), but I’m still proud of what Claude and I built. 

At this point, I act as a **very hands-on, slightly OCD team lead** to AI. I review its PRs, enforce quality control, and dive into debugging trenches when needed. I’m comfortable enough with TypeScript to make small fixes myself, saving the hassle of explaining everything to Claude. But most of my time is spent making **decisions** on how to implement the next feature, then reviewing AI’s output and refining it.  

Our code isn’t perfect -- we still do laundry. But it’s *consistent* enough that building new features is **fast, exciting, and fun**.  

And in the end, *fun* is all that matters.
