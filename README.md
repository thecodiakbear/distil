# Distil

**A persona-led, config-driven content pipeline that ingests news, drafts posts in a defined voice, generates a matching image, verifies both with an AI gate, and holds everything for human approval before publishing.**

Built on a direct-and-verify philosophy: AI does the fast work at every stage, but nothing it produces is trusted until a separate step has checked it.

> This is a project write-up describing the architecture and design decisions. The implementation is private.

---

## The problem

Running multiple content accounts has an obvious bottleneck - supply. There's endless source material and never enough time to read, judge, write, and illustrate it consistently. The naive fix is "have AI write the posts," but that just moves the problem: AI is fluent and confident, and it will happily draft a post that misrepresents its source or pairs it with a broken image. At volume, those errors publish themselves.

Distil is built to solve supply *without* surrendering quality - by embedding verification into the pipeline itself rather than trusting the model or eyeballing every output by hand.

## What it does

One pipeline runs many accounts. Each account is defined entirely by configuration - its niche, its voice, its sources, its destination - so the same machinery drives completely different accounts without being rebuilt. For each incoming item the pipeline:

1. Ingests it from a news source and de-duplicates against what it's already seen.
2. Scores it for relevance against that account's brief, and drops anything that isn't genuine signal.
3. Drafts a publish-ready post in the account's defined voice, grounded strictly in the source.
4. Verifies the draft against the source with an AI gate (see The Eye Test) - does it faithfully represent the article, in the right voice, within limits?
5. Generates a supporting image from the draft.
6. Stores the whole package and holds it for human approval.
7. Publishes approved items to the destination account.

## Architecture

```
 CONFIG (per account: niche, voice, source, destination)
        |  looked up per item
        v
 SOURCE -> DEDUP -> RELEVANCE -> DRAFT -> VERIFY -> IMAGE -> STORE -> HUMAN -> PUBLISH
            (store)  (score +    (in      (Eye     (gen)   (queue) (approve)
                     filter)     voice)   Test)
```

The design is deliberately **config-driven**: the behaviour of every stage is parameterised by a per-account configuration record, so adding an account is a data change, not an engineering change. This is the piece that turns "one automation" into "a platform."

The other defining choice is the **verification gate** sitting between drafting and publishing - an independent AI step that judges the drafted output against the source and brief, not just whether it's well-formed. Failed items never reach the human queue clean; they arrive flagged with a reason. (This gate is its own project - see The Eye Test.)

## Key design decisions

- **Direct-and-verify, made structural.** The writing step is *told* not to invent anything; the verification step independently checks that it didn't. Instruction plus independent verification - belt and braces - rather than trusting a single model call.
- **Verify before you spend.** The text is verified *before* image generation, so the pipeline never spends on illustrating a post that's going to be rejected.
- **Human-in-the-loop by design.** The pipeline's job is to produce a clean, verified, decision-ready queue - not to publish autonomously. A person approves. The AI does the volume; the human keeps the judgment.
- **Config over code.** Niche, voice, sources and destinations live in data, so the system scales to new accounts and new topics without touching the pipeline.

## Stack

Implemented on a low-code automation platform (Zapier - Zaps, Tables, AI actions) with an image-generation API, chosen for rapid iteration. The architecture is platform-agnostic: the same design maps cleanly onto a coded pipeline. AI drafting and verification steps, a configuration store, a content queue, and a human review surface.

## Status

Working end-to-end pipeline for a single account: ingestion through verification, image generation, storage, review, and publishing. Roadmap includes an image-level verification gate, character-consistent visuals, an owned-newsletter branch off the same approved content, and scaling to additional accounts via configuration.

---

*Built by Michael Williamson - The Codiak Bear - technical founder and AI engineer. I design agentic AI systems that direct and verify models rather than trusting them. (https://www.linkedin.com/in/michael-williamson-1789a774)*
