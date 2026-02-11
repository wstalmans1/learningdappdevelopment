# DApp Development Learning Path

A comprehensive, progressive curriculum for mastering decentralized application (DApp) development — from zero to production-ready.

## Live Site

**[learningdappdevelopment.eth.limo](https://learningdappdevelopment.eth.limo)**

The full curriculum is available as a browsable web interface hosted on IPFS via ENS.

## What This Is

A structured set of learning exercises that guide you through building **client-side only DApps** — web applications that run entirely in the browser and interact directly with the blockchain. No backend server required.

The curriculum covers:

1. **Environment Setup** — Tooling, project scaffolding, dev workflow
2. **Web3 & Wallet Integration** — Connecting to wallets, reading accounts
3. **Smart Contract Development** — Writing, compiling, deploying with Foundry
4. **Reading Blockchain Data** — Querying on-chain state, events, logs
5. **Writing Transactions** — Sending transactions, handling confirmations
6. **Advanced Interactions** — Multi-contract patterns, gas estimation
7. **State Management** — Caching, syncing, optimistic updates
8. **UX Best Practices** — Error handling, loading states, responsive design
9. **Production Readiness** — Testing, builds, deployment

## Repository Structure

```
├── 001_INTRO.md                  # Start here — overview & prerequisites
├── 002_DAPP_LEARNING_PATH.md     # Full curriculum overview
├── 003–011_*.md                  # Sequential learning modules
├── 012_QUICK_REFERENCE.md        # Cheat sheet for use while coding
├── dapp_setup_guides/            # Authoritative setup & architecture specs
└── index.html                    # Web frontend (served on IPFS/ENS)
```

## Getting Started

1. Visit **[learningdappdevelopment.eth.limo](https://learningdappdevelopment.eth.limo)** to browse the content online, or clone this repo and open the markdown files directly.
2. Start with `001_INTRO.md` for the overview and prerequisites.
3. Read `002_DAPP_LEARNING_PATH.md` to understand the full curriculum.
4. Follow the modules sequentially from `003` through `011`.
5. Keep `012_QUICK_REFERENCE.md` handy as a cheat sheet.

## Tech Stack (What You'll Learn)

- **Frontend:** React + TypeScript + Vite
- **Smart Contracts:** Solidity + Foundry
- **Blockchain Interaction:** wagmi + viem
- **Wallet Connection:** RainbowKit / ConnectKit
- **RPC Providers:** Alchemy / Infura

## License

MIT
