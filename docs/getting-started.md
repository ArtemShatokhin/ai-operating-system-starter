# Getting started

Install the CLI, fork and clone this repo, ship it, start a session, and review the change request the agent opens. Kortix is open source, and so is every file this walk touches. You can read the CLI, the manifest, the agent files, and the diff inside each change request. Nothing asks you to trust a setting you cannot open.

## 1. Install the CLI and sign in

```bash
curl -fsSL https://kortix.com/install | bash
kortix login
```

The install script supports macOS and Linux on x86_64 and arm64. On Windows, run it inside WSL. `kortix login` opens your browser. Approve the CLI there, and the token comes back on a local callback ([Quickstart](https://kortix.com/docs/quickstart)).

## 2. Fork and clone the repo

Fork `ArtemShatokhin/ai-operating-system-starter` on GitHub first. Then clone your fork:

```bash
git clone https://github.com/<your-account>/ai-operating-system-starter.git
cd ai-operating-system-starter
```

The fork matters because of what `kortix ship` does on its first run. With a GitHub origin, it links that repository to the project. The sessions you start later then push their branches to a repo you own. Cloning this repo directly also works. The origin then points here, which is somebody else's repository. If you skip the fork, remove the git origin before shipping. `kortix ship` then creates a Kortix-managed repository for the project.

## 3. Ship it

```bash
kortix ship
```

`kortix ship` does the whole first pass ([Quickstart](https://kortix.com/docs/quickstart)). It validates `kortix.yaml`, commits, asks for missing secrets, pushes the current branch, and asks you to connect declared connectors. Nothing sits under `env.required`, so there is nothing to fill in before a session starts. The one optional name, `STRIPE_API_KEY`, can wait until an agent actually needs Stripe. Set it then: `kortix secrets set STRIPE_API_KEY=<the value>`. The repo keeps the name; the secrets manager keeps the value.

After the first ship, plain `git push` and `git pull` work against the linked repository. `kortix validate` checks the manifest without shipping anything:

```bash
kortix validate
```

## 4. Start a session

```bash
kortix sessions new --prompt "Summarize yesterday's support tickets and open the digest as a change request"
```

A session runs one agent in its own sandbox. The sandbox clones the repo and works on a new branch named after the session ID ([Kortix docs](https://kortix.com/docs)). With no `--agent`, the session runs the project's default agent, which here is `kortix`, the front door. To pin a specialist instead:

```bash
kortix sessions new --agent ops-reporter --prompt "Write today's digest and open it as a change request"
```

While a session runs, a few commands cover the everyday work. Commands that need a session take the ID from `kortix sessions ls`:

| Task | Command |
| --- | --- |
| List sessions | `kortix sessions ls` |
| Talk to the most recent one | `kortix chat` |
| Attach the terminal UI | `kortix connect <session_id>` |
| See what automated runs did | `kortix sessions ls --automated` |

The `daily-ops-digest` trigger starts sessions on its own. It fires at 09:00 on weekdays, with its prompt and no person present. Its sessions show up under `--automated` like any other run a trigger started.

## 5. Review the change request

When the agent has work worth keeping, it opens a change request against `main`. Session work reaches `main` through a change request. Merge is default-deny for agents, so the review step belongs to a person ([Kortix docs](https://kortix.com/docs)).

```bash
kortix cr ls        # open change requests
kortix cr diff 1    # the unified diff
kortix cr merge 1   # merge into main
```

The web app's Review page does the same three steps with buttons. To send the work back instead of merging:

```bash
kortix cr request-changes 1 --message "Hold the digest until the numbers include refunds"
```

The note goes to the change request's session and wakes it if it sleeps. After a merge, `git pull` brings the work into your clone. The next session then starts from the updated repo.

## 6. Check the setup

```bash
kortix doctor
```

`kortix doctor` checks your login and the project. It then starts a throwaway session, waits for the agent's reply, and deletes the session again. An exit code of 0 means every check passed.

## Where to go next

- [Anatomy of an AI Operating System project](anatomy-of-an-ai-operating-system-project.md) explains what each file in the repo does, before you change any of them.
- [Customizing the agents](customizing-the-agents.md) picks up where this walk stops: editing prompts, adding skills, wiring connectors, and tightening the gates on sends.

The [Kortix docs](https://kortix.com/docs) cover the rest. Models, secrets, accounts, and the full `kortix.yaml` reference all live there.
