# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 510 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 462 (462 no candidate)
- Median model spend per attempt: 0.3657 USD (over 168 attempts with a recorded cost)

## Target perf:encode_ir

- Attempts: 24 (10 accepted, 2 rejected, 12 no candidate)
- Best accepted effect: 5.07% (ratio 0.9493), episode `301cb7037e4c`
- Accepted effects: 5.07%, 1.90%, 1.58%, 1.38%, 1.31%, 1.25%, 1.22%, 1.21%, 1.05%, 1.01%
- Why candidates failed: no_improvement (2)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/_train.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/gpt4.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/_train.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.4341 USD (over 24 attempts with a recorded cost)

## Target perf:train_ir

- Attempts: 24 (11 accepted, 3 rejected, 10 no candidate)
- Best accepted effect: 12.13% (ratio 0.8787), episode `7ca2f6293ab1`
- Accepted effects: 12.13%, 3.94%, 2.20%, 1.54%, 1.37%, 1.26%, 1.22%, 1.19%, 1.13%, 1.07%, 1.06%
- Why candidates failed: apply_conflict (1), stale_conflict (1), no_improvement (1)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/_slow.py`, `minbpe/_train.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.5223 USD (over 24 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I tried to cut encode_ir in minbpe/_heap.py. The C str.replace merge loop now skips a merge when one character of the pair can't be in the string, using a set of present characters. A second variant also dropped characters from that set once a merge had used them all up.
> Both builds passed with identical outputs and train_ir unchanged at ratio 1.0000. The first version moved encode_ir from 378399261 to 377030830, ratio 0.9964. It was rejected against the required 0.9900. The second version did worse at 377884786, ratio 0.9986. The extra scans needed to drop characters cost more than the skips they enabled. I submitted no candidate.
> I now think the replace loop is not where most of the encode cost is. Pruning merges by character presence gives well under 1%.
> Next I should profile encode end to end before editing. I want to see whether the time goes to regex chunking, the _chunk_table and _enc_state lookups, or per-chunk call overhead, and then target the largest share.
>
> 2026-10-12, episode `ced50c57875f` (no candidate)

> I tried to cut train_ir in minbpe/_train.py by changing how RegexTokenizer training builds pair counts. First I looked up byte-pair chunks on demand. Then I made that lookup lazy. Last I counted pairs at C level over a joined blob. All my edits were confined to lines 40-119 of that one file.
> Every attempt built and gave identical outputs, and every one was rejected as no improvement. train_ir went from parent 316370265 to 341915573 (ratio 1.0807). Next it went from 316370271 to 334093555 (1.0560), and then to 331832785 (1.0489). encode_ir held at 1.0000 throughout. I stopped without a candidate, with one evaluation left and no profiler.
> I now believe the existing pair counting in _train.py is already cheap relative to the extra work my restructurings added. Each rewrite cost more instructions than it saved. Guessing at hot spots without a profile is not working here.
> Next time I want a profile of training before editing anything. After that I would look at the merge and update loop rather than the initial pair counting.
>
> 2026-10-12, episode `8292addf3eb3` (no candidate)

> I tried to cut encode_ir in minbpe/_heap.py. In the C str.replace merge table, I skipped merges whose pair contains a token that no earlier merge produced. I read regex.py, _heap.py, base.py and gpt4.py first, then edited _heap.py several times and trimmed my first 24-line insert down to 8 lines.
> The sandbox evaluation was rejected for no_improvement. Outputs were identical and the guard was ok, but encode_ir went from 373964248 to 374802614, ratio 1.0022 against the required 0.9900. train_ir was flat at ratio 1.0000. I gave up and did not submit a candidate.
> I now believe the replace loop is a negligible share of encode_ir. Most of the cost looks fixed: interpreter start, importing and compiling the regex module, the harness itself, and findall. Pruning the merge table does not move it.
> Next time I should profile what encode_ir actually counts before editing. Then I would look at regex compile and import cost or at findall chunking, rather than the merge loop.
>
> 2026-10-12, episode `5bd3bc64f3c7` (no candidate)

> I tried to cut train_ir in minbpe/_train.py by reworking how the RegexTokenizer training setup builds its pair-to-chunk index. First I built only the index and summed each pair's weights in C. Then I carried the same index-only idea into the per-merge update. Last, I built the whole index in C with map/any over chained zips.
> None cleared the 0.9900 bar, and all three had identical outputs. The first gave train_ir 313040269 to 310917010, ratio 0.9932. The second was worse at 0.9947. The third was slower, ratio 1.0001. encode_ir stayed at 1.0000 or 0.9999 throughout. I submitted nothing.
> I now think the setup index is not where training time goes. Rewriting it saves well under 1%. The per-merge update path did not benefit from the index-only approach, and pushing the work into C idioms adds overhead instead of removing it.
> Next I should profile the merge loop itself before editing, perhaps the count updates and the pair selection. I should also stop leaving stray files like _train_tmp_unused.txt.
>
> 2026-10-12, episode `cb3cdd54b16c` (no candidate)

> I tried to cut encode_ir on minbpe by moving get_stats, merge, render_token, replace_control_characters and apply_merges off the encode import path into a new minbpe/_ops.py, loaded lazily through base.__getattr__. I also trimmed docstrings in regex.py. I rewrote _heap.py and basic.py, then reverted them.
> All three sandbox evaluations kept outputs identical and were rejected for no improvement. encode_ir went from 370418666 to 368331990 (ratio 0.9944), then 368877820 (0.9958), then 367722484 (0.9927), against a required 0.9900. train_ir rose slightly each time, worst ratio 1.0024. The last patch hit 199 lines of the 200 limit. I gave up and submitted no candidate.
> I now think import-path trimming alone can't reach the required 0.9900 here. I suspect the remaining encode cost sits in the encode loop itself, not in module loading, though I did not measure that.
> Next time I should drop the _ops.py shuffle to free line budget and target the per-chunk encode work in regex.py and _heap.py directly.
>
> 2026-10-12, episode `0fc486654339` (no candidate)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
