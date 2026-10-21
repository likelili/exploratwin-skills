# Troubleshooting

Every error is `{"error": {"code", "message", "hint"}}`. Read the code, do what
the table says, and never retry a refused call unchanged in a loop. When
nothing here helps, tell the person the code and message; they can report it
on the website (Support).

## Sign-in and access

| Code | Status | What to do |
|---|---|---|
| `key_required` | 401 | Send `Authorization: Bearer etw_...`. A website session or anything else is not accepted. |
| `key_invalid` | 401 | The key is wrong or cut short. Ask the person to check it, or to make a new one. |
| `key_revoked`, `key_expired` | 401 | Ask the person for a new key (Connect an agent, at the top right of the website). Stop until you have it. |
| `account_not_active` | 403 | The account is not approved or active. The person sorts this out on the website. |
| `agent_access_closed` | 403 | Agent access is not open to this account yet. Nothing to do but wait. |
| `capability_missing` | 403 | The key may not do this. The person can make a key with that capability. |
| `forbidden` | 403 | Something on the account is not available to it (for example twin banks). Read the message. |
| `rate_limited` | 429 | Too many calls. Wait `Retry-After` seconds, then go slower (about one call a second). |
| `internal_error` | 500 | Something went wrong on the server. Retry later with growing pauses; if it keeps happening, tell the person. |

## Surveys

| Code | Status | What to do |
|---|---|---|
| `survey_not_found` | 404 | No such project on this account. List them with `GET /surveys`. |
| `invalid_request` | 422 | The call's fields are wrong; `problems` names each. Fix them (see [api.md](api.md)). |
| `revision_conflict` | 409 | The survey changed since you read it. Read it again, reapply your change, save with the new revision. Never save over the newer version. |
| `project_holds_qsf`, `project_holds_builder_survey` | 409 | A project holds one kind of survey. Make a new project. |
| `project_already_ran` | 409 | The project ran, and its results stay with its survey. Read its survey (`GET /surveys/{id}/definition`), save it in a new project (`POST /surveys`), and change it or prepare the run there. |
| `earlier_survey_not_saved` | 409 | The project's survey was made in the website's earlier builder. Read it in the standard format (`GET /surveys/{id}/definition`), save it (`PUT`), then prepare the run. |
| `survey_not_in_format` | 422 | The JSON is too far from the standard format to be read; `problems` say where. Compare it with the schema. |
| `no_survey` | 409 | The project has no survey yet. Save one or upload a QSF. |
| `survey_has_problems` | 422 | The survey has errors (`problems`, each with a `path`). Fix them; `POST /survey-check` shows them all. |
| `survey_too_large` | 413 | Over 5 MB of JSON. Shorten it, or split it into two surveys. |
| `picture_not_found` | 422 | The survey uses an `upload:` picture this account does not have. Upload it (`POST /surveys/{id}/images`) and use the new source. |
| `pictures_too_large` | 413 | The survey's pictures pass the total limit (50 MB). Use smaller pictures. |
| `not_a_qsf` | 400 | Upload a `.qsf` file exported from Qualtrics. |
| `too_large` | 413 | The upload passes its size limit (`GET /options` lists them). |
| `preview_failed` | 400 | The preview could not be built; the message says why (for example, no twin matches). |
| `block_not_found` | 404 | No conjoint or MaxDiff block with that ID. |
| `twin_bank_not_found` | 404 | No such twin bank, version or view. `GET /options` lists the person's banks. |
| `twin_bank_warming` | 503 | The bank's files are being prepared on the server. Try again in a minute or two. |

## Run requests

| Code | Status | What to do |
|---|---|---|
| `idempotency_key_required` | 400 | Send an `Idempotency-Key` header. |
| `idempotency_key_reused` | 409 | That key was used for different settings. Use a new key for new settings; reuse a key only to retry the same request. |
| `model_not_allowed` | 422 | Pick a model from `GET /options`. |
| `too_many_twins` | 422 | Ask for no more twins than `limits.twins_per_run`. |
| `unknown_filter` | 422 | Use only the filter fields and values `GET /options` lists. |
| `invalid_settings` | 422 | A setting has the wrong form; `problems` names each (by the API's names). |
| `session_bank_not_supported` | 400 | A bank uploaded for one session cannot be used by a request. The person can run it on the website. |
| `too_many_waiting` | 429 | 20 requests already wait. Ask the person to decide them, or withdraw ones they no longer want. |
| `request_not_found` | 404 | No such request on this account. |
| `request_stale`, `request_declined`, `request_withdrawn`, `request_expired`, `request_executing`, `request_succeeded`, `request_failed` | 409 | The request is no longer waiting, so it cannot be withdrawn. Read it with `GET /requests/{id}`. |

## The persona generator

| Code | Status | What to do |
|---|---|---|
| `persona_generator_closed` | 403 | The generator is not open to this account (admins and beta testers have it). Tell the person; they can ask ExploraTwin for access. |
| `census_not_installed` | 503 | The census data is not on the server yet. Try again later; tell the person if it lasts. |
| `plan_has_problems` | 422 | The plan breaks a rule; `problems` lists each. Fix them and check with `POST /persona-generator/plan/check`. |
| `generator_refused` | 400 | The generator cannot do this as asked (for example, more profiles than the matching records can make different). Change the plan or the number as the message says. |
| `generation_running` | 409 | A bank is already being created for this account. Wait until it is done. |
| `insufficient_credits` | 402 | Not enough credits for the plan or the preview. Tell the person. |

A bank request that is `pending` again after an approval, with
`outcome.message`, could not start its bank (for example the server
restarted). It waits for the person again, so tell them the message.

A request's own `status` tells you what happened to it: see the table in
[api.md](api.md). A `failed` request's run ended in failure. A request that is
`pending` again after an approval, with `outcome.error` and `outcome.message`,
could not start its run (for example its twin bank was removed); it waits for
the person again, so tell them the message.

## Runs and results

| Code | Status | What to do |
|---|---|---|
| `run_not_found` | 404 | No such run on this account. |
| `run_not_finished` | 409 | Wait for `completed` (`GET /runs/{id}`, with backoff). Result files come only from a completed run. |
| `file_not_found` | 404 | Use a file name from `GET /runs/{id}/exports`. |

A run that ends `failed` reports `error`. Tell the person; do not prepare the
same run again without asking.

## Other problems

- **A 5xx error or a timeout.** Wait and retry a few times with growing pauses
  (30 seconds, 2 minutes, 5 minutes). Retry a run request with the same
  `Idempotency-Key`, so it is never prepared twice.
- **The person approved, but the request still says `pending`.** Read it again
  after a minute. If the person saw a message instead (a changed estimate,
  pictures the model cannot load, too few credits), they need to deal with it
  there. If `outcome.message` is set, the run could not start: tell them why.
- **Twins answered "skipped".** A skipped answer is recorded when a twin gave
  no usable answer or a picture could not be loaded. Report the count; the
  person can repair or rerun answers on the website.
