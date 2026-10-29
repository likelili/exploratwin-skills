# The ExploraTwin agent API

Version 1. Base address: `https://exploratwin.org/api/agent/v1`.

## Basics

- **Sign-in.** Every call sends `Authorization: Bearer etw_...`, the person's
  API key. The key acts as its owner, within its capabilities. The website's
  own sign-in is not accepted here, and the key is not accepted by the website.
- **Capabilities.** Each action below names the capability it needs. A key
  made with the defaults has `surveys:read`, `surveys:write`, `runs:request`
  and `results:read`. `previews:persona` (off by default) adds the persona text
  to previews.
- **Format.** JSON in, JSON out. Uploads are `multipart/form-data` with one
  field, `file`.
- **Errors.** Every error has the same shape, with a stable `code`:

  ```json
  {"error": {"code": "revision_conflict", "message": "The survey changed since you read it (it is now at revision 4).", "hint": "Read it again ...", "current_revision": 4}}
  ```

  Some errors add fields: `problems` (a survey's problems), `current_revision`,
  `request_id`. A request with missing or wrong fields gets
  `422 invalid_request` with `problems: [{"field", "message"}]`. Codes and
  what to do: [troubleshooting.md](troubleshooting.md).
- **Limits.** At most 120 calls a minute per key, and short bursts only: keep
  to about one call a second. A `429` carries `Retry-After` (seconds).
- **Paging.** Lists take `limit` (at most 100) and return `next_cursor`; pass it
  back as `cursor` for the next page. `null` means the last page.
- **Free actions.** Reads, checks, previews, counts and estimates never call a
  paid model. Only an approved run or persona bank costs credits, with the
  persona generator's steps before the bank as the exception: its plan (about
  $0.15), an update of the plan (about $0.01 to $0.05) and its preview (about
  $0.01) are charged when you call them, after the person said yes
  ([approvals.md](approvals.md)).

## Account and options

### `GET /me` (any capability)

```json
{
  "account": {"email": "...", "name": "..."},
  "key": {"name": "Claude Code on my laptop", "capabilities": ["surveys:read", "..."], "expires_at": "2027-01-11T..."},
  "credits": {"available_usd": "$24.10", "used_this_month_usd": "$5.90", "resets_at": "...", "runs_stop_when_used_up": true},
  "api_version": "1",
  "skill_version": "1.2.1",
  "skill_url": "https://exploratwin.org/agent/SKILL.md"
}
```

### `GET /options` (any capability)

What a run can use: `models` (each `id` with the provider's price per million
tokens), `default_model`, `twin_types`, `built_in_twins`, `filters` (field →
allowed values, for the built-in twins), `twin_banks` (the person's installed
banks: `bank_id`, `version_id`, `name`, `persona_count`, `views`), `limits`
(`twins_per_run`, `qsf_bytes`, `picture_bytes`, `pictures_total_bytes`,
`survey_json_chars`, `waiting_requests`, `request_days`,
`requests_per_minute`), `switches` and `survey_format` (`versions`, `latest`).

### `GET /twin-banks/{bank_id}` (any capability)

A twin bank's `views` (how its twins are described to the model) and
`filters` (each `field`, `label`, `values`, `value_labels`), for a run's
`twin_bank`. Optional `?version_id=`.

### `POST /twins/count` (`runs:request`)

```json
{"twin_type": "summary", "filters": {"region": ["West"], "age_band": ["18-29", "30-49"]}}
```

or, for a twin bank, `{"twin_bank": {"bank_id": "...", "representation_view_id": "...", "filters": {...}}}`.
Answer: `{"count": 412, "total": 2058, "twin_type": "summary", "filters": {...}}`. Use only the fields and values `GET /options` lists; others are refused (`422 unknown_filter`).

## Surveys

A survey lives in a **project**. A project holds a survey written in the
standard format (`source: "builder"`) or a QSF file (`source: "qsf"`), not both.

### `GET /surveys` (`surveys:read`)

`?limit=&cursor=`. Answer: `{"surveys": [{"id", "title", "source", "revision", "updated_at"}], "next_cursor"}`.
`source` is `builder`, `qsf`, `earlier_builder` (made in the website's earlier
builder: read it in the standard format, and save it before a run) or `none`.

### `POST /surveys` (`surveys:write`)

