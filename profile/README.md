# StellFlow

Cross-border payroll, invoicing and escrow for freelancers and the clients who
hire them, settling in USDC on Stellar. Cross-border freelance payments today
lose a meaningful cut to intermediary banks and FX spreads, take days to
settle, and give the freelancer no recourse if a client doesn't pay. StellFlow
holds the client's funds in a Soroban escrow contract, releases them on
milestone approval (or by dispute resolution), and keeps the invoicing and
records off-chain in a REST API and web app.

> **Status: testnet only, unaudited, no production users.** The escrow contract
> is deployed to Stellar **testnet** with a verified build; the API runs locally
> and has no deployment; the web app is live but its dashboard runs on labelled
> sample data. No security review has been performed on any of the three
> repositories. Do not use any of this with real funds.

## Proof

| | |
|---|---|
| Escrow contract (testnet) | `CA77HTQMZAFBU5GVVFOEHT6AGCOVZJ2MXSEZ33DJJSZWY6NFFPPI67RS` — [stellar.expert](https://stellar.expert/explorer/testnet/contract/CA77HTQMZAFBU5GVVFOEHT6AGCOVZJ2MXSEZ33DJJSZWY6NFFPPI67RS) · [Stellar Lab](https://lab.stellar.org/r/testnet/contract/CA77HTQMZAFBU5GVVFOEHT6AGCOVZJ2MXSEZ33DJJSZWY6NFFPPI67RS) |
| Build verification | [SEP-0055](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0055.md): the on-chain WASM (`47b29590…8192`) carries a GitHub [attestation](https://github.com/Steller-Flow/stellflow-smartcontract/attestations) linking it to tag [`v1.0.0`](https://github.com/Steller-Flow/stellflow-smartcontract/tree/v1.0.0), commit `17d980a`, built by [`release.yml`](https://github.com/Steller-Flow/stellflow-smartcontract/blob/main/.github/workflows/release.yml). Check it yourself: `stellar contract info build --id CA77HTQMZAFBU5GVVFOEHT6AGCOVZJ2MXSEZ33DJJSZWY6NFFPPI67RS --network testnet` |
| Live web app | https://stellflow.vercel.app — the landing page reads the contract above over Soroban RPC from your browser (admin, paused, version, storage TTL, latest ledger, `get_escrow` lookup) |
| CI | [![contract](https://github.com/Steller-Flow/stellflow-smartcontract/actions/workflows/ci.yml/badge.svg)](https://github.com/Steller-Flow/stellflow-smartcontract/actions/workflows/ci.yml) [![backend](https://github.com/Steller-Flow/stellflow-backend/actions/workflows/ci.yml/badge.svg)](https://github.com/Steller-Flow/stellflow-backend/actions/workflows/ci.yml) [![frontend](https://github.com/Steller-Flow/stellflow-frontend/actions/workflows/ci.yml/badge.svg)](https://github.com/Steller-Flow/stellflow-frontend/actions/workflows/ci.yml) |
| Tests | 114 (contract) + 125 (backend) + 65 unit and 3 browser (frontend), all run in CI on every push |
| Licence | MIT in all three repositories |

## Repositories

StellFlow is three separate repositories, not a monorepo.

| Repository | What it is | Stack | State today |
|---|---|---|---|
| [stellflow-smartcontract](https://github.com/Steller-Flow/stellflow-smartcontract) | The Soroban escrow contract that holds the funds. 32 exported functions: single and multi-milestone escrows, disputes, deadlines, fees, pause, storage TTL, versioning. | Rust, Soroban SDK, stellar-cli | Deployed to testnet with a SEP-0055 verified build. 114 tests in 9 suites. CI runs fmt, clippy, tests, WASM build and `cargo audit`. [README](https://github.com/Steller-Flow/stellflow-smartcontract#readme) has the full function reference and state machine. |
| [stellflow-backend](https://github.com/Steller-Flow/stellflow-backend) | The REST API: accounts and JWT sessions, invoices, an off-chain mirror of escrow state keyed by `txHash` / `contractId`, payment records, wallet linking, notifications, analytics, audit log. | TypeScript, Express 5, Prisma 7, PostgreSQL, socket.io, zod | 42 mounted endpoints with an OpenAPI 3.0 spec at `/api-docs`. 125 tests. Boots and serves requests against a local Postgres; **no deployment**. On-chain verification is a stub — it records whatever `txHash` it is sent ([#44](https://github.com/Steller-Flow/stellflow-backend/issues/44)). |
| [stellflow-frontend](https://github.com/Steller-Flow/stellflow-frontend) | The web app: landing page, Freighter wallet connect and onboarding, a dashboard for invoices, escrows, analytics and notifications. | Next.js 16, React 19, Tailwind 4, `@stellar/stellar-sdk`, `@stellar/freighter-api`, Playwright | Live on Vercel. Reads the deployed contract over Soroban RPC. Dashboard runs on in-memory sample data behind a `NEXT_PUBLIC_DEMO_MODE` flag with a banner on every page; it does not call the API or sign transactions yet. 65 unit tests + Playwright suite against a production build. |

Each repository has its own `README`, `CONTRIBUTING.md`, `SECURITY.md` and
`LICENSE`; the `README` in each states what is wired and what is not.

## Capabilities

✅ implemented and tested · ⚠️ partial · ❌ not started

| Capability | Status | Where / notes |
|---|---|---|
| Escrow lifecycle on-chain: create → fund → release / refund / cancel / modify | ✅ | contract; deployed |
| Multi-milestone escrows: submit / approve / reject / release per milestone | ✅ | contract (`create_escrow_with_milestones`, `release_milestone`) |
| Disputes with resolution by release, refund or proportional split | ✅ | contract; resolved by the contract **admin** |
| Per-escrow arbiter | ⚠️ | `set_arbiter` stores an address; `resolve_dispute` only accepts the admin |
| Deadlines and client timeout refund | ✅ | contract (`set_deadline`, `claim_timeout`) |
| Configurable platform fee (0–10 %) to a treasury, per-escrow and default | ✅ | contract |
| Emergency pause, storage TTL + expired-escrow cleanup, version + `migrate` | ✅ | contract |
| Role-based access control | ⚠️ | roles can be assigned and queried; no guard checks them, only the admin address is enforced |
| Reproducible, attested contract build (SEP-0055) | ✅ | `release.yml` → GitHub attestation → on-chain `source_repo` |
| REST API: auth, users, wallet linking, invoices, escrow mirror, payments, notifications, analytics, audit log | ✅ | backend, 42 endpoints, OpenAPI; runs locally |
| Backend verifies `txHash` / `contractId` against the network | ❌ | backend [#44](https://github.com/Steller-Flow/stellflow-backend/issues/44); records what it is told |
| Wallet-signature verification when linking an address | ❌ | backend [#41](https://github.com/Steller-Flow/stellflow-backend/issues/41) |
| Real-time notifications | ⚠️ | backend runs a socket.io server with a JWT handshake and emits `invoice:*`, `escrow:*`, `payment:*`, `notification:*` from its controllers; **the frontend has no socket.io client** (not even a dependency), so nothing consumes them |
| Web app reads the contract (admin, paused, version, TTL, ledger, `get_escrow`) | ✅ | frontend, Soroban RPC from the browser; landing page and `/dashboard/escrows` |
| Freighter wallet connect + onboarding | ✅ | frontend; Albedo / WalletConnect are placeholders ([#50](https://github.com/Steller-Flow/stellflow-frontend/issues/50)) |
| Invoice and escrow UI (create, list, filter, wizard, detail) | ⚠️ | frontend; in-memory sample data, labelled; not persisted |
| Web app builds and signs contract transactions (fund / release from the browser) | ❌ | frontend [#47](https://github.com/Steller-Flow/stellflow-frontend/issues/47) |
| Web app talks to the API | ❌ | frontend [#46](https://github.com/Steller-Flow/stellflow-frontend/issues/46); the client exists but has no callers and doesn't match the routes |
| Security audit | ❌ | none of the three repositories |
| Mainnet deployment | ❌ | |

## Roadmap

- [x] Escrow contract deployed to Stellar testnet
- [x] Reproducible, SEP-0055 attested contract build
- [x] Multi-milestone escrows
- [x] Dispute resolution (admin: release / refund / split)
- [x] Deadlines, timeout claims, platform fee, pause, TTL, versioning
- [x] REST API with OpenAPI spec and JWT auth, running locally
- [x] Web app live, Freighter connect, reads the contract over Soroban RPC
- [ ] Web app signs and submits `fund_escrow` / `release` through Freighter (frontend [#47](https://github.com/Steller-Flow/stellflow-frontend/issues/47))
- [ ] Web app wired to the API; sample data replaced by records (frontend [#46](https://github.com/Steller-Flow/stellflow-frontend/issues/46), [#52](https://github.com/Steller-Flow/stellflow-frontend/issues/52))
- [ ] Backend verifies transactions and contract state on-chain (backend [#44](https://github.com/Steller-Flow/stellflow-backend/issues/44))
- [ ] Real-time updates consumed by the web app (socket.io client)
- [ ] Per-escrow arbiter enforced in `resolve_dispute`; role checks enforced
- [ ] Backend deployment
- [ ] Security audit
- [ ] Mainnet deployment
- [ ] Automated payroll and recurring payments
- [ ] Fiat on/off ramps (NGN, KES, GHS, ZAR)
- [ ] Tax documentation and compliance tools
- [ ] Mobile app
- [ ] Agency management features

## Contributing

Issues are scoped with file references, acceptance criteria and verification
commands. Comment on an issue before starting; each repository's
`CONTRIBUTING.md` has the local checks CI will run.

| Repository | Open issues | `good first issue` | Guide |
|---|---|---|---|
| stellflow-smartcontract | [28](https://github.com/Steller-Flow/stellflow-smartcontract/issues) | [16](https://github.com/Steller-Flow/stellflow-smartcontract/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) | [CONTRIBUTING.md](https://github.com/Steller-Flow/stellflow-smartcontract/blob/main/CONTRIBUTING.md) |
| stellflow-backend | [12](https://github.com/Steller-Flow/stellflow-backend/issues) | [3](https://github.com/Steller-Flow/stellflow-backend/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) | [CONTRIBUTING.md](https://github.com/Steller-Flow/stellflow-backend/blob/main/CONTRIBUTING.md) |
| stellflow-frontend | [12](https://github.com/Steller-Flow/stellflow-frontend/issues) | [4](https://github.com/Steller-Flow/stellflow-frontend/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) | [CONTRIBUTING.md](https://github.com/Steller-Flow/stellflow-frontend/blob/main/CONTRIBUTING.md) |

Counts are as of this page's last edit; the links are live.

To report a vulnerability, use the private channel in the repository's
`SECURITY.md`
([contract](https://github.com/Steller-Flow/stellflow-smartcontract/blob/main/SECURITY.md) ·
[backend](https://github.com/Steller-Flow/stellflow-backend/blob/main/SECURITY.md) ·
[frontend](https://github.com/Steller-Flow/stellflow-frontend/blob/main/SECURITY.md))
rather than a public issue.

## How the pieces fit

1. The **client's wallet** signs `create_escrow` / `fund_escrow` against the
   contract; the **freelancer's wallet** signs `release`; disputes go to the
   contract admin. The API never holds keys.
2. The **web app** is where both wallets sign, and then reports the resulting
   `txHash` / `contractId` to the API. Today it does the wallet connect and
   the contract reads; the signing and the API calls are the open items above.
3. The **API** records the state, links it to the invoice, writes a payment row
   and an audit entry, and notifies the counterparty. Today it trusts what it
   is sent; verifying against the network is backend #44.

## Licence

MIT, in each repository.
