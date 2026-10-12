# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 518 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 463 (462 no candidate, 1 abandoned)
- Median model spend per attempt: 0.3657 USD (over 168 attempts with a recorded cost)

## Target perf:encode_ir

- Attempts: 28 (10 accepted, 2 rejected, 16 no candidate)
- Best accepted effect: 5.07% (ratio 0.9493), episode `301cb7037e4c`
- Accepted effects: 5.07%, 1.90%, 1.58%, 1.38%, 1.31%, 1.25%, 1.22%, 1.21%, 1.05%, 1.01%
- Why candidates failed: no_improvement (2)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/_train.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/gpt4.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/_train.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.4218 USD (over 28 attempts with a recorded cost)

## Target perf:train_ir

- Attempts: 27 (11 accepted, 4 rejected, 12 no candidate)
- Best accepted effect: 12.13% (ratio 0.8787), episode `7ca2f6293ab1`
- Accepted effects: 12.13%, 3.94%, 2.20%, 1.54%, 1.37%, 1.26%, 1.22%, 1.19%, 1.13%, 1.07%, 1.06%
- Why candidates failed: apply_conflict (1), stale_conflict (1), no_improvement (2)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/_slow.py`, `minbpe/_train.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.4328 USD (over 27 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I read lineage_bench.py and the minbpe sources: base.py, _heap.py, regex.py, __init__.py and _train.py. I was looking for a remaining hotspot in encode or train that could clear the 1% bar. I made no edits and ran no sandbox evaluations.
> Nothing was measured this session and I submitted no candidate. I gave up because both paths are already heavily optimized. Encode uses batched C str.replace merging and chunk caching, and training is incremental and heap-based. My notes from prior attempts put the remaining micro-optimizations between 0.99 and 1.002, which is short of the bar.
> I now believe the remaining cost is mostly fixed overhead the library barely controls: interpreter startup, the regex package import and pattern compile, and bench corpus generation. That is my reading of the code, not something I measured.
> Next time I should profile the bench end to end before touching the library, to confirm how the time splits between import, compile, corpus setup and the actual encode and train. If import or compile dominates, lazy imports or deferring the pattern compile might be worth a measured try.
>
> 2026-10-12, episode `16de224668ee` (no candidate)

> I read the bench, regex.py, base.py, _heap.py, _train.py and basic.py, and searched for the replace table and memo helpers. I tried to cut encode cost in minbpe/_heap.py by adding a skip check ahead of the per-merge str.replace loop. Then I reworked that edit and finally reverted the large block back to 2 lines.
> My one sandbox evaluation of the skip check was rejected for no improvement. encode_ir went from 376510556 to 377252453, ratio 1.0020 against a required 0.9900. train_ir was unchanged at 321277533. Outputs were identical and the guard passed. I submitted nothing.
> The encode path already merges in C, caches chunks and has trimmed imports. The replace scans are cheap and not the cost. Most of what is left in both metrics is fixed overhead: interpreter startup, the regex import and pattern compile, and the bench's own corpus generation.
> Next time I would profile the bench itself before editing library code, to see whether any non-fixed share is still worth chasing.
>
> 2026-10-12, episode `0fc6ec419f75` (no candidate)

> I read the bench, base.py, regex.py, basic.py, _heap.py and _train.py. Then I tried to cut train_ir by pushing the regex-train setup loop in _train.py into C-level map/deque calls, both for building the where index and for the weighted stats. I edited lines 37-54 several times and reverted part of that work.
> The sandbox run built fine and gave identical outputs, but train_ir went from 314202137 to 314852164, a ratio of 1.0021 against a required 0.9900. It was rejected for no improvement. encode_ir stayed at 373212448, ratio 1.0000. I submitted no candidate.
> I now believe setup is not where train_ir is spent. The cost has to be in the merge loop and heap updates in _heap.py. Encode already does one batched replace pass with cached chunks, so what remains there is mostly fixed interpreter and regex cost.
> Next time I should profile the merge loop before editing anything. The places to look are the heap push/pop churn and the per-merge pair count updates in _train.py.
>
> 2026-10-12, episode `2718adbc238e` (no candidate)

> I tried replacing the per-chunk merge updates in minbpe/_train.py with a full-corpus approach for RegexTokenizer training: join all chunks with a 0xff separator and apply each merge with str.replace and split over the whole text. I added the new path near line 169 and later collapsed my earlier setup edit at lines 40-56 back to one line.
> My sandbox evaluation built fine and gave identical outputs, but it was rejected. train_ir went from 311019829 to 336146877, ratio 1.0808, against a required 0.9900. encode_ir was unchanged at 380606937. I submitted no candidate.
> What I believe now is that the existing code wins because it works on deduplicated chunks. Running replace and split over the non-deduplicated text costs more than the Python per-chunk updates it replaces. Setup-only tweaks I tried earlier land just under 1%.
> Next time I should profile train_ir before editing, to find where the time actually goes in the merge loop. Any full-text rewrite should keep the deduplication.
>
> 2026-10-12, episode `a1bdd1eda41c` (no candidate)

> I worked on RegexTokenizer training setup in minbpe/_train.py. The setup loop updated the stats dict for every pair occurrence in Python. I changed it so the loop builds only the pair to chunk index. Each count is then the sum of chunk weights over that pair's index entries, done in C with sum(map(weights.__getitem__, ks)). Chunk id lists now come from map(list, map(str.encode...)).
> My first attempt was rejected. train_ir went from 312152498 to 326573746, ratio 1.0462 against a required 0.9900. After reworking the edit, evaluation 3 was accepted with train_ir 308862291, ratio 0.9895. encode_ir held at 1.0000, outputs were identical, and the change was 16 lines. I submitted it and it is queued for replay.
> The setup pass is a real cost in train_ir, but the margin is thin. A careless restructure easily makes it worse.
> Next I should look at the merge loop itself, where per-step stats updates likely cost more than setup.
>
> 2026-10-12, episode `a7f7ee2fe2a6` (rejected)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
