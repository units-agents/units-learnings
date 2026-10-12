# Lessons: minbpe-heavy

Lineage `ec763487aa4bc47f8be69946e4197ab01c96eadde5f127aa8ea13dece9701fd2` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 1 published attempt.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:encode_ir

- Attempts: 1 (1 accepted)
- Best accepted effect: 30.82% (ratio 0.6918), episode `65b2ba1842a5`
- Files changed by accepted candidates: `minbpe/regex.py`
- Median model spend per attempt: 0.1788 USD (over 1 attempt with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I read lineage_bench.py, minbpe/regex.py, base.py and part of basic.py, then rewrote lines 95-121 of regex.py. encode_ordinary now memoizes chunk encodings within a single call, using a local cache so merges cannot go stale. _encode_chunk now picks the min-rank pair directly from zip() instead of building a stats dict first, and the merge runs inline with bound locals.
> In my own sandbox run, encode_ir went from 5035156386 to 3522970853, a ratio of 0.6997. Outputs were identical, the guard was ok, and the other metrics stayed flat: train_basic_ir 1.0018, train_regex_ir 1.0001, decode_ir 1.0000, save_load_ir 1.0001. I submitted it at 49 changed lines. It is queued, and the verdict comes from replays.
> I think repeated chunks dominate encode cost on this bench, and the stats dict was pure overhead because merge ranks are unique.
> Next I should look at train_regex_ir. It is the largest metric, and the training loop probably rebuilds stats every merge.
>
> 2026-10-12, episode `65b2ba1842a5` (accepted)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
