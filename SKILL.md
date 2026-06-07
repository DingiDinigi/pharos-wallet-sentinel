---
name: pharos-wallet-sentinel
description: >
  Complete wallet health and monitoring skill for the Pharos blockchain.
  Invoke when a user wants to check wallet balances (native PHRS/PROS + ERC-20 tokens),
  view recent transaction history, get a portfolio summary, set low-balance alerts,
  or audit wallet activity on Pharos Atlantic Testnet or Pacific Mainnet.
  Covers: balance queries, tx history, portfolio overview, multi-wallet comparison,
  and agent-generated health reports. Works on both testnet and mainnet.
  Do not attempt Pharos wallet operations without this skill.
version: 1.0.0
author: DingiDinigi
requires:
  anyBins:
    - cast
    - jq
---

# Pharos Wallet Sentinel

A comprehensive wallet health and monitoring skill for the Pharos Network.
Check balances, review transaction history, compare wallets, and generate
full portfolio health reports — all via natural language.

---

## Prerequisites

### 1. Install Foundry (Required for all cast commands)

The Agent MUST first verify Foundry is installed:

```bash
which cast
```

If not found, install immediately:

```bash
curl -L https://foundry.paradigm.xyz | bash
source ~/.zshenv && foundryup
cast --version
```

Do NOT proceed without a working `cast` binary.

### 2. Install jq (Required for JSON parsing)

```bash
which jq || (sudo apt-get install -y jq 2>/dev/null || brew install jq 2>/dev/null)
```

### 3. Network Configuration

Read network config from `assets/networks.json`:

```bash
# Default: Atlantic Testnet
RPC_URL=$(jq -r '.networks[] | select(.name=="atlantic-testnet") | .rpcUrl' assets/networks.json)
EXPLORER=$(jq -r '.networks[] | select(.name=="atlantic-testnet") | .explorerUrl' assets/networks.json)
NATIVE=$(jq -r '.networks[] | select(.name=="atlantic-testnet") | .nativeToken' assets/networks.json)

# For mainnet: replace "atlantic-testnet" with "mainnet"
```

---

## Capabilities

### 1. Native Balance Check

Check the native token (PHRS/PROS) balance of any wallet address.

**Example prompt:** *"Check the balance of 0xABC... on Pharos testnet"*

**Agent steps:**

```bash
# Step 1: Read network config
RPC_URL=$(jq -r '.networks[] | select(.name=="atlantic-testnet") | .rpcUrl' assets/networks.json)
NATIVE=$(jq -r '.networks[] | select(.name=="atlantic-testnet") | .nativeToken' assets/networks.json)

# Step 2: Get balance in wei, convert to ether
BALANCE_WEI=$(cast balance <ADDRESS> --rpc-url $RPC_URL)
BALANCE_ETH=$(cast from-wei $BALANCE_WEI)

echo "Wallet: <ADDRESS>"
echo "Balance: $BALANCE_ETH $NATIVE"
```

**Health thresholds (Agent should report status):**
- `>1.0` native token → Healthy
- `0.1 – 1.0` → Low — advise topping up
- `<0.1` → Critical — warn user, provide faucet link if testnet

---

### 2. ERC-20 Token Balance

Check the balance of any ERC-20 token at a given contract address.

**Example prompt:** *"How much USDC does 0xABC... hold on Pharos?"*

```bash
RPC_URL=$(jq -r '.networks[] | select(.name=="atlantic-testnet") | .rpcUrl' assets/networks.json)

TOKEN_CONTRACT=<ERC20_ADDRESS>
WALLET=<WALLET_ADDRESS>

# Get raw balance
RAW=$(cast call $TOKEN_CONTRACT "balanceOf(address)(uint256)" $WALLET --rpc-url $RPC_URL)

# Get decimals
DECIMALS=$(cast call $TOKEN_CONTRACT "decimals()(uint8)" --rpc-url $RPC_URL)

# Get symbol
SYMBOL=$(cast call $TOKEN_CONTRACT "symbol()(string)" --rpc-url $RPC_URL)

# Convert to human-readable
BALANCE=$(echo "scale=6; $RAW / (10^$DECIMALS)" | bc)

echo "Token: $SYMBOL"
echo "Balance: $BALANCE"
```

---

### 3. Portfolio Overview (Multi-Token Summary)

Query native balance + multiple ERC-20 tokens and display a formatted summary.

**Example prompt:** *"Show me a full portfolio overview for 0xABC... on Pharos testnet"*

**Agent steps:**

