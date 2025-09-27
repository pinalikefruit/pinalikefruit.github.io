---
layout: default
title: "Rounding Errors"
---

# Rounding Errors
| How attackers can steal millions, one wei at a time.


## What is a Rounding Error?

A Rounding Error in smart contracts is a vulnerability that arises from the loss of precision during integer division. The Ethereum Virtual Machine (EVM) does not support floating-point numbers; all calculations are performed with integers. This means that any division operation truncates the result, discarding the remainder. For example, 7 / 2 equals 3, not 3.5.

While the loss of a fraction of a wei in a single transaction seems insignificant, this "dust" can be systematically accumulated by an attacker over thousands of transactions. The vulnerability is not a bug in the EVM itself, but a design flaw in a contract's mathematical logic where the protocol fails to account for these tiny, repeated losses, allowing an attacker to claim them.

### Real-World Impact: The Fei Protocol Exploit

In April 2021, Fei Protocol, an algorithmic stablecoin, was exploited due to a rounding error in its incentive mechanism. The protocol had a feature that would burn a percentage of FEI tokens from a user's balance when they exited a specific Uniswap pool. The burn calculation, however, contained a rounding vulnerability.

Under certain conditions, the calculation would round down in the user's favor, resulting in fewer tokens being burned than intended. An attacker noticed this, took out a large flash loan, and repeatedly entered and exited the pool in a loop. Each time, they benefited from the rounding error, accumulating the "dust" from the flawed burn calculation. They repeated this process thousands of times within a single transaction, draining a significant amount of value from the protocol's reserves.

## How the Attack Works

Attackers exploit rounding errors by identifying flawed mathematical operations and repeating them in a loop to amplify the effect. The most common patterns are:

1. Incorrect Order of Operations

This is the most frequent cause of rounding vulnerabilities. Performing division before multiplication unnecessarily destroys precision.

Consider a function that calculates a user's share of a reward pool:userReward = (userShares / totalShares) * totalRewardPool;

If userShares is less than totalShares, the initial division (userShares / totalShares) will truncate to 0. The user's reward will always be zero, and the funds will be locked in the contract, or claimable by a more fortunate user whose division doesn't truncate.


```sol
// VULNERABLE: Division is performed too early, destroying precision.
function calculateShare(uint256 userShares, uint256 totalShares, uint256 totalPool) internal pure returns (uint256) {
    // If userShares < totalShares, this will be 0.
    uint256 sharePercentage = userShares / totalShares; 
    return sharePercentage * totalPool;
}
```


2. Accumulating Precision "Dust"

This pattern, seen in the Fei hack, involves exploiting calculations where the rounding benefits the user. An attacker will find a function where the protocol loses a fraction of a token due to truncation and then call it repeatedly.

Attacker's Playbook:

Identify Flaw: Find a function where A - B is calculated, but B is the result of a division that rounds down (e.g., B = (A * fee) / 1000). The actual fee applied is slightly less than intended.

Craft Inputs: Provide inputs that maximize this rounding benefit.

Loop: Use a flash loan to acquire a large starting capital and call the function thousands of times in a single transaction.

Accumulate: With each call, the attacker saves a tiny amount of value that should have gone to the protocol. Over thousands of iterations, this dust accumulates into a substantial profit.

3. Off-by-One Errors in Distributions

When distributing rewards or funds among multiple recipients, flawed division logic can leave a remainder. If the contract doesn't explicitly handle this remainder, it can become locked or be claimed by the last person to interact with the function.

## How to Prevent It

Preventing rounding errors requires a disciplined approach to smart contract mathematics.

1. Amplify Before Dividing

This is the golden rule. Always perform multiplications before divisions to preserve the maximum amount of precision until the final step.

// CORRECT: Multiplication is performed first to maintain precision.
function calculateShare(uint256 userShares, uint256 totalShares, uint256 totalPool) internal pure returns (uint256) {
    // The result will be accurate, as precision is only lost at the very end.
    return (userShares * totalPool) / totalShares;
}

Use code with caution.Solidity

2. Use Battle-Tested Math Libraries

Do not reinvent proportional math. Use libraries that are audited and designed to handle these specific problems. OpenZeppelin's Math library is the industry standard.

mulDiv(): This function calculates (a * b) / c without overflowing the intermediate multiplication. It is the safest way to handle proportional calculations.

ceilDiv(): In some cases, you may need to round the result of a division up instead of down (truncating). ceilDiv() provides this functionality safely.

3. Account for Every Wei

A robust protocol should never "lose" value to rounding. The logic must be deterministic and account for any remainders from division.

Track the Remainder: After a distribution calculation, explicitly calculate the remainder.

Assign the Remainder: Decide what happens to the dust. Does it get sent to the treasury? Does it stay in the contract for the next distribution? Does the last user in a loop claim it? Leaving it unaccounted for is an invitation for an exploit.

4. Think in Basis Points, Not Percentages

When dealing with fees or percentages, avoid floating-point assumptions. Represent percentages as integers using basis points (where 1% = 100 basis points and 100% = 10,000). This allows all fee calculations to be performed with integer arithmetic, reducing the risk of error.

Generated solidity

uint256 feeBasisPoints = 50; // Represents 0.5%
uint256 fee = (amount * feeBasisPoints) / 10000;

