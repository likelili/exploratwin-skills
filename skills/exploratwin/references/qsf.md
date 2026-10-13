# Qualtrics (QSF) surveys

A QSF is the file Qualtrics exports for a survey (Tools → Import/Export →
Export Survey). ExploraTwin runs it as Qualtrics would show it to a
respondent: its blocks, questions, display and skip logic, branches,
randomizers, embedded data, piping and loops. What is and is not supported:
https://exploratwin.org/support/qsf.

## Steps

1. Make a project for it: `POST /surveys` with `{"title": "..."}`.
2. Upload the file: `POST /surveys/{id}/qsf` (multipart `file`, at most 10 MB;
   `GET /options` gives the current limit). The project is named after the
   survey when it still has a placeholder name.
3. Preview it: `POST /surveys/{id}/preview` with `{}`. Read:
   - `blocks`: each `block_id`, `name`, number of `questions`, and
     `can_end_the_survey` (a block whose answers can end the survey early, a
     screen-out);
   - `warnings`: what the file has that is not run, or is run differently;
   - `pictures`: every picture the selected blocks show, with its `status`:
     `supported`, `inaccessible` (the link does not load), `auth_required`
     (behind a Qualtrics login) or `skipped`;
   - `survey_text`: what a sample twin reads (a survey that runs in stages:
     its first stage).
4. Decide what to leave out. To leave blocks out of the run (for example a
   consent page, an attention check the person does not want, or a screen-out),
   list them in the run settings as `qsf_excluded_block_ids`, and preview again
   with the same list in `qsf_excluded_block_ids`. By default every block runs,
   as on the website. Ask the person before leaving out anything that changes
   the study.
5. Estimate and prepare the run as for any survey ([runs.md](runs.md)). The
   report setting defaults to `"none"` for a QSF.

## Pictures the model cannot load

A Qualtrics picture is often a link into the person's Qualtrics account; the
model cannot open a link that needs a login, or one that no longer works. A
question that shows such a picture is not asked (it is recorded as skipped),
and a picture in descriptive text is left out. The run request lists these
pictures (`images_the_model_cannot_load`); the person decides, when approving,
whether to run without them. Tell the person which ones, and that they can fix
the links in Qualtrics (a public library picture), export again, and have you
upload the new file. Never decide this for them.

## Things to know

- A project holds one survey: a QSF or a builder survey, not both. Uploading
  another QSF to the same project replaces the first, and its waiting requests
  become `stale` (the person would review the newer file).
- A request freezes the exact QSF file it was made with.
- A survey whose logic needs earlier answers before later questions runs in
  stages (`runs_in: "stages"`) and cannot use Batch.
- Qualtrics conjoint and MaxDiff projects are not in a QSF; write such a study
  as a conjoint or MaxDiff block in the standard format instead
  ([survey-format.md](survey-format.md)).
- Question JavaScript, quotas, and web services are not run; the preview's
  warnings say when a file has them.
