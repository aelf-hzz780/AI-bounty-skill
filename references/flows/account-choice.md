# Account Choice And Onboarding

Use this branch for generic claim requests before entering a concrete claim path.

## When To Use

Use this flow when any of the following is true:

- the user makes a generic claim request without explicitly choosing `AA/CA` or `EOA`
- examples: `help me claim`, `claim for me`, `帮我 Claim`, `帮我领取`
- the user has not yet chosen between `AA/CA` and `EOA`
- the user chose `AA`, `CA`, `AA/CA`, or `EOA`, but the local account context is not ready yet

## Required User-Facing Explanation

The agent must first explain:

- `AA/CA`: account-style experience, usually based on email / guardian / recovery flows, current campaign reward is `2 AIBOUNTY`
- `AA`: this is the preferred user-facing term in this skill
- `CA`: this is still accepted as an alias because some users still say `CA`
- `EOA`: traditional wallet experience, usually based on mnemonic / private key, current campaign reward is `1 AIBOUNTY`
- `AA/CA`: when fee balance looks low, the current environment may still provide a daily gas subsidy worth `1 ELF`, so the first confirmed AA/CA claim can usually be tried before fee is treated as the blocker
- `EOA`: if there is not enough `ELF`, the claim transaction can fail with `Transaction fee not enough`
- `EOA`: if the user cannot get enough `ELF`, they should switch to `AA/CA`
- recommendation: choose `AA/CA`

Then ask:

- `Which account type do you want to use: AA/CA or EOA?`

## Step-By-Step

1. Tell the user not to fill exchange or custodial addresses.
2. Explain the difference between `AA/CA` and `EOA` using the required explanation above.
3. Tell the user that `AA` is the preferred term in this skill, while `CA` is still accepted as the same route alias.
4. Recommend `AA/CA` because the current campaign reward is `2 AIBOUNTY`, while `EOA` gets `1 AIBOUNTY`, because `AA/CA` has a smoother gas experience in the current environment and can usually try one confirmed AA/CA claim before fee becomes the blocker, and because users who cannot prepare `ELF` for `EOA` can avoid that blocker by choosing `AA/CA`.
5. Ask the user to choose `AA/CA` or `EOA`.
6. If the user chooses `AA`, `CA`, or `AA/CA` and a local AA/CA account is already available, or the target `caHash` is already known, use the Portkey CA skill dependency, then switch to [portkey-ca.md](./portkey-ca.md).
7. If the user chooses `AA`, `CA`, or `AA/CA` but no local AA/CA account is ready, guide the user to create or recover a local Portkey AA/CA account first:
   - use the Portkey CA skill dependency
   - use a local Portkey account flow with email / guardian semantics
   - make sure the AA/CA context is available on `tDVV`
   - make sure the target `caHash` can be resolved on `tDVV`
8. After the local AA/CA account is ready, use the Portkey CA skill dependency, then switch to [portkey-ca.md](./portkey-ca.md).
9. If the user chooses `EOA` and a local EOA account is already available, use the Portkey EOA skill dependency, then switch to [eoa-skill.md](./eoa-skill.md).
10. If the user chooses `EOA` but no local EOA account is ready, guide the user to create a local self-custodied EOA first:
   - use the Portkey EOA skill dependency
   - use mnemonic / private key style account semantics
   - make sure the signer is locally available
   - make sure the user understands EOA only gets `1 AIBOUNTY` in the current campaign
   - make sure the user understands EOA must have enough `ELF` to pay gas before sending `Claim()`
   - make sure the user understands that if getting `ELF` is difficult, switching to `AA/CA` is the recommended fallback
11. After the local EOA account is ready, use the Portkey EOA skill dependency, then switch to [eoa-skill.md](./eoa-skill.md).

## Must-Stop Conditions

Stop and switch to [diagnostics-stop.md](./diagnostics-stop.md) if:

- the user tries to use an exchange or custodial address
- the user wants the agent to create or hold the wallet for them
- the user refuses to choose between `AA/CA` and `EOA`

## Output Shape

The response should contain:

- chosen flow: `Account Choice And Onboarding`
- `AA/CA vs EOA` explanation
- explicit recommendation for `AA/CA`
- explicit question asking the user to choose `AA/CA` or `EOA`
- if needed, the next local account creation step
- if the user leans toward `EOA`, a clear reminder that lack of `ELF` should trigger a switch to `AA/CA`
- explicit note that `CA` is still accepted as an alias for the `AA/CA` route

## Example Reference

Read [../examples/account-choice.md](../examples/account-choice.md) before replying.
