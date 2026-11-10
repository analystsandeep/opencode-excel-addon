# Model integrity audit, Northwind FY26 P&L (demo).xlsx

| Prepared | Tool | Mode | Currency |
| --- | --- | --- | --- |
| 2026-11-10 10:04 UTC | Excel Model Audit v2.2.0 | read-only, the workbook was never modified | not applicable |

## Verdict

| Risk score | Grade | Reading | High | Medium | Low |
| --- | --- | --- | --- | --- | --- |
| 92/100 | D | Do not rely on this build | 8 | 3 | 3 |

3 sheets, 221 occupied cells, 133 formulas, 1 defined names, 1 external workbook link(s).

Quantified exposure is 121,041.08, the sum of the differences this audit could measure, in the workbook's own units. It is a floor rather than a ceiling, because structure that cannot be evaluated is not counted in it.

## Recomputation

| Formulas | Recomputed and compared | Coverage | Stale results | Hidden errors | Circular |
| --- | --- | --- | --- | --- | --- |
| 133 | 129 | 97.0% | 2 | 1 | 1 |

Formulas outside the engine's library are reported and not guessed. They are [external workbook] x1.

## Workbook map

| Sheet | State | Used range | Cells | Formulas | Typed numbers |
| --- | --- | --- | --- | --- | --- |
| P&L | visible | A1:M33 | 186 | 133 | 18 |
| Inputs | visible | A1:B9 | 14 | 0 | 6 |
| Old workings | hidden | A1:B11 | 21 | 0 | 10 |

## Findings

| # | Severity | Check | Sheet | Cells | Finding |
| --- | --- | --- | --- | --- | --- |
| 1 | HIGH | broken_name |  | PriorYearTotal | Defined name points at a deleted range |
| 2 | HIGH | circular_reference |  | P&L!G33, P&L!H33 | Circular reference (1 loop) |
| 3 | HIGH | external_links |  | B25 | Workbook depends on another file |
| 4 | HIGH | tie_out |  |  | Tie-out fails for Total revenue equals the sum of every product line |
| 5 | HIGH | hardcoded_in_run | P&L | H16 | Typed value inside a calculated row |
| 6 | HIGH | hidden_error | P&L | B23 | Cell shows a value but the formula resolves to an error |
| 7 | HIGH | stale_cache | P&L | B12, C19 | Stored result does not match the formula |
| 8 | HIGH | sum_coverage | P&L | B9, C9, D9, E9 ... | Total excludes the row or column next to its range |
| 9 | MEDIUM | hidden_sheets |  | Old workings | Hidden sheets carry content (1) |
| 10 | MEDIUM | format_mixing | P&L | I21 | Same block displayed as both percent and decimal |
| 11 | MEDIUM | pattern_deviation | P&L | J5 | Formula does not follow its row pattern |
| 12 | LOW | no_cached_results |  | P&L!B25, P&L!B27, P&L!G33, P&L!H33 | Formulas have no stored results |
| 13 | LOW | hidden_rows_cols | P&L | A28, A29 | Hidden rows or columns inside the used range |
| 14 | LOW | volatile_function | P&L | B27 | NOW() used 1 time |

## Detail

### 1. Defined name points at a deleted range  
*High severity, check `broken_name`*

Name 'PriorYearTotal' refers to #REF!.

Why it matters. Any formula using this name is now pointing at nothing, and the failure can be masked by a cached value that still looks correct.

Suggested fix. Re-point 'PriorYearTotal' at the intended range or delete it, then search for its uses and correct them.

### 2. Circular reference (1 loop)  
*High severity, check `circular_reference`*

P&L!G33
P&L!H33
P&L!G33

Why it matters. A circular reference means the figure depends on itself. Excel resolves it iteratively, so the value drifts with every recalculation and is not reproducible from the inputs.

Suggested fix. Break the loop. Identify the cell that should be an input rather than a calculation, and replace one leg of the cycle with a typed assumption placed outside the loop.

### 3. Workbook depends on another file  
*High severity, check `external_links`*

References point outside this workbook. The files are Prior year P&L.xlsx.

