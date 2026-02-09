# Module 3: Reading Blockchain Data

> **📖 Important**: This module follows the setup guides in `dapp_setup_guides/`. For the complete, authoritative setup instructions, refer to:
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Step-by-step setup guide
>
> The setup guides are the **single source of truth** for scaffolding and architecture decisions.

## Exercise 3.1: Set Up Scopes Pattern for Cache Management

### Objective
Create a centralized scopes system for TanStack Query cache invalidation to prevent UI flicker and enable targeted cache updates.

### Prerequisites
- [x] Completed Module 2
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

## Exercise 3.2: Read Simple Contract State

### Objective
Read data from a smart contract using Wagmi hooks with proper scopes and error handling.

### Prerequisites
- [x] Completed Exercise 3.1
- [x] Have a deployed contract address (or use a public contract)
- [x] Contract ABI available

### Instructions

#### Step 1: Prepare Contract Information
1. For this exercise, you can use any public contract on Sepolia, or deploy a simple one:
   ```solidity
   // SimpleStorage.sol
   pragma solidity ^0.8.22;
   
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

2. Get the contract address and ABI

#### Step 2: Create Contract ABI File
1. Create `apps/frontend/src/abis/SimpleStorage.json`:
   ```json
   [
     {
       "inputs": [],
       "name": "storedValue",
       "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}],
       "stateMutability": "view",
       "type": "function"
     },
     {
       "inputs": [],
       "name": "storedString",
       "outputs": [{"internalType": "string", "name": "", "type": "string"}],
       "stateMutability": "view",
       "type": "function"
     }
   ]
   ```

#### Step 3: Create Read Component
1. Create `apps/frontend/src/components/ReadContract.tsx`:
   ```typescript
   import { useReadContract } from 'wagmi';
   import { useChainId } from 'wagmi';
   import { scopes } from '@/lib/scopes';
   import SimpleStorageABI from '@/abis/SimpleStorage.json';

   interface ReadContractProps {
     contractAddress: `0x${string}`;
   }

   export function ReadContract({ contractAddress }: ReadContractProps) {
     const chainId = useChainId();
     
     const { data: storedValue, isLoading: valueLoading, isError: valueError } = useReadContract({
       address: contractAddress,
       abi: SimpleStorageABI,
       functionName: 'storedValue',
       query: {
         scopeKey: scopes.storedValue(chainId, contractAddress),
         staleTime: 30_000,
       },
     });

     const { data: storedString, isLoading: stringLoading, isError: stringError } = useReadContract({
       address: contractAddress,
       abi: SimpleStorageABI,
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
- Component reads contract state
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

- [ ] Contract ABI file created
- [ ] Read component created
- [ ] Uses scope keys
- [ ] Handles loading states
- [ ] Handles error states
- [ ] Data displays correctly

---

## Exercise 3.3: Read Multiple Contract Values

### Objective
Read multiple contract values simultaneously using `useReadContracts` for batch reading.

### Prerequisites
- [x] Completed Exercise 3.2
- [x] Understanding of single value reading

### Instructions

#### Step 1: Create Batch Read Component
1. Create `apps/frontend/src/components/ReadMultiple.tsx`:
   ```typescript
   import { useReadContracts } from 'wagmi';
   import { useChainId } from 'wagmi';
   import SimpleStorageABI from '@/abis/SimpleStorage.json';

   interface ReadMultipleProps {
     contractAddress: `0x${string}`;
   }

   export function ReadMultiple({ contractAddress }: ReadMultipleProps) {
     const chainId = useChainId();
     
     const { data, isLoading, isError } = useReadContracts({
       contracts: [
         {
           address: contractAddress,
           abi: SimpleStorageABI,
           functionName: 'storedValue',
         },
         {
           address: contractAddress,
           abi: SimpleStorageABI,
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

## Exercise 3.4: Read ERC-20 Token Data

### Objective
Read ERC-20 token information (name, symbol, decimals, balance) using standard ERC-20 ABI.

### Prerequisites
- [x] Completed Exercise 3.3
- [x] Understanding of ERC-20 standard

### Instructions

#### Step 1: Create ERC-20 ABI
1. Create `apps/frontend/src/abis/ERC20.json`:
   ```json
   [
     {
       "inputs": [],
       "name": "name",
       "outputs": [{"internalType": "string", "name": "", "type": "string"}],
       "stateMutability": "view",
       "type": "function"
     },
     {
       "inputs": [],
       "name": "symbol",
       "outputs": [{"internalType": "string", "name": "", "type": "string"}],
       "stateMutability": "view",
       "type": "function"
     },
     {
       "inputs": [],
       "name": "decimals",
       "outputs": [{"internalType": "uint8", "name": "", "type": "uint8"}],
       "stateMutability": "view",
       "type": "function"
     },
     {
       "inputs": [{"internalType": "address", "name": "account", "type": "address"}],
       "name": "balanceOf",
       "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}],
       "stateMutability": "view",
       "type": "function"
     }
   ]
   ```

#### Step 2: Create Token Info Component
1. Create `apps/frontend/src/components/TokenInfo.tsx`:
   ```typescript
   import { useReadContract, useReadContracts } from 'wagmi';
   import { useAccount, useChainId } from 'wagmi';
   import { formatUnits } from 'viem';
   import ERC20ABI from '@/abis/ERC20.json';
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
         { address: tokenAddress, abi: ERC20ABI, functionName: 'name' },
         { address: tokenAddress, abi: ERC20ABI, functionName: 'symbol' },
         { address: tokenAddress, abi: ERC20ABI, functionName: 'decimals' },
       ],
       query: {
         scopeKey: scopes.tokenInfo(chainId, tokenAddress),
         staleTime: 60_000, // Token info changes rarely
       },
     });

     // Read user balance
     const { data: balance, isLoading: balanceLoading } = useReadContract({
       address: tokenAddress,
       abi: ERC20ABI,
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
- [ ] ERC-20 ABI created
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

- [ ] ERC-20 ABI created
- [ ] Token info component created
- [ ] Reads name, symbol, decimals
- [ ] Reads user balance
- [ ] Balance formatted correctly
- [ ] Uses `enabled` guard

---

## Exercise 3.5: Listen to Contract Events (Basic)

### Objective
Listen to contract events and update UI in real-time using `useWatchContractEvent`.

### Prerequisites
- [x] Completed Exercise 3.4
- [x] Understanding of contract events

### Instructions

#### Step 1: Add Event to ABI
1. Update `apps/frontend/src/abis/SimpleStorage.json` to include event:
   ```json
   [
     {
       "inputs": [],
       "name": "storedValue",
       "outputs": [{"internalType": "uint256", "name": "", "type": "uint256"}],
       "stateMutability": "view",
       "type": "function"
     },
     {
       "anonymous": false,
       "inputs": [
         {"indexed": false, "internalType": "uint256", "name": "newValue", "type": "uint256"}
       ],
       "name": "ValueSet",
       "type": "event"
     }
   ]
   ```

#### Step 2: Create Event Listener Component
1. Create `apps/frontend/src/components/EventListener.tsx`:
   ```typescript
   import { useWatchContractEvent, useReadContract } from 'wagmi';
   import { useQueryClient } from '@tanstack/react-query';
   import { useChainId } from 'wagmi';
   import { scopes } from '@/lib/scopes';
   import SimpleStorageABI from '@/abis/SimpleStorage.json';
   import { useState } from 'react';

   interface EventListenerProps {
     contractAddress: `0x${string}`;
   }

   export function EventListener({ contractAddress }: EventListenerProps) {
     const qc = useQueryClient();
     const chainId = useChainId();
     const [events, setEvents] = useState<Array<{ value: string; timestamp: Date }>>([]);
     
     // Read current value
     const { data: currentValue } = useReadContract({
       address: contractAddress,
       abi: SimpleStorageABI,
       functionName: 'storedValue',
       query: {
         scopeKey: scopes.storedValue(chainId, contractAddress),
       },
     });

     // Watch for ValueSet events
     useWatchContractEvent({
       address: contractAddress,
       abi: SimpleStorageABI,
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

- [ ] Event added to ABI
- [ ] Event listener component created
- [ ] Invalidates queries on events
- [ ] Displays event log
- [ ] UI updates when events occur

---

## Review Questions

1. What is the purpose of scope keys in TanStack Query?
2. What's the difference between `useReadContract` and `useReadContracts`?
3. Why do we use `enabled` guards in queries?
4. How do contract events help keep UI in sync with blockchain state?
5. Why is it important to format token balances using decimals?

---

## Next Steps

After completing this module, you should:
- Understand how to read contract state
- Know how to use scope keys for cache management
- Be able to read ERC-20 token data
- Understand how to listen to contract events
- Handle loading and error states properly
- Proceed to Module 4: Writing to Blockchain (Transactions)
