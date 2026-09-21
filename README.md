# Plans, AI agents and orchestration scripts for my-handicapped-pet.io

This repository is the local coordination repo for planning and
agent-orchestration experiments around
[my-handicapped-pet.io](https://my-handicapped-pet.io).

## Running agents locally

The first milestone is intentionally manual: run Junie CLI from this
repository and let the local agent files guide the session.

This repo also provides project-scoped Junie slash commands, so each
agent can be started from an interactive Junie session with one command.

### Directory layout

- `.junie/plans/` stores canonical human-facing planning artifacts.
- `.junie/tracking/` stores internal Junie task trackers, delivery-step
  status files, and execution notes.
- `.junie/agents/`, `.junie/commands/`, `.junie/templates/`, and
  `.junie/workflows/` store reusable agent instructions and workflow
  contracts.

### Prerequisites

- Create a dedicated local OS user for agent runs, as described below.
- Install Junie CLI and make sure `junie --version` works.
- Prepare dedicated GitHub and Junie tokens for the agent user, as
  described below.
- Install GitHub CLI (`gh`) if the session needs to create or update
  issues or pull requests.
- Start every agent session from this repository root.

### Dedicated agent OS user

Run interactive and headless agents under a separate local user instead
of your normal desktop user. This keeps Git, SSH, GitHub CLI, Junie
credentials, sessions, and auto-approval settings isolated from your
personal account.

Recommended properties for the agent user:

- Use a non-admin user, for example `sadist-agent`.
- Do not add the user to `sudo`, `wheel`, `docker`, or other powerful
  groups unless a specific task really requires it.
- Do not copy your personal `~/.ssh/`, `~/.gitconfig`, `~/.config/gh/`,
  or `~/.junie/` into the agent user's home directory.
- Give the user its own Git SSH key or GitHub deploy key for this repo.
- Give the user its own GitHub token only if issue or pull request
  operations are needed.
- Give the user its own Junie token or BYOK credentials for local agent
  runs.
- Keep Junie command allowlists and brave-mode settings under that
  user's `~/.junie/`, not under your personal account.

Create the user on Linux:

```sh
sudo useradd --create-home --shell /bin/bash sadist-agent
sudo passwd --lock sadist-agent
```

The locked password prevents direct password login. You can still start
a local shell as the user through `sudo` from your normal account:

```sh
sudo -iu sadist-agent
```

Install or expose the required tools for that user. If `junie`, `git`,
and `gh` are installed system-wide, verify them inside the agent shell:

```sh
junie --version
git --version
gh --version
```

If Junie was installed only under your personal home directory, install
it again while logged in as `sadist-agent` so the executable and Junie
state belong to the agent user.

#### Repository permissions

Prefer a checkout owned by the agent user:

```sh
sudo mkdir -p /srv/agents
sudo chown sadist-agent:sadist-agent /srv/agents
sudo -iu sadist-agent
git clone git@github.com:my-handicapped-pet/sadist-planner.git /srv/agents/sadist-planner
cd /srv/agents/sadist-planner
```

If you want both your personal user and the agent user to edit the same
working tree, use a shared group instead of making the agent user an
administrator:

```sh
sudo groupadd sadist-agents
sudo usermod -aG sadist-agents "$USER"
sudo usermod -aG sadist-agents sadist-agent
sudo chgrp -R sadist-agents /path/to/sadist-planner
sudo chmod -R g+rwX /path/to/sadist-planner
sudo find /path/to/sadist-planner -type d -exec chmod g+s {} +
```

After changing groups, start a new login session so group membership is
refreshed.

#### Token-based GitHub and Junie credentials

For parity with CI, prefer explicit tokens for `sadist-agent` instead
of browser-login state copied from a personal account. Keep the tokens
outside this repository: store them in the agent user's shell profile,
system secret store, password manager, or CI secret store. Never commit
tokens to `.junie/`, `README.md`, shell scripts, or Git-tracked config
files.

Use separate tokens for separate systems:

- `GITHUB_TOKEN` or `GH_TOKEN` for GitHub API, issue, and pull request
  operations.
- `JUNIE_API_KEY` or BYOK provider credentials for Junie CLI model
  access.

Use least-privilege scopes. For this coordination repo, a GitHub token
usually needs only repository contents, issues, and pull request access
for `my-handicapped-pet/sadist-planner`. Expand scopes only when a task
explicitly requires cross-repo work.

Configure GitHub CLI as the agent user with a token supplied from the
agent user's environment:

```sh
sudo -iu sadist-agent
export GH_TOKEN="<github-token-for-sadist-agent>"
gh auth status
```

If `gh auth status` does not pick up `GH_TOKEN` in your shell, store the
same token in the agent user's GitHub CLI credential store:

```sh
printf '%s' "$GH_TOKEN" | gh auth login --with-token
gh auth status
```

Authenticate Junie separately for the agent user with a dedicated token:

```sh
sudo -iu sadist-agent
cd /srv/agents/sadist-planner
export JUNIE_API_KEY="<junie-api-key-for-sadist-agent>"
junie --auth="$JUNIE_API_KEY" "Confirm Junie CLI token authentication works in this repo."
```

For interactive sessions, keep `JUNIE_API_KEY` available only to the
agent user's shell and then start Junie normally:

```sh
sudo -iu sadist-agent
cd /srv/agents/sadist-planner
export JUNIE_API_KEY="<junie-api-key-for-sadist-agent>"
junie
```

If you use BYOK instead of `JUNIE_API_KEY`, configure provider-specific
credentials only for the `sadist-agent` user and keep them out of the
repository.

Before a local run that should mirror CI, verify both token-backed tools
from the agent shell:

```sh
gh auth status
test -n "$JUNIE_API_KEY" || test -n "$ANTHROPIC_API_KEY" || test -n "$OPENAI_API_KEY"
```

Avoid relying on personal interactive browser sessions for repeatable
agent runs. Browser login is acceptable for one-off manual exploration,
but token-backed auth should be the default for `sadist-agent` whenever
you want local behavior to stay close to CI behavior.

#### Legacy interactive authentication

If token-based authentication is not available yet, authenticate `gh` as
the agent user only when GitHub issue or pull request operations are
needed:

```sh
sudo -iu sadist-agent
gh auth login
gh auth status
```

Use a dedicated GitHub account or token with the smallest practical
scope. Do not reuse your personal `gh` authentication files.

Authenticate Junie separately for the agent user:

```sh
sudo -iu sadist-agent
cd /srv/agents/sadist-planner
junie
```

For headless runs, use a dedicated `JUNIE_API_KEY` or BYOK credentials
stored in the agent user's environment or secret store.

#### Auto-approval safety

Auto-approval is safer when it is scoped to the agent user because the
allowlist is stored under that user's Junie home, normally
`~/.junie/allowlist.json`.

- Keep allowlist entries narrow, for example project-relative file paths
  and exact command prefixes required by this repo.
- Avoid broad entries such as allowing every shell command, every file
  edit outside the project, or unrestricted MCP tools.
- Review the agent user's `~/.junie/allowlist.json` before enabling brave
  mode for a long-running or headless task.
- Prefer running agents from a dedicated checkout such as
  `/srv/agents/sadist-planner` so any approved file edits stay inside the
  agent workspace.

#### Starting an agent session

Run agents from the repository root as the dedicated user:

```sh
sudo -iu sadist-agent
cd /srv/agents/sadist-planner
junie
```

Then use `/planner-agent` or `/repo-agent` as described below.

### `planner-agent`

Use `planner-agent` to turn a rough idea, issue, or planning document
into a human-reviewable plan.

Run in the project root:

```sh
junie
```

Then start the agent with one slash command:

```text
/planner-agent
```

The command expands to a prompt that follows
`.junie/agents/planner-agent.md`, uses
`.junie/templates/repo-handoff.md` when needed, and refers to
`.junie/workflows/local-coordination.md` before Git or GitHub work.
If the planning task was not provided yet, the agent will ask for it.

The equivalent manual prompt is:

```text
Use planner-agent to expand this idea into a plan.
Follow @.junie/agents/planner-agent.md and use @.junie/templates/repo-handoff.md
when a repo handoff is needed.
Source: @.junie/plans/0001-AI-Orchestration.md
```

The expected output is a canonical Markdown plan under `.junie/plans/`
with goal, non-goals, affected repositories, interface changes,
acceptance criteria, test strategy, rollout notes, and repo handoff
notes. Internal tracking files belong under `.junie/tracking/`.

### `repo-agent`

Use `repo-agent` for approved repo-scoped coordination changes, such as
updating planning files, preparing a branch, and preparing or opening a
pull request.

Run in the project root:

```sh
junie
```

Then start the agent with one slash command:

```text
/repo-agent
```

The command expands to a prompt that follows
`.junie/agents/repo-agent.md` and runs the pre-flight checks in
`.junie/workflows/local-coordination.md` before Git or GitHub work.
If the approved task was not provided yet, the agent will ask for it.

The equivalent manual prompt is:

```text
Use repo-agent to implement this approved coordination change.
Follow @.junie/agents/repo-agent.md and run the pre-flight checks in
@.junie/workflows/local-coordination.md before Git or GitHub work.
```

Before branch or pull request work, `repo-agent` should confirm the repo
root, read the local guidelines, inspect `git status --short`, confirm
the current branch, and check `gh --version` plus `gh auth status` when
GitHub operations are needed.

### Notes

- For milestone #1, agents work only in this coordination repository.
- Do not run `repo-agent` in target application repositories until those
  repositories are enrolled.
- Pull requests, pushes, and commits should happen only when explicitly
  requested by the current task.
