# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 350 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 342 (342 no candidate)
- Median model spend per attempt: 0.3927 USD (over 48 attempts with a recorded cost)

## Target perf:encode_ir

- Attempts: 4 (3 accepted, 1 rejected)
- Best accepted effect: 5.07% (ratio 0.9493), episode `301cb7037e4c`
- Accepted effects: 5.07%, 1.38%, 1.31%
- Why candidates failed: no_improvement (1)
- Files changed by accepted candidates: `minbpe/basic.py`, `minbpe/gpt4.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/basic.py`
- Median model spend per attempt: 0.3470 USD (over 4 attempts with a recorded cost)

## Target perf:train_ir

- Attempts: 4 (2 accepted, 2 rejected)
- Best accepted effect: 12.13% (ratio 0.8787), episode `7ca2f6293ab1`
- Accepted effects: 12.13%, 1.22%
- Why candidates failed: apply_conflict (1), stale_conflict (1)
- Files changed by accepted candidates: `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.3721 USD (over 4 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I read lineage_bench.py, minbpe/__init__.py, regex.py, base.py and basic.py in full, and searched for import lines (47 matches) to see what the package pulls in. Then I started changing minbpe/regex.py. I replaced lines 363-364 with 7 lines and line 35 with 30 lines.
> I never ran a sandbox evaluation, so I have no measured result for either edit. The session stopped without a candidate. Nothing was submitted, and the edits are untested.
> What I believe now: regex.py is where the work would land, and the benchmark harness in lineage_bench.py is short enough to understand quickly. I made two edits in that file without checking whether they help or even run. That was the mistake. I don't know if they are correct.
> Next time I will run the benchmark on unmodified code first to get a baseline. Then I will reapply the two regex.py changes one at a time, evaluating after each, and submit only if a change measurably beats the baseline.
>
> 2026-10-11, episode `73f7502440a9` (no candidate)

> I read lineage_bench.py, base.py, basic.py and regex.py, then tried to cut encode cost in basic.py by skipping replace-table scans when the pair's tokens are not present in the sequence. I also made small edits in regex.py. The guard reported 2 files and 36 lines.
> My one sandbox evaluation built fine and gave identical outputs, but it was rejected for no improvement. encode_ir went from 421193809 to 421567485, a ratio of 1.0009. train_ir went from 367393060 to 368085230, a ratio of 1.0019. The required ratio was 0.9900. I gave up and submitted nothing.
> I now believe the replace loop is not a meaningful share of encode_ir. Most of the cost looks like fixed startup and regex work, but I did not profile that, so it is a guess.
> Next time I should profile encode_ir first, before editing anything. If regex compile and caching dominate, I would look at the compile and _cache path in regex.py.
>
> 2026-10-11, episode `27eda722babe` (no candidate)

> I read the benchmark harness lineage_bench.py and the core of minbpe: regex.py, base.py, basic.py and __init__.py. I then made one edit in minbpe/regex.py, replacing lines 363-370 (8 lines) with 35 new lines. That is near the end of the file, in the encoding path of the regex tokenizer.
> I never ran a sandbox evaluation on that edit, so I have no measured result for it, correct or not. The session ended without a candidate. Nothing was submitted.
> What I believe now is limited to what reading showed. The regex tokenizer's encode tail is where I thought a change was worth making, and the benchmark lives in lineage_bench.py. Whether my 35 lines are faster or even equivalent to the original 8 is unknown.
> Next time I should start by running the benchmark on unmodified code to get a baseline. Then I would reapply or rewrite the regex.py change, check its output against the original encoder, and measure it. I should submit only if it holds up.
>
> 2026-10-11, episode `6bdf32b17c3e` (no candidate)

> I worked on the minbpe lineage aiming at encode_ir. I read lineage_bench.py, base.py, basic.py, regex.py and gpt4.py, then searched for the replace and chunk tables. I edited basic.py in two places, collapsing four lines near 211 into one and adding a 24 line block near 233. In regex.py I changed lines 18, 29-30 and 311-314.
> My one sandbox evaluation was rejected for no improvement. The guard passed with 2 files and 37 lines, both builds were ok, and the outputs were identical. encode_ir went from 363199897 parent... no, from 414203141 to 418563614, ratio 1.0105, against a required 0.9900. train_ir went from 363199897 to 363825525, ratio 1.0017. I stopped without submitting a candidate.
> My belief now is that these table-based edits add overhead and save nothing on the encode path. The existing loop is already lean, so small restructuring here costs instructions.
> Next time I will profile encode to find the actual hot spot before editing. I will also look at the merge lookup inside the per-chunk loop rather than the tables.
>
> 2026-10-11, episode `9f97a40f2881` (no candidate)

> I read lineage_bench.py, minbpe/__init__.py, regex.py, base.py, basic.py and gpt4.py in the minbpe lineage. I also ran one search for import lines, which matched 51 lines. I made no edits and ran no sandbox evaluations.
> The session ended with no candidate submitted. I gave up because I could not find a change I could confirm as worth at least 1% within my budget.
> My reading is that the merge, encode and train paths are already heavily optimized. What remains looks like fixed cost: interpreter startup, the regex import, compiling the GPT-4 pattern, and the benchmark's own corpus generation. Some of that sits outside the editable paths. For the rest, I could not tell whether it matters without profiling.
> Next time I should profile first, before reading the whole codebase, so I can see where time actually goes. If the regex import or the pattern compile shows up as a real share, I could try deferring or caching it. I should save budget for at least one edit and one evaluation.
>
> 2026-10-11, episode `68b708b666bb` (no candidate)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
