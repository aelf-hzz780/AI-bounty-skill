# Account Choice And Onboarding

Use this branch for generic claim requests before entering a concrete claim path.

## When To Use

Use this flow when any of the following is true:

- the user says `帮我 Claim`, `帮我领取`, or another generic claim request
- the user has not yet chosen between `CA` and `EOA`
- the user chose `CA` or `EOA`, but the local account context is not ready yet

## Required User-Facing Explanation

The agent must first explain:

- `CA`: account-style experience, usually based on email / guardian / recovery flows, current campaign reward is `2 tokens`
- `EOA`: traditional wallet experience, usually based on mnemonic / private key, current campaign reward is `1 token`
- recommendation: choose `CA`

Then ask:

- `要使用 CA 账号还是 EOA 账号？`

## Step-By-Step

1. Tell the user not to fill exchange or custodial addresses.
2. Explain the difference between `CA` and `EOA` using the required explanation above.
3. Recommend `CA` because the current campaign reward is `2 tokens`, while `EOA` gets `1 token`.
4. Ask the user to choose `CA` or `EOA`.
5. If the user chooses `CA` and a local CA account is already available, switch to [portkey-ca.md](./portkey-ca.md).
6. If the user chooses `CA` but no local CA account is ready, guide the user to create or recover a local Portkey CA account first:
   - use a local Portkey account flow with email / guardian semantics
   - make sure the CA context is available on `tDVV`
   - make sure a manager signer is available locally
7. After the local CA account is ready, switch to [portkey-ca.md](./portkey-ca.md).
8. If the user chooses `EOA` and a local EOA account is already available, switch to [eoa-skill.md](./eoa-skill.md).
9. If the user chooses `EOA` but no local EOA account is ready, guide the user to create a local self-custodied EOA first:
   - use mnemonic / private key style account semantics
   - make sure the signer is locally available
   - make sure the user understands EOA only gets `1 token` in the current campaign
10. After the local EOA account is ready, switch to [eoa-skill.md](./eoa-skill.md).

## Must-Stop Conditions

Stop and switch to [diagnostics-stop.md](./diagnostics-stop.md) if:

- the user tries to use an exchange or custodial address
- the user wants the agent to create or hold the wallet for them
- the user refuses to choose between `CA` and `EOA`

## Output Shape

The response should contain:

- chosen flow: `Account Choice And Onboarding`
- `CA vs EOA` explanation
- explicit recommendation for `CA`
- explicit question asking the user to choose `CA` or `EOA`
- if needed, the next local account creation step

## Example Reference

Read [../examples/account-choice.md](../examples/account-choice.md) before replying.
