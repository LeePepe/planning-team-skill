# Team Config

> Copy this file to `.claude/team.md` in your repo to customize teamwork routing.
> The team-lead and planner read this file before starting.

## Executor Routing

<!-- Override default routing. Only two executors: codex and copilot. -->
<!--
Examples:
- *.swift, *.m, *.xib → copilot
- *.ts, *.tsx, *.js   → codex
- src/ui/**           → copilot
- src/api/**          → codex
- tests/**            → codex
- *.py, *.sh          → copilot
-->

## Review Mode

<!-- Options: review, adversarial-review -->
default: review

## Verification

<!-- Optional post-execution verification commands (run in repo root). -->
<!--
Examples:
- npm run lint
- npm test
- pnpm -r test
- go test ./...
-->

## Notes

<!-- Context for the planner and team-lead about this repo -->
