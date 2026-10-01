# MongoDB Atlas Agent Engine Getting Started

**Author:** Carlos Biagolini-Jr.

**LinkedIn:** [https://www.linkedin.com/in/biagolini/](https://www.linkedin.com/in/biagolini/)

**Medium:** [https://medium.com/@biagolini](https://medium.com/@biagolini)

A minimal AI agent for [MongoDB Atlas Agent Engine](https://www.mongodb.com/products/platform/atlas-agent-engine), created with the `agentengine create` command from the official Hello World Agent template (LangGraph + Python). The agent uses Claude on Amazon Bedrock through the Anthropic-compatible endpoint, runs locally with `agentengine dev up`, and is deployed to Atlas Agent Engine with execution state stored in an Atlas Flex cluster.

Atlas Agent Engine is in public preview. Commands and generated files can change between CLI versions. This project was generated with `agentengine` CLI version 0.1.118.

## Repository Structure

```none
.
├── project-config.yaml         # Project level settings (memory service, unused while memory is off)
└── agents/
    └── demo-agent/             # Agent workspace: run every agentengine command from here
        ├── agent.yaml          # How the platform runs the agent (entrypoint, sandboxes, egress)
        ├── dev.yaml            # Local development settings (service ports)
        ├── env.example         # Template for the local .env file
        ├── pyproject.toml
        └── src/agent_demo_agent/
            ├── main.py         # App instance and LangGraph graph
            ├── llm.py          # Claude on Amazon Bedrock (Anthropic-compatible endpoint)
            ├── tools.py        # Agent tools
            ├── state.py
            └── system_message.py
```

The `.agents/` and `.claude/` folders inside the agent workspace contain skills that `agentengine create` installs for AI coding assistants.

## Prerequisites

- A MongoDB Atlas account and a project where you have the `Project Owner` role
- An Atlas Flex cluster (Free clusters are not supported by Atlas Agent Engine)
- The `agentengine` CLI, installed from the Atlas Agent Engine **Project Overview** page
- Docker Desktop (or Docker Engine with Docker Compose v2) and Python 3.11 or later
- An AWS account with access to a Claude model on Amazon Bedrock in `us-east-1`, and an Amazon Bedrock long-term API key

## Run Locally

```bash
cd agents/demo-agent
cp env.example .env
```

Set `LLM_API_KEY` in `.env` to your Amazon Bedrock API key. Leave `MONGODB_URI` and `VOYAGE_API_KEY` empty: the local stack starts its own MongoDB container, and memory is off. Never commit the `.env` file.

If you use a different model or Region, update `MODEL` and `base_url` in `src/agent_demo_agent/llm.py`, and the egress entry in `agent.yaml`.

```bash
agentengine dev up
```

Open the playground UI printed by the CLI (usually `http://localhost:3000`), or call the local endpoint:

```bash
curl -X POST 'http://localhost:3000/invoke' \
  -H 'Content-Type: application/json' \
  -d '{"message": "Hello"}'
```

Stop the stack with `agentengine dev stop`.

## Deploy to Atlas Agent Engine

From `agents/demo-agent`:

```bash
agentengine init
agentengine atlas setup
agentengine secret set LLM_API_KEY
agentengine deploy --auto
agentengine invoke --session session-1 "Hello! Who are you?"
```

`agentengine atlas setup` links the workspace to your Atlas cluster, creates a database user, stores `MONGODB_URI` as a platform secret, and adds the Agent Engine IP addresses to the Atlas IP access list. `agentengine secret set LLM_API_KEY` stores the Bedrock API key as a platform secret, because the deployed agent does not read the local `.env` file.

## Cleanup

To avoid charges, remove the resources when you finish:

- Delete the workspace with `agentengine workspace list` and `agentengine workspace delete <workspace-id> --yes`
- Delete the platform secrets with `agentengine secret delete LLM_API_KEY --yes` and `agentengine secret delete MONGODB_URI --yes`
- Terminate the Atlas Flex cluster and remove the database user and IP access list entries created by the setup
- Delete the Amazon Bedrock API key and the IAM user created with it

## References

- [Atlas Agent Engine documentation](https://www.mongodb.com/docs/agentengine/)
- [Get Started with the Atlas Agent Engine](https://www.mongodb.com/docs/agentengine/get-started/)
- [Set Up Atlas Resources](https://www.mongodb.com/docs/agentengine/deploy/atlas-setup/)
- [Inference using Anthropic Messages API on Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-messages-api.html)
- [MongoDB projects index](https://github.com/biagolini/MongoDB)
