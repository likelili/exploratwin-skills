# Connect to ExploraTwin

You are an AI agent, and the person you work for wants you to use ExploraTwin
(https://exploratwin.org) for them. ExploraTwin runs surveys on digital twins:
AI participants built from real people's survey answers. You can build surveys
or upload Qualtrics files, check them, estimate what a run costs, and prepare
runs. **You never start a run.** Every run you prepare waits on the website
until the person approves it; then you can follow it and fetch the results.

Do these steps once.

## 1. Ask for the key

Ask the person for their ExploraTwin API key. They make one on
https://exploratwin.org under **Settings → Connect an agent**; it starts with
`etw_`. Never ask for their password: the key is the only sign-in you use.

## 2. Keep the key safe

Store it where it never reaches a shared place:

- in the environment variable `EXPLORATWIN_API_KEY`, or
- in `~/.config/exploratwin/credentials.json` as `{"api_key": "etw_..."}`, readable
  only by the person (`chmod 600`).

Never write it into a project folder, a code repository, a notebook or a log.
Never print it in full; refer to it by its first characters. Send it only to
`https://exploratwin.org`.

## 3. Check it

```bash
curl -s https://exploratwin.org/api/agent/v1/me \
  -H "Authorization: Bearer $EXPLORATWIN_API_KEY"
```

A good key returns the account, what the key may do (`capabilities`), when it
expires, and `skill_version`. An error says why (`error.code`, `error.hint`);
see [troubleshooting](references/troubleshooting.md).

## 4. Get the skill

Read [SKILL.md](SKILL.md) and follow it from now on. If your tool keeps skills
(Claude Code, Codex and others), save these files together as one skill
folder named `exploratwin`, keeping their paths:

- https://exploratwin.org/agent/SKILL.md
- https://exploratwin.org/agent/references/api.md
- https://exploratwin.org/agent/references/survey-format.md
- https://exploratwin.org/agent/references/survey-format.schema.json
- https://exploratwin.org/agent/references/qsf.md
- https://exploratwin.org/agent/references/runs.md
- https://exploratwin.org/agent/references/results.md
- https://exploratwin.org/agent/references/twin-banks.md
- https://exploratwin.org/agent/references/approvals.md
- https://exploratwin.org/agent/references/troubleshooting.md
- https://exploratwin.org/agent/examples/plain-survey.json
- https://exploratwin.org/agent/examples/logic-and-loop.json
- https://exploratwin.org/agent/examples/conjoint.json
- https://exploratwin.org/agent/examples/adaptive-conjoint.json
- https://exploratwin.org/agent/examples/maxdiff.json
- https://exploratwin.org/agent/agents/openai.yaml (for Codex)

When `GET /me` reports a `skill_version` other than the one in your copy of
SKILL.md, fetch the files again.

## 5. Tell the person

Say that you are connected, to which account, and what the key lets you do.
Remind them that runs you prepare wait for their approval under
**Settings → Agent requests** on the website (they also get an email), and
that you will ask them before anything that deletes or changes their earlier
work.
