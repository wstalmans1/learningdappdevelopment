# DApp Development Learning Path: Zero to Hero

A progressive series of exercises designed to take you from zero knowledge to mastering DApp development using **production-ready setup guides** and modern web3 tooling.

> **📖 Critical**: This learning path is based on the setup guides in `dapp_setup_guides/`. Always refer to those guides for the **single source of truth** on scaffolding, architecture, and best practices.
>
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Complete setup instructions
> - **[bc_infrastructure_services.md](./dapp_setup_guides/bc_infrastructure_services.md)** - Blockchain infrastructure services comparison (reference for understanding RPC provider options)

## Overview

This learning path is structured in **8 progressive modules**, each building upon the previous one. Each module contains:
- **Learning objectives**
- **Prerequisites**
- **Hands-on exercises**
- **Key concepts to master**
- **Checkpoint assessment**

---

## Module 1: Environment Setup & Project Scaffolding 🏗️

**Difficulty:** Beginner

### Learning Objectives
- Set up a modern development environment for DApp development
- Understand the toolchain: Node.js, npm/yarn/pnpm, Git
- Initialize a React project with Vite and TypeScript
- Set up React Router DOM 6 with BrowserRouter
- Configure ESLint, Prettier, and basic project structure
- Understand the difference between traditional web apps and DApps

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
   ├── packages/              # (Created in later modules)
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
- **React Router DOM 6**: Client-side routing with BrowserRouter (HTML5 history API)
- **Static Hosting**: Using `_redirects` file for IPFS/Vercel/Netlify compatibility
- **Vite**: Fast build tool and development server
- **TypeScript**: Type safety in DApp development
- **Package Management**: Understanding dependencies vs devDependencies
- **Environment Variables**: `.env` for configuration

### Checkpoint
✅ Can initialize a React + Vite project from scratch  
✅ Understands React Router DOM 6 setup with BrowserRouter  
✅ Has configured `_redirects` for static hosting  
✅ Understands project structure and configuration files  
✅ Can explain the purpose of each tool in the stack

---

## Module 2: Web3 Fundamentals & Wallet Integration 🔐

**Difficulty:** Beginner

### Learning Objectives
- Understand blockchain basics (Ethereum, accounts, transactions)
- Learn about wallets (MetaMask, WalletConnect)
- Integrate RainbowKit for wallet connection UI
- Configure WAGMI for wallet interactions
- Understand Viem and its relationship with Wagmi
- Know when to use Viem utilities vs Wagmi hooks
- Handle wallet connection states

### Prerequisites
- Completed Module 1
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

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete dependency installation.

