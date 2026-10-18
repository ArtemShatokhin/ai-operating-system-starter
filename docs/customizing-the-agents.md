# Customizing the agents

Every agent in this project is a markdown file you can read and edit, and every grant that scopes an agent is a line in `kortix.yaml`. Four changes cover most of what you will want to do: edit an agent, add a skill, wire a connector, and gate a send.

Kortix is open source, and so is this starter. Every change here is a text edit in a repo you own.

## Where each thing lives

| What | Lives in | Who changes it |
| --- | --- | --- |
| An agent's prompt, model, and permission | `agents/<name>.md`, frontmatter plus body | you, by commit; or the agent, by change request |
| What an agent may reach: connectors, secrets, permissions, skills | the agent's entry in `kortix.yaml` | you, by commit; or the agent, by change request |
| Shared know-how | `skills/<name>/SKILL.md` | you, by commit |
| Credential values | the Kortix secrets manager | `kortix secrets set KEY=<value>` |
| Connector connections | the project, connected once and granted per agent | `kortix ship` asks, or the web app |

The governance comment at the top of the manifest is the rule behind that table. The project registers what exists; each agent decides which registered things it enables, by name. Nothing applies to every agent by itself.

## Edit an agent file

Open the file you want to change. The frontmatter is short, and this starter uses three keys in it: a one-line `description`, `mode`, and `permission`:

```yaml
---
description: "one line that says what this agent is for"
mode: primary
permission: allow
---
```

The body under the frontmatter is the prompt, and the prompt is where this agent's behavior is defined. Everything the support-triage agent does, including holding sends, is written there in plain English. To make it stricter, add a line at the point where the rules live:

```markdown
- Never send two replies for the same ticket. If a second message arrives, add it to the draft and wait.
```

Commit and push the file (`git push` works after the first ship), and the next session boots with the new instructions. An agent can also do this editing itself. An agent can edit its own configuration on its session branch and propose the change. A person approves it. Ask the front door to tighten the support-triage prompt, and review the diff like any other change request. The files that define the agents are files the agents can propose changes to, with a person holding the merge.

## Add a skill

Mirror what the repo already has. Create the folder and write the file:

```text
skills/
└── expense-policy/
    └── SKILL.md
```

The skill is now registered with the project, because skills live at the repo root, where every harness loads them. To make an agent use it, name the skill in that agent's prompt at the moment it matters, the way `ops-reporter.md` names `weekly-digest` for the digest. If your project scopes skills per agent, the grant sits in the agent's entry in `kortix.yaml`, next to the connector grants the specialists already show. The full grant reference is in the [Kortix docs](https://kortix.com/docs).

## Wire a connector

The three specialists start with `connectors: []`, which grants none of them anything. A connector reaches an agent in two steps:

1. Connect the connector to the project once. `kortix ship` asks about declared connectors on each run, and the web app connects the rest.
2. Add the connector's name to the agent's list in `kortix.yaml`:

```yaml
  support-triage:
    file: agents/support-triage.md
    connectors: [gmail]
```

The names come from the connector catalog. Grant the ones the agent's job needs and leave the rest out; v2 manifests are deny-by-default, so an empty list means that agent starts with nothing.

Credentials never live in the repo. Connector credentials are brokered server-side and never enter the machine. For plain secrets, declare the name under `env:` and set the value in the secrets manager with `kortix secrets set KEY=<value>`. Names under `env.required` must be set before a session starts; names under `env.optional`, like `STRIPE_API_KEY` here, can wait until an agent needs them.

## Gate sends behind approval

The starter already gates its riskiest action, sending support replies, at every layer it controls. Keep all of them when you customize:

1. **The prompt.** `support-triage.md` holds every send for a person. That rule exists because the file says so. It is the cheapest gate to add and the easiest to read, and it holds because the agent follows its own file.
2. **The connector policy.** Connector actions are `always_run`, `require_approval`, or `block`, shown as Allow, Ask, Block. Approval holds the call; the agent's turn pauses and resumes. Set Ask on the actions that leave the company, and Block on the ones you never want attempted.
3. **The change request.** Session work reaches `main` through a change request. Merge is default-deny for agents. A send happens outside the repo, so the first two gates are what stand in front of it; the change request is what keeps drafts and patches from becoming company state without a person reading them.

Note the default posture the starter ships in. The support inbox has no connector granted at all, so nothing can send until you grant one, and that is worth keeping until the prompt rule and the connector policy are both where you want them.

## Pin the digest to its specialist

The `daily-ops-digest` trigger boots the default agent because it names none. To run it on the agent built for the job, add the key:

```yaml
  - slug: daily-ops-digest
    type: cron
    agent: ops-reporter
    cron: '0 9 * * 1-5'
    prompt: Summarize yesterday's support tickets and open a change request with the digest.
```

`agent:` must name an agent declared in the same manifest. The same key works on any trigger you add later.

## Where to read more

The [Kortix docs](https://kortix.com/docs) hold the full reference for the manifest, agents, skills, connectors, and permissions. [Anatomy of an AI Operating System project](anatomy-of-an-ai-operating-system-project.md) explains why each file sits where it sits, and [Getting started](getting-started.md) runs the loop end to end.
