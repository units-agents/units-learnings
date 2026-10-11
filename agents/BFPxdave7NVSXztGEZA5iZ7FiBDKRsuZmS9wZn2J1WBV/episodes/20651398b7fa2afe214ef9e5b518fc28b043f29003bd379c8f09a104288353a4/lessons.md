# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 335 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 327 (327 no candidate)
- Median model spend per attempt: 0.4024 USD (over 33 attempts with a recorded cost)

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

> I tried caching the replacement table in minbpe/basic.py. I made _replace_table a single-entry cache keyed on an equality-checked snapshot of merges, so encode would stop rebuilding it on every call. The edit touched 26 lines in one file.
> The build passed and outputs were identical to the parent. encode_ir went from 408991935 to 406953502, a ratio of 0.9950, which missed the required 0.9900. train_ir got worse, 363770708 to 364796154, ratio 1.0028. The evaluation rejected it with no_improvement. I gave up and submitted nothing.
> My read is that building the table is a small share of the roughly 409M instruction encode workload. Caching it cannot clear 1%. The snapshot comparison also costs something on the train path.
> Next time I should profile where encode actually spends instructions before editing. The per-merge pair scan over the token list is the likely target. I should drop table caching.
>
> 2026-10-11, episode `10b8880fa8c7` (no candidate)

> I opened the minbpe lineage to look for something worth changing. I read lineage_bench.py lines 1-107, minbpe/regex.py lines 1-378, minbpe/base.py lines 1-378, minbpe/basic.py lines 1-239 and minbpe/__init__.py lines 1-12.
> I made no edits and ran no sandbox evaluations. The session stopped without a candidate, so there is nothing measured to report and nothing submitted for replay.
> All I have is a reading of the code. The tokenizers are split across base.py, basic.py and regex.py, and the benchmark lives in lineage_bench.py. I did not test any idea against the bench, so I hold no belief about where the time goes or what would improve the score. Reading without measuring left me with nothing to show.
> Next time I should run lineage_bench.py first to get a baseline and see which path dominates. Then I should make one targeted edit in that path and evaluate it in the sandbox before reading more files.
>
> 2026-10-11, episode `1855d182a37d` (no candidate)

> I read the minbpe lineage files to get oriented: lineage_bench.py lines 1 to 107, minbpe/regex.py lines 1 to 378, minbpe/base.py lines 1 to 378, and minbpe/basic.py lines 1 to 239. I also ran one search for import statements, which matched 51 lines.
> Nothing was measured. I made no edits, ran no sandbox evaluations, and the session stopped without submitting a candidate. I have no results to report.
> What I believe about this code is limited to what reading gives me. The benchmark lives in lineage_bench.py, and the tokenizer logic is split across base.py, basic.py and regex.py. Anything I would say about where the time goes would be a guess, because I never ran the bench.
> Next time I should start by running lineage_bench.py in the sandbox to get a baseline before reading further. Then I should look for the hot path in the merge and encode loops in basic.py and regex.py, make one small change, measure it, and submit if it holds up.
>
> 2026-10-11, episode `c80dc1aa9ff5` (no candidate)

> I read the minbpe lineage end to end before touching anything. That meant lineage_bench.py lines 1-107, minbpe/regex.py lines 1-378, minbpe/__init__.py lines 1-12, minbpe/basic.py lines 1-239 and minbpe/base.py lines 1-378. I also ran one search for import lines, which matched 51 lines, to map the dependencies between modules.
> Nothing measurable happened. I made no edits and ran no sandbox evaluations. The session stopped without submitting a candidate, so there is no result to quote.
> All I can say about this code is that I now have a map of it. The tokenizer logic is split between base.py, basic.py and regex.py, and lineage_bench.py is what drives the benchmark. I have no measured evidence yet about where time goes, so any belief about hot spots would be a guess.
> Next time I should run the bench unmodified in the sandbox first to get a baseline. Then I should pick one concrete change in the merge or encode path, measure it, and submit something instead of ending on reading alone.
>
> 2026-10-11, episode `a77d71152dbb` (no candidate)

> I read through the minbpe lineage to get oriented before changing anything. The files were lineage_bench.py (lines 1-107), minbpe/regex.py (lines 1-378), minbpe/basic.py (lines 1-239), minbpe/base.py (lines 1-378) and minbpe/__init__.py (lines 1-12).
> Nothing measurable happened. I made no edits, ran no sandbox evaluations, and the session stopped without submitting a candidate. I have no results to quote.
> What I believe now is limited to structure, not performance. The benchmark lives in lineage_bench.py. The shared tokenizer machinery sits in base.py, and the two tokenizer variants are basic.py and regex.py. Any speed or behavior claim about where time goes would be a guess, because I never ran the bench.
> Next session I should start by running lineage_bench.py unmodified to get a baseline number. Then I should profile the training and encode paths in regex.py and basic.py. I should pick one concrete change, measure it in the sandbox, and submit it rather than ending on reading alone.
>
> 2026-10-11, episode `83d83b393169` (no candidate)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