Why it matters. External links break silently. The recipient sees stale cached values, and any figure that came from the other file is no longer verifiable from this one.

Suggested fix. Break the links. Paste the required values as labelled inputs on a source sheet, or rebuild the reference inside the workbook. Then check that no formula still contains a bracketed file name.

### 4. Tie-out fails for Total revenue equals the sum of every product line  
*High severity, check `tie_out`*

'P&L'!B9-SUM('P&L'!B5:B8) evaluates to -404.00 against a tolerance of 0.50, so it is out by 404.00.

Why it matters. Reconciliations exist because these two figures are supposed to agree. When they do not, at least one of them is wrong.

Quantified impact. 404.00 in the workbook's units.

Suggested fix. Trace both sides to their inputs and correct the one that is incomplete. Re-run after the fix.

### 5. Typed value inside a calculated row  
*High severity, check `hardcoded_in_run`, sheet P&L*

Row 16 is calculated across B16:M16 but these cells hold typed numbers, starting with H16=12500. The pattern at B16 would give H16 = 492.55, so the typed value is off by +12,007.45.

Why it matters. A hardcoded number inside a calculated row is the most common way a model quietly stops tying. It survives every future update because nothing recalculates it.

Quantified impact. 12,007.45 in the workbook's units.

Suggested fix. Replace each typed value with the formula its neighbours use, or move the assumption to an input sheet and reference it so the override is visible and labelled.

### 6. Cell shows a value but the formula resolves to an error  
*High severity, check `hidden_error`, sheet P&L*

1 cell(s) carry a stored number while the formula evaluates to an error. For example, P&L!B23 stores 1370 but ='Old workings v0'!B2 resolves to #REF!.

Why it matters. The reader sees a plausible number. The moment anyone recalculates, or links to the cell from another model, it becomes an error or a blank. This is the most dangerous class of defect in a workbook that is about to be circulated.

Suggested fix. Correct the formula at its source, then recompute and confirm the cell now shows the intended figure.

### 7. Stored result does not match the formula  
*High severity, check `stale_cache`, sheet P&L*

2 cell(s) hold a cached value that disagrees with a recomputation of the formula. For example, P&L!B12 shows 99999 but =B9-B11 evaluates to 2175.0 (difference -97,824.00).

Why it matters. The number on the page is not the number the formula produces. This happens when a file is written by a script or another tool, when a workbook is edited outside Excel, or when a calculation was interrupted. Every reader trusts the cached value, so the error travels silently.

Quantified impact. 102,989.62 in the workbook's units.

Suggested fix. Recompute in Excel (F9), save, and re-run. If the recomputation changes a figure you have already reported, treat the cached value as wrong and trace what changed.

### 8. Total excludes the row or column next to its range  
*High severity, check `sum_coverage`, sheet P&L*

12 total cells in row 9 sum B5:B7 but the row directly outside the range (B8:M8) holds a value of 404 and is otherwise part of the same block. The totals affected are B9, C9, D9, E9, F9, G9 (+6 more). The excluded values sum to 5,640.00.

Why it matters. The classic broken model. A line is added after the total was built, the SUM range does not grow, and the figure is understated with no visible error. Nothing on the page looks wrong.

Quantified impact. 5,640.00 in the workbook's units.

Suggested fix. Confirm the block is meant to include it, then extend each total along its own axis, from B5:B7 to B5:B8 (for example B9 changes from =SUM(B5:B7) to =SUM(B5:B8)), or delete the orphan row if it should not be there.

### 9. Hidden sheets carry content (1)  
*Medium severity, check `hidden_sheets`*

Not visible in the workbook tabs. They are Old workings.

Why it matters. A sheet the reader cannot see is a place where inputs, workings or stale calculations survive review. Reviewers routinely miss them.

Suggested fix. Unhide each sheet, decide whether it is load-bearing, and either label it as workings or delete it.

### 10. Same block displayed as both percent and decimal  
*Medium severity, check `format_mixing`, sheet P&L*

Row 21 holds 12 numbers, 11 formatted as percent and 1 as plain number. The ambiguous cells are I21. For example I21 displays as a plain 0.25.

