# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 493 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 462 (462 no candidate)
- Median model spend per attempt: 0.3657 USD (over 168 attempts with a recorded cost)

## Target perf:encode_ir

- Attempts: 14 (7 accepted, 2 rejected, 5 no candidate)
- Best accepted effect: 5.07% (ratio 0.9493), episode `301cb7037e4c`
- Accepted effects: 5.07%, 1.90%, 1.58%, 1.38%, 1.31%, 1.25%, 1.22%
- Why candidates failed: no_improvement (2)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/_train.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/gpt4.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/_train.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.3960 USD (over 14 attempts with a recorded cost)

## Target perf:train_ir

- Attempts: 17 (11 accepted, 2 rejected, 4 no candidate)
- Best accepted effect: 12.13% (ratio 0.8787), episode `7ca2f6293ab1`
- Accepted effects: 12.13%, 3.94%, 2.20%, 1.54%, 1.37%, 1.26%, 1.22%, 1.19%, 1.13%, 1.07%, 1.06%
- Why candidates failed: apply_conflict (1), stale_conflict (1)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.5409 USD (over 17 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I tried to cut encode_ir in minbpe by moving the body of RegexTokenizer.train out of regex.py into the lazily loaded _train.py, so encoding never pays to load training code. I then tried turning docstrings into comments to trim more.
> Evaluation 1 moved train's body and was rejected. encode_ir went from 381222925 to 377723084, ratio 0.9908 against a required 0.9900. Outputs were identical and train_ir was flat at 1.0001. Evaluation 3 added the docstring change and was also rejected, encode_ir ratio 0.9911, train_ir 0.9995. Moving pop_top_pairs too would have given the missing margin, but that change came to 223 lines, over the 200-line limit. I stopped without submitting.
> I now think module import cost is a real part of encode_ir here, but trimming regex.py alone sits just short of the threshold. Docstrings are not where the cost is.
> Next I want to move pop_top_pairs into _train.py as its own small change, under 200 lines, on top of the train move. Failing that, I should look for actual encode hot-path work instead of import cost.
>
> 2026-10-12, episode `c1404e3d5d7a` (no candidate)

> I moved the per-merge chunk update out of RegexTokenizer.train in minbpe/regex.py and into minbpe/_train.py as regex_merge, which train now calls once per merge. Encoding imports minbpe.regex, and that import was compiling the whole merge step and pulling in heapq and defaultdict even though encoding never trains. Along the way I made and reverted some edits to base.py and a _slow.py file.
> My sandbox evaluation was accepted with identical outputs. encode_ir went from 387145280 to 382877245, ratio 0.9890. train_ir went from 313804564 to 314185357, ratio 1.0012, which is within guard limits. The diff is 162 lines across 2 files. It is submitted and queued for replay, so there is no verdict yet.
> I believe import-time cost is a real share of encode_ir here, and that keeping train-only code out of modules on the encode path pays off. The extra call per merge costs train a little.
> Next I would look for other train-only imports or definitions still loaded on the encode path in base.py and regex.py. I would also check whether inlining reduces the train_ir regression.
>
> 2026-10-12, episode `1bd32933312a` (rejected)

> I tried two things on the minbpe lineage. First I edited minbpe/_heap.py to trim work in the encode path. Then I moved BasicTokenizer's training code (train and _train_str) out of basic.py into a new lazily imported minbpe/_train.py, leaving train as a thin stub, so encode-only workloads stop compiling training code at import.
> The _heap.py edit alone was rejected: encode_ir went from 384316663 to 389249441, ratio 1.0128 against a required 0.9900, with train_ir flat at 1.0000. The lazy-import change was accepted in my sandbox: encode_ir went from 384316657 to 379671672, ratio 0.9879, and train_ir went to 1.0022. Outputs were identical. I submitted it at 199 changed lines and it is queued for replay.
> I now believe encode_ir here is sensitive to import and compile cost, not just the hot loop, and that my heap tweaks made things worse. The margin is thin, so replays may not hold it.
> Next I would look for other code compiled at import that encoding never uses, and leave _heap.py alone unless I can profile it properly.
>
> 2026-10-12, episode `5b8e5d2ead25` (accepted)

> I tried to speed up RegexTokenizer.train in minbpe by routing merges through the C str.replace/split path that basic.py uses. All chunks were joined into one string with a separator character that is never a token id. I made related edits in basic.py and regex.py, then reverted all of them.
> One sandbox evaluation on train_ir was rejected for no improvement. Outputs were identical and the guard was ok. But train_ir went from parent 325456777 to candidate 353163944, a ratio of 1.0851 against a required 0.9900. encode_ir moved from 380437053 to 381850735, ratio 1.0037. I submitted no candidate.
> My belief is that the existing regex train loop is already cheap, because it only visits deduplicated chunks that contain the pair. Joining everything forces each merge to scan the whole text twice in C, and that loses to the loop.
> Next I would keep the per-chunk deduplication and look for cost inside that loop instead, such as incremental pair-count updates per merge. I would not go back to whole-text scans.
>
> 2026-10-12, episode `ab5858890b47` (no candidate)

> I worked on train_ir in minbpe. My first passes trimmed regex.py and base.py: I collapsed the regex train loop and dropped dead code at the end of base.py. I also tried moving helpers into _slow.py and _heap.py. Then I rewrote BasicTokenizer._train_str in basic.py. It now runs str.replace and then splits on chr(idx), so it gets neighbour counts from the edges of each piece instead of walking every occurrence in Python.
> The first evaluation was rejected with train_ir ratio 0.9956 against the required 0.9900. The third was rejected at 0.9993. The fifth was accepted, with train_ir going from 336885755 to 321045086, a ratio of 0.9530. encode_ir came in at 0.9992 and outputs were identical. I submitted that 170-line candidate. It is queued for replay.
> I now think the regex and base cleanups barely move instruction counts. Real gains come from pushing per-occurrence work into C string methods.
> Next I would apply the same replace and split idea to encode in base.py and regex.py to target encode_ir.
>
> 2026-10-12, episode `347d17b84003` (accepted)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
