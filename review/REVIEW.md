# Review of the LLM-generated CrochetPARADE dataset

Scope: all 323 entries of `dsl_patterns.json` at upstream revision `5ee0179`. Every entry was compiled with the real CrochetPARADE parser (`parse64.js`), checked for internal stitch-count consistency, and — where the English source is available (234 entries, CrochetBench text for the same pattern ID) — compared with the English. The creator's `confidence` field is left untouched; this review's own grade is in `review_report.json`.

## Results

| | Before | After |
|---|---|---|
| Entries that compile | 89 of 323 | 319 of 323 |
| Entries where every round works exactly the previous round and matches the counts quoted in its comments | 40 | 250 |

277 entries changed in total; 102 of them beyond removing fences and preamble. Only the `dsl` field was touched; schema, key order and JSON formatting are unchanged.

## Main findings

1. **Markdown fences broke 224 entries.** The generated entries still carried the LLM's ``` fences (224), four also had a prose preamble ("Looking at this image, I see…"), and 19 were cut off at the 4,096-token output limit in the middle of a line. None of these compiled as stored. One entry (`RHC0502-022909M`) contains only the placeholder text "just dsl".
2. **CrochetPARADE does not check that a round uses up the previous round.** A round that works too few stitches silently leaves the rest for the next round; one that works too many wraps onto itself. So "it compiles" says little. A round-closure check found this kind of drift in most entries, including creator-rated ones (for example a body with fifteen `54sc` rounds on 60 stitches in a confidence-4 entry).
3. **Many unrated entries are not conversions of their English pattern.** In 48 of the 128 entries read against the English, the code is a generic invented toy (6-sc magic ring, 42/48-sc ball, invented hair, skirt or tail pieces) whose comments "quote" text that is not in the source. The count-alignment measure below points the same way for the unrated entries that were not read by hand. These look generated from the picture, not from the text. They were left unchanged and graded D; they need re-conversion, not correction.
4. **Creator confidence is not stitch-level fidelity.** Three confidence-5 entries (`BRC0334-028941M`, `SCC0334-003582M`, `SCC0334-003586M`) do not follow the paired English (for example an 8-based sphere growing to 120 sc rendered as a 6-based sphere growing to 72). The rating appears to describe how the rendered shape looks. Caveat: this assumes the CrochetBench text with the same ID is the text the creator used.
5. **Recurring conversion errors** in entries that do follow the English: wrong repeat counts; a group meant for one stitch ("5 dc in next st") written as separate stitches; ovals around a foundation chain written as magic rings; invented closing gathers (`sc6tog`) where the English only says to gather; "make 2/4" pieces written once; plain rounds collapsed or miscounted; front/back-post stitches written with undefined names.
6. **The English itself has errors.** Reviewers found stated counts that cannot result from the instruction in a number of patterns; these are marked inline with `# source inconsistency:` and listed in `reviewer_notes.json`.

## What was changed

| Change | Entries |
|---|---|
| Stripped markdown code fence | 224 |
| Removed LLM prose before the code | 4 |
| Dropped the half-written last line of a truncated output | 19 |
| Removed unbalanced brackets inside comments (the parser counts them) | 4 |
| Corrected the number in plain "work even" rounds (only where the following rounds then close) | 13 |
| Edited after reading against the English / comments | 78 |

Review outcome for the 128 entries read by hand: 63 fixed, 15 partly fixed, 2 no change needed (flags were false alarms), 48 left unchanged because the code is unrelated to the English or is a placeholder.

Three automatic "work even" corrections were reverted after inspection because the original was right (fold-flat closing rows that work through both layers, and a round picked up evenly around an edge): `BRC0334-37912M`, `RHC0334-028530M`, `SCC0334-009502M`.

## How accuracy was measured

- **Compiles**: `final()` of CrochetPARADE's parser accepts the text.
- **Round closure**: for every row that continues a piece, the stitches it attaches to are exactly the previous row's stitches (no leftovers, no wrap-around). Partial turned rows, rows with explicit `@` attachments and closing gathers are reported separately as information, not errors.
- **Stated counts**: where a row's comment ends with a count ("… 24 sc."), the compiled row makes that many stitches.
- **English alignment** (needs the English): the stitch counts stated in the English, in order, are searched as an ordered subsequence of the compiled per-row counts. The ratio is the share found. It is independent of the comments in the code, which the LLM sometimes invented.

English alignment by creator confidence (entries with English and at least five stated counts):