Why it matters. A cell showing 0.07 and a cell showing 7% read as different magnitudes even when they hold the same number. In a margin or growth block that is how an order-of-magnitude error survives review.

Suggested fix. Apply one convention to the whole row, then re-check the dependent rows still foot after the change.

### 11. Formula does not follow its row pattern  
*Medium severity, check `pattern_deviation`, sheet P&L*

J5 differs from the pattern used by 10 other cells on the same row 5. The pattern is =B5*1.03, as at C5. The deviation is J5 =I5+0.03.

Why it matters. Fill-across models rely on every cell in a period row being the same formula shifted. A deviation is either a deliberate exception that should be labelled or a copy-paste slip that will not survive the next roll-forward.

Suggested fix. Compare each listed formula with the pattern next to it and either restore the pattern or add a note explaining the exception.

### 12. Formulas have no stored results  
*Low severity, check `no_cached_results`*

4 formula cells carry no cached value, e.g. P&L!B25, P&L!B27, P&L!G33, P&L!H33.

Why it matters. The workbook has not been recalculated since it was last written by a tool or script. Any figures another system reads from it may be blank or stale, and no formula audit alone can confirm the expected result.

Suggested fix. Open the workbook in Excel, let it calculate fully, save, and re-run this audit so the check can compare formula and result.

### 13. Hidden rows or columns inside the used range  
*Low severity, check `hidden_rows_cols`, sheet P&L*

2 hidden row(s) (28, 29) and 0 hidden column(s).

Why it matters. Hidden rows and columns inside a data block are the standard way a period or entity is excluded from a total without anyone noticing.

Suggested fix. Unhide and review each one, or delete it. If a column is hidden only for presentation, move it out of the data block.

### 14. NOW() used 1 time  
*Low severity, check `volatile_function`, sheet P&L*

NOW() appears in B27.

Why it matters. NOW is volatile and changes every time the file recalculates. In an audit trail that means the reported figure cannot be reproduced from the file alone.

Suggested fix. Replace with a fixed input or a non-volatile equivalent where possible, and document the assumption date where it cannot be avoided.

## Checks run

- `stale_cache`, Stored results that disagree with a recomputation
- `hidden_error`, Cells showing a value while the formula resolves to an error
- `circular_reference`, Circular references, found by the calculation engine
- `tie_out`, Buyer-defined reconciliation expressions from policy.json
- `engine_coverage`, How much of the workbook the engine could recompute
- `error_values`, Formula errors such as #REF!, #DIV/0!, #VALUE! and #N/A
- `hardcoded_in_run`, Typed values sitting inside calculated rows and columns
- `pattern_deviation`, Cells whose formula breaks the row or column fill pattern
- `sum_coverage`, Totals whose range stops short of the block they total
- `external_links`, References to another workbook file
- `volatile_function`, Volatile functions (NOW, RAND, OFFSET, INDIRECT)
- `hidden_sheets`, Hidden and very-hidden sheets
- `hidden_rows_cols`, Hidden rows and columns inside the used range
- `format_mixing`, Percent and decimal formatting mixed in one column
- `broken_name`, Defined names pointing at deleted ranges or missing sheets
- `long_formula`, Over-long formulas and deeply nested IF logic
- `merged_data`, Merged cells covering populated ranges
- `no_cached_results`, Formula cells with no stored result

## Limits of this audit

This is a structural and numerical review, not an audit opinion. Formulas are re-derived by the package's own calculation engine and compared with the values stored in the file. It does not run Excel, Power Query or VBA, and any function outside the engine's library is reported as uncovered rather than guessed. LET and LAMBDA are evaluated, including a LAMBDA held in a defined name and a LAMBDA that calls itself, and recursion is capped so a runaway one returns #NUM! instead of hanging. Dynamic-array behaviour, which covers spilling, implicit intersection and array broadcasting, is not modelled. A formula that depends on it is reported as uncovered and never approximated. It cannot confirm that a formula encodes the intended business logic, and it does not evaluate accounting policy, tax treatment or the reasonableness of an assumption.

## Sign-off

| | Name | Date |
| --- | --- | --- |
| Prepared by | | |
| Reviewed by | | |
