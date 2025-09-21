---
title: "On Making AI Reasoning Transparent"
date: 2025-09-20T21:29:33Z
draft: false
---

LLMs give you answers but hide how they got there. Ask _ChatGPT_ about a legal question and you get a conclusion with what looks like solid reasoning. Chain-of-Thought prompting has made this even better - models now show their step-by-step thinking in natural language. But that reasoning is still embedded in free-form text. You can read through the explanation, but you can't really verify each logical step independently. The reasoning path is there, but it's not structured in a way that lets you examine specific components.

This matters in domains where explaining your reasoning is crucial. A lawyer needs to justify their argument to a judge. A doctor explains their diagnosis to colleagues. But current AI systems, even with improved reasoning capabilities, don't provide the kind of structured, verifiable reasoning paths that high-stakes decisions require.

I kept wondering: what if I explicitly separated language understanding from logical reasoning? Instead of asking one model to do everything at once - interpret text, extract meaning, apply rules, and generate explanations - what if I broke it down into distinct, verifiable steps?

I tried splitting legal reasoning into three parts. First, **entity identification**: extract the relevant pieces from messy text. What are the key people, events, statements mentioned? This step converts unstructured language into structured data points. Second, **property extraction**: figure out what relationships and attributes apply to those entities. Is this statement made in court or out of court? Who said what to whom? What's the context and purpose? Third, **symbolic rule application**: use formal logic to apply the legal rules deterministically. No more pattern matching or implicit reasoning - just explicit logical evaluation that can be verified.

Each step produces explicit outputs you can examine independently. If something goes wrong, you can pinpoint exactly where the breakdown occurred.

I picked hearsay determination to test this approach - a specific legal task about whether evidence is admissible. The rule seems straightforward but requires nuanced reasoning about statements, context, and purpose. Take this example:

> Martin smiled and nodded when asked if he did so by an officer on the scene.

In the *first step*, the system identifies Martin as a person, the smile and nod as communicative conduct, and the officer interaction as the context. It extracts these as discrete entities with explanations for each identification.

The *second step* determines the properties and relationships: this happened out of court, the conduct was assertive (intended to communicate), and it implies Martin acknowledged doing something specific. Each property gets mapped to the identified entities with justification.

The *third step* applies formal logic using an SMT solver: _out-of-court statement_ + _offered to prove the truth of what was asserted_ = hearsay. The logic is deterministic and transparent.

Instead of getting one long explanation mixing interpretation with reasoning, I get structured outputs at each stage. A legal expert can verify: did the system correctly identify the relevant statement? Did it properly classify the context and relationships? Is the final logical application sound?

The results were encouraging. On hearsay cases (_LegalBench_), this approach achieved 92.9% accuracy (F1) on _o1_ compared to 71.4% with standard prompting. More interesting was the transparency. When the system made mistakes, I could trace exactly where things went wrong. Sometimes it misidentified entities, sometimes it missed crucial relationships, sometimes the logical rules had gaps.

This was just one experiment in one narrow domain. The approach feels brittle in practice - it requires careful prompt engineering and works much better with some models than others. The OpenAI o-family models showed substantial improvements, while others struggled.

But it points toward something useful: maybe building explainable AI systems isn't about solving the entire black box problem at once. Maybe it's about creating explicit inspection points where humans can verify reasoning components step by step. The three-step framework is just one way to think about decomposition, but it suggests we can build AI systems that show their work systematically, not just provide post-hoc explanations.

If you're interested in the technical details and experimental results, the full paper is [available here](https://arxiv.org/abs/2506.16335).
