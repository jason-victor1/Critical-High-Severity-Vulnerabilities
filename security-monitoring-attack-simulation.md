# 🔍 Comprehensive Guide to Smart Contract Security Monitoring & Attack Simulation

This guide will cover:

1. **Setting Up Live Security Alerts Using Forta & Tenderly**
2. **Simulating a Smart Contract Attack with Forta & Tenderly**
3. **Performing Blockchain Forensic Analysis After an Exploit**

---

## 📌 1️⃣ Setting Up Live Security Alerts (Forta & Tenderly)

We will create real-time security alerts for detecting suspicious activity on a smart contract.

### 🛠️ Step 1: Setting Up Forta for Live Monitoring

#### 1️⃣ Install Forta CLI

```bash
npm install -g forta-agent
forta init
```

This sets up Forta on your system.

#### 2️⃣ Create a Security Bot for Monitoring Large Withdrawals

We will create a bot that detects withdrawals over 10 ETH from a smart contract.

```javascript
const { Finding, HandleTransaction } = require("forta-agent");

const HIGH_WITHDRAWAL_THRESHOLD = 10 * 10 ** 18; // 10 ETH

function handleTransaction(txEvent) {
  const findings = [];
  txEvent.traces.forEach((trace) => {
    if (trace.action.value > HIGH_WITHDRAWAL_THRESHOLD) {
      findings.push(
        Finding.fromObject({
          name: "Large Withdrawal Detected",
          description: `A withdrawal of ${trace.action.value} ETH was detected`,
          alertId: "FORTA-1",
          severity: Finding.Severity.Medium,
        })
      );
    }
  });
  return findings;
}

module.exports = { handleTransaction };
```

#### 3️⃣ Deploy the Bot to Forta Network

```bash
forta publish
```

✅ Now, Forta will monitor all transactions and alert you for high-value withdrawals.

### 🛠️ Step 2: Setting Up Tenderly Alerts for Unauthorized Admin Calls

1️⃣ Go to **Tenderly → Alerts**  
2️⃣ Create a New Alert with the following parameters:

- **Contract:** `YourContract.sol`
- **Function:** `setAdmin()`
- **Condition:** If caller ≠ owner, trigger alert
- **Notification Method:** 📩 Email | 🚨 Telegram | 📢 Discord

3️⃣ Save & Enable Monitoring

✅ Now, Tenderly will notify you if an unauthorized user attempts to change admin privileges.

---

## 📌 2️⃣ Simulating a Smart Contract Attack Using Forta & Tenderly

We will simulate a reentrancy attack and monitor it in Forta & Tenderly.

### 🛠️ Step 1: Deploy the Vulnerable Smart Contract

```solidity
pragma solidity ^0.8.0;

contract VulnerableBank {
    mapping(address => uint256) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        (bool success, ) = msg.sender.call{value: amount}("");  // 🚨 Reentrancy risk!
        require(success, "Transfer failed");
        balances[msg.sender] -= amount;  // ❌ Balance updates after transfer!
    }
}
```

### 🛠️ Step 2: Deploy the Attacker Contract

```solidity
pragma solidity ^0.8.0;

import "./VulnerableBank.sol";

contract Attacker {
    VulnerableBank public vulnerableBank;

    constructor(address _vulnerableBank) {
        vulnerableBank = VulnerableBank(_vulnerableBank);
    }

    function attack() public payable {
        vulnerableBank.deposit{value: msg.value}();
        vulnerableBank.withdraw(msg.value);
    }

    receive() external payable {
        if (address(vulnerableBank).balance > 1 ether) {
            vulnerableBank.withdraw(1 ether); // 🚨 Exploit reentrancy!
        }
    }
}
```

### 🛠️ Step 3: Monitor the Attack Using Forta

#### 1️⃣ Create a Forta Bot to Detect Reentrancy Attacks

```javascript
const { Finding, HandleTransaction } = require("forta-agent");

function handleTransaction(txEvent) {
  const findings = [];
  txEvent.traces.forEach((trace) => {
    if (trace.action.to === trace.action.from) {
      // 🚨 Same address calling itself
      findings.push(
        Finding.fromObject({
          name: "Reentrancy Attack Detected",
          description: `Reentrancy attack detected on contract ${trace.action.to}`,
          alertId: "FORTA-2",
          severity: Finding.Severity.High,
        })
      );
    }
  });
  return findings;
}

module.exports = { handleTransaction };
```

#### 2️⃣ Deploy the Bot to Forta

```bash
forta publish
```

✅ Now, Forta will alert you whenever a contract calls itself (indicating a possible reentrancy attack).

### 🛠️ Step 4: Analyze the Attack in Tenderly

1️⃣ Go to **Tenderly → Transactions**

- Find the attack transaction.
- Replay it in the Tenderly Debugger.
- See how the attack drained funds step by step.

2️⃣ Check the Function Call Stack:

- Look for repeated `withdraw()` calls.
- Confirm the contract allows reentrant withdrawals.

✅ Now, you can analyze and test fixes before deploying a patch!

---

## 📌 3️⃣ Performing Blockchain Forensic Analysis After an Exploit

Once an attack happens, you need to trace stolen funds and identify the attacker.

### 🛠️ Step 1: Identify the Exploit Transaction

- Use Tenderly or Etherscan to locate the attack transaction:
  - Go to **Tenderly → Transactions**
  - Find the suspicious transaction.
  - Copy the attacker’s address.

### 🛠️ Step 2: Trace Stolen Funds Using Forta

Forta allows you to track where stolen ETH is sent.

#### 1️⃣ Create a Forta Bot to Detect Fund Movements

```javascript
const { Finding, HandleTransaction } = require("forta-agent");

const ATTACKER_ADDRESS = "0xAttackerAddress";

function handleTransaction(txEvent) {
  const findings = [];
  txEvent.traces.forEach((trace) => {
    if (trace.action.from === ATTACKER_ADDRESS) {
      findings.push(
        Finding.fromObject({
          name: "Stolen Funds Moved",
          description: `The attacker transferred funds to ${trace.action.to}`,
          alertId: "FORTA-3",
          severity: Finding.Severity.Critical,
        })
      );
    }
  });
  return findings;
}

module.exports = { handleTransaction };
```

#### 2️⃣ Deploy the Bot to Forta

```bash
forta publish
```

✅ Now, Forta will alert you whenever the attacker moves stolen funds.

### 🛠️ Step 3: Use Blockchain Explorers to Track Funds

Go to:

- 🔍 [Etherscan](https://etherscan.io)
- 🔍 [Tenderly](https://tenderly.co)

  - Enter the attacker’s wallet address.
  - See where the funds were sent.
  - Check if they moved to an exchange (possible cashout).

### 🛠️ Step 4: Notify Authorities & DeFi Platforms

If funds end up on an exchange, notify:

- ✔️ Binance
- ✔️ Coinbase
- ✔️ FTX (if still operational 😅)
- ✔️ DeFi Protocols (Maker, Aave, etc.)

They may freeze the attacker’s account before funds are cashed out.

---

## 🚀 Summary: Full Security & Forensics Checklist

| **Task**                            | **Tool Used**             |
| ----------------------------------- | ------------------------- |
| ✅ Set Up Real-Time Alerts          | Forta, Tenderly           |
| ✅ Detect Large Withdrawals         | Forta Security Bot        |
| ✅ Simulate Smart Contract Attacks  | Tenderly Sandbox          |
| ✅ Monitor Unauthorized Admin Calls | Tenderly Alerts           |
| ✅ Track Stolen Funds               | Etherscan, Forta          |
| ✅ Report Attacker Activity         | DeFi Platforms, Exchanges |
