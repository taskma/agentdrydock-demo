# Demo Notes

- Demo video: https://www.youtube.com/watch?v=2IbIenrjvfg
- Website: https://www.agentdrydock.com/
- Contact: taskma@gmail.com

## What the demo is showing (conceptually)

1) A VS Code-centered workflow triggers the loop.
2) Docker artifacts (Dockerfile + docker-compose) are generated for the project context.
3) A dedicated helper container runs the orchestrator + agents.
4) The target project executes inside a disposable sandbox container.
5) Real failure signals (stdout/stderr/exit codes/tests) drive patch suggestions.
6) Patches are written back through a mounted project volume.
7) The system re-runs and verifies until it passes (or hits a stop policy).

> Note: Demo content may include randomly generated code. No proprietary or production code is included.
