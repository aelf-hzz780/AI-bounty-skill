# Example: Diagnostics And Stop

## Example 1: OKX Address

### User Input

- English: `this is my OKX deposit address, can I claim with it?`
- 中文: `这个地址是 OKX 充值地址，能直接领吗？`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that `Claim()` requires a user-controlled signer
- explain that an OKX deposit address does not provide the private key
- tell the user directly not to fill exchange addresses and to use a local EOA address or local recovered CA account instead
- stop without offering wallet creation

## Example 2: Mainnet CA Exists But `tDVV` Fails

### User Input

- English: `my Portkey CA exists on mainnet, but it cannot be found on tDVV`
- 中文: `mainnet Portkey 已经有 CA，但是 tDVV 查不到。`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that the bounty flow requires holder and manager resolution on `tDVV`
- explain that the current prerequisite is not met
- tell the user to recover or re-establish the Portkey context on `tDVV` first
- stop without guessing `ca_hash`

## Example 3: Guardian Already Exists

### User Input

- English: `the guardian already exists and recovery is required, but I still want to try the contract call first`
- 中文: `Guardian 已存在，需要恢复，但是我想先试着调用合约。`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that recovery must be completed before the CA claim can be trusted
- tell the user to finish the recovery flow first
- stop without calling the contract

## Example 4: Contract Error

### User Input

- English: `the contract returned Sender is not a manager of the CA holder.`
- 中文: `合约报错 Sender is not a manager of the CA holder.`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- repeat the exact chain error
- explain that the current signer is not a valid manager on `tDVV`
- tell the user to switch to a valid manager signer
- stop without retrying blindly

## Example 5: RPC Root Returns 404

### User Input

- English: `the RPC root URL returns 404, so the tDVV node must be down`
- 中文: `tdvv-public-node.aelf.io 直接访问是 404，所以节点应该挂了`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that the RPC root URL returning `404` is not enough to prove the node is down
- tell the user to validate the node with `https://tdvv-public-node.aelf.io/api/blockChain/chainStatus`
- only treat RPC access as failed if `/api/blockChain/chainStatus` cannot be reached
