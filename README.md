# ⛏ PYRATHOS — Mining Ecosystem on Monad Mainnet

> A fully on-chain Bitcoin-inspired mining game. Mint ASTRA-CORE miner NFTs, stake them to earn PYRATHOS tokens, upgrade your rigs, and watch rewards halve every ~1 year — all deployed on Monad Mainnet.

[![Monad](https://img.shields.io/badge/Network-Monad%20Mainnet-7c3aed?style=flat-square)](https://monad.xyz)
[![Solidity](https://img.shields.io/badge/Solidity-0.8.20-a855f7?style=flat-square)](https://soliditylang.org)
[![License](https://img.shields.io/badge/License-MIT-22d3ee?style=flat-square)](LICENSE)

---

## 🌌 Live Dashboard

**[→ Open Mining Dashboard](https://YOUR_USERNAME.github.io/pyrathos-mining/)**

Connect MetaMask → Monad Mainnet → Mint, Stake, Earn.

---

## 📦 Deployed Contracts

| Contract | Address | Purpose |
|---|---|---|
| **PyrathosToken** | `0x76444dEEf12a2fC0e2C7FeFf77829fa802380E48` | ERC-20, max 21M supply |
| **HalvingEngine** | `0x101a36482668984e23720c552b0b78d309c57BF3` | Reward halving logic |
| **AstraCoreMiner** | `0x0A4242458782e49bfaAa8f785331c58e2568608a` | ERC-721 miner NFTs |
| **MiningPool** | `0xFB4c669890d4664c026531F98eEb1365fadf38f1` | Staking & reward engine |
| **UpgradeSystem** | `0x8F200edb60D3f6F41dA2fFA8aeb70be6fB2c0671` | 10-level miner upgrades |

**Chain ID:** 143 | **RPC:** `https://monad-mainnet.g.alchemy.com/v2/...`

---

## 🗂 Repository Structure

```
pyrathos-mining/
├── index.html                        # Mining dashboard (GitHub Pages)
├── src/
│   ├── PyrathosToken.sol             # ERC-20 token contract
│   ├── HalvingEngine.sol             # Halving reward logic
│   ├── AstraCoreMiner.sol            # ERC-721 NFT with on-chain SVG art
│   ├── MiningPool.sol                # Stake miners → earn PYRATHOS
│   └── UpgradeSystem.sol             # Burn PYRATHOS to upgrade miners
├── script/
│   └── DeployPyrathos.s.sol          # Foundry deploy script
├── foundry.toml                      # Foundry config
└── README.md
```

---

## ⚙️ Contract Functions

### PyrathosToken (`ERC-20`)

| Function | Type | Description |
|---|---|---|
| `mine(address to, uint256 amount)` | `external` | Mint PYRATHOS — only callable by MiningPool |
| `burn(uint256 amount)` | `external` | Burn tokens from your own balance |
| `burnFrom(address account, uint256 amount)` | `external` | Burn tokens from approved address |
| `setMinter(address _minter)` | `onlyOwner` | Set the address allowed to mint (MiningPool) |
| `remainingSupply()` | `view` | Returns `MAX_SUPPLY - totalSupply()` |
| `MAX_SUPPLY()` | `view` | Returns `21,000,000 * 1e18` |
| `minter()` | `view` | Returns current minter address |
| `balanceOf(address)` | `view` | ERC-20 standard — token balance |
| `transfer / transferFrom / approve` | — | Standard ERC-20 functions |

---

### HalvingEngine

| Function | Type | Description |
|---|---|---|
| `currentHalving()` | `view` | Returns current halving era number (0 = genesis) |
| `currentRate()` | `view` | Returns current reward rate in wei/GH/s/s (halves each era) |
| `currentEra()` | `view` | Alias for `currentHalving()` |
| `blocksUntilNextHalving()` | `view` | Returns blocks remaining until next halving |
| `setBaseRewardRate(uint256 _r)` | `onlyOwner` | Update the base reward rate |
| `setHalvingInterval(uint256 _i)` | `onlyOwner` | Update blocks per halving interval |
| `resetGenesis()` | `onlyOwner` | Reset genesis block to current block |
| `genesisBlock()` | `view` | Block number when mining started |
| `halvingInterval()` | `view` | Blocks per era (default: 525,600 ≈ 1 year) |
| `baseRewardRate()` | `view` | Current base rate before halving |
| `MAX_HALVINGS()` | `view` | Maximum halvings (32 — then rewards = 0) |

---

### AstraCoreMiner (`ERC-721`)

**Variants**

| ID | Name | Base Hashrate | Price |
|---|---|---|---|
| 0 | STELLAR-I | 500 GH/s | 100 MON |
| 1 | STELLAR-II | 2,000 GH/s | 500 MON |
| 2 | NOVA-X | 8,000 GH/s | 1,000 MON |
| 3 | QUANTUM-CORE | 25,000 GH/s | 10,000 MON |

**Mint Functions**

| Function | Type | Description |
|---|---|---|
| `mint(uint8 variant)` | `payable` | Mint a miner NFT — send exact MON price |
| `ownerMint(address to, uint8 variant, uint256 qty)` | `onlyOwner` | Owner mint without payment |

**Read Functions**

| Function | Type | Description |
|---|---|---|
| `getMiner(uint256 tokenId)` | `view` | Returns full `MinerData` struct |
| `getCurrentCondition(uint256 tokenId)` | `view` | Returns current condition (0–100), degrades over time |
| `getEffectiveHashrate(uint256 tokenId)` | `view` | Returns hashrate adjusted by condition |
| `tokenURI(uint256 tokenId)` | `view` | Returns base64 JSON with on-chain SVG art |
| `totalSupply()` | `view` | Total miners minted |
| `balanceOf(address owner)` | `view` | NFTs owned by address |
| `ownerOf(uint256 tokenId)` | `view` | Owner of a specific token |

**Write Functions**

| Function | Type | Description |
|---|---|---|
| `applyDegradation(uint256 tokenId)` | `external` | Sync on-chain condition with time elapsed |
| `performMaintenance(uint256 tokenId)` | `external` | Restore condition to 100% — only MiningPool/UpgradeSystem |
| `applyUpgrade(uint256 tokenId, uint8 newLevel, uint32 newHashrate)` | `external` | Apply upgrade — only UpgradeSystem |
| `approve(address to, uint256 tokenId)` | `external` | ERC-721 approve for staking |
| `setApprovalForAll(address operator, bool approved)` | `external` | Approve all tokens |

**Admin Functions**

| Function | Type | Description |
|---|---|---|
| `setMintOpen(bool o)` | `onlyOwner` | Open/close public minting |
| `setMiningPool(address a)` | `onlyOwner` | Set MiningPool address |
| `setUpgradeSystem(address a)` | `onlyOwner` | Set UpgradeSystem address |
| `setVariantPrice(uint8 v, uint256 p)` | `onlyOwner` | Update price for a variant |
| `withdrawMON()` | `onlyOwner` | Withdraw mint proceeds to owner |

**MinerData Struct**

```solidity
struct MinerData {
    uint8   variant;          // 0–3 (STELLAR-I to QUANTUM-CORE)
    uint32  baseHashrate;     // Original base GH/s
    uint32  hashrate;         // Current GH/s (after upgrades)
    uint8   level;            // 1–10
    uint8   condition;        // 0–100 (degrades 2%/day after 7 days unmaintained)
    uint40  mintedAt;         // Timestamp of mint
    uint40  lastMaintenance;  // Timestamp of last maintenance
    bytes32 seed;             // Random seed for art generation
}
```

---

### MiningPool

**Staking**

| Function | Type | Description |
|---|---|---|
| `stake(uint256 tokenId)` | `external` | Stake one miner (approve NFT first) |
| `stakeBatch(uint256[] tokenIds)` | `external` | Stake multiple miners at once |
| `unstake(uint256 tokenId)` | `external` | Unstake miner + auto-claim pending rewards |
| `claim(uint256 tokenId)` | `external` | Claim pending rewards for one miner |
| `claimAll()` | `external` | Claim all pending rewards for all staked miners |
| `payMaintenance(uint256 tokenId)` | `external` | Pay PYRATHOS to restore miner condition |

**Read Functions**

| Function | Type | Description |
|---|---|---|
| `pendingReward(uint256 tokenId)` | `view` | Pending PYRATHOS for a specific miner |
| `totalPending(address player)` | `view` | Total pending across all staked miners |
| `getPoolBonus(address player)` | `view` | Current multiplier (100–200 = 1.0x–2.0x) |
| `getStakedMiners(address player)` | `view` | Array of staked token IDs |
| `getStakedCount(address player)` | `view` | Number of staked miners |
| `getStakeInfo(uint256 tokenId)` | `view` | Full `StakeInfo` struct for a miner |
| `totalStaked()` | `view` | Total miners staked globally |
| `globalMined()` | `view` | Total PYRATHOS minted via pool |
| `totalMined(address player)` | `view` | Total PYRATHOS earned by a player |

**Pool Bonus Tiers**

| Miners Staked | Multiplier |
|---|---|
| 1–2 | 1.00x |
| 3–4 | 1.10x |
| 5–9 | 1.25x |
| 10–24 | 1.50x |
| 25–49 | 1.75x |
| 50+ | 2.00x |

**Reward Formula**

```
reward = (hashrate × elapsed_seconds × currentRate × poolBonus) / (1e18 × 100)
```

**Admin Functions**

| Function | Type | Description |
|---|---|---|
| `setPoolBonusThreshold(uint256 i, uint256 v)` | `onlyOwner` | Update bonus threshold |
| `setPoolBonusValue(uint256 i, uint256 v)` | `onlyOwner` | Update bonus multiplier value |
| `setPyrathosToken(address a)` | `onlyOwner` | Update PYRATHOS token address |
| `setHalvingEngine(address a)` | `onlyOwner` | Update HalvingEngine address |
| `setMinerNFT(address a)` | `onlyOwner` | Update AstraCoreMiner address |

---

### UpgradeSystem

**Upgrade Tiers**

| From Level | PYRATHOS Cost | MON Cost | Hashrate Bonus | Success Rate | Can Demote |
|---|---|---|---|---|---|
| 1 → 2 | 500 | 0 | +200 GH/s | 100% | No |
| 2 → 3 | 1,500 | 0 | +400 GH/s | 95% | No |
| 3 → 4 | 4,000 | 0.01 MON | +700 GH/s | 90% | No |
| 4 → 5 | 8,000 | 0.02 MON | +1,000 GH/s | 85% | No |
| 5 → 6 | 15,000 | 0.05 MON | +1,500 GH/s | 80% | No |
| 6 → 7 | 25,000 | 0.1 MON | +2,000 GH/s | 75% | Yes |
| 7 → 8 | 40,000 | 0.2 MON | +3,000 GH/s | 70% | Yes |
| 8 → 9 | 65,000 | 0.4 MON | +4,500 GH/s | 65% | Yes |
| 9 → 10 | 100,000 | 0.8 MON | +7,000 GH/s | 60% | Yes |

> At levels 6–10, a failed upgrade has a 50% chance of demoting the miner by one level.

| Function | Type | Description |
|---|---|---|
| `upgrade(uint256 tokenId)` | `payable` | Attempt to upgrade miner (burns PYRATHOS + optional MON) |
| `getUpgradeCost(uint256 tokenId)` | `view` | Returns cost, MON required, success rate, canDemote for next upgrade |
| `setUpgradeTier(uint8 fromLevel, UpgradeTier tier)` | `onlyOwner` | Update an upgrade tier |
| `withdrawMON()` | `onlyOwner` | Withdraw MON upgrade fees |

---

## 🚀 How to Deploy (Foundry)

```bash
# Clone and install
git clone https://github.com/YOUR_USERNAME/pyrathos-mining
cd pyrathos-mining
forge install

# Set private key
echo "PRIVATE_KEY=0xYOUR_KEY" > .env
source .env

# Build
forge build

# Deploy all contracts
forge script script/DeployPyrathos.s.sol \
  --rpc-url monad \
  --private-key $PRIVATE_KEY \
  --broadcast

# Verify contracts
forge verify-contract \
  --rpc-url https://rpc.monad.xyz \
  --verifier sourcify \
  --verifier-url 'https://sourcify-api-monad.blockvision.org/' \
  --chain-id 143 --via-ir --compiler-version 0.8.20 \
  ADDRESS src/CONTRACT.sol:CONTRACT
```

**foundry.toml**
```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc = "0.8.20"
via_ir = true
optimizer = true
optimizer_runs = 200

[rpc_endpoints]
monad = "https://monad-mainnet.g.alchemy.com/v2/YOUR_KEY"
```

---

## 🔧 Common Cast Commands

```bash
# Check PYRATHOS balance
cast call 0x76444dEEf12a2fC0e2C7FeFf77829fa802380E48 \
  "balanceOf(address)(uint256)" YOUR_ADDRESS --rpc-url monad

# Check pending rewards for miner #1
cast call 0xFB4c669890d4664c026531F98eEb1365fadf38f1 \
  "pendingReward(uint256)(uint256)" 1 --rpc-url monad

# Check current halving era
cast call 0x101a36482668984e23720c552b0b78d309c57BF3 \
  "currentEra()(uint256)" --rpc-url monad

# Check blocks until next halving
cast call 0x101a36482668984e23720c552b0b78d309c57BF3 \
  "blocksUntilNextHalving()(uint256)" --rpc-url monad

# Approve miner for staking
cast send 0x0A4242458782e49bfaAa8f785331c58e2568608a \
  "approve(address,uint256)" \
  0xFB4c669890d4664c026531F98eEb1365fadf38f1 TOKEN_ID \
  --rpc-url monad --private-key $PRIVATE_KEY

# Stake a miner
cast send 0xFB4c669890d4664c026531F98eEb1365fadf38f1 \
  "stake(uint256)" TOKEN_ID \
  --rpc-url monad --private-key $PRIVATE_KEY

# Claim rewards
cast send 0xFB4c669890d4664c026531F98eEb1365fadf38f1 \
  "claim(uint256)" TOKEN_ID \
  --rpc-url monad --private-key $PRIVATE_KEY

# Unstake (auto-claims rewards)
cast send 0xFB4c669890d4664c026531F98eEb1365fadf38f1 \
  "unstake(uint256)" TOKEN_ID \
  --rpc-url monad --private-key $PRIVATE_KEY

# View miner SVG art in browser (copy output → paste in browser address bar)
cast call 0x0A4242458782e49bfaAa8f785331c58e2568608a \
  "tokenURI(uint256)(string)" TOKEN_ID --rpc-url monad

# Update reward rate
cast send 0x101a36482668984e23720c552b0b78d309c57BF3 \
  "setBaseRewardRate(uint256)" RATE \
  --rpc-url monad --private-key $PRIVATE_KEY
```

---

## 🎮 How It Works

```
1. MINT     — Buy an ASTRA-CORE miner NFT (100–10,000 MON)
               Each miner has randomized hashrate (±20% from base)

2. STAKE    — Deposit miner into MiningPool
               Rewards start accruing immediately

3. EARN     — reward = hashrate × time × halvingRate × poolBonus
               Pool bonus up to 2x for 50+ staked miners

4. MAINTAIN — Condition degrades 2%/day after 7 days unmaintained
               Pay PYRATHOS to restore to 100%

5. UPGRADE  — Burn PYRATHOS + optional MON to boost hashrate
               10 levels — higher levels have risk of demotion

6. HALVING  — Every 525,600 blocks (~1 year), rewards halve
               32 halvings max, then mining ends
               Max supply: 21,000,000 PYRATHOS
```

---

## 🖼 On-Chain Art

All ASTRA-CORE miner NFTs are fully on-chain. The `tokenURI` function returns a base64-encoded JSON with an embedded SVG image — no IPFS, no centralized servers.

Each variant has a unique color scheme:
- **STELLAR-I** — Cyan (`#00ffe7`)
- **STELLAR-II** — Purple (`#aa44ff`)
- **NOVA-X** — Orange (`#ff6a00`)
- **QUANTUM-CORE** — Yellow (`#f5ff00`)

The SVG includes animated blinking lights, a hashrate display, condition bar, and level indicator — all generated from on-chain data.

---

## 🔐 Security Notes

- `PyrathosToken.mine()` is restricted to `minter` address (MiningPool only)
- `performMaintenance()` is restricted to MiningPool and UpgradeSystem
- `applyUpgrade()` is restricted to UpgradeSystem only
- All staking/unstaking/claiming uses `ReentrancyGuard`
- Upgrade randomness uses `block.prevrandao` — appropriate for game use

---

## 📄 License

MIT — see [LICENSE](LICENSE)

---

*Built on [Monad Mainnet](https://monad.xyz) · Solidity 0.8.20 · Foundry*