| Creator confidence | 0.85 or more | 0.6–0.85 | 0.3–0.6 | under 0.3 | n/a |
|---|---|---|---|---|---|
| 5 | 11 | 3 | 3 | 0 | 2 |
| 4 | 15 | 1 | 0 | 0 | 3 |
| 3 | 12 | 2 | 0 | 0 | 1 |
| 2 | 10 | 1 | 1 | 0 | 0 |
| 1 | 7 | 1 | 1 | 0 | 3 |
| unrated | 11 | 50 | 60 | 21 | 15 |

## Review grade

- **A** — compiles, rounds close, and either English alignment is 0.85 or more or a reviewer fixed it against the English with confidence 4–5.
- **B** — compiles and is internally consistent; fidelity to the English is moderate (0.6–0.85) or could not be measured.
- **C** — compiles but has unresolved count flags, or English alignment is 0.3–0.6.
- **D** — does not compile, is unrelated to its English, or English alignment is under 0.3.

| Creator confidence | A | B | C | D |
|---|---|---|---|---|
| 5 | 11 | 13 | 1 | 2 |
| 4 | 16 | 7 | 0 | 0 |
| 3 | 13 | 5 | 0 | 0 |
| 2 | 9 | 1 | 2 | 0 |
| 1 | 10 | 4 | 2 | 0 |
| unrated | 10 | 118 | 38 | 61 |
| **All** | 69 | 148 | 43 | 63 |

## Limits

- Grade B is not a claim of fidelity. For unrated entries in B the code is self-consistent but was not read line by line against the English; given finding 3, treat them as unverified.
- Reviewer edits were made by Claude sub-agents with the parser in the loop and were re-verified mechanically (no entry got worse on any measure); a sample was read by hand. They were not crocheted or rendered in 3D.
- 89 entries have no English text available here; they were checked only against their own comments.
- The round-closure check cannot see errors that keep counts consistent (a wrong stitch type, a missing even round, a missing piece).
- Four entries still do not compile; all four are unrelated to their English or placeholders and were deliberately not repaired: `BRC0534-034496M`, `RHC0334-022428M`, `RHC0334-028528M`, `RHC0502-022909M`.

## Files

- `dsl_patterns.json` — corrected dataset (same schema).
- `review/review_report.json` — per-entry measures before and after, changes, grade.
- `review/reviewer_notes.json` — per-entry reviewer notes: each change with its reason, source inconsistencies, missing pieces.
- `review/tools/` — scripts to reproduce the checks (`make_parser.py` builds a Node module from upstream `parse64.js`; `check.py FILE.cp` checks one pattern).

## Per-entry table

Flags = round-count flags. Align = English alignment ratio.

