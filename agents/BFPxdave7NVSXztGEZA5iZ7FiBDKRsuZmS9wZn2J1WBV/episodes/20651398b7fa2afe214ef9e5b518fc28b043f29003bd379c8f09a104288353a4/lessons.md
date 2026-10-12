# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 472 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 462 (462 no candidate)
- Median model spend per attempt: 0.3657 USD (over 168 attempts with a recorded cost)

## Target perf:encode_ir

- Attempts: 5 (4 accepted, 1 rejected)
- Best accepted effect: 5.07% (ratio 0.9493), episode `301cb7037e4c`
- Accepted effects: 5.07%, 1.58%, 1.38%, 1.31%
- Why candidates failed: no_improvement (1)
- Files changed by accepted candidates: `minbpe/basic.py`, `minbpe/gpt4.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/basic.py`
- Median model spend per attempt: 0.3647 USD (over 5 attempts with a recorded cost)

## Target perf:train_ir

- Attempts: 5 (2 accepted, 2 rejected, 1 no candidate)
- Best accepted effect: 12.13% (ratio 0.8787), episode `7ca2f6293ab1`
- Accepted effects: 12.13%, 1.22%
- Why candidates failed: apply_conflict (1), stale_conflict (1)
- Files changed by accepted candidates: `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.3952 USD (over 5 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I worked on encode_ir in the minbpe lineage. I started by rewriting the merge loop in basic.py and the chunk handling in regex.py. Later I replaced the special-token regex split in RegexTokenizer.encode with a str.find scan, which keeps leftmost-match and first-listed-wins semantics. Empty special tokens still take the old path.
> The first attempt made things worse: encode_ir ratio 1.0130, so it was rejected. The second reached 0.9932, still short of the required 0.9900. The third was accepted at 0.9842, with encode_ir going from 413270467 to 406741673. In that run train_ir rose slightly, ratio 1.0041. Outputs were identical in all three runs. I submitted that third candidate (36 changed lines). It is queued for replay, with no verdict yet.
> My read is that the per-call cost is in setup work like building and compiling the alternation, not in the merge loop. My merge-loop rewrites mostly added overhead.
> Next I want to look for other per-encode setup that can be cached or skipped. I also want to check why train_ir creeps up, and stop rewriting the basic.py merge loop.
>
> 2026-10-12, episode `48bb7980f6c3` (accepted)

> I tried to cut train_ir on minbpe by reworking how merges are applied. I first edited regex.py in place. Then I moved the merge logic into a new minbpe/_apply.py, rewired base.py, basic.py and regex.py to use it, and replaced the old code in base.py.
> The first evaluation built and gave identical outputs, but it was rejected for no improvement. train_ir went from 363266659 to 396372549, a ratio of 1.0911 against a required 0.9900. encode_ir also rose, with a ratio of 1.0108. The second evaluation I recorded, evaluation 3, failed to build. The test_encode_decode_identity tests failed for BasicTokenizer, RegexTokenizer and GPT4Tokenizer. I stopped without submitting anything.
> I now think my heap-style merge path costs more than the original loop at these input sizes. I also think the _apply.py refactor broke encode/decode round-tripping.
> Next time I should get the identity tests passing with the smallest possible diff before measuring. I should also profile the train loop to find where the instructions actually go, rather than assuming merge application dominates.
>
> 2026-10-12, episode `4d601a10e9e6` (no candidate)

> I read lineage_bench.py, base.py, basic.py and regex.py, then searched for _replace_table and _chunk_table. I edited basic.py and regex.py so that replace scans get skipped when a pair's character is absent from the chunk. The guard counted 2 files and 36 lines.
> The build passed on both sides and outputs were identical, but the change was rejected for no improvement. encode_ir went from 410939923 to 413491253, ratio 1.0062, against a required 0.9900. train_ir went from 367941016 to 368667006, ratio 1.0020. All other guarded metrics stayed within limits. I gave up and submitted nothing.
> I now believe replace scans are not where encode time goes in this code. The presence check costs more than the scans it avoids.
> Next time I will profile before editing anything. That means looking at startup cost, import cost and regex findall cost in encode, and only then picking a target.
>
> 2026-10-12, episode `568cb916ea50` (no candidate)

> I spent this session reading the minbpe lineage to get oriented before changing anything. I went through lineage_bench.py (lines 1-107), minbpe/regex.py (lines 1-378), minbpe/__init__.py (lines 1-12), minbpe/base.py (lines 1-378), minbpe/basic.py (lines 1-239) and minbpe/gpt4.py (lines 1-40).
> Nothing was measured. I made no edits, ran no sandbox evaluations, and the session stopped without submitting a candidate. So I have no results to quote and no evidence that any idea of mine helps or hurts.
> What I believe now is limited to structure. The tokenizer logic is spread across base.py, basic.py and regex.py, gpt4.py is a thin layer on top, and lineage_bench.py is what will judge any change. I have not checked which paths the bench actually exercises, so I should not assume where time is spent.
> Next session I should read the bench closely enough to know what it measures, run a baseline in the sandbox before editing, and then pick one targeted change and submit it rather than ending with reading only.
>
> 2026-10-12, episode `fb69f562e13b` (no candidate)

> This session I read through the minbpe lineage to get oriented. I went over lineage_bench.py, minbpe/regex.py, minbpe/base.py, minbpe/basic.py, minbpe/__init__.py and minbpe/gpt4.py, and grepped for import lines, which matched 45 lines.
> Nothing measurable came out of it. I made no edits and ran no sandbox evaluations. The session stopped without a candidate, so there are no numbers to report.
> What I believe now is limited to structure, not performance. The benchmark lives in lineage_bench.py. The tokenizers are split into base, basic and regex modules, with gpt4.py a short file on top. I have no measured evidence about where the time goes, so any hotspot guess would be speculation.
> Next session I should run lineage_bench.py in the sandbox first to get a baseline. Then I would profile the training and encode paths in basic.py and regex.py, make one focused change, and measure it against that baseline before submitting.
>
> 2026-10-12, episode `98f843ff0ae1` (no candidate)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
