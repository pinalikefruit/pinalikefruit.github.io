---
layout: default
title: "Malicious Insider"
rank: 3
amount_stolen: "$95,000,000"
incidents: 17
risk_level: "high"
year: 2024
---

# Malicious Insider

| How attackers exploit the ultimate vulnerability: human trust.

## What is a Malicious Insider Attack?

A Malicious Insider attack is a security breach originating from an individual with legitimate, trusted access to a project's internal systems, information, or personnel. Unlike a smart contract exploit, this is an **operational security (OPSEC)** failure. The attacker doesn't break the code; they abuse the trust placed in them.

This threat manifests in two primary forms:

1. **The True Insider**: A current or former employee, contractor, or core contributor who uses their privileged access for malicious purposes.

2. **The Impersonator**: An external attacker who engages in sophisticated social engineering to build a credible persona, gain the trust of the team, and infiltrate the project's inner circle.

In both cases, the attacker's goal is to bypass technical defenses by exploiting human relationships and operational weaknesses.

### Real-World Impact: The "Nick Franklin" Impersonator

For over a year, a persona known as "Nick Franklin" operated as a respected security researcher in the Web3 space. He provided timely, insightful analyses of major hacks, built a network of contacts, and became a trusted voice. In reality, he was a state-sponsored operative for North Korea's Lazarus Group.

The facade crumbled when he sent a malicious file disguised as a security report to 1inch co-founder Anton Bukev. The subsequent investigation revealed that "Franklin" was not just analyzing hacks; he was implicated in creating them. An address he used was linked to testing the very exploit that drained **$50 million from Radiant Capital**. This was not a code bug; it was a year-long infiltration campaign that weaponized trust to get close to high-value targets and protocols.

## How the Attack Works

A malicious insider or impersonator follows a methodical playbook that focuses on exploiting people and processes, not code.

1. **Infiltration and Trust Building**

This is the most critical phase. The attacker invests significant time in becoming a known and trusted entity.

* **Create a Credible Persona**: They build a history on platforms like Twitter, GitHub, and Telegram, often by contributing high-quality public goods like exploit analyses or open-source tools.

* **Be Consistently Helpful**: They actively participate in public discussions, answer questions, and offer help, slowly building a reputation for expertise and reliability.

* **Infiltrate Inner Circles**: Their reputation eventually grants them access to semi-private or private group chats, where they can interact directly with core team members.

2. **Reconnaissance and Information Gathering**

Once inside, the attacker shifts from building trust to gathering intelligence.

* **Identify Key Personnel**: They map out the organization to find who holds the keys: multisig signers, lead developers with deployment access, and founders.

* **Access Non-Public Information**: They gain visibility into private GitHub repositories, internal roadmaps, and sensitive security discussions.

* **Probe for Weaknesses**: As seen with "Nick Franklin" asking pointed questions about Radiant Capital before the hack, they probe the team's security awareness and internal procedures.

3. **The Exploit: Weaponizing Trust**

With trust established and intelligence gathered, the attacker executes the final phase. This can take several forms:

* **Social Engineering & Malware**: The attacker sends a malicious file (e.g., a fake report, a "new tool") to a key target. If opened, the malware can exfiltrate private keys, API secrets, or session tokens, giving the attacker direct access to wallets or infrastructure.

* **Direct Abuse of Privileges**: A true insider with legitimate access (e.g., a developer on the multisig) simply executes a malicious transaction to drain the treasury or change critical contract parameters.

* **Information Leakage**: The insider doesn't perform the hack themselves. Instead, they leak a known-but-unpatched vulnerability, a set of private keys, or sensitive operational details to an external team that carries out the attack.

## How to Prevent It

Preventing human-level exploits requires shifting focus from purely technical audits to robust operational security.

1. **Vet Your People**

Code gets multiple audits while people often get none. For core team members with privileged access, this needs to change.

* **Background Checks**: For key roles (e.g., multisig signers, finance ops), conduct formal background checks.

* **Identity Verification**: While anonymity is a part of crypto, core teams with control over millions in assets should not be completely anonymous to each other. Implement KYC/KYB for foundational members.

* **Be Skeptical of Newcomers**: Treat new, overly helpful community members with friendly skepticism. Verify their claims and track record before granting any level of trusted access.

2. **Enforce Strict Digital Hygiene and Access Control**

Apply the same security principles from smart contracts to your operations.

* **Principle of Least Privilege**: An employee should only have access to the systems they absolutely need for their job. A community manager does not need access to production servers or deployment keys.

* **Rigorous Offboarding**: Create an ironclad checklist for when an employee or contractor leaves. Immediately revoke all access to GitHub, Discord, email, cloud services, and any other team accounts.

* **Use Hardware Wallets & Multisigs**: Mandate the use of hardware wallets for all team members. Secure treasury funds and protocol controls with a multisig wallet requiring a quorum of trusted, verified individuals.

3. **Compartmentalize Information**

Secure Communication Channels: Do not discuss sensitive vulnerabilities or security procedures in public or semi-public group chats. Use end-to-end encrypted channels (like Signal with disappearing messages) for high-stakes conversations.

**Need-to-Know Basis**: Limit the dissemination of sensitive information. Not everyone on the team needs to know the details of a planned security upgrade or the full list of multisig signers.

4. **Foster a Culture of Security**

Train the Team: Educate your team on social engineering and phishing risks. Teach them to be suspicious of unsolicited files, unexpected requests, or anyone creating a sense of urgency.

* **Establish Clear Protocols**: Create rules for communication. For example: "We will never ask for your private key or send you an executable file in a DM."

* **Verify, Don't Trust**: Encourage team members to independently verify unusual requests, even if they appear to come from a founder or lead developer. A quick, separate call can thwart an impersonator.