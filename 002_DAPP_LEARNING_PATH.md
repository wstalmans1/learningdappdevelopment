# DApp Development Learning Path: Zero to Hero

A progressive series of exercises designed to take you from zero knowledge to mastering DApp development using **production-ready setup guides** and modern web3 tooling.

> **Critical**: This learning path is based on the setup guides in `dapp_setup_guides/`. Always refer to those guides for the **single source of truth** on scaffolding, architecture, and best practices.
>
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Complete setup instructions
> - **[bc_infrastructure_services.md](./dapp_setup_guides/bc_infrastructure_services.md)** - Blockchain infrastructure services comparison (reference for understanding RPC provider options)

## Overview

This learning path is structured in **9 progressive modules**, each building upon the previous one. Each module contains:
- **Learning objectives**
- **Prerequisites**
- **Hands-on exercises**
- **Key concepts to master**
- **Checkpoint assessment**

> **Architecture note:** Every DApp you build in this curriculum is **client-side only**. There is no backend server, no SSR, no server-side API. The React frontend runs entirely in the browser, talks to the blockchain via an RPC provider (Alchemy), and uses the user's wallet for authentication. The build output is a static folder you deploy to any static host. See `001_README.md` for the full architecture diagram.

---

## Environment Setup & Project Scaffolding

**Difficulty:** Beginner

### Learning Objectives
- Understand the client-side DApp architecture: no backend server, blockchain = database, wallet = auth
- Set up a modern development environment for DApp development
- Understand the toolchain: Node.js, npm/yarn/pnpm, Git
- Initialize a React project with Vite and TypeScript
- Set up React Router DOM 6 with BrowserRouter
- Configure ESLint, Prettier, and basic project structure

### Prerequisites
- Basic knowledge of JavaScript/TypeScript
- Familiarity with command line
- Git basics

### Exercises

#### Exercise 1.1: Initial Environment Setup
1. Install Node.js (v18+), npm/yarn/pnpm
2. Install Git and configure it
3. Install VS Code with recommended extensions (ESLint, Prettier, Solidity)
4. Create a GitHub account and set up SSH keys

#### Exercise 1.2: Project Initialization (Monorepo Structure)
1. Create monorepo root with pnpm workspace:
   ```bash
   mkdir dapp-learning && cd dapp-learning
   pnpm init
   ```
2. Create `pnpm-workspace.yaml`:
   ```yaml
   packages:
     - 'apps/*'
     - 'packages/*'
   ```
3. Create frontend app:
   ```bash
   pnpm create vite@latest apps/frontend --template react-ts
   ```
4. Install React Router DOM 6:
   ```bash
   cd apps/frontend
   pnpm add react-router-dom@^6.20.0
   ```
5. Set up monorepo structure:
   ```
   dapp-learning/              # Monorepo root
   ├── apps/
   │   └── frontend/          # React + Vite + TypeScript
   │       ├── src/
   │       │   ├── components/
   │       │   ├── hooks/
   │       │   ├── lib/
   │       │   ├── pages/
   │       │   ├── config/
   │       │   ├── stores/
   │       │   ├── types/
   │       │   ├── utils/
   │       │   ├── App.tsx
   │       │   └── main.tsx
   │       └── public/
   │           └── _redirects  # For static hosting
   ├── packages/              # (Created in Smart Contract Development)
   │   └── contracts/         # Hardhat 2 + Solidity
   ├── pnpm-workspace.yaml
   └── package.json
   ```
6. Configure root `package.json` with workspace scripts
7. Set up `.gitignore` for Node.js and environment files
8. Create `_redirects` file in `apps/frontend/public/` for static hosting

#### Exercise 1.3: Development Tools Configuration
1. Configure ESLint for React and TypeScript
2. Set up Prettier with consistent formatting rules
3. Configure React Router DOM 6 with BrowserRouter
4. Set up `_redirects` file for static hosting compatibility
5. Add Husky for pre-commit hooks (optional)
6. Create a `README.md` with project setup instructions

