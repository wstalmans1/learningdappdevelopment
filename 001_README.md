# DApp Development Learning Exercises

A comprehensive, progressive learning path for mastering DApp development.

## 📚 Overview

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
- **OpenZeppelin Contracts 5.x** - Security-audited contract libraries

> **📖 Important**: This learning path is based on the setup guides in `dapp_setup_guides/`. Always refer to those guides for the **single source of truth** on scaffolding, architecture, and best practices.

## 🗺️ Learning Path Structure

The learning path consists of **8 progressive modules**:

1. **Environment Setup & Project Scaffolding**
   - Development environment setup
   - Project initialization
   - Development tools configuration

2. **Web3 Fundamentals & Wallet Integration**
   - Blockchain basics
   - RainbowKit integration
   - Wallet connection

3. **Reading Blockchain Data**
   - Reading contract state
   - Using WAGMI hooks
   - Event listening

4. **Writing to Blockchain (Transactions)**
   - Transaction lifecycle
   - Writing to contracts
   - Gas estimation
   - Transaction feedback

5. **Advanced Contract Interactions**
   - ERC-20/ERC-721 interactions
   - Multi-step transactions
   - Contract factories

6. **State Management & Data Fetching**
   - Global state management
   - Data caching
   - Optimistic updates

7. **User Experience & Best Practices**
   - Error handling
   - Loading states
   - Responsive design
   - Accessibility

8. **Advanced Topics & Production Readiness**
   - Security best practices
   - Testing strategies
   - CI/CD pipelines
   - Monitoring

## 📁 File Structure & Organization

This repository is organized with numbered files for easy navigation. Here's how the files are structured:

### Root Level Files (Learning Materials)

```
Learning_exercises/
├── 001_README.md                    # 📖 Start here! Main entry point and overview
├── 002_DAPP_LEARNING_PATH.md        # 🗺️ Complete curriculum overview with all 8 modules
│
├── 003_MODULE_1_EXERCISES.md        # 📝 Detailed step-by-step exercises for Module 1
├── 004_MODULE_2_EXERCISES.md        # 📝 Detailed step-by-step exercises for Module 2
├── 005_MODULE_3_EXERCISES.md        # 📝 Detailed step-by-step exercises for Module 3
├── 006_MODULE_4_EXERCISES.md        # 📝 Detailed step-by-step exercises for Module 4
├── 007_MODULE_5_EXERCISES.md        # 📝 Detailed step-by-step exercises for Module 5
├── 008_MODULE_6_EXERCISES.md        # 📝 Detailed step-by-step exercises for Module 6
├── 009_MODULE_7_EXERCISES.md        # 📝 Detailed step-by-step exercises for Module 7
├── 010_MODULE_8_EXERCISES.md        # 📝 Detailed step-by-step exercises for Module 8
│
└── 011_QUICK_REFERENCE.md            # 🔍 Quick lookup guide (use while working)
```

### Setup Guides (Single Source of Truth)

```
dapp_setup_guides/
├── 001_scaffolding_specs.md         # 🏗️ Tech stack, architecture, and patterns
├── 002_setup_instructions_and_best_practices.md  # ⚙️ Complete setup guide
└── bc_infrastructure_services.md    # 📊 RPC provider comparison (reference)
```

### Understanding the Numbering

