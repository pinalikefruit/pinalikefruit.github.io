---
layout: default
title: "Integer Overflow and Underflow"
---

# Integer Overflow and Underflow

|How a simple mathematical error can turn a contract's balance from a million to zero, or vice-versa.


## What is an Integer Overflow/Underflow?

An Integer Overflow or Underflow is a vulnerability that occurs when an arithmetic operation attempts to create a number that is outside the range that can be stored in its data type. In computing, integers are stored in fixed-size blocks of memory (e.g., uint8, uint256). When a calculation exceeds the maximum value for that type, it "wraps around" to the minimum value (overflow). Conversely, when it goes below the minimum value, it wraps around to the maximum (underflow).

Think of a car's odometer: if it has six digits, driving one mile past 999,999 will roll it over to 000,000. This is an overflow. An underflow is the reverse: 0 - 1 becomes the maximum possible value.

Before Solidity version 0.8.0, this behavior was the default for all arithmetic, making it a common and devastating attack vector. While modern Solidity has built-in protections, this vulnerability can be re-introduced through custom mathematical logic or explicit unchecked blocks.

### Real-World Impact: The Cetus Hack

In May 2023 (Note: The source article's date of 2025 is a typo, the hack was in 2023), Cetus, a major DEX on the Sui blockchain, was exploited for an estimated $223 million. While this attack occurred in the Move programming language, the principle is universal and serves as a stark warning for all smart contract developers.

The attacker exploited a flawed overflow check in a function that calculated liquidity. The check was intended to prevent a bit-shift operation from overflowing, but it used an incorrect constant, allowing certain values to pass the check while still causing an overflow during the actual calculation. The attacker supplied a carefully crafted liquidity value that triggered this overflow, causing the required number of tokens for the deposit to be miscalculated as just one. They deposited a single token, were credited with a massive amount of liquidity, and proceeded to drain the protocol's pools.

### How the Attack Works

The attack vector depends on the context, but it always involves manipulating a contract's state by exploiting its mathematical limitations.

1. Classic Underflow in Token Balances (Pre-Solidity 0.8.0)

This is the canonical example. A transfer function subtracts from the sender's balance without a safety check.



```sol
// VULNERABLE on Solidity < 0.8.0
mapping(address => uint256) public balances;

function transfer(address _to, uint256 _value) public {
    require(balances[msg.sender] >= _value); // Check passes if balance is 0 and value is 0
    balances[msg.sender] -= _value; // Underflow! 0 - 1 becomes type(uint256).max
    balances[_to] += _value;
}
```

Attacker's Playbook:

The attacker starts with a balance of 0 tokens.

They call transfer(attacker_address_2, 1).

The require check balances[msg.sender] >= _value (i.e., 0 >= 1) fails, as expected.

However, if they call withdraw(_value) where _value is greater than their balance, the line balances[msg.sender] -= _value would underflow, giving them a massive balance. A poorly written withdraw function could be a target.

A more common attack was on functions that allowed transfers of 0 value. If an attacker with 0 balance called transfer(any_address, 1) on a contract where the check was flawed, the subtraction 0 - 1 would underflow, giving them type(uint256).max tokens.

2. Flawed Custom Mathematical Checks (The Cetus Pattern)

Even in modern Solidity, developers often write complex custom math functions for performance or unique logic. If the safety checks in these functions are flawed, they re-introduce overflow risks.

Vulnerable Logic (Conceptual):Imagine a function that calculates rewards based on a user's shares and a global multiplier.

```sol
function calculateReward(uint256 userShares) internal view returns (uint256) {
    // Flawed check: developer assumes multiplication won't overflow
    // if the inputs are within a certain range, but their assumption is wrong.
    uint256 reward = userShares * rewardMultiplier; 
    return reward;
}
```

Attacker's Playbook:

The attacker finds a complex calculation with a custom, flawed, or missing overflow check.

They manipulate the contract's state (e.g., via flash loan) to provide specific inputs to this function.

The inputs are crafted to cause an intermediate or final calculation to overflow. For example, rewardMultiplier is pumped to a huge value.

The reward calculation overflows, wrapping around to a small but non-zero number, or the logic is exploited in another way. In the Cetus hack, the overflow resulted in a required deposit of 1.

The attacker leverages this miscalculated result to their advantage, either claiming an illegitimate reward or draining funds.

How to Prevent It

Prevention relies on a combination of modern tools, battle-tested libraries, and disciplined coding practices.

1. Use Modern Solidity (>= 0.8.0)

This is the most important line of defense. Solidity versions 0.8.0 and later have built-in overflow and underflow protection. Any arithmetic that overflows or underflows will cause the transaction to revert. This single change mitigates the vast majority of classic integer vulnerabilities.

2. Use Battle-Tested Math Libraries

For complex operations or when working with older Solidity versions, always use a standard math library. OpenZeppelin's SafeMath was the gold standard for pre-0.8.0 contracts. Even in modern Solidity, OpenZeppelin's Math library provides safe and gas-efficient functions for more complex calculations like sqrt, ceil, and mulDiv.

3. Use unchecked Blocks with Extreme Caution

Solidity 0.8.0+ introduced unchecked blocks to allow for intentional overflows for gas optimization. Any code inside an unchecked block loses all default safety checks. This is a high-risk area that must be treated with extreme care.

```
// Use unchecked only when you are mathematically certain an overflow is impossible.
unchecked {
    // This will NOT revert on overflow.
    counter++; 
}
```


Only use unchecked when you have mathematically proven that the operation cannot overflow under any circumstances, or when the wrap-around behavior is explicitly desired.

4. Rigorously Test All Mathematical Logic

If you write custom math functions, you are responsible for their safety.

Unit Testing: Write extensive tests covering all edge cases: zero, type(uint).max, values just below and above overflow thresholds.

Fuzzing: Use tools like Foundry's fuzzer to automatically test your functions with a huge range of random inputs to find unexpected edge cases.

Formal Verification: For highly critical and complex mathematical logic, consider using formal verification tools to mathematically prove the correctness of your code.

5. Be Careful with Type Casting

Explicitly casting a larger integer type to a smaller one (e.g., uint256 to uint64) can truncate the value, which is a form of data loss similar to an overflow. Ensure that you have validated the value fits within the smaller type before casting.
