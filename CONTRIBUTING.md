# Contributing to StarPass Contracts

Thank you for your interest in contributing to StarPass. This project participates in the [Stellar Wave Program](https://drips.network/wave/stellar) on Drips — contributors earn USDC rewards for resolving issues.

## Prerequisites

- [Rust](https://rustup.rs/) 1.75+
- [Soroban CLI](https://developers.stellar.org/docs/smart-contracts/getting-started/setup)

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install WASM target
rustup target add wasm32-unknown-unknown

# Install Soroban CLI
cargo install --locked soroban-cli
```

## Setup

```bash
# Fork the repo on GitHub, then clone your fork
git clone https://github.com/YOUR_USERNAME/starpass-contracts
cd starpass-contracts

# Verify everything builds
cargo build

# Run all tests — all should pass before you start
cargo test
```

## Workflow

1. Browse [open issues](https://github.com/laugh-tales/starpass-contracts/issues) and find one you want to work on
2. Leave a comment on the issue expressing your interest
3. Wait to be assigned — do not start work before assignment
4. Fork the repo and create a branch:

```bash
git checkout -b feat/your-feature-name
# or
git checkout -b fix/issue-description
```

5. Make your changes
6. Run the full check suite:

```bash
cargo fmt          # format code
cargo clippy       # lint — fix all warnings
cargo test         # all tests must pass
```

7. Commit using conventional commit format (see below)
8. Push and open a Pull Request against `main`

## Commit Message Format

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>: <short description>
```

Types:
| Type | When to use |
|---|---|
| `feat` | New feature or function |
| `fix` | Bug fix |
| `test` | Adding or updating tests |
| `docs` | Documentation only |
| `refactor` | Code change with no behavior change |
| `chore` | Build, CI, tooling |

**Examples:**
```
feat: add pass renewal function to contract
fix: correct fee calculation for zero-fee edge case
test: add expiry edge cases to has_valid_pass tests
docs: document deploy script usage
```

## Pull Request Requirements

Before opening a PR:
- [ ] `cargo fmt` passes with no changes
- [ ] `cargo clippy` passes with no warnings
- [ ] `cargo test` — all tests pass
- [ ] New functionality has corresponding tests
- [ ] PR description references the issue it closes (`Closes #123`)

## Running Tests

```bash
# Run all tests
cargo test

# Run tests with output
cargo test -- --nocapture

# Run a specific test
cargo test test_mint_pass
```

## Code Style

- Follow standard Rust conventions
- Document all public functions with `///` doc comments
- Keep functions focused — one responsibility per function
- Use `assert!` with descriptive error messages
- Prefer `persistent()` storage for user data, `instance()` for contract globals

## Questions

Open a GitHub Discussion or comment on the relevant issue. Do not DM maintainers.
