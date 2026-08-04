---
name: test-contracts
description: >
  Run unit and integration test on in-memory OdraVM or (and) real WASMs on CasperVM
  and report results.
  Use when the user says "test contract", "verify contract(s)", "unit test".
allowed-tools: Bash(cargo odra *), Bash(./scripts/*), Read, WebFetch
---

Use `AskUserQuestion` tool to verify the user intent if not explicitly expressed.
If the tests should be run on `OdraVM`, `CasperVM` or both.

Running tests on `OdraVM`:

```bash
cargo odra test
```

Running tests on `CasperVM`:

First, check if WASMs need to be rebuilt:

```bash
./scripts/needs_rebuild.sh
```

Then run tests with the appropriate flag based on the result:

```bash
# If rebuild is needed (needs_rebuild.sh exited 0):
./scripts/run_tests.sh --rebuild

# If no rebuild is needed (needs_rebuild.sh exited 1):
./scripts/run_tests.sh
```

If tests fail, help the user fix the issues. Explain any errors in context.

## Reference

- [Testing](https://odra.dev/docs/basics/testing) — test API, asserting on errors and events
- [OdraVM](https://odra.dev/docs/backends/odra-vm) vs [Casper](https://odra.dev/docs/backends/casper)
  — read these when a test passes on OdraVM but fails on CasperVM; the two do not model everything
  the same way
- [`docs-map.md`](../../reference/docs-map.md) — index of the whole documentation set
