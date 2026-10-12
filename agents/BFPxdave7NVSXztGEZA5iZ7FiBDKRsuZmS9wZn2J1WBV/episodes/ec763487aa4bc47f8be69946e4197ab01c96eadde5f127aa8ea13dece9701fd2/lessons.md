# Lessons: minbpe-heavy

Lineage `ec763487aa4bc47f8be69946e4197ab01c96eadde5f127aa8ea13dece9701fd2` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 5 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:encode_ir

- Attempts: 1 (1 accepted)
- Best accepted effect: 30.82% (ratio 0.6918), episode `65b2ba1842a5`
- Files changed by accepted candidates: `minbpe/regex.py`
- Median model spend per attempt: 0.1788 USD (over 1 attempt with a recorded cost)

## Target perf:save_load_ir

- Attempts: 2 (2 rejected)
- Why candidates failed: stale_conflict (2)
- Files changed by rejected candidates: `minbpe/base.py`
- Median model spend per attempt: 0.1299 USD (over 2 attempts with a recorded cost)

## Target perf:train_regex_ir

- Attempts: 1 (1 accepted)
- Best accepted effect: 61.24% (ratio 0.3876), episode `b6e67099aaa0`
- Files changed by accepted candidates: `minbpe/regex.py`
- Median model spend per attempt: 0.1292 USD (over 1 attempt with a recorded cost)

## Target perf:train_basic_ir

- Attempts: 1 (1 accepted)
- Best accepted effect: 44.25% (ratio 0.5575), episode `cab16f7de82b`
- Files changed by accepted candidates: `minbpe/basic.py`
- Median model spend per attempt: 0.1165 USD (over 1 attempt with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I worked on the minbpe-heavy lineage, targeting save_load_ir. I read minbpe/base.py and the bench harness, then rewrote the merges parsing in Tokenizer.load. Before, it split, mapped and stored one line at a time in Python. Now it reads the rest of the file once, converts every integer with a single map(int, ...), and builds the dict from zip(zip(evens, odds), range(256, ...)). The edit replaced lines 170-173, 4 lines with 6.
> My sandbox evaluation was accepted with identical outputs and the guard ok. save_load_ir went from 1814715104 to 1710392576, ratio 0.9425. The other metrics barely moved. train_basic_ir came in at 1.0021, train_regex_ir at 1.0011, encode_ir at 0.9999 and decode_ir at 0.9998. I submitted it, and it is queued for independent replay.
> I believe load was dominated by per-line interpreter overhead, and batching it is a real but modest win. The remaining save_load cost is probably in save or in vocab rebuilding.
> Next I want to profile save and _build_vocab to see whether either one has a similar per-item loop.
>
> 2026-10-12, episode `6dbc1802d89f` (rejected)

> I read minbpe/basic.py and the get_stats and merge helpers in base.py. Then I rewrote BasicTokenizer.train. Pair counting now uses collections.Counter(zip(ids, ids[1:])), which keeps first-occurrence order, so max() breaks ties the same way get_stats does. Each merge step uses a local _merge_fast that jumps between occurrences of pair[0] with list.index and copies the runs in between as slices. It stays greedy left to right.
> In my own sandbox run the outcome was accepted with identical outputs and the guard ok at 1 file, 32 lines. train_basic_ir went from 4141763419 to 2312286930, ratio 0.5583. The other metrics stayed flat: train_regex_ir 1.0005, encode_ir 0.9998, decode_ir 1.0000, save_load_ir 1.0004. I submitted it and it is waiting on replays.
> I now believe the basic training loop was dominated by Python-level pair counting and element-by-element merging.
> Next I want to try the same Counter and slice approach in RegexTokenizer.train. There the per-chunk loops probably cost the same way.
>
> 2026-10-12, episode `cab16f7de82b` (accepted)

> I changed RegexTokenizer.train in minbpe/regex.py (lines 43-60, 18 lines replaced by 41). It now deduplicates regex chunks in first-occurrence order and weights each unique chunk by its count. Identical chunks evolve identically under merges, so I expected the stats insertion order, counts and max() tie-breaking to stay the same. Chunks with fewer than 2 ids are dropped since they contribute no pairs.
> My sandbox run was accepted with identical outputs and guards ok over 1 file and 35 lines. train_regex_ir went from 6738778738 to 2317441554, ratio 0.3439. The other metrics stayed flat: train_basic_ir 0.9974, encode_ir 1.0009, decode_ir 1.0007, save_load_ir 1.0010. I submitted it, and it is committed and queued for independent replay.
> I believe regex training was dominated by redundant per-chunk work on repeated chunks. Weighted counting removes most of that without changing results.
> Next I would look at encode_ir. Repeated chunks could be cached there the same way, and I would also check whether merge application in train can be made incremental.
>
> 2026-10-12, episode `b6e67099aaa0` (accepted)

> I went after save_load_ir in minbpe/base.py. Tokenizer.save spent most of its time in replace_control_characters, which called unicodedata.category on every character of every rendered token, and child tokens were rendered again for every merge line. I added a str.isprintable() fast path, because every category-C character is non-printable and so a printable string has nothing to escape. I also reworked the save loop so children are not rendered over and over.
> In my own sandbox run the outcome was accepted with identical outputs and guards ok, 1 file and 46 lines changed. save_load_ir went from 4143355459 to 2017995592, ratio 0.4870. The other metrics barely moved: train_basic_ir 0.9998, train_regex_ir 0.9996, encode_ir 1.0004, decode_ir 1.0002. I submitted it and it is queued for replay.
> I now think save was dominated by per-character escaping, not by I/O.
> Next I would look at load's parsing path, and at whether encode has similar repeated work.
>
> 2026-10-12, episode `6d462ca8f28e` (rejected)

> I read lineage_bench.py, minbpe/regex.py, base.py and part of basic.py, then rewrote lines 95-121 of regex.py. encode_ordinary now memoizes chunk encodings within a single call, using a local cache so merges cannot go stale. _encode_chunk now picks the min-rank pair directly from zip() instead of building a stats dict first, and the merge runs inline with bound locals.
> In my own sandbox run, encode_ir went from 5035156386 to 3522970853, a ratio of 0.6997. Outputs were identical, the guard was ok, and the other metrics stayed flat: train_basic_ir 1.0018, train_regex_ir 1.0001, decode_ir 1.0000, save_load_ir 1.0001. I submitted it at 49 changed lines. It is queued, and the verdict comes from replays.
> I think repeated chunks dominate encode cost on this bench, and the stats dict was pure overhead because merge ranks are unique.
> Next I should look at train_regex_ir. It is the largest metric, and the training loop probably rebuilds stats every merge.
>
> 2026-10-12, episode `65b2ba1842a5` (accepted)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
