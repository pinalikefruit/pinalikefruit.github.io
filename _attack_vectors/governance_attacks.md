---
layout: default
title: "Governance"
rank: 9
amount_stolen: "$30,000,000"
incidents: 8
risk_level: "low"
year: 2024
---

# Governance Attacks

| A Governance Attack is the malicious exploitation of a decentralized protocol's intended governance mechanics to enact changes that benefit an attacker at the expense of the protocol and its users. 

## What is a Governance Attack?

A Governance Attack occurs when an entity exploits a protocol's legitimate governance processes to enact changes that benefit themselves at the expense of the DAO. Unlike code exploits, these attacks manipulate the rules of the system itself, turning it into a weapon to drain treasuries, alter critical parameters, or permanently damage the protocol.

The core vulnerability stems from equating token holdings with voting power. This design assumes that large stakeholders are invested in the protocol's long-term health, an assumption that is shattered when an attacker uses economic manipulation, like a flash loan, to acquire temporary, overwhelming voting power with no genuine stake in the outcome.

### Real-World Impact: The Beanstalk DAO Hack

In April 2022, Beanstalk DAO was exploited for $182 million in one of the most infamous governance attacks. The attacker executed the following steps:

1. Submitted two proposals: a malicious one (BIP-18) to transfer all funds to themselves, and a decoy (BIP-19) to donate funds to Ukraine, making the action seem less suspicious.

2. Acquired a massive flash loan of various tokens from Aave, Uniswap, and SushiSwap.

3. Used these borrowed funds to acquire a huge amount of Stalk, Beanstalk's governance token, gaining a supermajority of voting power.

4. With this voting power, they instantly passed the malicious proposal.

5. The proposal's code executed a function to transfer all of the protocol's treasury funds to the attacker's address.

6. The attacker then repaid the flash loan, walking away with the profits.

The entire attack was executed within a single transaction, using the DAO's own rules against it.

## How the Attack Works

The attack surface of a DAO is defined by its core components:

* **Voting Tokens**: These grant the right to vote on proposals. The fundamental assumption is that owning these tokens aligns incentives.

* **Proposals**: These are executable bundles of code that can make changes to the protocol if passed. The danger lies in what a proposal is allowed to do.

* **Voting Mechanism**: This includes the rules for passing a proposal, such as the voting period, the quorum (minimum participation), and the threshold (minimum 'yes' votes).

A governance attack exploits a weakness in one or more of these areas. The most common vectors are:

1. **Flash Loan-Based Voting** :If a protocol's governance mechanism checks voting power at the exact moment of a vote, an attacker can use a flash loan to borrow a massive number of governance tokens, vote to pass their own malicious proposal, execute it, and repay the loan all in one atomic transaction.

2.** Malicious Proposal Execution**:A proposal's text description can be deceptive. The actual payload is executable on-chain code, which can be obfuscated or hidden. For example, a proposal might claim to be a simple parameter tweak, but its code could use CREATE2 to deploy a new, malicious contract and then delegatecall into it, draining funds. Without rigorous analysis of the proposal's on-chain execution, voters can be easily tricked.

3. **Emergency Proposal Abuse**: Some DAOs have an "emergency" track that bypasses or shortens the standard timelock and lowers the quorum requirements. This is intended for rapid bug fixes. This mechanism can be abused by attackers to fast-track a malicious proposal, giving the community no time to react or organize a counter-vote.

## How to Prevent It

Preventing governance attacks requires a multi-layered defense combining technical and procedural safeguards.

1. **Counter Flash Loans with Time-Gated Voting**: The primary goal is to make temporary voting power useless.

    * T**ime-Weighted Voting** (Vote-Escrow or Snapshots): This is the most effective defense. Instead of counting votes based on current token balances, voting power should be calculated based on balances from a past block (e.g., block.number - X) or the duration for which tokens have been locked. This makes flash-loaned tokens worthless for voting, as they weren't held at the required snapshot time.

    * **Vote Locking**: Require users to lock their governance tokens for the duration of the voting and execution period. This introduces a real economic cost and risk for an attacker, as they cannot simply borrow and return tokens in a single transaction.

2. **Enforce Mandatory TimelocksEnforce** 
A non-bypassable delay between when a proposal passes and when it can be executed. A timelock of 2-7 days gives the community time to analyze the proposal's code, react to any malicious intent, and exit the protocol if necessary. Emergency tracks should be used with extreme caution, if at all.

3. **Limit Governance Power**  
Strictly scope what governance can and cannot do. For example, governance should be able to change protocol fees but should never have the power to arbitrarily upgrade contracts or directly access user funds in unrelated vaults. This limits the blast radius of a successful attack.

4. **Use Battle-Tested Frameworks** 
Implement governance using robust, audited frameworks like OpenZeppelin Governor (based on Compound's proven model). These have built-in protections and follow best practices.

5. **Implement Procedural Safeguards & Verification**

    * **Proposal Vetting and Simulation**: Establish a formal, off-chain process where security researchers or a dedicated committee analyze the code of every proposal before it goes to an on-chain vote. Simulate the proposal's execution on a forked mainnet to ensure it does exactly what it claims.

    * **Quorum and Threshold Tuning**: Set quorum and voting thresholds high enough to ensure a significant portion of genuine stakeholders must participate for a proposal to pass, preventing a small group from seizing control.

    * **Multi-Layered Governance**: Consider a "Security Council" or a trusted multi-sig that has the power to veto or delay the execution of a passed proposal that is identified as clearly malicious. While this introduces a centralization trade-off, it can act as a critical backstop against catastrophic attacks.