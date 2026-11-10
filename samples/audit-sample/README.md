# Demo audit output

Read-only model audit of [`Northwind FY26 P&L (demo).xlsx`](<Northwind FY26 P&L (demo).xlsx>), a fictional
three-sheet monthly P&L with 221 occupied cells and 133 formulas, built with defects planted on purpose in both the
structure and the numbers.

| File | What it is |
| --- | --- |
| [`audit.md`](audit.md) | The workpaper with verdict, recomputation table, workbook map, findings, limits and sign-off |
| [`audit.html`](audit.html) | The same report as a self-contained page, filterable by severity |
| [`audit.xlsx`](audit.xlsx) | The findings as a filterable Excel sheet, colour-coded by severity |
| [`audit.json`](audit.json) | Structured findings, for a pipeline or dashboard |
| [`Northwind FY26 P&L (demo).xlsx`](<Northwind FY26 P&L (demo).xlsx>) | The input, so the numbers above can be reproduced |

## The result

```
score 92/100 (grade D). Do not rely on this build
high 8  medium 3  low 3
quantified exposure 121,041.08
recomputed 129/133 formulas (97.0%), 2 stale, 1 hidden errors, 1 circular
```

## How it was produced

```bash
python3 audit.py "Northwind FY26 P&L (demo).xlsx" \
        --out ./audit-sample --policy policy.json --html --xlsx
```

Excel Model Audit v2.2.0 ran read-only, and the workbook is byte-identical after the run. The audit script itself
is not part of this repository. These are its published output files, kept here so the numbers in the main README
can be checked rather than taken on trust.
