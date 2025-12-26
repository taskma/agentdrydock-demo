# Terminology

- **VS Code Extension**: The UI/trigger layer that generates Docker artifacts, spawns the helper container, and streams logs.
- **Project Workspace**: The user’s codebase and related files (source + dependency manifests).
- **Project Container**: Disposable sandbox container used to build/run/test the project.
- **Developer Helper Container**: Control-plane container running the orchestrator and agents.
- **Mounted Volume**: The shared filesystem mount that allows the helper to apply patches to the project workspace.
- **Attempt**: One cycle of run → observe → patch.
- **Signal Pack**: Structured output captured from execution (stdout, stderr, exit code, optional test report).
- **Verifier**: The step that re-runs build/tests and decides pass/fail.
- **Docker Files Creator Agent**: Agent responsible for generating/adjusting Dockerfile and docker-compose when needed.
- **Build & Run Agent**: Agent responsible for proposing patches based on observed failure signals.
- **VS Code Output**: The output panel where loop progress and results are presented to the user.