A new project, empty or with a survey. Never overwrites anything.

```json
{"title": "Coffee habits", "definition": { ...a survey in the standard format... }}
```

Answer (`201`): the project as in `GET /surveys/{id}`, plus the survey's
`problems` and `warnings` when a definition was given. A survey with problems
is saved (so it can be fixed), but cannot run.

### `GET /surveys/{id}` (`surveys:read`)

`id`, `title`, `source`, `revision`, counts of `problems`, `warnings` and
`pictures` (builder surveys), `qsf_files` (QSF projects), and the project's
`requests` and `runs` (newest first).

### `GET /surveys/{id}/definition` (`surveys:read`)

`{"definition": {...}, "revision": 3, "source": "builder", "problems": [...], "warnings": [...]}`.
`source` says what a run on the project uses: this survey only when it is
`builder`. For a survey made in the website's earlier builder (`source:
"earlier_builder"`), the answer gives it converted to the standard format, with
`"converted_from_earlier_builder": true` and revision `0`; save it with `PUT`
(and `expected_revision: 0`) before preparing a run, as the website's builder
does. The earlier survey stays stored.
Each problem: `{"path": "$.questions[2].response.options", "message": "...", "code": "...", "level": "error"}`.

### `PUT /surveys/{id}/definition` (`surveys:write`)

```json
{"definition": {...}, "expected_revision": 3}
```

Saves only when the project is still at `expected_revision` (use `0` for a
project with no survey yet). Otherwise `409 revision_conflict` with
`current_revision`: read the survey again and reapply your change. Each save
raises the revision by one and keeps the earlier version. Answer:
`{"revision": 4, "problems": [...], "warnings": [...]}`. Refused for a QSF
project (`409 project_holds_qsf`) and for a project that ran (`409
project_already_ran`: its results stay with the survey they came from; save the
changed survey in a new project).

The server tidies a survey as it saves it (the format version it needs, and
settings that no longer choose anything), so read it back before your next
edit.

### `POST /survey-check` (`surveys:read`)

`{"definition": {...}}`. Checks without saving: the full server checks, not
only the JSON Schema. Answer: the tidied `definition`, `problems`, `warnings`,
`answers_per_twin` (answers each twin gives) and, when there are no problems,
`runs_in`: `"one call"` or `"stages"` (the survey's logic needs earlier answers
before later questions; such a run cannot use Batch).

### `GET /survey-format` (any capability)

`{"versions": ["0.1", "0.2"], "latest": "0.2", "json_schema": {...}}`: the
format as a JSON Schema (also in [survey-format.schema.json](survey-format.schema.json)).
The schema checks structure only; `POST /survey-check` checks everything.

### `POST /surveys/{id}/images` (`surveys:write`)

Multipart `file`: a PNG, JPEG, WEBP or GIF picture. Answer (`201`):

```json
{"image_id": "img_3f...", "name": "shelf.png", "mime": "image/png", "bytes": 48213, "width": 800, "height": 600,
 "source": "upload:img_3f...", "use": {"type": "image", "source": "upload:img_3f..."}}
```

Put `source` in the survey (image content, or an option's, row's, item's or
level's `media`) and save. A picture stays with the project until the project
is deleted, or until a save of the survey without it, when no other survey of
the account uses it. A picture can be used in several of the account's surveys.

### `POST /surveys/{id}/qsf` (`surveys:write`)

Multipart `file`: a `.qsf` file. Answer (`201`): the project, the file's `id`
and `file_name`. Then preview it. Refused for a builder project
(`409 project_holds_builder_survey`). A later upload replaces the earlier one,
and makes the project's waiting requests `stale`. See [qsf.md](qsf.md).

### `POST /surveys/{id}/preview` (`surveys:read`)

```json
{"twin_type": "summary", "seed": 42, "twin_index": 0, "qsf_excluded_block_ids": []}
```

All fields optional. What one sample twin reads:

- builder survey: `survey_text` (the questions and answer instructions),
  `runs_in`, `questions_in_this_preview`, `warnings`;
- QSF: also `blocks` (each `block_id`, `name`, `questions`,
  `can_end_the_survey`), `warnings` (what the file has that is not run) and
  `pictures` (each picture's `status`: `supported`, `inaccessible`,
  `auth_required` or `skipped`).

`persona_text` is `null` unless the key has `previews:persona`. A survey that
runs in stages shows its first stage; each twin gets its own randomization.

### `POST /surveys/{id}/experiment-preview` (`surveys:read`)

`{"block_id": "coffee", "seed": 42, "twins": 3}`. A conjoint or MaxDiff
block's tasks for sample twins, and how often each level or item is shown. An
adaptive conjoint's tasks here follow made-up answers.

## Estimates and run requests

### `POST /surveys/{id}/estimate` (`runs:request`)

Body: the run settings ([runs.md](runs.md)). Answer:

```json
{"estimated_charge_usd": "$3.12", "estimated_charge_nanos_usd": 3120000000, "persona_count": 100,
 "question_count": 24, "report_mode": "none", "batch_discount_applied": false,
 "note": "An estimate, not a cap: ..."}
