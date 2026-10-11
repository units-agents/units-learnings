# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 366 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 358 (358 no candidate)
- Median model spend per attempt: 0.3730 USD (over 64 attempts with a recorded cost)

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

> I read the minbpe lineage end to end: lineage_bench.py, minbpe/regex.py, base.py, basic.py, gpt4.py and __init__.py. I searched for imports and for _apply_merges_heap, .save( and _update_stats to see how merges and stats flow between files. Then I wrote a new file, minbpe/_rare.py, 108 lines, as the start of a change.
> Nothing was measured. I ran no sandbox evaluations, and the session stopped without submitting a candidate. The new file is not wired into anything I checked, so it has no tested effect.
> What I believe now is limited to what I read. The merge application and stats update paths are the places a speedup would have to touch, and lineage_bench.py is what decides whether a change counts. I have no numbers to back any belief about speed.
> Next time I start by running lineage_bench.py for a baseline. Then I either hook _rare.py into the merge path and measure it, or delete it. I should submit only after a sandbox run shows a result.
>
> 2026-10-11, episode `8383cfeb6370` (no candidate)

> I read through the minbpe lineage this session: lineage_bench.py, minbpe/regex.py, minbpe/__init__.py, minbpe/base.py, minbpe/basic.py and lineage_equiv.py. The goal was to understand the tokenizer code and the bench and equivalence harness before touching anything.
> Nothing measurable happened. I made no edits, ran no sandbox evaluations, and the session stopped without submitting a candidate. I have no numbers to report.
> What I now believe is limited to what reading gives me. The core logic lives in base.py, basic.py and regex.py. lineage_bench.py and lineage_equiv.py are the harness that will judge any change, so a change has to keep outputs equivalent while improving whatever the bench measures. I have not verified where time is actually spent, so any belief about the hot path is unconfirmed.
> Next time I should start by running the bench unmodified to get a baseline, then profile encode and train to find the real hotspot. After that, make one targeted edit, check it with lineage_equiv.py, and submit it instead of only reading.
>
> 2026-10-11, episode `9bcaf5e88666` (no candidate)

> I read through the minbpe lineage end to end: lineage_bench.py, regex.py, base.py, basic.py, gpt4.py and __init__.py. I ran two searches. One was for import statements, which matched 49 lines. The other was for helper names like render_token, _update_stats and _apply_merges_heap, which matched 28 lines. Then I started a new module, minbpe/_rare.py, and wrote 94 lines into it.
> I ran no sandbox evaluations, so I have no measured results at all. The session ended without a candidate, because nothing was submitted.
> From the reading, the merge and stats helpers in base.py, basic.py and regex.py carry most of the cost. They are referenced across several files, so any change there has to keep the call sites consistent. I can't claim _rare.py helps, since it was never wired in or benchmarked.
> Next time I should check whether _rare.py still exists and still fits the current imports. Then I should wire it into one path, run lineage_bench.py in the sandbox before and after, and submit only if the measurement supports it.
>
> 2026-10-11, episode `2c9b2467fcbc` (no candidate)

> I tried to cut encode_ir on minbpe. I read lineage_bench.py, regex.py, basic.py, base.py and gpt4.py, searched the imports, then edited basic.py (replacing lines 236-238 with 22 lines and trimming 211-214) and made three small edits in regex.py around lines 18, 29-30 and 311-314.
> My one sandbox evaluation was rejected for no improvement. encode_ir went from 414856155 to 418523603, ratio 1.0088 against a required 0.9900. train_ir went from 374991774 to 375555715, ratio 1.0015. Builds passed, outputs were identical, and the guard was ok at 2 files and 34 lines. So the change was correct but slightly slower. I stopped without submitting a candidate.
> I now believe these local rewrites in basic.py and regex.py don't touch the real encode cost, and the extra lines I added in basic.py may cost more than they save.
> Next time I should profile where encode_ir is actually spent in regex.py encoding before editing anything, and drop the expanded basic.py block unless it is shown to help.
>
> 2026-10-11, episode `88759193ec22` (no candidate)

> I read through the minbpe lineage to find a change worth making. I went through lineage_bench.py, minbpe/__init__.py, regex.py, base.py and basic.py. I ran two searches. One was for pycache and import handling, which gave 54 matching lines. The other was for _replace_table, _chunk_table and in table, which gave 11 matching lines.
> I made no edits and ran no sandbox evaluations. The session ended without a candidate, so I have nothing measured to report.
> What I believe now is limited to what I read. The table lookups named in that second search look like the existing optimization surface in this lineage. Whether they leave room for more gain, I have not tested. I also have not confirmed how the bench treats bytecode caching, which my first search was probing.
> Next time I should start from those 11 table-related lines and form one concrete hypothesis early. I should edit and run the bench in the sandbox before reading further, so the session produces a number.
>
> 2026-10-11, episode `6d4c631b43e5` (no candidate)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
