---
title: "Does Structured Decomposition Generalize?"
date: 2026-04-17
draft: false
---

A few months ago I wrote about splitting legal reasoning into three steps: entity identification, property extraction, and symbolic rule application. It worked for hearsay determination. But I ended that post with a caveat - it was one experiment in one narrow domain; some models responded well, others struggled.

We tested the same idea across three domains and eleven language models. The short version is that the approach generalizes, but not uniformly, and the story is more interesting than a single number.

We kept the three-step decomposition from the first paper, but grounded the task definitions in semantic web ontologies with OWL and SWRL rules instead of ad-hoc JSON. This lets us use standard semantic web tooling and a proper OWL reasoner (Pellet) for the symbolic step. We then picked three tasks that satisfy two conditions: a decision boundary that can be expressed as a rule, and predicates that can be extracted from text. Hearsay determination from LegalBench. Method application from SciERC, which asks whether a scientific text describes a method being applied to a task. Clinical trial eligibility from NLI4CT, which asks whether a patient claim follows from stated inclusion and exclusion criteria.

Across all 33 model-task combinations, structured decomposition beat few-shot prompting by 4.6 percentage points in F1 (79.8% vs 75.2%). The improvement was statistically significant. Nine of eleven models showed positive gains on average.

Method application showed the biggest gains, +7.3 percentage points, significant on its own. Clinical trial eligibility showed +3.6 points. Hearsay, the original task, showed only +2.8 points and did not reach significance on its own.

I think this makes sense in retrospect. Hearsay requires subtle interpretation of legal language. The hard part is reading the text carefully, not combining the predicates. Method application is closer to the sweet spot. The logical structure matters, and the predicates are relatively easy to extract. Clinical eligibility sits in between. It has clean rules but demands medical knowledge for synonymy and implicit exclusion.

The model-level results follow a pattern too. Larger models benefit more than smaller ones. Gemini 2.5 Pro gained 6.9 points on average; Gemini 2.5 Flash lost 1.1 points. Claude Sonnet 4.5 gained 7.5 points; Claude Haiku 4.5 lost 6.4 points. The decomposition only helps if the model can actually perform the extraction steps reliably. A weaker model makes more extraction mistakes, and no amount of symbolic verification downstream can fix bad inputs.

One more finding that surprised me. In the first paper, I used something called complementary predicates, where the model has to assert both the positive and the negative condition explicitly. This helped a lot with the o-family models at the time. In the new evaluation, across a broader set of current models, it hurt. Performance dropped by 5 points.

So, does it generalize? I'd say yes, but with caveats. The approach works beyond legal reasoning, plus it helps across model families. The gains are real and statistically significant in aggregate. But the size of the gain depends on the task, and the approach is not a substitute for model capability. If you are working on a rule-governed domain with clear predicates and a capable model, structured decomposition is worth trying. As an extra note: the primary reason for this exercise is explainability not the accuracy.

The full paper is [available here](https://arxiv.org/abs/2601.01609).
