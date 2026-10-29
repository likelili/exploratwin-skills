# The persona generator: synthetic profiles as a private bank

The persona generator makes synthetic profiles of an audience the person
describes and saves them as the person's private persona bank: one of their
twin banks, usable in runs and seen by no one else. A survey run on that bank
asks these profiles instead of the built-in twins.

You use the website's own generator: the same form, the same plan, the same
review and the same bank. A plan you make is the person's plan. The website's
generator offers it to them (Twin library, **Generate synthetic persona bank**),
and they can review and change it there too.

How a bank is made:

1. **The plan** (AI, about 4 minutes). From the description: 10 to 15
   demographics (age, gender and region in every plan, the AI's choice of the
   rest), up to 8 characteristics, the audience restrictions everyone meets,
   which fields depend on which, and the share of each value. The AI searches
   the web to choose the demographics, the characteristics and their categories,
   and lists the sources it referred to. The shares are the AI's predictions,
   made without web search.
2. **The review.** The person (with you, or on the website) checks and changes
   the restrictions, the categories, the shares and the dependencies.
3. **The bank.** Code simulates each profile field by field from the predicted
   shares and weights the profiles to match them; the AI writes a paragraph
   about each person's everyday life.

The generator is open to admins and beta testers, and to everyone once
ExploraTwin opens it: `GET /persona-generator/options` says whether it is open
to the person (`available`).

## What costs what, and who says yes

| Step | Estimated cost | Before it |
|---|---|---|
| Read the options, the plans and a plan | free | nothing |
| Make a plan | about $0.15 | the person's OK in the chat |
| Change the name, shares, impossible combinations or names | free | the person asked for the change |
| Update a plan (other changes, run again) | about $0.01 to $0.05 | the person's OK in the chat |
| Run a stopped plan again | as the step that stopped | the person's OK in the chat |
| Preview the bank's first 5 profiles | about $0.01 | the person's OK in the chat |
| Prepare the bank request | free | nothing |
| The bank itself | about $0.12 for 200 profiles | the person approves it on the website |

`GET /persona-generator/options` (`costs`) and every answer's
`charge_estimate` give the current figures. Charges follow actual use.

- **By default, ask before each paid step.** Tell the person in the chat what
  you will do and its estimate, and wait for their OK.
- **Or ask once for an allowance.** For example: "Making this bank takes a plan
  (about $0.15) and usually an update or two and a preview (about $0.01 to $0.05
  each). May I spend up to $0.40 on these without asking at each step?" With a
  yes, go through the planning steps within that amount. Ask again before you go
  over it, and for each new bank.
- **Report every paid step** in the chat, with its estimate and the total so
  far, whether or not you asked first. Each one also shows on the website, under
  **Agent requests**, with a link to the plan.
- **The bank is always the person's decision on the website.** You prepare a
  request; they approve or decline it there. Never ask them to approve it in the
  chat instead, and never look for another way.
- **Never replace the person's unfinished plan without asking.** Each account
  has one current plan in the website's generator. When a new plan would replace
  one (`409 unfinished_plan`, which names it), ask the person. Send
  `"replace_unfinished": true` only after they agree, or when the unfinished
  plan is one you made for them in this conversation and they asked for a new one.

## The steps

1. **Check it is open**: `GET /persona-generator/options`. `available` must be
   `true`. It also gives the limits, the required demographics, the age groups
   and the costs.
2. **Ask once** about what is still open: who the people are, how many profiles
   (1 to 1,000; the default is 200), demographics the bank must include (with
   categories, and shares if the person has them), the characteristics the
   study needs (with values, if they have them), and whether profiles need
   names. Say what the plan costs, and ask for their OK (or an allowance).
3. **Make the plan**: `POST /persona-generator/plans`:

   ```json
   {"audience": "U.S. adults aged 18 to 60 in the Northeast or Midwest with household incomes of at least $75,000 who may buy a new winter coat.",
    "demographics": [{"name": "Household income", "values": ["$75k–$99k", "$100k–$149k", "$150k–$199k", "$200k+"]},
                     {"name": "Gender", "values": ["Female 52%", "Male 46%", "Non-binary 2%"]}],
    "characteristics": [{"name": "Preferred coat style", "values": ["Puffer", "Parka", "Wool coat", "Down jacket"]},
                        {"name": "Price sensitivity", "values": []}],
    "names": false}
   ```

   Or start from example people: `"examples": ["A nurse in Chicago who works
   night shifts", "A retired teacher in Ohio"]` with `"profiles_per_example": 10`
   (1 to 100; 1 makes exactly the people listed). The answer (`202`) is the plan,
   `status` `queued`, with its `id`, `review_url` and `charge_estimate`.

   What the form takes:

   - **Demographics**: up to 10 of the person's own. Their categories are kept
     as written, and the AI may add one more, such as Other, so everyone fits.
     A share after a category (`Female 52%`) sets the mix (shares for every
     category or none, adding to 100%). An Age or a Region sets the plan's age
     groups or regions; age groups are whole years in order (`18–24`, `25–34`,
     `55 and over`). A state or province comes with the region: name the places
     in the audience description instead. Every simulated person has a
     category, so leave out categories such as "Prefer not to say", "Unknown",
     "Refused" or "Don't know".
   - **Characteristics**: up to 8 in all, counting the AI's own, each with 2 to
     10 values or none (the AI proposes them). Values given are kept, plus at
     most one the AI adds. A demographic is never a characteristic. An opinion
     or attitude can be one, when the person asks for it.
   - **Example people**: up to 50 lines, and 1,000 profiles in all.

