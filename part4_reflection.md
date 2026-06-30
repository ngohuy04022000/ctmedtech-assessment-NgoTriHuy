# Part 4 — Judgment & Reflection

---

## Q1 — Describe a real situation where an AI gave you a wrong or misleading answer. How did you notice, and what did you do?

While integrating a third-party payment library, I asked an AI assistant to generate the
boilerplate for webhook signature verification. It confidently produced code that called
`hmac.compare_digest(payload_signature, expected_signature)` — but it had the argument
order reversed. The library's documentation specified that the first argument must be the
**received** signature and the second the **computed** one; swapping them doesn't cause
an error or a test failure when both values are correct, but it silently passes when
the received signature is empty (because `compare_digest("", "")` is `True`).

I noticed because I wrote a dedicated negative test: I sent a webhook with a deliberately
wrong signature and the verification passed when it should have failed. The AI's code
had passed all the happy-path tests, so without that negative test it would have shipped.

What I did: went back to the official library docs, found the discrepancy, fixed the
argument order, and added the negative test permanently to the suite. I also added a
comment explaining why the argument order matters — a future reader would not see the
danger from the function name alone.

---

## Q2 — In your Part 1 solution, name one place where you deliberately did not use AI — and explain why.

I wrote the system prompt in `src/generator.py` myself without using AI for the first
draft.

The system prompt is the primary mechanism for the two hardest constraints in this
assessment: citation format and refusal behaviour. If I had asked an AI to write it, the
output would have been a reasonable but vague template — something like "always cite your
sources and say you don't know if unsure." That phrasing is not testable. I need a
specific, consistent refusal sentence (`REFUSAL_PHRASE`) that I can match programmatically
in the evaluator and in the unit tests.

Precision here is the whole point. I knew exactly what I needed, and writing it myself
took five minutes. Involving an AI would have added a review-and-edit cycle without
shortening the total time.

---

## Q3 — How do you keep AI-generated code maintainable and trustworthy when shipping to production with a team?

Three concrete things I do:

**Read every line before committing.** AI code that "looks right" often contains
assumptions that are wrong for our context (wrong API version, wrong argument order, wrong
edge-case handling). I read the diff the same way I would read a junior engineer's PR:
line by line, not just skim.

**Write the tests before reviewing the implementation.** If I know what the code is
supposed to do, I write at least one test for the edge case most likely to be wrong — the
empty input, the over-limit input, the adversarial input. If the AI-generated code passes
all happy-path tests but fails the edge case, I know exactly where to look.

**Name what the AI generated.** In commit messages and PR descriptions I note which parts
came from AI output. This is not shame — it is information. A reviewer who knows a block
is AI-generated pays closer attention to its assumptions; a future maintainer who has to
change it knows to re-verify rather than trust inertia.

---

## Q4 — You're working with patient / medical data. What are your personal rules for using AI tools responsibly in that context?

**No real patient data in prompts — ever.** If I need to test something against realistic
data, I generate synthetic records or use officially anonymized datasets. I treat the AI
API endpoint as a third-party service with an unknown retention policy, because that is
what it is.

**AI output is a suggestion, not a decision.** In a clinical context, an AI's risk
assessment (Low / Medium / High) is an input to a clinician's decision, not a replacement
for it. I would never design a system where the AI output triggers an irreversible action
(sending a referral, updating a diagnosis) without a human in the loop.

**Audit trail first.** Every AI call that touches patient-adjacent data should be logged:
input, output, model version, timestamp. Not for debugging — for the patient. If a
decision is ever questioned, the clinician and the patient's legal team need to be able
to reconstruct what the system saw and said.

**Be explicit about what the model does not know.** The RAG refusal logic in Part 1 is an
example of this: better to say "not in the documents" than to hallucinate a treatment
protocol. In medical contexts the cost of a confident wrong answer is much higher than
the cost of an honest "I don't know."
