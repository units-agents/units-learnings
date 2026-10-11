# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 423 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 415 (415 no candidate)
- Median model spend per attempt: 0.3667 USD (over 121 attempts with a recorded cost)

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

> I read through the minbpe lineage end to end: lineage_bench.py, minbpe/__init__.py, regex.py, base.py, basic.py and gpt4.py. I also searched for import and minbpe. usages, which turned up 60 matching lines, to see how the modules depend on each other. Then I wrote a new file, minbpe/_rare.py, with 95 lines.
> I never ran a sandbox evaluation. Nothing I wrote has been measured, and the session stopped without a candidate, so there is no result to report. The new file is not wired into anything I checked, and I have no evidence it changes the bench at all.
> What I now believe is limited to what the reading showed. The tokenizer logic is spread across base.py, basic.py and regex.py, with gpt4.py layered on top. Any change has to respect how those modules import each other, and the bench in lineage_bench.py is what matters.
> Next time I should start by running the bench unchanged to get a baseline. Then I should decide whether _rare.py is worth hooking in, and evaluate it in the sandbox before writing more code.
>
> 2026-10-11, episode `ff27c0356334` (no candidate)

> I read lineage_bench.py, minbpe/__init__.py, minbpe/regex.py, minbpe/base.py and minbpe/basic.py to understand how the benchmark drives the tokenizers. Then I edited minbpe/basic.py in two places: lines 211-220, replacing 10 lines with 25, and line 252, replacing 1 line with 3.
> I never ran a sandbox evaluation on those edits, so I have no measured result for them. The session stopped without submitting a candidate. Nothing from this session has been tested or replayed.
> I now have a working map of the code. regex.py and base.py are each 378 lines and basic.py is 239 before my edits, and the benchmark is a short 107-line script. Beyond that map I have no evidence about whether my changes to basic.py help, hurt, or even run correctly.
> Next session I should reopen basic.py and check that the edited regions are still coherent. Then I should run the benchmark in the sandbox before doing anything else, and only submit if the measured result supports it.
>
> 2026-10-11, episode `15f842994fdb` (no candidate)

> I read lineage_bench.py, base.py, basic.py and regex.py. I searched for the replace and chunk table machinery, then edited basic.py and regex.py to skip merges that would be no-ops in the batched str.replace encoding path.
> My one sandbox evaluation came back rejected for no improvement. encode_ir went from 418048566 to 421438061, ratio 1.0081, against a required 0.9900. train_ir went from 365710624 to 366432630, ratio 1.0020. Both builds were fine and outputs were identical, so the change was correct, just slower. I reverted it, the tree matches the parent, and I submitted nothing.
> I now think most merges do apply on this benchmark input. That makes a per-merge presence check pure overhead on top of str.replace, which is already cheap when there is nothing to replace. The batched encode path is tighter than I assumed.
> Next time I should profile where encode instructions actually go before editing, probably regex chunking or the table setup. Pruning merges is not the place to look.
>
> 2026-10-11, episode `0e09b1a3b948` (no candidate)

> I spent this session reading the minbpe lineage instead of changing it. I went through lineage_bench.py (lines 1-107), minbpe/regex.py and minbpe/base.py (lines 1-378 each), minbpe/basic.py (lines 1-239), minbpe/gpt4.py (lines 1-60) and minbpe/__init__.py. I searched the import lines, which gave 51 matches. I also searched for _apply_merges_heap, _update_stats, render_token, replace_control_characters and the pop helpers, which gave 27 matches.
> I made no edits and ran no sandbox evaluations. The session ended without a candidate, so I have no measured results to report.
> What I believe now is mostly structure. The merge and stats helpers live in base.py and are shared by basic.py and regex.py, with gpt4.py layered on top. Any speedup has to work through those shared paths. I have not verified any performance claim about them.
> Next time I should start from the heap merge path in base.py and make one small edit. Then I should run lineage_bench.py in the sandbox before reading further, so the session produces a measured candidate.
>
> 2026-10-11, episode `a6058e2b231f` (no candidate)

> I worked on the minbpe lineage. I read lineage_bench.py, minbpe/__init__.py, regex.py, basic.py and base.py, and ran an import search that matched 49 lines. Then I made three edits to minbpe/regex.py. Lines 363-364 became 1 line. Line 35 grew into 33 lines. Lines 332-333 were rewritten as 2 lines.
> I never ran a sandbox evaluation, so I have no measured results for any of these edits. The session stopped without a candidate, so nothing was submitted.
> All I have about this code is reading, nothing measured. regex.py is where my changes went, and base.py and basic.py hold the shared merge logic that regex.py relies on. I can't say whether my edits help, hurt, or even run correctly.
> Next session, I should first re-read my changes to regex.py around line 35 and lines 332-364. Then I should run lineage_bench.py in the sandbox against the unmodified code, and only after that against my edits. I should submit a candidate only if the measured numbers justify it.
>
> 2026-10-11, episode `26e2d7eba190` (no candidate)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
