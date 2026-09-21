# Local Coordination Workflow

This file defines the first local workflow for `planner-agent` and
`repo-agent` in the `sadist-planner` coordination repository.

It intentionally stays small for milestone #1. The goal is to support
manual, interactive Junie runs before cross-repo automation exists.

## Pre-flight Checks

Run these checks before an agent does Git or GitHub work:

1. Confirm the current directory is the coordination repo root.
2. Read `.junie/AGENTS.md` and any task-specific plan file.
3. Check the working tree state with `git status --short`.
4. Confirm the current branch with `git branch --show-current`.
5. Confirm GitHub CLI availability with `gh --version` when a pull
   request or issue operation is needed.
6. Confirm token-backed GitHub authentication with `gh auth status`
   when a pull request or issue operation is needed. Prefer `GH_TOKEN`
   or a dedicated `sadist-agent` credential store entry over personal
   browser-login state.
7. Confirm Junie token availability with `test -n "$JUNIE_API_KEY"` or
   the relevant BYOK provider token when a run should mirror CI.

If any check reveals unrelated user changes, authentication problems,
missing tokens, or an unexpected branch, stop and ask the user how to
proceed.

## Planning Sequence

Use this sequence for `planner-agent`:

1. Read the source idea, issue, or planning document.
2. Produce or update a Markdown plan under `.junie/plans/`.
3. Keep internal tracking files under `.junie/tracking/` when a
   session needs delivery-step status or execution notes.
4. Include goal, non-goals, affected repos, interfaces,
   acceptance criteria, tests, rollout notes, and handoff.
5. Ask for human review before implementation begins.
6. Record the final status summary for the branch or pull request.

## Repo Change Sequence

Use this sequence for `repo-agent` in this repo:

1. Run the pre-flight checks.
2. Create a branch named like `agents/<short-task-name>` unless the
   user provides another branch name.
3. Apply the approved planning or coordination changes.
4. Validate Markdown and configuration files by inspection, and run
   project-specific checks if such checks are added later.
5. Prepare a pull request title and body.
6. Open the pull request only when explicitly requested or when the
   current task requires it.

## Pull Request Body Checklist

Use this checklist in pull request descriptions:

- [ ] Summary of the planning or coordination change.
- [ ] Link to the source issue or plan.
- [ ] Affected repositories listed.
- [ ] Acceptance criteria captured.
- [ ] Test strategy captured.
- [ ] Handoff notes captured where needed.
- [ ] Human review requested.

## Deferred Work

The following are intentionally deferred until the local workflow is
clear:

- Target-repo `repo-agent` execution.
- Issue and pull request templates.
- Routing labels.
- GitHub Actions workflows.
- Generic GitHub command wrappers.
- Cross-repo dispatching.
- Status aggregation.
- Chatbot or messenger integration.