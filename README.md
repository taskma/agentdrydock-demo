<!-- PROJECT BANNER -->
<p align="center">
  <img src="assets/banner.png" alt="AgentDrydock banner" />
</p>

<h1 align="center">AgentDrydock — Demo / Portfolio Showcase</h1>

<p align="center">
  <strong>Agentic AI & Coding Assistant for Containerized Build/Run Fix Loops</strong><br/>
  A personal demo prototype exploring execution-first automation for the debug loop:
  <code>edit → build → run → fail → fix</code>
</p>

<p align="center">
  <a href="https://www.agentdrydock.com/">Website</a> •
  <a href="#demo">Demo</a> •
  <a href="#how-it-works">How it Works</a> •
  <a href="#architecture">Architecture</a> •
  <a href="#roadmap">Roadmap</a> •
  <a href="#disclaimer">Disclaimer</a>
</p>

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/status-demo%2Fportfolio-blue">
  <img alt="Scope" src="https://img.shields.io/badge/scope-showcase%20repo-informational">
  <img alt="Safety" src="https://img.shields.io/badge/runs-in%20containers-important">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-green">
</p>

> **This repository is a public showcase** (docs, diagrams, video links).  
> **Not a commercial product.** No SLA. No paid support. Use at your own risk.

---

## Table of Contents
- [What is AgentDrydock?](#what-is-agentdrydock)
- [Demo](#demo)
- [Why Execution-First?](#why-execution-first)
- [How it Works](#how-it-works)
- [Architecture](#architecture)
  - [Conceptual Flow](#conceptual-flow)
  - [Step-by-Step Conceptual Flow](#step-by-step-conceptual-flow)
  - [Sequence Diagram](#sequence-diagram)
- [Why Containers?](#why-containers)
- [Use Cases](#use-cases)
- [Non-goals](#non-goals)
- [Roadmap](#roadmap)
- [Collaboration](#collaboration)
- [Legal](#legal)
- [Disclaimer](#disclaimer)

---

## What is AgentDrydock?

Many AI coding assistants can propose fixes, but without executing the code they can:
- hallucinate libraries/APIs,
- miss environment-specific context,
- produce patches that don’t compile,
- fix one error while breaking tests.

**AgentDrydock’s core idea:**  
> Don’t guess. **Run in a real environment**, capture real failure signals, feed them back to the agent loop, and verify by re-running.

---

## Demo

🎥 **Demo video:** https://www.youtube.com/watch?v=2IbIenrjvfg  
🌐 **Website:** https://www.agentdrydock.com/  
✉️ **Contact:** taskma@gmail.com

<p align="center">
  <img src="assets/workflow.gif" alt="Demo workflow" />
</p>

Example loop:

```bash
./run-agent.sh --watch
# Agent initialized. Monitoring build context...
Detected change in src/main.rs
Compiling target...
Error: implementation of `Drydock trait` missing
Agent: Analyzing compiler output...
Agent: Generating fix for impl block...
✓ Fix applied automatically
```

> Goal: faster iteration on repeat failures (often ~2–3×) — varies by codebase and failure type.

---

## Why Execution-First?

AI can “sound right” while being wrong. AgentDrydock forces grounding by using:
- real compiler output
- real test output
- real exit codes
- real runtime logs

The loop is guided by facts, not assumptions.

---

## How it Works

AgentDrydock is designed around a **VS Code-driven workflow** with a **helper container** coordinating the agent loop and a **project container** acting as the disposable execution sandbox.

1) **VS Code Extension prepares the run**
   - Generates/updates **Dockerfile** and **docker-compose.yml** for the current project context.
   - Triggers a run when you request “Build & Run” (or watch mode detects changes).

2) **Spawn Helper Container**
   - A dedicated **Developer Helper Container** is started.
   - The project workspace is mounted into the helper via a **volume** for safe, controlled edits.

3) **Run in Project Container (Sandbox)**
   - The helper uses the generated compose config to build and run the **Project Container**.

4) **Observe real signals**
   - Collects **stdout/stderr**, exit codes, and (optionally) test output.

5) **Agent loop: patch → re-run → verify**
   - A “Docker Files Creator” agent (when needed) adjusts container configs.
   - A “Build & Run” agent proposes code/config changes based on real failures.
   - The helper writes patches back through the mounted volume and re-runs until verified.

6) **Report to VS Code Output**
   - The extension streams progress and final results to **VS Code Output**.

---

## Architecture

This section includes diagrams that render on GitHub using Mermaid code blocks.

### Conceptual Flow

```mermaid
%%{init: {
  "theme":"base",
  "flowchart":{"curve":"linear"},
  "themeVariables":{
    "background":"#FFFFFF",
    "mainBkg":"#FFFFFF",
    "lineColor":"#0F172A",
    "edgeLabelBackground":"#FFFFFF",
    "fontFamily":"ui-sans-serif, system-ui"
  }
}}%%
flowchart TB
  %% --- VS Code side ---
  subgraph VS["Visual Studio Code"]
    EXT["Developer Helper Extension"]
    OUT["VS Code Output"]
  end

  %% --- Project workspace ---
  subgraph PROJ["Project Workspace (host)"]
    SRC["Source files (e.g., main.py / src/*)"]
    REQ["Dependency manifest (requirements.txt / package.json / etc.)"]
    DF["Dockerfile (generated/updated)"]
    DC["docker-compose.yml (generated/updated)"]
  end

  %% --- Helper control plane ---
  subgraph HELP["Developer Helper Container (control plane)"]
    ORCH["main.py (Orchestrator)"]
    AG_DOCKER["Docker Files Creator Agent"]
    AG_BUILD["Build & Run Agent"]
    VOL[("Mounted Volume (workspace mount)")]
  end

  %% --- Execution sandbox ---
  subgraph RUN["Project Container (sandbox)"]
    RUNNER["Build / Run / Tests"]
  end

  %% --- Flows ---
  EXT -->|"create/update"| DF
  EXT -->|"create/update"| DC

  SRC --- VOL
  REQ --- VOL
  DF --- VOL
  DC --- VOL

  EXT -->|"spawn helper container"| ORCH
  VOL --> ORCH

  ORCH --> AG_DOCKER
  ORCH --> AG_BUILD

  ORCH -->|"docker compose build/up"| RUNNER
  RUNNER -->|"stdout/stderr/exit + test results"| ORCH

  ORCH -->|"patch code/config/deps via volume"| VOL
  ORCH -->|"progress + final status"| OUT

  %% Styling (boxes & areas)
  classDef vscode fill:#E8F0FF,stroke:#2B5FD9,stroke-width:2px,color:#0B1B3A;
  classDef workspace fill:#E9FAF2,stroke:#1C8E5A,stroke-width:2px,color:#06301E;
  classDef helper fill:#FFF2E5,stroke:#C46A00,stroke-width:2px,color:#3A1F00;
  classDef sandbox fill:#F2F3F7,stroke:#5B6474,stroke-width:2px,color:#141922;
  classDef output fill:#F6E9FF,stroke:#7B2CBF,stroke-width:2px,color:#2A0A3A;

  class EXT vscode;
  class OUT output;
  class SRC,REQ,DF,DC workspace;
  class ORCH,AG_DOCKER,AG_BUILD,VOL helper;
  class RUNNER sandbox;

  style VS fill:#F3F7FF,stroke:#2B5FD9,stroke-width:3px,rx:10,ry:10
  style PROJ fill:#F2FFF8,stroke:#1C8E5A,stroke-width:3px,rx:10,ry:10
  style HELP fill:#FFF8EE,stroke:#C46A00,stroke-width:3px,rx:10,ry:10
  style RUN fill:#F7F8FB,stroke:#5B6474,stroke-width:3px,rx:10,ry:10

  %% Make ALL lines/arrowheads visible
  linkStyle default stroke:#E5E7EB,stroke-width:3px,opacity:1;




```

### Step-by-Step Conceptual Flow

```mermaid
flowchart TB
  %% =========================
  %% VS Code side
  %% =========================
  subgraph VS["Visual Studio Code"]
    EXT["Developer Helper Extension"]
    OUT["VS Code Output"]
  end

  %% =========================
  %% Project workspace
  %% =========================
  subgraph PROJ["Project Workspace (host)"]
    SRC["Source files (e.g., main.py / src/*)"]
    REQ["Dependency manifest (requirements.txt / package.json / etc.)"]
    DF["Dockerfile (generated/updated)"]
    DC["docker-compose.yml (generated/updated)"]
  end

  %% =========================
  %% Helper control plane
  %% =========================
  subgraph HELP["Developer Helper Container (control plane)"]
    ORCH["main.py (Orchestrator)"]
    AG_DOCKER["Docker Files Creator Agent"]
    AG_BUILD["Build & Run Agent"]
    VOL[("Mounted Volume (workspace mount)")]
  end

  %% =========================
  %% Execution sandbox
  %% =========================
  subgraph RUN["Project Container (sandbox)"]
    RUNNER["Build / Run / Tests"]
  end

  %% --- (1.x) Preparation & artifact generation ---
  EXT -->|"(1.1) Detect project context"| SRC
  EXT -->|"(1.2) Read deps manifest"| REQ
  EXT -->|"(1.3) Create/Update Dockerfile"| DF
  EXT -->|"(1.4) Create/Update docker-compose.yml"| DC

  %% --- (1.5) Spawn helper + mount workspace ---
  EXT -->|"(1.5) Spawn helper container"| ORCH
  SRC ---|"(1.6) Mount workspace"| VOL
  REQ ---|"(1.6) Mount workspace"| VOL
  DF  ---|"(1.6) Mount workspace"| VOL
  DC  ---|"(1.6) Mount workspace"| VOL
  VOL -->|"(1.7) Provide editable workspace view"| ORCH

  %% --- (2.x) Agent loop: run -> observe -> patch -> verify ---
  ORCH -->|"(2.1) Delegate: ensure Docker artifacts OK"| AG_DOCKER
  AG_DOCKER -->|"(2.2) (If needed) refine Dockerfile/compose"| VOL

  ORCH -->|"(2.3) Delegate: attempt build/run"| AG_BUILD
  AG_BUILD -->|"(2.4) docker compose build/up"| RUNNER
  RUNNER -->|"(2.5) stdout/stderr/exit + tests"| ORCH

  ORCH -->|"(2.6) Create patch from real signals"| AG_BUILD
  AG_BUILD -->|"(2.7) Apply patch via volume"| VOL

  ORCH -->|"(2.8) Re-run until verified or stop policy"| RUNNER

  %% --- Reporting ---
  ORCH -->|"(3.1) Stream progress + final result"| OUT
  EXT -->|"(3.2) Surface logs in VS Code"| OUT
```

### Sequence Diagram

> This diagram shows the same flow with a time-ordered view (who calls whom, and when).

```mermaid
sequenceDiagram
  autonumber

  participant U as Developer
  participant EXT as VS Code Extension
  participant WS as Project Workspace (Host)
  participant VOL as Mounted Volume
  participant ORCH as Helper Container (Orchestrator)
  participant AGD as Docker Files Creator Agent
  participant AGB as Build & Run Agent
  participant RUN as Project Container (Sandbox)
  participant OUT as VS Code Output

  U->>EXT: Trigger Build/Run (or watch detects change)
  EXT->>WS: Read context (source + deps)
  EXT->>WS: Generate/Update Dockerfile
  EXT->>WS: Generate/Update docker-compose.yml

  EXT->>ORCH: Spawn helper container
  WS-->>VOL: Mount workspace into helper (volume)
  VOL-->>ORCH: Provide editable view of workspace

  ORCH->>AGD: Validate/adjust Dockerfile/compose (if needed)
  AGD->>VOL: Update artifacts (optional)

  loop Agent loop (until verified / stop policy)
    ORCH->>AGB: Attempt build/run
    AGB->>RUN: docker compose build/up
    RUN-->>ORCH: stdout/stderr/exit + test results
    ORCH->>AGB: Decide patch using real signals
    AGB->>VOL: Apply patch to workspace
  end

  ORCH-->>OUT: Stream progress + final result
  EXT-->>OUT: Surface logs in VS Code
```

---

## Why Containers?

Traditional agents run in your local shell (risky + messy). Containers provide:
- **Isolation** — no host pollution or global config drift
- **Reproducibility** — consistent environment per attempt
- **Control** — resource limits + deterministic snapshots

---

## Use Cases
- **Missing Dependencies** — install missing system libs / language packages
- **Config Mismatches** — resolve env var conflicts (local vs CI/CD)
- **Refactoring Regressions** — fix broken tests after changes
- **Version Conflicts** — experiment with dependency matrices in isolation

---

## Non-goals
To keep this a safe, honest demo artifact:
- No claims of production readiness, SLA, or paid support
- No proprietary or customer code
- No “magic fixes” without verification
- Demo videos may contain randomly generated code

---

## Roadmap
- [ ] Smarter Docker artifact generation (language/runtime templates)
- [ ] Pluggable “run profiles” (build-only, test-only, full pipeline)
- [ ] Richer signal packs (test reports, traces, structured logs)
- [ ] Reviewer gates (agent proposes → reviewer approves → verify)
- [ ] Capability sandbox (allowed commands / restricted operations)
- [ ] Deterministic caching while preserving isolation

---

## Collaboration
This is a portfolio/demo artifact exploring a vision for developer tooling.

✅ Suggestions welcome:
- architecture reviews
- nasty failure scenarios to test
- UX feedback on the workflow

Contact: taskma@gmail.com

---

## Legal
- Demo terms: `legal/demo-terms.md`
- Privacy: `legal/privacy.md`

---

## Disclaimer
AgentDrydock is a personal technical project and portfolio/demo artifact.
All code runs in isolated containers, but **use at your own risk**.
No commercial offering. No SLA. No paid support.

© 2025 Agent Drydock. All rights reserved.
