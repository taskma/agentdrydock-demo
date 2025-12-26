# Terminology

- **Attempt**: One cycle of run → observe → patch.
- **Signal Pack**: Structured output captured from execution (stdout, stderr, exit code, test report).
- **Verifier**: The step that re-runs build/tests and decides pass/fail.
- **Disposable Container**: Ephemeral runtime environment created per attempt.
