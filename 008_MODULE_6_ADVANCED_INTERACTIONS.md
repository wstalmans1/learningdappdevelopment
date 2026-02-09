# Module 6: Advanced Contract Interactions

> **📖 Important**: This module follows the setup guides in `dapp_setup_guides/`. For the complete, authoritative setup instructions, refer to:
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Step-by-step setup guide
>
> The setup guides are the **single source of truth** for scaffolding and architecture decisions.

## Exercise 6.1: ERC-20 Token Transfer with Approval

### Objective
Implement ERC-20 token transfer with the approval pattern (two-step transaction).

### Prerequisites
- [x] Completed Module 5
- [x] Token contract deployed (from Module 3)
- [x] Understanding of ERC-20 standard

### Instructions

#### Step 1: Create Approval Component
1. Create `apps/frontend/src/components/TokenApproval.tsx`:
   ```typescript
   import { useState } from 'react';
   import { useWriteContract, useReadContract, useWaitForTransactionReceipt } from 'wagmi';
   import { useAccount, useChainId } from 'wagmi';
   import { useQueryClient } from '@tanstack/react-query';
   import { useModalStore } from '@/stores/modalStore';
   import { parseUnits, formatUnits } from 'viem';
   import { erc20Abi } from 'viem';
   import { scopes } from '@/lib/scopes';

   interface TokenApprovalProps {
     tokenAddress: `0x${string}`;
     spenderAddress: `0x${string}`;
   }

   export function TokenApproval({ tokenAddress, spenderAddress }: TokenApprovalProps) {
     const [amount, setAmount] = useState('');
     const { address } = useAccount();
     const chainId = useChainId();
     const qc = useQueryClient();
     const { open } = useModalStore();

     // Read token decimals
     const { data: decimals } = useReadContract({
       address: tokenAddress,
       abi: erc20Abi,
       functionName: 'decimals',
       query: {
         scopeKey: scopes.tokenInfo(chainId, tokenAddress),
       },
     });

     // Read current allowance
     const { data: allowance } = useReadContract({
       address: tokenAddress,
       abi: erc20Abi,
       functionName: 'allowance',
       args: [address!, spenderAddress],
       query: {
         enabled: !!address,
         scopeKey: ['token', 'allowance', chainId, tokenAddress, address, spenderAddress],
       },
     });

     const { writeContract: approve, data: approveHash, isPending: isApproving } = useWriteContract();
     const { isLoading: isApprovingConfirming, isSuccess: approveSuccess } = useWaitForTransactionReceipt({
       hash: approveHash,
     });

     const { writeContract: transfer, data: transferHash, isPending: isTransferring } = useWriteContract();
     const { isLoading: isTransferringConfirming, isSuccess: transferSuccess } = useWaitForTransactionReceipt({
       hash: transferHash,
     });

     // Show overlays
     if (isApproving || isApprovingConfirming) {
       open('transaction', { type: isApproving ? 'pending' : 'confirming' });
     }
     if (isTransferring || isTransferringConfirming) {
       open('transaction', { type: isTransferring ? 'pending' : 'confirming' });
     }
     if (approveSuccess || transferSuccess) {
       open('success', { transactionHash: approveHash || transferHash });
       // Invalidate queries
       qc.invalidateQueries({ queryKey: ['token', 'allowance', chainId] });
       qc.invalidateQueries({ queryKey: scopes.tokenBalance(chainId, tokenAddress, address!) });
     }

     const handleApprove = () => {
       if (!decimals || !amount) return;
       const amountWei = parseUnits(amount, decimals);
       approve({
         address: tokenAddress,
         abi: erc20Abi,
         functionName: 'approve',
         args: [spenderAddress, amountWei],
       });
     };

     const handleTransfer = () => {
       if (!decimals || !amount) return;
       const amountWei = parseUnits(amount, decimals);
       transfer({
         address: tokenAddress,
         abi: erc20Abi,
         functionName: 'transfer',
         args: [spenderAddress, amountWei],
       });
     };

     const needsApproval = allowance && decimals && amount
       ? BigInt(allowance.toString()) < parseUnits(amount, decimals)
       : true;

     return (
       <div className="p-4 border rounded-lg space-y-4">
         <h3 className="font-bold">Token Transfer</h3>
         <div>
           <label className="block text-sm font-medium mb-1">Amount</label>
           <input
             type="text"
             value={amount}
             onChange={(e) => setAmount(e.target.value)}
             placeholder="0.0"
             className="border rounded px-3 py-2 w-full"
             disabled={isApproving || isTransferring}
           />
         </div>
         <div className="space-y-2">
           {needsApproval ? (
             <button
               onClick={handleApprove}
               disabled={isApproving || isApprovingConfirming || !amount}
               className="w-full px-4 py-2 bg-yellow-600 text-white rounded hover:bg-yellow-700 disabled:opacity-50"
             >
               {isApproving ? 'Approving...' : 'Approve Token Spending'}
             </button>
           ) : (
             <button
               onClick={handleTransfer}
               disabled={isTransferring || isTransferringConfirming || !amount}
               className="w-full px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:opacity-50"
             >
               {isTransferring ? 'Transferring...' : 'Transfer Tokens'}
             </button>
           )}
         </div>
         {allowance && decimals && (
           <p className="text-sm text-gray-600">
             Current allowance: {formatUnits(BigInt(allowance.toString()), decimals)}
           </p>
         )}
       </div>
     );
   }
   ```

