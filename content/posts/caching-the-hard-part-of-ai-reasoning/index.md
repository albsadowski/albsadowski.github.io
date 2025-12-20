---
title: "Caching the Hard Part of AI Reasoning"
date: 2025-12-20
draft: false
---

In [this blogpost](/on-making-ai-reasoning-transparent), I wrote about making AI reasoning transparent by splitting a prompt into manual steps like entity identification and symbolic application. That approach worked for hearsay determination, but it relied on an external task definition. I had to manually shape the structure of the terms, predicates, and rules myself. The model was just a processor for a logic framework I provided.

My new paper, ["On Verifiable Legal Reasoning,"](https://dl.acm.org/doi/10.1145/3746252.3761057) extends this by automating the hardest part: the task definition itself. We built a modular architecture called SOLAR (Structured Ontological Legal Analysis Reasoner) that uses a multi-agent system to figure out the ontology and the rules directly from the statute.

The core engineering shift is to handle this "hard part", the ontology learning and interpreter writing, once, and then reuse it.

Legal reasoning is hard because it requires interpreting complex statutory language and maintaining logical consistency. Instead of asking a model to reinvent the law for every query, we split the process into two stages.

First, we use a high-end reasoning model in a _Knowledge Acquisition_ stage. It analyzes the statute to build a TBox (Terminological Box) that captures the formal vocabulary and conditional rules. It also generates a Python-based interpreter to operationalize that knowledge.

Once this task definition is handled, you can let a smaller, cheaper foundational model focus on the "easier" part: mapping case facts onto that pre-defined schema.

If you have worked with function calling, this feels very similar. In a standard setup, you define a schema and the LLM extracts arguments from text to fit it. In SOLAR, those extracted facts form an ABox (Assertional Box).

But by using a formal ontology instead of just a flat JSON schema, we win something much more powerful: symbolic inference. Because the system employs an SMT solver, it can deduce facts that aren't explicitly stated in the natural language query.

For example, if a case describes a taxpayer’s deceased spouse and their household arrangements, the SMT solver can formally derive "Surviving Spouse" status. The small model just handles the "easy" task of fact extraction; the symbolic engine handles the logical deduction.

I evaluated this on the SARA numeric dataset, which involves complex U.S. federal tax calculations. Foundational models are historically terrible here, with a baseline accuracy of just 18.8%.

By offloading the task definition to the SOLAR framework, those same foundational models jumped to 76.4% accuracy. We narrowed the gap between cheap models and specialized reasoning models (like o1) from 68.2 percentage points down to just 5.9 points.

Beyond the accuracy, this approach represents fundamentally better engineering because it strategically optimizes for both efficiency and reliability. By passing a compact TBox representation instead of the full statutory text, the system achieved a reduction in token usage of approximately 45%. The architecture also ensures significantly higher consistency; because the logical rules are locked into a formal framework rather than being reinvented for every query, the performance variance was reduced from 0.28 in zero-shot approaches to just 0.08. Most importantly, this creates a verifiable "glass-box" system where you are no longer debugging an opaque black box. You can independently check the ABox to see if the small model misidentified a fact or inspect the TBox to see if the agents misunderstood the law, providing explicit inspection points throughout the process.

If you're interested in the technical details and experimental results, the full paper is [available here](https://dl.acm.org/doi/10.1145/3746252.3761057).
