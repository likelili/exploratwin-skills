# Results

When `GET /runs/{id}` says `completed`, `GET /runs/{id}/exports` lists the
files. Download one with `GET /runs/{id}/exports/{file}`; it may answer with a
redirect to a signed link, so follow redirects (`curl -L`).

| File | What it holds |
|---|---|
| `labels.csv` | One row per twin, one column per answer, answers as labels, laid out as Qualtrics' "choice text" export: three header rows (column names, question text, import IDs), then the data. |
| `values.csv` | The same layout, answers as codes (Qualtrics' "numeric values"). |
| `repaired.csv` | The same layout as `labels.csv`, after any repairs the person made on the website. |
| `wide.csv` | One header row, then one row per twin: the twin's descriptors and survey variables next to the answers. |
| `long.csv` | One row per twin and answer, with the twin's descriptors: for analysis in R, Stata or Python. |
| `conjoint-exposures.csv` | Conjoint blocks: every profile each twin saw, task by task, with the choice. |
| `conjoint-partworths.csv` | Adaptive conjoint blocks: each twin's final estimated value (partworth) of every level, fitted from all its answers, with the estimate's status (an estimate that could not be made has blank values). |
| `maxdiff-exposures.csv` | MaxDiff blocks: every subset each twin saw, with the best and worst picks. |
| `question-logic.json` | The survey as it ran: questions, options, logic. |
| `report.pdf` | The written report, when the run had `report: "full"`. |
| `manifest.json` | What each file holds, column by column. |
| `bundle.zip` | All of the above in one file. |

Only the files that apply to the run are listed.

## Working with results

- Save the files on the person's computer, where they ask (or in a results
  folder of the project you work in). They are the person's research data.
- In the conversation, report summaries: counts, shares, means, differences,
  a few short quotes at most. Do not paste rows of participants' answers.
- A skipped answer is a real outcome (the twin did not answer, or a picture
  could not be loaded): report how many, never fill them in.
- Twins are AI participants built from real people's answers. Describe results
  as what the twins answered, not as what people would answer; the website's
  accuracy notes are at https://exploratwin.org/support/digital-twin-accuracy.
- Repairs and reruns of flagged answers are done by the person on the
  website; ask before suggesting them as something you will do.
