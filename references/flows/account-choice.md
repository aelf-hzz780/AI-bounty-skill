# Account Choice And Onboarding

Use this branch for generic claim requests before entering the AA/CA claim path.

## When To Use

Use this flow when any of the following is true:

- the user makes a generic claim request
- examples: `help me claim`, `claim for me`, `帮我 Claim`, `帮我领取`
- the local Portkey AA/CA account context is not ready yet
- the user only has `email` and needs recovery/login first
- the user uses `CA` as the account term and needs alias clarification

## Required User-Facing Explanation

The agent must first explain:

- `AA/CA`: account-style experience, usually based on email / guardian / recovery flows, current campaign reward is `2 AIBOUNTY`
- `AA`: this is the preferred user-facing term in this skill
- `CA`: this is still accepted as an alias because some users still say `CA`
- `AA/CA`: this route now requires `@portkey/ca-agent-skills >= 2.3.0` because claim preflight depends on `manager-sync-status` and holder lookup on `tDVV`
- `AA/CA`: when fee balance looks low, the current environment may still provide a daily gas subsidy worth `1 ELF`, so the first confirmed AA/CA claim can usually be tried before fee is treated as the blocker
- `AA/CA`: before any claim write, we must first check manager sync and holder resolution on `tDVV`
- `EOA`: this public reward-claim skill no longer supports the legacy EOA route, even if older contract source snapshots still show legacy methods

Then ask:

- `Is your local Portkey AA/CA context already ready on tDVV, or do you want to recover/login first with email?`

## Step-By-Step

1. Tell the user not to fill exchange or custodial addresses.
2. Explain that the current public bounty path is `AA/CA only` using the required explanation above.
3. Tell the user that `AA` is the preferred term in this skill, while `CA` is still accepted as the same route alias.
4. Ask whether the user already has a local Portkey AA/CA context on `tDVV`, or wants to recover/login first with `email`.
5. If the user explicitly asks for `EOA`, explain that the current reward claim no longer supports EOA and switch to [diagnostics-stop.md](./diagnostics-stop.md).
6. If a local AA/CA account is already available, or the target `caHash` is already known, use the Portkey CA skill dependency at version `>= 2.3.0`, then switch to [portkey-ca.md](./portkey-ca.md).
7. If no local AA/CA account is ready, guide the user to create or recover a local Portkey AA/CA account first:
   - use the Portkey CA skill dependency at version `>= 2.3.0`
   - use a local Portkey account flow with email / guardian semantics
   - make sure the AA/CA context is available on `tDVV`
   - make sure the target `caHash` can be resolved on `tDVV`
   - make sure manager sync and holder lookup can be checked on `tDVV` before claim
8. After the local AA/CA account is ready, use the Portkey CA skill dependency at version `>= 2.3.0`, then switch to [portkey-ca.md](./portkey-ca.md).

## Must-Stop Conditions

Stop and switch to [diagnostics-stop.md](./diagnostics-stop.md) if:

- the user tries to use an exchange or custodial address
- the user wants the agent to create or hold the wallet for them
- the user insists on the old EOA route instead of creating or recovering a Portkey AA/CA account

## Output Shape

The response should contain:

- chosen flow: `Account Choice And Onboarding`
- explanation that the public bounty route is now `AA/CA only`
- explicit note that `CA` is still accepted as an alias for the `AA/CA` route
- explicit question asking whether local Portkey AA/CA is already ready or recovery/login with `email` is needed
- a clear note that the claim path now checks manager sync and holder readiness on `tDVV` before any write
- if needed, the next local account creation or recovery step

## Example Reference

Read [../examples/account-choice.md](../examples/account-choice.md) before replying.
