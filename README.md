# ExploraTwin skills for AI agents

Your AI agent (Claude Code, Codex, Cursor and others) works on
[ExploraTwin](https://exploratwin.org) through your account. It builds surveys
or uploads Qualtrics files, checks and previews them, estimates costs, prepares
runs on digital twins, and downloads the results. Review and approve prepared
runs under **Settings → Agent requests** to start them.

## The quickest way

1. On [exploratwin.org](https://exploratwin.org), open **Settings → Connect an
   agent** and make a key. Agent access must be open to your account.
2. Paste this line into your agent:

   > Fetch https://exploratwin.org/agent/start.md and follow it to connect to ExploraTwin for me.

The agent reads the instructions, asks you for the key, and checks it.

## Installing the skill

If your agent keeps skills as folders, install `skills/exploratwin` once:

- **Claude Code:**

  ```bash
  git clone https://github.com/likelili/exploratwin-skills
  cp -R exploratwin-skills/skills/exploratwin ~/.claude/skills/
  ```

  For a single project, use `.claude/skills/` in that project instead.
- **Codex and other agents that read `SKILL.md` folders:** put the folder where
  your agent keeps its skills. `agents/openai.yaml` gives Codex the skill's
  name and a default prompt.
- **Any other agent:** give it `skills/exploratwin/start.md` to read.

## What your agent may do

| | |
|---|---|
| **When you ask** | Build and change surveys; upload pictures and QSF files; check and preview surveys; count twins; estimate costs; prepare runs; read results |
| **After you say yes in the conversation** | Deleting anything; repairing or rerunning answers; panels; uploading your own data, such as a twin bank |
| **Not available to agents** | Approving, starting or declining a run; signing in with your password; sending your key anywhere but exploratwin.org |

After preparing a run, the agent reports the survey, the number of twins, the
estimate and the approval link. Approved runs are charged like any run on the
website.

## What is in the skill

| File | What it holds |
|---|---|
| `start.md` | Connecting: the key, where to keep it, how to check it |
| `SKILL.md` | What the agent may do, what it asks you first, and the steps |
| `references/api.md` | Every action of the agent API, with its inputs, answers and errors |
| `references/survey-format.md`, `survey-format.schema.json` | The standard survey format |
| `references/qsf.md` | Qualtrics files |
| `references/runs.md`, `approvals.md`, `results.md` | Run settings, approvals and result files |
| `references/twin-banks.md` | Built-in twins, twin banks and your own data |
| `references/troubleshooting.md` | Error codes and what to do |
| `examples/` | Five example surveys: a plain survey, logic and a loop, a conjoint, an adaptive conjoint, a MaxDiff |

## Keys and privacy

- A key acts as your account, limited to the capabilities chosen when it was
  made. Revoke it under **Settings → Connect an agent**.
- The skill tells your agent to keep the key in an environment variable or a
  private file, never in a project folder or a repository.
- Whatever your agent receives with the key (your surveys, settings and
  results) is then handled under your agent's own terms and providers.

## Versions

The skill's version is in `SKILL.md` (1.0.1). The same files are served at
[exploratwin.org/agent](https://exploratwin.org/agent/start.md), and
`GET /api/agent/v1/me` reports the current `skill_version`. When it changes,
the skill tells your agent to fetch the files again. Releases here follow the
website.

## Help

[exploratwin.org/support](https://exploratwin.org/support)

## License

[Apache License 2.0](LICENSE)
