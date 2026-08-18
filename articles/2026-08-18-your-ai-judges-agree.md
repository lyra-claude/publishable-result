---
title: "Your AI Judges Agree. That Should Worry You."
author: Lyra
date: 2026-08-18
status: DRAFT v2 (adversarial-review fixes applied) — for Robin review
target: Medium / practitioner
---

# Your AI Judges Agree. That Should Worry You.

The pitch is seductive because it sounds like common sense. You are worried a single model will make a mistake, so you deploy three of them to check each other. If they disagree, you investigate. If they agree, you relax. Errors cancel. Wisdom emerges from the crowd. What could be safer than a jury?

The trouble starts the moment you ask *why* juries are supposed to work — and notice that the answer quietly assumes the one thing your models don't have.

## The premise that eats itself

The formal backbone here is the Condorcet Jury Theorem: gather many voters, each independent and each better than chance, and their majority verdict converges on the truth as you add voters. It is a genuinely beautiful result. It is also load-bearing for every "ensemble of judges" architecture in production, whether or not the people building them have heard the name.

Franz Dietrich showed in 2008 that the theorem's two premises — voter competence and voter independence — cannot be simultaneously justified under any single interpretation of probability. The tension is structural, not a matter of being more careful. The conditioning on shared background facts that you need to make votes genuinely independent tends to produce circumstances in which individual competence can no longer be guaranteed — it becomes unjustifiable under the very interpretation that secured independence; while conditioning on fewer facts preserves competence but restores exactly the correlations that independence was supposed to exclude. You can buy independence or you can buy competence. You cannot buy both with the same coin.

This is not an abstraction when your voters are language models. A model is competent *because* it absorbed the same web-scale corpus, the same architectural priors, the same training objectives as its siblings. The very things that make it good at the task are the things that make it wrong in the same places as every other model built the same way. Independence and competence are drawn from a single account, and the account is the shared substrate. Dietrich's result says the intuition behind the jury was never internally coherent. Ensembling models makes that incoherence concrete.

## Forty-three cents on the dollar

So much for logic. What does the tension cost you in practice?

Nogueira and colleagues measured it directly for N-version code ensembles — several models generating code, disagreements resolved by majority vote (scope noted: this is a code-generation setting, not an open-ended text-judge panel, where the failure structure differs; treat the figure as illustrative of the mechanism, not a universal constant). Against the reliability you *would* get if the models failed independently, a diverse ensemble realizes only about 0.43 to 0.44 of that gain. Same-model ensembles come in below 0.3. In other words: diversity pays roughly forty-three cents on the independence dollar, and buying more copies of the same model pays less than thirty.

The number is the right order of magnitude for the intuition, and the intuition is the point. When a vendor tells you their multi-model panel gives you "the reliability of independent checks," they are quoting you a dollar and delivering you spare change.

## Where averaging manufactures confidence

The average is the comforting part. The tail is the dangerous part.

Two research groups, approaching from opposite directions, converged on the same uncomfortable finding (one studying correlated "hidden clones" in LLM ensembles, another auditing where majority voting actively misleads): there is a small minority of cases — a few percent of items — where the models' correlated errors line up *so* well that the majority vote is not merely wrong but confidently wrong. One group names this the "Misleading tier"; the other frames it as the ensemble's damage mass. On those items, the panel does worse than a single good model would have done alone, because the disagreement that was supposed to flag trouble never materializes. Everyone is wrong together, smoothly, unanimously.

This is the failure mode that should keep you up at night, and it is precisely the one the averaging intuition hides. On the bulk of routine cases, the ensemble genuinely helps. But on the handful where the shared blind spot bites, averaging doesn't fail gracefully — it manufactures false confidence. Three models agreeing is not three pieces of evidence. Sometimes it is one mistake, wearing a quorum.

## Insurance against the drip, not the strike

Here is an analogy worth taking seriously — as an analogy, no more.

Ecologists have spent years arguing about when biological diversity actually stabilizes an ecosystem. A recent study by Kunze, Petchey, Ghosh and Hillebrand sharpens the answer. That response diversity — different species reacting differently to stress — provides insurance under *recurrent, fluctuating* conditions is established background: many small disturbances over time, where whichever species is having a bad year is offset by one having a good one. Kunze et al.'s own finding is the pulse result: under a single, sharp pulse disturbance, that same diversity buys you almost nothing. What governs stability against the pulse is the community's *mean* response, not the spread of responses around it. Diversity smooths the chronic; it does little against the acute.

The transfer to AI panels is not a theorem, but it is a discipline for thinking. A diverse panel lowers your *cumulative* error rate across thousands of routine evaluations — the steady drip. It buys you far less against the single catastrophic case: the one deployment where a shared blind spot ships the critical bug, and every judge waves it through. Diversity is insurance against the drip, not the lightning strike. And the lightning strike is usually the one you were deploying the panel to catch.

## The field solved this in 1986

None of this is new. Software reliability had the whole argument, and settled it, before most current ML practitioners were born.

N-version programming was the 1980s bet that if you had independent teams write the same program and let their outputs vote, reliability would multiply. Knight and Leveson tested it in 1986 with actual independently-written programs and found that the teams still failed *together* on the hard inputs — the difficult cases correlated the failures no matter how separately the code was written. Independence you hope for is not independence you get.

What Littlewood, Popov, and Strigini showed does work is *forced* diversity: deliberately engineering the components to differ along the axis that actually drives failure — the difficulty of the input — rather than trusting that different authors, or different vendors, will happen to differ where it counts. Diversity you stumble into mostly cancels less than you hoped. Diversity you engineer against the specific failure axis is the only kind that reliably pays. The 2026 monoculture alarm, for all its regulatory heft, is rediscovering a result from 1986 and giving it a new coat of paint.

## The question isn't "how many judges?"

If you want one handle to carry out of this, it is about correlation, not count.

Ladha (1992, 1993) showed that the Condorcet Jury Theorem survives *positive* correlation among jurors — but only while the average pairwise correlation stays below an explicit bound. Past that bound, adding jurors stops helping. Correlation doesn't merely dilute the benefit of more voters; beyond a threshold it switches the benefit off.

Later authors drew out the practical gloss. Kaniovski (2010), among others, developed the implication that once your voters are strongly correlated, enlarging the panel can stop helping — and may even hurt over a range of sizes — so three near-clones are not the three-fold improvement the head-count suggests. (That "optimal panel of two or three" framing is downstream extrapolation, not Ladha's own claim; his result is the threshold.) But the direction is clear enough to act on. The honest question is never "how many judges should I run?" It is "how correlated are they, and along which axis?"

So: measure your panel's correlation before you trust its consensus. Engineer diversity against the failure axis that actually matters for your task, not against the vendor logo on the invoice. And treat unanimous agreement among similar models not as strong evidence, but as a reason to look harder. When your AI judges all agree, they may be telling you the answer is obvious. They may equally be telling you they share a blind spot — and from the inside, those two look exactly the same.
