# The standard survey format

A survey in the standard format is one JSON document that says everything a run
needs: what each question shows, what it asks for, and how the survey moves
from question to question. The website's survey builder writes this format,
and you write it too. The prompts a twin reads come from these fields.

Versions: `0.1`, and `0.2`, which adds conjoint answer formats other than a
choice and adaptive conjoint designs. Write `"schema_version": "0.1"`; the
server sets the version a survey needs when it checks or saves it. The machine-readable
definition is [survey-format.schema.json](survey-format.schema.json) (also
`GET /survey-format`). It checks structure only: `POST /survey-check` runs every
check, and its answer is what counts.

The full codebook (every field, how a twin reads it, common mistakes) and the
guide to converting surveys from other tools and documents are in the
survey-format skill: https://exploratwin.org/skills/survey-format/SKILL.md.

Worked examples, each passing the server's checks:
[plain-survey.json](../examples/plain-survey.json),
[logic-and-loop.json](../examples/logic-and-loop.json),
[conjoint.json](../examples/conjoint.json),
[adaptive-conjoint.json](../examples/adaptive-conjoint.json),
[maxdiff.json](../examples/maxdiff.json).

## A small survey

```json
{
  "schema_version": "0.1",
  "id": "pets",
  "title": "Pets",
  "questions": [
    {
      "id": "has_pet",
      "content": [{"type": "text", "text": "Do you have a pet?"}],
      "response": {
        "type": "choice",
        "mode": "single",
        "required": true,
        "options": [{"id": "yes", "label": "Yes"}, {"id": "no", "label": "No"}]
      }
    },
    {
      "id": "pet_name",
      "show_if": {"op": "selected", "question": "has_pet", "option": "yes"},
      "content": [{"type": "text", "text": "What is your pet's name?"}],
      "response": {"type": "text", "max_length": 40}
    }
  ],
  "blocks": [{"id": "main", "kind": "survey", "questions": ["has_pet", "pet_name"]}],
  "flow": [{"type": "show_block", "block": "main"}]
}
```

Every question is in a block, and the flow shows the blocks. A question in no
block, or a block the flow never shows, is never asked.

## How a survey is checked

Two passes, and nothing runs until both pass:

1. **Shape.** Every object has the fields this page lists, of the right type;
   required fields are present; a field not listed here is an error (a key
   starting with `x-` is allowed anywhere, kept and never used).
2. **Meaning.** IDs are unique where answers are recorded by them; every
   reference resolves (a block in the flow, a question in a block, an option in
   a condition, a skip target later in the same block, a piped answer); limits
   are possible; pictures can be loaded; and no question offers two options
   with the same label, since the model answers with the label.

Each problem names its field as a path (`$.questions[2].response.options[1].label`),
a `code` and a `level`: `error` stops the survey, `warning` does not. Common codes:

| Code | Meaning |
|---|---|
| `schema` | A missing or unknown field, a wrong type, a value not allowed. |
| `id.duplicate` | Two questions, blocks, flow steps, variables, options, rows, columns or items share an ID. |
| `label.duplicate` | Two options (or columns, or items) of one question read the same, ignoring case. |
| `reference.block`, `reference.question`, `reference.condition`, `reference.skip`, `reference.pipe` | A reference to something the survey does not define (or a skip that does not go forward in its block). |
| `response.limits` | A limit that cannot be met (a minimum above its maximum, more selections than options). |
| `picture.source` | A picture that is not an uploaded image (`upload:...`) or a web link. |
| `experiment.design`, `experiment.adaptive`, `experiment.infeasible` | A conjoint or MaxDiff design that is incomplete or cannot be drawn. |
| `flow.empty` | Questions but no flow, so nothing would be asked. |
| `variable.unset`, `block.unused`, `question.unused` | Warnings: a variable read but never set, a block never shown, a question in no block. |

## The survey

