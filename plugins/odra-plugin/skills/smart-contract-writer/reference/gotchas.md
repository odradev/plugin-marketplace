# Odra gotchas

Traps that cost time when writing Odra 2.9 code. Every item here was hit and reproduced against a
real project — these are not hypotheticals.

Read this file before writing any non-trivial contract, and again when a compile error does not make
obvious sense.

## Project setup

- **Never hand-create a project — scaffold it.** `cargo odra new --name <snake_case> -t <template>`.
  A project has interlocking parts (`Odra.toml`, `build.rs`, three `[[bin]]` targets, a pinned
  `rust-toolchain`, a `cfg(not(target_arch = "wasm32"))` dependency section); writing them by hand
  produces something that looks right and fails later in ways that read as contract bugs.
- **The project pins a nightly toolchain.** A generated project contains a `rust-toolchain` file
  (e.g. `nightly-2026-01-01`). rustup installs it on first build, so the first `cargo odra test` is
  slow. `rustup target add wasm32-unknown-unknown` must apply to *that* toolchain — run it from
  inside the project directory.
- **The folder name is taken verbatim from `--name`; the package name is normalized.**
  `cargo odra new --name my-project` gives a `my-project/` directory containing a package called
  `my_project`. Pass an underscore name to keep them identical.
- **A new module is not picked up until it is declared.** Adding `src/foo.rs` does nothing until
  `pub mod foo;` is in `src/lib.rs`. For a contract you also want to build to wasm, add a
  `[[contracts]]` entry to `Odra.toml`. OdraVM tests pass without the `Odra.toml` entry, so this
  failure only surfaces later, at `cargo odra build`.
- **`fqn` is a Rust path, not a package path.** Single-crate: `fqn = "flipper::Flipper"`
  (`<module>::<Struct>`). Workspace: `fqn = "flipper::flipper::Flipper"` (`<crate>::<module>::<Struct>`).
- **The wasm file is named after the struct, case included** — `Flipper.wasm`, not `flipper.wasm`.
- **`cargo odra build` has no `-b`/`--backend` flag.** Only `cargo odra test` does. `-c` selects
  contracts and takes **space**-separated names: `cargo odra build -c Foo Bar`.
- **`wasm/` does not exist after a plain `cargo odra test`** — only after `cargo odra build` or
  `cargo odra test -b casper`.
- **Templates are `full`, `blank`, `workspace`, `cep18`, `cep95`.** There is no `cep78` template.
  `cargo odra list-templates` is authoritative.

## Storage

- **`get_or_default()` requires `T: Default`.** `Address` does not implement it, so
  `Var<Address>::get_or_default()` does not compile. Use `get().unwrap()` or
  `get_or_revert_with(Error::X)`. Same for your own `#[odra::odra_type]` types unless they derive
  `Default` — and a unit-only enum needs `#[derive(Default)]` plus `#[default]` on a variant.
- **`Var::get_or_revert_with(err)` takes one argument.** The similarly named `unwrap_or_revert_with`
  comes from the `UnwrapOrRevert` trait on `Option`/`Result` and takes **two**
  (`.unwrap_or_revert_with(self, err)`). Reach for `get_or_revert_with` on a `Var`.
- **`Sequence::next_value()` returns `0` on its first call**, then `1`, `2`… It is a zero-based
  counter, not a pre-increment. `get_current_value()` also returns `0` for an untouched sequence, so
  it cannot tell you whether the sequence has been used.
- **Nested mappings go through `Mapping::module(&key)`**, which yields a `SubModule<V>` with storage
  isolated per key. `Mapping` also has `add()`/`subtract()` for numeric values.
- **Field indices start at 1**, not 0. The compact key encoding only holds while every index in the
  path is ≤ 15; nesting deeper than 8 levels reverts with `PathIndexOutOfBounds`.

## Modules, attributes, calls

- **`#[odra::module]` goes on the struct *and* on the impl block.** Without it on the impl, no entry
  points are generated and the type never implements `OdraContract`/`Deployer` — the error surfaces
  far away, at `Foo::deploy(...)`, as an unsatisfied trait bound.
- **A second, un-annotated `impl` block holds internal helpers.** Do not annotate two impl blocks for
  the same struct — that is a duplicate-impl error.
- **`ContractEnv::attached_value()` returns `U512`**, not `U256`.
- **`self.env()` is a method, not a field.** `self.env.revert(...)` fails with E0615.
- **`#[odra(payable)]` on `init` is silently ignored.** It compiles, but a constructor can never
  receive attached tokens. Fund the contract through a separate payable entry point after deploy.
- **The cross-call type is `{Module}ContractRef`**, and `new()` comes from the `ContractRef` trait —
  `use odra::ContractRef;` must be in scope wherever you construct one by hand.
