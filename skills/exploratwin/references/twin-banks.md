# Twins and twin banks

## The built-in twins

Without a `twin_bank`, a run uses the built-in twins (Twin-2K-500: about 2,000
US adults, each described from their own answers to a long survey). Filter them
with `filters` in the run settings, using the fields and values
`GET /options` lists (`region`, `age_band`, `sex_at_birth`, `income_bracket`,
`education`, `race_origin`, `political_affiliation`, and more). Check how many
match with `POST /twins/count`.

## Twin banks

A twin bank (a Croissant Twins bank) is another set of twins, from another
study or source, in a standard package. `GET /options` lists the banks the
person installed (`twin_banks`); `GET /twin-banks/{bank_id}` gives a bank's
views (how its twins are described to the model) and its filter fields. To run
on one, set `twin_bank` in the run settings:

```json
{"twin_bank": {"bank_id": "...", "version_id": "...", "representation_view_id": "...", "filters": {"country": ["DE"]}},
 "twins": 150}
```

Count with `POST /twins/count` and the same `twin_bank`. Installing a bank from
the store is done by the person on the website.

## Synthetic profiles

The persona generator makes a private bank of synthetic profiles of any
audience the person describes: the AI plans the demographics and
characteristics and predicts their shares, code simulates the profiles from
that plan, and the AI writes a description of each. You make and review the
plan with the person, and prepare the bank as a request they approve; the
finished bank is listed in `twin_banks` like any other. See
[persona-generator.md](persona-generator.md).

## The person's own data

A person can turn their own individual-level data (customer records, a survey
dataset) into a twin bank with the bank-builder skill:
https://exploratwin.org/build-croissant-twins-bank.zip (guide:
https://exploratwin.org/support/create-persona-bank). The skill runs on the
person's computer; follow it when they ask you to build a bank.

Uploading a bank sends people's data to ExploraTwin, so:

- ask the person first, and wait for a clear yes ([approvals.md](approvals.md));
- this API does not upload banks yet: the person uploads the zip on the
  website (the persona bank store's upload button), which shows the size limit
  (750 MB by default) and how long an uploaded bank is kept;
- a run request cannot use a bank uploaded for one session yet: such a bank
  may be gone before the request is approved (`session_bank_not_supported`).
  The person can run on it from the website directly.
