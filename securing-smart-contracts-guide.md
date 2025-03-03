# 🔍 Deep Dive into Smart Contract Security Monitoring Tools: Forta & Tenderly

## 📌 Why Smart Contract Monitoring Matters

Traditional smart contract audits help detect vulnerabilities before deployment, but real-time monitoring is crucial to detect live threats like:

- ✅ Flash loan attacks
- ✅ Reentrancy exploits
- ✅ Price oracle manipulations
- ✅ Unauthorized contract interactions
- ✅ Large, unexpected token movements

Tools like Forta and Tenderly enable proactive security by monitoring on-chain activity and alerting users of suspicious behavior.

## 🛡️ 1️⃣ Forta: Real-Time Smart Contract Security Monitoring

### 📌 What is Forta?

Forta is a decentralized security network that:

- Monitors blockchain transactions
- Detects hacks, exploits, and anomalies
- Alerts users in real time

### 🛠️ How Forta Works

- **Detects Threats:** Monitors DeFi protocols for attacks.
- **Uses Bots:** Security researchers create custom bots to track specific threats.
- **Sends Alerts:** Notifications via email, Telegram, or Discord when anomalies occur.

### 📌 How to Set Up Forta for Smart Contract Monitoring

**1️⃣ Step: Install Forta CLI**

```bash
npm install -g forta-agent
forta init
```

This sets up Forta on your local machine.

**2️⃣ Step: Create a Custom Monitoring Bot**

_Example: Detecting Large Transfers from a DeFi Protocol_

```javascript
const { Finding, HandleTransaction } = require("forta-agent");

const HIGH_TRANSFER_THRESHOLD = 10000; // Alert for transfers over 10,000 tokens

function handleTransaction(txEvent) {
  const findings = [];
  txEvent.traces.forEach((trace) => {
    if (trace.action.value > HIGH_TRANSFER_THRESHOLD) {
      findings.push(
        Finding.fromObject({
          name: "Large Token Transfer",
          description: `A transfer of ${trace.action.value} tokens was detected`,
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

**3️⃣ Step: Deploy the Bot to Forta Network**

```bash
forta publish
```

Now, your bot will scan transactions in real-time and alert you if large transfers occur.

**4️⃣ Step: Connect Forta Alerts to Telegram**

1. Go to Forta App → Alerts
2. Set up a webhook to Telegram or Discord
3. Receive notifications when suspicious transactions happen! 🚨

## 🔍 2️⃣ Tenderly: Smart Contract Monitoring & Debugging

### 📌 What is Tenderly?

Tenderly is an all-in-one debugging and monitoring platform for Ethereum smart contracts.

### 🛠️ How Tenderly Works

- **Replays Transactions:** Simulates transactions without deploying to mainnet.
- **Monitors Events:** Tracks abnormal function calls & reverts.
- **Sends Alerts:** Notifies users of unexpected behaviors.

### 📌 How to Use Tenderly for Smart Contract Security

**1️⃣ Step: Set Up a Tenderly Account**

1. Sign up at [Tenderly](https://tenderly.co/)
2. Connect your Ethereum wallet
3. Import your contract address

**2️⃣ Step: Simulate a Smart Contract Attack**

Tenderly lets you test attacks in a sandbox before they happen on-chain.

- **Example: Testing a Reentrancy Attack**
  1. Load the vulnerable smart contract into Tenderly.
  2. Simulate calling `withdraw()` multiple times in one transaction.
  3. See if the contract allows reentrant withdrawals.

**3️⃣ Step: Set Up Real-Time Monitoring for Your Smart Contracts**

Tenderly lets you monitor live contract activity and get notified for:

- ✅ Failed transactions
- ✅ Large withdrawals
- ✅ Unauthorized admin function calls
- ✅ Unexpected gas spikes

**📌 Example: Setting Up an Alert for Large Withdrawals**

1. Go to Tenderly → Alerts
2. Create a New Alert
3. Set conditions:
   - **Contract:** `YourContract.sol`
   - **Function:** `withdraw()`
   - **Condition:** If amount > 1000 ETH, send alert
4. Choose Notification Method:
   - 📧 Email
   - 📩 Telegram
   - 🚨 Discord
5. Save & Enable Monitoring

## 🛠️ Forta vs. Tenderly: Which One Should You Use?

| Feature                        | Forta  | Tenderly |
| ------------------------------ | ------ | -------- |
| Real-Time Threat Detection     | ✅ Yes | ✅ Yes   |
| Transaction Simulation         | ❌ No  | ✅ Yes   |
| Reentrancy & Flash Loan Alerts | ✅ Yes | ✅ Yes   |
| On-Chain Attack Monitoring     | ✅ Yes | ✅ Yes   |
| Sandbox Testing                | ❌ No  | ✅ Yes   |
| Gas Usage Analysis             | ❌ No  | ✅ Yes   |
| Telegram/Discord Alerts        | ✅ Yes | ✅ Yes   |

**🛡️ Best Practice:**

- Use Forta for real-time attack detection (hacks, flash loans, reentrancy).
- Use Tenderly for debugging & transaction simulations before execution.

## 🚀 Summary: Best Practices for Smart Contract Security Monitoring

- ✅ Use Forta to detect live exploits & threats
- ✅ Set up Tenderly alerts for unusual contract interactions
- ✅ Monitor admin function calls & large transactions
- ✅ Simulate attacks in Tenderly before deployment
- ✅ Automate alerts for flash loan & price oracle manipulations
