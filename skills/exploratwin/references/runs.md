# Runs: settings, estimates and requests

A run asks a survey of a number of digital twins. You prepare it as a
**request**; it runs only after the person approves it on the website
([approvals.md](approvals.md)).

## Settings

The same object goes to `POST /surveys/{id}/estimate` and, as `settings`, to
`POST /surveys/{id}/run-requests`.

| Field | Default | Meaning |
|---|---|---|
| `model` | `gpt-5.6-luna-low` | The model the twins answer with: an `id` from `GET /options`. Larger models cost more. |
| `twin_type` | `summary` | How each twin is described to the model: `summary` (a paragraph: demographics, personality, thinking style; the usual choice), `full` (every answer the person gave in the source survey; long, costs more), `demographics`, or `empty` (no persona: a baseline). |
| `twins` | `100` | How many twins answer, at most `limits.twins_per_run` of `GET /options`. |
| `filters` | none | For the built-in twins: fields and values from `GET /options` `filters`, such as `{"region": ["West"], "age_band": ["30-49"]}`. Values of one field are alternatives (or); fields combine (and). |
| `twin_bank` | none | Twins from one of the person's twin banks instead ([twin-banks.md](twin-banks.md)): `{"bank_id", "version_id", "representation_view_id", "filters"}`. |
| `seed` | none | A number that fixes which twins are drawn and how the survey is randomized for each; the same seed and settings give the same draws. |
| `use_batch_api` | `false` | Batch: about half the cost; finishes within 24 hours. Only for a survey that runs in one call (`runs_in: "one call"` in `POST /survey-check` or the preview). |
| `report` | `none` for a QSF, `full` for a builder survey | `full` adds a written report of the results (it costs more); `none` gives the data files only. |
| `system_instruction` | the website's | An extra instruction for every twin. Leave it out unless the person asks. |
| `qsf_excluded_block_ids` | none | QSF only: blocks to leave out ([qsf.md](qsf.md)). |
| `email_when_ready` | `true` | Email the person when the run finishes. |

## Before you request

1. `POST /twins/count` with the twin type and filters: are there enough twins?
   A request for more twins than match runs with those that match.
2. `POST /surveys/{id}/estimate` with the settings. The estimate grows with the
   number of twins, questions, the model and the twin type; a report adds to
   it; Batch halves it. It is an estimate, not a cap: the person is charged for
   what the run uses, as for any run on the website.
3. `GET /me` shows the credits available. When the estimate exceeds them, say
   so: the person may not be able to approve until they have more.

## The request

```bash
curl -s -X POST https://exploratwin.org/api/agent/v1/surveys/$SURVEY/run-requests \
  -H "Authorization: Bearer $EXPLORATWIN_API_KEY" -H "Content-Type: application/json" \
  -H "Idempotency-Key: coffee-pilot-2026-10-13" \
  -d '{"settings": {"model": "gpt-5.6-luna-low", "twin_type": "summary", "twins": 200, "seed": 11,
                    "filters": {"region": ["West"]}, "report": "none"},
       "reason": "A 200-twin pilot of the coffee survey, West region, before the field launch."}'
```

- **`Idempotency-Key`** (required): a value you make up for this request.
  Reuse it when you retry the same request after a timeout or an error: you
  get the same request, never a second one. Use a new value for new settings.
- **`reason`**: one sentence that describes the run. The person sees it as
  the run's **Description** in the email and on the approval card. Describe
  the run; do not argue for it.
- The request **freezes** the survey as it is now, the twin bank's exact
  version and view, and every setting. If the survey changes before approval
  (or a newer QSF is uploaded), the request becomes `stale` and will not run;
  prepare a new one.
- At most 20 requests wait at a time; each waits up to 7 days.

The answer includes `approve_url`, the `estimate`, and
`images_the_model_cannot_load`. Give all three to the person, then stop.

## After approval

`GET /requests/{id}` shows `status: "executing"` and the `run_id`. Then
`GET /runs/{id}`:

| Run status | What to do |
|---|---|
| `queued` | Wait; `queue.position` and `queue.estimated_wait_seconds` say how long. |
| `running` | Wait. Check every 30 seconds at first, then every few minutes. |
| `completed` | Fetch the results ([results.md](results.md)). |
| `failed` | Report `error` to the person. Do not prepare the same run again without asking. |
| `canceled`, `cancel_requested` | The person stopped it on the website. |

A Batch run may wait in `running` for hours: check every 15 to 30 minutes, or
tell the person you will look again later (they also get an email).