```

### `POST /surveys/{id}/run-requests` (`runs:request`)

Header `Idempotency-Key: <a value you choose>` (required). Body:

```json
{"settings": {"model": "gpt-5.6-luna-low", "twin_type": "summary", "twins": 200, "seed": 7,
              "filters": {"region": ["West"]}, "report": "none"},
 "reason": "Pilot of the coffee survey before the field launch."}
```

Answer (`201`): the request:

```json
{"id": "8c1...", "kind": "run", "status": "pending", "survey_id": "...",
 "survey": {"source": "definition", "title": "Coffee habits", "revision": 4},
 "settings": {...as frozen...}, "estimate": {...}, "images_the_model_cannot_load": [],
 "reason": "...", "created_at": "...", "expires_at": "...", "run_id": null,
 "approve_url": "https://exploratwin.org/?agent_request=8c1..."}
```

The request freezes the survey as it is now (a copy, or the exact QSF file),
the twin bank's exact version and view, and every setting; the run will use
exactly these. Sending the same
`Idempotency-Key` with the same body returns the same request; with a
different body, `409 idempotency_key_reused`. At most 20 requests wait at a
time (`429 too_many_waiting`). A project runs once: a request for a project
that already ran is refused (`409 project_already_ran`); save the survey in a
new project and prepare the run there.

### `GET /requests` (`runs:request`)

`?status=pending,stale&limit=&cursor=`. The account's requests, newest first:
runs (`kind: "run"`) and persona banks (`kind: "persona_bank"`).

### `GET /requests/{id}` (`runs:request`)

One request. `status`:

| Status | Meaning |
|---|---|
| `pending` | Waiting for the person |
| `stale` | The survey changed (or a newer QSF was uploaded) after the request was made, or the project ran from another request (`outcome.stale_reason: "project_ran"`); it will not run |
| `declined` | The person said no |
| `withdrawn` | You withdrew it, or the key was revoked |
| `expired` | Not decided within 7 days |
| `executing` | Approved; `run_id` is set (a request whose run could not start goes back to `pending`, with `outcome.message`) |
| `succeeded`, `failed` | The run's outcome (`outcome` says why it failed) |

### `POST /requests/{id}/withdraw` (`runs:request`)

Withdraw a `pending` or `stale` request.

## The persona generator

Synthetic profiles saved as the person's private persona bank, with the
website's own generator. The workflow, what to ask the person, and the plan's
parts: [persona-generator.md](persona-generator.md). A plan, an update and a
preview are charged when you make them (each also shows on the website under
Agent requests); the bank waits for the person's approval.

### `GET /persona-generator/options` (any capability)

`available` (open to this account), `limits` (`profiles` with `min`, `max`,
`default`; `preview_profiles`; `characteristics`, `values`, `examples`,
`per_example`, `people`, `parents`, `combinations`, `text_chars`, and
`demographics` with `min`, `max`, `categories` and `asked`: the person's own,
at most), `required_demographics` (`["age", "gender", "region"]`), `age_groups`,
`models`, `costs` (`plan`, `update` by kind (`audience`, `map`, `examples`,
`tables`), `preview`, `per_profile`, `bank_of_200`, `note`), `times` and
`guide`. Each cost is an estimate (`estimated_charge_usd`,
`estimated_charge_nanos_usd`, `estimated_provider_cost_usd`); dollar amounts
are decimal text, as in every estimate.

### `POST /persona-generator/plans` (`runs:request`, charged: about $0.15)

```json
{"audience": "U.S. adults aged 18 to 60 who may buy a new winter coat.",
 "examples": [], "profiles_per_example": 1,
 "demographics": [{"name": "Household income", "values": ["$75k–$99k", "$100k or more"]}],
 "characteristics": [{"name": "Preferred coat style", "values": []}],
 "names": false, "replace_unfinished": false}