- **`ContractRef::new` takes `Rc<ContractEnv>`, never `&HostEnv`.** From a host-side test:
  `FooContractRef::new(Rc::new(env.contract_env()), address)`.
- **`Deployer::deploy` is implemented on the module struct, not on the `ContractRef`.** Use
  `Foo::deploy(&env, args)`, which returns a `FooHostRef`.
- **`.address()` comes from `Addressable`,** not `HostRef` — import `odra::prelude::Addressable`.
- **Factory deployments do not work on OdraVM.** Anything calling `new_contract` must run under
  `cargo odra test -b casper`.

## Host / testing

- **`HostEnv::emitted_event` takes the event by value**, not by reference: `emitted_event(&contract, Ev { .. })`.
- **`HostEnv::transfer(to, amount) -> OdraResult<()>`** is the host-side transfer. There is no
  `HostEnv::transfer_tokens`; that name only exists on `ContractEnv` (and returns `()`, reverting on
  failure).
- **To assert a revert, use the `try_` variant.** `contract.foo()` panics the test on revert; only
  `contract.try_foo()` returns a comparable `Result`.
- **A contract with a constructor gets `{Module}InitArgs`; one without gets nothing** — use
  `odra::host::NoArgs`.
- **Arithmetic overflow is a raw Rust panic, not a clean revert.** Odra does not add Solidity-style
  checked arithmetic. OdraVM surfaces it only as `Err(VmError(Panic))` with no message, so guard
  subtractions explicitly.

## odra-modules

- The security module is spelled **`Pauseable`** (`odra_modules::security::Pauseable`), with the
  extra `e`. There is no `Pausable` alias.
- **`Cep95` has no `mint`** — the public minting function is `raw_mint(to, token_id, metadata)`, and
  it is deliberately unguarded. Add your own access control around it.
- **`Cep18` has no `assert_caller`.** Compare `self.env().caller()` yourself and revert.
- **`AccessControl` needs bootstrapping.** Nothing grants the first role, so a standalone deployment
  can never pass its own admin checks. The wrapping contract's `init` must call
  `unchecked_grant_role(&DEFAULT_ADMIN_ROLE, &self.env().caller())`.

## Livenet / deployment

- **`odra-casper-livenet-env` is not in a generated project.** Add it as an optional dependency and
  gate it behind a `livenet` feature, or `--features livenet` fails with "does not contain this feature".
- **Binaries live in `bin/`, not `src/bin/`,** and are registered with `[[bin]]` (double brackets).
- **Build the wasm before running a livenet binary**, or it fails with `Failed to find wasm file`.
- **Every deploy and every mutating call needs `env.set_gas(...)`.** No gas set is rejected locally
  with `Gas not set`; too little gas *is* sent and fails on-chain with `Out of gas error`. Gas budgets
  are per contract — expect to tune them.
- **Extra livenet accounts are numbered from 1** (`ODRA_CASPER_LIVENET_KEY_1`); account 0 is
  `ODRA_CASPER_LIVENET_SECRET_KEY_PATH`.
- **`ODRA_LOG_LEVEL=debug`** prints the full transaction JSON before it is sent — the fastest way to
  see what was actually signed.

## Building wasm

**Always build through `cargo odra build` or `cargo odra test -b casper`.** They run the whole wasm
pipeline for you. Never invoke `wasm-opt`/`wasm-strip` by hand to "fix" a build — doing so hides the
real problem and gets the order wrong (stripping first removes the sections `wasm-opt` needs, which
fails with thousands of validation errors).

A failure in those commands is almost always a missing tool, not a code problem. Read the message and
install what it names, rather than working around it:

| Message | Missing |
| --- | --- |
| `wasm32-unknown-unknown target is not present` | `rustup target add wasm32-unknown-unknown` — run it inside the project so it applies to the pinned nightly |
| `There was an error while running wasm-opt - is it installed?` | binaryen (`brew install binaryen`) |
| `There was an error while running wasm-strip - is it installed?` | wabt (`brew install wabt`) |
| `wasm-pack is not installed` | wasm-pack, needed only by `cargo odra generate-client` |

Driving `cargo` directly is a deliberate escape hatch for controlling feature flags, not a
troubleshooting step; if you genuinely need it, follow the "Building contracts manually" page in the
Odra docs rather than improvising.

## Odra CLI

- The name in `CommandArg::new("number", ...)` and the key in `args.get_single::<u64>("number")` are
  matched at runtime, not compile time. A mismatch compiles fine and fails only when the scenario
  runs, with `Arg error: Unexpected arg: <name>`. Grep any scenario you write to confirm every
  registered name has an exactly matching lookup key.
- `--json` only formats the final report of a *successful* command. Errors and transaction-progress
  lines are always plain text, so parse the last JSON object in stdout rather than the whole stream.
