# DApp Development Learning Exercises

A comprehensive, progressive learning path for mastering DApp development.

## What You're Building

You're building **client-side only DApps** -- web applications that run entirely in the browser and talk directly to the blockchain. There is **no backend server**.

```
┌─────────────────────────────────────────────────────┐
│                Traditional Web App                   │
│                                                      │
│   Browser  ──→  Your Server  ──→  Your Database     │
│            ←──  (Express,    ←──  (PostgreSQL,      │
│                  Next.js)         MongoDB)           │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│          DApp (what you're building)                  │
│                                                      │
│   Browser  ──────────────→  Blockchain               │
│   (React)  ←──────────────  (Ethereum / Sepolia)    │
│      │                          │                    │
│      │  wallet = your auth      │  smart contracts   │
│      │  RPC = your API          │  = your backend    │
│      │  static hosting = done   │  = your database   │
└─────────────────────────────────────────────────────┘
```

**Key implications of this architecture:**
- **No Express, no Next.js, no server-side API routes, no SSR.** The output is a static folder of HTML/JS/CSS that you upload to any static host (Vercel, Netlify, IPFS).
- **Authentication is wallet-based.** The user's wallet (MetaMask, etc.) signs transactions. There are no usernames, passwords, or sessions.
- **The blockchain is your database.** Smart contracts store and enforce your application logic. Reading data is free; writing costs gas.
- **The RPC provider is your API.** Your frontend talks to the blockchain through an RPC provider (Alchemy) -- think of it as a read/write API to Ethereum.

This is why the monorepo has only two parts: `apps/frontend` (the static React app) and `packages/contracts` (the Solidity contracts + Hardhat tooling). No `apps/api`, no `apps/server`.

---

## Overview

This repository contains a structured learning path designed to get an introduction to DApp development. The curriculum is based on **production-ready DApp setup guides** and modern web3 tooling including:

- **RainbowKit 2.x** - Beautiful wallet connection UI
- **Wagmi 2.x** - React hooks for Ethereum
- **Viem 2.x** - TypeScript Ethereum library
- **React 18** - UI library
- **React Router DOM 6** - Client-side routing with BrowserRouter
- **Vite 5+** - Build tool and development server
- **TypeScript** - Type-safe development (strict mode)
- **TanStack Query 5.x** - Data fetching and caching
- **Zustand 4.x** - Lightweight state management
- **Hardhat 2.x** - Smart contract development
- **Solidity 0.8.x** - Smart contract language
- **OpenZeppelin Contracts 5.x** - Security-audited contract libraries

> **Important**: This learning path is based on the setup guides in `dapp_setup_guides/`. Always refer to those guides for the **single source of truth** on scaffolding, architecture, and best practices.

## Learning Path Structure

The learning path consists of **9 progressive modules**:

1. **Environment Setup & Project Scaffolding**
   - Development environment setup
   - Project initialization
   - Development tools configuration

2. **Web3 Fundamentals & Wallet Integration**
   - Blockchain basics
   - RainbowKit integration
   - Wallet connection

3. **Smart Contract Development & Deployment**
   - Solidity fundamentals
   - Hardhat setup, compilation, and testing
   - Deploying to local and testnet
   - OpenZeppelin contracts & upgradeable patterns
   - ABI sharing with the frontend
   - Contract verification (Sourcify + Blockscout)

4. **Reading Blockchain Data**
   - Reading contract state
   - Using WAGMI hooks
   - Event listening

5. **Writing to Blockchain (Transactions)**
   - Transaction lifecycle
   - Writing to contracts
   - Gas estimation
   - Transaction feedback

6. **Advanced Contract Interactions**
   - ERC-20/ERC-721 interactions
   - Multi-step transactions (approval patterns)
   - Real-time event system (WebSocket)

7. **State Management & Data Fetching**
   - Global state management
   - Data caching
   - Optimistic updates

8. **User Experience & Best Practices**
   - Error handling
   - Loading states
   - Responsive design
   - Connection health monitoring

9. **Advanced Topics & Production Readiness**
   - Build versioning
   - Testing strategies
   - Production builds & deployment

## File Structure & Organization

This repository is organized with numbered files for easy navigation. Here's how the files are structured:

### Root Level Files (Learning Materials)

