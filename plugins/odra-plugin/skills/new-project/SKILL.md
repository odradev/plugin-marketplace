---
name: new-project
description: >
  Scaffold a new Odra project with `cargo odra new`, or add Odra to an existing
  empty directory with `cargo odra init`. Use when the user says "new odra
  project", "create a project", "start a project", "scaffold", "set up odra",
  "init odra", or asks for a contract when no Odra project exists yet.
allowed-tools: Bash(cargo odra *), Bash(ls *), Bash(cat *), Read, Edit, AskUserQuestion
---

# Create a new Odra project

**Never hand-write an Odra project.** Always scaffold with `cargo odra new`. A project has
interlocking pieces — `Odra.toml`, `build.rs`, three `[[bin]]` targets, a pinned `rust-toolchain`,
a `cfg(not(target_arch = "wasm32"))` dependency section — and hand-rolling them produces a project
that looks right and fails in non-obvious ways. The template is the source of truth.

---

## Step 1 — Check there is no project already

```bash
ls Odra.toml
```

If `Odra.toml` exists, this is already an Odra project: stop, and use `smart-contract-writer` to add
a contract instead.

---

## Step 2 — Choose a template

```bash
cargo odra list-templates
```

Use the output, not memory — the set changes between releases. As of Odra 2.9 it is:

| Template    | Use when |
| ---         | --- |
| `full`      | Default. One crate, a sample `Flipper` contract, tests, and a CLI binary. |
| `blank`     | You want the wiring but none of the sample code. |
| `workspace` | Several contract crates plus a shared `cli` crate. |
| `cep18`     | Starting from a fungible token (CEP-18). |
| `cep95`     | Starting from an NFT (CEP-95). |

If the user has not said which they want, ask with `AskUserQuestion` — but pick `full` without asking
when they just said "a new project" and are clearly getting started.

---

## Step 3 — Scaffold

Pass a name that is already a valid Rust package name — lowercase, underscores, no hyphens:

```bash
cargo odra new --name my_project --template full
```

:::note
The directory is named **verbatim** from `--name`, while the Cargo package name is normalized.
`--name my-project` therefore creates `my-project/` containing a package called `my_project`, and
every `cd my_project` afterwards fails. Passing an underscore name keeps the two identical.
:::

To scaffold into the current directory instead (it must be empty), use `cargo odra init` with the
same flags.

Other flags worth knowing:
- `-s, --source` — pin the Odra version: a crates.io version, a git branch, a commit, or a local path
  (`-s ../odra`). Defaults to the latest release.
- `-r, --repo-uri` — a different template repository.

---

## Step 4 — Verify it works before writing any code

```bash
cargo odra test
```

The first run is slow: the project pins a **nightly** toolchain via `rust-toolchain`, and rustup
downloads it on demand. Later runs are seconds.

If this fails, it is an environment problem, not a code problem — run `/odra-plugin:check-env` and
install whatever it names. Do not start editing the generated code to make a failing build pass.

---

## Step 5 — Orient the user

Point out what was generated and what each piece is for:

- `src/` — your contracts. Every new module needs `pub mod <name>;` in `src/lib.rs`.
- `Odra.toml` — the list of contracts to build to wasm. Each entry's `fqn` is the Rust path to the
  struct (`<module>::<Struct>`, or `<crate>::<module>::<Struct>` in a workspace). A contract missing
  from here still passes OdraVM tests but never produces a wasm file.
- `bin/build_contract.rs`, `bin/build_schema.rs` — generated tooling; do not edit.
- `bin/cli.rs` — your deploy script and scenarios; this one *is* meant to be edited.
- `build.rs`, `rust-toolchain` — leave alone.
- `wasm/` — appears only after `cargo odra build` or `cargo odra test -b casper`.

Then hand off: `smart-contract-writer` to write contracts, `/odra-plugin:test-contracts` to run
tests, `/odra-plugin:deploy-to-livenet` to deploy.
