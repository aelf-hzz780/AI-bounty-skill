# Example: Diagnostics And Stop

## Example 1: OKX Address

### User Input

- English: `this is my OKX deposit address, can I claim with it?`
- 中文: `这个地址是 OKX 充值地址，能直接领吗？`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that the public bounty path requires a user-controlled Portkey AA/CA context
- explain that an OKX deposit address does not provide the private key
- tell the user directly not to fill exchange addresses and to use a local recovered AA/CA account instead
- stop without offering wallet creation

## Example 2: Mainnet AA/CA Exists But `tDVV` Fails

### User Input

- English: `my Portkey CA exists on mainnet, but it cannot be found on tDVV`
- 中文: `mainnet Portkey 已经有 CA，但是 tDVV 查不到。`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that the bounty flow requires holder and `caHash` resolution on `tDVV`
- explain that the current prerequisite is not met
- tell the user to recover or re-establish the Portkey AA/CA context on `tDVV` first
- stop without guessing `caHash`

## Example 3: Guardian Already Exists

### User Input

- English: `the guardian already exists and recovery is required, but I still want to try the contract call first`
- 中文: `Guardian 已存在，需要恢复，但是我想先试着调用合约。`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that recovery must be completed before the AA/CA claim can be trusted
- tell the user to finish the recovery flow first
- stop without calling the contract

## Example 4: Newly Registered `tDVV` Account Is Still Syncing

### User Input

- English: `I just registered on tDVV and now holder lookup still fails`
- 中文: `我刚在 tDVV 注册，现在 holder lookup 还是失败。`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that the on-chain AA/CA context is still syncing
- explain that this is a holder or chain-context readiness problem, not a fee problem
- tell the user to wait and retry later
- explicitly avoid suggesting `ELF` top-up

## Example 5: Old `AELF` Account Manager Not Synced To `tDVV`

### User Input

- English: `my old AELF account is ready, but manager-sync-status on tDVV is false`
- 中文: `我的老 AELF 账号已经有了，但 tDVV 上 manager-sync-status 是 false。`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that cross-chain sync to the sidechain may still be in progress
- explain that the current blocker is manager sync, not transaction fee
- tell the user to wait for manager and holder sync to complete, or re-login later
- stop without sending `ManagerForwardCall`

## Example 6: `Holder is not found`

### User Input

- English: `the contract returned Holder is not found`
- 中文: `合约返回 Holder is not found`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- repeat the exact chain error
- explain that the holder, `caHash`, or target-chain context is not ready on `tDVV`
- tell the user to wait for sync or re-check `caHash` and chain
- stop without suggesting `ELF`

## Example 7: Fee Not Enough After `NOTEXISTED`

### User Input

- English: `the transaction was sent, first it showed NOTEXISTED, then it failed with Transaction fee not enough`
- 中文: `交易已发送，先显示 NOTEXISTED，随后失败并提示 Transaction fee not enough`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that `NOTEXISTED` only means the transaction was still pending lookup at that time
- explain that the final failure reason is insufficient transaction fee during transaction sending
- explain that this is not an RPC outage, not a claim logic failure, and not a holder lookup failure
- for `AA/CA`, explain that one AA/CA attempt was allowed because subsidy may apply, but the final chain result still shows fee was not enough
- if a `txId` is already known, include `txId` and `https://aelfscan.io/tDVV/tx/<txid>`

## Example 8: View-Only API Misread As Full Method List

### User Input

- English: `contractViewMethodList only shows GetConfig, HasAddressClaimed, and HasCaClaimed, so this reward contract must not have ClaimByPortkeyToCa`
- 中文: `contractViewMethodList 只返回 GetConfig、HasAddressClaimed、HasCaClaimed，所以这个奖励合约应该没有 ClaimByPortkeyToCa`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that `/api/contract/contractViewMethodList` is a view-only list and cannot be used to deny write methods
- explain that seeing only read methods does not prove `ClaimByPortkeyToCa(Hash ca_hash)` is missing
- if full method verification is still needed, tell the user to use `/api/blockChain/contractFileDescriptorSet` as an optional verification path
- explain that node introspection must use the normalized contract address format, not the wrapped `ELF_..._tDVV` address string
- tell the user to keep the canonical reward contract and supported write methods already defined by the skill unless a verified source disproves them

## Example 9: Reward Still Goes To The AA/CA Address

### User Input

- English: `if my manager signer sends ManagerForwardCall for ClaimByPortkeyToCa, does the reward go to that manager signer?`
- 中文: `如果我的 manager signer 发起 ManagerForwardCall 去调用 ClaimByPortkeyToCa，奖励会发给这个 manager signer 吗？`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that the standard AA/CA wallet path is `ManagerForwardCall(...) -> ClaimByPortkeyToCa(Hash ca_hash)`
- explain that the reward goes to the AA/CA address resolved from that `caHash`, not to the manager signer
- tell the user to verify the target `caHash` and local AA/CA context instead of focusing on sender payout
- stop if the user still cannot provide or resolve the correct `caHash`

## Example 10: Wrapped Address And Wrong `caHash` Encoding

### User Input

- English: `I passed ELF_..._tDVV into the SDK and sent caHash as a string. Could that be the problem?`
- 中文: `我把 ELF_..._tDVV 包装地址直接传给 SDK 了，而且把 caHash 当字符串发了，这会是问题吗？`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that some SDK and helper calls require raw addresses such as `2Uth...` and `2fc5...`, not wrapped `ELF_..._tDVV` addresses
- explain that `ClaimByPortkeyToCa` expects `.aelf.Hash`, not a plain string
- tell the user to encode the forwarded args as `args: { value: Buffer.from(caHash, "hex") }`
- if descriptor-based encoding is involved, remind the user to call `root.resolveAll()` before resolving the method or encoding params

## Example 11: User Still Wants EOA

### User Input

- English: `I still want to claim with an EOA wallet.`
- 中文: `我还是想用 EOA 钱包来领。`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that this public reward-claim skill no longer supports the legacy EOA route
- explain that older contract source snapshots may still show legacy methods, but they are not exposed as supported public user paths here
- tell the user to create, recover, or log in to a local Portkey AA/CA account first
- stop without planning around `Claim()`
