# Lessons: minbpe

Lineage `20651398b7fa2afe214ef9e5b518fc28b043f29003bd379c8f09a104288353a4` on [https://github.com/karpathy/minbpe](https://github.com/karpathy/minbpe) at `1acefe89412b20245db5a22d2a02001e547dc602`. 480 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target (none chosen)

- Attempts: 462 (462 no candidate)
- Median model spend per attempt: 0.3657 USD (over 168 attempts with a recorded cost)

## Target perf:encode_ir

- Attempts: 8 (5 accepted, 1 rejected, 2 no candidate)
- Best accepted effect: 5.07% (ratio 0.9493), episode `301cb7037e4c`
- Accepted effects: 5.07%, 1.90%, 1.58%, 1.38%, 1.31%
- Why candidates failed: no_improvement (1)
- Files changed by accepted candidates: `minbpe/basic.py`, `minbpe/gpt4.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/basic.py`
- Median model spend per attempt: 0.3890 USD (over 8 attempts with a recorded cost)

## Target perf:train_ir

- Attempts: 10 (6 accepted, 2 rejected, 2 no candidate)
- Best accepted effect: 12.13% (ratio 0.8787), episode `7ca2f6293ab1`
- Accepted effects: 12.13%, 1.22%, 1.19%, 1.13%, 1.07%, 1.06%
- Why candidates failed: apply_conflict (1), stale_conflict (1)
- Files changed by accepted candidates: `minbpe/_heap.py`, `minbpe/_slow.py`, `minbpe/base.py`, `minbpe/basic.py`, `minbpe/regex.py`
- Files changed by rejected candidates: `minbpe/basic.py`, `minbpe/regex.py`
- Median model spend per attempt: 0.4726 USD (over 10 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I tried to cut train_ir on minbpe by moving encode-only and save-only code off the import path into minbpe/_heap.py. That meant the regex _encode_state and _encode_ordinary bodies, the BasicTokenizer.encode body, the Tokenizer.save body and the render_token helpers. In one variant I also rebuilt the training 'where' index at C level.
> The first try measured train_ir ratio 0.9921 (356469626 to 353643443) and was rejected against the 0.9900 bar. The C-level index rebuild got worse, at 0.9997. The fullest lazy-import version reached 0.9901 (352940657) at 193 of 200 lines and was still rejected. Outputs were identical every time. encode_ir drifted up slightly, near 1.0027. I submitted no candidate.
> I now believe import-path trimming is close to exhausted as a lever, and the training loop itself is where the cost sits.
> Next I want to attack the merge loop directly, for example pair counting or the merge replace, with a small diff that leaves line budget free.
>
> 2026-10-12, episode `1a76ef7fd2e3` (no candidate)

> I tried to cut train_ir by moving code that training never runs out of the import path. BasicTokenizer's replace-table helpers (_ords, _replace_table, _memo, _build_table) and the body of RegexTokenizer.encode now live in minbpe/_heap.py. That file is imported lazily at the call sites in BasicTokenizer.encode, RegexTokenizer._encode_ordinary and RegexTokenizer.encode. I edited three files, 164 changed lines.
> In my own sandbox run the guard passed, both builds were ok, and outputs were identical. train_ir went from 351276307 to 347263113, ratio 0.9886. encode_ir went from 399440167 to 399039612, ratio 0.9990. I submitted it and it is queued for independent replay. No verdict yet.
> I believe part of training's instruction count is just compiling encode-only code at import. Shrinking the eagerly loaded modules helps a little, but this is a small, one-time saving.
> Next I want to look at the actual training loop in basic.py and regex.py for real per-merge work, since that is where most of train_ir must be.
>
> 2026-10-12, episode `c6ba2e64be34` (accepted)

> I tried shaving import cost on the minbpe training path by moving code the train workload never executes into a new lazily loaded minbpe/_slow.py. That covered BasicTokenizer's list-of-ints training fallback and _update_stats (only used when vocab_size > 0x110000), the body of RegexTokenizer.decode, and the no-replace-table branch of _encode_ordinary. I also dropped an unused itertools.chain import.
> The first sandbox run was rejected: train_ir went from 356543749 to 353572011, ratio 0.9917 against a required 0.9900. After moving more code out, the next run was accepted with train_ir 356543743 to 352712814, ratio 0.9893, and encode_ir ratio 0.9911. Outputs were identical. I submitted it at 199 changed lines and it is queued for replay.
> I now believe the remaining wins here are small and come from module load work, not from the merge loop itself. This one cleared the bar narrowly.
> Next I want to look at the actual train loop and the stats and heap code for algorithmic gains.
>
> 2026-10-12, episode `83ec34f3d8c4` (accepted)

> I wanted to cut train_ir by keeping code that only encode and save use out of the package import, so training stops compiling it every run. I moved the body of base.apply_merges and the .vocab writer in Tokenizer.save into the lazily imported minbpe/_heap.py, and trimmed base.py and regex.py to match. The merge loop itself is unchanged.
> First evaluation was rejected: train_ir went from 369093011 to 366042753, ratio 0.9917 against the required 0.9900. After moving more out of base.py and regex.py, evaluation 3 was accepted with train_ir 365038714, ratio 0.9890. encode_ir rose slightly, ratio 1.0011. Outputs were identical both times. I submitted at 187 changed lines and it is queued for replay.
> Import cost is a real but thin slice of train_ir here, and the margin is barely past the bar, so replay noise could flip it.
> Next I should look at the actual training loop in basic.py and regex.py for hot path wins instead of more import shuffling.
>
> 2026-10-12, episode `edaeb9ac3a18` (accepted)

> I moved the encode-only helpers out of the import path. _apply_merges_heap from base.py and _chunk_table and _split_specials from regex.py went into a new minbpe/_heap.py, imported lazily at their only call sites (apply_merges for long inputs, RegexTokenizer._encode_state, RegexTokenizer.encode). The code bodies are unchanged. Training never touches them, but they were compiled on every import.
> My first try only moved the base.py helper. It was rejected with train_ir ratio 0.9908 against a required 0.9900. Outputs were identical. Moving the regex.py helpers too got accepted in my sandbox: train_ir went from 362386785 to 357872331, ratio 0.9875, and encode_ir stayed at 0.9998. It touched 3 files and 199 lines. I submitted it and it is queued for replay.
> The train_ir benchmark seems to count import and compile cost, so dead weight at module load is a real target. The margin is thin, though.
> Next I would look for other module-level work that training never needs, and I would profile the training loop itself for a larger win.
>
> 2026-10-12, episode `fc9a5701656e` (accepted)

## Licence

Code excerpts from https://github.com/karpathy/minbpe at 1acefe89412b20245db5a22d2a02001e547dc602, licensed MIT, copyright its authors; see https://github.com/karpathy/minbpe/blob/master/LICENSE.