```

Starts the plan; it is made in the background (about 4 minutes). Answer
(`202`): the plan (as `GET /persona-generator/plans/{id}` gives it, `status`
`queued`) and `charge_estimate`. Refusals: `400 form_has_problems`
(`problems`), `409 unfinished_plan` (the person has an unfinished plan in the
website's generator, named in `plan`: ask them, then send
`"replace_unfinished": true`), `409 plan_running`, `402 insufficient_credits`.

### `GET /persona-generator/plans` (`runs:request`, free)

The person's recent plans, newest first; the first is the one the website's
generator offers them. Each: `id`, `status`, `name`, `audience`,
`made_by_agent` (the key's name, or `null` for a plan made on the website),
`unfinished`, `review_url`, `created_at`, `updated_at`.

### `GET /persona-generator/plans/{id}` (`runs:request`, free)

```json
{"id": "3f1...", "status": "ready", "step": null, "progress": {"done": 0, "total": 0}, "error": null,
 "form": {"who": "...", "examples": [], "variations": 1, "demographics": [...], "characteristics": [...], "names": false},
 "plan": {"name": "...", "population": "...", "rule": {"allowed": {...}, "age_min": 18, "age_max": 60, "text": "..."},
          "demographics": {"fields": [...], "finer": {...}, "sources": [...]}, "age_bands": ["18–24", "..."],
          "characteristics": [...], "characteristic_sources": [...], "order": [...], "parents": {...}, "why": {...},
          "overall": {...}, "targets": {}, "rules": [], "examples": [], "names": false},
 "checks": {"problems": [], "warnings": [], "trial": []}, "profiles": 200,
 "estimates": {"update": {...}, "preview": {...}, "per_profile": {...}},
 "made_by_agent": "Claude Code", "review_url": "https://exploratwin.org/?mode=library&persona_plan=3f1...",
 "created_at": "...", "updated_at": "...", "finished_at": "..."}
```

While it is made, `status` is `queued`, `demographics`, `characteristics`,
`audience`, `map` or `tables`, `step` says it in words and `plan` is `null`
(or, during an update, the plan as edited). `failed` gives `error`. `plan`
leaves out the server's percentage tables and logs.

### `POST /persona-generator/plans/{id}/edit` (`runs:request`; free, or charged for an update)

Any of: `name`, `names`, `targets`, `rules` (free, at once); `demographics`,
`characteristics`, `rule`, `examples`, `parents` (an update, run again:
about $0.01 to $0.05). Shapes: [persona-generator.md](persona-generator.md),
step 6. Answer: the plan, with `charged` (and `charge_estimate` when charged);
an update's `status` is `queued` until it is `ready` again. Refusals:
`422 edit_has_problems` (nothing changed; `problems`), `409 plan_not_ready`
(being made or updated), `409 plan_failed`, `400 nothing_to_change`.

### `POST /persona-generator/plans/{id}/retry` (`runs:request`, charged)

A `failed` plan (or update) run again as it was asked. `409 plan_not_failed`
for any other plan.

### `POST /persona-generator/plans/{id}/preview` (`runs:request`, charged: about $0.01)

`{"profiles": 200, "seed": 7}`. The bank's first profiles (up to five,
never more than `profiles`), written as the bank's are. Answer: `seed`,
`profiles` (each `profile_id`, `name`, `group`, `facts`, `description`,
`text`: what a twin's model reads), `warnings`, `charge_estimate`. A plan with
problems: `422 plan_has_problems`.

### `POST /persona-generator/requests` (`runs:request`, free)

Header `Idempotency-Key` (required). Body:

```json
{"plan_id": "3f1...", "profiles": 200, "seed": 7, "name": "Northern U.S. luxury coat shoppers",
 "reason": "Profiles for the winter coat conjoint."}
