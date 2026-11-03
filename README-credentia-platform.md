# 🎓 Credentia Platform

> **Issue, hold and verify credentials on Stellar. Issuers sign in with a wallet and anchor each credential on-chain, holders keep the document, and anyone can check it in seconds.**

![CI](https://github.com/eogenyi23-creator/credentia-platform/actions/workflows/ci.yml/badge.svg)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-blue.svg)
![Stellar](https://img.shields.io/badge/Built%20on-Stellar-0e75dd.svg)

| 🔗 Links | |
|---|---|
| 📜 [Registry Contract](https://github.com/eogenyi23-creator/credentia-registry) | 🧰 [Credentia SDK](https://github.com/eogenyi23-creator/credentia-sdk) |
| 📐 [Spec](docs/spec.md) | 🐛 [Report a Bug](https://github.com/eogenyi23-creator/credentia-platform/issues) |

**Jump to:** [About](#-about) · [Architecture](#-architecture) · [Features](#-features) · [How it works](#-how-it-works) · [Tech Stack](#️-tech-stack) · [Getting Started](#-getting-started) · [Security](#-security) · [Roadmap](#-roadmap)

---

## 📖 About

> Credentia Platform is the application layer of Credentia.

An issuer such as a school or training body creates a credential, the platform fingerprints it, and the issuer signs a transaction that anchors the fingerprint in the [credentia-registry](https://github.com/eogenyi23-creator/credentia-registry) Soroban contract. The holder gets a shareable link and a downloadable file. Anyone can open the public verify page, drop in a credential, and see whether it is valid, revoked or expired.

The platform is **non-custodial**. It never holds a secret key. For every on-chain action the API builds an unsigned transaction, the user signs it in their own wallet, and the web app submits it. The API then checks the transaction on-chain before it records anything.

---

## 🧩 Architecture

| Part | Purpose | Tech |
|---|---|---|
| `apps/web` | Issuer dashboard, holder page, public verify page | Next.js, Tailwind |
| `apps/api` | Auth, credential storage, transaction building, on-chain checks | Node.js, Express, Drizzle, PostgreSQL |
| `packages/shared` | Types and validation shared by web and API | TypeScript, Zod |

Hashing, verification and transaction building come from [credentia-sdk](https://github.com/eogenyi23-creator/credentia-sdk), so the platform and any other app compute the same fingerprint.

---

## ✨ Features

| | Feature | Details |
|:-:|---|---|
| 🔐 | **Wallet sign-in** | Freighter, xBull and Albedo through a signed challenge; no passwords |
| 🏫 | **Issuer onboarding** | register an issuer profile; an admin accredits it on-chain |
| 🧾 | **Issue credentials** | fill a form, preview the document, sign once to anchor it |
| 📦 | **Bulk issue** | upload a CSV and issue many credentials, signing each batch in the wallet |
| 👤 | **Holder page** | a shareable link and a QR code for each credential, and a JSON download |
| ✅ | **Public verify page** | paste an ID or drop a credential file; the page recomputes the hash and checks the contract |
| 🚫 | **Revocation** | issuers revoke with a reason; the verify page shows it right away |
| 📜 | **Audit log** | every issue and revoke is recorded with its transaction hash and a link to the explorer |
| 🔒 | **No personal data on-chain** | only the fingerprint is anchored |

---

## 🔄 How it works

```
Issuer fills form -> API builds credential + hash -> API builds unsigned `issue` tx
-> issuer signs in wallet -> web submits to Soroban RPC -> API verifies on-chain, stores record
Verifier opens link or drops file -> hash recomputed -> registry `verify` -> result shown
```

---

## 🛠️ Tech Stack

TypeScript everywhere, Next.js 14, Express, Drizzle ORM, PostgreSQL 16, `@stellar/stellar-sdk`, `@creit.tech/stellar-wallets-kit`, Jest, GitHub Actions CI.

---

## 🚀 Getting Started

### 📋 Prerequisites

- Node.js 20+
- Docker (for PostgreSQL)
- A deployed `credentia-registry` contract on testnet
- A funded Stellar testnet key for `DEPLOYER_SECRET` — it only pays for platform-side transactions, it never holds user funds

### ⚙️ Setup

```bash
git clone https://github.com/eogenyi23-creator/credentia-platform.git
cd credentia-platform
npm ci
cp .env.example .env      # fill in JWT_SECRET, JWT_REFRESH_SECRET and REGISTRY_CONTRACT_ID
docker compose up -d
npm run db:migrate --workspace=apps/api
npm run dev
```

The web app runs on port 3000 and the API on port 5000.

> **Setup notes**
>
> - The API reads `.env` from `apps/api`, so link or copy the root `.env` there, and to `apps/web/.env.local`.
> - In GitHub Codespaces, make port 5000 public and set `NEXT_PUBLIC_API_URL` and `CORS_ORIGIN` to the forwarded addresses with no trailing slash. Restart after changing `.env`.

### 🧭 Try it on testnet

| Step | What to do |
|:-:|---|
| **1️⃣** | Connect a wallet and sign in. Albedo works in any browser. |
| **2️⃣** | Promote your user to admin: `psql "$DATABASE_URL" -c "UPDATE users SET role = 'admin' WHERE stellar_address = '<YOUR_STELLAR_ADDRESS>';"` |
| **3️⃣** | Register an issuer profile, then accredit it from the admin page. |
| **4️⃣** | Issue a credential and sign the transaction. |
| **5️⃣** | Open the holder link, then open `/verify` in a private window and check it. |
| **6️⃣** | Revoke it and check again. |

### 🔑 Environment variables

| Variable | Required | Description |
|---|:-:|---|
| `DATABASE_URL` | **Yes** | PostgreSQL connection string |
| `JWT_SECRET` | **Yes** | Secret for access tokens |
| `JWT_REFRESH_SECRET` | **Yes** | Secret for refresh tokens |
| `REGISTRY_CONTRACT_ID` | **Yes** | The deployed `credentia-registry` contract |
| `STELLAR_NETWORK` | No (default `testnet`) | `testnet` or `mainnet` |
| `SOROBAN_RPC_URL` | No | Soroban RPC endpoint |
| `CORS_ORIGIN` | No | Allowed web origin |
| `NEXT_PUBLIC_API_URL` | No (default `http://localhost:5000`) | API address used by the web app |
| `NEXT_PUBLIC_EXPLORER_URL` | No | Explorer base URL for transaction links |

### 📟 Scripts

| Command | What it does |
|---|---|
| `npm run dev` | Start the API and web app together |
| `npm run lint` | Lint every workspace |
| `npm run typecheck` | Build shared packages, then typecheck every workspace |
| `npm test` | Run all tests |
| `npm run build` | Build everything |

---

## 🔐 Security

- Non-custodial: no secret key is ever sent to the API.
- Every recorded issue or revoke is checked against the contract on-chain first.
- Credential documents are stored off-chain; the chain holds only the fingerprint.
- See [SECURITY.md](SECURITY.md) to report a vulnerability.

> ⚠️ **This is unaudited testnet software.**

---

## 🗺️ Roadmap

Not built yet:

- [ ] Email delivery of holder links
- [ ] Issuer domain verification
- [ ] Selective disclosure
- [ ] Mainnet deployment guide

---

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Pull requests target `main` and CI must pass.

## 📜 License

[MIT](LICENSE) © Credentia