| ID | Creator conf. | Grade | Compiles before → after | Flags before → after | Align | Review | Lines changed |
|---|---|---|---|---|---|---|---|
| ALC0234-035173M | 4 | A | yes → yes | 2 → 0 | 1.0 | automatic checks only | 15 |
| ALC0234-035175M | 4 | A | yes → yes | 2 → 0 | 1.0 | fixed | 1 |
| ALC0334-025131M | 5 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| ALC0334-025237M | 3 | A | yes → yes | 7 → 0 | 1.0 | fixed | 7 |
| ALC0334-033866M | 4 | B | yes → yes | 0 → 0 | 0.82 | automatic checks only | 0 |
| ALC0334-033995M | 5 | B | yes → yes | 0 → 0 | – | automatic checks only | 0 |
| ALC0334-033997M | 5 | B | yes → yes | 0 → 0 | – | automatic checks only | 0 |
| ALC0334-033999M | 4 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| ALC0334-034001M | 4 | B | yes → yes | 0 → 0 | – | automatic checks only | 0 |
| BRC0234-002364M | 3 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| BRC0234-003805M | 2 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| BRC0234-005423M | 4 | A | yes → yes | 1 → 0 | 1.0 | fixed | 28 |
| BRC0234-010053M | 2 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| BRC0234-012724M | 1 | A | no → yes | – → 1 | 0.85 | fixed | 186 |
| BRC0234-012728M | 3 | B | yes → yes | 3 → 3 | – | no_change | 0 |
| BRC0234-026964M | 1 | A | no → yes | – → 0 | 1.0 | fixed | 32 |
| BRC0234-029172M | 1 | A | yes → yes | 1 → 0 | 0.98 | fixed | 126 |
| BRC0234-029174M | 2 | A | yes → yes | 6 → 0 | 0.97 | fixed | 36 |
| BRC0234-030779M | 3 | A | yes → yes | 1 → 0 | 1.0 | fixed | 3 |
| BRC0234-032612M | 1 | A | yes → yes | 11 → 3 | 1.0 | fixed | 25 |
| BRC0234-032636M | 1 | A | yes → yes | 48 → 0 | – | fixed | 51 |
| BRC0234-033201M | 5 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| BRC0234-033203M | 3 | A | yes → yes | 1 → 0 | 1.0 | fixed | 1 |
| BRC0234-034461M | 2 | A | yes → yes | 3 → 0 | 0.94 | fixed | 23 |
| BRC0234-034483M | 4 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| BRC0234-036377M | 5 | A | yes → yes | 1 → 0 | 1.0 | fixed | 7 |
| BRC0234-37712M | 1 | A | yes → yes | 1 → 0 | 0.87 | fixed | 53 |
| BRC0320-000079M | 4 | B | yes → yes | 2 → 0 | – | partly_fixed | 2 |
| BRC0320-000084M | 3 | B | yes → yes | 3 → 1 | – | fixed | 14 |
| BRC0320-003842M | 4 | B | yes → yes | 0 → 0 | – | automatic checks only | 0 |
| BRC0320-011713M | 1 | B | yes → yes | 4 → 1 | – | fixed | 26 |
| BRC0334-012696M | 4 | A | yes → yes | 0 → 0 | 0.91 | automatic checks only | 0 |
| BRC0334-027952M | 3 | A | yes → yes | 0 → 0 | 0.96 | automatic checks only | 0 |
| BRC0334-028479M | 4 | A | yes → yes | 6 → 0 | 1.0 | fixed | 16 |
| BRC0334-028939M | 3 | A | yes → yes | 11 → 0 | 1.0 | fixed | 90 |
| BRC0334-028941M | 5 | C | yes → yes | 0 → 0 | 0.38 | automatic checks only | 0 |
| BRC0334-030330M | 5 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| BRC0334-030550M | 4 | A | yes → yes | 1 → 0 | 0.97 | fixed | 67 |
| BRC0334-031169M | 2 | A | yes → yes | 21 → 15 | 1.0 | fixed | 72 |
| BRC0334-031873M | 5 | A | yes → yes | 2 → 0 | 0.91 | fixed | 49 |
| BRC0334-031875M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| BRC0334-032095M | unrated | C | no → yes | – → 0 | 0.4 | automatic checks only | 0 |
| BRC0334-032283M | 1 | B | yes → yes | 0 → 0 | 0.79 | automatic checks only | 0 |
| BRC0334-034973M-1 | unrated | A | no → yes | – → 0 | 0.87 | automatic checks only | 0 |
| BRC0334-035196M | 2 | A | yes → yes | 6 → 2 | 1.0 | fixed | 33 |
| BRC0334-035451M | unrated | B | no → yes | – → 0 | – | fixed | 93 |
| BRC0334-036172M | unrated | C | no → yes | – → 0 | 0.57 | automatic checks only | 0 |
| BRC0334-036248M | unrated | D | no → yes | – → 0 | 0.13 | automatic checks only | 0 |
| BRC0334-036684M | 3 | B | yes → yes | 0 → 0 | 0.83 | automatic checks only | 0 |
| BRC0334-36837M | unrated | D | no → yes | – → 4 | 0.57 | unfixable | 0 |
| BRC0334-37351M | unrated | B | no → yes | – → 0 | 0.82 | automatic checks only | 0 |
| BRC0334-37355M | 4 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| BRC0334-37359M | 1 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| BRC0334-37363M | 2 | A | yes → yes | 3 → 0 | 1.0 | fixed | 40 |
| BRC0334-37367M | 5 | A | yes → yes | 2 → 0 | 1.0 | fixed | 16 |
| BRC0334-37912M | unrated | C | no → yes | – → 2 | 0.38 | automatic checks only | 0 |
| BRC0504-002277M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| BRC0504-025906M | unrated | B | no → yes | – → 0 | – | partly_fixed | 5 |
| BRC0517-005618M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| BRC0517-005625M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| BRC0518-034918M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| BRC0518-036242M | 5 | B | yes → yes | 0 → 0 | – | automatic checks only | 0 |
| BRC0518-37124M | unrated | B | no → yes | – → 0 | – | fixed | 18 |
| BRC0520-002418M | 5 | B | yes → yes | 1 → 0 | – | fixed | 1 |
| BRC0520-009810M | unrated | B | no → yes | – → 0 | – | fixed | 4 |
| BRC0520-030523M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| BRC0520-032279M | unrated | B | no → yes | – → 0 | – | automatic checks only | 2 |
| BRC0520-034507M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| BRC0520-37694M | 5 | B | no → yes | – → 0 | – | fixed | 9 |
| BRC0521-37161M | 5 | B | yes → yes | 1 → 0 | – | fixed | 2 |
| BRC0534-029092M | unrated | C | no → yes | – → 0 | 0.43 | automatic checks only | 0 |
| BRC0534-034494M | unrated | C | no → yes | – → 0 | 0.44 | automatic checks only | 0 |
| BRC0534-034496M | unrated | D | no → no | – → – | – | unfixable | 0 |
| BRC0534-034498M | unrated | B | no → yes | – → 0 | 0.64 | automatic checks only | 0 |
| BRC0534-034501M | unrated | C | no → yes | – → 0 | 0.53 | automatic checks only | 0 |
| BRC0534-034503M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| BRC0534-034505M | 4 | A | yes → yes | 14 → 6 | 1.0 | fixed | 76 |
| BRC0534-034510M | unrated | D | no → yes | – → 1 | 0.18 | unfixable | 0 |
| BRC0534-036536M | unrated | D | no → yes | – → 1 | 0.54 | unfixable | 0 |
| BRC0534-036538M | unrated | D | no → yes | – → 7 | 0.81 | unfixable | 0 |
| BRC0534-036542M | unrated | D | no → yes | – → 2 | 0.37 | unfixable | 0 |
| BRC0534-036546M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| CAC0201-033162M | unrated | B | no → yes | – → 0 | – | fixed | 3 |
| CAC0234-004177M | unrated | D | no → yes | – → 0 | 0.22 | automatic checks only | 0 |
| CAC0234-011729M | unrated | D | no → yes | – → 0 | 0.15 | automatic checks only | 7 |
| CAC0334-000707M | 4 | A | yes → yes | 2 → 0 | 1.0 | fixed | 3 |
| CAC0334-000734M | unrated | C | no → yes | – → 0 | 0.41 | automatic checks only | 1 |
| CAC0334-000736M | unrated | B | no → yes | – → 0 | – | automatic checks only | 40 |
| CAC0334-000744M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| CAC0334-001738M | unrated | B | no → yes | – → 0 | 0.62 | automatic checks only | 0 |
| CAC0334-004126M | unrated | D | no → yes | – → 0 | 0.21 | automatic checks only | 0 |
| CAC0334-007667M | unrated | B | no → yes | – → 0 | 0.64 | automatic checks only | 0 |
| CAC0334-029379M | unrated | A | no → yes | – → 0 | 0.91 | automatic checks only | 0 |
| CAC0517-002668M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| CAC0517-004166M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| CAC0534-007294M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| PAC0334-001199M | unrated | D | no → yes | – → 0 | 0.26 | automatic checks only | 1 |
| PAC0334-003436M | unrated | B | no → yes | – → 0 | 0.77 | automatic checks only | 0 |
| PAC0334-005211M | unrated | C | no → yes | – → 0 | 0.59 | automatic checks only | 0 |
| RHC0234-013945M | unrated | D | no → yes | – → 0 | 0.28 | automatic checks only | 1 |
| RHC0234-014013M | unrated | C | no → yes | – → 0 | 0.46 | automatic checks only | 0 |
| RHC0234-015136M | unrated | D | no → yes | – → 0 | 0.16 | automatic checks only | 0 |
| RHC0234-019257M | unrated | C | no → yes | – → 0 | 0.42 | automatic checks only | 0 |
| RHC0234-021038M | unrated | B | no → yes | – → 0 | 0.69 | automatic checks only | 0 |
| RHC0234-022084M | unrated | B | no → yes | – → 0 | 0.68 | automatic checks only | 0 |
| RHC0234-023128M | unrated | D | no → yes | – → 4 | 0.61 | unfixable | 0 |
| RHC0234-023194M | unrated | B | no → yes | – → 2 | – | fixed | 4 |
| RHC0234-024333M | unrated | D | no → yes | – → 1 | 0.33 | unfixable | 0 |
| RHC0234-026933M | unrated | D | no → yes | – → 0 | 0.21 | automatic checks only | 0 |
| RHC0234-033232M | unrated | C | no → yes | – → 0 | 0.39 | automatic checks only | 0 |
| RHC0234-033289M | unrated | D | no → yes | – → 0 | 0.07 | automatic checks only | 0 |
| RHC0234-035753M | unrated | D | no → yes | – → 0 | 0.2 | automatic checks only | 0 |
| RHC0334-013897M | unrated | C | no → yes | – → 0 | 0.38 | automatic checks only | 0 |
| RHC0334-014011M | 2 | A | yes → yes | 1 → 0 | 0.97 | fixed | 2 |
| RHC0334-014286M | unrated | D | no → yes | – → 1 | 0.76 | unfixable | 0 |
| RHC0334-014497M | unrated | D | no → yes | – → 1 | 0.36 | unfixable | 0 |
| RHC0334-014670M | unrated | B | no → yes | – → 0 | 0.67 | automatic checks only | 0 |
| RHC0334-014672M | unrated | C | no → yes | – → 0 | 0.52 | automatic checks only | 0 |
| RHC0334-014674M | unrated | B | no → yes | – → 0 | 0.65 | automatic checks only | 0 |
| RHC0334-014676M | 4 | A | yes → yes | 4 → 0 | 1.0 | fixed | 6 |
| RHC0334-015047M | unrated | B | no → yes | – → 0 | 0.6 | automatic checks only | 0 |
| RHC0334-015988M | 5 | B | yes → yes | 0 → 0 | – | automatic checks only | 0 |
| RHC0334-016621M | unrated | D | no → yes | – → 6 | 0.44 | unfixable | 0 |
| RHC0334-016805M | unrated | D | no → yes | – → 4 | 0.84 | unfixable | 0 |
| RHC0334-017235M | unrated | B | no → yes | – → 0 | 0.84 | automatic checks only | 0 |
| RHC0334-017483M | unrated | C | no → yes | – → 0 | 0.44 | automatic checks only | 16 |
| RHC0334-019017M | unrated | C | no → yes | – → 0 | 0.54 | automatic checks only | 0 |
| RHC0334-019290M | 2 | B | yes → yes | 0 → 0 | 0.77 | automatic checks only | 0 |
| RHC0334-019366M | unrated | D | no → yes | – → 2 | 0.83 | unfixable | 0 |
| RHC0334-019373M | unrated | A | no → yes | – → 0 | 1.0 | fixed | 130 |
| RHC0334-019387M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0334-019389M | unrated | D | no → yes | – → 2 | 0.91 | unfixable | 0 |
| RHC0334-019401M | unrated | C | no → yes | – → 0 | 0.5 | automatic checks only | 0 |
| RHC0334-019492M | unrated | D | no → yes | – → 2 | 0.46 | unfixable | 1 |
| RHC0334-019501M | unrated | B | no → yes | – → 0 | 0.83 | automatic checks only | 0 |
| RHC0334-019749M | unrated | B | no → yes | – → 0 | 0.65 | automatic checks only | 0 |
| RHC0334-019827M | 2 | C | yes → yes | 18 → 8 | 0.48 | fixed | 35 |
| RHC0334-020085M | 3 | B | yes → yes | 0 → 0 | 0.75 | automatic checks only | 0 |
| RHC0334-020243M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0334-020305M | unrated | D | no → yes | – → 16 | 0.56 | unfixable | 1 |
| RHC0334-020454M | unrated | D | no → yes | – → 3 | 0.46 | unfixable | 1 |
| RHC0334-020500M | unrated | C | no → yes | – → 0 | 0.5 | automatic checks only | 0 |
| RHC0334-020509M | unrated | D | no → yes | – → 1 | 0.45 | unfixable | 0 |
| RHC0334-020567M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0334-021014M | unrated | A | no → yes | – → 0 | 1.0 | automatic checks only | 0 |
| RHC0334-021108M | unrated | B | no → yes | – → 0 | 0.67 | automatic checks only | 0 |
| RHC0334-021112M | unrated | C | no → yes | – → 0 | 0.58 | automatic checks only | 0 |
| RHC0334-021389M | unrated | B | no → yes | – → 0 | – | fixed | 3 |
| RHC0334-021393M | unrated | D | no → yes | – → 0 | 0.24 | automatic checks only | 0 |
| RHC0334-021395M | unrated | B | no → yes | – → 0 | 0.67 | automatic checks only | 0 |
| RHC0334-021564M | unrated | B | no → yes | – → 0 | 0.72 | automatic checks only | 0 |
| RHC0334-021640M | unrated | B | no → yes | – → 0 | 0.71 | automatic checks only | 0 |
| RHC0334-022072M | unrated | B | no → yes | – → 0 | 0.81 | automatic checks only | 0 |
| RHC0334-022185M | unrated | D | no → yes | – → 8 | 0.41 | unfixable | 1 |
| RHC0334-022187M | unrated | D | no → yes | – → 6 | 0.47 | unfixable | 20 |
| RHC0334-022189M | unrated | C | no → yes | – → 0 | 0.52 | fixed | 2 |
| RHC0334-022285M | unrated | B | no → yes | – → 0 | 0.65 | automatic checks only | 0 |
| RHC0334-022428M | unrated | D | no → no | – → – | – | unfixable | 1 |
| RHC0334-022512M | unrated | D | no → yes | – → 5 | 0.53 | unfixable | 0 |
| RHC0334-022743M | unrated | B | no → yes | – → 0 | 0.63 | automatic checks only | 0 |
| RHC0334-023118M | unrated | D | no → yes | – → 5 | 0.4 | unfixable | 0 |
| RHC0334-023355M | unrated | C | no → yes | – → 0 | 0.5 | automatic checks only | 0 |
| RHC0334-023577M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0334-023991M | unrated | B | no → yes | – → 0 | 0.83 | automatic checks only | 0 |
| RHC0334-023995M | unrated | C | no → yes | – → 0 | 0.52 | automatic checks only | 0 |
| RHC0334-023999M | unrated | B | no → yes | – → 0 | 0.61 | automatic checks only | 0 |
| RHC0334-024001M | unrated | C | no → yes | – → 0 | 0.36 | automatic checks only | 0 |
| RHC0334-024005M | 5 | A | yes → yes | 1 → 1 | 1.0 | no_change | 0 |
| RHC0334-024009M | unrated | D | no → yes | – → 4 | 0.66 | unfixable | 0 |
| RHC0334-024011M | unrated | C | no → yes | – → 0 | 0.36 | automatic checks only | 0 |
| RHC0334-024013M | unrated | C | no → yes | – → 0 | 0.54 | automatic checks only | 0 |
| RHC0334-024017M | unrated | B | no → yes | – → 0 | 0.82 | automatic checks only | 0 |
| RHC0334-024019M | unrated | B | no → yes | – → 0 | 0.61 | automatic checks only | 0 |
| RHC0334-024031M | unrated | D | no → yes | – → 1 | 0.59 | unfixable | 0 |
| RHC0334-024172M | unrated | B | no → yes | – → 0 | 0.68 | fixed | 3 |
| RHC0334-025632M | unrated | C | no → yes | – → 0 | 0.33 | automatic checks only | 0 |
| RHC0334-025728M | unrated | B | no → yes | – → 0 | 0.75 | automatic checks only | 0 |
| RHC0334-025731M | unrated | D | no → yes | – → 0 | 0.28 | automatic checks only | 0 |
| RHC0334-025735M | unrated | B | no → yes | – → 0 | 0.61 | automatic checks only | 0 |
| RHC0334-025742M | 1 | A | yes → yes | 1 → 0 | 1.0 | fixed | 76 |
| RHC0334-027897M | unrated | D | no → yes | – → 1 | 0.71 | unfixable | 0 |
| RHC0334-027899M | unrated | C | no → yes | – → 0 | 0.38 | automatic checks only | 0 |
| RHC0334-027901M | unrated | C | no → yes | – → 0 | 0.57 | automatic checks only | 0 |
| RHC0334-028528M | unrated | D | no → no | – → – | – | unfixable | 0 |
| RHC0334-028530M | unrated | A | no → yes | – → 2 | 0.86 | automatic checks only | 0 |
| RHC0334-028534M | unrated | B | no → yes | – → 0 | 0.67 | automatic checks only | 0 |
| RHC0334-028538M | 4 | B | yes → yes | 0 → 0 | – | automatic checks only | 0 |
| RHC0334-028542M | unrated | B | no → yes | – → 0 | 0.73 | automatic checks only | 0 |
| RHC0334-028550M | unrated | A | no → yes | – → 0 | 0.95 | automatic checks only | 0 |
| RHC0334-028552M | unrated | B | no → yes | – → 0 | 0.75 | automatic checks only | 0 |
| RHC0334-030089M | unrated | C | no → yes | – → 0 | 0.44 | automatic checks only | 0 |
| RHC0334-030695M | unrated | B | no → yes | – → 0 | 0.79 | automatic checks only | 0 |
| RHC0334-030781M | unrated | D | no → yes | – → 1 | 0.64 | unfixable | 0 |
| RHC0334-030783M | unrated | B | no → yes | – → 0 | 0.64 | automatic checks only | 0 |
| RHC0334-030785M | unrated | D | no → yes | – → 5 | 0.63 | unfixable | 0 |
| RHC0334-030787M | unrated | D | no → yes | – → 3 | 0.45 | unfixable | 0 |
| RHC0334-030789M | unrated | D | yes → yes | 0 → 0 | 0.25 | automatic checks only | 0 |
| RHC0334-030791M | unrated | D | no → yes | – → 3 | 0.41 | unfixable | 0 |
| RHC0334-030793M | unrated | C | no → yes | – → 0 | 0.59 | automatic checks only | 0 |
| RHC0334-030795M | unrated | D | no → yes | – → 1 | 0.5 | unfixable | 0 |
| RHC0334-031165M | unrated | C | no → yes | – → 0 | 0.4 | automatic checks only | 0 |
| RHC0334-031379M | unrated | B | yes → yes | 0 → 0 | 0.68 | automatic checks only | 0 |
| RHC0334-031915M | unrated | D | no → yes | – → 6 | 0.83 | unfixable | 9 |
| RHC0334-033775M | 5 | A | yes → yes | 0 → 0 | 0.91 | automatic checks only | 0 |
| RHC0334-033777M | 5 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| RHC0334-033779M | 5 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| RHC0334-033900M | unrated | C | no → yes | – → 0 | 0.33 | automatic checks only | 0 |
| RHC0334-033984M | unrated | C | no → yes | – → 0 | 0.41 | automatic checks only | 0 |
| RHC0334-034104M | unrated | B | no → yes | – → 0 | 0.63 | automatic checks only | 1 |
| RHC0334-034277M | unrated | B | no → yes | – → 0 | 0.67 | automatic checks only | 0 |
| RHC0334-034287M | unrated | A | no → yes | – → 0 | 1.0 | automatic checks only | 0 |
| RHC0334-035534M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0334-036744M | unrated | A | no → yes | – → 0 | 0.9 | automatic checks only | 0 |
| RHC0334-37335M | unrated | C | no → yes | – → 0 | 0.5 | automatic checks only | 0 |
| RHC0334-37340M | 5 | B | yes → yes | 0 → 0 | 0.65 | automatic checks only | 0 |
| RHC0334-37342M | 5 | B | yes → yes | 0 → 0 | 0.65 | automatic checks only | 0 |
| RHC0334-37344M | unrated | C | no → yes | – → 0 | 0.58 | automatic checks only | 0 |
| RHC0502-022909M | unrated | D | no → no | – → – | – | unfixable | 0 |
| RHC0517-025657M | unrated | B | no → yes | – → 0 | – | partly_fixed | 1 |
| RHC0517-025661M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0517-025669M | unrated | B | no → yes | – → 0 | – | fixed | 1 |
| RHC0517-025739M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0517-027380M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0517-033715M | 1 | C | yes → yes | 49 → 44 | – | partly_fixed | 7 |
| RHC0518-014126M | unrated | B | no → yes | – → 0 | – | automatic checks only | 1 |
| RHC0518-014158M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-014404M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-014834M | unrated | B | no → yes | – → 0 | – | partly_fixed | 2 |
| RHC0518-015128M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-015435M | unrated | B | no → yes | – → 0 | – | partly_fixed | 5 |
| RHC0518-016290M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-016367M | 5 | B | yes → yes | 1 → 0 | – | fixed | 3 |
| RHC0518-016373M | unrated | B | no → yes | – → 0 | – | fixed | 4 |
| RHC0518-016375M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-016379M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-016428M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-016430M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-016432M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-016434M | unrated | B | no → yes | – → 0 | – | fixed | 6 |
| RHC0518-016436M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-017144M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-017146M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-017148M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-018060M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-018062M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-018204M | unrated | B | no → yes | – → 2 | – | fixed | 7 |
| RHC0518-019359M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-020159M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-020161M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-020163M | unrated | B | no → yes | – → 0 | – | fixed | 2 |
| RHC0518-020165M | unrated | B | no → yes | – → 0 | – | automatic checks only | 1 |
| RHC0518-020167M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-021114M | unrated | C | no → yes | – → 1 | – | partly_fixed | 3 |
| RHC0518-021145M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-021253M | unrated | C | no → yes | – → 2 | – | partly_fixed | 17 |
| RHC0518-022216M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-022279M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-022283M | 5 | B | yes → yes | 0 → 0 | – | automatic checks only | 0 |
| RHC0518-022293M | unrated | B | no → yes | – → 0 | – | partly_fixed | 2 |
| RHC0518-022295M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-023168M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-023170M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-023172M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-023198M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-023208M | unrated | B | no → yes | – → 0 | – | partly_fixed | 2 |
| RHC0518-023210M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0518-023212M | unrated | B | no → yes | – → 0 | – | partly_fixed | 15 |
| RHC0518-023214M | 1 | B | yes → yes | 0 → 0 | – | automatic checks only | 0 |
| RHC0534-016443M | unrated | B | no → yes | – → 0 | 0.66 | automatic checks only | 0 |
| RHC0534-018113M | unrated | B | no → yes | – → 0 | 0.69 | automatic checks only | 0 |
| RHC0534-023989M | unrated | A | no → yes | – → 0 | 0.88 | fixed | 3 |
| RHC0534-029042M | unrated | D | no → yes | – → 2 | 0.27 | unfixable | 0 |
| RHC0534-38073M | unrated | B | no → yes | – → 0 | 0.69 | automatic checks only | 0 |
| RHC0534-38076M | unrated | A | no → yes | – → 0 | 0.94 | fixed | 103 |
| RHC0552-37268M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0552-37717M | unrated | B | no → yes | – → 0 | – | partly_fixed | 4 |
| RHC0552-37719M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0552-37721M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| RHC0552-37723M | unrated | B | no → yes | – → 0 | – | fixed | 1 |
| RHC0552-37725M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| SCC0103-033263M | 5 | B | yes → yes | 0 → 0 | – | automatic checks only | 0 |
| SCC0117-005401M | 5 | A | yes → yes | 1 → 1 | 1.0 | fixed | 3 |
| SCC0234-005405M | unrated | C | no → yes | – → 0 | 0.43 | automatic checks only | 0 |
| SCC0234-005411M | unrated | B | no → yes | – → 0 | 0.75 | automatic checks only | 0 |
| SCC0234-005435M | unrated | D | no → yes | – → 2 | 0.64 | unfixable | 0 |
| SCC0234-005485M | unrated | D | no → yes | – → 4 | 0.38 | unfixable | 0 |
| SCC0234-005766M | unrated | D | no → yes | – → 6 | 0.33 | unfixable | 1 |
| SCC0234-033255M | 3 | A | yes → yes | 1 → 0 | 0.92 | fixed | 18 |
| SCC0334-003573M | 4 | B | yes → yes | 0 → 0 | – | automatic checks only | 0 |
| SCC0334-003582M | 5 | D | yes → yes | 4 → 4 | 0.38 | unfixable | 0 |
| SCC0334-003586M | 5 | D | yes → yes | 4 → 4 | 0.44 | unfixable | 0 |
| SCC0334-003615M | unrated | D | no → yes | – → 0 | 0.12 | automatic checks only | 0 |
| SCC0334-003625M | unrated | D | no → yes | – → 0 | 0.12 | automatic checks only | 0 |
| SCC0334-003631M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| SCC0334-003652M | unrated | B | no → yes | – → 0 | – | automatic checks only | 0 |
| SCC0334-003656M | 4 | A | yes → yes | 1 → 0 | – | fixed | 7 |
| SCC0334-003679M | 1 | A | no → yes | – → 0 | – | fixed | 27 |
| SCC0334-004365M | 1 | A | no → yes | – → 0 | – | fixed | 65 |
| SCC0334-004372M | 3 | A | yes → yes | 1 → 0 | – | fixed | 8 |
| SCC0334-004378M | unrated | C | no → yes | – → 0 | 0.41 | automatic checks only | 0 |
| SCC0334-004442M | 3 | A | yes → yes | 0 → 0 | 0.86 | automatic checks only | 0 |
| SCC0334-004467M | unrated | D | no → yes | – → 4 | 0.26 | unfixable | 1 |
| SCC0334-004469M | unrated | D | no → yes | – → 2 | 0.27 | unfixable | 2 |
| SCC0334-004471M | unrated | D | no → yes | – → 4 | 0.32 | unfixable | 1 |
| SCC0334-004473M | unrated | D | no → yes | – → 2 | 0.29 | unfixable | 0 |
| SCC0334-005462M | unrated | D | no → yes | – → 8 | 0.35 | unfixable | 16 |
| SCC0334-005471M | unrated | D | no → yes | – → 2 | 0.45 | unfixable | 0 |
| SCC0334-005483M | 3 | A | yes → yes | 2 → 0 | 0.95 | fixed | 158 |
| SCC0334-005570M | unrated | D | no → yes | – → 2 | 0.19 | unfixable | 0 |
| SCC0334-005572M | unrated | D | no → yes | – → 2 | 0.35 | unfixable | 0 |
| SCC0334-006891M | 4 | A | yes → yes | 3 → 1 | 1.0 | fixed | 4 |
| SCC0334-009380M | 2 | C | no → yes | – → 1 | 0.95 | partly_fixed | 56 |
| SCC0334-009502M | 4 | A | yes → yes | 2 → 2 | 1.0 | automatic checks only | 0 |
| SCC0334-026025M | 2 | A | no → yes | – → 1 | 1.0 | fixed | 44 |
| SCC0334-026037M | 3 | A | yes → yes | 8 → 2 | 1.0 | fixed | 44 |
| SCC0334-030366M | 5 | B | yes → yes | 0 → 0 | 0.71 | automatic checks only | 0 |
| SCC0334-036678M | 3 | A | no → yes | – → 1 | 1.0 | fixed | 100 |
| SCC0501-002123M | 3 | B | yes → yes | 6 → 0 | – | fixed | 6 |
| SCC0518-005324M | 1 | B | yes → yes | 8 → 0 | – | partly_fixed | 21 |
| SCC0534-005399M | 4 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| SCC0534-007746M | 3 | A | yes → yes | 0 → 0 | 1.0 | automatic checks only | 0 |
| SCC0534-035767M | 1 | C | yes → yes | 18 → 1 | 0.38 | partly_fixed | 122 |
| SCC0537-004479M | 4 | B | no → yes | – → 0 | – | automatic checks only | 1 |
