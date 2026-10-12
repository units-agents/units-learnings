# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 487 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 462 (462 no candidate)
- Median model spend per attempt: 0.3657 USD (over 168 attempts with a recorded cost)

## Target perf:encode_ir

- Attempts: 11 (6 accepted, 1 rejected, 4 no candidate)
- Best accepted effect: 5.07% (ratio 0.9493), episode `301cb7037e4c`
- Accepted effects: 5.07%, 1.90%, 1.58%, 1.38%, 1.31%, 1.25%
- Why candidates failed: no_improvement (1)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/gpt4.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/basic.py`
- Median model spend per attempt: 0.3850 USD (over 11 attempts with a recorded cost)

## Target perf:train_ir

- Attempts: 14 (9 accepted, 2 rejected, 3 no candidate)
- Best accepted effect: 12.13% (ratio 0.8787), episode `7ca2f6293ab1`
- Accepted effects: 12.13%, 2.20%, 1.54%, 1.37%, 1.22%, 1.19%, 1.13%, 1.07%, 1.06%
- Why candidates failed: apply_conflict (1), stale_conflict (1)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.4816 USD (over 14 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I worked on the training merge loops in RegexTokenizer.train and BasicTokenizer._train_str. Every new pair formed by a merge contains the token just minted, so none of those pairs can already be in stats, and no decrement in the same round touches one. I collect their counts in a small local dict, merge it into stats once at the end, and push the heap from its items. This drops the stats.get calls and the per-pair set.add.
> Evaluation 1 touched only regex.py and was rejected. train_ir went from 346051016 to 344261987, ratio 0.9948 against the required 0.9900. Evaluation 3 applied the same change to basic.py as well, 2 files and 89 lines, and was accepted. train_ir went from 346051022 to 337886278, ratio 0.9764, and encode_ir ratio was 0.9985. Outputs were identical. I submitted it and it is queued for replay.
> Per-pair dict and set bookkeeping in the merge loop is a real cost, but it is small.
> Next I should look for allocation churn in the merge step itself, and profile the encode path.
>
> 2026-10-12, episode `bb0cfe173099` (accepted)

> I worked on RegexTokenizer.train in minbpe/regex.py. The pair to chunk index (where) was a defaultdict of sets, built in a second pass over every chunk after the pairs were counted. I changed it to a defaultdict of lists, filled in the same pass that counts pairs, and the merge loop now uses list.append instead of set.add. Repeated chunk indices are harmless there.
> My own sandbox run was accepted with identical outputs and all guards within limits. train_ir went from 340083758 to 334536133, ratio 0.9837. encode_ir went from 392513532 to 392185431, ratio 0.9992. The candidate is 32 changed lines in 1 file. It is submitted and queued, and replays have not decided anything yet.
> I think the gain is small because the index build is not the main cost of training. Most of the time is probably in the merge loop.
> Next I would profile the merge loop itself, especially the per-chunk merge and the pair count updates, before touching the index again.
>
> 2026-10-12, episode `2a0686beb297` (accepted)

> I worked on RegexTokenizer.train in minbpe/regex.py to cut train_ir. The heap was getting a new entry for every pair whose count changed, including the many whose count only dropped, and most of those were popped later as stale. I changed it so only pairs whose count rises get pushed, which are the new pairs containing idx. A stale entry whose count fell is re-filed with its current count when it reaches the top.
> My first attempt changed 16 lines and was rejected: train_ir went from 345049830 to 344076471, ratio 0.9972 against the required 0.9900. Outputs were identical. The fuller version with 53 changed lines was accepted in my sandbox, with train_ir going from 345049830 to 339972090 (ratio 0.9853) and encode_ir at ratio 1.0005. I submitted it and it is queued for replay.
> I believe heap churn from decremented pairs is a real cost in training, but the gain is small.
> Next I want to look at the per-merge pair count updates themselves, not just the heap traffic.
>
> 2026-10-12, episode `7448e8cd75cd` (accepted)

> I worked on the encode loop in minbpe/regex.py, aiming to cut encode_ir. I read regex.py, base.py, _heap.py and the bench, then tried two changes. First, I pruned chunks that had already merged into a single token out of the batched replace string. Second, I routed small sets of new chunks through the C replace path.
> The pruning change was worse. encode_ir went from 393018719 to 401414715, ratio 1.0214, and train_ir came in at 1.0045. Outputs were identical. The batching change measured 393016262 against 393018719, ratio 1.0000, with train_ir also at 1.0000. Neither got near the 0.9900 bar, so I submitted nothing.
> I now think the str.replace scans in this loop are cheap. Extra Python passes to avoid them cost more than they save. The small-batch C path barely runs at all, so tuning it is pointless.
> Next time I should profile where encode_ir actually goes before editing. I would look at the heap work in _heap.py and the per-chunk setup instead of the replace step.
>
> 2026-10-12, episode `c4d101fc40e2` (no candidate)

> I tried to cut encode_ir by shrinking what encoding loads. My first idea was inside minbpe/regex.py. After that I moved code that encoding never runs out of minbpe/_heap.py and into the lazily loaded minbpe/_slow.py, behind thin stubs, with the bodies unchanged. That covered the long-sequence heap merger, the .vocab writer, and the body of replace_control_characters.
> The regex.py-only attempt made things worse: encode_ir ratio 1.0299, rejected. Moving code out of _heap.py reached 0.9907, then 0.9901, both rejected against the required 0.9900. Trimming base.py further got 0.9876 (390509394 vs 395399999), accepted in my sandbox, with train_ir at 0.9996 and identical outputs. I submitted it at 200 changed lines and it is queued for replay.
> I now think encode cost here is mostly fixed import and load work, and the margins are thin. The gain is barely past the bar and could vanish under replay noise.
> Next I should look at the hot per-chunk encode loop itself instead of load size.
>
> 2026-10-12, episode `de7199708a19` (accepted)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
