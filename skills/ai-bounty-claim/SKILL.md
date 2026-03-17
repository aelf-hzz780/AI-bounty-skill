---
name: ai-bounty-claim
description: Claim the AI bounty on tDVV by delegating to the canonical repo SKILL and its branch docs; prefer AA/CA and require explicit write confirmation.
homepage: https://github.com/aelf-hzz780/AI-bounty-skill
metadata: {"openclaw":{"homepage":"https://github.com/aelf-hzz780/AI-bounty-skill"}}
---

# AI Bounty Claim

This workspace OpenClaw skill is a thin bootstrap wrapper. The canonical instructions live in `SKILL.md` at the repository root.

## Use this skill when

- the user wants to claim the AI bounty on `tDVV`
- the user needs `AA/CA` vs `EOA` routing
- the user wants to use `email` to recover/login before an `AA/CA` claim

## Required steps

- Read `SKILL.md` at the repository root first and treat it as the source of truth.
- Choose exactly one branch and then read the matching files in `references/flows/` and `references/examples/`.
- Prefer `AA/CA`; accept `CA` as the same route alias.
- Never ask the user to paste an exchange or custodial address.
- Before any chain write, show the manager signer, `caHash`, CA contract raw address, reward contract raw address, method chain, receiver semantics, reward amount, and gas note, then require explicit confirmation.
- If a transaction fails, surface the exact chain error and stop.
