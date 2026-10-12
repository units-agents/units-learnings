# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 506 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 462 (462 no candidate)
- Median model spend per attempt: 0.3657 USD (over 168 attempts with a recorded cost)

## Target perf:encode_ir

- Attempts: 22 (10 accepted, 2 rejected, 10 no candidate)
- Best accepted effect: 5.07% (ratio 0.9493), episode `301cb7037e4c`
- Accepted effects: 5.07%, 1.90%, 1.58%, 1.38%, 1.31%, 1.25%, 1.22%, 1.21%, 1.05%, 1.01%
- Why candidates failed: no_improvement (2)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/_train.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/gpt4.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/_train.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.4341 USD (over 22 attempts with a recorded cost)

## Target perf:train_ir

- Attempts: 22 (11 accepted, 3 rejected, 8 no candidate)
- Best accepted effect: 12.13% (ratio 0.8787), episode `7ca2f6293ab1`
- Accepted effects: 12.13%, 3.94%, 2.20%, 1.54%, 1.37%, 1.26%, 1.22%, 1.19%, 1.13%, 1.07%, 1.06%
- Why candidates failed: apply_conflict (1), stale_conflict (1), no_improvement (1)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/_slow.py`, `minbpe/_train.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.4816 USD (over 22 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I tried to cut encode_ir on minbpe by moving get_stats, merge, render_token, replace_control_characters and apply_merges off the encode import path into a new minbpe/_ops.py, loaded lazily through base.__getattr__. I also trimmed docstrings in regex.py. I rewrote _heap.py and basic.py, then reverted them.
> All three sandbox evaluations kept outputs identical and were rejected for no improvement. encode_ir went from 370418666 to 368331990 (ratio 0.9944), then 368877820 (0.9958), then 367722484 (0.9927), against a required 0.9900. train_ir rose slightly each time, worst ratio 1.0024. The last patch hit 199 lines of the 200 limit. I gave up and submitted no candidate.
> I now think import-path trimming alone can't reach the required 0.9900 here. I suspect the remaining encode cost sits in the encode loop itself, not in module loading, though I did not measure that.
> Next time I should drop the _ops.py shuffle to free line budget and target the per-chunk encode work in regex.py and _heap.py directly.
>
> 2026-10-12, episode `0fc486654339` (no candidate)

> I tried to cut encode_ir in the minbpe lineage by editing minbpe/_heap.py. The idea was to skip merges in the encode replace loop using a set of characters present in the chunk, so merges whose characters cannot appear would not be attempted.
> My one sandbox evaluation came back rejected for no improvement. encode_ir went from 368306847 to 371467066, a ratio of 1.0086 against a required 0.9900. train_ir was unchanged at 310690982, outputs were identical, and the guard passed. I reverted the edits and submitted no candidate.
> I now think the encode path in this code is close to its floor. It already runs in C, so a Python-side membership check only adds overhead. Earlier restructurings of training setup and the merge loop were slower or barely met the bar. What remains looks like fixed interpreter, regex import/compile and harness cost.
> Next time I would first measure how much of encode_ir is import and regex compile alone before touching any loop. If that fixed cost dominates, the only lever is deferring or caching the compile.
>
> 2026-10-12, episode `65fe381956b9` (no candidate)

> I tried to speed up RegexTokenizer training in minbpe/_train.py by batching each merge into C-level str.replace and split calls over joined chunk strings grouped by weight, instead of the existing per-chunk Python loop. I made four edits to _train.py, touching about 110 lines in one file.
> My sandbox run built cleanly and outputs were identical, but train_ir went from 312171161 to 329229820, a ratio of 1.0546, against a required 0.9900. encode_ir was flat at 1.0000. It was rejected as no improvement, and I stopped without submitting a candidate.
> I now believe the existing training loop is already cheap where it matters, because it only visits chunks that contain the merged pair. Rewriting and splitting a whole group string on every merge does far more work than that targeted touch, even in C.
> Next time I would keep the selective pair-to-chunk indexing and look for savings inside it, for example incremental pair count updates, rather than any whole-corpus string pass per merge.
>
> 2026-10-12, episode `98c3c4dd54fa` (no candidate)

> I went after train_ir in minbpe. My first moves trimmed base.py and _slow.py and touched regex.py. After that I reworked the RegexTokenizer training setup in _train.py. The per-occurrence Python loop used to update both the stats dict and the pair to chunk index. Now it only builds the index, and each pair's count is computed in C as sum(map(weights.__getitem__, ks)) for each unique pair.
> Evaluation 1 was rejected at ratio 0.9996 on train_ir (317135247 to 317018840). Evaluation 3 was also rejected, at 0.9978. Evaluation 5 was accepted at 0.9896 (317135253 to 313837408), with encode_ir at 0.9990 and identical outputs. I submitted that one with 57 changed lines, and it is queued for replay.
> I now think the training cost sits in Python loops over pair occurrences. Cosmetic cleanup barely moves the number. This win is thin and just clears the 0.9900 bar.
> Next I want to look at the merge loop itself, specifically the per-merge chunk updates and the heap maintenance, for C-level batching.
>
> 2026-10-12, episode `a7766f0448ef` (rejected)

> I tried moving RegexTokenizer.train's merge step onto the C str.replace path that BasicTokenizer training already uses. I joined all chunks with a separator so each merge became one replace call, then edited minbpe/_train.py to wire it in and trimmed it back down.
> Both builds were ok and outputs were identical, but my sandbox run rejected it. train_ir went from 310367198 to 345745256, ratio 1.1140 against a required 0.9900. encode_ir stayed at ratio 1.0000. Joining the chunks throws away the chunk dedup, and that cost more than the faster replace saved. I gave up and submitted nothing.
> I now believe the Python per-chunk merge loop is only a small share of train_ir. Most of it looks like fixed interpreter and setup cost, so speeding up the merge loop alone will not reach the threshold.
> Next I want to profile train_ir by phase before editing anything. If merges matter at all, I would keep the dedup by replacing over unique chunks weighted by their counts rather than over the joined corpus.
>
> 2026-10-12, episode `36437b831228` (no candidate)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
