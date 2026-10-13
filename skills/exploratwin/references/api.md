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
  paid model. Only an approved run costs credits.

## Account and options

### `GET /me` (any capability)

```json
{
  "account": {"email": "...", "name": "..."},
  "key": {"name": "Claude Code on my laptop", "capabilities": ["surveys:read", "..."], "expires_at": "2027-01-11T..."},
  "credits": {"available_usd": "$24.10", "used_this_month_usd": "$5.90", "resets_at": "...", "runs_stop_when_used_up": true},
  "api_version": "1",
  "skill_version": "1.0.0",
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
builder; agents cannot run it) or `none`.

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
`builder`.
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
project (`409 project_holds_qsf`) and for a project whose survey was made in
the website's earlier builder (`409 project_holds_earlier_survey`).

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
time (`429 too_many_waiting`).

### `GET /requests` (`runs:request`)

`?status=pending,stale&limit=&cursor=`. The account's requests, newest first.

### `GET /requests/{id}` (`runs:request`)

One request. `status`:

| Status | Meaning |
|---|---|
| `pending` | Waiting for the person |
| `stale` | The survey changed (or a newer QSF was uploaded) after the request was made; it will not run |
| `declined` | The person said no |
| `withdrawn` | You withdrew it, or the key was revoked |
| `expired` | Not decided within 7 days |
| `executing` | Approved; `run_id` is set (a request whose run could not start goes back to `pending`, with `outcome.message`) |
| `succeeded`, `failed` | The run's outcome (`outcome` says why it failed) |

### `POST /requests/{id}/withdraw` (`runs:request`)

Withdraw a `pending` or `stale` request.

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
settings, starting a run directly, deleting anything, repairs and reruns,
panels and live Twin Meet calls, uploading twin banks, and the website's own AI
survey writer. The person does these on the website.

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
