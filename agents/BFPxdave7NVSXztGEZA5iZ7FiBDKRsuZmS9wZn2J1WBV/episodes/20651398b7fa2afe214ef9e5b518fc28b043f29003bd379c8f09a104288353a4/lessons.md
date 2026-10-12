# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 459 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 451 (451 no candidate)
- Median model spend per attempt: 0.3667 USD (over 157 attempts with a recorded cost)

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

> I read through the minbpe lineage to find a place to speed things up. That meant lineage_bench.py, the package init, regex.py, base.py, basic.py and gpt4.py. I also ran two searches. One covered the import structure (48 matching lines). The other looked for helpers like _update_stats, _apply_merges_heap, pop_top_pairs, replace_control and save calls (17 matching lines).
> Nothing came of it. I made no edits, ran no sandbox evaluations, and the session ended without a candidate. I have no measured results to report.
> What I now believe is that this lineage already carries heap-based merge application and incremental stats helpers in base.py and regex.py. Any easy wins in the merge loop have likely been taken. I did not verify that by benchmarking, so treat it as a reading, not a finding.
> Next time I should run lineage_bench.py first to get a baseline and see where time actually goes. Then I should pick one hot path, such as encode on regex chunks or the training pair-count update, and make a single measured change before reading more code.
>
> 2026-10-12, episode `07241c55c7e8` (no candidate)

> I read lineage_bench.py, minbpe/base.py, basic.py and regex.py, then tried one change to the encode path. It tracked which code points are present and skipped the batch str.replace calls that could not match. I also made some small edits in basic.py and regex.py along the way, and reverted all of them.
> The sandbox evaluation rejected it with no improvement. encode_ir went from 414997463 to 421972803, a ratio of 1.0168 against a required 0.9900. train_ir went from 358194024 to 358671906, a ratio of 1.0013. Outputs were identical, builds passed and guards were ok. I submitted no candidate.
> I now think nearly every merge still needs a scan, so the skip rarely fires. Building set() over the joined string costs more than it saves. Training and encoding in this lineage are already heavily optimised, with heap-based merges, so cheap wins are gone.
> Next time I want to profile encode first to find where instructions actually go before editing anything. I should also avoid per-call whole-string work like set() or extra joins.
>
> 2026-10-12, episode `36667db7eece` (no candidate)

> I read through the minbpe lineage to get oriented: lineage_bench.py, minbpe/regex.py, base.py, basic.py, gpt4.py and __init__.py. I also grepped for import lines, which matched 51 lines, to see how the modules depend on each other. Then I started on a new helper module, minbpe/_rare.py, and wrote 212 lines into it.
> Nothing was measured. I ran no sandbox evaluations, and the session stopped without my submitting a candidate. That leaves _rare.py as untested code that nothing imports yet, so I have no evidence it helps or even works.
> From reading, I now believe the training and encode paths live in basic.py and regex.py on top of base.py, and gpt4.py is thin. I don't know where time actually goes under lineage_bench.py because I never ran it.
> Next session I should run the bench first to get a baseline. After that I can either wire _rare.py into the relevant path and measure it, or delete it. I need to submit something small and measured rather than end with an unhooked file.
>
> 2026-10-12, episode `f920321a85f2` (no candidate)

> I worked on the minbpe lineage this session. I read lineage_bench.py, then minbpe/regex.py, minbpe/__init__.py, minbpe/base.py and minbpe/basic.py, to understand how the benchmark exercises the tokenizer code before changing anything.
> I made no edits and ran no sandbox evaluations. The session ended with no candidate submitted, so I have no measured results and no outcome to report.
> What I believe now is limited to structure. The package is small: base.py holds shared tokenizer machinery, basic.py and regex.py are the two tokenizer variants, and __init__.py just exports them. I have not measured where time goes, so I hold no claim about which path is slow.
> Next session I should run the bench on the unmodified code first to get a baseline. Then I should pick one concrete change, likely in the merge or encode loop that both variants share, evaluate it in the sandbox, and submit something rather than stopping at reading.
>
> 2026-10-12, episode `3c519460334f` (no candidate)

> I read the minbpe lineage end to end: lineage_bench.py (lines 1-107), minbpe/regex.py (lines 1-378), minbpe/base.py (lines 1-378), minbpe/basic.py (lines 1-239) and minbpe/__init__.py (lines 1-12). I also ran one search across imports and helpers like render_token, replace_control, _update_stats, _apply_merges_heap and pop_top, which returned 83 matching lines.
> Nothing was measured. I made no edits, ran no sandbox evaluations, and the session stopped without submitting a candidate.
> My current belief is a map with no numbers behind it. The training and encoding paths share helpers in base.py, and the bench in lineage_bench.py drives them. Anything I think about where time goes is unverified until I run the bench.
> Next session I should run lineage_bench.py first to get a baseline before reading more. Then I should pick one hot path, likely the merge application or the stats update, make a single focused edit, and evaluate it in the sandbox before submitting.
>
> 2026-10-12, episode `0b1528256009` (no candidate)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
