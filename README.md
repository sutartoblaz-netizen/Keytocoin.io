# 🪙 KEYTOCOIN (KTC) — Post-Quantum ML-DSA-65 Wallet & Node

KEYTOCOIN is an experimental blockchain network implementation that integrates the post-quantum cryptography standard (**NIST FIPS 204 / ML-DSA-65**) directly in the browser, powered by a lightweight Node.js backend, SHA3-256 hashing, and CPU mining simulation.

---

## 🚀 Key Features

- **Post-Quantum Cryptography:** Utilizes the **ML-DSA-65** digital signature algorithm via `@noble/post-quantum` to secure wallets and transactions against quantum computing threats.
- **Local Client Encryption:** Private keys are stored locally in the browser using **AES-256-GCM** encryption with **PBKDF2-SHA256 (310,000 iterations)** key derivation.
- **SHA3-256 Address Derivation:** Wallet addresses are cryptographically derived from public keys using the SHA3-256 hash function.
- **CPU Mining Simulation:** Coin mining feature directly from the node dashboard with a maximum token supply cap of 45,000,000 KTC.
- **Block Explorer & Mempool:** Real-time network status monitoring, block height, pending transactions (*mempool*), and block history.

---

## 📂 Project Structure

Ensure your project directory structure looks like this:

```text
ktc-node/
├── public/
│   └── index.html      # Frontend web interface (HTML + CSS + ES Module)
├── server.js           # Node.js backend server (Express)
├── config.js           # Network parameter & port configuration (Optional)
├── package.json        # Node.js project dependencies & scripts
└── README.md           # Project documentation