**Expected Outcome:**
- Token approval component created
- Two-step transaction flow (approve then transfer)
- Checks allowance before transfer

**Checkpoint:**
- [ ] Approval component created
- [ ] Checks current allowance
- [ ] Shows approve button if needed
- [ ] Shows transfer button if approved
- [ ] Handles both transactions

### Submission Checklist

- [ ] Token approval component created
- [ ] Approval pattern implemented
- [ ] Allowance checking implemented
- [ ] Two-step flow works correctly

---

## Exercise 6.2: Real-Time Event System (Production)

### Objective
Set up a real-time event system using a separate WebSocket client to prevent HTTP polling.

### Prerequisites
- [x] Completed Exercise 6.1
- [x] Understanding of event watching
- [x] Contract with events deployed

### Instructions

#### Step 1: Create WebSocket Client
1. Create `apps/frontend/src/realtime/wsClient.ts`:
   ```typescript
   import { createPublicClient, webSocket } from 'viem';
   import { APP_CHAIN } from '@/config/chain';
   import { ALCHEMY_WS_SEPOLIA } from '@/config/providers';

   // Create a WebSocket-only client for event watching
   // This client will NEVER fall back to HTTP polling
   export const wsClient = createPublicClient({
     chain: APP_CHAIN,
     transport: webSocket(ALCHEMY_WS_SEPOLIA),
   });
   ```

#### Step 2: Create Global Event Hook
1. Create `apps/frontend/src/realtime/useGlobalEvents.ts`:
   ```typescript
   import { useEffect, useRef } from 'react';
   import { useQueryClient } from '@tanstack/react-query';
   import { wsClient } from './wsClient';
   import { APP_CHAIN_ID } from '@/config/chain';
   import { scopes } from '@/lib/scopes';
   import { SIMPLE_STORAGE_ABI } from '@/contracts/SimpleStorage';

   type Addr = `0x${string}`;

   export function useGlobalEvents(watchAddresses: Addr[]) {
     const qc = useQueryClient();
     const unsubsRef = useRef<(() => void)[]>([]);

     useEffect(() => {
       async function setup() {
         // Clear existing watchers
         unsubsRef.current.forEach(u => {
           try { u() } catch {}
         });
         unsubsRef.current = [];

         // Watch ValueSet events for all addresses
         if (watchAddresses.length > 0) {
           unsubsRef.current.push(
             wsClient.watchContractEvent({
               address: watchAddresses,
               abi: SIMPLE_STORAGE_ABI,
               eventName: 'ValueSet',
               strict: false,
               poll: false, // WebSocket-only, no HTTP polling
               onLogs: (logs) => {
                 // Invalidate relevant queries for each affected contract
                 logs.forEach((log) => {
                   const contractAddress = (log.address as Addr).toLowerCase();
                   qc.invalidateQueries({ 
                     queryKey: scopes.storedValue(APP_CHAIN_ID, contractAddress),
                     exact: true 
                   });
                 });
               },
             })
           );
         }
       }

       setup();

       return () => {
         // Cleanup on unmount
         unsubsRef.current.forEach(u => {
           try { u() } catch {}
         });
         unsubsRef.current = [];
       };
     }, [watchAddresses, qc]);
   }
   ```

#### Step 3: Use Global Event Hook in App
1. Update `apps/frontend/src/App.tsx`:
   ```typescript
   import { useGlobalEvents } from './realtime/useGlobalEvents';

   function App() {
     // Watch events for contracts you're interested in
     const contractAddresses: `0x${string}`[] = [
       '0x...', // Your contract addresses
     ];
     useGlobalEvents(contractAddresses);

     return (
       // Your app content
     );
   }
   ```

**Expected Outcome:**
- Separate WebSocket client created
- Global event hook implemented
- Events watched without HTTP polling
- Cache invalidated on events

**Checkpoint:**
- [ ] WebSocket client created
- [ ] Global event hook created
- [ ] Hook mounted in App
- [ ] Events trigger cache invalidation
- [ ] No HTTP polling occurs

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete event system implementation.

### Submission Checklist

- [ ] WebSocket client created
- [ ] Global event hook created
- [ ] Hook integrated in App
- [ ] Events trigger cache updates
- [ ] No HTTP polling

---

## Review Questions

1. Why do ERC-20 transfers require approval first?
2. Why use a separate WebSocket client for event watching?
3. What is the difference between HTTP polling and WebSocket subscriptions?
4. How does event-driven cache invalidation improve UX?

---

## Next Steps

After completing this module, you should:
- Be able to implement approval patterns
- Understand real-time event systems
- Know how to use WebSocket for event watching
- Proceed to Module 7: State Management & Data Fetching
