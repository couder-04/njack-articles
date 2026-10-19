# Suggested additions

These stayed out of **For the Architecture Hungry Enthus** so that section stays a reconstruction of the forward pass. Each item below is paste-ready, with a home in the article.

The source is Bijit Ghosh, “Inside Jev, Architecture of a Decision Model” (17 Sep 2026). That post is explicit that the architecture and the RLCD recipe are undisclosed, and that the 10,000-call investigation supports a reconstruction rather than an implementation.

## 1. Sharpen “2. Confidence”

The current paragraph says an LLM will not show how unsure it was. The stronger measurement point is different: even a generated “91% confident” is the probability of those characters, not a 91% chance the label is right.

Paste after the existing confidence paragraph:

> A generated “91% confident” does not establish a 91% chance of being right. That number, when you can see it at all, is about how likely those characters were. Jev’s output is the distribution over the answers you allowed, which is the quantity a threshold can actually use.

## 2. A short “Act on the probability” block

This is decision theory, so it sits better after the architecture section or inside **Where Jev Fits**, not inside the forward-pass explanation.

> Suppose a needless escalation costs 1, and missing an urgent case costs 9. Let p be the probability the case is urgent. Escalating then has expected cost 1 − p. Holding it back has expected cost 9p. Escalate when 1 − p < 9p, which is p > 0.1. That arithmetic assumes a correct action costs nothing, and that p is actually calibrated. If either assumption fails, the threshold is theater.

Worth one sentence under it: start on a bounded routing workflow, store each prediction next to the outcome, and count mistakes among the cases you handled automatically before widening the scope.

## 3. Two limitation bullets

Add under **Limitations and Ongoing Research**:

- **Option sets move the answer.** Adding or reordering choices can change the decision. Test that equivalent questions still agree.
- **Benchmark calibration is not your calibration.** An MMLU calibration error of 0.031 is about a 3.1 point average gap on that test’s bins, mostly at high confidence. Measure probability quality, confidence against accuracy, and the error rate above your automation threshold on labeled examples from the workflow you will actually run.

## 4. Further reading entry

Add to the numbered list:

> 7. **Inside Jev, Architecture of a Decision Model**, Bijit Ghosh (17 Sep 2026): a public reconstruction of the shared-context forward pass, isolated question branches, direct probability readouts, and RLCD. Treat the mechanism as a hypothesis. The post says the implementation is undisclosed.

## 5. Optional figure

A diagram would do more work than another paragraph, placed under **One context, many questions**:

- One shared prefix (the context), keys and values computed once per layer.
- Three suffixes beside it (queue, urgency, human review), each attending to the prefix and to itself.
- A mask drawn so no suffix can see another suffix.
- Three small readouts, each a probability bar, with no generated tokens after them.
- A caption stating the token saving as “about QS down to S for the shared state,” and that each branch still attends over S.

## 6. One line the architecture section can live without

If the enthusiast section feels long, the temperature-scaling sentence can move here and out of the article. It matters for readers who will recalibrate logged scores, and it is not part of the forward pass:

> If you have labeled outcomes, temperature scaling rescales the raw scores so the probabilities get sharper or softer without changing which answer ranks first. That is a post-hoc fix. It is not evidence about RLCD.
