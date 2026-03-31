# AI Bounty Claim Bootstrap For Claude Code

Use this file as a bootstrap entry for Claude Code. The canonical instructions live in `SKILL.md` at the repository root.

## Required steps

1. Read `SKILL.md` at the repository root first and treat it as the source of truth.
2. After choosing one branch, read the matching files under `references/flows/` and `references/examples/`.
3. If your local Claude Code setup supports a project-local skill workflow, make this repository available locally. Otherwise continue directly from the repository files.
4. For generic claim requests, explain that the current public bounty path is `AA/CA only`.
5. If the user already has local AA/CA context or supplied an email for recovery/login, follow the AA/CA path.
6. Require `@portkey/ca-agent-skills >= 2.3.0` for AA/CA because the flow depends on `manager-sync-status` or `portkey_manager_sync_status`.
7. Use the provided email only to recover/login when needed to resolve local AA/CA context or a usable manager signer on `tDVV`.
8. Before any AA/CA write, resolve local AA/CA context and `caHash`, run `manager-sync-status`, run `GetHolderInfo(caHash)`, and stop unless both manager sync and holder lookup are ready on `tDVV`.
9. Never infer a fee problem from `Holder is not found` or `CA holder not found`; only the final chain error `Transaction fee not enough` may trigger fee guidance.
10. If a user asks for the old EOA route, explain that this public skill no longer supports it and redirect them to AA/CA onboarding or diagnostics.
11. Never use exchange or custodial addresses.
12. Before any on-chain write, show the write summary and wait for explicit confirmation.
13. If a transaction fails, surface the exact chain error and stop.

## Write summary checklist

- manager signer
- manager sync status
- holder lookup status
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
