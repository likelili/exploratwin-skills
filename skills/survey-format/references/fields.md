# Field reference

Every object of the standard survey format and every field it takes, written
from the JSON Schema the checks use ([survey-format.schema.json](survey-format.schema.json)).
What the fields mean together, and how a twin reads them, is in the
[codebook](codebook.md). This page is generated; do not edit it by hand.

A field marked *required* must be present. A key starting with `x-` is allowed
in any object, kept and never used. "Text" means plain text, or text with piped
values (`{"parts": [...]}`, see `Template`).


## `Survey`

A survey in the standard format.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `schema_version` | one of `"0.1"`, `"0.2"` | required |  |
| `id` | text (no spaces, not empty) | required |  |
| `title` | text |  |  |
| `description` | text |  |  |
| `language` | text |  |  |
| `variables` | list of `Variable` |  |  |
| `questions` | list of `Question` |  |  |
| `blocks` | list of `Block` |  |  |
| `flow` | list of `FlowNode` |  |  |
| `source` | any value |  | Where an imported survey came from, and what the reader found. |

## `Variable`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (not empty) | required |  |
| `type` | one of `"string"`, `"number"`, `"boolean"` |  |  |
| `description` | text |  |  |
| `from_outside` | true or false |  | The value comes from outside the survey (a panel, a link). |

## `Question`

A question, or an instruction when it has no response.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `export_name` | text |  | The column name in exports (Qualtrics DataExportTag). |
| `content` | list of `Content` | required |  |
| `response` | `Response` |  | Absent for an instruction, which is shown and never answered. |
| `kind` | one of `"timing"`, `"meta_info"`, `"captcha"`, `"unsupported"` |  | A source question that is never simulated. |
| `show_if` | `Condition` |  |  |
| `skip` | list of `SkipRule` |  |  |
| `presentation` | `Presentation` |  |  |
| `source` | `QuestionSource` |  |  |

## `Content`

Something a question shows.

One of: `TextContent`, `ImageContent`, `TableContent`.

## `TextContent`

Text, kept with its line and paragraph breaks.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"text"` | required |  |
| `text` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `format` | one of `"plain"`, `"html"` |  | 'plain' (the default) or 'html' as exported by a platform. |

## `Template`

Text with piped values, in order.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `parts` | list of `TextPart` or `RefPart` (at least 1) | required |  |

## `TextPart`

Literal text.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `text` | text | required |  |

## `RefPart`

A piped value.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `ref` | `Ref` | required |  |

## `Ref`

A piped value.

One of: `VariableRef`, `AnswerRef`, `LoopRef`, `RandomRef`, `SourceRef`.

## `VariableRef`

A survey variable (Qualtrics embedded data): ${e://Field/name}.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `scope` | `"variable"` | required |  |
| `variable` | text (not empty) | required | The variable's name. |

## `AnswerRef`

Part of an earlier answer: ${q://QID/...}.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `scope` | `"answer"` | required |  |
| `question` | text (no spaces, not empty) | required | The question answered earlier. |
| `field` | one of `"selected_choices"`, `"selected_choices_recode"`, `"selected_choices_text_entry"`, `"unselected_choices"`, `"displayed_choices"`, `"undisplayed_choices"`, `"all_choices"`, `"selected_count"`, `"choice_label"`, `"choice_recode"`, `"text_entry"`, `"number_entry"`, `"question_text"` | required | Which part of the answer. |
| `item` | text (not empty) |  | An option, statement or field ID, for fields that name one. |

## `LoopRef`

A value of the current repeat row: ${lm://Field/1}, ${lm://CurrentLoopNumber}, ${lm://TotalLoops}.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `scope` | `"loop"` | required |  |
| `field` | text (not empty) |  | A column of the repeat table. |
| `value` | one of `"current_index"`, `"total"` |  | The repeat's position or length. |

## `RandomRef`

A random number drawn once per participant and place: ${rand://int/1:10}.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `scope` | `"random"` | required |  |
| `kind` | one of `"int"`, `"float"` | required |  |
| `min` | number | required |  |
| `max` | number | required |  |

## `SourceRef`

A piped expression of the source platform with no standard form, kept as written.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `scope` | `"source"` | required |  |
| `expression` | text (not empty) | required |  |

## `ImageContent`

An image shown with the question.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"image"` | required |  |
| `source` | text (not empty) | required | A web link, a data: address, or the path of a PNG, JPEG, WEBP or GIF file: absolute, or relative to the folder of the survey's JSON file. |
| `alt` | text |  |  |
| `title` | text |  |  |

