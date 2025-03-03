# 🚨 The Most Commonly Exploited Smart Contract Vulnerabilities

Smart contract security is critical to prevent hacks, financial losses, and system failures. Below is a list of the most exploited vulnerabilities, their impact, and how to prevent them.

## 📌 1. Reentrancy Attacks

**🛠️ What is it?**

- A malicious contract repeatedly calls a vulnerable function before the first execution completes, draining funds.

**💥 Real-World Example:**

- The DAO Hack (2016) – $60M Stolen

**❌ Vulnerable Code:**

```solidity
function withdraw(uint amount) public {
    require(balance[msg.sender] >= amount, "Insufficient balance");
    msg.sender.call{value: amount}("");  // 🚨 External call before state update!
    balance[msg.sender] -= amount;  // ❌ Updates balance after sending ETH
}
```

````

**✅ Fix:**

- Use OpenZeppelin’s `ReentrancyGuard` modifier.
- Update state before making external calls.

```solidity
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SecureBank is ReentrancyGuard {
    function withdraw(uint amount) public nonReentrant {
        require(balance[msg.sender] >= amount, "Insufficient balance");
        balance[msg.sender] -= amount;  // ✅ Update state before transfer
        payable(msg.sender).transfer(amount);
    }
}
```

## 📌 2. Integer Overflows & Underflows

**🛠️ What is it?**

- An overflow occurs when a number exceeds its max value.
- An underflow happens when a number goes below zero, causing unexpected results.

**💥 Real-World Example:**

- The BeautyChain Hack (2018) – $500K Stolen

**❌ Vulnerable Code:**

```solidity
function transfer(address to, uint amount) public {
    balance[msg.sender] -= amount;  // ❌ Can underflow to a large number!
    balance[to] += amount;
}
```

**✅ Fix:**

- Use SafeMath (Solidity >=0.8.0 automatically prevents overflows).
- Check inputs before arithmetic operations.

```solidity
function transfer(address to, uint amount) public {
    require(balance[msg.sender] >= amount, "Insufficient funds");
    unchecked {
        balance[msg.sender] -= amount;
        balance[to] += amount;
    }
}
```

## 📌 3. Unchecked External Calls (`call()`)

**🛠️ What is it?**

- Using `call()` instead of `transfer()` or `send()` allows attackers to manipulate transactions.

**💥 Real-World Example:**

- Parity Multisig Hack (2017) – $150M Stolen

**❌ Vulnerable Code:**

```solidity
to.call{value: amount}("");  // ❌ Can fail unexpectedly, locking funds
```

**✅ Fix:**

- Use `transfer()` instead, which limits gas usage and prevents funds from getting stuck.
- Check the return value of `call()` if it’s necessary to use.

```solidity
payable(to).transfer(amount);  // ✅ Safe transfer method
```

## 📌 4. Flash Loan Attacks (Oracle Manipulation)

**🛠️ What is it?**

- Flash loans allow attackers to borrow massive funds and manipulate on-chain prices, draining lending pools.

**💥 Real-World Example:**

- bZx Flash Loan Attack (2020) – $8M Stolen

**❌ Vulnerable Code:**

```solidity
function getPrice() public view returns (uint) {
    (uint112 reserve0, uint112 reserve1, ) = uniswap.getReserves();
    return reserve1 / reserve0;  // 🚨 Price is directly fetched from Uniswap!
}
```

**✅ Fix:**

- Use Chainlink Oracles instead of on-chain DEX prices.
- Implement Time-Weighted Average Price (TWAP) to smooth price fluctuations.

```solidity
import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";

AggregatorV3Interface public priceFeed;
function getPrice() public view returns (uint) {
    (, int price, , , ) = priceFeed.latestRoundData();
    return uint(price);
}
```

## 📌 5. Access Control Vulnerabilities

**🛠️ What is it?**

- Lack of access control allows unauthorized users to call sensitive functions.

**💥 Real-World Example:**

- Poly Network Hack (2021) – $600M Stolen

**❌ Vulnerable Code:**

```solidity
function setAdmin(address newAdmin) public {
    admin = newAdmin;  // ❌ No access control!
}
```

**✅ Fix:**

- Restrict admin functions using `onlyOwner`.
- Use OpenZeppelin’s `Ownable` contract.

```solidity
import "@openzeppelin/contracts/access/Ownable.sol";

contract SecureAdmin is Ownable {
    function setAdmin(address newAdmin) public onlyOwner {
        admin = newAdmin;
    }
}
```

## 📌 6. Self-Destruct Vulnerability

**🛠️ What is it?**

- The `selfdestruct()` function allows the contract to be deleted, potentially locking user funds.

**💥 Real-World Example:**

- Parity Wallet Hack (2017) – $150M Lost

**❌ Vulnerable Code:**

```solidity
function destroy() public {
    selfdestruct(payable(owner));  // ❌ Can be called by anyone!
}
```

**✅ Fix:**

- Remove `selfdestruct()` unless absolutely necessary.
- Restrict access using `onlyOwner`.

```solidity
function destroy() public onlyOwner {
    selfdestruct(payable(owner));
}
```

## 📌 7. Improper Randomness

**🛠️ What is it?**

- Using `block.timestamp` for randomness allows attackers to manipulate results.

**💥 Real-World Example:**

- Fomo3D Lottery Exploit

**❌ Vulnerable Code:**

```solidity
function random() public view returns (uint) {
    return uint(keccak256(abi.encodePacked(block.timestamp, msg.sender)));  // ❌ Predictable randomness
}
```

**✅ Fix:**

- Use Chainlink VRF (Verifiable Random Function) for unpredictable randomness.

```solidity
import "@chainlink/contracts/src/v0.8/VRFConsumerBase.sol";

contract SecureRandom is VRFConsumerBase {
    function requestRandomness() public {
        uint256 randomNumber = uint256(keccak256(abi.encodePacked(block.difficulty, block.timestamp)));
    }
}
```

## 🚀 Summary: Security Checklist for Common Vulnerabilities

| Vulnerability                  | Prevention Method                                                           |
| ------------------------------ | --------------------------------------------------------------------------- |
| **Reentrancy**                 | Use `ReentrancyGuard`, update state before external calls                   |
| **Integer Overflow/Underflow** | Use Solidity >=0.8.0, apply `SafeMath` library                              |
| **Unchecked External Calls**   | Use `transfer()`, avoid `call()`                                            |
| **Flash Loan Attacks**         | Use Chainlink Oracles, implement Time-Weighted Average Price (TWAP) pricing |
| **Access Control Issues**      | Implement `onlyOwner` modifier, role-based access controls                  |
| **Self-Destruct Exploits**     | Restrict or remove use of `selfdestruct()`                                  |
| **Weak Randomness**            | Use Chainlink Verifiable Random Function (VRF) for secure randomness        |

````
