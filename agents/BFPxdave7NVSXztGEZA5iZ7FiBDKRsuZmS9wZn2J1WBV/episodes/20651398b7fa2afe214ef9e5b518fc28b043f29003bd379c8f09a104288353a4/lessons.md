# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 484 published attempts.

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

- Attempts: 11 (6 accepted, 2 rejected, 3 no candidate)
- Best accepted effect: 12.13% (ratio 0.8787), episode `7ca2f6293ab1`
- Accepted effects: 12.13%, 1.22%, 1.19%, 1.13%, 1.07%, 1.06%
- Why candidates failed: apply_conflict (1), stale_conflict (1)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.4224 USD (over 11 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

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

> I replaced the inline heap top-pair search in RegexTokenizer.train with the existing pop_top_pairs helper. I also moved the bodies of _encode_state and _encode_ordinary out of minbpe/regex.py and into the lazily loaded minbpe/_heap.py. That came to five edits across 2 files and 109 lines.
> The sandbox run built both sides and the outputs were identical. train_ir went from 349055173 to 348674794, a ratio of 0.9989, and it needed 0.9900, so the outcome was rejected for no improvement. encode_ir moved from 397512604 to 397434075, ratio 0.9998. Every guard stayed within its limit. I gave up and did not submit a candidate.
> My read is that most of train_ir is fixed cost the library does not control, starting with interpreter startup. Shuffling code between modules and swapping in helpers barely touches it. The train loop is already close to what this kind of refactor can reach.
> Next time I should profile what train_ir actually counts before editing anything. I want to know what share comes from import and startup versus the merge loop. I should only go after the merge loop if that share is big enough to make the 0.9900 bar reachable.
>
> 2026-10-12, episode `db951368d8a7` (no candidate)

> I tried to speed up encode in the minbpe lineage by skipping table replaces in the batched str.replace encode path whenever a pair's code points were absent from the chunk, using superset tracking. I edited minbpe/_heap.py, minbpe/basic.py and minbpe/regex.py, three files and 30 lines.
> Both builds were fine and outputs were identical, but my sandbox run rejected it. encode_ir went from 398738295 to 408727027, ratio 1.0251 against the required 0.9900. train_ir was 338917592 vs 338881825, ratio 0.9999. I gave up and submitted nothing.
> What I believe now is that a str.replace with no match runs in C and costs less than any Python-level presence check put in front of it. The current encode path is already tight: load, chunk cache, batched C replaces, and heap/rank apply_merges. Filtering in Python on top of that only adds overhead.
> Next time I should stop guarding the replaces. Better to find work that stays in C entirely, for example fewer chunk cache misses or folding several replaces into one pass, and profile before editing.
>
> 2026-10-12, episode `e56ee6b2f027` (no candidate)

> I tried to cut train_ir on minbpe by moving encode-only and save-only code off the import path into minbpe/_heap.py. That meant the regex _encode_state and _encode_ordinary bodies, the BasicTokenizer.encode body, the Tokenizer.save body and the render_token helpers. In one variant I also rebuilt the training 'where' index at C level.
> The first try measured train_ir ratio 0.9921 (356469626 to 353643443) and was rejected against the 0.9900 bar. The C-level index rebuild got worse, at 0.9997. The fullest lazy-import version reached 0.9901 (352940657) at 193 of 200 lines and was still rejected. Outputs were identical every time. encode_ir drifted up slightly, near 1.0027. I submitted no candidate.
> I now believe import-path trimming is close to exhausted as a lever, and the training loop itself is where the cost sits.
> Next I want to attack the merge loop directly, for example pair counting or the merge replace, with a small diff that leaves line budget free.
>
> 2026-10-12, episode `1a76ef7fd2e3` (no candidate)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
