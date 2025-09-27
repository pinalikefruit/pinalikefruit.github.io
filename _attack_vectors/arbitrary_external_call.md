---
layout: default
title: "Arbitrary External Call"
rank: 8
amount_stolen: "$27,000,000"
incidents: 11
risk_level: "low"
year: 2024
---

# Arbitrary External Call

|  How attackers turn your contract into a weapon against itself

## What is an Arbitrary External Call?

An Arbitrary External Call is a critical vulnerability where a smart contract executes a call to an address with data specified by an untrusted user. The vulnerability stems from the contract lending its identity and authority to the caller. 

When Contract A executes target.call(data), the msg.sender within the target contract is Contract A. If an attacker can control the target and data, they can impersonate the protocol, forcing it to perform any action it has permission to do.

While such external calls are essential for DeFi composability (e.g., interacting with tokens, oracles, or other protocols), they become a backdoor when the destination and instructions are not strictly validated. 

This allows an attacker to force the protocol to execute unintended actions, such as approving token spending, interacting with malicious contracts, or triggering reentrancy.

### Real-World Impact: The CoW Swap (GPv2) Hack

In 2022, CoW Swap's GPv2 settlement contract was exploited due to this vulnerability. The protocol allowed "solvers" to specify arbitrary on-chain interactions to facilitate swaps. An attacker acted as a solver and submitted a transaction with a malicious payload that forced the GPv2 contract to call approve() on its own stablecoin tokens, granting the attacker infinite approval. Because the GPv2 contract was the msg.sender of the approve call, the token contract accepted it. The attacker then simply called transferFrom() to drain the funds from the GPv2 vault.

## How the Attack Works

The vulnerability is not in the call opcode itself, but in the trust placed in user-supplied inputs that dictate its parameters. A vulnerable function often looks conceptually like this:

```sol
function execute(address target, bytes calldata data) external {
    // The contract blindly executes whatever the user provides
    (bool success, ) = target.call(data);
    require(success, "External call failed");
}
```

**Attacker's Playbook**:

1. **Discovery**: The attacker finds a function like the one above, which allows them to control both the target and data of an external call. This is often found in complex protocols that require high degrees of composability, such as DEX aggregators or multi-step transaction batchers.

2. **Payload Crafting**: The attacker determines a privileged action the protocol contract can perform and crafts the payload to execute it. This involves selecting:

3. **target**: The address of a contract the protocol interacts with (e.g., a WETH or USDC token contract).

4. **data**: The encoded function signature and arguments for the desired action. For example, abi.encodeWithSignature("approve(address,uint256)", attackerContractAddress, type(uint256).max).

5. **Execution**: The attacker calls the vulnerable execute function on the protocol contract, passing in their malicious target and data.

6. **Impersonation**: The protocol contract executes target.call(data). From the perspective of the target contract (e.g., USDC), the msg.sender is the protocol contract. The token contract sees a legitimate-looking request from a trusted address and processes it.

7. **Achieve Malicious Goal**: The attacker leverages the state change from step 4. This could be:

8. **Draining Funds**: Calling transferFrom() after gaining approval.

9. **Minting Tokens**: Forcing the contract to call a privileged mint function on a token it controls.

10. **Gaining Privileges**: Forcing the contract to call setOwner() or grantRole() on another contract, making the attacker an admin.

**Other Manifestations**:

* **Reentrancy**: If the external call is made before state changes (e.g., updating balances), an attacker can set the target to their own malicious contract, which calls back into the protocol to drain funds before their balance is updated.

* **Denial of Service (DoS)**: An attacker can set the target to a contract designed to always revert or consume all available gas, causing legitimate user transactions to fail.

## How to Prevent It

Prevention focuses on eliminating or heavily restricting the ability for users to dictate external calls.

1. **Whitelist Target Addresses**

The most effective defense is to not allow calls to arbitrary addresses. Maintain a hardcoded or governance-controlled whitelist of trusted contracts that your protocol is allowed to interact with.

```sol
mapping(address => bool) public isTrustedTarget;

function execute(address target, bytes calldata data) external {
    // Strongest defense: only call pre-approved contracts
    require(isTrustedTarget[target], "Target not whitelisted");

    (bool success, ) = target.call(data);
    require(success, "External call failed");
}
```

2. **Checks-Effects-Interactions**

This is a fundamental security pattern in Solidity. If an external call is necessary, ensure it happens last.

- **Checks**: Perform all validation (e.g., require(balance >= amount)).

- **Effects**: Make all state changes to your contract (e.g., balance -= amount).

- **Interactions**: Then, make the external call (e.g., token.transfer()).

This pattern single-handedly prevents all classic reentrancy attacks, as the contract's state is updated before the attacker has a chance to call back in.

3. **Use Reentrancy Guards**

For any function that performs an external call, use OpenZeppelin's ReentrancyGuard modifier. It provides a simple and gas-efficient lock to prevent a function from being re-entered while it is still executing.

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

- **Use try/catch**: To prevent DoS attacks where an external call might revert, wrap the call in a try/catch block. This allows your function to handle the failure gracefully instead of having the entire transaction revert.

- **Validate Calldata**: In advanced cases, you could restrict which functions can be called by inspecting the first 4 bytes of the calldata (the function selector) against a whitelist of allowed function IDs.