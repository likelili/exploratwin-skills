# ExploraTwin skills for AI agents

Let your own AI agent (Claude Code, Codex, Cursor and others) work on
[ExploraTwin](https://exploratwin.org) for you. It can build surveys or upload
Qualtrics files, check and preview them, estimate costs, prepare runs on
digital twins, and download the results.

**Your agent never starts a run on its own.** Each run it prepares waits on the
website until you approve it.

## The quickest way

1. On [exploratwin.org](https://exploratwin.org), open **Settings → Connect an
   agent** and make a key. Agent access must be open to your account.
2. Paste this line into your agent:

   > Fetch https://exploratwin.org/agent/start.md and follow it to connect to ExploraTwin for me.

The agent reads the instructions, asks you for the key, and checks it. Nothing
else to install.

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
| **Goes ahead when you ask** | Build and change surveys; upload pictures and QSF files; check and preview surveys; count twins; estimate costs; prepare runs (they wait for your approval); read results |
| **Asks you first, and waits for a clear yes** | Deleting anything; repairing or rerunning answers; panels; uploading your own data, such as a twin bank |
| **Never** | Approves, starts or declines a run; asks for your password; sends your key anywhere but exploratwin.org |

Runs you approve are charged as any run on the website. The agent shows you
the estimate and the approval link before anything runs.

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

- A key belongs to your account and acts as you, within what you allowed when
  you made it. Revoke it on the website at any time.
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
