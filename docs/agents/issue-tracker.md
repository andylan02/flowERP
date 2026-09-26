# Issue tracker

Issues and specs for this repo live as GitHub issues. Use the `gh` CLI for all operations.

## Conventions

- Create an issue: `gh issue create --title "..." --body "..."`. Use a heredoc for multiline bodies.
- Read an issue: `gh issue view <number> --comments`, optionally filtering comments by `jq` and labels.
- List issues: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'` with suitable filters.
- Comment on an issue: `gh issue comment <number> --body "..."`
- Apply or remove labels: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- Close an issue: `gh issue close <number> --comment "..."`

Infer the repo from `git remote -v`; `gh` will resolve the active repo automatically when run inside this clone.

## Pull requests as a triage surface

PRs as a request surface: no.

This repository uses GitHub Issues as the canonical work queue; PRs are not treated as a separate feature-request channel by default.

## When a skill says "publish to the issue tracker"

Create a GitHub issue with `gh issue create`.

## When a skill says "fetch the relevant ticket"

Run `gh issue view <number> --comments`.

## Wayfinding operations

Used by `/wayfinder`. The map is a single issue with child issues as tickets.

- Map: a single issue labelled `wayfinder:map` holding the notes, decisions, and frontier information.
- Child ticket: an issue linked to the map as a sub-issue. Where sub-issues are unavailable, use a task list in the map body and include `Part of #<map>` in the child body.
- Blocking: use GitHub's native issue dependencies when available.
- Claim: `gh issue edit <n> --add-assignee @me`
- Resolve: `gh issue comment <n> --body "<answer>"`, then `gh issue close <n>`
