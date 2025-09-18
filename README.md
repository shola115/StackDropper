#  Airdrop Distribution Smart Contract

## Overview

This is a **Clarity smart contract** for managing the secure and flexible distribution of fungible tokens via an airdrop on the **Stacks blockchain**. It supports eligibility control, tiered rewards, emergency withdrawals, pausing/resuming distribution, and reclaiming unclaimed tokens.

---

## 🚀 Features

* ✅ Airdrop distribution to eligible recipients
* 🔒 Access control via `CONTRACT-OWNER`
* 🔁 Tiered claim system with multipliers
* 🧾 Claim tracking (one claim per recipient)
* ⏸️ Pause & resume functionality
* 🔐 Emergency timelock withdrawals
* 🔥 Burn unclaimed tokens after a defined reclaim period
* 📜 Full event logging
* 🧪 Read-only functions for data access

---

## 📦 Initialization

Upon deployment:

* 1,000,000,000 tokens of `airdrop-distribution-token` are minted to `CONTRACT-OWNER`
* `is-airdrop-active` is set to `true`
* `reclaim-period-length` defaults to `10,000` blocks

---

## 📄 Contract Constants

| Constant         | Description                                  |
| ---------------- | -------------------------------------------- |
| `CONTRACT-OWNER` | Set as `tx-sender` at deployment             |
| `TIMELOCK-DELAY` | 144 blocks (24h assuming 10-minute blocks)   |
| `ERROR-*`        | Custom error codes for clarity and debugging |

---

## 🧠 State Variables

| Variable                       | Type   | Description                             |
| ------------------------------ | ------ | --------------------------------------- |
| `is-airdrop-active`            | `bool` | Controls claim access                   |
| `is-paused`                    | `bool` | Temporary pause state                   |
| `airdrop-amount-per-recipient` | `uint` | Base claimable amount                   |
| `total-tokens-distributed`     | `uint` | Tracks distributed tokens               |
| `airdrop-start-block`          | `uint` | Block when airdrop started              |
| `reclaim-period-length`        | `uint` | Blocks before reclaim allowed           |
| `emergency-timelock`           | `uint` | Block after which withdrawal is allowed |
| `eligible-airdrop-recipients`  | `map`  | Tracks who can claim                    |
| `claimed-airdrop-amounts`      | `map`  | Tracks how much has been claimed        |
| `tier-multipliers`             | `map`  | Tier system for bonus rewards           |
| `contract-events`              | `map`  | Logged events by ID                     |

---

## ⚙️ Public Functions

### 🛡️ Admin Functions

| Function                               | Description               |
| -------------------------------------- | ------------------------- |
| `add-eligible-recipient(principal)`    | Add a single recipient    |
| `remove-eligible-recipient(principal)` | Remove recipient          |
| `bulk-add-eligible-recipients(list)`   | Batch add (max 200)       |
| `update-airdrop-amount(uint)`          | Update base amount        |
| `update-reclaim-period(uint)`          | Update reclaim period     |
| `pause-airdrop()`                      | Temporarily pause         |
| `resume-airdrop()`                     | Resume airdrop            |
| `set-tier-multiplier(uint, uint)`      | Set multiplier for a tier |

---

### 💰 Claim Functions

| Function                           | Description                         |
| ---------------------------------- | ----------------------------------- |
| `claim-airdrop-tokens()`           | Standard one-time claim             |
| `claim-tiered-airdrop(tier-level)` | Claim with multiplier based on tier |

---

### 🔥 Reclaim & Emergency

| Function                                  | Description                                    |
| ----------------------------------------- | ---------------------------------------------- |
| `reclaim-unclaimed-tokens()`              | Burn all unclaimed tokens after reclaim period |
| `initiate-emergency-withdrawal()`         | Starts timelock before emergency withdrawal    |
| `execute-emergency-withdrawal(principal)` | Transfers all remaining tokens after timelock  |

---

## 🔎 Read-only Functions

| Function                                   | Returns                        |
| ------------------------------------------ | ------------------------------ |
| `get-airdrop-active-status()`              | `bool`                         |
| `get-pause-status()`                       | `bool`                         |
| `get-tier-multiplier(uint)`                | `uint`                         |
| `get-emergency-timelock()`                 | `uint`                         |
| `is-recipient-eligible(principal)`         | `bool`                         |
| `has-recipient-claimed-airdrop(principal)` | `bool`                         |
| `get-recipient-claimed-amount(principal)`  | `uint`                         |
| `get-total-tokens-distributed()`           | `uint`                         |
| `get-airdrop-amount-per-recipient()`       | `uint`                         |
| `get-reclaim-period()`                     | `uint`                         |
| `get-airdrop-start-block()`                | `uint`                         |
| `get-event(uint)`                          | `{event-type, data}` or `none` |

---

## 📚 Event Logging

Events are logged using a centralized ID counter. Logged types include:

* `"recipient-add"`
* `"recipient-remove"`
* `"bulk-recipients-add"`
* `"amount-updated"`
* `"period-updated"`
* `"tokens-claimed"`
* `"tiered-claim"`
* `"tokens-reclaimed"`
* `"emerg-withdrawal"`
* `"emerg-executed"`
* `"airdrop-paused"`
* `"airdrop-resumed"`

Use `get-event(uint)` to fetch logs.

---

## 🧪 Example Flow

1. **Admin Setup**

   ```clojure
   (add-eligible-recipient 'SP...') ;; Add users
   (update-airdrop-amount u200) ;; Optional update
   ```

2. **User Claims Tokens**

   ```clojure
   (claim-airdrop-tokens) ;; Claim once
   ;; or
   (claim-tiered-airdrop u2) ;; With tier multiplier
   ```

3. **Pause/Resume by Admin**

   ```clojure
   (pause-airdrop)
   (resume-airdrop)
   ```

4. **Reclaim After Expiry**

   ```clojure
   (reclaim-unclaimed-tokens) ;; Burn leftovers after `reclaim-period-length`
   ```

5. **Emergency Withdrawal**

   ```clojure
   (initiate-emergency-withdrawal)
   ;; Wait 144 blocks
   (execute-emergency-withdrawal 'SP...')
   ```

---

## 🛑 Error Codes

| Error Code | Meaning                 |
| ---------- | ----------------------- |
| `u100`     | Not contract owner      |
| `u101`     | Already claimed         |
| `u102`     | Not eligible            |
| `u103`     | Insufficient balance    |
| `u104`     | Airdrop inactive        |
| `u105`     | Invalid amount          |
| `u106`     | Reclaim too early       |
| `u107`     | Invalid recipient       |
| `u108`     | Invalid period          |
| `u109`     | Already paused          |
| `u110`     | Not paused              |
| `u111`     | Emergency not initiated |
| `u112`     | Timelock not reached    |

---

## 🏁 Deployment Checklist

* ✅ Deploy contract with `CONTRACT-OWNER` as deployer
* ✅ Mint tokens: done automatically on deploy
* ✅ Set `airdrop-amount-per-recipient`
* ✅ Add eligible recipients
* ✅ Optionally configure tier multipliers

---

## 🛠️ Development Notes

* Written in **Clarity**, the smart contract language of the Stacks blockchain.
* Designed for **security**, **auditability**, and **upgradability**.
* Compatible with **Clarity SDKs** and **Stacks blockchain tools**.

---

## 📬 License

MIT License. Free to use and modify with attribution.

---
