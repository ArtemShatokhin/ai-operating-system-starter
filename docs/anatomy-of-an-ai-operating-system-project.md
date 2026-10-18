# Anatomy of an AI Operating System project

An AI Operating System project is a git repo with `kortix.yaml` at its root, and agents, skills, memory, tools, triggers and connectors are files in it ([Kortix docs](https://kortix.com/docs)). This repo is one: acme-co, a small company, held in four agent files, one skill, one trigger, and an empty memory directory. Kortix is open source and so is this starter, so every part of the system here is a file a person can open.

The [Kortix docs](https://kortix.com/docs) map each part of an operating system onto a Kortix feature. Filled in with this repo's actual files, the map reads:

| Operating system part | In this repo |
| --- | --- |
| File system | the repo itself |
| Programs | `agents/*.md` and `skills/weekly-digest/SKILL.md` |
| Scheduler | the `daily-ops-digest` trigger |
| Processes | sessions, one sandbox and one branch each |
| Drivers | connectors, granted per agent |
| Updates | change requests, merged by a person |

## The repo is the file system

Everything the company is lives in the repo as text. `kortix.yaml` holds the configuration, `agents/` holds behavior, `skills/` holds know-how, and `memory/` holds what the agents learn along the way. Git keeps every version of all of it, so the company's history reads as commits. You can grep the whole company, which is the point of keeping it in files instead of in a settings screen.

## `kortix.yaml` is the manifest

`kortix.yaml` speaks for the whole project. It names the project (`acme-co`), the default agent, the environment names the runtime may need (`STRIPE_API_KEY`, declared optional), the four agents, and the trigger. Everything in it is governance: what exists, and what each agent may reach. Agent behavior lives elsewhere, in the agent files. The comment at the top of the manifest says the rule in one paragraph:

> The project REGISTERS what exists (skills, tools, commands, extensions, connectors, secrets). Each AGENT decides: its prompt, model and permission, and which registered things it enables, by name. Nothing applies to every agent by itself, and an agent sees only what it enables.

The YAML follows the rule. The three specialist entries carry `connectors: []`, so each starts with no connectors at all; in a v2 manifest an omitted grant resolves to none, as the starter manifest notes. The trigger carries no `agent:` key, so its sessions boot `default_agent`.

## `agents/` holds the programs

Each agent is a single markdown file with two parts: a short frontmatter that names it and scopes it, and a body that is the prompt. The frontmatter of the front door carries `mode: primary` and `permission: allow`, and the three specialists follow the same shape. `kortix.md`, the front door, opens:

> You are a Kortix general knowledge worker for {{projectName}}.

`{{projectName}}` comes from `project.name` in the manifest, so it reads acme-co here. The lines right after it give every session its ground rules:

> The session you're in is an isolated sandbox — an ephemeral branch of this repo, your own /workspace — so you can install, experiment, and recover freely. Only what you commit + push survives.

The three specialists are shorter, and each is scoped on purpose. All of them currently grant no connectors:

- `ops-reporter.md` reads the day's logs and numbers, writes the daily digest, and opens it as a change request.
- `support-triage.md` triages the support inbox, drafts replies, and holds every send for a person.
- `code-fixer.md` reproduces a reported bug, patches it, runs the checks, and opens a change request.

One generalist at the door and three specialists behind it is a shape you can copy. Nothing in the manifest prevents a fifth file when the company needs one.

## `skills/weekly-digest/` holds shared know-how

A skill encodes how the company does one job. This one is a single `SKILL.md` under `skills/weekly-digest/`, and it exists so the digest has one format no matter who writes it. Skills sit at the repo root, where every harness loads them, and an agent picks one up by name when its prompt calls for it, the way `ops-reporter.md` does for the digest.

## `memory/` is the company brain

The directory ships empty, and that is the honest starting state: this company has not done any work yet. As sessions run, what they learn that is worth keeping goes into `memory/` as plain files, and later sessions read those files before they start. The Kortix starter calls this directory the project brain. In operating-system terms, it is the persistent state that survives any one process ending.

## Triggers are the scheduler

One trigger ships with this project. It is short enough to read in full:

```yaml
- slug: daily-ops-digest
  type: cron
  cron: '0 9 * * 1-5'
  prompt: Summarize yesterday's support tickets and open a change request with the digest.
```

At 09:00, Monday through Friday, it starts a session with that prompt and no person present. The session runs the default agent, because the trigger names no `agent:`. The agent reads what yesterday left behind, writes the digest, and opens it as a change request. The scheduler is only YAML: edit the cron line and the company's morning moves with it.

## Sessions are processes

A session is one unit of agent work. One isolated sandbox per session. Each session has its own isolated machine and branch, so two agents never collide mid-task; each proposes its changes separately. The sandbox clones the repo at boot and works on a branch named after the session ID ([Kortix docs](https://kortix.com/docs)). When the work is done, the branch and its commits wait in the change request for review.

## Change requests are updates

Nothing an agent does reaches the company until a person says so. Session work reaches `main` through a change request. Merge is default-deny for agents, and a person with the merge permission (`project.gitops.merge`) reviews the diff and merges it. The next session then boots from the updated repo. Updates ship through review, and the system never changed underneath the team without a record of who changed it.

## Reading the whole system in ten minutes

Start with `kortix.yaml`, which tells you what exists. Then `agents/kortix.md` for the ground rules every session shares. Then the three specialists in whichever order matters to you, then `skills/weekly-digest/SKILL.md`, then a glance at the empty `memory/` on the way past. Ten minutes covers it, which is the whole argument for keeping a company in files.
