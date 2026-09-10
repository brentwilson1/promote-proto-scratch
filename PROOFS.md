# Evidence index

Question: can a promote button drive a gated publish workflow using **only the
default `GITHUB_TOKEN`**, by calling publish via `workflow_call` instead of
relying on a tag push to re-trigger it?

This repo has **zero secrets** (`gh secret list` is empty) and the repo default
workflow permission is **read**. Every push below is the default token, elevated
only by the `permissions:` block in the workflow itself.

| # | Claim | Result | Run |
|---|-------|--------|-----|
| 1 | Default-token tag push does NOT trigger the `push: tags:` workflow | Tag `acme/prod/v1` created; zero push-event runs | [34424869794](https://github.com/brentwilson1/promote-proto-scratch/actions/runs/34424869794) |
| 2 | `workflow_call` job binds to `environment: prod` and pauses | Run status `waiting`, job `call-publish / publish` waiting | [34424869794](https://github.com/brentwilson1/promote-proto-scratch/actions/runs/34424869794) |
| 3 | Prevent-self-review holds through `workflow_call` | `current_user_can_approve: false`; approval API returned 422 | [34424869794](https://github.com/brentwilson1/promote-proto-scratch/actions/runs/34424869794) |
| 4 | Approval path completes with correct inputs | `org=acme env=prod sha=26e7589` after approval | [34424869794](https://github.com/brentwilson1/promote-proto-scratch/actions/runs/34424869794) |
| 5 | Control: user-credential tag push DOES trigger publish | `publish` run created, `event=push`, `ref=acme/prod/v900` | [34425006336](https://github.com/brentwilson1/promote-proto-scratch/actions/runs/34425006336) |
| 6 | No PAT / App anywhere | No `secrets.*` reference; no secrets exist; repo default = read | grep in `.github/workflows/` |
| 7a | Ruleset `creation` restricted, no bypass -> token blocked | `GH013 ... creations being restricted` | [34425124312](https://github.com/brentwilson1/promote-proto-scratch/actions/runs/34425124312) |
| 7b | `RepositoryRole: write` bypass does NOT admit the token | `GH013` again | [34425355457](https://github.com/brentwilson1/promote-proto-scratch/actions/runs/34425355457) |
| 7c | Ruleset `update`+`deletion` only, **zero bypass actors** -> token creates tag | tag created; gate still holds | [34425428946](https://github.com/brentwilson1/promote-proto-scratch/actions/runs/34425428946) |
| 7d | Pointer ruleset `creation`+`update` restricted, no bypass -> token blocked | `acme/prod/current` rejected | [34425567212](https://github.com/brentwilson1/promote-proto-scratch/actions/runs/34425567212) |
| 7e | Pointer ruleset `deletion` only -> token writes pointer | `moved pointer acme/prod/current` | [34425657548](https://github.com/brentwilson1/promote-proto-scratch/actions/runs/34425657548) |
| 7f | Token can force-MOVE an existing pointer | `(forced update)` | [34425726610](https://github.com/brentwilson1/promote-proto-scratch/actions/runs/34425726610) |

Also verified: the GitHub Actions app (id 15368) **cannot** be added as a ruleset
bypass actor here — the API rejects it with
`Actor GitHub Actions integration must be part of the ruleset source or owner organization`.

Immutability holds against everyone, including the repo owner: deleting or moving
`acme/prod/v1` is rejected with `GH013` with `bypass_actors: []`.
