# The standard survey format: codebook

A survey in the standard format is one JSON document that says everything a run
on digital twins needs: what each question shows, what it asks for, and how the
survey moves from question to question. ExploraTwin's survey builder writes this
format, the surveytwin library reads it, and the prompts a twin reads are made
from these fields. This codebook explains every part, how a twin reads it, and
the mistakes to avoid. [fields.md](fields.md) lists every object and field
exhaustively (generated from the code); the
[JSON Schema](survey-format.schema.json) checks structure only.

Contents: [the whole survey](#a-small-survey) · [checks](#how-a-survey-is-checked) ·
[the survey object](#the-survey) · [questions](#questions) ·
[content](#content) · [pictures](#pictures) · [piped text](#text-and-piped-values) ·
[choosing a response](#choosing-a-response) · [responses](#responses) ·
[options](#options) · [randomization](#randomization) · [presentation](#presentation) ·
[conditions](#conditions) · [skip rules](#skip-rules) · [blocks](#blocks) ·
[repeated blocks](#repeat) · [flow](#flow) · [variables](#variables) ·
[conjoint](#conjoint-blocks) · [MaxDiff](#maxdiff-blocks) ·
[how answers are recorded](#how-answers-are-recorded) ·
[writing for twins](#writing-for-twins) · [common mistakes](#common-mistakes) ·
[versions](#versions) · [limitations](#what-the-format-cannot-say)

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

Three lists make a survey: `questions` (every question and instruction, each
once), `blocks` (questions grouped and shown together, in order) and `flow` (the
order the blocks are shown in, with conditions, random arms and variables). A
question in no block, or a block the flow never shows, is never asked.

## How a survey is checked

Nothing runs until both passes succeed:

1. **Shape.** Every object has only the fields this codebook lists, of the right
   type; required fields are present; a field that is not listed is an error (a
   key starting with `x-` is allowed anywhere, kept and never used, for your own
   notes); `schema_version` is `"0.1"` or `"0.2"`.
2. **Meaning.** IDs are unique where answers are recorded by them; every
   reference resolves (a block in the flow, a question in a block, an option in
   a condition, a skip target later in the same block, a piped answer); limits
   are possible (a minimum not above its maximum); the rows of a repeated block
   are listed in the order they run; no question offers two options with the
   same label (the model answers with the label); pictures can be loaded; a
   conjoint or MaxDiff design can be drawn.

Each problem names its field as a path (`$.questions[2].response.options[1].label`),
a `code`, and a `level`: an `error` stops the survey, a `warning` does not.
[checking.md](checking.md) says how to run the checks and what each code means.

## The survey

| Field | Meaning |
|---|---|
| `schema_version` | Required. `"0.1"`, or `"0.2"` when the survey uses a feature of version 0.2 (see [Versions](#versions)). |
| `id` | Required. The survey's ID, no spaces (`"coffee_habits"`). |
| `title` | The survey's name. ExploraTwin names the project after it. |
| `description`, `language` | For people. `language` is a code such as `"EN"`. |
| `variables` | The variables the survey uses, declared (see [Variables](#variables)). |
| `questions` | Every question and instruction, each once. |
| `blocks` | Groups of questions, and conjoint or MaxDiff blocks. |
| `flow` | The order blocks are shown in. |
| `source` | Only in a survey read from a Qualtrics file: what that file said with no field here. Never write it yourself. |

## Questions

A question shows content and, unless it is an instruction, asks for a response.

| Field | Meaning |
|---|---|
| `id` | Required. Answers are recorded by it. Short, stable, no spaces: `q1`, `drinks_coffee`. Never change it once answers exist. |
| `export_name` | The column name in exports (Qualtrics' "data export tag"), such as `Q7` or `age_group`. The `id` when absent. |
| `content` | Required. What the question shows, in order: text, pictures, tables. |
| `response` | What the question asks for (see [Responses](#responses)). Leave it out for an **instruction**: text shown and never answered (an introduction, a consent text, a description before a set of questions). |
| `show_if` | A [condition](#conditions): the question is shown only when it holds. |
| `skip` | [Skip rules](#skip-rules): after the answer, jump ahead when a rule holds. |
| `presentation` | How the response is shown (see [Presentation](#presentation)). |
| `kind` | Only in a survey read from a Qualtrics file: `"timing"`, `"meta_info"`, `"captcha"` or `"unsupported"`, a question that is never asked. Never write it yourself. |

## Content

`content` is a list; its items are shown in order, separated by a blank line.

| `type` | Fields | Shown as |
|---|---|---|
| `text` | `text` (required; [text](#text-and-piped-values)); `format`: `"plain"` (the default) or `"html"` | The text. Plain text keeps its line breaks. HTML loses its markup and keeps its lines as a browser shows them (see below). |
| `image` | `source` (required), `alt`, `title` | The picture, sent to the model where it stands among the content (see [Pictures](#pictures)). |
| `table` | `columns` (each `id`, `label`); `rows` (each `id`, `label`, `cells` by column ID); `caption`; `corner` (the label above the row labels) | A table the question shows, every cell under its column, as Markdown. A table is something to read; to ask one answer per row, use a [grid](#matrix). |

How HTML text reads: a `<br>` ends a line; a `<div>` or a list item is a line of
its own; a paragraph, a list or a table is set apart by a blank line; line breaks
in the HTML source are spaces. A list item starts with "- " (or its number in a
numbered list). A table with a heading row and one row of values reads as
"Heading: value. Heading: value"; a wider table with labels down its first
column gives one "Label: values" line per row; any other table becomes a
Markdown table. Prefer plain text when you write a survey; HTML is for text
copied from a source that has markup.

A question whose content is only pictures reads "[Image stimulus attached]"
where its text would be: give it a line of text too.

## Pictures

A picture's `source`, in image content or in any `media` list (options, grid rows
and columns, ranking and allocation items, MaxDiff items, conjoint levels), is
one of:

- a web link (`https://...`). The model's provider fetches it, so it must be
  public and stay up: a link that stops working leaves its question unasked;
- a `data:image/png;base64,...` address: the picture itself inside the JSON;
- **on ExploraTwin**: `upload:<image id>`, a picture uploaded to the survey's
  project (an agent connected to ExploraTwin uploads with
  `POST /surveys/{id}/images`, which returns this source; in the builder, add
  the picture with the picture button);
- **with the surveytwin library on your computer**: the path of a picture file,
  absolute or relative to the folder of the survey's JSON file
  (`"source": "images/ad_high_price.png"`). ExploraTwin cannot open your files:
  change such sources to uploads or links before you send the survey there.

PNG, JPEG, WEBP or GIF. ExploraTwin takes at most 5 MB a picture and 50 MB a
survey; the library, 20 MB a picture. A damaged or cut-off file is an error.

Always write `alt`. The prompt names each picture where it stands and adds its
`alt` and `title` as text ("Image attached to Q4. Alt text: A bottle of the new
drink. Title: Ad A"), so they tell the model what the picture is, and they are
what a reader without the picture gets. In a conjoint level, `media` shows the
picture in the task's cell.

## Text and piped values

Wherever this codebook says *text*, a value is plain text or an object with
`parts`, each `{"text": "..."}` or `{"ref": ...}`. A reference is replaced by its
value when the question is shown:

```json
{"parts": [
  {"text": "You said you drink "},
  {"ref": {"scope": "answer", "question": "cups", "field": "number_entry"}},
  {"text": " cups a day."}
]}
```

| `scope` | Fields | The value |
|---|---|---|
| `variable` | `variable` | A [variable](#variables)'s value (empty when never set). |
| `answer` | `question`, `field`, `item` (for fields about one option) | Part of an earlier answer (see below). A question that pipes an answer is asked after that answer exists, so the survey runs in stages. |
| `loop` | `field`, or `value`: `"current_index"` / `"total"` | In a [repeated block](#repeat): a value of the current row, its position (1, 2, ...), or the number of rows. |
| `random` | `kind` (`"int"` or `"float"`), `min`, `max` | A random number, drawn once per twin and place (the same in every run with the same seed). |
| `source` | `expression` | Only in a survey read from a Qualtrics file: a piped expression with no form here, kept as written. |

Answer fields: `selected_choices`, `selected_choices_recode`,
`selected_choices_text_entry`, `unselected_choices`, `displayed_choices`,
`undisplayed_choices`, `all_choices`, `selected_count`, `question_text`; and
for one option (`item`): `choice_label`, `choice_recode`, `text_entry` (the text
typed for that option) and `number_entry` (the number typed for that option, or
the number given for a numeric question).

A Qualtrics piped field such as `${q://QID3/ChoiceGroup/SelectedChoices}` is
written as `{"ref": {"scope": "answer", "question": "q3", "field": "selected_choices"}}`;
`${e://Field/condition}` as `{"ref": {"scope": "variable", "variable": "condition"}}`;
`${lm://Field/1}` as `{"ref": {"scope": "loop", "field": "1"}}`.

## Choosing a response

| You want to ask | Write |
|---|---|
| One answer from a list | `choice`, `mode: "single"` |
| Any number of answers from a list ("select all that apply") | `choice`, `mode: "multiple"` (with `min_selected` / `max_selected` when the source says "up to 3") |
| An option with its own box ("Other, please specify") | the option with `write_in: true` |
| A number typed in a box | `text` with `format: "number"` and its limits in `number` (`min`, `max`, `decimals`) |
| A number on a slider, a bar or stars | `number` (with `min`, `max`, `step`; `presentation.control` for bar or stars) |
| A short or long text answer | `text` (`presentation.control`: `single_line` for a short box, `multi_line` or `essay` for a long one) |
| An e-mail address, a date, a ZIP code | `text` with `format` |
| The same scale for several statements (agree/disagree, a Likert grid) | `matrix`, `mode: "per_row"`, `row_response` a `choice` |
| A slider for each of several statements | `matrix`, `mode: "per_row"`, `row_response` a `number` |
| A number or text in each cell of a table | `matrix`, `mode: "per_cell"` with `columns` and `cell_response` |
| Each row's cells adding up to a total | `matrix`, `per_cell`, number cells, `constraints: [{"type": "sum_equals", "axis": "row", "total": 100}]` |
| Several labelled text boxes (name, city, ...) | `form` |
| An order of items, 1 the highest | `ranking` |
| An amount split across items (a constant sum) | `allocation` |
| Grids side by side sharing their statements | `group` |
| Text, an introduction or a picture with nothing to answer | an instruction: no `response` |
| Profiles of products to choose from | a [conjoint block](#conjoint-blocks) |
| The best and the worst of a few items at a time | a [MaxDiff block](#maxdiff-blocks) |

## Responses

Every response has a `type` and may have `required`: `true` (an answer is
needed), `false` (the default) or `"request"` (an answer is asked for, but the
twin may go on without one).

### `choice`

| Field | Meaning |
|---|---|
| `mode` | Required. `"single"` or `"multiple"`. |
| `options` | Required. See [Options](#options). |
| `min_selected`, `max_selected` | (`multiple`) How many options must be selected, at least and at most. |
| `default` | Option IDs selected before the twin answers (one for `single`). |
| `randomization` | How the options are ordered for each twin (see [Randomization](#randomization)). |

### Options

| Field | Meaning |
|---|---|
| `id` | Required. Answers record this ID (or `code`), never the position. |
| `label` | Required. What the model reads and answers with. Labels of one question must differ (ignoring case). An option with a picture and no label reads "[Image option 2]". |
| `code` | The value exported for the option (a recode, such as `1` to `5` for a scale). The ID when absent. |
| `media` | Pictures shown with the option (`source`, `alt`, `title`). |
| `write_in` | `true`: the option has a box for the twin's own words, answered as its own question (`<question>_<option>_TEXT`). |
| `show_if` | The option is shown only when this condition holds. |

The same fields describe a grid's columns, a ranking's items and an
allocation's items. Keep the source's option order: twins read the options in
order, and an ordered scale (frequencies, price ranges, agreement) must stay in
its order unless the source randomizes it.

### `text`

| Field | Meaning |
|---|---|
| `min_length`, `max_length` | The number of characters, at least and at most. |
| `format` | `"number"`, `"email"`, `"phone"`, `"date"`, `"zip"`, `"us_state"`, `"text_only"`: what the text must be. With `"number"`, the question is asked as a number, with limits in `number`. |
| `number` | (`format: "number"`) `min`, `max`, `decimals`. |
| `default` | Text already in the box. |

A text question written in this format is asked as text whatever its wording.

### `number`

One number on a slider, a bar, stars or a number scale (the twin is told which). For a number typed in a box, use `text` with `format: "number"`.

| Field | Meaning |
|---|---|
| `min`, `max` | The range (0 to 100 when absent). Always give the range the source gives. |
| `step` | The answer moves in steps of this size (`1` for whole numbers). |
| `decimals` | Decimal places. |
| `not_applicable` | `{"label": "Not applicable"}`: an answer that is not a number. |

### `matrix`

A grid of statements (rows).

| Field | Meaning |
|---|---|
| `mode` | Required. `"per_row"` (one answer per row) or `"per_cell"` (one per row and column). |
| `rows` | Required. Each `id`, `label`, `media`, `show_if`, and `default` (a number for a number row, option IDs for a choice row). A bipolar row also has `left_label` and `right_label`. |
| `row_response` | (`per_row`) What every row asks: a `choice` of the scale's options, or a `number` (a slider for each statement). |
| `columns` | (`per_cell`) The columns, written as options. |
| `cell_response` | (`per_cell`) A `text` or a `number` response for each cell. |
| `constraints` | (`per_cell`, number cells) `[{"type": "sum_equals", "axis": "row", "total": 100}]`: each row's cells add up to the total. |
| `randomization` | How the rows are ordered. |

The twin answers each row as a question of its own: `<question>_<row>`
(`<question>_<row>_<column>` for each cell).

### `form`

Several labelled text boxes.

| Field | Meaning |
|---|---|
| `fields` | Required. Each `id`, `label`, `media`, `show_if`, `default` (text already in the box). |
| `field_response` | A `text` response every box gets (length, format). |

### `ranking`

A strict order of items, 1 the highest.

| Field | Meaning |
|---|---|
| `items` | Required. Written as options. |
| `min_ranked`, `max_ranked` | How many items must be ranked. |
| `default` | `[{"item": ID, "rank": n}]`: ranks set before the twin answers. |
| `randomization` | How the items are ordered. |

### `allocation`

An amount split across items (Qualtrics' constant sum).

| Field | Meaning |
|---|---|
| `items` | Required. Written as options. |
| `total` | What the amounts add up to; 100 when absent. |
| `min`, `max` | The least and the most one item may get. |
| `default` | Preset amounts by item ID. |
| `randomization` | How the items are ordered. |

### `group`

Several grids side by side (Qualtrics' side by side), with `layout:
"side_by_side"`. `fields` lists the columns: each has an `id`, a `label`, an
`export_name`, `media`, and a `response`, which is a [`matrix`](#matrix) with its
own statements and answer (a choice scale, or text). `randomization` orders each
column's options.

## Randomization

On a `choice`, a `matrix` (its rows), a `ranking` or an `allocation`:

| `order` | Fields | Meaning |
|---|---|---|
| `shuffle` | `take` | A new random order for each twin; with `take`, only that many, drawn at random. |
| `random_reverse` | `consistent` | Reversed for a random half of twins; with `consistent`, the same half for every such question. |
| `positions` | `positions`, `shuffled`, `subset`, `subset_take`, `hidden` | One entry per position shown: an option ID pinned there, or `{"draw": "shuffled"}` / `{"draw": "subset"}` filled from those lists. `hidden` options are never shown. Use it for "randomize, but keep 'None of these' last". |

A block's questions in a random order: the block's `randomization: {"order": "shuffle"}`.
Draws come from the run's seed, the twin and the place, so the same seed gives
the same orders.

## Presentation

`presentation.control` says how a response is shown. It never changes what is
asked, but the model is told some of it:

| Response | Controls |
|---|---|
| `choice` | `list` (default), `horizontal`, `columns`, `dropdown`, `select_box` |
| `text` | `single_line`, `multi_line` (default), `essay`, `password` |
| `number`, number rows | `slider` (default), `bar`, `stars`, `number_scale` |
| `matrix` with choice rows | `bipolar` (each row runs from its `left_label` to its `right_label`) |

## Conditions

A condition is a tree, used by `show_if` (questions and options), `skip` rules
and flow branches:

| `op` | Fields | Holds when |
|---|---|---|
| `selected`, `not_selected` | `question`, `option`, `loop` (`"all"` or `"any"` row of a repeated block) | The option was (not) selected. For a grid, `option` names a scale point, selected when any statement got it. |
| `displayed`, `not_displayed` | `question` | The question was (not) shown. |
| `eq`, `ne`, `lt`, `lte`, `gt`, `gte`, `contains`, `not_contains`, `matches`, `empty`, `not_empty` | `left` (a [reference](#text-and-piped-values): a variable, part of an answer), `right` (a value or text), `ignore_case` | The comparison holds. |
| `and`, `or` | `args` (two or more conditions) | All of them hold, or any. |
| `not` | `arg` | The condition does not hold. |
| `true`, `false` | | Always, or never. |

Any condition can carry a `description` for people. Examples:

```json
{"op": "selected", "question": "owns_car", "option": "yes"}
{"op": "gte", "left": {"scope": "answer", "question": "age", "field": "number_entry"}, "right": 18}
{"op": "and", "args": [
  {"op": "selected", "question": "owns_car", "option": "yes"},
  {"op": "eq", "left": {"scope": "variable", "variable": "arm"}, "right": "treatment"}
]}
```

A condition on an answer makes the survey run in stages (the answer must exist
first). A condition on a variable set before the question does not.

## Skip rules

On the question whose answer decides: `"skip": [{"when": <condition>, "to": <target>}]`.

| Field | Meaning |
|---|---|
| `when` | Required: `selected` or `not_selected` of an option of this same question. |
| `to` | Required: a later question of the same block, `"end_of_block"` or `"end_of_survey"`. |
| `id`, `description` | For people. |

A source's "If No, skip to Q10" is a skip rule when Q10 is later in the same
block. When the jump leaves the block, or depends on another question or a
number, write `show_if` on the questions skipped instead (each shown only when
the jump would not happen), or a flow [branch](#branch).

## Blocks

A block of `kind: "survey"` shows its questions together, in order. A block of
`kind: "conjoint"` or `"maxdiff"` is an experiment whose tasks are drawn for each
twin.

| Field | Meaning |
|---|---|
| `id`, `kind` | Required. |
| `title` | The block's name. |
| `questions` | Required. The block's question IDs, in order. A question belongs to one block. |
| `randomization` | `{"order": "shuffle"}`: the block's questions in a random order for each twin. |
| `repeat` | The block runs once per row (see [Repeat](#repeat)). |

Use one block per page or section of the source; it keeps the source's grouping
and lets the flow branch or randomize between sections.

### Repeat

A repeated block (Qualtrics' loop and merge) runs once per row; its questions are
asked once per row, as `1_<question>`, `2_<question>`, ..., and `loop` references
read the row's values.

| Field | Meaning |
|---|---|
| `rows` | Required. Each row has an `id` and `values` (text by field name). List them in the order they run: numbers in numeric order, otherwise alphabetically. |
| `order` | `"fixed"` or `"shuffle"` (a random order of rows for each twin). |
| `take` | (`shuffle`) Run only this many rows, drawn at random. |

## Flow

The flow is a list of steps, run in order. Every step except `show_block` has an
`id`.

| `type` | Fields | What it does |
|---|---|---|
| `show_block` | `block` | Shows a block. A block shown twice is asked twice. |
| `set_variables` | `assignments` | Sets [variables](#variables): each `{"variable": name, "value": text}` (the text may pipe values), or `{"variable": name, "from_outside": true}` for a value given with the run. |
| `branch` | `when`, `then`, `else` | Runs `then` (a list of steps) when the condition holds, and `else` (optional) when it does not. |
| `randomize` | `take`, `children`, `evenly` | Runs `take` of its `children` (steps), drawn at random for each twin. `evenly` (`true` when absent) balances how often each child is drawn across twins. |
| `group` | `children`, `title` | Runs its steps in order (a way to randomize several steps as one arm). |
| `end` | | Ends the survey (a screen-out). |

A two-arm experiment:

```json
"flow": [
  {"type": "show_block", "block": "intro"},
  {"type": "randomize", "id": "arms", "take": 1, "children": [
    {"type": "group", "id": "arm_a", "children": [
      {"type": "set_variables", "id": "set_a", "assignments": [{"variable": "arm", "value": "low_price"}]},
      {"type": "show_block", "block": "ad_low_price"}]},
    {"type": "group", "id": "arm_b", "children": [
      {"type": "set_variables", "id": "set_b", "assignments": [{"variable": "arm", "value": "high_price"}]},
      {"type": "show_block", "block": "ad_high_price"}]}
  ]},
  {"type": "show_block", "block": "outcomes"}
]
```

A screen-out: `{"type": "branch", "id": "screen", "when": {"op": "selected", "question": "age_group", "option": "under_18"}, "then": [{"type": "end", "id": "end_minor"}]}`.

### Branch

`else` is decided after `then` has run, as a second branch on the opposite
condition. So a branch with `else` cannot set, in its `then` steps, a variable
its condition reads; write two branches instead.

## Variables

A variable holds text (Qualtrics' embedded data): set it with `set_variables`,
read it with piped text or conditions. Declaring it in `variables` (`id`,
`type`: `"string"`, `"number"` or `"boolean"`, `description`, `from_outside`) is
optional but documents it; a variable that is read but never set or declared is
reported. A value from outside the survey (a panel's field, a link parameter)
is `from_outside: true`.

## Conjoint blocks

A conjoint block describes products by their attributes and levels, with rules
for combining them; each twin's tasks are made from them. A task shows a few
alternatives and asks about them.

| Field | Meaning |
|---|---|
| `id`, `kind` (`"conjoint"`), `title` | As for any block. |
| `attributes` | Required. Each `id`, `name`, `levels`, an optional `type` (`"categorical"`, `"numeric"` or `"price"`) with `currency` or `unit`. A level: `id`, `label` (what the twin reads, such as "$4.50"), optional `value` (a number kept with the data), `weight` (its chance of being drawn; 0 never), `media` (pictures shown in the task's cell). |
| `alternatives` | Required. `{"mode": "generic", "count": 3, "label": "Option {letter}"}`: alternatives labelled by position ("Option A", "Option B"; `{number}` gives 1, 2). Or `{"mode": "fixed", "items": [{"id", "label", "fixed_levels": {attribute: level}}]}`: named alternatives, such as one per brand. |
| `constraints` | `applicability`: tables of which levels of two attributes may go together (`row_attribute`, `row_levels`, `column_attribute`, `column_levels`, and `allowed`, a 0/1 table with one row per row level, every level of both attributes listed once). `forbidden_profiles`: combinations never shown in one profile, each a map of attribute ID to level ID. |
| `design` | Required. How tasks are made (below). |
| `presentation` | `instruction` (the question above the alternatives); `layout`: `"table"` (default: a column per alternative and a row per attribute) or `"list"`; `attribute_order` and `alternative_order`: `{"mode": "fixed"}` or `{"mode": "shuffle", "scope": "participant" or "task", "fixed_positions": {"brand": 1}}`. |
| `response` | What the twin answers in each task (below). |
| `administration` | `{"unit": "block"}` (default: all tasks in one model call) or `{"unit": "task"}` (one call per task). ExploraTwin always asks a generated or supplied design's tasks in one call; an adaptive design is always one per call. |

**Designs.**

- `"generated"`: `tasks_per_participant`, `generator: "random_valid_profiles"`,
  `duplicates`, `max_attempts`. Each alternative's levels are drawn at random by
  weight; a profile that breaks a rule is drawn again. Two identical options
  never share a task; `duplicates: "within_participant"` also keeps a twin from
  seeing the same profile twice. A twin gets the same set of options twice only
  when no new set can be drawn, and the repeat is recorded.
- `"supplied"`: `versions`, each with `id` and `tasks` (a task has `id` and
  `alternatives`, each `{"id", "levels": {attribute: level}}`; for fixed
  alternatives, the alternative's ID). Each twin gets one version, at random or
  `assignment: "by_index"` (by its position in the sample). Use it for a design
  made in another tool (a design file).
- `"adaptive"` (0.2): `generator: "polyhedral"`, `tasks_per_participant`. Each
  task is built from the twin's answers to the tasks before it (the polyhedral
  method of Toubia, Hauser and Simester, 2004), so the survey runs one task per
  call, in stages. The response must be a `choice`; a task shows at least two
  options; at most 60 levels in all; `duplicates: "within_participant"` is not
  allowed. Each twin's estimated level values come with the results.

**Responses.**

| `type` | Fields | The twin | Answers recorded |
|---|---|---|---|
| `choice` | `outside_option` (`{"id": "none", "label": "None of these", "code": 0}`), `required` | Picks one alternative, or the outside option. | `<block>_t<n>` |
| `rating` (0.2) | `scale` (two or more options, lowest first), `required` | Rates each alternative on the scale; a task may show one alternative. | `<block>_t<n>_<option>` for each alternative (`<block>_t<n>` with one) |
| `ranking` (0.2) | `required` | Ranks the alternatives, 1 the most preferred. | `<block>_t<n>_<option>` for each |
| `allocation` (0.2) | `total`, `outside_option`, `required` | Splits `total` points across the alternatives. | `<block>_t<n>_<option>` for each |
| `best_worst` (0.2) | `best_label`, `worst_label`, `required` | Picks the best and the worst, two different ones. | `<block>_t<n>_best`, `<block>_t<n>_worst` |

**What the twin reads.** Each task is the instruction, then the alternatives as
a table (a column per alternative in the order shown, a row per attribute in the
order drawn) or as a list, then the question in the block's answer format:

```
Which of these coffee shops would you go to?

| | Dunkin | McDonald's | Starbucks |
|---|---|---|---|
| Brand | Dunkin | McDonald's | Starbucks |
| Seating | Seating only | Seating and Wi-Fi | Seating only |
| Price of a medium latte | $4.50 | $4.50 | $4.50 |
```

A twin's tasks depend only on the run's seed, the twin and the block (and, for an
adaptive design, its answers): the same twin gets the same tasks in every mode,
preview and rerun. The run's exposure file lists every profile shown.

## MaxDiff blocks

A MaxDiff block shows a few items at a time from a pool and asks for the best and
the worst of them.

| Field | Meaning |
|---|---|
| `id`, `kind` (`"maxdiff"`), `title` | As for any block. |
| `items` | Required, two or more: each `id`, `label`, optional `code` and `media`. |
| `design` | Required: `"generated"` with `tasks_per_participant`, `items_per_task` and `generator` (`"uniform_subsets"`, or `"balanced_subsets"`: each item shown about equally often within a twin); or `"supplied"` with `versions` (each task a list of item IDs). A twin sees the same subset again only when every other has been shown. |
| `presentation` | `instruction`; `item_order`: `{"mode": "fixed"}` (the pool's order), `{"mode": "shuffle", "scope": "task"}` or `{"mode": "shuffle", "scope": "participant"}`, with `fixed_positions`. |
| `response` | Required: `{"type": "best_worst", "required": true, "best_label": "Most important", "worst_label": "Least important"}`. |
| `anchor` | A follow-up after each task about the same items: `id`, `content` and a single-choice `response` (such as "All / Some / None of these are important to me"). |
| `administration` | As for conjoint. |

Each task is recorded as `<block>_t<n>_best` and `<block>_t<n>_worst` (and
`<block>_t<n>_anchor`). A twin that names one item as both best and worst has
both answers recorded as skipped.

A Qualtrics "MaxDiff" made from a matrix question is a `matrix` (each item rated
on the best and worst columns); write a MaxDiff block when the study needs one
best and one different worst per task.

## How answers are recorded

Answers are recorded, and exports have their columns, by question `id` (or
`export_name` in the Qualtrics-style export):

- a single `choice`, a `text` or a `number` question: `<question>`; a multiple
  choice: a column per option, `<question>_<option>`; a write-in box:
  `<question>_<option>_TEXT`;
- a grid row: `<question>_<row>`; a grid cell: `<question>_<row>_<column>`;
- a ranked or allocated item, a form field: `<question>_<item>`;
- a question of a repeated block: `<row number>_<question>`;
- conjoint and MaxDiff tasks: as listed in their sections.

A choice is recorded by the option's `id` (or `code` in the values file), never
its position, so options can be reordered or randomized safely.

## Writing for twins

- Write every question as a respondent should read it. Twins read every word:
  instructions, labels, descriptive text.
- Give each option a distinct label, with numbers and units in the label
  ("$4.50", "3 to 5 hours a week").
- Use `required: true` for questions that must be answered, and `show_if` or skip
  rules for logic, not instructions like "skip if...".
- Keep ordered scales in order, and say a range for every number.
- On ExploraTwin each built-in twin is a real person's profile with known demographics
  (age, gender, race or ethnicity, education, income, employment, religion,
  marital status, household size, region or state, political identity). A
  survey written for twins need not ask them again; a converted survey keeps
  the source's questions, and the person may drop them later.
- Keep IDs short and stable; they name the columns.

## Common mistakes

| Mistake | What to do |
|---|---|
| A question in no block, or a block not in the flow | Every question in exactly one block; every block shown by the flow. |
| Logic written as text ("If you answered No, skip to Q10") | A skip rule or `show_if`; remove the sentence unless respondents saw it. |
| Two options with the same label | Make the labels differ: the model answers with the label. |
| `"mode": "single"` for "select all that apply" | `"mode": "multiple"`. |
| A Likert grid written as separate questions | One `matrix`, `per_row`, with the scale as `row_response` options. |
| A scale reordered or randomized that the source keeps in order | Keep the source's order; randomize only what the source randomizes. |
| A `number` without `min` / `max` | Give the source's range; otherwise 0 to 100 is assumed. |
| A skip to a question in another block | `show_if` on the skipped questions, or a flow branch. |
| A condition on a question asked later | Conditions read earlier answers only; move the question or the condition. |
| `schema_version: "0.1"` with a rating conjoint or an adaptive design | `"0.2"`. |
| Writing `source` or `kind` | Never: they are for surveys read from a Qualtrics file. |
| A picture only, no text | Add a line of text; write `alt`. |
| A local file path sent to ExploraTwin | Upload the picture (`upload:`), or use a public link. |

## Versions

Version `0.2` adds, for conjoint blocks only, the answer formats `rating`,
`ranking`, `allocation` and `best_worst`, and adaptive designs. A survey that
uses one says `"schema_version": "0.2"`; any other survey says `"0.1"`. A survey
that uses a 0.2 feature but says `"0.1"` is refused with the code `version`
(ExploraTwin's builder sets the version itself when it saves). Version 0.2 is
read by ExploraTwin and by surveytwin 0.3 and later; earlier versions of the
library read 0.1 only.

## What the format cannot say

- Question JavaScript, quotas, authenticators, timers that decide something,
  and display settings that never reach a respondent.
- Conjoint prices calculated from the levels shown (conditional pricing), a
  purchase question about the chosen profile (dual choice), partial profiles and
  holdout tasks.
- Loops over a question's answers (only a repeated block's own rows).

When a source has one of these, leave it out and say so in your conversion
report ([converting.md](converting.md)); keep everything else as written.

## A survey read from a Qualtrics file

ExploraTwin and the library read Qualtrics (QSF) files directly, so a Qualtrics
survey needs no conversion: upload the QSF, or run `surveytwin convert
study.qsf -o study.json` to get it in this format. For reference, how Qualtrics
maps to this format:

| Qualtrics | Standard form |
|---|---|
| Multiple choice (all single and multiple selectors) | `choice` |
| Text entry (single line, multi line, essay, password) | `text` |
| Text entry form | `form` |
| Matrix (Likert, bipolar, profile) | `matrix`, `per_row` with a `choice` |
| Matrix with the MaxDiff selector | `matrix`, `per_row` with a `choice` (a MaxDiff block enforces one best and a different worst) |
| Matrix text entry | `matrix`, `per_cell` with `text` cells |
| Matrix constant sum | `matrix`, `per_cell` with `number` cells and a sum |
| Slider, bar, stars, number scale | `matrix`, `per_row` with a `number` (one statement: a lone number) |
| Constant sum | `allocation` |
| Rank order | `ranking` |
| Side by side | `group` |
| Descriptive text, graphic | an instruction (no response) |
| Display logic, skip logic, option display logic | `show_if`, `skip`, the option's `show_if` |
| Embedded data, branch, randomizer, group, end of survey | `set_variables`, `branch`, `randomize`, `group`, `end` |
| Loop and merge (static) | a block's `repeat` |
| Question randomization in a block | a block's `randomization` |