```
learningdappdevelopment/
├── 001_INTRO.md                        # Start here! Main entry point and overview
├── 002_DAPP_LEARNING_PATH.md           # Complete curriculum overview with all 9 modules
│
├── 003_ENVIRONMENT_SETUP.md            # Environment Setup & Project Scaffolding
├── 004_WEB3_WALLET_INTEGRATION.md      # Web3 Fundamentals & Wallet Integration
├── 005_SMART_CONTRACT_DEV.md           # Smart Contract Development & Deployment
├── 006_READING_BLOCKCHAIN.md           # Reading Blockchain Data
├── 007_WRITING_TRANSACTIONS.md         # Writing to Blockchain (Transactions)
├── 008_ADVANCED_INTERACTIONS.md        # Advanced Contract Interactions
├── 009_STATE_MANAGEMENT.md             # State Management & Data Fetching
├── 010_UX_BEST_PRACTICES.md            # User Experience & Best Practices
├── 011_PRODUCTION_READINESS.md         # Advanced Topics & Production Readiness
│
└── 012_QUICK_REFERENCE.md              # Quick lookup guide (use while working)
```

### Setup Guides (Single Source of Truth)

```
dapp_setup_guides/
├── 001_scaffolding_specs.md         # Tech stack, architecture, and patterns
├── 002_setup_instructions_and_best_practices.md  # Complete setup guide
└── bc_infrastructure_services.md    # RPC provider comparison (reference)
```

### Understanding the Numbering

