# pharos-wallet-sentinel

> A Pharos Agent Skill — built for the Pharos Skill Builder Campaign by DingiDinigi

Monitor, audit, and analyze any wallet on Pharos Network. Check native and ERC-20 balances, view transaction history, compare multiple wallets, and generate full health reports — all through natural language with your AI agent.

---

## What This Skill Does

| Capability | Example Prompt |
|---|---|
| Native balance check | "Check the PHRS balance of 0xABC..." |
| ERC-20 token balance | "How much USDC does 0xABC... hold?" |
| Portfolio overview | "Show full portfolio for 0xABC... on Pharos" |
| Transaction history | "Show last 10 txs for 0xABC..." |
| Wallet health report | "Give me a health report for 0xABC..." |
| Multi-wallet compare | "Compare wallets 0xA..., 0xB..., 0xC..." |

Works on both **Atlantic Testnet** (Chain ID: 688688) and **Pacific Mainnet** (Chain ID: 1672).

---

## Installation Guide (Beginner Friendly)

### Step 1 — Install Foundry

Foundry gives your agent the `cast` command it needs to talk to the blockchain.

Open your terminal and run:

```bash
curl -L https://foundry.paradigm.xyz | bash
```

Then run:

```bash
source ~/.zshenv && foundryup
```

Confirm it worked:

```bash
cast --version
# Should print something like: cast 0.3.x ...
```

If you're on Windows, use WSL (Windows Subsystem for Linux) and run the same commands inside it.

### Step 2 — Install jq

`jq` is a tool for reading JSON files (used to load network config).

**Ubuntu/Debian/WSL:**
```bash
sudo apt-get install -y jq
```

**macOS:**
```bash
brew install jq
```

**Verify:**
```bash
jq --version
```

### Step 3 — Install the Skill

**For Claude Code:**

```bash
# Download the skill file
mkdir -p ~/.claude/skills
curl -L https://raw.githubusercontent.com/DingiDinigi/pharos-wallet-sentinel/main/SKILL.md \
  -o ~/.claude/skills/pharos-wallet-sentinel.md

# Download network config
mkdir -p ~/.claude/skills/assets
curl -L https://raw.githubusercontent.com/DingiDinigi/pharos-wallet-sentinel/main/assets/networks.json \
  -o ~/.claude/skills/assets/networks.json
```

**For Codex CLI:**

```bash
mkdir -p ~/.codex/skills/assets
curl -L https://raw.githubusercontent.com/DingiDinigi/pharos-wallet-sentinel/main/SKILL.md \
  -o ~/.codex/skills/pharos-wallet-sentinel.md
curl -L https://raw.githubusercontent.com/DingiDinigi/pharos-wallet-sentinel/main/assets/networks.json \
  -o ~/.codex/skills/assets/networks.json
```

**For OpenClaw:**

```bash
npx skills add https://github.com/DingiDinigi/pharos-wallet-sentinel
```

### Step 4 — Verify Installation

**Claude Code:** Type `/skills` in a new session — you should see `pharos-wallet-sentinel` listed.

**Codex:** Type `/skills` — same.

---

## How to Use

Once installed, just talk to your agent naturally:

```
You: Check the PHRS balance of 0x1234...abcd on Pharos testnet

Agent: Using pharos-wallet-sentinel...
       Wallet: 0x1234...abcd
       Balance: 2.4500 PHRS [HEALTHY]
       Network: Atlantic Testnet
       Explorer: https://testnet.pharosscan.xyz/address/0x1234...abcd
```

```
You: Show me a health report for 0x1234...abcd on Pharos mainnet

Agent: [MAINNET] Generating health report...
       Balance: 0.05 PROS [LOW - consider topping up]
       Last 10 txs: 8 success, 2 failed (20% fail rate)
       Recommendation: Review failed transactions
```

---

## Network Details

| Network | Chain ID | RPC | Explorer |
|---|---|---|---|
| Atlantic Testnet | 688688 | https://atlantic.dplabs-internal.com | https://testnet.pharosscan.xyz |
| Pacific Mainnet | 1672 | https://rpc.pharos.xyz | https://pharosscan.xyz |

---

## Dependencies

- [Foundry](https://getfoundry.sh) — `cast` binary for blockchain queries
- [jq](https://stedolan.github.io/jq/) — JSON parsing for network config
- No private key required for read operations

---

## Supported Frameworks

- Claude Code
- Codex CLI  
- OpenClaw

---

## License

MIT — Free to use, modify, and redistribute.

---

## Author

Built by **DingiDinigi** for the Pharos Agent Centre Skill Builder Campaign.

GitHub: [https://github.com/DingiDinigi/pharos-wallet-sentinel](https://github.com/DingiDinigi/pharos-wallet-sentinel)