## `TableContent`

A table shown as stimulus (not a response grid): every cell keeps its row and column.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"table"` | required |  |
| `columns` | list of `TableColumn` (at least 1) | required |  |
| `rows` | list of `TableRow` (at least 1) | required |  |
| `caption` | text or `Template` |  | Text, or {"parts": [...]} when it pipes values. |
| `corner` | text or `Template` |  | The label above the row labels. |

## `TableColumn`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |

## `TableRow`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `label` | text or `Template` |  | Text, or {"parts": [...]} when it pipes values. |
| `cells` | map of text to text or `Template` | required | Cell text by column ID. |

## `Response`

What a question asks for.

One of: `ChoiceResponse`, `TextResponse`, `NumberResponse`, `MatrixResponse`, `FormResponse`, `RankingResponse`, `AllocationResponse`, `GroupResponse`, `BestWorstResponse`.

## `ChoiceResponse`

One option, or several.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"choice"` | required |  |
| `mode` | one of `"single"`, `"multiple"` | required |  |
| `options` | list of `Option` (at least 1) | required |  |
| `required` | true or false or `"request"` |  |  |
| `min_selected` | whole number (at least 0) |  |  |
| `max_selected` | whole number (at least 1) |  |  |
| `default` | list of text (no spaces, not empty) |  | Options selected before the respondent answers. |
| `randomization` | `Randomization` |  |  |

## `Option`

An answer option, a scale point, a ranked item or an allocation item.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `code` | text or number or true or false |  | The value exported for this option (a recode); the ID when absent. |
| `media` | list of `Media` |  |  |
| `write_in` | true or false |  | The option has a text box for the respondent's own words. |
| `show_if` | `Condition` |  | The option is shown only when this holds. |

## `Media`

An image.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `source` | text (not empty) | required | A web link, a data: address, or the path of a PNG, JPEG, WEBP or GIF file: absolute, or relative to the folder of the survey's JSON file. |
| `alt` | text |  |  |
| `title` | text |  |  |

## `Condition`

A condition, as a tree. Combined conditions are read left to right.

One of: `ConditionGroup`, `ConditionGroup`, `NotCondition`, `ChoiceCondition`, `ChoiceCondition`, `DisplayCondition`, `DisplayCondition`, `Comparison`, `Comparison`, `Comparison`, `Comparison`, `Comparison`, `Comparison`, `Comparison`, `Comparison`, `Comparison`, `Comparison`, `Comparison`, `ConstantCondition`, `ConstantCondition`, `SourceCondition`.

## `ConditionGroup`

All ('and') or any ('or') of several conditions.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `op` | one of `"and"`, `"or"` | required |  |
| `args` | list of `Condition` (at least 1) | required |  |

## `NotCondition`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `op` | `"not"` | required |  |
| `arg` | `Condition` | required |  |

## `ChoiceCondition`