- **001-002**: Core learning materials (README and Learning Path overview)
- **003-011**: Sequential module exercises (follow in order)
- **012**: Quick Reference (use anytime as a lookup guide)
- **dapp_setup_guides/**: Authoritative setup guides (referenced throughout exercises)

### How to Use These Files

1. **Start with `001_INTRO.md`** - Read the overview and prerequisites
2. **Read `002_DAPP_LEARNING_PATH.md`** - Understand the full curriculum
3. **Follow exercises sequentially** - Start with `003_ENVIRONMENT_SETUP.md`, then `004_WEB3_WALLET_INTEGRATION.md`, etc.
4. **Keep `012_QUICK_REFERENCE.md` open** - Use it as a cheat sheet while working
5. **Refer to setup guides** - When exercises mention setup guides, check `dapp_setup_guides/` folder

> **Tip**: The numbered files (001-012) are your learning materials. The `dapp_setup_guides/` folder contains the authoritative technical specifications that all exercises are based on.

## Documentation

### Learning Path
- **[002_DAPP_LEARNING_PATH.md](./002_DAPP_LEARNING_PATH.md)** - Complete learning path with all modules, exercises, and concepts
- **[003_ENVIRONMENT_SETUP.md](./003_ENVIRONMENT_SETUP.md)** - Environment Setup & Project Scaffolding
- **[004_WEB3_WALLET_INTEGRATION.md](./004_WEB3_WALLET_INTEGRATION.md)** - Web3 Fundamentals & Wallet Integration
- **[005_SMART_CONTRACT_DEV.md](./005_SMART_CONTRACT_DEV.md)** - Smart Contract Development & Deployment
- **[006_READING_BLOCKCHAIN.md](./006_READING_BLOCKCHAIN.md)** - Reading Blockchain Data
- **[007_WRITING_TRANSACTIONS.md](./007_WRITING_TRANSACTIONS.md)** - Writing to Blockchain (Transactions)
- **[008_ADVANCED_INTERACTIONS.md](./008_ADVANCED_INTERACTIONS.md)** - Advanced Contract Interactions
- **[009_STATE_MANAGEMENT.md](./009_STATE_MANAGEMENT.md)** - State Management & Data Fetching
- **[010_UX_BEST_PRACTICES.md](./010_UX_BEST_PRACTICES.md)** - User Experience & Best Practices
- **[011_PRODUCTION_READINESS.md](./011_PRODUCTION_READINESS.md)** - Advanced Topics & Production Readiness
- **[012_QUICK_REFERENCE.md](./012_QUICK_REFERENCE.md)** - Quick reference guide for common tasks

### Setup Guides (Single Source of Truth)
- **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
- **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Complete setup instructions and best practices

### Reference Documentation
- **[bc_infrastructure_services.md](./dapp_setup_guides/bc_infrastructure_services.md)** - Blockchain infrastructure services comparison and information (RPC providers, WebSocket support, pricing, use cases)

> **Critical**: The setup guides (001 and 002) are the **single source of truth** for scaffolding and setup. The infrastructure services document provides comparison information to help understand provider options.

## Getting Started

1. **Start here**: Read `002_DAPP_LEARNING_PATH.md` to understand the full curriculum
2. **Begin with Environment Setup**: Follow `003_ENVIRONMENT_SETUP.md` step by step
3. **Progress Sequentially**: Complete each module before moving to the next
4. **Ask Questions**: Don't hesitate to ask for help when stuck
5. **Practice**: Build small projects between modules to reinforce learning

## Prerequisites

Before starting, make sure you have:

- **Node.js 18+** installed
- **pnpm 8+** installed
- Basic knowledge of JavaScript/TypeScript
- Familiarity with command line
- Understanding of Git basics
- Basic React knowledge (helpful but not required)
- **WalletConnect Project ID** (get from [WalletConnect Cloud](https://cloud.walletconnect.com/))
- **Alchemy API Key** (get from [Alchemy](https://www.alchemy.com/))

> **Note**: This learning path uses a **monorepo structure** with pnpm workspaces. Familiarity with monorepos is helpful but not required.

## Learning Objectives

By the end of this learning path, you will be able to:

- Understand the client-side DApp architecture (no backend, blockchain as database, wallet as auth)
- Set up a complete DApp development environment
- Write, test, and deploy Solidity smart contracts
- Use OpenZeppelin libraries and upgradeable patterns
- Integrate wallet connections using RainbowKit
- Read data from smart contracts
- Send transactions to smart contracts
- Handle complex contract interactions
- Manage application state effectively
- Create production-ready DApps with excellent UX
- Build and deploy to static hosting (Vercel, Netlify, IPFS)

## Self-Assessment

Each module includes:

- **Checkpoints** - Verify understanding at key points
- **Review Questions** - Test conceptual knowledge
- **Hands-on Exercises** - Practical implementation
- **Submission Checklist** - Ensure completeness

## Tools & Technologies

### Core Stack (Monorepo Structure)
- **Package Manager**: **pnpm** (required - workspace support)
- **Monorepo**: pnpm workspace with `apps/frontend` and `packages/contracts`
- **React 18.2+** - UI library
- **React Router DOM 6.20+** - Client-side routing (BrowserRouter)
- **Vite 5.4+** - Build tool and dev server
- **TypeScript 5.2+** - Type safety (strict mode)
- **Tailwind CSS 3.x or 4.x** - Utility-first CSS

### Web3 Stack
- **Wagmi 2.5+** - React hooks for Ethereum
- **Viem 2.7+** - TypeScript Ethereum library
- **RainbowKit 2.0+** - Wallet connection UI
- **TanStack Query 5.85+** - Data fetching and caching
- **Zustand 4.4+** - Lightweight state management

### Smart Contract Development
- **Hardhat 2.26+** - Ethereum development environment
- **Solidity 0.8.22+** - Smart contract language
- **OpenZeppelin Contracts 5.4+** - Security-audited libraries
- **OpenZeppelin Upgradeable** - Upgradeable contract patterns
- **@openzeppelin/hardhat-upgrades** - Required for proxy deployments

### Development Tools
- **VS Code / Cursor** - Code editor
- **ESLint** - Linting (flat config format)
- **Prettier** - Code formatting
- **Husky + lint-staged** - Pre-commit hooks
- **Git** - Version control

### RPC Provider
- **Alchemy** - Primary RPC provider (WebSocket desktop, HTTP mobile)
- **Free Tier**: 300M Compute Units/month

### Testing & Deployment
- **Vitest** - Unit testing (frontend)
- **Hardhat Tests** - Contract testing (Mocha + Chai + ethers.js)
- **Static Hosting** - IPFS, Vercel, Netlify (SPA with `_redirects`)

### Required Patterns (Production-Ready)
- **Transaction Overlay System** - Required for production UX
- **Real-Time Event System** - Separate WebSocket client for events
- **Connection Health Monitoring** - RPC health checks and reconnection
- **Scopes Pattern** - TanStack Query cache invalidation
- **Build Versioning** - Build counter system

## Resources

### Documentation
- [RainbowKit Docs](https://rainbowkit.com/)
- [WAGMI Docs](https://wagmi.sh/)
- [Viem Docs](https://viem.sh/)
- [React Docs](https://react.dev/)
- [React Router Docs](https://reactrouter.com/)
- [Vite Docs](https://vitejs.dev/)
- [Ethereum.org](https://ethereum.org/en/developers/)
- [Solidity Docs](https://docs.soliditylang.org/)
- [OpenZeppelin Docs](https://docs.openzeppelin.com/contracts/)
- [Hardhat Docs](https://hardhat.org/docs)

### Communities
- [Ethereum Stack Exchange](https://ethereum.stackexchange.com/)
- [r/ethdev](https://www.reddit.com/r/ethdev/)
- [Discord: Ethereum](https://discord.gg/ethereum)

---

**Happy Learning!**

*Remember: Learning DApp development is a marathon, not a sprint. Take your time, practice regularly, and don't be afraid to make mistakes. That's how you learn!*
