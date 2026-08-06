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

```bash
cargo odra test -b casper
```

This builds every contract in `Odra.toml` to wasm, runs `wasm-opt` and `wasm-strip` over each one,
and then executes the same tests against Casper's execution engine. `cargo odra` tracks whether the
wasm is stale, so there is no separate rebuild step to decide on — pass `--skip-build` only when you
have just built and want to reuse the existing wasm.

:::note
The `full`, `blank`, `cep18` and `cep95` templates do **not** generate a `scripts/` directory. Do not
look for `scripts/needs_rebuild.sh` or `scripts/run_tests.sh` — drive `cargo odra test` directly. If a
particular project happens to have its own `scripts/`, those are the user's, not Odra's.
:::

CasperVM runs require the `wasm32-unknown-unknown` target plus `wasm-opt` and `wasm-strip` on
`PATH`; OdraVM requires none of them. If `-b casper` fails on tooling rather than on a test
assertion, run `/odra-plugin:check-env` before touching the contract.

If tests fail, help the user fix the issues. Explain any errors in context.

## Reference

- [Testing](https://odra.dev/docs/basics/testing) — test API, asserting on errors and events
- [OdraVM](https://odra.dev/docs/backends/odra-vm) vs [Casper](https://odra.dev/docs/backends/casper)
  — read these when a test passes on OdraVM but fails on CasperVM; the two do not model everything
  the same way
- [`docs-map.md`](../../reference/docs-map.md) — index of the whole documentation set
