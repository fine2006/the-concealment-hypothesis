# Scheme-Matched Negatives: A Diagnostic Challenge Set for Fallacy Detection

Data release accompanying *The Concealment Hypothesis: What the "None" Class Doesn't Test
in Fallacy Detection* (under review).

This release contains the constructed negatives used in the paper: **matched negatives**
(valid arguments instantiating the same Walton scheme as a paired fallacy) and
**wrong-scheme negatives** (valid arguments on the same topic instantiating a *different*
scheme, the control condition). Both were generated from source fallacies in two public
benchmarks and filtered by an automated cross-model judge.

The set is intended for two uses: as a harder negative class against which to measure a
fallacy detector's false-positive rate, and as a worked example of the construction-quality
audit described in the paper.

---

## Files

| File | n | Condition | Corpus |
|---|---|---|---|
| `cocolofa_task5_final.csv` | 655 | matched | CoCoLoFa |
| `cocolofa_task6_final.csv` | 738 | wrong-scheme | CoCoLoFa |
| `reddit_task5_final.csv` | 387 | matched | Reddit |
| `reddit_task6_final.csv` | 389 | wrong-scheme | Reddit |

### Schemas

**`cocolofa_task5_final.csv`** — matched negatives, CoCoLoFa
- `fallacy_type` — the scheme the negative instantiates, matching the source fallacy's type
- `id` — internal per-type index (not unique across types; not a CoCoLoFa identifier).
  Use `(id, fallacy_type)` as the row key.
- `generated` — the generated matched negative
- `judge_register`, `judge_overall` — judge annotations (see *Judge fields* below)

**`cocolofa_task6_final.csv`** — wrong-scheme negatives, CoCoLoFa
- `source_fallacy` — the type of the source fallacy the item was built against
- `target_type` — the scheme the item actually instantiates (deliberately ≠ `source_fallacy`)
- `id`, `generated` — as above; `source` removed (see *Provenance and licensing*)
- `judge_scheme_switch`, `judge_overall` — judge annotations

**`reddit_task5_final.csv`** — matched negatives, Reddit
- `fallacy_type`, `id`, `generated` — as above
- `span` — the annotated fallacious span in the source comment
- `full_comment` — the full source comment

**`reddit_task6_final.csv`** — wrong-scheme negatives, Reddit
- `source_fallacy`, `target_type`, `id`, `span`, `generated`

### Label vocabularies

CoCoLoFa files and `reddit_task5_final.csv` use the eight scheme names in full
(`appeal to authority`, `appeal to majority`, `appeal to nature`, `appeal to tradition`,
`appeal to worse problems`, `false dilemma`, `hasty generalization`, `slippery slope`).

`reddit_task6_final.csv` retains the source corpus's raw label vocabulary in `target_type`.
The mapping is: `authority` → appeal to authority, `population` → appeal to majority,
`natural` → appeal to nature, `tradition` → appeal to tradition,
`worse_problems` → appeal to worse problems, `blackwhite` → false dilemma,
`hasty_generalization` → hasty generalization, `slippery_slope` → slippery slope.

### Judge fields

`judge_overall` takes values `pass` and `needs_review`. **`needs_review` is a register/style
flag, not a validity flag** — it does not indicate that an item failed scheme fidelity or
conclusion preservation. Items failing those criteria were dropped before this release and
are not included. All items in these files are part of the evaluated sets reported in the
paper; do not filter on `judge_overall` to reconstruct those results.

---

## Measured validity

Each condition carries a human-validated precision estimate. **The two estimates are not
interchangeable — each applies only to its own condition.**

**Matched negatives — 93.1%** (67 of 72 sampled items confirmed valid by majority vote of
three independent annotators; 94.6% on CoCoLoFa, 87.5% on Reddit). The residual 6.9% is
real impurity: those items were retained by the judge but not confirmed valid by human
annotators.

**Wrong-scheme negatives — 77.5%** (31 of 40; 95% CI [62.5%, 87.7%]; 81.2% on CoCoLoFa,
62.5% on Reddit). The impurity here is **concentrated in two target schemes**: appeal to
nature (1 of 5 confirmed valid) and false dilemma (2 of 5). The remaining six target schemes
ranged from 80% to 100%. Users should treat wrong-scheme items with those two target schemes
with corresponding caution.

