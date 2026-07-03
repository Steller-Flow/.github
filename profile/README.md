# StellFlow — Cross-Border Payments for the Global Workforce


StellFlow is a cross-border payroll, invoice, and escrow platform built for African freelancers, remote workers, agencies, and global clients. It replaces slow, expensive, and opaque international payment rails with near-instant blockchain settlement using Stellar and USDC stablecoin.

The platform solves a real problem: freelancers in Africa lose 5-15% of earnings to transfer fees, face 3-5 day settlement times, and have no recourse when clients don't pay. StellFlow eliminates middlemen, cuts costs to fractions of a cent, and settles payments in seconds — while adding milestone-based escrow protection so both sides are safe.

## How It Works

### For Freelancers
1. **Connect a Stellar wallet** (Freighter, Lobstr, etc.) and complete onboarding
2. **Create invoices** with line items, send to clients, and track payment status
3. **Work within escrow agreements** — funds are locked before work begins, released on milestone approval
4. **Receive USDC instantly** with transparent transaction history and earnings analytics

### For Clients
1. **Fund escrows** with USDC on Stellar — funds are held securely in a smart contract
2. **Approve milestones** to release funds, or dispute if work isn't delivered
3. **Pay invoices** with one click — no bank transfers, no wire fees, no waiting
4. **Track spending** across freelancers, projects, and time periods

### The Escrow Flow
1. Client creates an escrow agreement specifying freelancer, amount, and milestones
2. Client funds the escrow — USDC is transferred into the smart contract
3. Freelancer completes milestones and submits for review
4. Client approves (releases funds) or disputes (triggers resolution)
5. Funds settle instantly to the freelancer's Stellar wallet

## Architecture

StellFlow is built as a monorepo with three packages:

### Smart Contract (`stellflow-smartcontract`)
- Written in **Rust** using the **Soroban SDK** for the Stellar blockchain
- Implements the full escrow lifecycle: create, fund, release, refund
- Handles on-chain state transitions, event emission, and token transfers
- Enforces authorization checks so only the correct parties can act

### Backend (`stellflow-backend`)
- Built with **TypeScript**, **Express 5**, and **Prisma ORM** on **PostgreSQL**
- Exposes a RESTful API for users, invoices, escrows, payments, and notifications
- Integrates with the Stellar SDK for blockchain interactions and wallet verification
- Handles authentication (JWT + bcrypt), validation (Zod), rate limiting, and audit logging

### Frontend (`stellflow-frontend`)
- Built with **Next.js 16** (App Router), **React 19**, and **TypeScript**
- Styled with **Tailwind CSS 4** and animated with **Framer Motion**
- Connects to Stellar wallets via the **Freighter API**
- State managed with **Zustand**, forms validated with **React Hook Form + Zod**

## Key Features

- **Instant settlement** — USDC on Stellar settles in 3-5 seconds vs. 3-5 days for bank transfers
- **Near-zero fees** — Stellar transaction fees are fractions of a cent
- **Milestone-based escrow** — Funds are locked before work begins, released on approval
- **Transparent history** — All transactions are on-chain and auditable
- **Multi-currency support** — Accept and send USDC, with fiat on/off ramps planned
- **Role-based access** — Freelancers, clients, and admins each have tailored experiences
- **Real-time notifications** — WebSocket-powered updates for payments and status changes
- **Analytics dashboard** — Track earnings, spending, and transaction history

## Why Stellar?

- **Speed** — 3-5 second finality vs. minutes/hours on other chains
- **Cost** — Transaction fees under $0.01
- **Stability** — USDC on Stellar is a regulated, fully-reserved stablecoin
- **Simplicity** — Stellar's account model is easier for non-crypto users than EVM wallets
- **Soroban** — Smart contracts are written in Rust, compile to WASM, and are cost-effective

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Smart Contract | Rust, Soroban SDK |
| Blockchain | Stellar (testnet → mainnet) |
| Backend | TypeScript, Express 5, Prisma, PostgreSQL |
| Frontend | Next.js 16, React 19, Tailwind CSS, Framer Motion |
| Wallet | Freighter API, Stellar SDK |
| Validation | Zod |
| State | Zustand |
| Auth | JWT, bcryptjs |

## Roadmap

- [ ] Smart contract deployment to Stellar testnet and mainnet
- [ ] Multi-milestone escrow support
- [ ] Dispute resolution with arbiter system
- [ ] WebSocket real-time updates
- [ ] Automated payroll and recurring payments
- [ ] Fiat on/off ramps (NGN, KES, GHS, ZAR)
- [ ] Tax documentation and compliance tools
- [ ] Mobile app (React Native)
- [ ] AI-powered bookkeeping and expense categorization
- [ ] Agency management features

## Getting Started

Each package has its own setup instructions. See the README in each directory for details:

- `stellflow-smartcontract/` — Rust toolchain, Soroban CLI, Stellar testnet account
- `stellflow-backend/` — Node.js, PostgreSQL, environment variables
- `stellflow-frontend/` — Node.js, npm/yarn

## License

MIT