#### Exercise 2.2: Configure Alchemy RPC Provider
1. Get Alchemy API key from [Alchemy](https://www.alchemy.com/)
   
   > **📖 Reference**: See `dapp_setup_guides/bc_infrastructure_services.md` for a comprehensive comparison of blockchain infrastructure providers (Alchemy, Infura, QuickNode, etc.) including features, pricing, and use cases. This document helps understand provider options but the setup guides specify Alchemy as the chosen provider.

2. Create `apps/frontend/.env.local`:
   ```bash
   VITE_ALCHEMY_API_KEY=your_alchemy_key_here
   VITE_WALLETCONNECT_PROJECT_ID=your_project_id_here
   VITE_APP_CHAIN_ID=11155111  # Sepolia
   ```

3. Create `apps/frontend/src/config/providers.ts`:
   ```typescript
   // Centralized provider configuration
   const ALCHEMY_KEY = import.meta.env.VITE_ALCHEMY_API_KEY;
   if (!ALCHEMY_KEY) {
     throw new Error('VITE_ALCHEMY_API_KEY is required');
   }
   
   export const ALCHEMY_HTTP_SEPOLIA = `https://eth-sepolia.g.alchemy.com/v2/${ALCHEMY_KEY}`;
   export const ALCHEMY_WS_SEPOLIA = `wss://eth-sepolia.g.alchemy.com/v2/${ALCHEMY_KEY}`;
   ```

#### Exercise 2.3: Configure Wagmi with RainbowKit
1. Create `apps/frontend/src/config/chain.ts`:
   ```typescript
   import { sepolia } from 'wagmi/chains';
   
   export const APP_CHAIN_ID = sepolia.id;
   export const APP_CHAIN = sepolia;
   ```

2. Create `apps/frontend/src/config/wagmi.ts`:
   ```typescript
   import { http, webSocket } from 'wagmi';
   import { sepolia } from 'wagmi/chains';
   import { getDefaultConfig } from '@rainbow-me/rainbowkit';
   import { ALCHEMY_HTTP_SEPOLIA, ALCHEMY_WS_SEPOLIA } from './providers';
   
   const projectId = import.meta.env.VITE_WALLETCONNECT_PROJECT_ID || 'fallback_id';
   
   // Mobile detection for transport selection
   const isMobile = typeof window !== 'undefined' && 
     /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
   
   export const config = getDefaultConfig({
     appName: 'DApp Learning',
     projectId,
     chains: [sepolia],
     transports: {
       [sepolia.id]: isMobile 
         ? http(ALCHEMY_HTTP_SEPOLIA)  // Mobile: HTTP
         : webSocket(ALCHEMY_WS_SEPOLIA),  // Desktop: WebSocket-only
     },
     ssr: false,  // Client-side only
   });
   ```

3. Update `apps/frontend/src/main.tsx` to wrap app with providers:
   ```typescript
   import { WagmiProvider } from 'wagmi';
   import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
   import { RainbowKitProvider } from '@rainbow-me/rainbowkit';
   import { BrowserRouter } from 'react-router-dom';
   import { config } from './config/wagmi';
   import '@rainbow-me/rainbowkit/styles.css';
   
   const queryClient = new QueryClient({
     defaultOptions: {
       queries: {
         placeholderData: (previousData) => previousData,  // Prevent flicker
         refetchOnWindowFocus: false,
         staleTime: 30_000,
       },
     },
   });
   
   // Wrap app with providers (BrowserRouter → WagmiProvider → QueryClientProvider → RainbowKitProvider)
   ```

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete Wagmi configuration.

#### Exercise 2.4: Create Wallet Connection Component
1. Create `apps/frontend/src/components/WalletConnect.tsx`:
   ```typescript
   import { ConnectButton } from '@rainbow-me/rainbowkit';
   
   export function WalletConnect() {
     return <ConnectButton />;
   }
   ```

2. Use Wagmi hooks for wallet state:
   ```typescript
   import { useAccount, useBalance, useChainId } from 'wagmi';
   
   export function WalletInfo() {
     const { address, isConnected } = useAccount();
     const chainId = useChainId();
     const { data: balance } = useBalance({ address });
     
     if (!isConnected) return <div>Not connected</div>;
     
     return (
       <div>
         <p>Address: {address}</p>
         <p>Chain ID: {chainId}</p>
         <p>Balance: {balance?.formatted} {balance?.symbol}</p>
       </div>
     );
   }
   ```

3. Handle chain switching and connection states

#### Exercise 2.5: Understanding Viem and the Wagmi-Viem Relationship
1. Learn about alternative web3 libraries (ethers.js, web3.js) and why Viem was chosen
2. Understand that Wagmi is built on top of Viem
3. Learn Viem utility functions (`formatUnits`, `parseUnits`, `isAddress`, etc.)
4. Create utility functions using Viem directly
5. Understand when to use Viem vs Wagmi hooks
6. Create a direct Viem client for non-React contexts

> **📖 Key Concept**: Wagmi provides React hooks for Ethereum, but it's built on Viem. Viem is the low-level TypeScript Ethereum library. Viem was chosen over alternatives (ethers.js, web3.js) for its TypeScript-first design, smaller bundle sizes, modern architecture, and excellent Wagmi integration. Use Wagmi hooks in React components, but use Viem utilities for formatting, validation, and non-React contexts.

### Key Concepts
- **Web3 Library Landscape**: Alternatives include ethers.js and web3.js, but Viem was chosen for TypeScript-first design, smaller bundles, and modern architecture
- **Viem**: Low-level TypeScript Ethereum library (foundation) - chosen for type safety, performance, and modern design
- **Wagmi**: React hooks wrapper around Viem (React convenience layer)
- **When to use each**: Wagmi hooks for React components, Viem utilities for pure functions, Viem clients for non-React contexts
- **EVM Chains**: Understanding different networks (Mainnet, Testnets, Local)
- **RPC Providers**: Alchemy with WebSocket (desktop) / HTTP (mobile) transport selection
  - **Reference**: See `dapp_setup_guides/bc_infrastructure_services.md` for provider comparison and information
- **Wallet Providers**: How wallets interact with DApps via WalletConnect
- **React Context**: How RainbowKit/Wagmi provide global state
- **Hooks Pattern**: Using React hooks for blockchain data
- **TanStack Query**: Data fetching and caching layer
- **Monorepo Structure**: Frontend in `apps/frontend/`, configs in `src/config/`

### Checkpoint
✅ Can connect and disconnect wallets  
✅ Understands wallet connection flow  
✅ Can display wallet information (address, balance, network)
✅ Understands the relationship between Wagmi and Viem
✅ Knows when to use Viem utilities vs Wagmi hooks

---

## Module 3: Reading Blockchain Data 📖

**Difficulty:** Beginner-Intermediate

### Learning Objectives
- Understand how to read data from smart contracts
- Use WAGMI hooks to query blockchain state
- Handle loading and error states
- Work with different data types (strings, numbers, arrays, structs)
- Understand the difference between read and write operations

### Prerequisites
- Completed Module 2
- Basic understanding of smart contracts

### Exercises

#### Exercise 3.1: Deploy a Simple Contract (or use existing)
1. Use a simple contract like:
   ```solidity
   contract SimpleStorage {
       uint256 public storedValue;
       string public storedString;
       
       function setValue(uint256 _value) public {
           storedValue = _value;
       }
       
       function setString(string memory _str) public {
           storedString = _str;
       }
   }
   ```
2. Deploy to Sepolia testnet (or use Hardhat local network)
3. Get the contract address and ABI

#### Exercise 3.2: Read Contract State with TanStack Query Scopes
1. Create `apps/frontend/src/lib/scopes.ts` for query key management:
   ```typescript
   export const scopes = {
     storedValue: (chainId: number, address: string) =>
       ['contract', 'storedValue', chainId, address.toLowerCase()] as const,
     storedString: (chainId: number, address: string) =>
       ['contract', 'storedString', chainId, address.toLowerCase()] as const,
   };
   ```

2. Use `useReadContract` (Wagmi 2) with scope keys:
   ```typescript
   import { useReadContract } from 'wagmi';
   import { useChainId } from 'wagmi';
   import { scopes } from '@/lib/scopes';
   
   export function ReadValue({ contractAddress, abi }) {
     const chainId = useChainId();
     
     const { data, isLoading, error } = useReadContract({
       address: contractAddress,
       abi,
       functionName: 'storedValue',
       query: {
         scopeKey: scopes.storedValue(chainId, contractAddress),
         staleTime: 30_000,
       },
     });
     
     if (isLoading) return <div>Loading...</div>;
     if (error) return <div>Error: {error.message}</div>;
     
     return <div>Value: {data?.toString()}</div>;
   }
   ```

3. Handle loading and error states properly

#### Exercise 3.3: Read Multiple Values
1. Use `useReadContracts` (Wagmi 2) for batch reading:
   ```typescript
   const { data, isLoading } = useReadContracts({
     contracts: [
       { address, abi, functionName: 'storedValue' },
       { address, abi, functionName: 'storedString' },
     ],
     query: {
       scopeKey: ['contract', 'batch', chainId, address],
     },
   });
   ```

2. Create a component that displays all contract state
3. Use TanStack Query's `placeholderData` to prevent flicker on refetch

#### Exercise 3.4: Read ERC-20 Token Data
1. Use a standard ERC-20 contract (e.g., USDC on Sepolia)
2. Read token name, symbol, decimals, total supply
3. Read user's token balance
4. Display formatted token amounts (considering decimals)

#### Exercise 3.5: Event Listening (Basic)
1. Use `useWatchContractEvent` (Wagmi 2) for basic event watching:
   ```typescript
   import { useWatchContractEvent } from 'wagmi';
   import { useQueryClient } from '@tanstack/react-query';
   
   export function EventListener({ contractAddress, abi }) {
     const qc = useQueryClient();
     const chainId = useChainId();
     
     useWatchContractEvent({
       address: contractAddress,
       abi,
       eventName: 'ValueSet',
       onLogs: (logs) => {
         // Invalidate relevant queries
         qc.invalidateQueries({ 
           queryKey: scopes.storedValue(chainId, contractAddress),
           exact: true 
         });
       },
     });
     
     return <div>Listening for events...</div>;
   }
   ```

2. Display event logs in real-time
3. Filter events by specific parameters
4. Create an event log viewer component

> **Note**: For production apps, use a separate WebSocket client for event watching (covered in Module 5).

### Key Concepts
- **ABI (Application Binary Interface)**: How to interact with contracts
- **Read Operations**: Free, no gas cost, synchronous
- **TanStack Query 5**: Query scopes pattern for targeted cache invalidation
- **Scope Keys**: Stable query keys organized by chainId and entity type
- **Data Fetching**: Prevent flicker with `placeholderData` and proper `staleTime`
- **Type Safety**: Using TypeScript with contract ABIs
- **Wagmi 2 Hooks**: `useReadContract`, `useReadContracts`, `useWatchContractEvent`

### Checkpoint
✅ Can read any public contract variable  
✅ Understands loading and error handling  
✅ Can work with different data types  
✅ Can listen to and display contract events

---

## Module 4: Writing to Blockchain (Transactions) ✍️

**Difficulty:** Intermediate

### Learning Objectives
- Understand transaction lifecycle (prepare → sign → send → confirm)
- Use WAGMI hooks for writing to contracts
- Handle transaction states (pending, success, error)
- Implement transaction feedback (toasts, loading states)
- Understand gas estimation and transaction costs
- Handle user rejection of transactions

### Prerequisites
- Completed Module 3
- Understanding of gas and transaction fees

### Exercises

#### Exercise 4.1: Write to Contract (Simple)
1. Use `useWriteContract` (Wagmi 2) for writing:
   ```typescript
   import { useWriteContract, useWaitForTransactionReceipt } from 'wagmi';
   import { useQueryClient } from '@tanstack/react-query';
   
   export function SetValue({ contractAddress, abi }) {
     const qc = useQueryClient();
     const chainId = useChainId();
     
     const { writeContract, data: hash, isPending, error } = useWriteContract();
     const { isLoading: isConfirming, isSuccess } = useWaitForTransactionReceipt({ 
       hash 
     });
     
     const handleSetValue = () => {
       writeContract({
         address: contractAddress,
         abi,
         functionName: 'setValue',
         args: [123],
       });
     };
     
     // Invalidate queries after success
     if (isSuccess) {
       qc.invalidateQueries({ 
         queryKey: scopes.storedValue(chainId, contractAddress),
         exact: true 
       });
     }
     
     return (
       <button 
         onClick={handleSetValue}
         disabled={isPending || isConfirming}
       >
         {isPending ? 'Confirming...' : 'Set Value'}
       </button>
     );
   }
   ```

2. Create a form to input new value
3. Handle transaction states (`isPending`, `isConfirming`)

#### Exercise 4.2: Transaction Overlay System (REQUIRED)
1. Create transaction overlay component (see setup guides for full implementation):
   ```typescript
   // apps/frontend/src/components/ui/TransactionOverlay.tsx
   export function TransactionOverlay({ 
     isVisible, 
     type,  // 'pending' | 'confirming'
     title,
     message 
   }) {
     if (!isVisible) return null;
     
     return (
       <div className="fixed inset-0 bg-black bg-opacity-50 backdrop-blur-sm z-50">
         <div className="flex items-center justify-center h-full">
           <div className="bg-white rounded-xl p-8">
             <h3>{title || (type === 'pending' ? 'Waiting for Signature' : 'Processing')}</h3>
             <p>{message}</p>
           </div>
         </div>
       </div>
     );
   }
   ```

2. Integrate overlay with transaction hooks:
   ```typescript
   {(isPending || isConfirming) && (
     <TransactionOverlay
       isVisible={true}
       type={isPending ? 'pending' : 'confirming'}
     />
   )}
   ```

3. Display transaction hash with link to block explorer
4. Show success state after confirmation

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete overlay system implementation.

#### Exercise 4.3: Gas Estimation
1. Estimate gas before sending transaction
2. Display estimated gas cost to user
3. Allow user to set custom gas limit
4. Handle gas estimation errors

#### Exercise 4.4: Complex Transaction (Multiple Parameters)
1. Create transaction with multiple parameters
2. Validate inputs before sending
3. Handle different data types (address, uint256, string, arrays)
4. Create a comprehensive form with validation

#### Exercise 4.5: Batch Transactions
1. Understand when to use multicall
2. Implement multiple writes in sequence
3. Handle partial failures
4. Create a transaction queue component

#### Exercise 4.6: Transaction History
1. Fetch user's transaction history
2. Display list of past transactions
3. Filter by contract, status, date
4. Link to block explorer for each transaction

### Key Concepts
- **Transaction Lifecycle**: From initiation (`isPending`) → confirmation (`isConfirming`) → success
- **Transaction Overlay System**: REQUIRED for production UX (full-screen feedback)
- **Gas**: Understanding gas limits, gas prices, and total cost
- **User Experience**: Clear feedback during async operations (overlays, not just toasts)
- **Error Handling**: Network errors, user rejection, contract errors
- **Cache Invalidation**: Invalidate relevant queries after successful transactions
- **Wagmi 2 Hooks**: `useWriteContract`, `useWaitForTransactionReceipt`

### Checkpoint
✅ Can send transactions successfully  
✅ Understands transaction states  
✅ Can estimate and display gas costs  
✅ Handles errors gracefully  
✅ Provides good UX feedback

---

## Module 5: Advanced Contract Interactions 🚀

**Difficulty:** Intermediate-Advanced

### Learning Objectives
- Work with complex contract interactions
- Implement approval patterns (ERC-20, ERC-721)
- Handle contract interactions that require multiple steps
- Work with contract factories and deployments
- Understand proxy patterns and upgradeable contracts (OpenZeppelin 5)
- Set up Hardhat 2 for smart contract development
- Deploy and verify contracts (Sourcify first, then Blockscout)

### Prerequisites
- Completed Module 4
- Understanding of ERC standards
- Basic Solidity knowledge

### Exercises

#### Exercise 5.0: Set Up Hardhat 2 (Smart Contract Development)
1. Create contracts package in monorepo:
   ```bash
   cd packages
   mkdir contracts && cd contracts
   pnpm init
   ```

2. Install Hardhat 2 and dependencies:
   ```bash
   pnpm add -D hardhat@^2.26.1 @nomicfoundation/hardhat-toolbox@^6.1.0
   pnpm add -D @openzeppelin/contracts@^5.4.0 @openzeppelin/contracts-upgradeable@^5.4.0
   pnpm add -D @openzeppelin/hardhat-upgrades@^3.5.0
   ```

3. Initialize Hardhat:
   ```bash
   pnpm exec hardhat init
   # Choose "Create a JavaScript project" (CommonJS)
   ```

4. Configure `hardhat.config.cjs`:
   ```javascript
   require("@nomicfoundation/hardhat-toolbox");
   require("@openzeppelin/hardhat-upgrades");
   
   module.exports = {
     solidity: {
       version: "0.8.22",
       settings: {
         optimizer: { enabled: true, runs: 200 },
       },
     },
     networks: {
       hardhat: { chainId: 1337 },
       sepolia: {
         url: process.env.SEPOLIA_URL,
         accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
       },
     },
   };
   ```

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete Hardhat setup.

#### Exercise 5.1: Deploy Upgradeable Contract
1. Create upgradeable contract using OpenZeppelin:
   ```solidity
   // packages/contracts/contracts/MyToken.sol
   import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
   import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
   import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
   
   contract MyToken is Initializable, ERC20Upgradeable, OwnableUpgradeable {
       function initialize() public initializer {
           __ERC20_init("MyToken", "MTK");
           __Ownable_init(msg.sender);
           _mint(msg.sender, 1000000 * 10**decimals());
       }
   }
   ```

2. Deploy using OpenZeppelin upgrades plugin:
   ```javascript
   // packages/contracts/scripts/deploy.js
   const { ethers, upgrades } = require("hardhat");
   
   async function main() {
     const MyToken = await ethers.getContractFactory("MyToken");
     const myToken = await upgrades.deployProxy(MyToken, [], { 
       initializer: "initialize" 
     });
     await myToken.waitForDeployment();
     console.log("MyToken deployed to:", await myToken.getAddress());
   }
   ```

3. Verify on Sourcify first, then Blockscout (no API keys required)

#### Exercise 5.2: ERC-20 Token Transfer
1. Implement token transfer functionality
2. Handle token approval before transfer
3. Check and display allowance
4. Create a token transfer component with approval flow

#### Exercise 5.2: ERC-721 NFT Interactions
1. Read NFT metadata (name, symbol, tokenURI)
2. Display NFT collection
3. Implement NFT transfer
4. Handle approval for NFT transfers
5. Create an NFT gallery component

#### Exercise 5.3: Multi-Step Contract Interactions
1. Create a flow requiring multiple transactions:
   - Approve token spending
   - Deposit tokens
   - Perform action
   - Withdraw tokens
2. Track progress through multi-step flow
3. Handle failures at any step
4. Allow user to retry failed steps

#### Exercise 5.4: Contract Factory Pattern
1. Deploy new contracts from a factory
2. Track deployed contract addresses
3. Interact with newly deployed contracts
4. Create a contract deployment interface

#### Exercise 5.5: Real-Time Event System (Production)
1. Create separate WebSocket client for event watching:
   ```typescript
   // apps/frontend/src/realtime/wsClient.ts
   import { createPublicClient, webSocket } from 'viem';
   import { APP_CHAIN } from '@/config/chain';
   import { ALCHEMY_WS_SEPOLIA } from '@/config/providers';
   
   export const wsClient = createPublicClient({
     chain: APP_CHAIN,
     transport: webSocket(ALCHEMY_WS_SEPOLIA),
   });
   ```

2. Create global event hook:
   ```typescript
   // apps/frontend/src/realtime/useGlobalEvents.ts
   import { wsClient } from './wsClient';
   import { useQueryClient } from '@tanstack/react-query';
   
   export function useGlobalEvents(watchAddresses: `0x${string}`[]) {
     const qc = useQueryClient();
     
     useEffect(() => {
       const unsubscribe = wsClient.watchContractEvent({
         address: watchAddresses,
         abi: CONTRACT_ABI,
         eventName: 'ValueSet',
         poll: false,  // WebSocket-only, no HTTP polling
         onLogs: (logs) => {
           // Invalidate relevant queries
           qc.invalidateQueries({ queryKey: ['contract', 'events'] });
         },
       });
       
       return () => unsubscribe();
     }, [watchAddresses, qc]);
   }
   ```

3. Mount at app root for real-time updates

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete event system.

### Key Concepts
- **Approval Patterns**: Two-step transactions (approve → transfer)
- **ERC Standards**: Understanding common token standards
- **Upgradeable Contracts**: OpenZeppelin 5 patterns (UUPS, TransparentProxy)
- **Hardhat 2**: Modern development environment
- **Contract Verification**: Sourcify first, then Blockscout (no API keys)
- **Real-Time Events**: Separate WebSocket client prevents HTTP polling
- **Transaction Sequencing**: Ordering dependent transactions
- **Error Recovery**: Handling partial failures

### Checkpoint
✅ Can implement complex multi-step interactions  
✅ Understands approval patterns  
✅ Can work with ERC-20 and ERC-721  
✅ Handles conditional logic in transactions

---

## Module 6: State Management & Data Fetching 📊

**Difficulty:** Intermediate

### Learning Objectives
- Manage complex application state with Zustand 4
- Cache blockchain data effectively with TanStack Query 5
- Implement scopes pattern for targeted cache invalidation
- Handle data synchronization across components
- Understand React Query patterns in web3
- Prevent UI flicker with proper cache configuration

### Prerequisites
- Completed Module 5
- Understanding of React state management

### Exercises

#### Exercise 6.1: Global State Management with Zustand 4
1. Create Zustand store for UI state:
   ```typescript
   // apps/frontend/src/stores/uiStore.ts
   import { create } from 'zustand';
   import { persist } from 'zustand/middleware';
   
   interface UIState {
     theme: 'light' | 'dark';
     setTheme: (theme: 'light' | 'dark') => void;
   }
   
   export const useUIStore = create<UIState>()(
     persist(
       (set) => ({
         theme: 'light',
         setTheme: (theme) => set({ theme }),
       }),
       { name: 'ui-storage' }
     )
   );
   ```

2. Use for UI state only (NOT for blockchain data)
3. Persist state to localStorage
4. Sync state across components

> **Important**: Use Zustand for UI state only. Blockchain data comes from Wagmi + TanStack Query.

#### Exercise 6.2: Scopes Pattern for Cache Invalidation
1. Centralize scope keys in `apps/frontend/src/lib/scopes.ts`:
   ```typescript
   export const scopes = {
     orgData: (chainId: number, address: string) =>
       ['orgs', 'data', chainId, address.toLowerCase()] as const,
     orgCount: (chainId: number) =>
       ['orgs', 'count', chainId] as const,
     orgPage: (chainId: number, page: number, size: number) =>
       ['orgs', 'page', chainId, page, size] as const,
   };
   ```

2. Use scope keys in queries:
   ```typescript
   const { data } = useReadContract({
     query: {
       scopeKey: scopes.orgData(chainId, address),
       staleTime: 30_000,
     },
   });
   ```

3. Invalidate targeted scopes after mutations:
   ```typescript
   const qc = useQueryClient();
   qc.invalidateQueries({ 
     queryKey: scopes.orgData(chainId, address),
     exact: true 
   });
   ```

4. Prevent flicker with `placeholderData`:
   ```typescript
   const queryClient = new QueryClient({
     defaultOptions: {
       queries: {
         placeholderData: (previousData) => previousData,
         refetchOnWindowFocus: false,
         staleTime: 30_000,
       },
     },
   });
   ```

#### Exercise 6.3: Optimistic Updates
1. Update UI immediately before transaction confirms
2. Revert on transaction failure
3. Implement optimistic updates for balance changes
4. Handle race conditions

#### Exercise 6.4: Real-time Data Sync
1. Use WebSocket or polling for real-time updates
2. Sync data across multiple browser tabs
3. Implement data refresh strategies
4. Handle network reconnection

#### Exercise 6.5: Data Aggregation
1. Fetch data from multiple contracts
2. Aggregate and transform data
3. Create custom hooks for complex data fetching
4. Implement data normalization

### Key Concepts
- **Zustand 4**: Lightweight state management for UI state only
- **TanStack Query 5**: Scopes pattern for organized cache invalidation
- **Scope Keys**: Stable, hierarchical query keys (chainId, entity type, id)
- **Targeted Invalidation**: Invalidate specific scopes, not entire cache
- **Prevent Flicker**: Use `placeholderData` to keep previous data while refetching
- **State Separation**: Wagmi for blockchain state, TanStack Query for caching, Zustand for UI
- **Custom Hooks**: Reusable data fetching logic with proper scopes

### Checkpoint
✅ Can manage complex application state  
✅ Understands caching strategies  
✅ Implements optimistic updates  
✅ Creates reusable data fetching hooks

---

## Module 7: User Experience & Best Practices 🎨

**Difficulty:** Intermediate-Advanced

### Learning Objectives
- Design intuitive DApp interfaces
- Implement transaction overlay system (REQUIRED for production)
- Implement connection health monitoring
- Handle edge cases and errors gracefully
- Implement loading states and skeletons
- Create responsive designs
- Follow web3 UX best practices
- Implement accessibility features

### Prerequisites
- Completed Module 6
- Basic understanding of UI/UX principles

### Exercises

#### Exercise 7.1: Transaction Overlay System (REQUIRED)
1. Create modal store with Zustand:
   ```typescript
   // apps/frontend/src/stores/modalStore.ts
   import { create } from 'zustand';
   
   type ModalType = 'transaction' | 'success' | null;
   
   export const useModalStore = create<{
     current: ModalType;
     payload?: Record<string, unknown>;
     open: (type: ModalType, payload?: Record<string, unknown>) => void;
     close: () => void;
   }>((set) => ({
     current: null,
     payload: undefined,
     open: (type, payload) => set({ current: type, payload }),
     close: () => set({ current: null, payload: undefined }),
   }));
   ```

2. Create transaction overlay component (see Module 4)
3. Create success overlay component
4. Integrate with Wagmi hooks (`isPending`, `isConfirming`)

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete overlay system.

#### Exercise 7.2: Connection Health Monitoring
1. Create connection health store:
   ```typescript
   // apps/frontend/src/stores/connectionHealthStore.ts
   import { create } from 'zustand';
   
   type ConnectionQuality = 'excellent' | 'good' | 'poor' | 'failed';
   
   export const useConnectionHealthStore = create<{
     quality: ConnectionQuality;
     consecutiveFailures: number;
     updateHealth: (quality: ConnectionQuality) => void;
   }>((set) => ({
     quality: 'excellent',
     consecutiveFailures: 0,
     updateHealth: (quality) => set({ quality }),
   }));
   ```

2. Implement health checks (periodic `getBlockNumber()`)
3. Track response time and consecutive failures
4. Show connection indicator in UI
5. Implement exponential backoff for reconnection

#### Exercise 7.3: Error Handling & User Feedback
1. Create comprehensive error handling system
2. Map contract errors to user-friendly messages
3. Implement toast notifications (in addition to overlays)
4. Create error boundary components
5. Handle network errors gracefully

#### Exercise 7.2: Loading States & Skeletons
1. Design loading skeletons for async operations
2. Implement progressive loading
3. Show transaction progress indicators
4. Create smooth transitions between states

#### Exercise 7.3: Responsive Design
1. Make DApp mobile-friendly
2. Test wallet connection on mobile devices
3. Optimize for different screen sizes
4. Handle mobile wallet apps (WalletConnect)

#### Exercise 7.4: Transaction Feedback
1. Create transaction status modals
2. Show transaction progress (pending → confirming → confirmed)
3. Display transaction details and links
4. Implement transaction history with filters

#### Exercise 7.5: Form Validation & UX
1. Validate inputs before transactions
2. Show helpful error messages
3. Prevent invalid transactions
4. Implement input formatting (addresses, amounts)

#### Exercise 7.6: Accessibility
1. Add ARIA labels to interactive elements
2. Ensure keyboard navigation works
3. Test with screen readers
4. Maintain color contrast ratios
5. Add focus indicators

#### Exercise 7.7: Performance Optimization
1. Implement code splitting
2. Lazy load components
3. Optimize image loading
4. Reduce bundle size
5. Implement virtual scrolling for long lists

### Key Concepts
- **Transaction Overlay System**: REQUIRED for production UX (full-screen feedback)
- **Connection Health**: Monitor RPC health and show status to users
- **Web3 UX Patterns**: Common patterns in successful DApps
- **Error Communication**: Translating technical errors to user language
- **Progressive Enhancement**: Building for all devices
- **Accessibility**: Making DApps usable for everyone
- **Mobile Optimization**: HTTP transport for mobile, WebSocket for desktop

### Checkpoint
✅ Creates intuitive and responsive interfaces  
✅ Handles all error cases gracefully  
✅ Provides excellent user feedback  
✅ Follows accessibility best practices  
✅ Optimizes for performance

---

## Module 8: Advanced Topics & Production Readiness 🏆

**Difficulty:** Advanced

### Learning Objectives
- Understand security best practices
- Implement testing strategies
- Set up CI/CD pipelines
- Handle production deployments
- Monitor and debug production issues
- Understand gas optimization
- Implement analytics and monitoring

### Prerequisites
- Completed Module 7
- Understanding of production deployment

### Exercises

#### Exercise 8.1: Contract Verification (Sourcify + Blockscout)
1. Verify contracts on Sourcify first (no API key required):
   ```bash
   # Use Sourcify API or Hardhat plugin
   pnpm exec hardhat verify --network sepolia <CONTRACT_ADDRESS>
   ```

2. Verify on Blockscout (no API key required):
   ```javascript
   // packages/contracts/hardhat.config.cjs
   networks: {
     sepolia: {
       url: process.env.SEPOLIA_URL,
       accounts: [process.env.PRIVATE_KEY],
       verify: {
         apiUrl: 'https://eth-sepolia.blockscout.com/api',
         apiKey: 'NO_API_KEY_NEEDED',
       },
     },
   }
   ```

3. Never use Etherscan API keys (use Sourcify/Blockscout only)

> **📖 Reference**: See `dapp_setup_guides/001_scaffolding_specs.md` section 7 for verification details.

#### Exercise 8.2: Build Versioning
1. Configure build counter in `apps/frontend/vite.config.ts`:
   ```typescript
   function getBuildNumber(): number {
     const counterFile = join(process.cwd(), '.build-counter');
     // Read/increment build counter
   }
   
   export default defineConfig({
     define: {
       __BUILD_ID__: JSON.stringify(`build-${getBuildNumber()}`),
     },
   });
   ```

2. Display build ID in dev tools or footer
3. Use for debugging and support

#### Exercise 8.3: Security Best Practices
1. Implement input sanitization
2. Validate contract addresses
3. Prevent common vulnerabilities (reentrancy, overflow)
4. Use secure random number generation
5. Implement rate limiting
6. Add security headers

#### Exercise 8.4: Testing Strategy
1. Write unit tests for hooks and utilities
2. Write integration tests for contract interactions
3. Set up E2E tests with Playwright/Cypress
4. Test wallet connection flows
5. Mock blockchain responses
6. Achieve good test coverage

#### Exercise 8.5: Environment Configuration
1. Set up different environments (dev, staging, prod)
2. Manage environment variables securely
3. Configure different chains per environment
4. Use feature flags
5. Implement environment-specific configurations

#### Exercise 8.6: CI/CD Pipeline
1. Set up GitHub Actions
2. Run tests on every PR
3. Build and deploy automatically
4. Run linting and type checking
5. Generate build artifacts
6. Deploy to Vercel/Netlify

#### Exercise 8.7: Monitoring & Analytics
1. Set up error tracking (Sentry)
2. Implement analytics (Mixpanel, Google Analytics)
3. Monitor transaction success rates
4. Track user interactions
5. Set up alerts for critical errors

#### Exercise 8.8: Gas Optimization
1. Understand gas costs of different operations
2. Batch transactions when possible
3. Use events instead of storage when appropriate
4. Optimize contract interactions
5. Implement gas price estimation

#### Exercise 8.9: Documentation
1. Write comprehensive README
2. Document API/hooks usage
3. Create user guides
4. Write developer documentation
5. Add code comments and JSDoc

#### Exercise 8.10: Final Project
Create a complete DApp that demonstrates:
- Wallet connection
- Reading and writing to contracts
- Complex state management
- Excellent UX
- Error handling
- Responsive design
- Production-ready code

### Key Concepts
- **Contract Verification**: Sourcify first, then Blockscout (no Etherscan API keys)
- **Build Versioning**: Build counter system for debugging and support
- **Security**: Protecting users and their funds
- **Testing**: Ensuring reliability (Vitest for frontend, Hardhat for contracts)
- **Deployment**: Static hosting with `_redirects` file (SPA routing)
- **Monitoring**: Understanding user behavior and errors
- **Optimization**: Efficient and cost-effective operations
- **Manual Chunks**: Vite build optimization (vendor, wagmi, rainbowkit chunks)

### Checkpoint
✅ Understands security best practices  
✅ Has comprehensive test coverage  
✅ Can deploy to production  
✅ Monitors and debugs issues  
✅ Creates production-ready code

---

## Capstone Project Ideas 🎯

After completing all modules, choose one of these projects to demonstrate mastery:

1. **NFT Marketplace**: Full-featured marketplace with buying, selling, bidding
2. **DeFi Dashboard**: Portfolio tracker with multiple protocols
3. **DAO Voting Platform**: Governance interface with proposals and voting
4. **Token Launchpad**: Platform for launching new tokens
5. **Multi-sig Wallet Interface**: Interface for managing multi-signature wallets
6. **Cross-chain Bridge UI**: Interface for bridging assets between chains

---

## Resources & Further Learning 📚

### Documentation
- [RainbowKit Docs](https://rainbowkit.com/)
- [WAGMI Docs](https://wagmi.sh/)
- [Viem Docs](https://viem.sh/)
- [React Docs](https://react.dev/)
- [React Router Docs](https://reactrouter.com/)
- [Vite Docs](https://vitejs.dev/)
- [Ethereum.org](https://ethereum.org/en/developers/)

### Tools
- [Hardhat](https://hardhat.org/) - Development environment
- [Foundry](https://book.getfoundry.sh/) - Smart contract toolkit
- [Remix](https://remix.ethereum.org/) - Online IDE
- [Etherscan](https://etherscan.io/) - Block explorer
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


---