```

Answer (`201`): the request, `kind: "persona_bank"`:

```json
{"id": "5d2...", "kind": "persona_bank", "status": "pending",
 "bank": {"name": "Northern U.S. luxury coat shoppers", "profiles": 200, "seed": 7, "plan_id": "3f1...",
          "audience": "...", "restrictions": ["Age: 18 to 60"], "demographics": ["Age", "Gender", "..."],
          "characteristics": ["Preferred coat style", "..."], "examples": 0, "names": false},
 "estimate": {...}, "reason": "...", "created_at": "...", "expires_at": "...", "decided_at": null,
 "generation_id": null, "bank_id": null, "outcome": {},
 "review_url": "https://exploratwin.org/?mode=library&persona_plan=3f1...",
 "approve_url": "https://exploratwin.org/?agent_request=5d2..."}
```

It waits for the person, like a run request (`GET /requests/{id}`, the same
statuses, `POST /requests/{id}/withdraw`). It becomes `stale` when the plan
changes after it was prepared (`outcome.stale_reason: "plan_changed"`). While
the bank is created (`executing`), `progress` gives `status`, `done` and
`total`. `succeeded` sets `bank_id` (and `outcome.version_id`); `failed` sets
`outcome.message`. A request whose bank could not start goes back to
`pending`, with `outcome.message`. Refusals: `404 plan_not_found`,
`409 plan_not_ready`, `409 plan_failed`, `422 plan_has_problems`,
`400 generator_refused` (the bank cannot be made as asked; the message says
why), `400 plan_id_required`.

### The earlier generator's routes

`POST /persona-generator/plan`, `/plan/check` and `/preview`, and a bank
request with a `plan` instead of a `plan_id`, answer `410 generator_changed`:
fetch the skill again.

## Runs and results

### `GET /runs/{id}` (`results:read`)

```json
{"id": "...", "survey_id": "...", "status": "running", "created_at": "...", "started_at": "...",
 "finished_at": null, "error": null, "queue": {"position": null, "estimated_wait_seconds": null, "worker_status": "running"},
 "request_id": "8c1...", "settings": {...}, "next": "Check again later, with backoff."}
```

`status`: `queued`, `running`, `cancel_requested`, `canceled`, `completed`,
`failed`.

### `GET /runs/{id}/exports` (`results:read`)

For a completed run: `{"files": [{"file": "labels.csv", "file_name": "...", "download": "/api/agent/v1/runs/{id}/exports/labels.csv"}]}`.
Earlier: `409 run_not_finished`.

### `GET /runs/{id}/exports/{file}` (`results:read`)

Downloads one file of a completed run (`409 run_not_finished` before); it may
answer with a redirect to a signed link, so follow redirects (`curl -L`). Files: `labels.csv`, `values.csv`, `repaired.csv`,
`long.csv`, `wide.csv`, `conjoint-exposures.csv`, `conjoint-partworths.csv`, `maxdiff-exposures.csv`,
`report.pdf`, `question-logic.json`, `manifest.json`, `bundle.zip`. See
[results.md](results.md).

## Not in this API

Approving or declining requests, making or revoking keys, account and billing
settings, starting a run or creating a bank directly, deleting anything,
repairs and reruns, panels and live Twin Meet calls, uploading twin banks, and
the website's own AI survey writer. The person does these on the website.

## Examples

```bash
API=https://exploratwin.org/api/agent/v1
AUTH="Authorization: Bearer $EXPLORATWIN_API_KEY"

# check a survey, then save it in a new project
curl -s -X POST $API/survey-check -H "$AUTH" -H "Content-Type: application/json" -d @survey-check.json
curl -s -X POST $API/surveys -H "$AUTH" -H "Content-Type: application/json" -d @new-survey.json

# upload a picture and a QSF
curl -s -X POST $API/surveys/$SURVEY/images -H "$AUTH" -F "file=@shelf.png"
curl -s -X POST $API/surveys/$SURVEY/qsf -H "$AUTH" -F "file=@study.qsf"

# prepare a run (the same key again returns the same request)
curl -s -X POST $API/surveys/$SURVEY/run-requests -H "$AUTH" -H "Content-Type: application/json" \
  -H "Idempotency-Key: coffee-pilot-1" -d @run-request.json

# download a result file
curl -sL $API/runs/$RUN/exports/labels.csv -H "$AUTH" -o labels.csv
```
