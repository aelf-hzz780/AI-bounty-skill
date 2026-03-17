# AI Bounty Claim Extension

This Gemini CLI extension is context-only. It does not ship extra MCP servers; it teaches Gemini CLI how to use the canonical claim instructions from this repository.

## Use this extension when

- the user wants to claim the AI bounty on `tDVV`
- the user needs `AA/CA` vs `EOA` routing
- the user wants to use `email` to recover/login before an `AA/CA` claim

## Required steps

- Read `SKILL.md` in this repository first and treat it as the source of truth.
- After choosing a branch, read the matching files under `references/flows/` and `references/examples/`.
- Recommend `AA/CA` because the current campaign default reward is `2 AIBOUNTY`, compared with `1 AIBOUNTY` for `EOA`.
- Accept `email` or `caHash` as the AA/CA starting input.
- Never use exchange or custodial addresses.
- Before any chain write, show the write summary and require explicit confirmation.
- If a transaction fails, surface the exact chain error and stop.
