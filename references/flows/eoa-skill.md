# EOA Claim

Use this branch for the EOA claim path after account type selection is complete.

## Required Dependency

Use the Portkey EOA skill explicitly:

- `https://github.com/Portkey-Wallet/eoa-agent-skills`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `EOA`
- a local EOA account is already available
- no CA identity flow is involved
- account choice is already complete

## Required Facts

- Method: `Claim()`
- Receiver: `Context.Sender`
- Current campaign default reward: `1 AIBOUNTY`
- `ca_hash` is not part of this flow

## Step-By-Step

1. Tell the user not to fill exchange or custodial addresses.
2. Validate the `tDVV` RPC through `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`.
3. Use the Portkey EOA skill dependency to resolve the active signer from the local EOA account context.
4. If the resolved address is exchange-managed, custodial, or signer ownership is unclear, stop and switch to [diagnostics-stop.md](./diagnostics-stop.md).
5. Check that the claim contract is currently claimable on `tDVV`.
6. Check that the resolved local EOA address has not already claimed.
7. Show the write summary:
   - signer address
   - contract address
   - method `Claim()`
   - receiver semantics `reward goes to Context.Sender`
   - source of signer `resolved from local EOA account`
   - expected reward `1 AIBOUNTY` in the current campaign
   - RPC validation endpoint `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`
8. Ask for explicit confirmation.
9. Only after explicit confirmation, send `Claim()`.
10. Report the `txid` and the exact chain result.
11. If the transaction fails, surface the original error and stop.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- the address belongs to OKX or another exchange
- the contract is not claimable yet
- the address has already claimed
- the user asks for `ca_hash` handling in this branch
- no local EOA account can be resolved
- `/api/blockChain/chainStatus` cannot be reached

## Output Shape

The response before sending the transaction should contain:

- chosen flow: `EOA Claim`
- resolved signer
- target contract
- method
- receiver semantics
- signer source `local EOA account`
- reward amount
- explicit confirmation request

## Example Reference

Read [../examples/eoa-skill.md](../examples/eoa-skill.md) before replying.
