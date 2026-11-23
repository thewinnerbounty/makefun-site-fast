# site-gen — a faster drop-in for makefun-site 1.0.0

Entry for make.fun bounty `site-generator-speed-01-run-20261122`.

## What it is

`solve.js` is a dependency-free ES module exporting `run(input)`, which returns exactly the output tree that makefun-site 1.0.0's `build()` returns for the same input — byte for byte, for every input (front matter, template dialect, Markdown subset, slug rule and build report are makefun-site's own, reimplemented from its reference semantics).

`main.ts` is the harness driver: it reads `/input/cases.json`, answers each case with `run`, and writes `/output/answers.json`. Outside a container, `MAKEFUN_INPUT_DIR` / `MAKEFUN_OUTPUT_DIR` stand in for `/input` / `/output`. It turns V8's incremental marking off before importing `solve.js`, as the house baseline does.

## How it is faster than the baseline

- `inline()` renders Markdown text in whole plain chunks with one escape pass each, instead of escaping character by character.
- `escapeHtml()` returns unchanged strings without a replacement pass.
- Template variables are pre-compiled closures with per-node escape memoization (titles, URLs and tags repeat across every page), removing per-node switch dispatch at render time.
- Allocation-free word counting for reading minutes; single-pass block/inline lexing otherwise unchanged.

## Verification

- Byte-identical to the house baseline on all 3 practice cases.
- Byte-identical over 120 cases drawn with the kit's generator from 40 random seeds.
- Byte-identical on hand-crafted adversarial inputs (refusals, drafts, reserved dirs, duplicate slugs, Unicode titles, CRLF, broken links, template errors, static shadowing).
- The kit's `grade-local` (docker driver): `{"cases_passed":3,"cases_total":3,"passed":1}`, L1 regrade reproduced byte for byte.
- ~2.0x the house baseline's speed on CPU time over the practice set (same machine, interleaved runs).

## Run

    node main.ts   # Node 22.18+

## License

MIT — see LICENSE.
