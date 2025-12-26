# Architecture Notes (Conceptual)

AgentDrydock is organized as a **VS Code-driven control loop** that keeps AI suggestions grounded in **real execution**. The core split is:

- **VS Code Extension**: user-facing trigger + Docker artifact generation + log streaming
- **Developer Helper Container**: control plane (orchestrator + agents)
- **Project Container**: disposable execution sandbox (build/run/test)

---

## High-level goals

1) **Execution-first grounding**
   - Every fix must be driven by real signals (compiler/test/runtime output).

2) **Isolation**
   - Target code runs in an ephemeral sandbox container, not on the host.

3) **Controlled write-back**
   - Changes to the project happen through a mounted volume, so edits are explicit and auditable.

---

## Components

### 1) VS Code Extension (Trigger + UX)
Responsibilities:
- Detect user action / watch mode (“Build & Run” / file change)
- Generate/update:
  - `Dockerfile`
  - `docker-compose.yml`
- Spawn the Developer Helper Container
- Stream loop progress and final status to **VS Code Output**

### 2) Project Workspace (Target)
Inputs include:
- source files (e.g., a Python file / Rust file / etc.)
- `requirements.txt` or equivalent dependency manifest
- generated artifacts:
  - `Dockerfile`
  - `docker-compose.yml`

### 3) Developer Helper Container (Control Plane)
Responsibilities:
- Run the orchestrator (e.g., `main.py`)
- Maintain the attempt loop (run → observe → patch → verify)
- Coordinate agents:
  - **Docker Files Creator Agent** (Dockerfile/compose generation or adjustments)
  - **Build & Run Agent** (fix proposals driven by real failures)
- Apply patches to workspace through the mounted volume
- Initiate `docker compose` actions to build/run the sandbox container

### 4) Project Container (Execution Sandbox)
Responsibilities:
- Build/run/test the target code in isolation
- Produce ground-truth signals:
  - stdout/stderr
  - exit codes
  - test results (optional)
- Is discarded after a successful verification (or after stop policy limits)

---

## Data & control flow (typical)

1) Extension generates Docker artifacts in the project workspace  
2) Extension spawns the helper container with a mounted project volume  
3) Helper uses compose to build/run the project container  
4) Project container emits failure signals (logs + exit codes)  
5) Helper summarizes signals and asks the agent(s) for a patch  
6) Patch is applied to the workspace through the mounted volume  
7) Helper re-runs until verification passes or stop conditions hit  
8) Logs are streamed back to VS Code Output

---

## Verification model

A fix is only “accepted” if it is verified by:
- successful rebuild
- and/or passing test suite
- and/or a successful runtime check

No fix should be presented as final without a verification step.

---

## Safety notes (demo-friendly defaults)

Recommended constraints for demos:
- explicit allowlist for commands that can run
- resource limits (CPU/memory/timeouts)
- maximum attempts per run
- redact secrets from logs before sending to any model endpoint
