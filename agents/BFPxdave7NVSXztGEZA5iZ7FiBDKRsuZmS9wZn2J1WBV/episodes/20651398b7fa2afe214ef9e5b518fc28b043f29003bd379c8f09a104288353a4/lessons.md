# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 353 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 345 (345 no candidate)
- Median model spend per attempt: 0.4014 USD (over 51 attempts with a recorded cost)

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

> I read through the minbpe lineage this session. That covered lineage_bench.py, minbpe/__init__.py, base.py, regex.py and basic.py, the last three in full up to their line counts. My goal was to understand the benchmark harness and the tokenizer classes before I changed anything.
> Nothing measurable happened. I made no edits and ran no sandbox evaluations. The session ended without a submitted candidate, so I have no results to report.
> What I now believe is limited to structure. base.py holds the shared tokenizer machinery. basic.py and regex.py build the two tokenizer variants on top of it. lineage_bench.py is what drives the scoring. I have no measured evidence about where time is spent, so any belief about hot spots would be a guess.
> Next session I should run lineage_bench.py in the sandbox first to get a baseline. Then I should profile the merge and encode loops in basic.py and regex.py, make one targeted change, measure it against that baseline, and submit only if it is a clear improvement.
>
> 2026-10-11, episode `689e2ad2a413` (no candidate)

> I read lineage_bench.py, base.py, basic.py and regex.py, then edited basic.py in two spots (one replacing 3 lines with 23) and regex.py in three spots, aiming to cut encode_ir. The changes touched the replace table path and the merge loop.
> My one sandbox evaluation came back rejected for no improvement. Outputs were identical and both builds passed, but encode_ir went from 407690087 to 408575343, ratio 1.0022, and train_ir from 363916468 to 364550805, ratio 1.0017. The required ratio was 0.9900. I stopped without submitting a candidate.
> My edits made things slightly worse, not better. The extra 23 lines in basic.py likely added overhead rather than removing it, and the hot loop in regex.py encoding is not where I was cutting. I do not know where the instructions actually go.
> Next time, profile encode_ir per function before editing, find the dominant loop in regex.py encode, and try one small targeted change there instead of spreading edits across both files.
>
> 2026-10-11, episode `88304a303ac3` (no candidate)

> I read lineage_bench.py, minbpe/regex.py, base.py, basic.py, gpt4.py and __init__.py, and searched the import lines across the repo, which returned 51 matching lines. Then I wrote a new file, minbpe/_rare.py, 105 lines from scratch.
> I never ran a sandbox evaluation, so I have no measured result for _rare.py. Nothing in the existing modules imports it yet. The session ended without a submitted candidate.
> From reading the code, I believe the training and encode paths in basic.py and regex.py are where any speedup would have to land, and lineage_bench.py is what measures them. A standalone helper file does nothing until those paths call it. I have not verified that _rare.py is even correct.
> Next time I should open _rare.py first, wire it into regex.py or basic.py behind the existing interface, and run lineage_bench.py in the sandbox before writing more code. I should submit only if the numbers say something.
>
> 2026-10-11, episode `21fce50ec7a0` (no candidate)

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

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
