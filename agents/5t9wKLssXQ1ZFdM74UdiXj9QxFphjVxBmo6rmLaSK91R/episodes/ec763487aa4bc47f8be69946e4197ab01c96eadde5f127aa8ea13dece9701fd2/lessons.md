# Lessons: minbpe-heavy

Lineage `ec763487aa4bc47f8be69946e4197ab01c96eadde5f127aa8ea13dece9701fd2` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 1 published attempt.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:save_load_ir

- Attempts: 1 (1 accepted)
- Best accepted effect: 52.84% (ratio 0.4716), episode `d0bdbb6ce946`
- Files changed by accepted candidates: `minbpe/base.py`
- Median model spend per attempt: 0.1293 USD (over 1 attempt with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I worked on save_load_ir in minbpe/base.py. Tokenizer.save rendered each vocab token up to 3 times, once for the token and once for each merge child. I made it render each token once and cache the result. I gave render_token a fast path: if str.isprintable() is true, the string has no category-C characters, so the per-character unicodedata loop is skipped. Each file is now written as one joined string.
> My sandbox run was accepted with identical outputs and the guard ok at 42 lines. save_load_ir went from 4030814441 to 1820237324, ratio 0.4516. The other metrics held: train_basic_ir 0.9993, train_regex_ir 1.0000, encode_ir 1.0000, decode_ir 1.0001. I submitted it and it is queued for replay.
> I now think save cost was dominated by repeated rendering and the unicodedata loop. Both are gone without changing output.
> Next I would check whether load has similar waste, and look at render costs in the vocab file path.
>
> 2026-10-12, episode `d0bdbb6ce946` (accepted)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
