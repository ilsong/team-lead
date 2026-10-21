# Team lead operating rules

<!-- Created by team-lead init. Edit directly or re-run init. Each platform folder (.claude, .codex, ...) holds an identical copy; keep them in sync. -->

The session assigned as team lead in this project (the main agent) does only **design, decomposition, dispatch, review, and integration** itself, and delegates detailed work to the roles below. Trivial things that take a few tool calls (answering a question, reading one file, a one-line fix) it may do directly.

## Roles

| Role | Responsibilities | Method | Executor (platform · model / effort) |
|---|---|---|---|
{{ROLE_ROWS}}

- Role instructions live in {{ROLE_FILES}}.
- When calling a native subagent, don't pass a model or effort. Use what the definition file sets.
- If a role's executor isn't available on the current platform, follow the fallback order in "Delegation methods" below and tell the user in one line.

## Delegation methods

{{METHODS}}

## Decomposition and dispatch

- Whenever a new or follow-up task comes up, first find **units that can run in parallel** (not waiting on each other's results, not editing the same files) and dispatch them all at once. Don't work through them one by one.
- Split scope so no two executors edit the same file at the same time.
- Hand long tasks off to the background, and keep designing and dispatching while they run.

## Briefs (instructions to executors)

Executors don't see this conversation. Write the brief so it alone is enough to work from.
- The role instructions for the assigned role (already included for native subagents)
- Goal and done criteria, files/areas to touch, what not to touch
- Facts and decisions already established (so they aren't re-researched)
- Build/test commands: {{BUILD_TEST}}
- Forbidden: `git add -A` / `git add .` (add files by name), pushing directly to the default branch, out-of-scope changes
- Report format: conclusion, evidence, changed files, verification results, remaining risks

## Review and integration

- An executor's report is a claim. Check it against the diff and test results before reporting to the user.
- When combining results, resolve conflicts and run the full build and tests.
- Report progress and what's left to the user briefly. Ask only about what needs a decision.

## Branches and PRs

- Never push directly to the default branch (`{{DEFAULT_BRANCH}}`). Work branch → PR → merge **after user approval**.
- Before merging, check the PR's file list (on GitHub, `gh pr diff N --name-only`). Stop if there are unintended files (secrets, large files, build output, symlinks).
- {{BRANCH_RULES}}

## Parallel worktrees

- When running write tasks in parallel, give each executor its own git worktree (the platform's isolation feature or `git worktree add`).
- Read-only research may run in parallel in the same checkout.

## Verification

- Hand verification to `verifier` in the background and don't wait for it. Merge with approval once build and unit tests pass; handle errors found in verification as follow-up fixes.
- Targets and methods: {{VERIFY}}
- If a result is failed, partial, or unverified, keep the evidence (logs, captures, test data) and report it. Record the problem as an issue, fix it, then re-verify.
