---
title: "Inspectable Is Not Interpretable"
date: 2026-05-30
draft: false
---

I went to [AAMAS](https://cyprusconferences.org/aamas2026/) this year. One talk, on neuro-argumentative learning, caught my attention. The pitch, roughly: a model reasons through a structured debate over past cases, with arguments that attack and support each other, and it can match a neural network's accuracy while staying inspectable. The structure is the reasoning. You can look at it directly. Nothing is hidden from view.

The contrast the authors drew felt wrong. Neural networks are inspectable too. Every weight is visible. There is no sealed compartment in a transformer where the real work happens out of sight. You can write the whole computation down. What you cannot do is follow it, because there is too much of it, and because the intermediate quantities do not correspond to anything you can name.

So maybe interpretability is just an artifact of size. Take any transparent symbolic structure and grow it to the scale at which a network stops being readable, and it stops being readable too. A learned argumentation graph over a four-feature toy dataset is already a tangle you have to filter before you can even draw it. Push it to real data and a chain of cases you recognise is no longer something a person can hold in their head.

There is being inspectable, where every part is visible - networks have this, and so do argumentation graphs. There is being interpretable, where a human can follow it, and this degrades with complexity whatever the substrate, symbolic or neural. And there is being faithful, where the explanation is the mechanism itself, and not a story told about it after the fact.

The last one is what the size objection does not touch in the same way. A saliency map, an attention plot, a post-hoc rationale, each is an approximation of what the network did, and each can quietly diverge from it. A structure that is itself the computation cannot misrepresent the computation, however large it grows. That looks like a difference in kind. So the author's claim is not empty. It is only narrower than it sounded. The defensible version is not that the system is interpretable at a scale where networks are not. It is that the system is faithful by construction.

Still, the structure is faithful to itself. Whether it is faithful to the decision is another question. If a neural component does work upstream and the argument graph mostly rationalises the result, then the part you can read is not the part that decided. The method is called neuro-argumentative for a reason. So I would hold the claim loosely until I know how much of the decision actually lives in the readable structure.

There is also a catch. Suppose the faithful structure is too large to comprehend. A trace of ten thousand cases attacking and supporting one another, every weight exactly the mechanism, and no human able to take it in. Faithfulness still buys something here. The trace can be audited in principle. A second program can check it. It cannot lie about what it did. But in principle and a second program are carrying the weight of that sentence. What faithfulness does not buy, on its own, is a person understanding the decision.

So faithfulness does not really survive the size objection. The question used to be whether a human can read the structure. Now the question is whether anything can check it. That is a real gain, because a check by a program scales where a human does not. But it is a different gain from the one the word interpretable suggests, and it is worth not confusing the two.

This also changes how I see the comparison with a readable post-hoc story that might be slightly wrong. The problem with the readable story is not mainly that it is wrong. It is that you cannot tell when it is wrong. It gives the feeling of having checked something. The faithful trace gives you a check you can run, even if you cannot run it in your head. One offers understanding you cannot verify. The other offers verification you cannot personally perform.

So I came away thinking the inspectable versus black box framing is the wrong axis. Two things matter, and they come apart. One is how faithful the explanation is to the mechanism. The other is how much of it a person can actually hold. A system can do well on one and quietly concede the other. The systems worth building are the ones trying for both at the same time, and I think it helps to keep the two apart, so you can see which one a given method is really buying you.