- **001-002**: Core learning materials (README and Learning Path overview)
- **003-010**: Sequential module exercises (follow in order: Module 1 → Module 2 → ... → Module 8)
- **011**: Quick Reference (use anytime as a lookup guide)
- **dapp_setup_guides/**: Authoritative setup guides (referenced throughout exercises)

### How to Use These Files

1. **Start with `001_README.md`** - Read the overview and prerequisites
2. **Read `002_DAPP_LEARNING_PATH.md`** - Understand the full curriculum
3. **Follow exercises sequentially** - Start with `003_MODULE_1_EXERCISES.md`, then `004_MODULE_2_EXERCISES.md`, etc.
4. **Keep `011_QUICK_REFERENCE.md` open** - Use it as a cheat sheet while working
5. **Refer to setup guides** - When exercises mention setup guides, check `dapp_setup_guides/` folder

> **💡 Tip**: The numbered files (001-011) are your learning materials. The `dapp_setup_guides/` folder contains the authoritative technical specifications that all exercises are based on.

## 📖 Documentation

### Learning Path
- **[002_DAPP_LEARNING_PATH.md](./002_DAPP_LEARNING_PATH.md)** - Complete learning path with all modules, exercises, and concepts
- **[003_MODULE_1_EXERCISES.md](./003_MODULE_1_EXERCISES.md)** - Detailed exercises for Module 1: Environment Setup & Project Scaffolding
- **[004_MODULE_2_EXERCISES.md](./004_MODULE_2_EXERCISES.md)** - Detailed exercises for Module 2: Web3 Fundamentals & Wallet Integration
- **[005_MODULE_3_EXERCISES.md](./005_MODULE_3_EXERCISES.md)** - Detailed exercises for Module 3: Reading Blockchain Data
- **[006_MODULE_4_EXERCISES.md](./006_MODULE_4_EXERCISES.md)** - Detailed exercises for Module 4: Writing to Blockchain (Transactions)
- **[007_MODULE_5_EXERCISES.md](./007_MODULE_5_EXERCISES.md)** - Detailed exercises for Module 5: Advanced Contract Interactions
- **[008_MODULE_6_EXERCISES.md](./008_MODULE_6_EXERCISES.md)** - Detailed exercises for Module 6: State Management & Data Fetching
- **[009_MODULE_7_EXERCISES.md](./009_MODULE_7_EXERCISES.md)** - Detailed exercises for Module 7: User Experience & Best Practices
- **[010_MODULE_8_EXERCISES.md](./010_MODULE_8_EXERCISES.md)** - Detailed exercises for Module 8: Advanced Topics & Production Readiness
- **[011_QUICK_REFERENCE.md](./011_QUICK_REFERENCE.md)** - Quick reference guide for common tasks

### Setup Guides (Single Source of Truth)
- **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
- **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Complete setup instructions and best practices

### Reference Documentation
- **[bc_infrastructure_services.md](./dapp_setup_guides/bc_infrastructure_services.md)** - Blockchain infrastructure services comparison and information (RPC providers, WebSocket support, pricing, use cases)

> **⚠️ Critical**: The setup guides (001 and 002) are the **single source of truth** for scaffolding and setup. The infrastructure services document provides comparison information to help understand provider options.

## 🚀 Getting Started

1. **Start with Module 1**: Read `002_DAPP_LEARNING_PATH.md` to understand the full curriculum
2. **Complete Module 1 Exercises**: Follow `003_MODULE_1_EXERCISES.md` step by step
3. **Progress Sequentially**: Complete each module before moving to the next
4. **Ask Questions**: Don't hesitate to ask for help when stuck
5. **Practice**: Build small projects between modules to reinforce learning

## 📋 Prerequisites

Before starting, make sure you have:

- **Node.js 18+** installed
- **pnpm 8+** installed (required - not npm/yarn)
- Basic knowledge of JavaScript/TypeScript
- Familiarity with command line
- Understanding of Git basics
- Basic React knowledge (helpful but not required)
- **WalletConnect Project ID** (get from [WalletConnect Cloud](https://cloud.walletconnect.com/))
- **Alchemy API Key** (get from [Alchemy](https://www.alchemy.com/))

> **Note**: This learning path uses a **monorepo structure** with pnpm workspaces. Familiarity with monorepos is helpful but not required.

## 🎯 Learning Objectives

By the end of this learning path, you will be able to:

- ✅ Set up a complete DApp development environment
- ✅ Integrate wallet connections using RainbowKit
- ✅ Read data from smart contracts
- ✅ Send transactions to smart contracts
- ✅ Handle complex contract interactions
- ✅ Manage application state effectively
- ✅ Create production-ready DApps with excellent UX
- ✅ Deploy and monitor DApps in production

## 📝 Self-Assessment

Each module includes:

- **Checkpoints** - Verify understanding at key points
- **Review Questions** - Test conceptual knowledge
- **Hands-on Exercises** - Practical implementation
- **Submission Checklist** - Ensure completeness

## 🔧 Tools & Technologies

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
- **VS Code** - Code editor
- **ESLint** - Linting (flat config format)
- **Prettier** - Code formatting
- **Husky + lint-staged** - Pre-commit hooks
- **Git** - Version control

### RPC Provider
- **Alchemy** - Primary RPC provider (WebSocket desktop, HTTP mobile)
- **Free Tier**: 300M Compute Units/month

### Testing & Deployment
- **Vitest** - Unit testing (frontend)
- **Hardhat Tests** - Contract testing
- **Foundry** (optional) - Fast contract tests
- **Static Hosting** - IPFS, Vercel, Netlify (SPA with `_redirects`)

### Required Patterns (Production-Ready)
- **Transaction Overlay System** - Required for production UX
- **Real-Time Event System** - Separate WebSocket client for events
- **Connection Health Monitoring** - RPC health checks and reconnection
- **Scopes Pattern** - TanStack Query cache invalidation
- **Build Versioning** - Build counter system

## 📚 Resources

### Documentation
- [RainbowKit Docs](https://rainbowkit.com/)
- [WAGMI Docs](https://wagmi.sh/)
- [Viem Docs](https://viem.sh/)
- [React Docs](https://react.dev/)
- [React Router Docs](https://reactrouter.com/)
- [Vite Docs](https://vitejs.dev/)
- [Ethereum.org](https://ethereum.org/en/developers/)

### Communities
- [Ethereum Stack Exchange](https://ethereum.stackexchange.com/)
- [r/ethdev](https://www.reddit.com/r/ethdev/)
- [Discord: Ethereum](https://discord.gg/ethereum)

---

**Happy Learning! 🚀**

*Remember: The journey from zero to hero is a marathon, not a sprint. Take your time, practice regularly, and don't be afraid to make mistakes. That's how you learn!*
