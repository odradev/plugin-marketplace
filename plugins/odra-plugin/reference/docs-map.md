# Odra documentation map

Where to look when the plugin's bundled reference does not answer the question.

The skills under `skills/*/reference/` cover the common path for Odra 2.9. This file covers
everything else: it maps a question to the page that answers it, so you can fetch one page
instead of searching the whole site.

**Fetch pages with `WebFetch`.** Every URL below is a normal documentation page.

## Version scheme

| URL shape | Serves |
| --- | --- |
| `https://odra.dev/docs/<path>` | the latest released docs (2.9.0) — use this by default |
| `https://odra.dev/docs/2.8.0/<path>` | an older release |
| `https://odra.dev/docs/next/<path>` | unreleased, built from `master` of the docs repo |

Check the project's `Cargo.toml` for the `odra` version. If it is older than the latest release,
prefer the matching versioned URL — API details do change between minor versions.

`https://odra.dev/llms.txt` is the machine-readable index of all of it, regenerated on every
docs build. If a page below 404s, fetch `llms.txt` — it is authoritative and this file is a copy.

---

## Start here for common gaps

| The question | The page |
| --- | --- |
| Which tools do I need installed? | [Installation](https://odra.dev/docs/getting-started/installation) |
| Setup is failing on Ubuntu or WSL | [Ubuntu / WSL setup](https://odra.dev/docs/getting-started/ubuntu-wsl-setup) — troubleshooting table keyed by error message |
| What do the generated files do? | [Directory structure](https://odra.dev/docs/basics/directory-structure) |
| How do I register a contract for building? | [Odra.toml](https://odra.dev/docs/basics/odra-toml) |
| Which storage type do I use? | [Storage interaction](https://odra.dev/docs/basics/storage-interaction) |
| What does `#[odra::module]` expand to? | [Flipper Internals](https://odra.dev/docs/basics/flipper-internals) |
| How do I test a revert / an event? | [Testing](https://odra.dev/docs/basics/testing), [Errors](https://odra.dev/docs/basics/errors) |
| How do I call another contract? | [Cross calls](https://odra.dev/docs/basics/cross-calls) |
| How do I take or send CSPR? | [Native token](https://odra.dev/docs/basics/native-token) |
| Why does OdraVM pass but CasperVM fail? | [OdraVM](https://odra.dev/docs/backends/odra-vm), [Casper](https://odra.dev/docs/backends/casper) |
| How do I deploy to a real network? | [Livenet](https://odra.dev/docs/backends/livenet) |
| The API changed under me | [Migrations](https://odra.dev/docs/migrations/to-2.6.0) and the [CHANGELOG](https://github.com/odradev/odra/blob/HEAD/CHANGELOG.md) |
| I need an exact signature | [docs.rs/odra](https://docs.rs/odra/latest/odra/) |

---

## Getting started

- [Installation](https://odra.dev/docs/getting-started/installation) — prerequisites, installing
  `cargo-odra`, creating and testing a first project
- [Ubuntu / WSL setup](https://odra.dev/docs/getting-started/ubuntu-wsl-setup) — the same process
  with exact, verified commands for Ubuntu and WSL, plus a troubleshooting table keyed by error
  message. Use it for any "how do I install X" question on Linux.
- [Flipper example](https://odra.dev/docs/getting-started/flipper) — the contract every project is
  scaffolded with

## Basics

- [Cargo Odra](https://odra.dev/docs/basics/cargo-odra) — every `cargo odra` command and its flags
- [Directory structure](https://odra.dev/docs/basics/directory-structure) — what a generated project
  contains and what each file is for
- [Odra.toml](https://odra.dev/docs/basics/odra-toml) — the contract registry; a contract missing
  here will not build
- [Flipper Internals](https://odra.dev/docs/basics/flipper-internals) — line-by-line explanation of
  the Odra-specific parts of a module
- [Storage interaction](https://odra.dev/docs/basics/storage-interaction) — `Var`, `Mapping`, `List`
  and when each is the right choice
- [Host Communication](https://odra.dev/docs/basics/communicating-with-host) — `self.env()`: caller,
  block time, and the rest of the contract context
- [Testing](https://odra.dev/docs/basics/testing) — writing unit and integration tests against a
  deployed contract
- [Errors](https://odra.dev/docs/basics/errors) — declaring error enums, reverting, asserting on
  failures in tests
- [Events](https://odra.dev/docs/basics/events) — declaring, emitting and asserting on events
- [Cross calls](https://odra.dev/docs/basics/cross-calls) — calling one contract from another
- [Modules](https://odra.dev/docs/basics/modules) — reusing code across contracts with `SubModule`
- [Native token](https://odra.dev/docs/basics/native-token) — attaching, receiving and transferring
  CSPR
- [Casper Contract Schema](https://odra.dev/docs/basics/casper-contract-schema) — the schema
  generated into `resources/`, and the attributes that drive it

## Advanced

- [Delegate](https://odra.dev/docs/advanced/delegate) — `delegate!` macro; re-export a submodule's
  methods from the parent without boilerplate
- [Advanced Storage Concepts](https://odra.dev/docs/advanced/advanced-storage) — nested `Mapping`,
  `Sequence`, and storage patterns beyond the basics
- [Attributes](https://odra.dev/docs/advanced/attributes) — `payable`, `non_reentrant` and the rest;
  the Odra answer to Solidity modifiers
- [Storage Layout](https://odra.dev/docs/advanced/storage-layout) — how module state maps onto
  Casper named keys and dictionaries
- [Memory allocators](https://odra.dev/docs/advanced/using-different-allocator) — `no-std` builds and
  swapping the wasm allocator
- [Building contracts manually](https://odra.dev/docs/advanced/building-manually) — what
  `cargo odra build` does, for when you need control over the cargo/compiler flags
- [Signatures](https://odra.dev/docs/advanced/signatures) — verifying signatures in a contract and
  signing in tests
- [Delegating CSPR to Validators](https://odra.dev/docs/advanced/delegating-cspr) — staking from a
  contract (Casper 2.0+)
- [Factory](https://odra.dev/docs/advanced/factory) — deploying contracts from a contract
- [Wasm-Client](https://odra.dev/docs/advanced/wasm-client) — generating a TypeScript client for a
  deployed contract

## Backends

- [What is a backend?](https://odra.dev/docs/backends/what-is-a-backend) — the `-b` flag and what the
  choices mean
- [OdraVM](https://odra.dev/docs/backends/odra-vm) — the in-memory VM used by `cargo odra test`; read
  this to know which behaviours it does *not* model
- [Casper](https://odra.dev/docs/backends/casper) — compiling to wasm and running against Casper's
  execution engine locally
- [Livenet](https://odra.dev/docs/backends/livenet) — deploying to nctl, testnet or mainnet, and the
  env vars that configure it

## Examples

- [odra-examples](https://odra.dev/docs/examples/odra-examples) — guide to the `examples/` crate in
  the framework repository
- [Using odra-modules](https://odra.dev/docs/examples/using-odra-modules) — pulling in the
  ready-made modules shipped as `odra-modules`

## Tutorials

Full worked examples. When a user asks for something in this list, follow the tutorial rather than
inventing an implementation.

- [Access Control](https://odra.dev/docs/tutorials/access-control) — role-based access, beyond a
  single owner
- [Ownable](https://odra.dev/docs/tutorials/ownable) — the single-owner module
- [Pausable](https://odra.dev/docs/tutorials/pauseable) — pausing entry points
- [OwnedToken](https://odra.dev/docs/tutorials/owned-token) — composing the modules above
- [ERC-20](https://odra.dev/docs/tutorials/erc20) — fungible token, Ethereum-style
- [CEP-18](https://odra.dev/docs/tutorials/cep18) — fungible token, Casper standard; the default
  choice for a token on Casper
- [Ticketing System](https://odra.dev/docs/tutorials/nft) — NFTs via the CEP-95 standard
- [Odra CLI](https://odra.dev/docs/tutorials/odra-cli) — building a CLI client for a contract
- [Build, Deploy and Read the State of a Contract](https://odra.dev/docs/tutorials/build-deploy-read)
  — the full path end to end
- [Deploying a Token on Casper Livenet](https://odra.dev/docs/tutorials/deploying-on-casper) — a real
  deployment, start to finish
- [Upgrading Contracts](https://odra.dev/docs/tutorials/upgrades) — upgrading an already deployed
  contract
- [Using Proxy Caller](https://odra.dev/docs/tutorials/using-proxy-caller) — calling payable entry
  points that need a purse
- [Odra for Solidity developers](https://odra.dev/docs/tutorials/odra-sol) — Solidity concept to Odra
  equivalent, side by side

## Migrations

Read the guide for the version the project is moving to when code that used to compile no longer
does. Newest first:

- [to v2.6.0 from 2.*](https://odra.dev/docs/migrations/to-2.6.0) — requires `cargo-odra` v0.1.7
- [to v2.1.0 from 2.0.*](https://odra.dev/docs/migrations/to-2.1.0) — test code changes, `odra` CLI
- [to v2.0.0 from 1.*](https://odra.dev/docs/migrations/to-2.0.0) — toolchain change, little else
- [to v1.3.0](https://odra.dev/docs/migrations/to-1.3.0)
- [to v0.9.0](https://odra.dev/docs/migrations/to-0.9.0)
- [to v0.8.0](https://odra.dev/docs/migrations/to-0.8.0)

---

## Beyond the docs site

When the documentation is silent, ambiguous, or looks out of date, go to the source:

- [docs.rs/odra](https://docs.rs/odra/latest/odra/) — exact signatures, traits and type parameters.
  The fastest way to settle "does this method take `&self` or `&mut self`" and similar.
- [odradev/odra](https://github.com/odradev/odra) — the framework itself:
  - [`examples/`](https://github.com/odradev/odra/tree/HEAD/examples) — compiling contracts for
    nearly every feature; the best answer to "show me a working version of this"
  - [`modules/`](https://github.com/odradev/odra/tree/HEAD/modules) — source of the shipped modules
    (`Ownable`, `AccessControl`, `Cep18`, `Cep95`, ...)
  - [`CHANGELOG.md`](https://github.com/odradev/odra/blob/HEAD/CHANGELOG.md) — what changed in each
    release
  - [`templates/`](https://github.com/odradev/odra/tree/HEAD/templates) — the project templates
    `cargo odra new` generates from
- [odradev/cargo-odra](https://github.com/odradev/cargo-odra) — the build tool; read it when a
  `cargo odra` command behaves unexpectedly. `cargo odra <command> --help` is usually faster.
- [odradev/odradev.github.io](https://github.com/odradev/odradev.github.io) — the docs source. Pages
  live in `docusaurus/docs/`; if a page is wrong, that is where the fix belongs.

Prefer the documentation over your own recollection of the API. Odra's API has changed across
2.x releases, and a plausible-looking guess costs a compile cycle to disprove.
