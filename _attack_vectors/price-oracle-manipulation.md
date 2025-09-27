---
layout: default
title: "Price Oracle Manipulation"
---

# Price Oracle Manipulation

| How tricking a protocol about an asset's price can allow an attacker to drain its vaults.

## What is Price Oracle Manipulation?

Price Oracle Manipulation is an attack where a malicious actor intentionally distorts the price data fed to a smart contract. DeFi protocols rely on oracles to determine the value of assets for critical operations like lending, borrowing, and liquidations. If an attacker can manipulate this price feed, they can trick the protocol into making catastrophic financial decisions, such as accepting worthless collateral at an inflated value or selling assets for a fraction of their worth.

The vulnerability stems from a protocol's choice of oracle. Many exploits target oracles that are susceptible to short-term manipulation, most commonly those that rely on the spot price from a single, on-chain source like a DEX liquidity pool.

### Real-World Impact: The WOOFi Exploit

The WOOFi decentralized exchange used its own internal pricing mechanism, which calculated the spot price based on pool liquidity. This mechanism was not time-weighted and was designed to fall back to a Chainlink oracle in cases of extreme price deviation. However, for one low-liquidity market on Arbitrum, the Chainlink fallback was not properly configured. An attacker exploited this by using a series of flash loans and trades to momentarily crash the asset's price within the WOOFi pool. Because the fallback failed, the protocol accepted the manipulated, near-zero price as legitimate, allowing the attacker to drain the pool of its assets for pennies on the dollar.

## How the Attack Works

Oracle manipulation attacks almost always involve an attacker creating a temporary, artificial price for an asset and getting a victim protocol to act on it within a single atomic transaction.

1.**Spot Price Manipulation via Flash Loan**

This is the most common and classic oracle attack. It targets protocols that use the instantaneous price from a DEX (like a Uniswap V2 pool) as their oracle.

* **Vulnerable Oracle Logic**: A lending protocol reads the ratio of tokenA/tokenB in a liquidity pool to determine the price of tokenA.

* **Attacker's Playbook**:

    - **Flash Loan**: The attacker borrows a massive amount of tokenB from a lending protocol like Aave.

    - **Distort the Pool**: They swap the borrowed tokenB for tokenA in the target liquidity pool. This floods the pool with tokenB and drains tokenA, causing the spot price of tokenA to skyrocket.

    - **Exploit the Victim Protocol**: The attacker goes to the victim lending protocol. They deposit a small amount of their now "super-valuable" tokenA as collateral and use it to borrow out all of the protocol's valuable assets (like ETH or stablecoins).

    - **Repay and Profit**: The attacker swaps the assets back in the DEX to restore the original price, repays the flash loan, and walks away with the stolen funds. The HYDT and PolterFinance exploits followed this exact pattern.

2. **Low-Liquidity Asset Manipulation**

This attack is similar to the flash loan vector but doesn't always require one. If an asset used as collateral has very thin liquidity, an attacker can use their own capital to buy up most of the supply in a DEX pool, artificially inflating its price. They then use this overvalued collateral to borrow against, as seen in the PolterFinance hack where even averaging two spot prices was not enough to prevent manipulation.

3. **Exploiting Flawed Pricing Formulas**

Some protocols use custom, non-standard pricing formulas instead of relying on market-based oracles. If these formulas have mathematical flaws, they can be exploited.

**The AIZPT314 Exploit**: This protocol used a custom formula that was not a constant-product AMM. An attacker discovered that by making a large initial trade and then trading back small amounts, they could extract more value than they put in, effectively draining the contract due to the flawed math.

## How to Prevent It

Preventing oracle manipulation requires a robust, multi-layered approach to sourcing price data. Never trust a single, easily manipulated price feed.

1. **Do Not Use On-Chain DEX Spot Prices as an Oracle**

This is the cardinal rule. The spot price from a single DEX pool is the most vulnerable type of price feed and should never be used directly for critical functions. It is trivial to manipulate within a single transaction using a flash loan.

2. **Use Time-Weighted Average Price (TWAP) Oracles**

A TWAP oracle calculates the average price of an asset over a period of time (e.g., 30 minutes), rather than its instantaneous price. This makes flash loan manipulation impossible, as an attacker cannot sustain a distorted price for the entire time window.

* **Implementation**: Uniswap V2 and V3 offer built-in functionality for creating TWAP oracles. However, they require careful implementation to be secure and can be complex to manage.

3. **Use Decentralized Oracle Networks (DONs)**

This is the industry standard and most secure approach. Services like Chainlink provide decentralized price feeds that are aggregated from numerous high-quality, off-chain data sources and on-chain exchanges.

**Resilience**: DONs are resistant to single points of failure and manipulation. An attacker would need to compromise multiple independent data sources simultaneously, which is economically and practically infeasible.

Liveness and Accuracy: They provide reliable updates even during high network congestion or volatility.

4. **Implement Circuit Breakers and Sanity Checks**

As a defense-in-depth measure, your protocol should have internal safety checks.

**Price Deviation Checks**: If a new price from your oracle deviates by more than a certain percentage (e.g., 10%) from the last reported price, the protocol should pause operations or revert the transaction. This can act as a backstop against a sudden, catastrophic price swing.

**Redundant Oracles**: For critical assets, consider sourcing prices from two independent oracles (e.g., Chainlink and another DON). If their prices diverge significantly, pause the system and require manual intervention. This is what WOOFi intended to do but failed to configure correctly.

5. **For Low-Liquidity Assets** 

If your protocol must support a token that has no robust oracle feed and low on-chain liquidity, extreme caution is required.

**Collateral Factor**: Assign a very low collateral factor to the asset, limiting how much can be borrowed against it.

**Debt Ceilings**: Impose a hard cap on the total amount of debt that can be created using that specific asset as collateral. This limits the total potential loss from a manipulation attack.

**Governance Control**: Ultimately, the decision to list a volatile, low-liquidity asset is a risk management decision that should be handled by protocol governance with a full understanding of the potential dangers.