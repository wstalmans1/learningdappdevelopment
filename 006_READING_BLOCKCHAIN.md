# Module 4: Reading Blockchain Data

> **📖 Important**: This module follows the setup guides in `dapp_setup_guides/`. For the complete, authoritative setup instructions, refer to:
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Step-by-step setup guide
>
> The setup guides are the **single source of truth** for scaffolding and architecture decisions.

## Exercise 4.1: Understanding the Web3 Library Landscape

### Objective
Before you start reading blockchain data from the frontend, take a step back and understand the full picture of the JavaScript/TypeScript libraries you're using, why the project uses *different* libraries in different contexts, and how they relate to each other. This is the foundation you need to debug problems and read documentation effectively.

### Prerequisites
- [x] Completed Module 3 (you've already used ethers.js in Hardhat tests and scripts)
- [x] Completed Module 2 (you've already used Wagmi hooks and Viem utilities)

### Instructions

#### Step 1: The Three Major Ethereum JS Libraries

There are three libraries that dominate the Ethereum JavaScript ecosystem. You need to know they exist, what they're good at, and which one you'll use where.

| Library | Philosophy | Bundle size | TypeScript | Primary use case |
|---------|-----------|-------------|-----------|-----------------|
| **web3.js** | The original. Broad feature set, verbose API. | ~590 kB min | Retrofitted (v4 improved) | Legacy projects, some tooling |
| **ethers.js** (v6) | Clean, well-documented. De-facto standard for Node-side tooling. | ~120 kB min | Good (v6 much better) | Hardhat, scripts, server-side, any non-React context |
| **Viem** (v2) | TypeScript-first, modular, tree-shakeable. Newest of the three. | ~35 kB min (tree-shaken) | Excellent (designed for it) | Modern frontends, production DApps |

> All three can do the same things -- read contracts, send transactions, format values, interact with wallets. They differ in API design, bundle size, type safety, and ecosystem integrations.

#### Step 2: Why This Project Uses Two Libraries

You already noticed this in Module 3:

| Layer | Library | Reason |
|-------|---------|--------|
| **Hardhat** (contracts, tests, deploy scripts) | **ethers.js** | Hardhat's plugin ecosystem is built around ethers. `hardhat-ethers`, `hardhat-upgrades`, and most Hardhat tutorials assume ethers. Switching would mean losing that tooling. |
| **React frontend** | **Viem + Wagmi** | Viem is TypeScript-first and tree-shakeable, giving much smaller production bundles. Wagmi wraps Viem in React hooks with built-in caching (via TanStack Query), automatic re-renders, and wallet management. |

This is a **common pattern** in production DApps -- it's not a mistake or a compromise. The contract toolchain and the frontend have different needs:

- **Contract toolchain** needs: rich plugin ecosystem, mature testing utilities, extensive documentation for Solidity patterns.
- **Frontend** needs: small bundle, excellent TypeScript inference, React integration, automatic cache management.

#### Step 3: The Full Stack Picture

```
┌──────────────────────────────────────────────────────┐
│                     Your DApp                         │
│                                                       │
│  ┌─────────────────────┐  ┌────────────────────────┐ │
│  │  packages/contracts  │  │    apps/frontend        │ │
│  │                      │  │                         │ │
│  │  Hardhat + ethers.js │  │  React                  │ │
│  │  • Compile Solidity  │  │    ↕                    │ │
│  │  • Run tests         │  │  Wagmi hooks            │ │
│  │  • Deploy scripts    │  │  (useReadContract, etc.)│ │
│  │  • Console interact  │  │    ↕                    │ │
│  │                      │  │  Viem                   │ │
│  │  Output: ABI + addr ─┼──┼→ (low-level calls,     │ │
│  │                      │  │   formatting, encoding) │ │
│  └─────────────────────┘  └────────────────────────┘ │
│                                     ↕                 │
│                              RPC Provider             │
│                         (Alchemy / Infura)            │
│                                     ↕                 │
│                              Blockchain               │
└──────────────────────────────────────────────────────┘
```

The **bridge** between the two sides is the ABI and contract address you exported in Module 3, Exercise 3.7. The frontend never touches ethers.js; the contracts side never touches Viem.

#### Step 4: Viem -- Deep Dive

Viem has three layers you'll use in this project:

1. **Utility functions** (pure, no side effects)
   - `formatEther`, `parseEther` -- convert between wei and ether strings
   - `formatUnits`, `parseUnits` -- convert with arbitrary decimals (for tokens)
   - `isAddress` -- validate Ethereum addresses
   - `encodeFunctionData`, `decodeFunctionResult` -- ABI encoding (advanced)
   - `keccak256`, `toHex`, `fromHex` -- low-level encoding

2. **Clients** (stateful, connect to the blockchain)
   - `createPublicClient` -- read-only client for querying chain data
   - `createWalletClient` -- client that can sign and send transactions
   - Each client needs a **transport** (`http(url)` or `webSocket(url)`) and a **chain** (`sepolia`, `mainnet`, etc.)

3. **Standard ABIs** (pre-built type-safe ABIs)
   - `erc20Abi` -- complete ERC-20 interface
   - `erc721Abi` -- complete ERC-721 interface
   - These give you full type inference: the compiler knows exactly what functions exist and what types they return

> In React components, you typically **don't create Viem clients directly**. Instead, Wagmi creates them for you based on your config (from Module 2, Exercise 2.3). You use Wagmi hooks, and they use the Viem client under the hood. You'll only create Viem clients directly for non-React contexts (utility scripts, background event watchers).

#### Step 5: Wagmi -- Deep Dive

Wagmi provides React hooks that wrap Viem's functionality with:

| Hook | Purpose | Viem equivalent under the hood |
|------|---------|-------------------------------|
| `useReadContract` | Read a single contract value | `client.readContract(...)` |
| `useReadContracts` | Batch-read multiple values in one RPC call | `client.multicall(...)` |
| `useWriteContract` | Send a transaction that modifies state | `walletClient.writeContract(...)` |
| `useWaitForTransactionReceipt` | Wait for tx confirmation | `client.waitForTransactionReceipt(...)` |
| `useWatchContractEvent` | Subscribe to contract events | `client.watchContractEvent(...)` |
| `useAccount` | Current wallet address & connection state | (wallet connector) |
| `useBalance` | Native ETH balance | `client.getBalance(...)` |
| `useChainId` | Connected chain | (config state) |
| `useEstimateGas` | Estimate gas for a transaction | `client.estimateGas(...)` |

**What Wagmi adds on top of Viem:**
- **Automatic caching** via TanStack Query -- blockchain reads are cached and deduplicated
- **React state management** -- `isLoading`, `isError`, `isSuccess` states
- **Automatic re-renders** -- when data changes, your component re-renders
- **`enabled` guards** -- conditionally skip queries (e.g., don't read balance if wallet not connected)
- **Scope keys** -- custom cache keys for targeted invalidation (you'll set this up next)

#### Step 6: When to Use What -- Decision Guide

Use this mental model going forward:

```
"Am I in a React component?"
    YES → Use Wagmi hooks
    NO  → "Am I in the contracts/ package (tests, scripts)?"
              YES → Use ethers.js (Hardhat provides it)
              NO  → Use Viem directly (utility scripts, server-side)

"I need to format/parse a value?"
    → Use Viem utilities (formatEther, parseUnits, etc.)
      They work everywhere -- React, Node, scripts.

"I need a pre-built ABI?"
    → Import from Viem (erc20Abi, erc721Abi)
      OR import the custom ABI you extracted in Module 3.
```

### Key Takeaways

1. **ethers.js** and **Viem** both talk to the same blockchain -- they're different wrappers around the same JSON-RPC protocol
2. This project uses ethers.js *only* in Hardhat (contracts), and Viem/Wagmi *only* on the frontend -- this is standard practice
3. **Viem utilities** (formatting, parsing, validation) are pure functions and work anywhere
4. **Wagmi hooks** add React state, caching, and re-renders on top of Viem
5. You already used Viem utilities in Module 2 (Exercise 2.5) and ethers.js in Module 3 (Exercises 3.4-3.6) -- now you understand *why*

### Submission Checklist

- [ ] Can explain why the project uses two different Ethereum libraries
- [ ] Can describe the three layers of Viem (utilities, clients, ABIs)
- [ ] Knows which Wagmi hooks map to which blockchain operations
- [ ] Understands the "Am I in React?" decision guide

---

## Exercise 4.2: Set Up Scopes Pattern for Cache Management

### Objective
Create a centralized scopes system for TanStack Query cache invalidation to prevent UI flicker and enable targeted cache updates.

### Prerequisites
- [x] Completed Exercise 4.1
- [x] TanStack Query configured
- [x] Wagmi set up

### Instructions

#### Step 1: Create Scopes File
1. Create `apps/frontend/src/lib/scopes.ts`:
   ```typescript
   import { QueryClient } from '@tanstack/react-query';

   // Centralized scope key names to avoid typos/mismatch
   export const scopes = {
     // Contract-specific scopes
     storedValue: (chainId: number, address: string) =>
       ['contract', 'storedValue', chainId, address.toLowerCase()] as const,
     
     storedString: (chainId: number, address: string) =>
       ['contract', 'storedString', chainId, address.toLowerCase()] as const,
     
     // ERC-20 token scopes
     tokenBalance: (chainId: number, tokenAddress: string, userAddress: string) =>
       ['token', 'balance', chainId, tokenAddress.toLowerCase(), userAddress.toLowerCase()] as const,
     
     tokenInfo: (chainId: number, tokenAddress: string) =>
       ['token', 'info', chainId, tokenAddress.toLowerCase()] as const,
     
     // Organization/entity scopes (for future use)
     orgData: (chainId: number, address: string) =>
       ['orgs', 'data', chainId, address.toLowerCase()] as const,
     
     orgCount: (chainId: number) =>
       ['orgs', 'count', chainId] as const,
   } as const;

   // Helper function for targeted invalidation
   export function invalidateScope(
     qc: QueryClient,
     scope: readonly unknown[]
   ) {
     qc.invalidateQueries({ 
       queryKey: scope,
       exact: true 
     });
   }
   ```

**Expected Outcome:**
- Centralized scopes file created
- Stable query keys organized by chainId and entity type

**Checkpoint:**
- [ ] `src/lib/scopes.ts` exists
- [ ] Scope keys follow pattern: [entityType, property, chainId, ...identifiers]
- [ ] Helper function for invalidation created

### Common Issues & Solutions

**Issue:** Scope keys not matching between read and invalidation
- **Solution**: Always use the `scopes` object functions, never create keys manually

**Issue:** Cache not invalidating
- **Solution**: Ensure `exact: true` is used for precise invalidation, or `exact: false` for partial matches

### Submission Checklist

- [ ] Scopes file created
- [ ] Scope keys follow consistent pattern
- [ ] Helper function for invalidation created
- [ ] Keys use `as const` for type safety

---

## Exercise 4.3: Read Simple Contract State

### Objective
Read data from the SimpleStorage contract you deployed in Module 3 using Wagmi hooks with proper scopes and error handling.

### Prerequisites
- [x] Completed Exercise 4.2
- [x] SimpleStorage deployed (from Module 3) and address available
- [x] Contract ABI shared with frontend (from Module 3, Exercise 3.7)

### Instructions

#### Step 1: Import Contract Info
1. Import from the contracts file you created in Module 3:
   ```typescript
   import { SIMPLE_STORAGE_ADDRESS, SIMPLE_STORAGE_ABI } from '@/contracts/SimpleStorage';
   ```

#### Step 2: Create Read Component
1. Create `apps/frontend/src/components/ReadContract.tsx`:
   ```typescript
   import { useReadContract } from 'wagmi';
   import { useChainId } from 'wagmi';
   import { scopes } from '@/lib/scopes';
   import { SIMPLE_STORAGE_ADDRESS, SIMPLE_STORAGE_ABI } from '@/contracts/SimpleStorage';

   export function ReadContract() {
     const chainId = useChainId();
     const contractAddress = SIMPLE_STORAGE_ADDRESS;
     
     const { data: storedValue, isLoading: valueLoading, isError: valueError } = useReadContract({
       address: contractAddress,
       abi: SIMPLE_STORAGE_ABI,
       functionName: 'storedValue',
       query: {
         scopeKey: scopes.storedValue(chainId, contractAddress),
         staleTime: 30_000,
       },
     });

     const { data: storedString, isLoading: stringLoading, isError: stringError } = useReadContract({
       address: contractAddress,
       abi: SIMPLE_STORAGE_ABI,
       functionName: 'storedString',
       query: {
         scopeKey: scopes.storedString(chainId, contractAddress),
         staleTime: 30_000,
       },
     });

     if (valueLoading || stringLoading) {
       return <div>Loading contract data...</div>;
     }

     if (valueError || stringError) {
       return <div>Error reading contract data</div>;
     }

     return (
       <div className="p-4 border rounded-lg space-y-2">
         <h3 className="font-bold">Contract State</h3>
         <p><strong>Stored Value:</strong> {storedValue?.toString()}</p>
         <p><strong>Stored String:</strong> {storedString}</p>
       </div>
     );
   }
   ```

**Expected Outcome:**
- Component reads contract state from the contract deployed in Module 3
- Uses scope keys for cache management
- Handles loading and error states

**Checkpoint:**
- [ ] Component uses `useReadContract` hook
- [ ] Scope keys used in query configuration
- [ ] Loading states handled
- [ ] Error states handled
- [ ] Data displayed correctly

### Common Issues & Solutions

**Issue:** `useReadContract` returns undefined
- **Solution**: 
  - Check contract address is valid
  - Verify ABI matches contract
  - Ensure wallet is connected to correct chain

**Issue:** Data not updating after contract changes
- **Solution**: 
  - Use scope keys for proper cache invalidation
  - Check `staleTime` configuration
  - Manually invalidate queries after writes

### Submission Checklist

- [ ] Read component created
- [ ] Uses contract info from Module 3
- [ ] Uses scope keys
- [ ] Handles loading states
- [ ] Handles error states
- [ ] Data displays correctly

---

## Exercise 4.4: Read Multiple Contract Values

### Objective
Read multiple contract values simultaneously using `useReadContracts` for batch reading.

### Prerequisites
- [x] Completed Exercise 4.3
- [x] Understanding of single value reading

### Instructions

#### Step 1: Create Batch Read Component
1. Create `apps/frontend/src/components/ReadMultiple.tsx`:
   ```typescript
   import { useReadContracts } from 'wagmi';
   import { useChainId } from 'wagmi';
   import { SIMPLE_STORAGE_ADDRESS, SIMPLE_STORAGE_ABI } from '@/contracts/SimpleStorage';

   export function ReadMultiple() {
     const chainId = useChainId();
     const contractAddress = SIMPLE_STORAGE_ADDRESS;
     
     const { data, isLoading, isError } = useReadContracts({
       contracts: [
         {
           address: contractAddress,
           abi: SIMPLE_STORAGE_ABI,
           functionName: 'storedValue',
         },
         {
           address: contractAddress,
           abi: SIMPLE_STORAGE_ABI,
           functionName: 'storedString',
         },
       ],
       query: {
         scopeKey: ['contract', 'batch', chainId, contractAddress.toLowerCase()],
         staleTime: 30_000,
       },
     });

     if (isLoading) {
       return <div>Loading contract data...</div>;
     }

     if (isError) {
       return <div>Error reading contract data</div>;
     }

     const [valueResult, stringResult] = data || [];

     return (
       <div className="p-4 border rounded-lg space-y-2">
         <h3 className="font-bold">Contract State (Batch Read)</h3>
         <p><strong>Stored Value:</strong> {valueResult?.result?.toString()}</p>
         <p><strong>Stored String:</strong> {stringResult?.result}</p>
       </div>
     );
   }
   ```

**Expected Outcome:**
- Batch reading implemented
- Multiple values read in single query
- Proper error handling

**Checkpoint:**
- [ ] Uses `useReadContracts` hook
- [ ] Multiple contracts in array
- [ ] Results accessed correctly (`data?.[0]?.result`)
- [ ] Error handling implemented

### Common Issues & Solutions

**Issue:** Results array is undefined
- **Solution**: Use optional chaining: `data?.[0]?.result`

**Issue:** Results in wrong order
- **Solution**: Ensure contracts array order matches expected results order

### Submission Checklist

- [ ] Batch read component created
- [ ] Multiple values read simultaneously
- [ ] Results accessed correctly
- [ ] Error handling implemented

---

## Exercise 4.5: Read ERC-20 Token Data

### Objective
Read ERC-20 token information (name, symbol, decimals, balance) using standard ERC-20 ABI.

### Prerequisites
- [x] Completed Exercise 4.4
- [x] Understanding of ERC-20 standard

### Instructions

#### Step 1: Create Token Info Component
1. Create `apps/frontend/src/components/TokenInfo.tsx`:
   ```typescript
   import { useReadContract, useReadContracts } from 'wagmi';
   import { useAccount, useChainId } from 'wagmi';
   import { formatUnits } from 'viem';
   import { erc20Abi } from 'viem';
   import { scopes } from '@/lib/scopes';

   interface TokenInfoProps {
     tokenAddress: `0x${string}`;
   }

   export function TokenInfo({ tokenAddress }: TokenInfoProps) {
     const { address } = useAccount();
     const chainId = useChainId();
     
     // Read token info (name, symbol, decimals)
     const { data: tokenInfo, isLoading: infoLoading } = useReadContracts({
       contracts: [
         { address: tokenAddress, abi: erc20Abi, functionName: 'name' },
         { address: tokenAddress, abi: erc20Abi, functionName: 'symbol' },
         { address: tokenAddress, abi: erc20Abi, functionName: 'decimals' },
       ],
       query: {
         scopeKey: scopes.tokenInfo(chainId, tokenAddress),
         staleTime: 60_000, // Token info changes rarely
       },
     });

     // Read user balance
     const { data: balance, isLoading: balanceLoading } = useReadContract({
       address: tokenAddress,
       abi: erc20Abi,
       functionName: 'balanceOf',
       args: [address!],
       query: {
         scopeKey: scopes.tokenBalance(chainId, tokenAddress, address!),
         enabled: !!address,
         staleTime: 30_000,
       },
     });

     if (infoLoading) {
       return <div>Loading token info...</div>;
     }

     const [nameResult, symbolResult, decimalsResult] = tokenInfo || [];
     const name = nameResult?.result as string;
     const symbol = symbolResult?.result as string;
     const decimals = decimalsResult?.result as number;

     if (!name || !symbol || decimals === undefined) {
       return <div>Error loading token info</div>;
     }

     const formattedBalance = address && balance
       ? formatUnits(balance as bigint, decimals)
       : '0';

     return (
       <div className="p-4 border rounded-lg space-y-2">
         <h3 className="font-bold">Token Information</h3>
         <p><strong>Name:</strong> {name}</p>
         <p><strong>Symbol:</strong> {symbol}</p>
         <p><strong>Decimals:</strong> {decimals}</p>
         {address && (
           <div>
             <p><strong>Your Balance:</strong></p>
             {balanceLoading ? (
               <p>Loading...</p>
             ) : (
               <p className="text-lg font-semibold">
                 {formattedBalance} {symbol}
               </p>
             )}
           </div>
         )}
       </div>
     );
   }
   ```

**Expected Outcome:**
- Token information component created
- Reads name, symbol, decimals, and balance
- Formats balance using decimals

**Checkpoint:**
- [ ] Component reads token info
- [ ] Component reads user balance
- [ ] Balance formatted correctly with decimals
- [ ] Uses `enabled` guard for balance query

### Common Issues & Solutions

**Issue:** Balance shows incorrect amount
- **Solution**: Ensure you're using `formatUnits` with the correct decimals value

**Issue:** Balance query runs when address is undefined
- **Solution**: Use `enabled: !!address` in query configuration

### Submission Checklist

- [ ] Token info component created
- [ ] Reads name, symbol, decimals
- [ ] Reads user balance
- [ ] Balance formatted correctly
- [ ] Uses `enabled` guard

---

## Exercise 4.6: Listen to Contract Events (Basic)

### Objective
Listen to contract events and update UI in real-time using `useWatchContractEvent`.

### Prerequisites
- [x] Completed Exercise 4.5
- [x] Understanding of contract events

### Instructions

#### Step 1: Create Event Listener Component
1. Create `apps/frontend/src/components/EventListener.tsx`:
   ```typescript
   import { useWatchContractEvent, useReadContract } from 'wagmi';
   import { useQueryClient } from '@tanstack/react-query';
   import { useChainId } from 'wagmi';
   import { scopes } from '@/lib/scopes';
   import { SIMPLE_STORAGE_ADDRESS, SIMPLE_STORAGE_ABI } from '@/contracts/SimpleStorage';
   import { useState } from 'react';

   export function EventListener() {
     const qc = useQueryClient();
     const chainId = useChainId();
     const contractAddress = SIMPLE_STORAGE_ADDRESS;
     const [events, setEvents] = useState<Array<{ value: string; timestamp: Date }>>([]);
     
     // Read current value
     const { data: currentValue } = useReadContract({
       address: contractAddress,
       abi: SIMPLE_STORAGE_ABI,
       functionName: 'storedValue',
       query: {
         scopeKey: scopes.storedValue(chainId, contractAddress),
       },
     });

     // Watch for ValueSet events
     useWatchContractEvent({
       address: contractAddress,
       abi: SIMPLE_STORAGE_ABI,
       eventName: 'ValueSet',
       onLogs: (logs) => {
         // Invalidate query to refresh data
         qc.invalidateQueries({ 
           queryKey: scopes.storedValue(chainId, contractAddress),
           exact: true 
         });
         
         // Add to events log
         logs.forEach((log) => {
           const newValue = (log.args as any).newValue?.toString();
           setEvents((prev) => [
             { value: newValue, timestamp: new Date() },
             ...prev.slice(0, 9), // Keep last 10 events
           ]);
         });
       },
     });

     return (
       <div className="p-4 border rounded-lg space-y-4">
         <h3 className="font-bold">Event Listener</h3>
         <p><strong>Current Value:</strong> {currentValue?.toString()}</p>
         
         <div>
           <h4 className="font-semibold mb-2">Recent Events:</h4>
           {events.length === 0 ? (
             <p className="text-gray-500">No events yet. Change the value to see events.</p>
           ) : (
             <ul className="space-y-1">
               {events.map((event, idx) => (
                 <li key={idx} className="text-sm">
                   Value set to {event.value} at {event.timestamp.toLocaleTimeString()}
                 </li>
               ))}
             </ul>
           )}
         </div>
       </div>
     );
   }
   ```

**Expected Outcome:**
- Event listener component created
- Listens to contract events
- Invalidates cache on events
- Displays event log

**Checkpoint:**
- [ ] Component uses `useWatchContractEvent`
- [ ] Invalidates relevant queries on events
- [ ] Displays event log
- [ ] Updates UI when events occur

### Common Issues & Solutions

**Issue:** Events not firing
- **Solution**: 
  - Verify event name matches contract
  - Check contract address is correct
  - Ensure you're on the correct chain

**Issue:** Too many events causing performance issues
- **Solution**: Limit event log size (keep last N events)

### Submission Checklist

- [ ] Event listener component created
- [ ] Invalidates queries on events
- [ ] Displays event log
- [ ] UI updates when events occur

---

## Review Questions

1. Why does this project use ethers.js in Hardhat and Viem on the frontend? Could you use just one?
2. Name three things Wagmi adds on top of raw Viem calls.
3. What is the purpose of scope keys in TanStack Query?
4. What's the difference between `useReadContract` and `useReadContracts`?
5. Why do we use `enabled` guards in queries?
6. How do contract events help keep UI in sync with blockchain state?
7. Why is it important to format token balances using decimals?

---

## Next Steps

After completing this module, you should:
- Understand the web3 library landscape (ethers.js, Viem, Wagmi)
- Know when to use which library in which context
- Understand how to read contract state with Wagmi hooks
- Know how to use scope keys for cache management
- Be able to read ERC-20 token data
- Understand how to listen to contract events
- Handle loading and error states properly
- Proceed to Module 5: Writing to Blockchain (Transactions)