4. **Wait for it**: `GET /persona-generator/plans/{id}`, every 20 to 30
   seconds. `status` goes `queued`, `demographics`, `characteristics`,
   `audience`, `map`, `tables`, then `ready` (`step` says it in words; about 4
   minutes). `failed` gives `error`: with the person's OK,
   `POST /persona-generator/plans/{id}/retry` runs it again.
5. **Review it with the person.** Say it in plain words, and give them the
   `review_url`: they can look at it, and change it, on the website. What to
   show, from `plan`:
   - the `name`, and the restrictions in `rule` (`age_min`, `age_max`, and
     `allowed`: the values a field is limited to);
   - the demographics in `demographics.fields` (each `label` and `values`), the
     age groups (`age_bands`) and where people live (the region, and
     `demographics.finer`, such as the state), with the sources
     (`demographics.sources`);
   - the characteristics in `characteristics`, with their sources
     (`characteristic_sources`);
   - the predicted shares in `overall` (by field id, then value, as shares of
     1) and the shares the person set in `targets`;
   - the dependencies in `parents` (each field's parent fields) with the AI's
     reasons in `why`, and the impossible combinations in `rules`;
   - `checks`: `problems` must be empty before a bank; `warnings` are worth
     telling the person.
   The shares that matter to the study deserve the person's look before the bank.
6. **Change it** as the person wants: `POST /persona-generator/plans/{id}/edit`
   with only the parts that change, in the plan's own shapes. Read the plan
   again first: the person may have changed it on the website.
   - Free, at once: `name`, `names`, `targets` (`{"field id": {"value": share}}`;
     `null` for a field gives back the predicted shares) and `rules`
     (`[["field id", "value", "field id", "value"], ...]`, two values no one
     has together).
   - An update (charged, made again): `demographics`
     (`{"fields": [...], "age_bands": [...]}`: `demographics.fields` and
     `age_bands` as the plan has them, changed; a new demographic has no `id`;
     age, gender and region stay), `characteristics` (the whole list, each
     `{"id", "label", "values"}`; a new one has no `id`), `rule` (`allowed`,
     `age_min`, `age_max`), `examples` (the whole list) and `parents`
     (`{"field id": [up to 3 fields earlier in "order"]}`).

   The answer says whether it was `charged`. An update is made like the plan:
   wait for `ready` as in step 4 (about 30 seconds to 2 minutes). A
   `422 edit_has_problems` changes nothing; its `problems` say why.
7. **Preview**: `POST /persona-generator/plans/{id}/preview` with
   `{"profiles": 200, "seed": 7}`. It writes the bank's first profiles (up to
   five), as the bank's will be: with the same number of profiles and seed, the
   bank has these same people, their paragraphs written again. Read them with
   the person: do they fit the audience?
8. **Prepare the bank**: `POST /persona-generator/requests` with an
   `Idempotency-Key` header (a value you make up, reused when you retry):

   ```json
   {"plan_id": "3f1...", "profiles": 200, "seed": 7, "name": "Northern U.S. luxury coat shoppers",
    "reason": "Profiles for the winter coat conjoint."}
   ```

   `reason` is one sentence the person sees as the bank's Description. The
   request freezes a copy of the plan as it is now, with the number of
   profiles, the seed and the name. If the plan changes afterwards, the request
   becomes `stale` (`outcome.stale_reason: "plan_changed"`): prepare a new one
   if the person still wants the bank.
9. **Stop and report**, as plain facts:

   > Prepared the persona bank "{name}" with {profiles} profiles. Estimated
   > cost: {estimate}. The plan, updates and preview cost about {amount}.
   > Review and approve it: {approve_url}

10. **After approval**: `GET /requests/{id}`. While the bank is created,
    `status` is `executing` and `progress` gives `done` and `total`. Check
    every 30 seconds; 200 profiles take about 2 minutes. `succeeded` gives
    `bank_id` (and `outcome.version_id`); `failed` gives `outcome.message`.
11. **Use it in a run**: `GET /twin-banks/{bank_id}` gives its views
    (`default_view_id`) and filter fields. Set `twin_bank` in the run settings
    (`{"bank_id", "version_id", "representation_view_id"}`), count with
    `POST /twins/count`, then prepare the run as usual ([runs.md](runs.md)).
    Ask for no more twins than the bank has profiles.

## What to tell the person

- The shares are the AI's predictions for this audience, not real data. Its
  sources shaped only the demographics, the characteristics and their
  categories.
- A demographic or characteristic they list keeps their categories, plus at
  most one the AI adds (such as Other).
- Descriptions can read alike. Encourage them to read a few profiles, in the
  preview or on the bank's page on the website.
- The bank is private to them. The website's article explains the generator in
  full: https://exploratwin.org/support/generate-persona-bank