The asymmetry has a procedural cause. The judge applied five criteria to matched negatives
(register, critical-question satisfaction, absence of over-proving, conclusion preservation,
scheme fidelity) but only three to wrong-scheme negatives (register, on-topicality, scheme
switch) — deliberately *not* critical-question satisfaction, which a wrong-scheme item is
not required to have relative to the source scheme. The wrong-scheme condition was therefore
never certified at critical-question level by the automated pipeline; the 77.5% figure is
the first measurement of that.

Both validation samples were drawn from the *retained* sets released here (not from
discarded items), annotators were blind to any classifier's behaviour, and genuine
benchmark-labelled fallacies were planted in each sample as a discrimination check.
Full protocol in the paper's appendix.

---

## How the items were built

For each fallacious source item, two negatives were generated with **Gemini 2.5 Flash**,
using per-type prompts that state the target scheme and its critical question explicitly and
are calibrated to the source corpus's register. Items are **written afresh** rather than
minimally edited from the source, so topic, length, and register are held comparable while
the source's surface wording is not carried over — this prevents residual lexical overlap
from standing in for scheme content.

A matched negative instantiates the source fallacy's own scheme and answers the critical
question that fallacy leaves unanswered. A wrong-scheme negative instantiates a different
scheme, balanced across the other seven types. Both conditions come off the same pipeline
and differ only in target scheme.

Generated items were then filtered by a **cross-model judge (DeepSeek)** — chosen for being
of a different architecture and training lineage from the generator, so that it does not
share the generator's systematic blind spots. Items failing scheme fidelity or conclusion
preservation were dropped. On the CoCoLoFa batches the judge removed 128 matched items
(124 for scheme drift, 4 for conclusion flips) and 58 wrong-scheme items for failure to
switch scheme. The full per-type prompts and judge criteria are reproduced in the paper's
appendix.

---

## Suggested use

To audit an existing fallacy benchmark, evaluate a detector trained on that benchmark
against the matched negatives for the types it covers, and compare the resulting
false-positive rate to the rate over the benchmark's native "none" class. The wrong-scheme
set is the control: a scheme-specific effect shows up as matched negatives being classified
as the *source* type far more often than wrong-scheme negatives are, not as a difference in
raw flag rates.

Two cautions. First, these items are prototypical scheme instantiations; naturally occurring
arguments are more often elliptical, and the paper does not offer rates measured on this set
as estimates of deployment-time magnitude. Second, the measured impurity above is real —
report it rather than treating the sets as perfectly clean.

---

## Provenance and licensing

Source fallacies are drawn from two public benchmarks:

- **CoCoLoFa** (Yeh et al., 2024) — EMNLP 2024. Crowd-written comments responding to Global
  Voices news articles. The dataset release states no license; the underlying articles are
  CC-BY 3.0.
- **Reddit fallacy corpus** (Sahai et al., 2021) — ACL 2021, released under the **MIT
  License** at `github.com/sahaisaumya/informal_fallacies`.

The `generated` column is original content produced for this work and is released under the
**Creative Commons Attribution 4.0 International License (CC BY 4.0)**. Full license text at
`LICENSES/CC-BY-4.0.txt`; see https://creativecommons.org/licenses/by/4.0/.

**CoCoLoFa source text is not redistributed here.** Because the CoCoLoFa release carries no
license, the two CoCoLoFa files omit the `source` column. The `id` field in these files is an
internal per-type index, not a CoCoLoFa comment identifier, so the generated items cannot be
mechanically re-paired with their source comments from this release alone. The generated
negatives are self-contained for the primary use — measuring a detector's false-positive rate
on scheme-matched valid arguments — which requires the generated text and its scheme label
only. Users needing the source pairing should obtain CoCoLoFa from its own release.

Reddit source text (`span`, `full_comment`) **is** included: the MIT License permits
redistribution, and its license text is reproduced at `LICENSES/sahai-2021-MIT.txt` as that
license requires.

Please cite both source datasets alongside this release.

## Citation

```
[Anonymized for review. Citation to be added on publication.]
```

Please also cite the source corpora:

```
Yeh et al. 2024. CoCoLoFa: A Dataset of News Comments with Common Logical Fallacies
  Written by LLM-Assisted Crowds.
Sahai et al. 2021. Breaking Down the Invisible Wall of Informal Fallacies in
  Online Discussions.
```
