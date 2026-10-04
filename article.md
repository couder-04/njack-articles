# Not Just Another Switch Statement

### A switch statement, but smarter and cheaper

![Illustration of overkill: using a frontier LLM for a simple classification decision](images/hero-toast-analogy.png)

## You're Paying a 5-Star Chef to Toast Bread

Somewhere in someone's workflow automation, a 200 **billion** parameter model is deciding whether an email is spam or a lead. It's the same model that can debug your code, explain a topic the night before an exam, and summarize a 100-page PDF. And someone has asked it to say one word.

That's like hiring a 5-star chef to press the button on the toaster. The toast comes out. You might want to check the invoice.

Route to agent A or agent B. Lead or spam? Score this transaction's fraud risk. None of these is reasoning. They're closed-set classification problems with a fixed list of outputs, handed to a model trained and priced to write essays.

**Not every natural-language problem is a reasoning problem. Some are decisions.**

![Diagram showing an LLM used as a closed-set classifier](images/llm-as-classifier.png)

## It's Just a Label, Not a Reasoned Answer

Strip away the prompt and graph engineering and most of these calls look identical:

> *"Given this support ticket, respond with exactly one of: billing, engineering, sales."*

That's a switch statement wearing Doctor Strange's cape. A frontier LLM produces that label one token at a time, running the full network at every step, to deliver a few bits of information. It's a jet engine powering a light switch.

![Comparison highlighting frontier LLM cost and compute overkill for labeling](images/overkill-frontier-model.png)

## Three Problems With Using LLMs as Decision Engines

### 1. Latency

