# ai-operating-system-starter

Kortix is open source, and this starter is that idea at its smallest: a small company called acme-co, configured as an AI Operating System. Four agent files, one manifest, one trigger, and an empty memory directory hold the whole thing. Read every file in about ten minutes and you will know where the work gets done and who signs off before it reaches `main`.

The loop it runs is easy to describe and strict about review: a trigger or a person starts a session, an agent works in its own sandbox, and anything it wants to keep arrives as a change request a person merges. Nothing an agent does skips that last step.

## What is in the box

```text
.
├── kortix.yaml                    # the manifest: what exists, and who may use it
├── agents/
│   ├── kortix.md                  # the front door: a general knowledge worker
│   ├── ops-reporter.md            # reads the day's logs and numbers, writes the digest, opens it as a change request
│   ├── support-triage.md          # triages the support inbox, drafts replies, holds sends for a person
│   └── code-fixer.md              # reproduces a reported bug, patches it, runs the checks, opens a change request
├── skills/
│   └── weekly-digest/
│       └── SKILL.md               # the shape of the daily digest, loaded by the ops-reporter
└── memory/                        # empty on purpose: the company brain accumulates here
```

## The manifest

Six files and one empty directory hold the whole company; `kortix.yaml` is the only configuration file among them. Adapted from the Kortix starter, it declares the project, the environment names it may need, the four agents, and one trigger:

```yaml
kortix_version: 2
default_agent: kortix

project:
  name: acme-co
  description: An example company run as an AI operating system.

env:
  required: []
  optional: [STRIPE_API_KEY]

agents:
  kortix:
    file: agents/kortix.md
  ops-reporter:
    file: agents/ops-reporter.md
    connectors: []
  support-triage:
    file: agents/support-triage.md
    connectors: []
  code-fixer:
    file: agents/code-fixer.md
    connectors: []

triggers:
  - slug: daily-ops-digest
    type: cron
    cron: '0 9 * * 1-5'
    prompt: Summarize yesterday's support tickets and open a change request with the digest.
```

The comment at the top of the file carries the governance rule, adapted from the starter manifest. It explains why the three specialists grant no connectors:

> The project REGISTERS what exists (skills, tools, commands, extensions, connectors, secrets). Each AGENT decides: its prompt, model and permission, and which registered things it enables, by name. Nothing applies to every agent by itself, and an agent sees only what it enables.

Read it against the YAML and it stops being abstract. The three specialists are granted no connectors, so none of them can reach an outside app until you grant one. The trigger names no agent, so its sessions boot `default_agent`, and the cron fires at 09:00, Monday through Friday.

## The four agents

**`agents/kortix.md`** is the front door, adapted from the Kortix starter agent. It opens:

> You are a Kortix general knowledge worker for {{projectName}}.

and the next lines set the rule every session in this repo lives by:

> The session you're in is an isolated sandbox — an ephemeral branch of this repo, your own /workspace — so you can install, experiment, and recover freely. Only what you commit + push survives.

`{{projectName}}` comes from `project.name` in the manifest, so here it reads acme-co. The three specialists are shorter, and each does one job:

- **`agents/ops-reporter.md`** reads the day's logs and numbers, writes the daily digest, and opens it as a change request. The digest format lives in `skills/weekly-digest/SKILL.md`, which it loads by name.
- **`agents/support-triage.md`** triages the support inbox, drafts replies, and holds every send for a person to approve.
- **`agents/code-fixer.md`** reproduces a reported bug, patches it, runs the checks, and opens a change request with the fix.

## How work runs

A session runs one agent in its own sandbox. The sandbox clones the repo and works on a new branch whose name is the session ID; when the agent wants to keep something, it commits and opens a change request against `main` ([Kortix docs](https://kortix.com/docs)). Session work reaches `main` through a change request. Merge is default-deny for agents, so a person reviews the diff and merges it, and the next session starts from what `main` now says.

That makes the repo the company's state: configuration in `kortix.yaml`, behavior in `agents/`, know-how in `skills/`, memory in `memory/`. Edit the files and you have edited the company. The review in the middle is what keeps that honest.

## Try it

```bash
curl -fsSL https://kortix.com/install | bash
git clone https://github.com/<your-account>/ai-operating-system-starter.git
cd ai-operating-system-starter
kortix ship
```

Fork the repo first so the GitHub origin points at a repository you own. The full walk, from install to a merged change request, is in [Getting started](docs/getting-started.md).

## The docs

- **[Getting started](docs/getting-started.md)**: install the CLI, ship the repo, start a session, review the change request.
- **[Anatomy of an AI Operating System project](docs/anatomy-of-an-ai-operating-system-project.md)**: every file here, mapped to the part of an operating system it plays.
- **[Customizing the agents](docs/customizing-the-agents.md)**: edit an agent file, add a skill, wire a connector, and gate sends behind approval.

## The system underneath

This repo runs on [Kortix](https://kortix.com), the open-source AI Operating System. The code is on [Kortix on GitHub](https://github.com/kortix-ai/suna). Open source. Read it, fork it, audit it. Run it on Kortix Cloud, in your VPC, or on your own on-prem network. Self-host is free, and [self-hosting a server you own](https://opensourceaioperatingsystem.com/self-hosted-ai-operating-system) is the walkthrough. If you want Kortix next to the rest of the open-source field, opensourceaioperatingsystem.com keeps the comparison in one place, and [an AI operating system for companies](https://opensourceaioperatingsystem.com/ai-operating-system-for-companies) shows what this repo holds once more than one team runs on it.