### Key Concepts
- **Client-side only architecture**: No backend server -- the blockchain is your backend, the wallet is your auth, static hosting is your deployment
- **React Router DOM 6**: Client-side routing with BrowserRouter (HTML5 history API)
- **Static Hosting**: Using `_redirects` file for IPFS/Vercel/Netlify compatibility (because the output is just HTML/JS/CSS)
- **Vite**: Fast build tool and development server
- **TypeScript**: Type safety in DApp development
- **Package Management**: Understanding dependencies vs devDependencies
- **Environment Variables**: `.env` for configuration

### Checkpoint
- Can explain why a DApp doesn't need a traditional backend server
- Can initialize a React + Vite project from scratch
- Understands React Router DOM 6 setup with BrowserRouter
- Has configured `_redirects` for static hosting
- Understands project structure and configuration files
- Can explain the purpose of each tool in the stack

---

## Web3 Fundamentals & Wallet Integration

**Difficulty:** Beginner

### Learning Objectives
- Understand blockchain basics (Ethereum, accounts, transactions)
- Learn about wallets (MetaMask, WalletConnect)
- Integrate RainbowKit for wallet connection UI
- Configure Wagmi for wallet interactions
- Get a first taste of Viem utility functions
- Handle wallet connection states

### Prerequisites
- Completed Environment Setup & Project Scaffolding
- Basic understanding of blockchain concepts

### Exercises

#### Exercise 2.1: Install Web3 Dependencies
1. Navigate to frontend app:
   ```bash
   cd apps/frontend
   ```

2. Install core packages with correct versions:
   ```bash
   pnpm add wagmi@^2.5.0 viem@^2.7.0 @rainbow-me/rainbowkit@^2.0.0
   pnpm add @tanstack/react-query@^5.85.5 zustand@^4.4.7
   ```

3. Install Node.js polyfills for browser:
   ```bash
   pnpm add buffer@^6.0.3 process@^0.11.10 util@^0.12.5
   ```

4. Install TanStack Query Devtools (dev only):
   ```bash
   pnpm add -D @tanstack/react-query-devtools@^5.85.5
   ```

5. Understand what each package does

> **Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete dependency installation.