Whether an option was selected.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `op` | one of `"selected"`, `"not_selected"` | required |  |
| `question` | text (no spaces, not empty) | required |  |
| `option` | text (no spaces, not empty) | required | The option ID (for a grid, a scale point's ID: selected when any statement got it). |
| `loop` | one of `"all"`, `"any"` |  | In a repeated block: every row ('all') or some row ('any'). |
| `description` | text |  | How the source platform describes this condition. |

## `DisplayCondition`

Whether a question (or one of its options) was shown.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `op` | one of `"displayed"`, `"not_displayed"` | required |  |
| `question` | text (no spaces, not empty) | required |  |
| `option` | text (no spaces, not empty) |  |  |
| `description` | text |  |  |

## `Comparison`

A comparison of a value with another.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `op` | one of `"eq"`, `"ne"`, `"lt"`, `"lte"`, `"gt"`, `"gte"`, `"contains"`, `"not_contains"`, `"matches"`, `"empty"`, `"not_empty"` | required |  |
| `left` | `Ref` | required | A variable, part of an answer, or a repeat value. |
| `right` | text or number or true or false or `Template` |  | The value compared with; omitted for 'empty' and 'not_empty'. |
| `ignore_case` | true or false |  |  |
| `description` | text |  |  |

## `ConstantCondition`

Always true or always false ('If True' / 'If False').

| Field | Type | Required | Meaning |
|---|---|---|---|
| `op` | one of `"true"`, `"false"` | required |  |
| `description` | text |  |  |

## `SourceCondition`

A condition with no standard form (a device, quota or location test), kept as written.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `op` | `"source"` | required |  |
| `qualtrics` | any value | required | The condition as the source platform wrote it. |
| `description` | text |  |  |

## `Randomization`

How options (or rows) are ordered for each participant.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `order` | one of `"shuffle"`, `"random_reverse"`, `"positions"`, `"source"` | required | 'shuffle': a random order (of 'take' drawn at random, when set); 'random_reverse': reversed for a random half of participants; 'positions': each position pinned or drawn; 'source': a source platform setting with no standard form, kept and reported, not run. |
| `take` | whole number (at least 1) |  | ('shuffle') Show only this many, drawn at random. |
| `consistent` | true or false |  | ('random_reverse') Reverse every such question for the same participants. |
| `positions` | list of text (no spaces, not empty) or `Draw` |  | ('positions') One entry per shown position: a pinned option, or a draw. |
| `shuffled` | list of text (no spaces, not empty) |  | ('positions') The options drawn into 'shuffled' slots, in random order. |
| `subset` | list of text (no spaces, not empty) |  | ('positions') The options a 'subset' slot draws from. |
| `subset_take` | whole number (at least 1) |  | ('positions') How many the 'subset' draws. |
| `hidden` | list of text (no spaces, not empty) |  | ('positions') Options never shown. |
| `qualtrics` | any value |  | ('source') The setting as the source platform wrote it. |

## `Draw`

A position filled by a draw.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `draw` | one of `"shuffled"`, `"subset"` | required |  |

## `TextResponse`

Free text.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"text"` | required |  |
| `required` | true or false or `"request"` |  |  |
| `min_length` | whole number (at least 0) |  |  |
| `max_length` | whole number (at least 1) |  |  |
| `format` | one of `"number"`, `"email"`, `"phone"`, `"date"`, `"zip"`, `"us_state"`, `"text_only"` |  | A named content check. |
| `number` | `NumberCheck` |  | The limits of a 'number' format. |
| `default` | text or `Template` |  | Text already in the box. |

## `NumberCheck`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `min` | number |  |  |
| `max` | number |  |  |
| `decimals` | whole number (at least 0) |  |  |

## `NumberResponse`

A number (typed, or on a slider, scale or stars).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"number"` | required |  |
| `required` | true or false or `"request"` |  |  |
| `min` | number |  |  |
| `max` | number |  |  |
| `step` | number (at least 0) |  |  |
| `decimals` | whole number (at least 0) |  |  |
| `not_applicable` | `NotApplicable` |  | An answer that is not a number. |

## `NotApplicable`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |

## `MatrixResponse`

A grid of statements.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"matrix"` | required |  |
| `mode` | one of `"per_row"`, `"per_cell"` | required | One answer per row, or one per row and column. |
| `rows` | list of `MatrixRow` (at least 1) | required |  |
| `row_response` | `Response` |  | ('per_row') The response every row takes. |
| `columns` | list of `MatrixColumn` |  | ('per_cell') The columns. |
| `cell_response` | `Response` |  | ('per_cell') The response every cell takes. |
| `constraints` | list of `SumConstraint` |  |  |
| `required` | true or false or `"request"` |  |  |
| `randomization` | `Randomization` |  | How the rows are ordered. |

## `MatrixRow`

A statement of a grid.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `media` | list of `Media` |  |  |
| `show_if` | `Condition` |  |  |
| `left_label` | text or `Template` |  | A bipolar row's left end. |
| `right_label` | text or `Template` |  | A bipolar row's right end. |
| `default` | text or number or true or false or list of text (no spaces, not empty) |  | The row's preset answer: a number for a number row, option IDs for a choice row. |

## `MatrixColumn`

A column of a grid that asks one answer per cell.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `code` | text or number or true or false |  |  |
| `media` | list of `Media` |  |  |
| `show_if` | `Condition` |  |  |

## `SumConstraint`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"sum_equals"` | required |  |
| `axis` | one of `"row"`, `"column"` | required |  |
| `total` | number | required |  |

## `FormResponse`

Several labelled text boxes.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"form"` | required |  |
| `fields` | list of `FormField` (at least 1) | required |  |
| `field_response` | `TextResponse` |  | The checks every field's text gets (length, format). |
| `required` | true or false or `"request"` |  |  |

## `FormField`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `media` | list of `Media` |  |  |
| `show_if` | `Condition` |  |  |
| `default` | text or `Template` |  | Text already in the box. |

## `RankingResponse`

A strict order of items, 1 the highest.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"ranking"` | required |  |
| `items` | list of `Option` (at least 1) | required |  |
| `required` | true or false or `"request"` |  |  |
| `min_ranked` | whole number (at least 0) |  |  |
| `max_ranked` | whole number (at least 1) |  |  |
| `default` | list of `RankDefault` |  |  |
| `randomization` | `Randomization` |  |  |

## `RankDefault`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `item` | text (no spaces, not empty) | required |  |
| `rank` | whole number (at least 1) | required |  |

## `AllocationResponse`

An amount split across items that adds up to a total.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"allocation"` | required |  |
| `items` | list of `Option` (at least 1) | required |  |
| `total` | number |  | What the amounts add up to; 100 when absent. |
| `required` | true or false or `"request"` |  |  |
| `min` | number |  | The least one item may get. |
| `max` | number |  | The most one item may get. |
| `default` | map of text to text or number or true or false |  | Preset amounts by item ID. |
| `randomization` | `Randomization` |  |  |

## `GroupResponse`

Several grids side by side, each column with its own statements and scale.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"group"` | required |  |
| `layout` | `"side_by_side"` | required |  |
| `fields` | list of `GroupField` (at least 1) | required |  |
| `required` | true or false or `"request"` |  |  |
| `randomization` | `Randomization` |  | How each column's options are ordered. |

## `GroupField`

One column of a side-by-side question.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `export_name` | text |  | The column's name in exports. |
| `media` | list of `Media` |  |  |
| `response` | `MatrixResponse` | required | The column: its statements and the answer each takes. |
| `source` | any value |  |  |

## `BestWorstResponse`

The best and the worst of the items shown, two different ones.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"best_worst"` | required |  |
| `best_label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `worst_label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `required` | true or false or `"request"` |  |  |
| `options_source` | `"shown_items"` |  |  |

## `SkipRule`

After answering, jump ahead when a condition holds.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) |  |  |
| `when` | `Condition` | required |  |
| `to` | text (not empty) | required | A later question's ID, 'end_of_block' or 'end_of_survey'. |
| `description` | text |  |  |
| `source` | any value |  |  |

## `Presentation`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `control` | one of `"list"`, `"horizontal"`, `"columns"`, `"dropdown"`, `"select_box"`, `"single_line"`, `"multi_line"`, `"essay"`, `"password"`, `"slider"`, `"bar"`, `"stars"`, `"number_scale"`, `"drag_and_drop"`, `"text_boxes"`, `"bipolar"`, `"carousel"`, `"graphic"` |  | How the response is shown (it can change how the question is described to the model, never what is asked). |

## `QuestionSource`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `format` | text |  |  |
| `type` | text |  |  |
| `settings` | map of text to any value |  | Source settings that change nothing simulated, kept as read. |
| `notes` | list of text |  |  |

## `Block`

One of: `SurveyBlock`, `ConjointBlock`, `MaxDiffBlock`.

## `SurveyBlock`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `kind` | `"survey"` | required |  |
| `title` | text |  |  |
| `questions` | list of text (no spaces, not empty) | required | The block's questions, in order. |
| `randomization` | `BlockRandomization` |  |  |
| `repeat` | `Repeat` |  |  |
| `source` | any value |  |  |

## `BlockRandomization`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `order` | one of `"shuffle"`, `"source"` | required | 'shuffle': the block's questions in a random order; 'source': a setting with no standard form, kept and reported, not run. |
| `qualtrics` | any value |  | ('source') The setting as the source platform wrote it. |

## `Repeat`

The block runs once per row (Qualtrics Loop & Merge); its question IDs get the row ID in front (1_QID5).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `rows` | list of `RepeatRow` (at least 1) | required | One run of the block per row, in this order. |
| `order` | one of `"fixed"`, `"shuffle"` |  | 'shuffle' runs the rows in a random order. |
| `take` | whole number (at least 1) |  | ('shuffle') Run only this many rows, drawn at random. |

## `RepeatRow`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `values` | map of text to text or `Template` | required | The row's values by field name; ${lm://Field/name} pipes one. |

## `ConjointBlock`

A choice-based conjoint: the engine draws each participant's profiles from the attribute levels and rules.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `kind` | `"conjoint"` | required |  |
| `title` | text |  |  |
| `attributes` | list of `Attribute` (at least 1) | required |  |
| `alternatives` | `Alternatives` | required |  |
| `constraints` | `ConjointConstraints` |  |  |
| `design` | `ConjointDesign` | required |  |
| `presentation` | `ConjointPresentation` |  |  |
| `response` | `ConjointResponse` |  |  |
| `administration` | `Administration` |  |  |

## `Attribute`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `name` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `type` | one of `"categorical"`, `"numeric"`, `"price"` |  |  |
| `currency` | text |  |  |
| `unit` | text |  |  |
| `levels` | list of `Level` (at least 1) | required |  |

## `Level`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `value` | text or number or true or false |  | A number or price value, distinct from the label. |
| `weight` | number (at least 0) |  | Relative chance of drawing this level; 0 never draws it. Default 1. |
| `media` | list of `Media` |  |  |

## `Alternatives`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `mode` | one of `"generic"`, `"fixed"` | required |  |
| `count` | whole number (at least 1) |  | ('generic') Alternatives per task. |
| `label` | text |  | ('generic') Label template; {letter} and {number} name the position. Default 'Option {letter}'. |
| `items` | list of `Alternative` (at least 1) |  | ('fixed') The alternatives every task shows. |

## `Alternative`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `fixed_levels` | map of text to text (no spaces, not empty) |  | Attribute ID to the level this alternative always has. |

## `ConjointConstraints`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `applicability` | list of `Applicability` |  |  |
| `forbidden_profiles` | list of map of text to text (no spaces, not empty) |  | Each map of attribute ID to level ID is a combination never shown in one profile. |

## `Applicability`

Which levels of two attributes may appear together in one profile.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `row_attribute` | text (no spaces, not empty) | required |  |
| `row_levels` | list of text (no spaces, not empty) (at least 1) | required |  |
| `column_attribute` | text (no spaces, not empty) | required |  |
| `column_levels` | list of text (no spaces, not empty) (at least 1) | required |  |
| `allowed` | list of list of whole number (at least 0, at most 1) (at least 1) (at least 1) | required | 1 where the row level may go with the column level. |

## `ConjointDesign`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `mode` | one of `"generated"`, `"supplied"`, `"adaptive"` | required | Where the tasks come from: drawn for each participant ('generated'), listed in versions ('supplied'), or (0.2) built one at a time from the participant's earlier answers ('adaptive'; asked one task per call, the answer a choice). |
| `generator` | one of `"random_valid_profiles"`, `"polyhedral"` |  | 'random_valid_profiles' for a generated design; 'polyhedral' for an adaptive one (Toubia, Hauser and Simester 2004). |
| `tasks_per_participant` | whole number (at least 1) |  |  |
| `generation_timing` | one of `"block_entry"` |  |  |
| `duplicates` | one of `"allow"`, `"within_task"`, `"within_participant"` |  | Where two identical profiles may not appear: in one task ('within_task', the default; two identical options never share a task, and 'allow' from older surveys reads the same), or anywhere in one participant's tasks ('within_participant'; not for an adaptive design). The generator tries to avoid repeated sets of options; a repeat is recorded when its bounded search finds no new set. |
| `max_attempts` | whole number (at least 1) |  | Draws tried per profile before the design is called infeasible. Default 1000. |
| `versions` | list of `DesignVersion` (at least 1) |  | ('supplied') Complete task versions. |
| `assignment` | one of `"random"`, `"by_index"` |  | ('supplied') How a participant gets a version. |

## `DesignVersion`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `tasks` | list of `DesignTask` (at least 1) | required |  |

## `DesignTask`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `alternatives` | list of `DesignAlternative` (at least 1) | required |  |

## `DesignAlternative`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `levels` | map of text to text (no spaces, not empty) | required |  |

## `ConjointPresentation`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `instruction` | text or `Template` |  | Text, or {"parts": [...]} when it pipes values. |
| `layout` | one of `"table"`, `"list"` |  | How a task shows its alternatives: a table with a column per alternative and a row per attribute ('table', the default), or each alternative with a line per attribute ('list'). |
| `attribute_order` | `OrderSetting` |  |  |
| `alternative_order` | `OrderSetting` |  |  |

## `OrderSetting`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `mode` | one of `"fixed"`, `"shuffle"` | required |  |
| `scope` | one of `"participant"`, `"task"` |  | Draw the order once per participant, or once per task. |
| `fixed_positions` | map of text to whole number (at least 1) |  | ID to a 1-based position kept while the rest shuffle. |

## `ConjointResponse`

What the twin answers in each task.

One of: `ConjointChoiceResponse`, `ConjointRatingResponse`, `ConjointRankingResponse`, `ConjointAllocationResponse`, `ConjointBestWorstResponse`.

## `ConjointChoiceResponse`

Pick one of the alternatives shown (choice-based conjoint).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"choice"` | required |  |
| `required` | true or false or `"request"` |  |  |
| `outside_option` | `OutsideOption` |  | An extra option such as 'None of these'. |

## `OutsideOption`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `code` | text or number or true or false |  |  |

## `ConjointRatingResponse`

(0.2) Rate each alternative shown on one scale (rating-based conjoint). A task may show one alternative.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"rating"` | required |  |
| `required` | true or false or `"request"` |  |  |
| `scale` | list of `Option` (at least 2) | required | The scale every alternative is rated on, lowest first; an option's code is exported. |

## `ConjointRankingResponse`

(0.2) Rank the alternatives shown, 1 the most preferred (ranking-based conjoint).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"ranking"` | required |  |
| `required` | true or false or `"request"` |  |  |

## `ConjointAllocationResponse`

(0.2) Split a number of points across the alternatives shown (point allocation, volumetric conjoint).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"allocation"` | required |  |
| `required` | true or false or `"request"` |  |  |
| `total` | number (at least 0) | required | The points (or purchases) split across the alternatives, such as 10 or 100. |
| `outside_option` | `OutsideOption` |  | An extra row such as 'None of these', which may also get points. |

## `ConjointBestWorstResponse`

(0.2) Pick the best and the worst of the alternatives shown, two different ones (best-worst conjoint).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `type` | `"best_worst"` | required |  |
| `required` | true or false or `"request"` |  |  |
| `best_label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `worst_label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |

## `Administration`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `unit` | one of `"task"`, `"block"` |  | Ask all tasks of the block in one model call, each with its own answer ('block', the default), or one task per call ('task'). An adaptive design is always asked one task per call. |

## `MaxDiffBlock`

A best/worst (MaxDiff) block: the engine draws each participant's item subsets.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `kind` | `"maxdiff"` | required |  |
| `title` | text |  |  |
| `items` | list of `MaxDiffItem` (at least 2) | required |  |
| `design` | `MaxDiffDesign` | required |  |
| `presentation` | `MaxDiffPresentation` |  |  |
| `response` | `BestWorstResponse` | required |  |
| `anchor` | `Anchor` |  |  |
| `administration` | `Administration` |  |  |

## `MaxDiffItem`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `label` | text or `Template` | required | Text, or {"parts": [...]} when it pipes values. |
| `code` | text or number or true or false |  |  |
| `media` | list of `Media` |  |  |

## `MaxDiffDesign`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `mode` | one of `"generated"`, `"supplied"` | required |  |
| `generator` | one of `"uniform_subsets"`, `"balanced_subsets"` |  |  |
| `tasks_per_participant` | whole number (at least 1) |  |  |
| `items_per_task` | whole number (at least 2) |  |  |
| `generation_timing` | one of `"block_entry"` |  |  |
| `duplicate_tasks` | one of `"allow"`, `"forbid"` |  | Read from older surveys, and no longer used: a participant gets the same subset twice only when every other has been shown. |
| `max_attempts` | whole number (at least 1) |  |  |
| `versions` | list of `MaxDiffVersion` (at least 1) |  | ('supplied') Each version's tasks, as lists of item IDs. |
| `assignment` | one of `"random"`, `"by_index"` |  |  |

## `MaxDiffVersion`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `tasks` | list of list of text (no spaces, not empty) (at least 2) (at least 1) | required |  |

## `MaxDiffPresentation`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `instruction` | text or `Template` |  | Text, or {"parts": [...]} when it pipes values. |
| `item_order` | `OrderSetting` |  |  |

## `Anchor`

A follow-up after each task about the same items (anchored MaxDiff).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `content` | list of `Content` | required |  |
| `response` | `ChoiceResponse` | required |  |

## `FlowNode`

A step of the survey flow.

One of: `ShowBlock`, `SetVariables`, `Branch`, `Randomize`, `Group`, `End`.

## `ShowBlock`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) |  |  |
| `type` | `"show_block"` | required |  |
| `block` | text (no spaces, not empty) | required |  |

## `SetVariables`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `type` | `"set_variables"` | required |  |
| `assignments` | list of `Assignment` | required |  |

## `Assignment`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `variable` | text (not empty) | required |  |
| `value` | text or `Template` |  | The value set; text may pipe other values. |
| `from_outside` | true or false |  | The value comes from outside the survey (a panel, a link). |
| `source` | any value |  |  |

## `Branch`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `type` | `"branch"` | required |  |
| `when` | `Condition` | required |  |
| `then` | list of `FlowNode` | required | The steps run when the condition holds. |
| `else` | list of `FlowNode` |  | The steps run when it does not (decided before 'then' runs). |
| `source` | any value |  |  |

## `Randomize`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `type` | `"randomize"` | required |  |
| `take` | whole number (at least 1) | required |  |
| `evenly` | true or false |  | Balance the arms across participants (true, the default), or draw each participant's arms independently (false). |
| `children` | list of `FlowNode` | required |  |
| `source` | any value |  |  |

## `Group`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `type` | `"group"` | required |  |
| `title` | text |  |  |
| `children` | list of `FlowNode` | required |  |
| `source` | any value |  |  |

## `End`

| Field | Type | Required | Meaning |
|---|---|---|---|
| `id` | text (no spaces, not empty) | required |  |
| `type` | `"end"` | required |  |
| `source` | any value |  |  |
