# Lusser's Law Lies (When Your Agents Share a Brain)

You have probably seen the math that is supposed to scare you.

Your agent does ten things in a row. Each step works 95% of the time. So the whole pipeline works 0.95^10 ≈ 0.599 of the time — about 60%. Push the per-step number down to a more honest 85% and it gets worse fast: 0.85^10 ≈ 0.197, under 20%. Ten competent-looking steps, and four times out of five the pipeline fails somewhere. People call this "the reliability tax" or "compounding failure," and it travels well because it feels true. Anyone who has watched a long agent chain fall over recognizes the shape of it.

It is a good story. It is also built on an assumption nobody says out loud, and once you see the assumption you stop trusting the number — but not in the direction the story implies.

## The hidden word is "independent"

That multiplication — 0.95 × 0.95 × ... ten times — is only valid if the ten steps fail *independently*. Multiplying probabilities is what you do when the events have nothing to do with each other. Coin flips. Independent dice.

But your ten steps are not independent. They are, very often, the *same model* called ten times with different prompts. They share a base. They share training data, tokenizer quirks, a specific way of misreading a specific kind of instruction. When step three trips over an ambiguous date format, step seven — same brain, same blind spot — is more likely to trip over it too, on the same input. The failures are correlated because the failing thing is, underneath, one thing.

This has a name in older, less glamorous fields. Reliability engineers and finance-risk people have long words for a fleet of "diverse" components that all fail the same way at the same time — correlated failure, common-cause failure, and in the systemic-risk literature it goes by "model monoculture." The vocabulary exists. It just hasn't crossed over into how we reason about agents yet.

The interesting part is that "the steps are correlated" does not simply make the number worse. It makes it *wrong in two different directions*, and which direction depends on how your agents are wired together.

## Two topologies, two lies

There are two ways to wire multiple agents, and the independence assumption lies about each of them differently. If you take one thing from this piece, take this: **before you trust any reliability number, ask whether you are looking at a series or a parallel system.**

### Series: a chain, where every step must succeed

This is the pipeline the scary math describes. Retrieve, then plan, then call the tool, then check, then write — and if any link breaks, the whole thing breaks. Success requires *all* steps to succeed.

Here is the part that surprises people. In a pure chain, positive correlation between steps does not lower your average success rate. It raises it.

The quick intuition: if two steps fail together and succeed together, then on a good input they *both* sail through, and 0.95 (they rise and fall as one) beats 0.95 × 0.95 = 0.9025 (two independent hurdles). Perfect correlation collapses ten hurdles back into essentially one. So 0.95^10 ≈ 60% is not the ceiling for a shared-brain chain — it is a *pessimistic floor* for the mean. The independence math is, if anything, too harsh on your average day.

So the compounding-tax story is not lying about the average. It is lying about the *tail*.

When steps are correlated, failures don't sprinkle themselves evenly across runs. They clump. Most days the whole chain glides through because today's input sits in the model's comfort zone and every step is fine. Then some days the input hits the shared blind spot and *everything* falls over at once — step three and step seven and step nine, together, because they were never really independent judges of the input to begin with. Same mean as the independence model, maybe a better mean — but a nastier, lumpier distribution. Your bad days are much worse than 0.95^10 would ever predict, and they arrive in bunches. If you provisioned for "fails 40% of the time, spread out," the correlated reality — smooth stretches punctuated by total collapses — is the thing that pages you at 3am.

The lie in a chain is about variance, not average. The naive number quietly promises you well-behaved, independent failures. You do not get those.

### Parallel: redundancy, where you vote or take the best

Now wire them the other way. Run the same task through N agents and take a majority vote, or best-of-N, or add a verifier that double-checks the worker. Here you are not chaining — you are *hedging*. You are spending compute to buy error-cancellation, on the theory that N sets of eyes catch what one set misses.

In this topology, independence is the *optimistic* assumption, and now it hurts you.

The whole promise of redundancy is that errors cancel. Five voters, each wrong sometimes but wrong about *different* things, should be far more reliable than any one of them. That math also assumes independence — and here the assumption inflates your expectations instead of deflating your average. If your five voters share a brain, they don't make five independent mistakes. They make roughly the *same* mistake, and majority voting on five copies of the same wrong answer just returns the wrong answer with more confidence.

This is where a measurement from the research side earns its keep. When people actually measure how many *effectively independent* judges sit inside a nominal panel, the number is brutal. Kohli et al. (arXiv 2605.29800) measured the φ-based Kish effective number of independent judges across LLM panels: a panel that looks like five judges behaves like about **2.18** independent ones — and when you make the obvious diversity move and keep only the best judge from each model family, stripping the panel to seven nominally diverse judges, it slides down to **1.93**. The strongest judges concentrate their errors on the same items; family diversity alone doesn't buy back independence. Call it the effective number of independent components. You paid for five. You got two. The redundancy you're paying for shrinks exactly where you engineered the diversity in.

That is why "adding more agents stopped helping" is such a common lament. It isn't that redundancy is a bad idea. It's that you were counting heads, and the correct count — the effective one — flatlined around two a while ago. Practitioners in the wild have already noticed the symptom without the name: reports of extra agents barely moving the needle, of unanimous agreement that should be reassuring somehow correlating with confident wrongness. A useful folk rule is already circulating out there — *treat unanimity as suspicious*. That is this same fact wearing work clothes: when your voters share a brain, agreement is cheap and tells you less than you think.

(One thing to be careful about: that 2.18 is a *parallel*, voting-panel number. Do not sprinkle it onto a chain's per-step reliability curve — different structure, different question. The two topologies don't share a correction factor.)

## Why the shared base makes both worse

Both failure modes trace to the same root. A decade ago your "ten steps" might have been ten genuinely different pieces of software written by different teams, and independence was a half-decent approximation. Today the ten steps, or the five voters, are frequently one base model in ten costumes. Fine-tuning on top doesn't buy back much independence, either — measurements of co-failure across different fine-tunes of the same base still come out high. The costumes differ. The brain doesn't. Independence was the thing you were implicitly buying, and a shared base is exactly the thing that stops selling it.

## What to actually do

The fix is not "trust the compounding math" and it is not "panic." It is three unglamorous moves.

**Know which topology you're in.** Draw it. If every component must succeed, you're in series — and your worry is clustered catastrophic failures, not the average. If you're voting or verifying, you're in parallel — and your worry is that your effective N is a fraction of your nominal N.

**Stop trusting the naive number as a point estimate.** 0.95^10 = 60% is not a prediction; it's what you'd get in a world where your steps were strangers. They aren't. In a chain, expect a better mean and a much worse tail. In a panel, expect fewer effective voters than you're paying for.

**Measure the correlation instead of assuming it away.** This is the part almost nobody does, and it's the part that turns the argument into engineering. You don't need to derive anything fancy: you need to watch, per item, how often your components fail *together* rather than how often each fails alone. Co-failure — the rate at which the whole panel or the whole chain goes down on the same input — is a quantity you can monitor in production, one item at a time, and it's the number that actually predicts your bad days. Log it. Watch it drift. When it climbs, your redundancy is quietly evaporating and no amount of adding-more-agents will bring it back.

The compounding-failure math isn't wrong because the arithmetic is wrong. It's wrong because it assumes your agents are strangers, and increasingly they're the same mind wearing different hats. Name that — correlated errors, effective number of independent components — and the reliability curve you should actually plan around comes into focus. It's a better mean than you feared, a worse tail than you were promised, and a redundancy budget that stopped compounding a long time before your headcount did.
