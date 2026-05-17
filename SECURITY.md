# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in StarPass Contracts, please do **not** open a public GitHub issue.

Instead, email the maintainer directly with:
- A description of the vulnerability
- Steps to reproduce
- Potential impact

We will respond within 48 hours and work with you to resolve the issue before any public disclosure.

## Supported Versions

| Version | Supported |
|---|---|
| 0.1.x | ✅ Yes |

## Known Issues

This contract is currently in pre-audit state. Do not deploy to mainnet with real funds until a full security audit has been completed.

## Scope

The following are in scope for security reports:
- `contracts/starpass/src/lib.rs` — all contract functions
- Auth bypass or unauthorized fund access
- Integer overflow or underflow
- Storage manipulation

## Out of Scope

- Frontend/UI issues
- Documentation errors
- Issues in third-party dependencies
