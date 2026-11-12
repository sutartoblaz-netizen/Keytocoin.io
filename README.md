# KEYTOCOIN — Sepolia public testnet candidate

**Status: testnet candidate, not audited, not Mainnet-ready.** This repository is designed to let the team compile, test locally, deploy to Ethereum Sepolia, and run a first public test. It does not claim that any contract has already been deployed or audited.

## Locked project parameters

- ERC-20: `KeytoCoin (KTC)`; fixed total supply **45,000,000 KTC**.
- Community mining pool: **40,000,000 KTC**, prefunded into the mining contract.
- Owner allocation: **5,000,000 KTC**, irrevocable vesting with 12-month cliff followed by linear vesting over the next 36 months (48 months total).
- Reward fee: **5%** of gross reward to treasury; **95%** to miner.
- Initial test defaults: 1 KTC gross reward per wallet per epoch; 60-second epoch; configurable PoW target. These are test settings, not final economic parameters.
- CPU proof: `keccak256(abi.encode(chainId, miningContract, wallet, epoch, nonce)) <= powTarget`.
- Claim requires at least 3 unique, sorted EIP-712 signatures from the fixed five-attestor set.
- Emergency pause requires 3 unique, sorted EIP-712 signatures from the fixed five-guardian set.
- AI/security monitor is read-only. It has no key, no signer, no transaction method, and no contract administration permission.

## Important trust and decentralization limits

1. The token contract has no mint function after constructor and no upgrade/admin controls. It has no owner.
2. Mining contract has no mint function. Rewards can only be paid from its prefunded 40m pool.
3. The owner of mining is initially the deployer so the deployment can configure the guardian council. The script then starts a two-step ownership transfer to the Safe. The Safe must execute `acceptOwnership()` before the project can claim that administrative control is held by Safe.
4. Five attestor operators must be independent people/organizations, with separate signing keys, hosts, and operational controls. Three URLs run by one operator do not provide decentralization.
5. The current Telegram binding is enforced independently by each attestor's local database. It is not proof of one-human-one-account and cannot prevent all Sybil accounts. Community policy and additional anti-abuse work remain necessary.
6. Each attestor verifies Telegram `initData` independently. The bot owner should keep `TELEGRAM_BOT_TOKEN` private and run `node scripts/derive-telegram-secret.js` locally. Give the resulting `TELEGRAM_INITDATA_SECRET` to the five attestors over a secure channel; do not share the bot token itself. The derived verifier secret is still sensitive because it can validate/forge Telegram initData assertions. The bot owner keeps `TELEGRAM_BOT_TOKEN` only in the root `.env`; attestor operators put only the derived secret in `attestor/.env`. Do not send either secret to ChatGPT or commit either to Git.
7. Telegram Mini App requires a public HTTPS URL and a WalletConnect Cloud project ID (public client ID) or an injected EVM provider. If Telegram's in-app browser does not expose `window.ethereum`, WalletConnect connects an external EVM wallet while preserving Telegram `initData` in the Mini App context.
8. The included monitor is rule-based read-only monitoring, not a magical autonomous AI. An LLM can later summarize alerts, but must remain isolated from keys and transaction execution.
9. A fixed five-key attestor/guardian set is not automatically decentralized. Key custody, rotation, recovery, and operator succession need a separately reviewed governance process. This version intentionally has immutable signer sets.

## Requirements

- Node.js 20+ and npm.
- Sepolia RPC URL.
- A fresh testnet deployer wallet with Sepolia ETH only.
- A Safe multisig deployed on Sepolia.
- Five independent attestor addresses and five independent guardian addresses. These should not be the same five people unless the community explicitly accepts that concentration.
- Treasury and vesting beneficiary addresses.

## 1. Install and test locally

```bash
npm install
cp .env.example .env
npm test
npm run compile
```

The local tests check fixed supply, vesting schedule, 95/5 split, attestor quorum, duplicate-signature rejection, replay rejection, and guardian quorum. Tests are a baseline, not a security audit.

## 2. Configure Sepolia

Edit `.env` locally. Never paste secrets into chat or commit `.env`.

Required:
- `SEPOLIA_RPC_URL`
- `DEPLOYER_PRIVATE_KEY` (testnet-only wallet)
- `SAFE_ADDRESS`
- `TREASURY_ADDRESS`
- `OWNER_BENEFICIARY`
- `ATTESTOR_ADDRESSES` (exactly five unique addresses)
- `GUARDIAN_ADDRESSES` (exactly five unique addresses)

Optional test parameters: `GROSS_REWARD_KTC=1`, `EPOCH_SECONDS=60`, `POW_TARGET=0x0000ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff`.

The sample target is deliberately easy for a demo. Do not use it for Mainnet. Benchmark mining and choose target/epoch/reward economics after testnet measurements and community review.

## 3. Deploy to Sepolia

Fund the deployer with Sepolia ETH from a reputable faucet. Confirm all destination addresses with their owners. Then:

```bash
npm run deploy:sepolia
npm run verify:sepolia
```

The script refuses any chain other than chain ID 11155111. It deploys token, vesting, mining, and guardian council; moves 40m KTC to the mining pool and 5m to vesting; configures the guardian; and proposes Safe ownership transfer.

