# ExploraTwin skills for AI agents

Your AI agent (Claude Code, Codex, Cursor and others) works on
[ExploraTwin](https://exploratwin.org) through your account. It builds surveys
or uploads Qualtrics files, checks and previews them, estimates costs, prepares
runs on digital twins, makes persona banks of synthetic profiles, and downloads
the results. Review and approve prepared runs and banks under
**Agent requests**, at the top right of the website, to start them.

## The quickest way

1. On [exploratwin.org](https://exploratwin.org), click **Connect an agent** at
   the top right and make a key. Agent access must be open to your account.
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
| **When you ask** | Build and change surveys; upload pictures and QSF files; check and preview surveys; count twins; estimate costs; prepare runs and persona banks; read results |
| **After you say yes in the conversation, or within an amount you allowed** | The persona generator's plan (about $0.15), plan updates (about $0.01 to $0.05) and preview (about $0.01), charged directly; each also shows under **Agent requests** |
| **After you say yes in the conversation** | Deleting anything; repairing or rerunning answers; panels; uploading your own data, such as a twin bank |
| **Not available to agents** | Approving, starting or declining a run or a bank; signing in with your password; sending your key anywhere but exploratwin.org |

After preparing a run, the agent reports the survey, the number of twins, the
estimate and the approval link; for a persona bank, its name, the number of
profiles, the estimate and the link. Approved runs and banks are charged as on
the website. A persona bank plan your agent makes is yours: it opens in the
website's persona generator too, where you can review or change it.

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
| `references/persona-generator.md` | The persona generator: synthetic profiles saved as a private persona bank |
| `references/troubleshooting.md` | Error codes and what to do |
| `examples/` | Five example surveys: a plain survey, logic and a loop, a conjoint, an adaptive conjoint, a MaxDiff |

## The survey-format skill

`skills/survey-format` teaches an agent the standard survey format, the JSON
format ExploraTwin and the [surveytwin library](https://github.com/likelili/surveytwin-lib)
run. With it, an agent writes surveys in the format, converts questionnaires from
other tools and documents into it word for word (SurveyMonkey, Google Forms,
Microsoft Forms, REDCap, LimeSurvey, XLSForm, SurveyJS, Typeform, Word, PDF,
text), and checks them. It needs no ExploraTwin account: the surveytwin library
checks a survey on your computer (Python 3.11 or later).

Install it as the `exploratwin` skill, for example in Claude Code:

```bash
cp -R exploratwin-skills/skills/survey-format ~/.claude/skills/
```

| File | What it holds |
|---|---|
| `SKILL.md` | Writing, converting and checking a survey, and the rules of a faithful conversion |
| `references/codebook.md` | Every part of the format, how a twin reads it, common mistakes |
| `references/fields.md` | Every object and field |
| `references/converting.md` | Converting from other tools and documents, tool by tool |
| `references/checking.md` | The three ways to check a survey, and every check code |
| `references/survey-format.schema.json` | The JSON Schema |
| `examples/` | Six example surveys and four worked conversions (Word, Google Forms, REDCap, SurveyMonkey) |

## Keys and privacy

- A key acts as your account, limited to the capabilities chosen when it was
  made. Revoke it under **Connect an agent**.
- The skill tells your agent to keep the key in an environment variable or a
  private file, never in a project folder or a repository.
- Whatever your agent receives with the key (your surveys, settings and
  results) is then handled under your agent's own terms and providers.

## Versions

The skill's version is in `SKILL.md` (1.2.0). The same files are served at
[exploratwin.org/agent](https://exploratwin.org/agent/start.md), and
`GET /api/agent/v1/me` reports the current `skill_version`. When it changes,
the skill tells your agent to fetch the files again. Releases here follow the
website. The survey-format skill has its own version, in its `SKILL.md` (1.0.0).

## Help

- [Connect an AI agent to ExploraTwin](https://exploratwin.org/support/connect-an-agent):
  connecting, what your agent can do, and when it asks for your approval.
- More guides: [exploratwin.org/support](https://exploratwin.org/support)

## License

[Apache License 2.0](LICENSE)