#### Exercise 2.2: Configure Alchemy RPC Provider
1. Get Alchemy API key from [Alchemy](https://www.alchemy.com/)

   > **Reference**: See `dapp_setup_guides/bc_infrastructure_services.md` for a comprehensive comparison of blockchain infrastructure providers (Alchemy, Infura, QuickNode, etc.) including features, pricing, and use cases.

2. Create `apps/frontend/.env.local`:
   ```bash
   VITE_ALCHEMY_API_KEY=your_alchemy_key_here
   VITE_WALLETCONNECT_PROJECT_ID=your_project_id_here
   VITE_APP_CHAIN_ID=11155111  # Sepolia
   ```

3. Create `apps/frontend/src/config/providers.ts`:
   ```typescript
   const ALCHEMY_KEY = import.meta.env.VITE_ALCHEMY_API_KEY;
   if (!ALCHEMY_KEY) {
     throw new Error('VITE_ALCHEMY_API_KEY is required');
   }
   
   export const ALCHEMY_HTTP_SEPOLIA = `https://eth-sepolia.g.alchemy.com/v2/${ALCHEMY_KEY}`;
   export const ALCHEMY_WS_SEPOLIA = `wss://eth-sepolia.g.alchemy.com/v2/${ALCHEMY_KEY}`;
   ```

#### Exercise 2.3: Configure Wagmi with RainbowKit
1. Create `apps/frontend/src/config/chain.ts` and `apps/frontend/src/config/wagmi.ts`
2. Configure transport selection (WebSocket desktop, HTTP mobile)
3. Update `apps/frontend/src/main.tsx` to wrap app with providers (BrowserRouter, WagmiProvider, QueryClientProvider, RainbowKitProvider)

> **Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete Wagmi configuration.

#### Exercise 2.4: Create Wallet Connection Component
1. Create wallet connect component using RainbowKit's `ConnectButton`
2. Use Wagmi hooks for wallet state (`useAccount`, `useBalance`, `useChainId`)
3. Handle chain switching and connection states

#### Exercise 2.5: First Taste of Viem Utilities
1. Understand the big picture: Viem → Wagmi → RainbowKit stack
2. Create Viem helper functions (`formatAddress`, `formatWeiToEther`, etc.)
3. Use helpers in WalletInfo component

> **Note**: A deep dive into the full web3 library landscape (ethers.js vs Viem vs web3.js, when to use what) comes in Reading Blockchain Data, once you have real contracts and Hardhat experience to give it context.

### Key Concepts
- **Viem → Wagmi → RainbowKit**: Three layers from low-level to UI
- **Viem utilities**: Pure functions for formatting/validation that work anywhere
- **EVM Chains**: Understanding different networks (Mainnet, Testnets, Local)
- **RPC Providers**: Alchemy with WebSocket (desktop) / HTTP (mobile) transport selection
- **Wallet Providers**: How wallets interact with DApps via WalletConnect
- **React Context**: How RainbowKit/Wagmi provide global state
- **TanStack Query**: Data fetching and caching layer

### Checkpoint
- Can connect and disconnect wallets
- Understands wallet connection flow
- Can display wallet information (address, balance, network)
- Has created and used Viem utility functions

---

## Smart Contract Development & Deployment

**Difficulty:** Beginner-Intermediate

### Learning Objectives
- Understand Solidity fundamentals (types, functions, events, modifiers)
- Set up Hardhat 2 for contract compilation, testing, and deployment
- Write smart contracts with NatSpec documentation
- Write comprehensive contract tests
- Deploy to local Hardhat network and Sepolia testnet
- Share ABIs with the frontend
- Use OpenZeppelin contract libraries and upgradeable patterns
- Verify contracts on Sourcify and Blockscout

### Prerequisites
- Completed Web3 Fundamentals & Wallet Integration
- Basic programming knowledge

### Exercises

#### Exercise 3.1: Solidity Fundamentals
1. Learn Solidity value types, visibility modifiers, function types
2. Understand events, custom errors, mappings, structs, enums
3. Understand `storage` vs `memory` vs `calldata`

#### Exercise 3.2: Set Up Hardhat 2 in the Monorepo
1. Create `packages/contracts/` with Hardhat 2
2. Install OpenZeppelin contracts and plugins
3. Configure hardhat.config.cjs
4. Add root workspace scripts

#### Exercise 3.3: Write Your First Smart Contract
1. Write a SimpleStorage contract with NatSpec
2. Include events (ValueSet, StringSet)
3. Compile and understand the generated ABI

#### Exercise 3.4: Write Contract Tests
1. Write tests using Chai and Hardhat
2. Test deployment, state changes, events, edge cases
3. Understand `beforeEach`, `connect()`, `.to.emit()`

#### Exercise 3.5: Deploy to Local Hardhat Network
1. Create deployment script
2. Run local Hardhat node
3. Deploy and interact via Hardhat console

#### Exercise 3.6: Deploy to Sepolia Testnet
1. Get testnet ETH from a faucet
2. Configure environment for Sepolia
3. Deploy and verify on block explorer

#### Exercise 3.7: Share ABIs with the Frontend
1. Extract ABI from compilation artifacts
2. Create contract info file in `apps/frontend/src/contracts/`
3. Configure contract address and ABI for frontend import

#### Exercise 3.8: OpenZeppelin Contracts & Upgradeable Patterns
1. Write a standard ERC-20 token using OpenZeppelin
2. Write an upgradeable ERC-20 using OpenZeppelin Upgradeable
3. Deploy using `upgrades.deployProxy()`
4. Understand proxy pattern basics

#### Exercise 3.9: Contract Verification (Sourcify + Blockscout)
1. Verify on Sourcify (no API key required)
2. Verify on Blockscout (no API key required)
3. No Etherscan API keys needed

### Key Concepts
- **Solidity**: Types, functions, events, modifiers, inheritance
- **NatSpec**: Documentation standard for Solidity contracts
- **Hardhat 2**: Compilation, testing, deployment, local network
- **Testing**: Chai assertions, event testing, account management
- **ABI**: How the frontend communicates with contracts
- **OpenZeppelin**: Audited contract libraries (ERC-20, Ownable, etc.)
- **Upgradeable Contracts**: Proxy pattern, `initialize()` instead of constructor
- **Contract Verification**: Sourcify first, then Blockscout (no API keys)

### Checkpoint
- Can write Solidity contracts with NatSpec
- Can test contracts comprehensively
- Can deploy to local and testnet
- Understands OpenZeppelin inheritance
- Knows how to share ABIs with frontend
- Can verify contracts on Sourcify/Blockscout

---

## Reading Blockchain Data

**Difficulty:** Intermediate

### Learning Objectives
- Understand the full web3 library landscape (ethers.js, Viem, web3.js)
- Know why this project uses ethers.js in Hardhat and Viem/Wagmi on the frontend
- Understand how to read data from smart contracts using Wagmi hooks
- Handle loading and error states
- Work with different data types (strings, numbers, arrays, structs)
- Understand the difference between read and write operations

### Prerequisites
- Completed Smart Contract Development & Deployment (contract deployed, ABI shared with frontend, used ethers.js in tests)

### Exercises

#### Exercise 4.1: Understanding the Web3 Library Landscape
1. Learn the three major Ethereum JS libraries (web3.js, ethers.js, Viem) and when each is used
2. Understand why this project uses ethers.js in Hardhat and Viem/Wagmi on the frontend
3. Deep dive into Viem's three layers (utilities, clients, ABIs)
4. Deep dive into Wagmi hooks and what they add on top of Viem
5. Learn the "Am I in React?" decision guide for choosing the right library

#### Exercise 4.2: Set Up Scopes Pattern for Cache Management
1. Create centralized scopes for TanStack Query cache invalidation
2. Create helper functions for targeted invalidation

#### Exercise 4.3: Read Simple Contract State
1. Use `useReadContract` with contract info from Smart Contract Development
2. Handle loading and error states properly
3. Use scope keys for cache management

#### Exercise 4.4: Read Multiple Values
1. Use `useReadContracts` for batch reading
2. Use TanStack Query's `placeholderData` to prevent flicker on refetch

#### Exercise 4.5: Read ERC-20 Token Data
1. Read token name, symbol, decimals, total supply
2. Read user's token balance
3. Display formatted token amounts (considering decimals)

#### Exercise 4.6: Event Listening (Basic)
1. Use `useWatchContractEvent` for basic event watching
2. Invalidate relevant queries on events
3. Display event logs in real-time

### Key Concepts
- **Web3 library landscape**: ethers.js (Hardhat), Viem (frontend low-level), Wagmi (React hooks)
- **Why two libraries**: contract tooling needs vs frontend needs
- **Viem layers**: utilities, clients, standard ABIs
- **Wagmi additions**: caching, React state, re-renders, `enabled` guards, scope keys
- **ABI (Application Binary Interface)**: How to interact with contracts
- **Read Operations**: Free, no gas cost, synchronous
- **TanStack Query 5**: Query scopes pattern for targeted cache invalidation
- **Scope Keys**: Stable query keys organized by chainId and entity type
- **Wagmi 2 Hooks**: `useReadContract`, `useReadContracts`, `useWatchContractEvent`

### Checkpoint
- Can explain why the project uses two Ethereum libraries
- Knows when to use ethers.js, Viem, and Wagmi
- Can read any public contract variable
- Understands loading and error handling
- Can work with different data types
- Can listen to and display contract events

---

## Writing to Blockchain (Transactions)

**Difficulty:** Intermediate

### Learning Objectives
- Understand transaction lifecycle (prepare, sign, send, confirm)
- Use WAGMI hooks for writing to contracts
- Handle transaction states (pending, success, error)
- Implement transaction feedback (overlays, loading states)
- Understand gas estimation and transaction costs
- Handle user rejection of transactions

### Prerequisites
- Completed Reading Blockchain Data
- Understanding of gas and transaction fees

### Exercises

#### Exercise 5.1: Create Transaction Overlay System (REQUIRED)
1. Create modal store with Zustand
2. Create TransactionOverlay component (pending + confirming states)
3. Create SuccessOverlay component
4. Create ModalManager and add to App

#### Exercise 5.2: Write to Contract (Simple)
1. Use `useWriteContract` and `useWaitForTransactionReceipt`
2. Integrate with transaction overlay
3. Invalidate cache after success

#### Exercise 5.3: Gas Estimation
1. Estimate gas before sending transaction
2. Display estimated gas cost to user

#### Exercise 5.4: Complex Transaction with Validation
1. Create transaction with multiple parameters
2. Validate inputs before sending

### Key Concepts
- **Transaction Lifecycle**: From initiation (isPending) to confirmation (isConfirming) to success
- **Transaction Overlay System**: REQUIRED for production UX
- **Gas**: Understanding gas limits, gas prices, and total cost
- **Cache Invalidation**: Invalidate relevant queries after successful transactions
- **Wagmi 2 Hooks**: `useWriteContract`, `useWaitForTransactionReceipt`

### Checkpoint
- Can send transactions successfully
- Understands transaction states
- Can estimate and display gas costs
- Handles errors gracefully
- Provides good UX feedback

---

## Advanced Contract Interactions

**Difficulty:** Intermediate-Advanced

### Learning Objectives
- Implement approval patterns (ERC-20, ERC-721)
- Handle contract interactions that require multiple steps
- Set up a production real-time event system (separate WebSocket client)

### Prerequisites
- Completed Writing to Blockchain (Transactions)
- Understanding of ERC standards

### Exercises

#### Exercise 6.1: ERC-20 Token Transfer with Approval
1. Implement two-step transaction flow (approve then transfer)
2. Check and display allowance
3. Handle both transactions with overlay feedback

#### Exercise 6.2: Real-Time Event System (Production)
1. Create separate WebSocket client for event watching
2. Create global event hook with targeted cache invalidation
3. Mount at app root for real-time updates

### Key Concepts
- **Approval Patterns**: Two-step transactions (approve then transfer)
- **ERC Standards**: Understanding common token standards
- **Real-Time Events**: Separate WebSocket client prevents HTTP polling
- **Transaction Sequencing**: Ordering dependent transactions

### Checkpoint
- Can implement complex multi-step interactions
- Understands approval patterns
- Can work with ERC-20 and ERC-721
- Real-time event system working

---

## State Management & Data Fetching

**Difficulty:** Intermediate

### Learning Objectives
- Manage complex application state with Zustand 4
- Cache blockchain data effectively with TanStack Query 5
- Implement scopes pattern for targeted cache invalidation
- Handle data synchronization across components
- Prevent UI flicker with proper cache configuration

### Prerequisites
- Completed Advanced Contract Interactions
- Understanding of React state management

### Exercises

#### Exercise 7.1: Global State Management with Zustand 4
1. Create Zustand store for UI state
2. Use for UI state only (NOT for blockchain data)
3. Persist state to localStorage

#### Exercise 7.2: Advanced Scopes Pattern
1. Expand scopes with pagination, umbrella keys
2. Implement debounced invalidation to prevent flicker
3. Create helper functions for common invalidation patterns

#### Exercise 7.3: Prevent UI Flicker with placeholderData
1. Configure QueryClient with `placeholderData`
2. Use in components for smooth UI updates

### Key Concepts
- **Zustand 4**: Lightweight state management for UI state only
- **TanStack Query 5**: Scopes pattern for organized cache invalidation
- **State Separation**: Wagmi for blockchain state, TanStack Query for caching, Zustand for UI
- **Prevent Flicker**: Use `placeholderData` to keep previous data while refetching

### Checkpoint
- Can manage complex application state
- Understands caching strategies
- Creates reusable data fetching hooks

---

## User Experience & Best Practices

**Difficulty:** Intermediate-Advanced

### Learning Objectives
- Implement connection health monitoring
- Handle edge cases and errors gracefully
- Implement loading states and skeletons
- Create responsive designs

### Prerequisites
- Completed State Management & Data Fetching
- Basic understanding of UI/UX principles

### Exercises

#### Exercise 8.1: Connection Health Monitoring
1. Create connection health store with Zustand
2. Implement periodic health checks (`getBlockNumber()`)
3. Show connection indicator in UI
4. Implement exponential backoff for reconnection

#### Exercise 8.2: Enhanced Error Handling
1. Map contract errors to user-friendly messages
2. Create error display component
3. Handle user rejections differently from actual errors

#### Exercise 8.3: Loading States and Skeletons
1. Create skeleton components
2. Use in loading states for smooth UX

#### Exercise 8.4: Responsive Design
1. Make DApp mobile-friendly with Tailwind responsive classes
2. Test wallet connection and transaction overlays on mobile

### Key Concepts
- **Connection Health**: Monitor RPC health and show status to users
- **Error Communication**: Translating technical errors to user language
- **Progressive Enhancement**: Building for all devices
- **Mobile Optimization**: HTTP transport for mobile, WebSocket for desktop

### Checkpoint
- Creates intuitive and responsive interfaces
- Handles all error cases gracefully
- Provides excellent user feedback

---

## Advanced Topics & Production Readiness

**Difficulty:** Advanced

### Learning Objectives
- Implement build versioning
- Set up testing strategies
- Handle production builds and deployment
- Understand production optimization (manual chunks, code splitting)

### Prerequisites
- Completed User Experience & Best Practices
- Understanding of production deployment

### Exercises

#### Exercise 9.1: Build Versioning
1. Implement build counter in Vite config
2. Display build ID in dev mode

#### Exercise 9.2: Testing Strategy
1. Set up Vitest for frontend testing
2. Write frontend tests
3. Verify contract tests from Smart Contract Development

#### Exercise 9.3: Production Build and Deployment
1. Configure manual chunks for production build
2. Build and verify output (including `_redirects`)
3. Test production build locally

### Key Concepts
- **Build Versioning**: Build counter system for debugging and support
- **Testing**: Ensuring reliability (Vitest for frontend, Hardhat for contracts)
- **Deployment**: Static hosting with `_redirects` file (SPA routing)
- **Manual Chunks**: Vite build optimization (vendor, wagmi, rainbowkit chunks)

### Checkpoint
- Has comprehensive test coverage
- Can deploy to production
- Creates production-ready code

---

## Capstone Project Ideas

After completing all modules, choose one of these projects to demonstrate mastery:

1. **NFT Marketplace**: Full-featured marketplace with buying, selling, bidding
2. **DeFi Dashboard**: Portfolio tracker with multiple protocols
3. **DAO Voting Platform**: Governance interface with proposals and voting
4. **Token Launchpad**: Platform for launching new tokens
5. **Multi-sig Wallet Interface**: Interface for managing multi-signature wallets
6. **Cross-chain Bridge UI**: Interface for bridging assets between chains

---

## Resources & Further Learning

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

### Tools
- [Hardhat](https://hardhat.org/) - Development environment
- [Foundry](https://book.getfoundry.sh/) - Smart contract toolkit
- [Remix](https://remix.ethereum.org/) - Online IDE
- [Tenderly](https://tenderly.co/) - Debugging and monitoring

### Communities
- [Ethereum Stack Exchange](https://ethereum.stackexchange.com/)
- [r/ethdev](https://www.reddit.com/r/ethdev/)
- [Discord: Ethereum](https://discord.gg/ethereum)

---

## Assessment Rubric

Each module should be assessed on:
- **Understanding**: Can explain concepts clearly
- **Implementation**: Code works correctly
- **Best Practices**: Follows conventions and patterns
- **Problem Solving**: Can debug and solve issues independently
- **Code Quality**: Clean, readable, maintainable code

---
