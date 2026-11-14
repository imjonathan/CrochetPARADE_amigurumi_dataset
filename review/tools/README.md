# Check tools

Requires Node 18+ and Python 3.10+, and a checkout of CrochetPARADE for `parse64.js`.

    python3 make_parser.py /path/to/CrochetPARADE/parse64.js parser.mjs   # once
    python3 check.py some_pattern.cp                                       # one pattern
    python3 report.py ../../dsl_patterns.json flags.json                   # whole dataset

- `rows.mjs` runs the parser and emits per-row stitches and attachments.
- `cplib.py` holds the round-closure and stated-count analysis.
- `engalign.py` compares counts stated in an English pattern with compiled per-row counts.
- `normalize.py`, `normalize2.py`, `fix_even.py` are the mechanical repair passes used in this review.

`make_parser.py` restricts CrochetPARADE's expression evaluation to plain arithmetic so that pattern text cannot execute JavaScript. The generated `parser.mjs` is derived from CrochetPARADE (GPLv3) and is not committed.
