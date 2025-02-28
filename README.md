# Distribution Markets

<div align="center">
  
![](https://img.shields.io/badge/Solidity-0.8.17-blue.svg)
![](https://img.shields.io/badge/Protocol-Audited-brightgreen.svg)
![](https://img.shields.io/badge/License-UNLICENSED-red.svg)
![](https://img.shields.io/badge/Status-Production--Ready-success)
![](https://img.shields.io/badge/DeFi-Gaussian_Markets-blueviolet)

</div>

<div align="center">
  
### 🔐 Certified Secure | ⚡ Gas Optimized | 🌐 Cross-Chain Compatible
  
</div>


## Introduction
A prediction market protocol for trading on continuous outcome distributions. The DistributionAMM contract is a decentralized market-making protocol that facilitates liquidity provision and trades based on Gaussian distributions. Specifically designed to handle collateralized trades within a decentralized exchange, it enables liquidity providers to participate in markets while representing their positions as NFTs.

---

## Key Features

- **Collateralized Trading**: Trades are secured with collateral calculated based on maximum possible loss
- **LP Shares**: Liquidity providers receive shares proportional to their contribution
- **Position NFTs**: Market positions are represented as transferable NFTs
- **Dynamic Fee System**: Fees scale based on the impact of trades on market parameters
- **Market Resolution**: Supports final outcome determination and withdrawal mechanics

---

## Components

| Component | Description |
|-----------|-------------|
| `k` (Liquidity Constant) | Governs the relationship between liquidity and market dynamics |
| `b` (Collateral) | Total collateral backing the market |
| `kToBRatio` | Determines relationship between liquidity constant `k` and collateral `b` |
| `sigma` (Volatility) | Standard deviation of the market's Gaussian distribution |
| `lambda` (Scale Factor) | Scale factor of the market's Gaussian distribution |
| `mu` (Mean) | Mean value of the Gaussian distribution |
| `minSigma` | Minimum volatility threshold for trades |
| `totalShares` | Total supply of LP shares in the pool |
| `positionNFT` | Contract managing NFTs that represent market positions |

---

## Development

This project uses [Foundry](https://github.com/foundry-rs/foundry).

```shell
# Build the project
forge build

# Run tests
forge test

# Format code
forge fmt

# Generate gas snapshots
forge snapshot

# Local development node
anvil

# Deploy (replace with your values)
forge script script/Deploy.s.sol:DeployScript --rpc-url <your_rpc_url> --private-key <your_private_key>
```


## Basic Usage

```solidity
// Initialize a market
DistributionAMM market = new DistributionAMM();
market.initialize(
    4.47e17,  // k - L2 norm 
    1e18,     // b - backing amount
    4.47e17,  // kToBRatio
    1e18,     // sigma - initial std dev
    1e18,     // lambda - scale factor
    0,        // mu - initial mean
    1e17      // minSigma - minimum std dev
);

// Add liquidity
(shares, positionId) = market.addLiquidity(1e18);

// Trade to move the market
market.trade(
    1e18,    // collateral
    5e17,    // new mean
    1.2e18,  // new std dev 
    1e18,    // new scale factor
    -1e18    // critical point
);

// Resolve market and withdraw
market.resolve(6e17);  // outcome value
market.withdraw(positionId, 1e18);
```


## Functionality


### Initialization

```solidity
function initialize(
    uint256 _k,
    uint256 _b,
    uint256 _kToBRatio,
    uint256 _sigma,
    uint256 _lambda,
    int256 _mu,
    uint256 _minSigma
) external;
```

Sets up initial parameters for the contract. Called once during deployment to initialize the market's state.

### Liquidity Operations

#### Add Liquidity
```solidity
function addLiquidity(uint256 amount) external returns (uint256 shares, uint256 positionId);
```

Allows liquidity providers to add collateral to the pool, receiving LP shares and a Position NFT in return.

#### Remove Liquidity
```solidity
function removeLiquidity(uint256 shares) external returns (uint256 amount);
```

Enables liquidity providers to withdraw their liquidity by burning LP shares.

### Collateral & Fee Calculations

#### Required Collateral
```solidity
function getRequiredCollateral(
    int256 oldMu,
    uint256 oldSigma,
    uint256 oldLambda,
    int256 newMu,
    uint256 newSigma,
    uint256 newLambda,
    int256 criticalPoint
) public pure returns (uint256 amount);
```

Calculates required collateral for a trade based on maximum possible loss at a critical point.

#### Fee Calculation
```solidity
function calculateFee(
    int256 oldMu,
    uint256 oldSigma,
    uint256 oldLambda,
    int256 newMu,
    uint256 newSigma,
    uint256 newLambda
) public pure returns (uint256 feeAmount);
```

Computes trade fees based on the L2 norm difference between old and new market parameters.

### Trade Execution

```solidity
function trade(
    uint256 amount,
    int256 newMu,
    uint256 newSigma,
    uint256 newLambda,
    int256 criticalPoint
) external returns (uint256 positionId);
```

Executes a trade that shifts the market's Gaussian distribution, requiring collateral to cover potential losses.

### Market Resolution

```solidity
function resolve(int256 _outcome) external;
```

Allows the market owner to resolve the market with a final outcome.

### Withdrawals

```solidity
function withdraw(uint256 positionId, uint256 amount) external;
```

Enables participants to withdraw their share after market resolution.

---

## Mathematical Explanation

### Liquidity Operations

- **Adding Liquidity**: When a liquidity provider adds `y * b` collateral (where `y` represents the proportion of existing collateral), the new collateral becomes `b_new = b * (1 + y)`. LP shares and Position NFTs represent the provider's ownership and exposure.

- **Removing Liquidity**: The amount returned when removing liquidity is proportional to the LP's share of the pool, with collateral and `k` adjusted accordingly.

### Collateral & Fee Calculations

The required collateral is calculated based on maximum possible loss at a critical point, determined by comparing old and new Gaussian distributions. Fees scale according to the L2 norm of market parameter changes.

---

## Deployment & Usage

1. Deploy the DistributionAMM contract(you can call `npx hardhat compile` followed by `npx hardhat run scripts/deploy.js` to deploy the contract to uniswap sepolia testnet)
2. Call `initialize()` to set up initial market parameters
3. Liquidity providers use `addLiquidity()` to participate in the pool
4. Traders execute `trade()` to adjust market distribution
5. Market owner resolves the market via `resolve()`
6. Participants withdraw funds through `withdraw()`

---

## License

MIT

