---
title: "When Memories Disagree"
date: 2026-04-10
draft: false
---

Six months ago, your team gave a client a 15% discount to keep the deal alive. Now the client's subsidiary wants the same terms. You ask your AI assistant what it knows about that original concession. What should it tell you?

The answer depends on who is asking and why. If you are preparing a relationship pitch, the concession was a trust-building move. If you are running a risk review, it was a precedent that could expose you legally. If you are doing financial planning, it was margin erosion. These are not different facts. They are different interpretations of the same event. And any AI agent that has been working with your team for months needs to hold all of them at once.

Current memory systems for AI agents assume there is one correct way to store an experience. You either commit to an interpretation at encoding time, like a knowledge graph with a fixed schema, or you defer everything to retrieval time, like a vector store that just pattern-matches on semantic similarity. The first approach locks you into one perspective before you know which one will matter. The second never builds any goal-relevant structure at all. The raw text _agreed to 15% discount after client mentioned competitors_ does not use risk vocabulary, so a risk-oriented query might never find it.

I started thinking about this through the lens of cognitive science. There is a classic result by Morris et al. from 1977 showing that encoding something for one purpose produces memories poorly suited for another purpose. Even when the _deeper_ encoding would normally be considered better. Encoding context and retrieval context need to match. Minsky had a similar intuition when he argued that minds are better modeled as societies of competing agents than as unified systems.

What if goals parameterized encoding, not just retrieval? Each strategic perspective gets its own encoding agent, its own ontology, and its own knowledge graph. When Relationship Strategy encodes the concession, it stores a trust signal and discards margin data. When Financial Planning encodes it, it stores margin erosion and discards relationship framing. The resulting memories are genuinely separate, shaped by separate goals.

This creates a coordination problem. When a query arrives, we have multiple perspectives with conflicting interpretations. How do you decide which one to surface?

The resolution mechanism comes from argumentation theory. Specifically, Dung's abstract argumentation semantics from 1995. At query time, each perspective proposes an interpretation. Then perspectives critique each other's proposals. Risk Management might attack Relationship Strategy by arguing that trust metrics do not constitute risk exposure. These attacks form a directed graph. You compute which proposals survive using the grounded extension, a deterministic fixed-point computation.

Three distinct retrieval modes emerge naturally from the topology of this attack graph. If one perspective survives unchallenged, the system selects it. If multiple perspectives survive because they complement rather than contradict each other, the system composes them. And if every perspective attacks every other, the grounded extension is empty, and the system surfaces the disagreement itself rather than forcing a resolution.

We call this _Rashomon Memory_, after Kurosawa's film where the same event gets irreconcilable accounts from different witnesses.

For the proof-of-concept, we set up three perspectives (Relationship Strategy, Risk Management, Financial Planning) encoding eight observations from a simulated client engagement. Each perspective independently decided which observations to encode. Out of 24 possible encodings, only 13 passed the relevance filters. The same observation got encoded differently by different perspectives. Then we ran four queries. The same three perspectives produced attack graphs with zero, two, three, and six edges across four queries. No manual tuning. The retrieval mode emerged from the interaction between query context and perspective goals.

This is a workshop paper, so it comes with plenty of limitations and open questions. The encoding pipeline required 37 LLM calls for 8 observations. Each query needed up to 7. That is fine for deliberative tasks like negotiation preparation, but too slow for anything interactive. I have not compared this against standard RAG on retrieval metrics. I have not run user studies. The attack quality depends on prompt design. There is a lot of future work ahead.

Still, the core idea feels right to me. For agents that operate over long time horizons and serve multiple goals, memory is not a storage problem. It is a coordination problem. And when interpretations genuinely conflict, the honest thing is to show the conflict rather than hide it behind a single confident answer.

The paper has been accepted to the EXTRAAMAS workshop at AAMAS 2026. Full paper is available [here](https://arxiv.org/abs/2604.03588).
