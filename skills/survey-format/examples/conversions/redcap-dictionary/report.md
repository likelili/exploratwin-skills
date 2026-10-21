# Conversion report: wellbeing (REDCap data dictionary)

**Source**: a REDCap data dictionary (CSV) with one instrument, `wellbeing`, of 12 fields.

**What the survey has**: 8 questions and 2 instructions (the two section headers) in one block, in the dictionary's order, with REDCap's field names as IDs and its choice codes as option IDs and codes. Every field shown to respondents is in it, worded as written.

**How the source was read**:
- Branching logic `[consent] = '1'` on each field: each is shown only when the respondent agreed (consent Yes = 1). `[coping(99)] = '1'`: "What else helps you cope?" is shown only when "Something else" was checked.
- `age`, a text field validated as an integer from 18 to 99: a typed whole number with those limits.
- `rested`, a slider: a number from 0 to 100 (REDCap's slider values); its three slider labels, as written, are a line under the question (REDCap shows them at the left end, the middle and the right end).
- The three `stress_*` fields share the matrix group `stress` and its choices: one grid, its section header as the question text; its rows are recorded as `stress_work`, `stress_money` and `stress_health`, REDCap's field names.
- `comments`: its field note "Optional" stays under the question.

**Left out**: `record_id`, REDCap's record ID, which a survey respondent does not answer; `stress_score`, a calculated field marked `@HIDDEN`, which respondents never see.

**Questions for the person**: none.

**The check**: passed, no errors.
