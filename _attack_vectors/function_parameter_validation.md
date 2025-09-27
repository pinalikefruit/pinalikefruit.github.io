---
layout: default
title: "Function Parameter Validation"
rank: 4
amount_stolen: "$69,000,000"
incidents: 21
risk_level: "low"
year: 2024
---

# Function Parameter Validation

| How trusting user input can allow attackers to hijack your contract's logic.

## What is  Function Parameter Validation?

Insufficient Function Parameter Validation is a vulnerability where a smart contract fails to properly scrutinize the inputs supplied by a user. This allows an attacker to pass malicious or unexpected values—such as a zero address, a hostile contract address, or an out-of-range number—to manipulate the contract's behavior. When a contract blindly trusts its inputs, it effectively gives an attacker a lever to bypass security checks, corrupt state, or drain funds.

The vulnerability stems from a broken assumption: that users will always provide data within the expected parameters. Secure contracts operate on a "zero trust" model for all external input. Failing to validate parameters is like giving a user direct control over the contract's internal machinery.

### Real-World Impact: The Beanstalk Well Exploit

The Beanstalk protocol allowed users to withdraw liquidity from pools called "Wells." The withdrawal function required the user to specify the address of the Well they were withdrawing from. However, the protocol never validated that the provided address was a legitimate, whitelisted Well contract. An attacker created their own malicious Well contract. When the withdrawal function called this hostile contract to calculate the payout, the malicious Well simply returned a value equal to the entire BEAN token balance of the main Beanstalk contract. Because the input was never validated, the protocol trusted the malicious calculation and sent all its funds to the attacker.

## How the Attack Works

This vulnerability can be exploited in numerous ways, often by targeting a single, unguarded parameter in a function call.

1. **Untrusted Contract Address Parameter**

This is the pattern seen in the Beanstalk hack. A function takes a contract address as an argument and interacts with it, but fails to check if that address belongs to a trusted, known contract.

**Attacker's Playbook (Beanstalk):**

- **Discovery**: The attacker identifies a function like withdraw(address well, ...) that takes a contract address as an input and makes an external call to it.

- **Payload Crafting**: The attacker deploys their own malicious contract (MaliciousWell.sol) that implements the same interface as a real Well. Its calculatePayout() function is hardcoded to return a massive number.

- **Execution**: The attacker calls the protocol's withdraw() function, passing in the address of their MaliciousWell.sol.

- **Hijacked Logic**: The protocol, trusting the input, calls maliciousWell.calculatePayout(). It receives the inflated value and, believing it to be a legitimate calculation, transfers a catastrophic amount of funds to the attacker.

2. **Lack of Zero-Address Validation**

A simple but common oversight is failing to check if an address parameter is address(0). This can be used to burn tokens, lock contracts by setting the owner to a null address, or exploit logic that treats the zero address as a special case (e.g., for native ETH).

3. **Abusing Special Parameters**

Some functions allow a user to pass in arbitrary bytes data to be used in a subsequent call. If this data is not validated, an attacker can craft a payload to execute a completely unintended action.

* **The Dough Finance Exploit**: A function designed to execute a token swap via Paraswap took a swapData parameter. The protocol failed to validate this data. An attacker passed in a malicious payload that was not a swap, but instead an encoded transferFrom call. Because the protocol had been approved to spend user funds, the attacker used this unguarded parameter to force the contract to transfer those funds directly to them.

4. Exploiting Flash Loan Callbacks

Flash loans often pass user-defined data to the onFlashLoan callback function. If this callback doesn't validate who initiated the loan or what the data contains, it can be triggered directly and manipulated.

**The Prisma Finance Exploit**: The onFlashLoan function in a migration contract was intended to be called only as part of a larger, controlled migration process. However, it could be triggered by anyone who initiated a flash loan. An attacker called it directly with malicious data, allowing them to manipulate other users' positions (Troves) and steal the underlying collateral.

## How to Prevent It

Prevention boils down to a single mantra: Never trust external input. Every parameter passed into a public or external function must be considered hostile until proven otherwise.

1. **Use Whitelists for Contract Address Inputs**

If a function must interact with other contracts, do not allow users to supply an arbitrary address. Maintain a secure, governance-controlled whitelist of trusted contract addresses and validate the input against it.

```sol
mapping(address => bool) public isTrustedWell;

function withdraw(address well, ...) public {
    // CRITICAL: Validate the input against the whitelist
    require(isTrustedWell[well], "Well not whitelisted");

    // ... proceed with withdrawal logic ...
}
```
2. **Implement require Statements for All Inputs**

Every parameter should be checked with a require statement at the beginning of the function.

* **Address Validation**: Check that address parameters are not `address(0).require(_user != address(0), "Invalid address");`

* **Numerical Validation**: Check that numerical inputs are within an expected `range.require(_amount > 0, "Amount must be greater than zero");`

- **Array Validation**: Check that arrays are not empty if they are expected to contain elements.`require(_targets.length > 0, "Targets array cannot be empty");`

3. **Sanitize and Restrict Arbitrary Calldata**

If your function must accept arbitrary bytes data, severely restrict what it can do.

- **Function Selector Whitelisting**: A strong defense is to inspect the first 4 bytes of the data (the function selector) and validate it against a whitelist of allowed function signatures. This ensures that even if an attacker can control the arguments, they can only call pre-approved functions.

- **Use try/catch**: When making an external call with user-supplied data, wrap it in a try/catch block to handle potential reverts gracefully and prevent a Denial of Service attack.

4. **Secure Flash Loan Callbacks**

The `onFlashLoan` (or equivalent) callback function must be protected.

- **Initiator Check**: The callback must validate that the initiator of the flash loan is a trusted address (e.g., the contract itself). This prevents external users from triggering it directly.

- **State-Based Validation**: Use a state variable as a reentrancy-style guard (e.g., isMigrating = true;) before initiating the flash loan, and check for this state variable inside the callback. This ensures the callback can only execute as part of a legitimate, multi-step process.