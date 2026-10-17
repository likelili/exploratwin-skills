# The persona generator: synthetic profiles as a private bank

The persona generator makes synthetic profiles of US adults and saves them as
the person's private persona bank: one of their twin banks, usable in runs
and seen by no one else. A survey run on that bank asks these profiles
instead of the built-in twins.

How a profile is made:

- **Demographics** (age, gender, state, education, income and the other
  census fields) come whole from real records of the
  American Community Survey 2024 (US Census Bureau).
- **Other characteristics** (such as occupation or the device people use)
  are drawn by the proportions in the plan. The AI proposes those proportions
  (estimates); the person can set them.
- **A description** of each profile's everyday life is written by a model,
  keeping to every fact drawn.
- No two profiles share every fact. The survey under test is never used.

The generator is open to admins and beta testers for now:
`GET /persona-generator/options` says whether it is open to the person
(`available`).

## Costs, and what waits for the person

| Action | Cost | |
|---|---|---|
| `GET /persona-generator/options` | free | |
| `POST /persona-generator/plan` | about $0.01 | charged when you call it |
| `POST /persona-generator/plan/check` | free | |
| `POST /persona-generator/preview` | about $0.01 | charged when you call it |
| `POST /persona-generator/requests` | free | prepares the bank; nothing is drawn or written |
| the bank itself | about $0.10 for 200 profiles | only after the person approves it on the website |

`GET /persona-generator/options` (`costs`) gives the current figures. All
charges follow actual use, as on the website.

The plan and the preview are small charges you may make when the person asks
for a bank. Make one plan, and preview it once or twice. Do not make more
than three plans or three previews for one bank without asking the person.
Say what you spent in your report.

## The steps

1. **Check it is open**: `GET /persona-generator/options`. `available` and
   `census_ready` must both be `true`. It also lists the census fields and
   their values (`demographic_fields`), the standard characteristics, the
   limits and the costs.
2. **Ask once** about what is still open: who the profiles are, how many
   (1 to 1,000; the default is 200), the characteristics they need (with
   values, if the person has them), and whether profiles need names.
3. **Make the plan**: `POST /persona-generator/plan`:

   ```json
   {"audience": "US adults aged 18 to 64 who used an AI chatbot in the past month.",
    "characteristics": [{"name": "Occupation"},
                        {"name": "Monthly subscription spending", "values": ["Under $10", "$10 to $30", "Over $30"]}]}
   ```

   Or start from example profiles: `"examples": ["A software engineer in San
   Francisco", "A retired teacher in Ohio"]` with `"profiles_per_example": 10`
   (1 to 100; 1 makes exactly the profiles listed). The answer: `plan`,
   `problems`, `notes`, `profiles`, `matching_sample_records` (census records
   that fit the criteria), `group_profiles` (profiles per group),
   `warnings` and `estimate`.
4. **Review it**, and edit it when the person wants changes (see "The plan"
   below). Check every edit with `POST /persona-generator/plan/check`
   (`{"plan": ..., "profiles": 200}`, free) until `problems` is empty.
5. **Preview**: `POST /persona-generator/preview` with
   `{"plan": ..., "profiles": 200}` (and a `seed` if you like). It writes up
   to five profiles as the bank would. They are drawn separately from the
   bank's profiles. Read them: do they fit the audience?
6. **Prepare the bank**: `POST /persona-generator/requests` with an
   `Idempotency-Key` header (a value you make up, reused when you retry):

   ```json
   {"plan": {...}, "profiles": 200, "seed": 7, "name": "AI chatbot users",
    "reason": "Profiles for the AI subscription survey pilot."}
   ```

   `reason` is one sentence the person sees as the bank's Description. The
   request freezes the plan, the number of profiles, the seed and the name.
7. **Stop and report**, as plain facts:

   > Prepared the persona bank "{name}" with {profiles} profiles. Estimated
   > cost: {estimate}. The plan and preview cost about {amount}.
   > Review and approve it: {approve_url}

8. **After approval**: `GET /requests/{id}`. While the bank is created,
   `status` is `executing` and `progress` gives `done` and `total`. Check
   every 30 seconds; 200 profiles take a few minutes. `succeeded` gives
   `bank_id` (and `outcome.version_id`); `failed` gives `outcome.message`.
9. **Use it in a run**: `GET /twin-banks/{bank_id}` gives its views
   (`default_view_id`) and filter fields. Set `twin_bank` in the run settings
   (`{"bank_id", "version_id", "representation_view_id"}`), count with
   `POST /twins/count`, then prepare the run as usual ([runs.md](runs.md)).
   Ask for no more twins than the bank has profiles.

## The plan

The plan is JSON. Edit it, then check it.

- `name`: the bank's name.
- `start`: `description` (a described audience, split into groups by
  proportion) or `example` (one group per example profile).
- `population`: the audience, in the person's words.
- `restrictions`: the demographic criteria. Each key is a census field from
  `demographic_fields` with a list of allowed values, written exactly as
  listed (`{"state": ["Ohio", "Texas"]}`), plus `age_min` and `age_max`.
- `segments` (the groups): each has a `name` and:
  - for a described audience, a `share` (the shares are scaled to add to 1);
  - for examples, a `count` (profiles like the example) and `exact`: `true`,
    with a `count` of 1, makes exactly the profile on the line;
  - optionally `restrictions` (narrower criteria for the group) and `traits`
    (characteristic values every profile in the group has, such as
    `{"occupation": "Student"}`).
- `traits` (the characteristics): each has an `id`, a `label`, `values`
  (at most 12), and `shares` (one per value). Optionally `depends_on` (a
  census field or another characteristic) with `shares_by` (for each of its
  values, the shares to use), and `source`: set `"researcher"` when the person
  gave the values and proportions, so they are kept as given.
- `names`: `true` gives each profile a name (no two the same).
- `topic`: what the person studies. Descriptions say nothing about it beyond
  the listed characteristics.

Rules the check enforces:

- Demographics come from the census. Never add one as a characteristic
  (gender, age, race, income, education, employment, marital status,
  household size, citizenship, region, state): narrow it with `restrictions`.
- Characteristics describe who people are and how they live, never their
  opinions on the topic or the answers the study measures. Never copy the
  survey's questions into the plan.
- At most 16 characteristics (10 the person names), 10 groups or 50
  examples, and 1,000 profiles.

## What to tell the person

- The demographics are real census records; each record is used once while
  unused ones remain.
- Proportions the person did not set are the AI's estimates. When they
  matter to the study, show them to the person before preparing the bank.
- Descriptions can read alike. Encourage the person to read a few profiles
  in the preview, or on the bank's page on the website.
