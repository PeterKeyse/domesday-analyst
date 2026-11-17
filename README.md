# Domesday Analyst

A conversational data analyst agent built with the [Mistral Agents SDK](https://docs.mistral.ai/studio/agents/).
Give it a link to a CSV file, ask questions in plain language, and it answers with real
numbers computed from your data — describing columns, filtering rows, and remembering the
dataset and every previous finding for the rest of the conversation.

Ships with the Open Domesday manors dataset (`data/DD_Manors.csv`, from
[opendomesday.org](https://opendomesday.org)): 7,876 manors recorded in 1086 across 33
counties, with lords in 1066 and 1086, tenants-in-chief, households, livestock, plough
teams, and annual values before and after the Norman Conquest.

## How it works

- `main.py` defines the agent: model `mistral-large-4`, a `MistralSandbox` with full
  network access, and a setup script that seeds the sandbox with the Domesday CSV and
  pandas on session start.
- The agent runs pandas over the CSV in its sandbox for every quantitative answer and
  keeps a `MEMORY.md` in the workspace with the dataset profile and findings so far, so
  follow-ups never require re-explaining.
- Sessions are durable: a deployed session survives idle periods (4h idle / 24h lifetime
  by default) and can be reopened by session id.

## Local development

```bash
uv sync
export MISTRAL_API_KEY="<your key>"
uv run agents chat          # chat with the local agent
uv run agents info          # inspect the agent definition
```

## Deploy (Mistral-managed)

Requires a GitHub repository for this project, the GitHub connector connected in
[Mistral Studio](https://console.mistral.ai/build/connectors), and a `MISTRAL_API_KEY`
whose user has Workflows access and the `cloud_user` role.

```bash
git add --all && git commit -m "Domesday analyst agent" && git push -u origin HEAD
uv run --env-file .env agents deploy
```

After deploying, chat with it in Vibe Code Web at the URL the deploy prints
(`https://chat.mistral.ai/managed-agents/domesday-analyst?source=agents_api`), or from
any program:

```python
import asyncio
from mistralai import agents

async def main() -> None:
    agent = agents.Agent.load_from(agent_name="domesday-analyst")
    result = await agent.run("How much did Yorkshire's manors fall in value after 1066?")
    print(result.text)

asyncio.run(main())
```

## Things to ask it

- "Describe the dataset" — columns, coverage, and data-quality notes.
- "Which lords held the most land in 1066, and who held it in 1086?" — the transfer of
  estates from Anglo-Saxon lords to William's followers.
- "How did manor values change between 1066 and 1086, county by county?" — including the
  wasted manors of the Harrying of the North.
- "Where were King William's own manors concentrated?" — geographic distribution via
  county and hundred.
- "What happened to the livestock?" — sheep, pig and cattle counts in 1066 vs 1086 on the
  manors that recorded both.
