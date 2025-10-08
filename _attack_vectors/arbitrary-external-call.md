---
layout: default
title: "Arbitrary External Call"
---

# Arbitrary External Call

## What is an Arbitrary External Call?

An Arbitrary External Call is a vulnerability where a smart contract executes a call to an address with data specified by an untrusted user. The vulnerability stems from the contract lending its identity and authority to the caller. 

When Contract A executes `target.call(data)`, the `msg.sender` within the target contract is Contract A. If an attacker can control the target and data, they can impersonate the protocol, forcing it to perform any action it has permission to do.

External calls are for any interaction with any external contract, oracles, etc. If used correctly, they can perform unwanted actions against the protocol.

### Real-World Impact: The CoW Swap (GPv2) Hack

In 2022, [CoW Swap's GPv2](https://blog.solidityscan.com/cow-swap-hack-analysis-arbitrary-callable-swapguard-6a6ee3de346f) settlement contract was exploited due to this vulnerability. The protocol allowed "solvers" to specify arbitrary on-chain interactions to facilitate swaps. An attacker acted as a solver and submitted a transaction with a malicious payload that forced the GPv2 contract to call `approve()` on its own stablecoin tokens, granting the attacker infinite approval. Because the GPv2 contract was the `msg.sender` of the approve call, the token contract accepted it. The attacker then simply called `transferFrom()` to drain the funds from the GPv2 vault.

## How the Attack Works

Any external call made by a smart contract can be susceptible to manipulation, depending on how input parameters are handled and the level of control the user has over them.

```sol
function execute(address target, bytes calldata data) external {
    // The contract blindly executes whatever the user provides
    (bool success, ) = target.call(data);
    require(success, "External call failed");
}
```


**Other Manifestations**:

* **Reentrancy**: If the external call is made before state changes (e.g., updating balances), an attacker can set the target to their own malicious contract, which calls back into the protocol to drain funds before their balance is updated.

* **Denial of Service (DoS)**: An attacker can set the target to a contract designed to always revert or consume all available gas, causing legitimate user transactions to fail.

## How to Prevent It

Prevention focuses on eliminating or heavily restricting the ability for users to dictate external calls.

1. **Whitelist Target Addresses**

    The effective defense is to not allow calls to arbitrary addresses. Maintain a hardcoded or governance-controlled whitelist of trusted contracts that your protocol is allowed to interact with.

    ```sol
    mapping(address => bool) public isTrustedTarget;

    function execute(address target, bytes calldata data) external {
        require(isTrustedTarget[target], "Target not whitelisted");

        (bool success, ) = target.call(data);
        require(success, "External call failed");
    }
    ```

2. **Checks-Effects-Interactions**

    If an external call is necessary, ensure it happens last.

    - **Checks**: Perform all validation (e.g., `require(balance >= amount)`).

    - **Effects**: Make all state changes to your contract (e.g., `balance -= amount`).

    - **Interactions**: Then, make the external call (e.g., `token.transfer(amount)`).

    This pattern single-handedly prevents all classic reentrancy attacks, as the contract's state is updated before the attacker has a chance to call back in.

3. **Use Reentrancy Guards**

    For any function that performs an external call, use [OpenZeppelin's ReentrancyGuard](https://docs.openzeppelin.com/contracts/5.x/api/utils#ReentrancyGuard) modifier. 

    ```sol
    import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

    contract MyContract is ReentrancyGuard {
        function myFunc() external nonReentrant {
            // ... code with an external call
        }
    }
    ```

4. **Sanitize Inputs and Handle Errors**

    - **Access Control**: If a function must allow arbitrary calls, ensure it is protected with strict access control (e.g., onlyOwner) so that only a trusted multi-sig can use it.

    - **Use try/catch**: To prevent DoS attacks where an external call might revert, wrap the call in a `try/catch` block. This allows your function to handle the failure gracefully instead of having the entire transaction revert.