**Safe acceptance is a separate required step:** the Safe must execute `acceptOwnership()` on the mining contract. Until then, the deployer remains the current owner. Verify the result with `owner()` and `pendingOwner()`.

The deployment record is written to `deployments/sepolia.json`. Keep it and publish it with the test report only after verifying every address.

## 4. Run an independent attestor

Each of the five operators should clone the same reviewed commit on their own infrastructure. In that clone, copy `attestor/.env.example` to `attestor/.env`, then set:

- `TELEGRAM_INITDATA_SECRET` — derived by the bot owner; see trust note above.
- `ATTESTOR_PRIVATE_KEY` — only that operator's attestor key.
- `MONITOR_RPC_URL` or `SEPOLIA_RPC_URL`.
- `KTC_MINING_ADDRESS`.
- `ATTESTOR_PORT` (default 8790).

Then run:

```bash
npm install
npm run attestor
```

Check `GET /health`. Expose only through HTTPS reverse proxy with rate limiting, request-size limits, firewall, monitoring, and a stable URL. Do not run all five keys on one phone/server. Do not share private keys among operators. The local `attestor/telegram-wallet-bindings.json` file is sensitive operational data; back it up securely and restrict file permissions. Set `ATTESTOR_CORS_ORIGIN` to the exact HTTPS origin serving the Mini App.

## 5. Telegram Mini App

1. Copy `web/config.example.js` to `web/config.js` and paste your WalletConnect Cloud project ID. The project ID is public configuration; it is not a private key.
2. Host the `web/` directory (including `config.js`) on a public HTTPS domain.
3. Configure the bot's Mini App domain in BotFather and set `WEBAPP_URL` in `.env`.
4. Run `node telegram-bot.js` on the bot operator's machine.
5. Open the bot in a private Telegram chat and send `/start`.
6. Open the Mini App, connect a Sepolia-compatible wallet using WalletConnect or an injected provider, paste at least three independent attestor `/attest` URLs, and start CPU mining.
7. Claim only after 3-of-5 signatures are collected. Review the wallet's transaction details before signing.

The HTML page prompts for the verified mining contract address once and stores it in browser local storage. The contract address must be checked against the deployment record and a block explorer before use.

## 6. Emergency guardian pause signing

Each guardian copies `guardian/.env.example` to `guardian/.env` on their own device, fills their own `GUARDIAN_PRIVATE_KEY`, `KTC_GUARDIAN_ADDRESS`, and `KTC_MINING_ADDRESS`, then runs `node scripts/sign-guardian-pause.js`. All guardians must use the same `GUARDIAN_PAUSE_DEADLINE` Unix timestamp. They send only the resulting EIP-712 signature and public metadata to the incident coordinator. Collect three signatures for the same mining address, nonce, and deadline; sort them by recovered guardian address and save a JSON file with `{ "mining": "0x...", "deadline": 123, "signatures": ["0x...", "0x...", "0x..."] }`. An executor can submit it with `npx hardhat run scripts/execute-guardian-pause.js --network sepolia -- signatures.json`. The guardian private keys never leave their owners' devices.

## 7. Read-only security monitor

Set `MONITOR_RPC_URL`, `KTC_TOKEN_ADDRESS`, `KTC_MINING_ADDRESS`, and `KTC_GUARDIAN_ADDRESS`, then:

```bash
npm run monitor
```

It checks supply, pool balance, paused state, ownership transfer status, reward events, community watchlist matches, dust transfers, and simple address-poisoning lookalike patterns. The watchlist in `monitor/known-bad-addresses.json` must be maintained by the community with evidence; heuristic matches are warnings, not proof of theft. Alerts are appended to `monitor/security-events.jsonl`. It never signs or sends transactions.

## 8. Before inviting public testers

- [ ] Compile and run tests on a clean machine/CI.
- [ ] Deploy to Sepolia and verify source code on Etherscan.
- [ ] Safe accepts mining ownership; verify `owner()` is Safe.
- [ ] Verify token supply = 45m, mining pool = 40m, vesting = 5m.
- [ ] Confirm five attestor addresses and five guardian addresses with operators out-of-band.
- [ ] Run five independent attestor services and test failure of 0, 1, and 2 services; claim must fail. Test 3 and 4 valid signers; claim should pass.
- [ ] Test invalid Telegram initData, expired auth, wallet mismatch, stale epoch, stale nonce, duplicate claim, invalid proof, expired signature, and replay.
- [ ] Test guardian 2-of-5 fails and 3-of-5 pauses; test owner/Safe resume procedure.
- [ ] Load test CPU worker and RPC endpoints; check reward pool exhaustion and fee accounting.
- [ ] Run static analysis (Slither), fuzz/invariant tests, dependency audit, and independent contract review.
- [ ] Publish bug bounty and incident-response process.

## 9. Mainnet gate

Do not deploy Mainnet until the Sepolia test report is complete, all high/critical findings are fixed, an independent audit has been published, the Safe and guardian processes are rehearsed, and legal/community disclosures are approved. Sepolia tests do not establish security or value on Mainnet.
