# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 499 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 462 (462 no candidate)
- Median model spend per attempt: 0.3657 USD (over 168 attempts with a recorded cost)

## Target perf:encode_ir

- Attempts: 18 (10 accepted, 2 rejected, 6 no candidate)
- Best accepted effect: 5.07% (ratio 0.9493), episode `301cb7037e4c`
- Accepted effects: 5.07%, 1.90%, 1.58%, 1.38%, 1.31%, 1.25%, 1.22%, 1.21%, 1.05%, 1.01%
- Why candidates failed: no_improvement (2)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/_train.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/gpt4.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/_train.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.4730 USD (over 18 attempts with a recorded cost)

## Target perf:train_ir

- Attempts: 19 (11 accepted, 2 rejected, 6 no candidate)
- Best accepted effect: 12.13% (ratio 0.8787), episode `7ca2f6293ab1`
- Accepted effects: 12.13%, 3.94%, 2.20%, 1.54%, 1.37%, 1.26%, 1.22%, 1.19%, 1.13%, 1.07%, 1.06%
- Why candidates failed: apply_conflict (1), stale_conflict (1)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.4224 USD (over 19 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I tried to speed up encoding in minbpe. My first attempt was a small edit in minbpe/_heap.py. After that I rewrote _heap.py, added a str.replace merge-table path to _slow.py, and changed the batch threshold in base.py. Now every non-empty batch of new chunks goes through the table, where before only batches of 16 or more did.
> The first evaluation was rejected. encode_ir went from 366173622 to 370946319, ratio 1.0130 against a required 0.9900. The third evaluation was accepted. encode_ir went from 366173622 to 362334623, ratio 0.9895, and train_ir came in at ratio 0.9997. Outputs were identical in both. I submitted that version as a perf candidate of 163 changed lines, and it is queued for replay.
> What I believe now is that the table path is correct for any number of chunks. Most of the encode cost was small batches falling back to the pure-Python apply_merges loop. My margin is thin, since 0.9895 only just clears 0.9900, so replay noise could flip the result.
> Next I would profile what is left in apply_merges and in the chunk-cache lookup, looking for a larger margin.
>
> 2026-10-12, episode `acc4dcba5659` (accepted)

> I worked on encode_ir in minbpe. My first idea was to trim minbpe/_heap.py, but that barely changed anything. Then I moved RegexTokenizer's training body and the training-only pop_top_pairs helper out of minbpe/regex.py and minbpe/base.py into the lazily loaded minbpe/_train.py. Train now delegates through a local import, so the encode path no longer compiles code it never runs.
> Evaluation 1 touched 1 file and 31 lines and was rejected. encode_ir went from 385836553 to 386388196, a ratio of 1.0014 against the required 0.9900. Evaluation 3 touched 4 files and 199 lines and was accepted. encode_ir went from 385836553 to 381379210, ratio 0.9884, and train_ir moved to ratio 1.0007. Outputs were identical in both runs. I submitted it, and it is queued for replay.
> I now think encode_ir in this harness is dominated by import and compile cost, not by heap micro-tweaks. My margin is thin, just under the threshold.
> Next, I'll look for other code that encode imports but never needs, such as more of base.py or _slow.py. I also want to check whether replays hold 0.9884.
>
> 2026-10-12, episode `2758846d1833` (accepted)

> I tried to cut encode_ir by keeping training-only code out of the encode import path. The point was to stop minbpe.regex and minbpe.base from compiling the per-merge update and importing heapq and defaultdict when encoding never trains. I moved the merge step into _train.py as regex_merge, made train import its dependencies locally, and trimmed long docstrings and helpers in base.py. I also adjusted _slow.py imports to match.
> Evaluation 1 was accepted with identical outputs. encode_ir went from 382932185 to 378825037 (ratio 0.9893) and train_ir rose slightly, ratio 1.0006 being the later figure, 1.0017 here. Evaluation 3 was also accepted: encode_ir 382932191 to 378375310 (ratio 0.9881), train_ir ratio 1.0006, 189 lines. I submitted that one. It is queued and has no verdict yet.
> I believe import-time cost is a real but small slice of encode_ir. The gain is around one percent, and training pays a tiny cost for the lazy imports.
> Next I should look at the encode hot loop itself instead of imports.
>
> 2026-10-12, episode `7d594b552cac` (accepted)

> I tried moving RegexTokenizer's merge step into C. I joined all chunks with a separator and applied str.replace across the whole string, which meant editing minbpe/_train.py and minbpe/regex.py. Afterwards I reverted the training-loop edits.
> My one sandbox evaluation was rejected. Outputs were identical, but train_ir went from 314611307 to 335917309, a ratio of 1.0677 against the required 0.9900. encode_ir moved from 382348906 to 382731173, ratio 1.0010. The guard passed with 2 files and 36 lines. I submitted no candidate.
> What I now believe is that the existing deduplicated Python merge loop is hard to beat with a naive joined-string approach. Without chunk deduplication, the per-occurrence split work costs more than the C replace saves.
> Next time I should only come back to this if I can keep chunk deduplication. For example, I could join only the unique chunks and carry their counts separately, then measure train_ir again before I touch anything else.
>
> 2026-10-12, episode `b42b4437d684` (no candidate)

> I went after encode_ir in minbpe. I read regex.py, base.py, _train.py, _heap.py, _slow.py and basic.py, then spread edits across all of them. Mostly that meant trimming imports and helpers, rewriting base.py from 177 lines down to 80, and reshuffling code in _train.py and _heap.py.
> Evaluation 1 came back with identical outputs. encode_ir went from 378448980 to 374769173, a ratio of 0.9903. That missed the required 0.9900 and was rejected as no_improvement. train_ir ratio was 1.0005. Evaluation 3, after more edits, was worse: encode_ir ratio 1.0004 and train_ir ratio 1.0002, also rejected. I stopped without submitting.
> What I believe now is that import and module cleanup barely moves instruction counts here. The 0.9903 gain was close to noise, and my later edits threw it away. Encode cost has to live in the actual merge loop, not in the file structure.
> Next time I would profile encode in regex.py and _heap.py and target the per-chunk merge application directly. I would also keep the evaluation 1 diff as a starting point instead of over-editing past it.
>
> 2026-10-12, episode `0500f8a67872` (no candidate)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
