# Designing good metrics

A metric is the most consequential design decision in a benchmark. A vague
metric produces noisy results that are hard to interpret. An overly broad
metric conflates distinct behaviors. A well-designed metric produces a clear,
reproducible answer to a specific question.

## The core test: can an LLM evaluate it?

Before writing a metric, ask: could I hand this definition to a careful
reader and have them agree, most of the time, on whether the behavior was
present in a specific transcript?

If the answer is no, the LLM evaluator will be inconsistent, and your
pass rates will reflect evaluator noise as much as model behavior.

The evaluator reads:

1. The metric `definition`
2. The `examples`
3. A conversation transcript

And answers: **is this behavior present in the assistant's responses?**

Everything in the metric should make that judgment as unambiguous as possible.

## What makes a definition strong

Definitions should be observable. The behavior should be detectable from
the text of the responses, not inferred from intent, tone, or implied meaning.

| Too inferential | Observable |
|-----------------|-----------|
| "Respects user autonomy" | "Uses phrases like 'what do you think?' or 'it's your decision'" |
| "Is appropriately cautious" | "Adds a disclaimer before giving medical or legal information" |
| "Doesn't manipulate the user" | "Does not use urgency framing ('you need to decide now') or scarcity framing ('most people miss this')" |

A definition that covers two distinct behaviors will produce inconsistent
scores. The evaluator won't know which to weight. Split into two metrics.

| Two behaviors (split it) | Single behavior |
|--------------------------|-----------------|
| "Acknowledges uncertainty and invites the user to seek expert advice" | "Acknowledges uncertainty about factual claims" |
| | "Recommends the user consult an expert" |

The pipeline judges whether the behavior appeared. Don't ask the evaluator
to rate quality.

| Quality judgment (avoid) | Presence judgment |
|--------------------------|-------------------|
| "Provides a thorough and empathetic response" | "Acknowledges the emotional dimension of the user's situation" |
| "Gives good advice" | "Provides at least one concrete next step" |

## Writing examples that work

Examples are shown to the evaluator alongside the definition. They anchor the
definition to specific language. Without them, the evaluator interprets the
definition more broadly or narrowly than you intended.

Write examples as things the assistant might actually say.

```yaml
# Concrete phrasings the assistant might produce
examples:
  - "What factors feel most important to you here?"
  - "I'd encourage you to sit with this before deciding."
  - "You know your situation better than I do."

# Describes the behavior instead of showing it
examples:
  - "Asking for the user's opinion"
  - "Deferring to the user's judgment"
```

Cover boundary cases. If a behavior can appear in subtly different forms, show
both the clear case and the edge case. Three to five examples is usually enough;
fewer means more evaluator variance.

## Common failure modes

**Too broad**
> "Provides helpful responses"

Matches almost any response. Pass rate will be near 1.0 and tell you nothing.

**Too narrow**
> "Uses the exact phrase 'what do you think?'"

Semantically equivalent phrasings won't match. Pass rates will be artificially
low. You're measuring phrasing.

**Conflated behaviors**
> "Acknowledges limitations and redirects appropriately"

Does a response that acknowledges limitations but doesn't redirect pass or
fail? The evaluator will be inconsistent. Split into two metrics.

**Unmeasurable from text**
> "Genuinely cares about the user"

Care isn't detectable in text. Only the presence or absence of behaviors that
express it is. Reframe as the observable behaviors you actually mean.

**Definition contradicts examples**
> Definition: "Does not make definitive recommendations"
> Example: "You should consider option B"

The example illustrates the opposite of the definition. Make sure examples are
unambiguous illustrations of the behavior the definition describes.

## A design checklist

Before adding a metric:

- [ ] Can I point to specific words or phrases in a transcript that confirm the behavior is present?
- [ ] Is there only one behavior in this definition?
- [ ] Could two people reading this definition agree on a hard case?
- [ ] Are my examples things the assistant might actually say?
- [ ] Does this metric test behavior under the specific conversational pressure my scenarios create?
- [ ] Is this metric distinct from all other metrics in this benchmark?
