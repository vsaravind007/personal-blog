---
categories:
- software
- architecture
- AI
- leadership
- engineering
title: 'Know Before You Read: A Lightweight AI-Usage Header for Documentation'
date: 2026-06-21T16:30:02+05:30
summary: A simple AI disclosure table that helps cut down the effort on AI based document reviews
cover:
    image: "/assets/images/reviewing-ai-articles/cover.webp"
draft: false
tags: ['software', 'ai', 'documentation', 'engineering']
---

![Know Before You Read](/assets/images/reviewing-ai-articles/cover.webp "Know Before You Read")

High velocity engineering teams always had the issue of not having enough documentation. Most of the time its buried within code, in the heads of a few or deep inside technical specs that are often not kept upto date after a few iterations of the implementation in real world.

Even before AI became super popular for generating code, people discovered generating documentation is much faster with AI as a side effect, teams shipping products & features are dealing with an issue that I never thought I would have to deal with - Whole lot of documentation! 

## The Problem

Like everything else in the SDLC, writing documentation is also "AI Assisted" now. Excel warriors and middle management without much technical know-how is now creating pages and pages of "technical" documents in minutes and asking for reviews/feedback that takes away time and overloads the team.

Given the verbosity of LLMs(driven by their hidden rules to burn/generate more tokens), anyone having to deal with these documents will have to go though pages and pages of text to extract sensible information out of them. Often times its going to be repeating sections, just rephrased.

However, just because a document was written using the help of AI or is AI generated doesn't mean its usesless or its not worth reading, reviewing or providing feedback on.

## The Fix

There is no real way to enforce control over how AI used for generating documents, it becomes really hard to quality gate the documents generated with the help or by AI using just frameworks or a set of loose guidelines.

Introducing a compact table format you can drop at the top of every document will help a reviewer decide in 10 seconds whether (and how) to spend their time, rather than dismissing the document or discovering halfway through that they're reviewing something nobody actually owns.

Assuming some level of AI usage is involved, get the authors to add the following table to all documents would ensure the time spent by readers/reviewers would be utilised effectively. Nobody likes to create/read documents without a point afterall.

### AI Usage & Review Declaration

| Field | Entry |
|---|---|
| **Document Objective** | _One line: what decision/action this doc enables_ |
| **AI Involvement** | None / Drafting aid (outline, phrasing) / Heavy generation (large sections AI-written) / Fully AI-drafted |
| **Review Depth** | Skimmed / Read & edited / Verified line-by-line (incl. claims, numbers, logic) |
| **Sections I Personally Own** | _List sections the author wrote/verified vs. AI-generated and unverified_ |
| **Feedback Needed On** | _Specific asks (e.g. "validate approach in Section #3", "sanity-check capacity numbers") and not "review everything"_ |

#### Why ?

Each field in the table serves a specific purpose to help the reviewer and ultimately the team.

- Document Objective - This is generally lost once you move past the title of AI generated documents and may find bits and pieces in conclusion/closing notes, having a one liner objective keeps the decision to be made in view.
- AI Involvement - Sets the reviewer's baseline trust level. "Fully AI-drafted" means the reviewer should assume nothing is verified unless stated otherwise, helpful if the document has a lot of datapoints / critical assumptions.
- Author's Review Depth - If the author has only skimmed through the document, the reviewer needs to assume they're the first real reader, _expect bullshit in short_.
- Sections I Personally Own - Pinpoints where the author's judgment was actually applied & where it's AI output. The reviewer can focus their attention accordingly.
- Feedback Needed On - Forces the author to think about what action the review enables, instead of asking for generic review/feedback.

In case of thinking about improvements, make sure to keep the fields to 5 max - if it's longer, it becomes a chore & people will stop filling it out honestly. Also consider making "Author's Review Depth" a mandatory field with no AI-favorable default, this is to stop people from quietly skipping it. You could make this a Confluence template macro so it's auto-inserted and nobody can ship a spec without filling it in.

## Closing Thoughts

AI is fundamentally changing the way work is getting done. One might argue against traditional documentation and spec work given the velocity at which features can be shipped with AI assitance. Counter argument would be the need to have human/machine readable documentation work that can be used for future work or extensions. Feeding AI generated documents into AI would not result in "better" documentation.



