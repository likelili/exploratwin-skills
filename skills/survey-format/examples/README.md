# Examples

Every survey here passes ExploraTwin's checks (a test keeps them passing).

## Surveys in the standard format

- [surveys/plain-survey.json](surveys/plain-survey.json): a short survey of common question types
- [surveys/logic-and-loop.json](surveys/logic-and-loop.json): display and skip logic, a branch, a random arm, piped text, a repeated block
- [surveys/conjoint.json](surveys/conjoint.json): a choice-based conjoint
- [surveys/conjoint-answer-formats.json](surveys/conjoint-answer-formats.json): conjoint tasks rated, ranked, split and answered best and worst (version 0.2)
- [surveys/adaptive-conjoint.json](surveys/adaptive-conjoint.json): an adaptive conjoint, each task built from the twin's earlier answers (version 0.2)
- [surveys/maxdiff.json](surveys/maxdiff.json): a MaxDiff block

## Worked conversions

Each folder holds the source as a document gives it (`source.*`), the converted
survey (`survey.json`) and its conversion report (`report.md`).

- [conversions/word-questionnaire/](conversions/word-questionnaire/): a Word questionnaire with programming notes: a screen-out, a skip, randomization with an anchored option, a grid, a ranking, a 0 to 10 scale
- [conversions/google-forms/](conversions/google-forms/): a printed Google Form: sections, "go to section based on answer", a linear scale, "Other:", a grid, a drop-down
- [conversions/redcap-dictionary/](conversions/redcap-dictionary/): a REDCap data dictionary: branching logic, choice codes, validation limits, a slider, a matrix group, fields left out
- [conversions/surveymonkey-pdf/](conversions/surveymonkey-pdf/): a SurveyMonkey PDF: star rating, matrix, ranking, Net Promoter Score, multiple textboxes, a file upload left out
