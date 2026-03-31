# AI Bounty Claim Extension

This Gemini CLI extension is context-only. It does not ship extra MCP servers; it teaches Gemini CLI how to use the canonical claim instructions from this repository.

## Use this extension when

- the user wants to claim the AI bounty on `tDVV`
- the user wants to use `email` to recover/login before an `AA/CA` claim
- the user needs AA/CA onboarding, sync preflight, or AA/CA diagnostics on `tDVV`

## Required steps

- Read `SKILL.md` in this repository first and treat it as the source of truth.
- After choosing a branch, read the matching files under `references/flows/` and `references/examples/`.
- Treat this public bounty skill as `AA/CA only`.
- Accept `email` or `caHash` as the AA/CA starting input.
- Require `@portkey/ca-agent-skills >= 2.3.0` for AA/CA because the flow depends on CLI `manager-sync-status` or MCP `portkey_manager_sync_status`.
- Before any AA/CA write, resolve local AA/CA context and `caHash`, run `manager-sync-status`, run `GetHolderInfo(caHash)`, and continue only when `isManagerSynced=true` and the holder resolves on `tDVV`.
- If a user asks for the old EOA route, explain that this public skill no longer supports it and redirect them to AA/CA onboarding or diagnostics.
- Do not infer a fee problem from `Holder is not found` or `CA holder not found`; only the final chain error `Transaction fee not enough` may trigger fee guidance.
- Never use exchange or custodial addresses.
- Before any chain write, show the write summary and require explicit confirmation.
- If a transaction fails, surface the exact chain error and stop.
