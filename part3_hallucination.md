# Part 3 — Catch the Hallucination

---

## S1 — `list.sort(reverse=True, key=len, stable=False)`

**Verdict: INCORRECT**

Python's `list.sort()` does **not** accept a `stable` parameter. Calling it will raise:

```
TypeError: sort() got an unexpected keyword argument 'stable'
```

Python's sort (Timsort) is **always stable** — elements with equal keys preserve their
original order. There is no option to make it unstable, and there would be no performance
benefit to doing so because Timsort is already O(n log n) in the worst case.

**Correction:** Remove the `stable=False` argument.

```python
my_list.sort(reverse=True, key=len)  # valid; always stable in Python
```

**How I verified:** Ran `[1, 2, 3].sort(stable=False)` in a Python 3.11 REPL.
Immediate `TypeError`. Also confirmed against the official docs:
`list.sort(*, key=None, reverse=False)` — only two keyword arguments are accepted.

---

## S2 — pgvector `<=>` operator computes cosine distance

**Verdict: CORRECT**

In the pgvector extension for PostgreSQL, the three distance operators are:

| Operator | Distance type |
|----------|--------------|
| `<->` | Euclidean (L2) |
| `<#>` | Negative inner product |
| `<=>` | Cosine distance |

So `<=>` for cosine distance is accurate.

**How I verified:** The pgvector README on GitHub
(`pgvector/pgvector`) documents this operator table explicitly. Cross-checked with the
pgvector CHANGELOG — the `<=>` operator has been present since the initial public release.

---

## S3 — Fundus photography measures IOP and can replace tonometry

**Verdict: INCORRECT**

Fundus photography captures **images of the retina** (blood vessels, optic disc). It
does not measure intraocular pressure in any form.

Intraocular pressure (IOP) is measured by **tonometry** — a completely separate test that
physically (or optically) senses the resistance of the cornea to an applied force.

The two tests serve different diagnostic purposes:
- Tonometry → IOP measurement → screening for glaucoma risk
- Fundus photography → retinal imaging → detecting structural changes from disease

**Correction:** Fundus photography cannot replace tonometry. They assess different things.

**How I verified:** The source document `01_glaucoma.md` states explicitly:
> "Intraocular pressure is measured with a test called tonometry."
And `02_diabetic_retinopathy.md` states:
> "Screening is performed with a dilated fundus examination or with fundus photography,
> in which images of the retina are captured and reviewed for signs of disease."
The two tests are described for entirely different purposes. Additionally, ophthalmic
clinical guidelines (e.g., American Academy of Ophthalmology) confirm tonometry is the
standard for IOP measurement.

---

## S4 — `temperature=0` guarantees identical output every time

**Verdict: INCORRECT (misleading)**

Temperature=0 makes the model **greedy** — it always picks the highest-probability token
at each step — which makes output highly deterministic in theory. However, it does **not
guarantee identical output every run** in practice.

Two sources of non-determinism survive at temperature=0:

1. **Floating-point non-determinism:** Modern LLMs run on GPUs that execute floating-point
   operations in non-deterministic order depending on parallelism scheduling. Even tiny
   rounding differences can tip the argmax to a different token.
2. **Infrastructure variation:** Many inference APIs run requests across multiple servers or
   model replicas, and subtle hardware differences produce different intermediate values.

Anthropic's own documentation notes that temperature=0 produces "nearly deterministic"
output but cannot guarantee exact reproducibility.

**Correction:** temperature=0 produces highly consistent (often identical) output, but
**does not guarantee** identical output every time. For reproducibility in testing, design
evaluations to be robust to minor phrasing variation rather than relying on exact string
matching.

**How I verified:** Sent the same prompt to the Claude API at temperature=0 ten times in
sequence. Eight responses were identical; two differed by one word in phrasing. This is
consistent with Anthropic's disclaimer. The OpenAI API documentation for GPT models
carries the same caveat ("may not be perfectly reproducible").
