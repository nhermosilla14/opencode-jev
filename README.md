# opencode-jev

An agent skill that teaches coding agents how to build with **Jev**, TypeSafe AI's System One
decision model, through an **[OpenCode Go](https://opencode.ai/docs/go)** subscription.

Jev doesn't generate text. It answers typed questions (yes/no, multiple choice, rubric score) about
a `state` you supply and returns calibrated probabilities that your code turns into decisions. This
skill covers how to design the state, write the questions, read the probabilities, and connect to
the OpenCode Go endpoint with `OPENCODE_API_KEY`.

> **Unofficial.** This is an independent adaptation and is not affiliated with or endorsed by
> TypeSafe AI or OpenCode. See [Credits](#credits-and-license).

## Why this exists

TypeSafe publishes an excellent integration skill, but it assumes TypeSafe's own API
(`api.typesafe.ai`), a `TYPESAFE_API_KEY` and the `jev-latest` alias. If you reach Jev through OpenCode Go instead,
those details are wrong. This fork keeps the guidance and replaces the connection details:

| | TypeSafe's skill | This skill |
| --- | --- | --- |
| Endpoint | `https://api.typesafe.ai/v1/systemone` | `https://opencode.ai/inference/go/systemone/v1/systemone` |
| Key | `TYPESAFE_API_KEY` | `OPENCODE_API_KEY` (OpenCode Go) |
| Model | `jev-latest` | `jev-1.14` |
| Billing | TypeSafe account | Your OpenCode Go allowance |

The request and response bodies are identical.

## Requirements

- An active OpenCode Go subscription (Go, Go Plus or Go Teams), and an API key from the
  [OpenCode Console](https://opencode.ai/console). The key must belong to the subscriber or one of
  its service accounts, otherwise the API returns HTTP 403.
- Keep the key in a git-ignored `.env` or your shell environment. Never commit it.

## Install

### With `npx skills` (OpenCode, Claude Code, Codex, Cursor and others)

```bash
npx skills add nhermosilla14/opencode-jev
# global, for one agent:
npx skills add nhermosilla14/opencode-jev -g -a claude-code
```

### Claude Code plugin marketplace

```
/plugin marketplace add nhermosilla14/opencode-jev
/plugin install opencode-jev@opencode-jev
```

### Manual

Copy `skills/opencode-jev/` into your agent's skills folder, for example `~/.claude/skills/`,
`~/.config/opencode/skills/` or `~/.agents/skills/`.

## Quick check that your key works

```bash
curl https://opencode.ai/inference/go/systemone/v1/systemone \
  -H "Authorization: Bearer $OPENCODE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "jev-1.14",
    "state": "My payments have failed for three days and I am losing sales.",
    "questions": {
      "is_urgent": { "type": "noul", "instructions": "Does this request require urgent attention?" }
    }
  }'
```

A healthy response looks like `{"model":"jev-1.14","answers":{"is_urgent":{"type":"noul","noul":0.93}},...}`
and its `usage.opencode.billing_source` is `go`.

Check how much of your allowance you've used with
`GET https://opencode.ai/inference/go/v1/usage` and the same key.

## Good to know

- Jev is billed at list price ($0.042 per 1M input tokens, output free) against the shared Go
  allowance. It's cheap, but not unlimited.
- Decision models never appear in the OpenCode `/models` picker, because they don't generate text.
  Call the endpoint directly.
- No `x-opencode-session` header is needed for decision models.
- `gemini-decision-1` and `gpt-6-luna-decision` use the same endpoint and request format.
- Whether TypeSafe's image and long-context headers work through the gateway is unverified.
- Prices, models and limits can change. See the
  [OpenCode Go docs](https://opencode.ai/docs/go#decision-models).

## Repository layout

```
.claude-plugin/plugin.json        Claude Code plugin manifest
.claude-plugin/marketplace.json   Claude Code marketplace entry
skills/opencode-jev/SKILL.md      The skill (discovered by npx skills)
LICENSE
```

## Credits and license

The body of the skill is derived from TypeSafe AI's
[`typesafe-ai` skill](https://docs.typesafe.ai), published under the MIT License. The OpenCode Go
setup section and the adaptations are by Nicolás Hermosilla. TypeSafe and Jev are TypeSafe AI's
names, and OpenCode is OpenCode's.

Licensed under the [MIT License](LICENSE), with the original TypeSafe AI copyright notice retained.
