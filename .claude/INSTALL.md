# AI Bounty Claim Bootstrap For Claude Code

Use this file as a bootstrap entry for Claude Code. The canonical instructions live in `SKILL.md` at the repository root.

## Required steps

1. Read `SKILL.md` at the repository root first and treat it as the source of truth.
2. After choosing one branch, read the matching files under `references/flows/` and `references/examples/`.
3. If your local Claude Code setup supports a project-local skill workflow, make this repository available locally. Otherwise continue directly from the repository files.
4. For generic claim requests, explain `AA/CA` vs `EOA` first, recommend `AA/CA`, and ask the user to choose.
5. If the user already chose `AA/CA` or supplied an email for recovery/login, follow the AA/CA path.
6. Use the provided email only to recover/login when needed to resolve local AA/CA context or a usable manager signer on `tDVV`.
7. Never use exchange or custodial addresses.
8. Before any on-chain write, show the write summary and wait for explicit confirmation.
9. If a transaction fails, surface the exact chain error and stop.

## Write summary checklist

- manager signer
- `caHash`
- Portkey CA contract raw address
- reward contract raw address
- method chain `ManagerForwardCall -> ClaimByPortkeyToCa(Hash ca_hash)`
- receiver semantics
- reward amount
- gas note

## Notes

- Treat `CA` as an accepted alias for the `AA/CA` route.
- Keep `SKILL.md` at the repository root as the canonical version when behavior looks inconsistent.
