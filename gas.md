Ethereum gas fees are computed from three core variables: gas used, base fee, and priority fee (tip).  The total fee in ETH is the effective gas price multiplied by the gas units actually consumed.

## Core gas fee formula

For an EIP-1559-style chain like Ethereum mainnet, the canonical formula is:

- Effective gas price (in gwei or wei):  
  \(\text{effectiveGasPrice} = \text{baseFeePerGas} + \text{priorityFeePerGas}\) 
- Total fee paid (in ETH):  
  \(\text{totalFee} = \text{gasUsed} \times \text{effectiveGasPrice}\) 

In practice UIs often show:

- \(\text{maxFeePerGas}\): user’s max total price per gas  
- \(\text{maxPriorityFeePerGas}\): user’s max tip  
- On-chain effective gas price is \(\min(\text{maxFeePerGas}, \text{baseFeePerGas} + \text{maxPriorityFeePerGas})\). 

So a more explicit formula is:

- \(\text{effectiveGasPrice} = \min(\text{maxFeePerGas}, \text{baseFeePerGas} + \text{maxPriorityFeePerGas})\) 
- \(\text{totalFee} = \text{gasUsed} \times \text{effectiveGasPrice}\) 
When people say “gas fee = gas price × gas limit”, they are implicitly assuming gasUsed ≈ gasLimit for the estimate. 
## Gas limit vs gas used vs gas price

- **Gas limit**  
  - User-specified upper bound on gas the tx can consume. 
  - If execution runs out of gas before completion, it reverts but still burns all gas used up to that point. 
- **Gas used**  
  - Actual gas consumed by EVM opcodes and intrinsic costs (base tx cost, calldata, storage writes, etc.). 
  - Billed post-execution; unused gas is refunded (except intrinsic gas). 
- **Gas price / fees per gas**  
  - On post-London Ethereum, decomposed into base fee (burned) + priority fee (paid to validator). 
  - Users cap these via maxFeePerGas and maxPriorityFeePerGas, which influence inclusion speed. 

Typical values:

- Simple ETH transfer: ~21,000 gas used. 
- ERC-20 approve: ~45,000 gas used.
- Complex DeFi interactions: several hundred thousand gas. 
## Where the formulas come from

Conceptually:

- Gas is a **unit of computational work and state access**; each opcode has a cost, plus extra for calldata bytes and storage writes. 
- Blocks are constrained by a **block gas limit** (target ~15M gas, hard max 30M on Ethereum currently), so validators must choose which transactions to include. 
- The protocol prices gas via an **auction**: users bid with maxFeePerGas / maxPriorityFeePerGas, and validators prefer more profitable txs. 

EIP-1559 adds:

- A dynamically adjusted **base fee** per gas that rises when blocks are over target gas and falls when they are under, ensuring fee market stability. 
- A separate **priority fee** (tip) to let users compete for inclusion order and speed. 

Hence, the linear formula:

- Work (gasUsed) × price per unit of work (effectiveGasPrice) = money paid (totalFee). 

## Other variables that influence gas

Beyond gas limit and price, several factors affect what you actually pay:

- **Transaction complexity**  
  - More storage writes, external calls, and loops → higher gasUsed. 
  - Contract design, packing, and avoiding cold SLOADs/SSTOREs materially change gas profile.

- **Network demand and congestion**  
  - High demand increases baseFeePerGas via EIP-1559’s adjustment mechanism, raising everyone’s fees. 
  - Arbs and MEV-driven traffic can spike priority fees for time-sensitive txs. 
- **Desired confirmation speed**  
  - “Fast” transactions typically use higher priority fees to outbid other users; “slow” ones underbid and wait for low-demand blocks. 

- **L1 vs L2 and data costs**  
  - Rollups pay L1 data availability costs; gas estimation for L2 includes both L2 execution gas and L1 calldata data fees in different units. 

- **Protocol-specific overhead**  
  - For account abstraction (ERC-4337), fee formula adds components like preVerificationGas, verificationGas, and callGas.
  - Actual fee is \(\text{gasFee} \times (\text{preVerificationGas} + \text{meteredVerificationGas} + \text{meteredCallGas})\). 

## Example real-time gas estimation function

At a high level, a real-time estimator needs:

- Current baseFeePerGas (from latest block header)  
- Recommended priority fees by speed tier (safe / standard / fast) from recent blocks  
- A good gasUsed estimate per tx type (via eth_estimateGas or historical profiling) 
Pseudocode for an Ethereum-like chain (JavaScript-style):

```ts
type SpeedTier = "slow" | "standard" | "fast";

interface GasMarketSample {
  baseFeePerGas: bigint;          // from latest block
  suggestedPriority: {
    slow: bigint;
    standard: bigint;
    fast: bigint;
  };                              // derived from recent txs
}

interface TxProfile {
  targetGasUsed: bigint;          // estimated gasUsed from eth_estimateGas or historical
  safetyMarginBps: number;        // e.g. 11000 for +10%
}

function estimateGasParams(
  market: GasMarketSample,
  profile: TxProfile,
  speed: SpeedTier,
  maxFeeMultiplierBps: number = 20000 // cap maxFee at 2x(base+priority) by default
) {
  const base = market.baseFeePerGas;                         // wei
  const tip = market.suggestedPriority[speed];               // wei

  const effective = base + tip;                              // wei
  const maxFeePerGas = (effective * BigInt(maxFeeMultiplierBps)) / 10_000n;

  const gasLimit = (profile.targetGasUsed * BigInt(profile.safetyMarginBps)) / 10_000n;

  const totalFeeWei = gasLimit * effective;                  // worst-case fee
  return { baseFeePerGas: base, maxPriorityFeePerGas: tip, maxFeePerGas, gasLimit, totalFeeWei };
}
```

- `targetGasUsed` should be obtained by calling `eth_estimateGas` with the intended transaction data, possibly averaged or padded by historical executions of the same function signature. 
- `suggestedPriority` can be produced by scanning the last N blocks and computing median tips for transactions by inclusion latency. 
- In practice, tools like Etherscan and ETH Gas Station do this kind of sampling and prediction to provide real-time suggested gas prices.

If you want, a follow-up can go deeper into opcode-level derivation of gasUsed for a given Solidity function and how to empirically profile and bound gas for auditing.