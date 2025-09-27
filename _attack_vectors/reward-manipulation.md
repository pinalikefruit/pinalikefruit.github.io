---
layout: default
title: "Reward Manipulation"
---

# Reward Manipulation

| Gaming the system to claim more than your fair share.

## What is Reward Manipulation?

Reward Manipulation is an attack that exploits flawed logic in a protocol's incentive mechanism. Instead of targeting a vault directly, attackers abuse the rules for distributing rewards—like staking yield, airdrops, or protocol fees—to claim a massively inflated or unearned share of the value.

The vulnerability stems from a protocol's failure to accurately measure a user's true economic contribution over time. Often, the reward calculation relies on a simple, instantaneous check (like balanceOf()) which can be easily gamed. An attacker can use a flash loan to temporarily acquire a huge stake, claim rewards meant for long-term holders, and then return the loan, all within a single transaction, having risked nothing.

This isn't about breaking the vault door; it's about tricking the cashier into handing over the keys to the rewards chest.

### Real-World Impact: The New Free Dao Airdrop

A classic example of this attack involved the New Free Dao airdrop contract. The protocol's goal was simple: reward existing token holders. However, its implementation was fatally flawed.

- **The Flaw**: The airdrop contract checked a user's token balance at the exact moment they called the claim() function. It did not care how long the user had held those tokens.

- **The Exploit**: An attacker executed the following steps in a single atomic transaction:

        - Took a massive flash loan of the DAO's governance token from a decentralized exchange.

        - Called the claim() function on the airdrop contract. The contract saw the attacker's huge (but temporary) balance and allocated them a disproportionately large share of the airdrop rewards.

        - Immediately repaid the flash loan.

* **The Aftermath**: The attacker walked away with a significant portion of the total airdrop supply, draining value intended for the genuine, long-term community members.

## How the Attack Works

Reward manipulation attacks prey on faulty assumptions in a protocol's accounting logic. The most common vectors are:

1. **Instantaneous Balance Exploits**

This is the most common pattern, as seen in the New Free Dao hack. The protocol checks a user's balance at the moment of a claim, making it vulnerable to flash loans.

```sol
Vulnerable Logic:rewardAmount = user.balance * rewardRate;
```

An attacker can artificially inflate their user.balance for a single block to maximize rewardAmount.

2. **Time-Independent Rewards**

The protocol calculates rewards based on the amount staked but fails to factor in the duration of the stake. This allows an attacker to deposit, claim, and withdraw in the same transaction, earning rewards that should have taken days or weeks to accrue.

```sol
function claimReward() external {
    uint256 stakedAmount = stakingToken.balanceOf(msg.sender);
    // VULNERABILITY: Reward is not proportional to time.
    // A user who staked 1 second ago gets the same reward as a user who staked for 1 month.
    uint256 reward = calculateReward(stakedAmount); 
    
    rewardToken.transfer(msg.sender, reward);
}
```



The MooCakeCTX exploit followed this pattern. The attacker used a flash loan to deposit, claim the reward immediately, and withdraw, all before the next block.

3. **Flawed Reward Accounting**

The contract's internal logic for tracking who has claimed what is broken. This can lead to double-spending or infinite reward claims.

* **Additive Rewards**: As seen in the Dark Pool exploit, the logic might incorrectly add a new reward on top of an existing one instead of updating it, allowing rewards to pile up unfairly.

* **Failure to Track Claims**: The Ramses Exchange exploit occurred because the contract didn't properly reduce the total available rewards after a claim was made, allowing users to claim the same rewards multiple times.

4. **Exploiting Rebasing/Deflationary Tokens**

Some attacks target reward mechanisms embedded within a token's transfer function. For example, a token might grant a reward whenever it's transferred into a liquidity pool. An attacker can exploit this by using a flash loan to transfer tokens to a pool, trigger the reward, and then use a function like skim() to pull the liquidity back out without selling, effectively tricking the token into giving them free rewards.

## How to Prevent It

Preventing reward manipulation requires moving from simple, instantaneous checks to robust, time-based accounting.

1. **Use Balance Snapshots**

This is the primary defense against flash loan-based attacks. Instead of using the current balanceOf(), the contract should rely on a user's balance from a past block. This makes flash-loaned funds useless, as they weren't present at the time of the snapshot. OpenZeppelin's ERC20Snapshot extension is a battle-tested implementation of this.

2. **Implement Time-Weighted Rewards**

Rewards must be proportional to both amount and time. A user's reward should be calculated based on amount_staked * time_duration. This ensures that a user who stakes 10 ETH for 10 days earns the same as a user who stakes 100 ETH for 1 day. This single-handedly defeats same-block deposit-and-claim attacks.


```sol
// Store the last time a user's rewards were updated
mapping(address => uint256) public userRewardPerTokenPaid;
mapping(address => uint256) public lastUpdateTime;

function claimReward() external {
    // Calculate rewards accrued since last update
    updateReward(msg.sender); 
    uint256 reward = rewards[msg.sender];
    
    if (reward > 0) {
        rewards[msg.sender] = 0;
        rewardToken.transfer(msg.sender, reward);
    }
}
```

3. **Introduce a Cooldown or Lock-up Period**

A simple but highly effective defense is to enforce a minimum staking duration before a user becomes eligible for rewards. For example, a contract could require funds to be staked for at least 24 hours or a certain number of blocks before `claimReward()` can be successfully called. This completely neutralizes flash loan attacks.

4. **Robust and Atomic Accounting**

Ensure your reward accounting is flawless. When a user claims a reward, the contract must atomically:

-  Calculate the reward owed up to that point.

- Update the user's internal reward balance to zero.

- Update global state variables (like totalRewardsClaimed).

= Transfer the funds.

Use a simple mapping `mapping(address => uint) public lastClaimPeriod` to prevent users from claiming rewards for the same period more than once.