| Field | Meaning |
|---|---|
| `schema_version` | Required. `"0.1"` (or `"0.2"`). |
| `id` | Required. The survey's ID (no spaces). |
| `title` | The survey's name; the project is named after it. |
| `description`, `language` | For people; `language` is a code such as `"EN"`. |
| `variables` | The variables the survey uses (see [Variables](#variables)). |
| `questions` | Every question and instruction, each once. |
| `blocks` | Groups of questions shown together, and experiment blocks. |
| `flow` | The order blocks are shown in, with conditions, random arms and variables. |

## Questions

| Field | Meaning |
|---|---|
| `id` | Required. Answers are recorded by it; keep it stable when you edit. |
| `export_name` | The column name in exports (Qualtrics' "data export tag"). |
| `content` | Required. What the question shows, in order (see below). |
| `response` | What the question asks for. Leave it out for an instruction (shown, never answered). |
| `show_if` | A [condition](#conditions): the question is shown only when it holds. |
| `skip` | [Skip rules](#skip-rules): after the answer, jump ahead when a rule holds. |
| `presentation` | How the response is shown (see [Presentation](#presentation)). |

### Content

| `type` | Fields | Shown as |
|---|---|---|
| `text` | `text` (required); `format`: `"plain"` (default) or `"html"` | The text. Plain text keeps its line breaks; HTML is read as a browser shows it, without its markup. |
| `image` | `source` (required), `alt`, `title` | The picture, sent to the model where it stands among the content. |
| `table` | `columns` (each `id`, `label`), `rows` (each `id`, `label`, `cells` by column ID), `caption`, `corner` | A table the question shows. To ask one answer per row, use a [grid](#matrix). |

Several content items are shown in order, separated by a blank line.

### Pictures

A picture's `source`, in image content or in a `media` list (options, grid
rows and columns, MaxDiff items, conjoint levels), is one of:

- `upload:<image id>`: a picture you uploaded with
  `POST /surveys/{id}/images` (it returns this source);
- a web link (`https://...`), which the model's provider fetches. Prefer an
  upload: a link that stops working leaves the question unasked;
- a `data:image/...` address, the picture written into the JSON.

PNG, JPEG, WEBP or GIF; at most 5 MB each and 50 MB for a survey. A picture
can carry `alt` text; write it, since it is what a reader without the picture
gets.

### Text and piped values

Wherever this page says *text*, a value is plain text or an object with
`parts`, each `{"text": "..."}` or `{"ref": ...}`. A reference is replaced by
its value when the question is shown:

```json
{"parts": [
  {"text": "You said you drink "},
  {"ref": {"scope": "answer", "question": "cups", "field": "number_entry"}},
  {"text": " cups a day."}
]}
```

| `scope` | Fields | The value |
|---|---|---|
| `variable` | `variable` | A variable's value (empty when never set). |
| `answer` | `question`, `field`, `item` (for fields about one option) | Part of an earlier answer. A question that pipes an answer is asked after it. |
| `loop` | `field`, or `value`: `"current_index"` / `"total"` | In a [repeated block](#repeat): the current row's value, its position, or the number of rows. |
| `random` | `kind` (`"int"` or `"float"`), `min`, `max` | A random number, drawn once per twin and place. |

Answer fields: `selected_choices`, `selected_choices_recode`,
`selected_choices_text_entry`, `unselected_choices`, `displayed_choices`,
`undisplayed_choices`, `all_choices`, `selected_count`, `question_text`, and
for one option (`item`): `choice_label`, `choice_recode`, `text_entry`,
`number_entry`.

## Responses

Every response has a `type` and may have `required`: `true`, `false` (the
default) or `"request"` (asked for, but the twin may go on without one).

### `choice`

| Field | Meaning |
|---|---|
| `mode` | Required. `"single"` or `"multiple"`. |
| `options` | Required. See [Options](#options). |
| `min_selected`, `max_selected` | (`multiple`) How many must be selected. |
| `default` | Option IDs selected before the twin answers. |
| `randomization` | How the options are ordered (see [Randomization](#randomization)). |

### Options

| Field | Meaning |
|---|---|
| `id` | Required. Answers record this ID (or `code`). |
| `label` | Required. What the model reads and answers with. Labels of one question must differ. |
| `code` | The value exported for the option (a recode). |
| `media` | Pictures shown with the option (`source`, `alt`, `title`). |
| `write_in` | `true`: a box for the twin's own words (answered as `<question>_<option>_TEXT`). |
| `show_if` | The option is shown only when this condition holds. |

The same fields describe a grid's columns, a ranking's items and an
allocation's items.

### `text`

`min_length`, `max_length`; `format`: `"number"`, `"email"`, `"phone"`,
`"date"`, `"zip"`, `"us_state"`, `"text_only"` (with `"number"`, the question
is asked as a number, with limits in `number`: `min`, `max`, `decimals`);
`default`.

### `number`

One number, typed or on a slider: `min`, `max` (0 to 100 when absent), `step`,
`decimals`, `not_applicable` (`{"label": "Not applicable"}`).

### `matrix`

A grid of statements (rows).

| Field | Meaning |
|---|---|
| `mode` | Required. `"per_row"` (one answer per row) or `"per_cell"` (one per row and column). |
| `rows` | Required. Each `id`, `label`, `media`, `show_if`, `default`; a bipolar row has `left_label`, `right_label`. |
| `row_response` | (`per_row`) What every row asks: a `choice` (the scale's options) or a `number`. |
| `columns`, `cell_response` | (`per_cell`) The columns, and a `text` or `number` response for each cell. |
| `constraints` | (`per_cell`, numbers) `[{"type": "sum_equals", "axis": "row", "total": 100}]`. |
| `randomization` | How the rows are ordered. |

### `form`, `ranking`, `allocation`, `group`

- `form`: labelled text boxes. `fields` (each `id`, `label`, `media`,
  `show_if`, `default`), `field_response` (a `text` response every box gets).
- `ranking`: a strict order, 1 the highest. `items`, `min_ranked`,
  `max_ranked`, `default` (`[{"item": ID, "rank": n}]`), `randomization`.
- `allocation`: an amount split across `items` (a constant sum). `total` (100
  when absent), `min`, `max`, `default`, `randomization`.
- `group`: grids side by side, `layout: "side_by_side"`; `fields`, each with
  `id`, `label`, `export_name` and a `response` that is a `matrix`.

### Randomization

| `order` | Fields | Meaning |
|---|---|---|
| `shuffle` | `take` | A new random order for each twin; with `take`, only that many, drawn at random. |
| `random_reverse` | `consistent` | Reversed for a random half of twins (the same half everywhere with `consistent`). |
| `positions` | `positions`, `shuffled`, `subset`, `subset_take`, `hidden` | One entry per position: an option ID pinned there, or `{"draw": "shuffled"}` / `{"draw": "subset"}` filled from those lists. |

### Presentation

`presentation.control` never changes what is asked, but the model is told some
of it:

| Response | Controls |
|---|---|
| `choice` | `list` (default), `horizontal`, `columns`, `dropdown`, `select_box` |
| `text` | `single_line`, `multi_line` (default), `essay`, `password` |
| `number`, number rows | `slider` (default), `bar`, `stars`, `number_scale` |
| `matrix` with choice rows | `bipolar` |

## Conditions

| `op` | Fields | Holds when |
|---|---|---|
| `selected`, `not_selected` | `question`, `option`, `loop` (`"all"` / `"any"` row of a repeated block) | The option was (not) selected. For a grid, a scale point any statement got. |
| `displayed`, `not_displayed` | `question` | The question was (not) shown. |
| `eq`, `ne`, `lt`, `lte`, `gt`, `gte`, `contains`, `not_contains`, `matches`, `empty`, `not_empty` | `left` (a reference: a variable, part of an answer), `right`, `ignore_case` | The comparison holds. |
| `and`, `or` | `args` (two or more) | All hold, or any. |
| `not` | `arg` | It does not hold. |
| `true`, `false` | | Always, or never. |

## Skip rules

`{"when": <selected/not_selected of an option of this question>, "to": <a later
question of the same block, "end_of_block" or "end_of_survey">}`.

## Blocks

A `"survey"` block shows its questions together:

| Field | Meaning |
|---|---|
| `id`, `kind` (`"survey"`) | Required. |
| `title` | The block's name. |
| `questions` | Required. Question IDs, in order. |
| `randomization` | `{"order": "shuffle"}`: the questions in a random order for each twin. |
| `repeat` | The block runs once per row (see below). |

### Repeat

A repeated block (Qualtrics' loop and merge) runs once per row; its questions
are recorded as `1_<question>`, `2_<question>`, ... `rows`: each `id` and
`values` (text by field name), listed in the order they run; `order`:
`"fixed"` or `"shuffle"`; `take` (with `shuffle`): run only this many rows.

## Flow

A list of steps, run in order. Every step except `show_block` has an `id`.

| `type` | Fields | What it does |
|---|---|---|
| `show_block` | `block` | Shows a block (a block shown twice is asked twice). |
| `set_variables` | `assignments` | Each `{"variable": name, "value": text}` (text may pipe values). |
| `branch` | `when`, `then`, `else` | Runs `then` when the condition holds, `else` (optional) when not. `else` is decided after `then` runs, so `then` may not set a variable the condition reads. |
| `randomize` | `take`, `children`, `evenly` | Runs `take` of its steps, drawn for each twin; `evenly` (default `true`) balances the arms across twins. |
| `group` | `children`, `title` | Runs its steps in order. |
| `end` | | Ends the survey. |

## Variables

A variable holds text: set it with `set_variables`, read it with piped text or
conditions. Declare it in `variables` (`id`, `type`: `string`, `number` or
`boolean`, `description`) to document it.

## Conjoint blocks

A conjoint block describes products by attributes and levels, with rules for
combining them; each twin's tasks are made from them. A task shows a few
alternatives and asks about them.

| Field | Meaning |
|---|---|
| `id`, `kind` (`"conjoint"`), `title` | As for any block. |
| `attributes` | Required. Each `id`, `name`, `levels`, optional `type` (`categorical`, `numeric`, `price`) with `currency` or `unit`. A level: `id`, `label`, optional `value`, `weight` (chance of being drawn; 0 never), `media`. |
| `alternatives` | Required. `{"mode": "generic", "count": 3, "label": "Option {letter}"}` (labelled by position; `{number}` gives 1, 2, ...), or `{"mode": "fixed", "items": [{"id", "label", "fixed_levels": {attribute: level}}]}` (named alternatives, such as one per brand). |
| `constraints` | `applicability` (which levels of two attributes may go together: `row_attribute`, `row_levels`, `column_attribute`, `column_levels`, and `allowed`, a 0/1 table) and `forbidden_profiles` (combinations never shown together, each a map of attribute to level). |
| `design` | Required; see below. |
| `presentation` | `instruction`; `layout`: `"table"` (default) or `"list"`; `attribute_order`, `alternative_order`: `{"mode": "fixed"}` or `{"mode": "shuffle", "scope": "participant" or "task", "fixed_positions": {...}}`. |
| `response` | What the twin answers in each task; see below. |

**Designs.**

- `"generated"`: `tasks_per_participant`, `generator: "random_valid_profiles"`,
  `duplicates`, `max_attempts`. Each alternative's levels are drawn at random
  by weight; profiles that break a rule are drawn again. Two identical options
  never share a task (`duplicates: "within_task"`, the default);
  `"within_participant"` also keeps a twin from seeing the same profile twice.
  The generator also avoids giving a twin the same set of options twice; when
  its search finds no new set, the repeat is recorded.
- `"supplied"`: `versions` (each `id` and `tasks`; a task has `id` and
  `alternatives`, each `{"id", "levels": {attribute: level}}`); each twin gets one
  version, at random or `assignment: "by_index"`.
- `"adaptive"` (0.2): `generator: "polyhedral"`, `tasks_per_participant`.
  Each task is built from the twin's earlier answers (the polyhedral method of
  Toubia, Hauser and Simester, 2004), so the survey runs one task per call, in
  stages. The response must be a `choice`; a task shows at least two options;
  at most 60 levels in all; `duplicates: "within_participant"` is not allowed.

**Responses.**

| `type` | Fields | The twin |
|---|---|---|
| `choice` | `outside_option` (`{"id": "none", "label": "None of these"}`) | Picks one alternative (or the outside option). |
| `rating` (0.2) | `scale` (two or more options, lowest first) | Rates each alternative on the scale; a task may show one alternative. |
| `ranking` (0.2) | | Ranks the alternatives, 1 the most preferred. |
| `allocation` (0.2) | `total`, `outside_option` | Splits `total` points across the alternatives. |
| `best_worst` (0.2) | `best_label`, `worst_label` | Picks the best and the worst, two different ones. |

On the website, a generated or supplied design asks all of a twin's tasks in
one call; an adaptive design asks one task per call. A task's answers are
recorded as `<block>_t<n>` (with a suffix when an answer has several parts);
the run's `conjoint-exposures.csv` lists every profile shown.
`POST /surveys/{id}/experiment-preview` shows sample tasks and how often each
level appears.

## MaxDiff blocks

| Field | Meaning |
|---|---|
| `id`, `kind` (`"maxdiff"`), `title` | As for any block. |
| `items` | Required, two or more: each `id`, `label`, optional `code`, `media`. |
| `design` | Required: `"generated"` with `tasks_per_participant`, `items_per_task` and `generator` (`"uniform_subsets"` or `"balanced_subsets"`, which shows each item about equally often within a twin); or `"supplied"` with `versions` (each task a list of item IDs). A twin sees the same subset again only when every other has been shown. |
| `presentation` | `instruction`; `item_order`: `{"mode": "fixed"}` or `{"mode": "shuffle", "scope": "task" or "participant"}`. |
| `response` | Required: `{"type": "best_worst", "required": true, "best_label": "Most important", "worst_label": "Least important"}`. |
| `anchor` | A follow-up after each task: `id`, `content`, and a single-choice `response`. |

Each task is recorded as `<block>_t<n>_best` and `<block>_t<n>_worst` (and
`<block>_t<n>_anchor`). A twin that names one item as both best and worst has
both answers recorded as skipped. `maxdiff-exposures.csv` lists every subset shown.

## Writing well for twins

- Write every question as a respondent should read it; twins read instructions,
  labels and descriptive text word for word.
- Give each option a distinct label, and give numbers and units in the label
  ("$4.50", "3 to 5 hours a week").
- Use `required: true` for questions that must be answered, and `show_if` or
  skip rules for logic instead of instructions like "skip if...".
- Keep IDs short and stable (`q1`, `drinks_coffee`); they name the columns.
