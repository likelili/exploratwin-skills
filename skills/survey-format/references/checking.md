# Checking a survey file

Check every survey you write or convert before you hand it over, and fix every
error. There are three ways, from most to least complete.

## 1. On ExploraTwin (with an account)

- **An agent connected to ExploraTwin** (the `exploratwin` skill, with an API
  key): `POST https://exploratwin.org/api/agent/v1/survey-check` with
  `{"definition": {...}}`. It runs every check the website runs and saves
  nothing. The answer lists `problems` and `warnings` (each with `path`,
  `message`, `code`, `level`), the tidied `definition` (with the format version
  it needs), `answers_per_twin`, and, when there are no problems, `runs_in`:
  `"one call"` or `"stages"`.
- **In the website**: start a survey with **Start building**, choose **Build
  your own**, then **Open survey file**. The builder opens the file and lists
  every problem under **Checks**.

ExploraTwin also checks what only it needs:

| Code | Level | Meaning |
|---|---|---|
| `picture.source` | error | A picture that is not an upload (`upload:...`), a web link or a `data:` address. A file path from your computer cannot be opened there. |
| `limit.questions`, `limit.blocks` | error | At most 300 questions and 40 blocks. |
| `limit.tasks`, `limit.attributes`, `limit.levels`, `limit.alternatives`, `limit.items` | error | A conjoint or MaxDiff block: at most 30 tasks per twin, 12 attributes, 15 levels an attribute, 8 alternatives a task, 40 MaxDiff items. |
| `content.empty` | warning | A question with no text. |
| `label.empty` | warning | An option, row or item with no text. |
| `experiment.levels` | warning | A conjoint attribute with one level never varies. |
| `experiment.level_number` | warning | A numeric or price level with no number, or whose label shows another number than its `value`. |
| `repeat.limit` | warning | A repeated block runs at most 10 rounds there; later rows are not asked. |
| `compile` | error | The survey could not be prepared for a run (the message says why). |

## 2. With the surveytwin library (no account)

The surveytwin library ([github.com/likelili/surveytwin-lib](https://github.com/likelili/surveytwin-lib))
reads and checks the format on your computer with the same checks (without the
ExploraTwin-only ones above), and makes no model call. It needs Python 3.11 or
later:

```bash
pip install "surveytwin @ git+https://github.com/likelili/surveytwin-lib"
surveytwin check survey.json
```

A survey with errors prints them and exits with code 2:

```text
error: 2 problems: $.blocks[0].questions[2]: no question has the ID 'nope'; $.questions[1].response.options: two options read 'yes'; the model answers with the label, so labels must differ
```

A survey that reads prints its blocks, how it runs (static, or in stages and
why), each finding, and last `OK` or `Needs attention before a run`. In Python,
`surveytwin.read_survey("survey.json")` raises `SurveyFormatError`, whose
`findings` list each problem with its `path`, `message`, `code` and `level`;
the warnings of a survey that reads are in `survey.read_warnings`.

Version 0.2 of the format (conjoint answer formats other than a choice,
adaptive designs) needs surveytwin 0.3 or later; an older version refuses such a
survey ("must be one of '0.1'"): check it on ExploraTwin instead. The library
takes picture files from your computer (paths relative to the JSON file), which
ExploraTwin does not.

## 3. With the JSON Schema only

[survey-format.schema.json](survey-format.schema.json) (JSON Schema 2020-12)
checks structure only: fields, types, allowed values. Any JSON Schema validator
can use it, for example in Python:

```python
import json, jsonschema
schema = json.load(open("survey-format.schema.json"))
jsonschema.validate(json.load(open("survey.json")), schema)
```

It does not check references, logic, limits, labels or designs. Use it only when
neither of the others is available, then go through the codebook's
[common mistakes](codebook.md#common-mistakes) by hand.

## What each code means

| Code | Level | Meaning, and the fix |
|---|---|---|
| `schema` | error | A missing or unknown field, a wrong type, a value not allowed. The path names the field; compare it with [fields.md](fields.md). |
| `version` | error | The survey uses a feature of version 0.2 but says `"0.1"`: write `"0.2"`. |
| `id.duplicate` | error | Two questions, blocks, flow steps, variables, options, rows, columns or items share an ID. |
| `label.duplicate` | error | Two options (or columns, items, alternatives) of one question read the same, ignoring case: the model answers with the label, so make them differ. |
| `reference.block` | error | The flow shows a block the survey does not define. |
| `reference.question` | error | A block lists a question the survey does not define, or a question is in two blocks. |
| `reference.condition` | error | A condition reads a question the survey does not define, or an option that question does not have. |
| `reference.skip` | error | A skip goes to a question that is not later in the same block. Use `show_if` or a branch instead. |
| `reference.pipe` | error | Piped text reads the answer of a question the survey does not define. |
| `reference.default` | error | A preset answer names an option or item the question does not have, or two items share a preset rank. |
| `reference.randomization` | error | A randomization names an option the question does not have. |
| `reference.experiment` | error | A conjoint or MaxDiff block names an alternative, item or level it does not have. |
| `response.limits` | error | A limit that cannot be met: a minimum above its maximum, more selections than options, a total of 0. |
| `response.mode` | error | A grid without what its mode needs (`row_response`, or `columns` and `cell_response`). |
| `repeat.order` | error | The rows of a repeated block are not listed in the order they run (numbers in numeric order). |
| `repeat.take` | error | `take` on a repeat whose rows are not shuffled. |
| `skip.condition` | error | A skip rule that does not test whether an option of its own question was selected. |
| `branch.else` | error | An `else` that cannot be decided as written: write two branches. |
| `randomize.take` | error | A randomizer that takes more steps than it holds. |
| `flow.empty` | error | Questions but no flow, so nothing would be asked. |
| `question.kind` | error | A question marked as never asked (`kind`, from a Qualtrics file) that also has a response. Do not write `kind`. |
| `response.type` | error | A `best_worst` response on a question: best and worst picks belong to a MaxDiff block. |
| `experiment.design` | error | A conjoint or MaxDiff design that is incomplete or does not fit the block. |
| `experiment.alternatives` | error | Alternatives that cannot be asked in the block's answer format (too few, a `count` with fixed alternatives, a label template with fields other than `{letter}` and `{number}`). |
| `experiment.response` | error | An answer that cannot be asked: best and worst labels the same, 0 points to split. |
| `experiment.constraints` | error | An applicability table that does not list every level once, or a supplied profile that breaks the rules. |
| `experiment.weights` | error | Every level of an attribute has weight 0. |
| `experiment.order` | error | A fixed position outside the order, or two at the same position. |
| `experiment.adaptive` | error | An adaptive design that does not ask a choice of at least two options, uses another generator, or forbids seeing an option twice. |
| `experiment.infeasible` | error | A design that cannot be drawn or built (too few valid profiles; more than 60 levels in an adaptive design). |
| `experiment.shown_twice` | error | A conjoint or MaxDiff block shown more than once. |
| `variable.unset` | warning | A variable is read but never set or declared; it reads as empty. |
| `block.unused` | warning | A block the flow never shows. |
| `question.unused` | warning | A question in no block, so never asked. |
| `experiment.repeated_tasks` | warning | Fewer different tasks can be made than each twin answers, so some repeat. |

Fix errors in the order they are listed: one wrong ID often causes several
reference errors after it. After fixing, check again.