Chat models are tuned to be fast enough for a human watching a chat window, not for the hot path of a backend. Multiply one slow call by every row in a batch job and it becomes your bottleneck. (TypeSafe reports 3 to 329 seconds across the frontier models in its comparison. That's vendor-reported, not independent.)

### 2. Confidence

An LLM will almost always give you an answer. It rarely gives you a calibrated sense of how sure it is. It will misroute the 5% it doesn't understand with the same serene tone it uses for the other 95%.

### 3. Edge Cases

Prompt-based routing looks great with three categories. With dozens of overlapping teams, the prompt swells into a novel, and a bigger prompt often adds confusion instead of removing it. Classification and generation are different jobs, and treating them as one gets brittle fast.

![Visual summary of latency, confidence, and edge-case problems with LLM decision engines](images/latency-confidence-edges.png)

## The Industry Already Half Knows This

LLM routers exist because everyone building these systems rediscovered that picking A or B doesn't need a frontier model. If cheap classifiers make sense for routing, why stop there? Ticket tagging, intent detection, and moderation often still run on frontier models elsewhere in the same stack. It's a fleet of Ferraris doing the school run.

## The Solution: Change the Computational Primitive

| Workload | Shape | Fit |
|---|---|---|
| **Reasoning** | Open-ended, explanatory, generative | LLM |
| **Decision-making** | Predefined output space, classification, routing | Decision model |

TypeSafe AI's **Jev** is what they call a "System One Model." It never generates text. You send it a piece of context plus named questions, and it returns typed, probabilistic answers in one parallel pass. There are three question types:

- **Choice** picks one option from a set you define and returns a probability for every option.
- **Score** rates the input on an ordered rubric, and can land between levels.
- **Noul** answers yes/no as a probability.

The output fits an `if` statement by construction. That guarantees the *format*, not the *correctness*, and the confidence number is what lets you act on the difference.

![Jev producing typed probabilistic decisions from a predefined schema](images/jev-typed-decisions.png)

TypeSafe claims 20-200x faster and 40-400x cheaper than frontier LLMs, at about $0.042 per million input tokens with output free. Independent tests confirm the direction but report different multiples by workload. Check current pricing before you build on it.

![Side-by-side contrast of reasoning workloads versus decision workloads](images/reasoning-vs-decision.png)

## What People Actually Built

Within days of launch, community builders had shipped dozens of demos (a snapshot of 74 posts, collected by the [awesome-jev-use-cases](https://github.com/walidboulanouar/awesome-jev-use-cases) list). These are self-reported numbers from X posts, but they show where the model is a good fit.

**Bulk judgment at near-zero cost**

- 500 emails triaged for 3.5 cents.
- Every ran 21 questions over 37 documents: 1,709 judgments for under a cent.
- One builder ran eight questions on each of 3,282 posts for $0.1282.
- 1,018 AI papers grouped by topic for $0.08.

**Real-time, in the loop**

- A second-hand shopping agent decides on each listing in about 406 ms.
- Jev plays Slay the Spire 2 at 0.7 seconds per move.
- Live post scoring about half a second after you stop typing.
- A real-time ad blocker and a slop detector that scores posts as you scroll.

**Routing and gating**

- Model routers that send each request to the cheapest model that can handle it.
- Skill and tool selection for coding agents.
- Guardrails that screen messages going in and out of an LLM app.

## The Surprise: Most Winners Pair Jev With an LLM

The list notes that the four most-liked demos (a Claude Code compaction plugin, a browser agent, an ad teardown, a Mac voice assistant) don't use Jev to generate text at all. The pattern is a **fast decision layer in front of an expensive generator**.

One fraud-detection demo shows it well. Jev classifies 100 emails in 1.42 seconds, and only the *uncertain* ones go to a larger model. Jev decides, and the LLM explains or handles the hard cases. That's the architecture the toast analogy points to.

## Four Patterns Worth Stealing

TypeSafe's own docs describe patterns the working demos converge on:

1. **Speculative fan-out.** Send many questions about one input in a single call and let your code decide which answers matter. TypeSafe's parallel-questions cookbook reports 12.2x cheaper and 10.0x faster than asking one at a time, with no change in answers.
2. **Confidence-gated routing.** The answer says *what*, and the confidence says *whether to act*. Act above a threshold, send low-confidence cases to a bigger model or a human.
3. **Composite scoring.** Break a fuzzy judgment ("is this a good post?") into atomic scores and combine them with weights **you** control in code.
4. **Intent routing.** Classify incoming requests, then hand each to a deterministic handler.

## Where Jev Fits

| Task | Approach |
|---|---|
| Sentiment → positive / negative | Decision model |
| Intent → billing / support / sales | Decision model |
| Agent or model routing | Decision model |
| Moderation, spam, PII screening | Decision model |
| Re-ranking and relevance filtering | Decision model, with realistic expectations (see below) |
| "Explain why this is spam" | LLM |
| Code generation | LLM |
| Complex reasoning, open-ended chat | LLM |

## Limitations and Ongoing Research

- **Weak at counting and dates.** Do these in code, then ask Jev about the result. TypeSafe's extraction cookbooks use regexes to find candidates, then let Jev pick the right one.
- **Sensitive to clutter.** Unrelated input can reduce accuracy.
- **Decisions only.** You need predefined options. Anything needing prose needs an LLM.
- **Typed ≠ correct.** Valid output can still be the wrong decision.
- **Hard tasks stay hard.** In TypeSafe's own legal re-ranking cookbook, top-1 accuracy rose from 5% to 18%. That's a real gain but nowhere near solved, so measure on your own data.
- **Numbers are vendor- or self-reported.** Treat demo costs and speeds as directional.
- **Open alternatives exist.** Community projects like openjev, kev, decider, and SemIf explore local and open-weight versions, and independent calibration audits are starting to appear.

![Overview of Jev limitations and areas of ongoing research](images/limitations-overview.png)

## The Bigger Idea

We keep using generative models for problems that are fundamentally decisions. Reasoning and decision-making are different workloads, even when they arrive as the same natural-language input. Matching the primitive to the job is the engineering move. Jev is one concrete way to do it.

A practical first step: find one LLM call in your stack whose output you immediately `if` on. Rewrite it as a typed question and compare cost, latency, and accuracy on a labeled sample. Then add a confidence threshold and send the uncertain cases up to the big model.

## How to Try It

Jev is available through TypeSafe and integrations including [OpenRouter](https://openrouter.ai/docs/guides/community/jev), [Vercel AI Gateway](https://vercel.com/ai-gateway/models/jev), and [Cloudflare Workers AI](https://developers.cloudflare.com/workers-ai/). Direct access and docs live at [madewithjev.com](https://madewithjev.com/what-is-jev) and [TypeSafe's documentation](https://docs.typesafe.ai/introduction/quickstart).

## Further Reading

1. **[What is Jev?](https://madewithjev.com/what-is-jev)**: the basic idea, and what Jev is designed for.
2. **[System One Concepts](https://docs.typesafe.ai/concepts/system-one)**: how System One models differ from traditional LLMs.
3. **[Primitives](https://docs.typesafe.ai/primitives)**: the three question types: `noul`, `choice`, and `score`.
4. **[Quick Start](https://docs.typesafe.ai/introduction/quickstart)**: make your first Jev API call.
5. **[Confidence Guidance](https://docs.typesafe.ai/confidence)**: how to use Jev's confidence scores and thresholds.
6. **[awesome-jev-use-cases](https://github.com/walidboulanouar/awesome-jev-use-cases)**: community demos, patterns, and documented limits.
