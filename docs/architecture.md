# Architecture Notes (Conceptual)

## Principles
- **Execution-first grounding:** decisions must be informed by real signals.
- **Disposable environments:** every attempt runs in a clean container.
- **Verify everything:** no fix is accepted without rebuild/retest.

## Core Pipeline
1. Prepare container + mount workspace
2. Run build/test command(s)
3. Capture signals (stdout/stderr/exit code + artifacts)
4. Summarize failure context for the agent
5. Produce patch (code/config/deps)
6. Re-run verification
7. Return only verified outcome

## Components
- Runner
- Observer
- Patch applicator
- Verifier
- Snapshot/Diff
- (Optional) Multi-model debate + reviewer gate

## Notes on safety
Prefer a capability model (allowed commands list) and resource limits to reduce risk in demos.