```bash
RPC_URL=$(jq -r '.networks[] | select(.name=="atlantic-testnet") | .rpcUrl' assets/networks.json)
NATIVE=$(jq -r '.networks[] | select(.name=="atlantic-testnet") | .nativeToken' assets/networks.json)
WALLET=<WALLET_ADDRESS>

echo "=============================="
echo " PHAROS WALLET SENTINEL"
echo " Portfolio Report"
echo "=============================="
echo "Wallet: $WALLET"
echo ""

# Native balance
BALANCE_WEI=$(cast balance $WALLET --rpc-url $RPC_URL)
BALANCE_ETH=$(cast from-wei $BALANCE_WEI)
echo "[$NATIVE] $BALANCE_ETH"

# Query known tokens (add any contract addresses known on the target network)
TOKENS=(
  "USDC:0xYOUR_USDC_ADDRESS:6"
  "USDT:0xYOUR_USDT_ADDRESS:6"
)

for TOKEN_ENTRY in "${TOKENS[@]}"; do
  SYMBOL=$(echo $TOKEN_ENTRY | cut -d: -f1)
  ADDR=$(echo $TOKEN_ENTRY | cut -d: -f2)
  DEC=$(echo $TOKEN_ENTRY | cut -d: -f3)
  RAW=$(cast call $ADDR "balanceOf(address)(uint256)" $WALLET --rpc-url $RPC_URL 2>/dev/null || echo "0")
  HUMAN=$(echo "scale=4; $RAW / (10^$DEC)" | bc 2>/dev/null || echo "0")
  echo "[$SYMBOL] $HUMAN"
done

echo "=============================="
echo " Explorer: $EXPLORER/address/$WALLET"
echo "=============================="
```

---

### 4. Recent Transaction History

Retrieve the last N transactions for a wallet using the Pharos block explorer API.

**Example prompt:** *"Show me the last 5 transactions for 0xABC... on Pharos"*

```bash
NETWORK="atlantic-testnet"
EXPLORER_API=$(jq -r ".networks[] | select(.name==\"$NETWORK\") | .explorerApiUrl" assets/networks.json)
WALLET=<WALLET_ADDRESS>
LIMIT=5

echo "Fetching last $LIMIT transactions for $WALLET..."

curl -s "$EXPLORER_API?module=account&action=txlist&address=$WALLET&page=1&offset=$LIMIT&sort=desc" \
  | jq -r '.result[] | "[\(.timeStamp | tonumber | strftime("%Y-%m-%d %H:%M"))] \(.hash[0:18])... | \(.from[0:10])... → \(.to[0:10])... | \(.value | tonumber / 1e18 | . * 10000 | round / 10000) PHRS | Status: \(if .isError == "0" then "✓ Success" else "✗ Failed" end)"'
```

**Output format (per transaction):**
```
[2026-06-01 14:22] 0x3f8a91bc2d... | 0xSender... → 0xReceiver... | 0.05 PHRS | Status: ✓ Success
```

---

### 5. Wallet Health Report

Generate a full health report with balance status, tx activity, and recommendations.

**Example prompt:** *"Give me a health report for my wallet 0xABC... on Pharos"*

Agent should:
1. Run native balance check → classify health tier
2. Query ERC-20 balances for known tokens
3. Fetch last 10 transactions → count successes vs failures
4. Calculate failure rate
5. Output a structured report:

```
=====================================
 PHAROS WALLET HEALTH REPORT
=====================================
Wallet:  0xABC...
Network: Atlantic Testnet
Time:    2026-06-07 12:00 UTC

BALANCES
--------
PHRS:    1.2340   [HEALTHY]
USDC:    45.00

ACTIVITY (Last 10 tx)
---------------------
Successes:  9
Failures:   1
Fail Rate:  10%
Status:     [NORMAL]

RECOMMENDATIONS
---------------
• Balance is healthy — no action needed
• 1 failed transaction detected — review tx 0x...
• Explorer: https://testnet.pharosscan.xyz/address/0xABC...
=====================================
```

---

### 6. Multi-Wallet Comparison

Compare balances across multiple wallets side by side.

**Example prompt:** *"Compare wallets 0xA..., 0xB..., 0xC... on Pharos testnet"*

```bash
RPC_URL=$(jq -r '.networks[] | select(.name=="atlantic-testnet") | .rpcUrl' assets/networks.json)
NATIVE=$(jq -r '.networks[] | select(.name=="atlantic-testnet") | .nativeToken' assets/networks.json)

WALLETS=("0xA..." "0xB..." "0xC...")

printf "%-45s %-15s\n" "WALLET" "$NATIVE BALANCE"
printf "%-45s %-15s\n" "------" "--------------"

for WALLET in "${WALLETS[@]}"; do
  BAL_WEI=$(cast balance $WALLET --rpc-url $RPC_URL)
  BAL_ETH=$(cast from-wei $BAL_WEI)
  printf "%-45s %-15s\n" "$WALLET" "$BAL_ETH"
done
```

---

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| `cast: command not found` | Foundry not installed | Run install commands above |
| `invalid address` | Wrong wallet format | Check address starts with 0x + 40 hex chars |
| `connection refused` | RPC down | Retry or use alternate RPC |
| `0x` balance response | Token contract wrong address | Verify ERC-20 contract address on explorer |
| `jq: command not found` | jq missing | Run: `sudo apt-get install jq` |

---

## Security Notes

- Never log or expose private keys in output
- Read-only operations in this skill require NO private key
- Always show the explorer link so users can independently verify data
- On mainnet queries, prefix output with `[MAINNET]` warning

---

## Faucet (Testnet Only)

If wallet balance is critical on testnet, direct user to:
`https://testnet.pharosnetwork.xyz` — claim up to 0.01 PHRS every 12 hours
