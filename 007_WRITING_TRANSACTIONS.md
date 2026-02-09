# Writing to Blockchain (Transactions)

> **📖 Important**: This module follows the setup guides in `dapp_setup_guides/`. For the complete, authoritative setup instructions, refer to:
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Step-by-step setup guide
>
> The setup guides are the **single source of truth** for scaffolding and architecture decisions.

## Exercise 5.1: Create Transaction Overlay System (REQUIRED)

### Objective
Implement a transaction overlay system for production-grade UX that provides clear feedback during blockchain transactions.

### Prerequisites
- [x] Completed Reading Blockchain Data
- [x] Understanding of transaction lifecycle
- [x] Zustand installed

### Instructions

#### Step 1: Create Modal Store
1. Create `apps/frontend/src/stores/modalStore.ts`:
   ```typescript
   import { create } from 'zustand';

   type ModalType = 'transaction' | 'success' | null;

   interface ModalState {
     current: ModalType;
     payload?: {
       type?: 'pending' | 'confirming';
       title?: string;
       message?: string;
       transactionHash?: string;
     };
     open: (type: ModalType, payload?: ModalState['payload']) => void;
     close: () => void;
   }

   export const useModalStore = create<ModalState>((set) => ({
     current: null,
     payload: undefined,
     open: (type, payload) => set({ current: type, payload }),
     close: () => set({ current: null, payload: undefined }),
   }));
   ```

**Expected Outcome:**
- Modal store created with Zustand
- Manages transaction overlay state

**Checkpoint:**
- [ ] Modal store created
- [ ] Can open/close modals
- [ ] Payload supports transaction states

#### Step 2: Create Transaction Overlay Component
1. Create `apps/frontend/src/components/ui/TransactionOverlay.tsx`:
   ```typescript
   interface TransactionOverlayProps {
     isVisible: boolean;
     type: 'pending' | 'confirming';
     title?: string;
     message?: string;
   }

   export function TransactionOverlay({ 
     isVisible, 
     type, 
     title,
     message 
   }: TransactionOverlayProps) {
     if (!isVisible) return null;

     const defaultTitles = {
       pending: 'Waiting for Signature',
       confirming: 'Processing Transaction',
     };

     const defaultMessages = {
       pending: 'Please confirm the transaction in your wallet. Check your mobile wallet if connected there.',
       confirming: 'Your transaction is being processed on the blockchain. Please wait...',
     };

     return (
       <div 
         className="fixed inset-0 bg-black bg-opacity-50 backdrop-blur-sm flex items-center justify-center z-50"
         role="dialog"
         aria-labelledby="transaction-overlay-title"
       >
         <div className="bg-white rounded-xl p-8 shadow-2xl border border-gray-200 max-w-md mx-4">
           <div className="text-center space-y-4">
             <div className="w-16 h-16 bg-blue-100 border-blue-600 rounded-full flex items-center justify-center mx-auto">
               {type === 'pending' ? (
                 <div className="w-8 h-8 border-4 border-blue-600 border-t-transparent rounded-full animate-spin"></div>
               ) : (
                 <div className="w-8 h-8 border-4 border-green-600 border-t-transparent rounded-full animate-spin"></div>
               )}
             </div>
             <div>
               <h3 
                 id="transaction-overlay-title"
                 className="text-xl font-bold text-gray-900 mb-2"
               >
                 {title || defaultTitles[type]}
               </h3>
               <p className="text-gray-600">
                 {message || defaultMessages[type]}
               </p>
             </div>
           </div>
         </div>
       </div>
     );
   }
   ```

#### Step 3: Create Success Overlay Component
1. Create `apps/frontend/src/components/ui/SuccessOverlay.tsx`:
   ```typescript
   interface SuccessOverlayProps {
     isVisible: boolean;
     transactionHash?: string;
     onClose: () => void;
   }

   export function SuccessOverlay({ 
     isVisible, 
     transactionHash,
     onClose 
   }: SuccessOverlayProps) {
     if (!isVisible) return null;

     const explorerUrl = `https://sepolia.etherscan.io/tx/${transactionHash}`;

     return (
       <div 
         className="fixed inset-0 bg-black bg-opacity-50 backdrop-blur-sm flex items-center justify-center z-50"
         role="dialog"
         aria-labelledby="success-overlay-title"
       >
         <div className="bg-white rounded-xl p-8 shadow-2xl border border-gray-200 max-w-md mx-4">
           <div className="text-center space-y-4">
             <div className="w-16 h-16 bg-green-100 border-green-600 rounded-full flex items-center justify-center mx-auto">
               <svg className="w-8 h-8 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                 <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M5 13l4 4L19 7" />
               </svg>
             </div>
             <div>
               <h3 
                 id="success-overlay-title"
                 className="text-xl font-bold text-gray-900 mb-2"
               >
                 Transaction Confirmed!
               </h3>
               <p className="text-gray-600 mb-4">
                 Your transaction has been successfully processed on the blockchain.
               </p>
               {transactionHash && (
                 <a 
                   href={explorerUrl}
                   target="_blank"
                   rel="noopener noreferrer"
                   className="text-blue-600 hover:underline"
                 >
                   View on Etherscan
                 </a>
               )}
             </div>
             <button
               onClick={onClose}
               className="mt-4 px-6 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700"
             >
               Close
             </button>
           </div>
         </div>
       </div>
     );
   }
   ```

#### Step 4: Create Modal Manager
1. Create `apps/frontend/src/components/ModalManager.tsx`:
   ```typescript
   import { useModalStore } from '@/stores/modalStore';
   import { TransactionOverlay } from './ui/TransactionOverlay';
   import { SuccessOverlay } from './ui/SuccessOverlay';

   export function ModalManager() {
     const { current, payload, close } = useModalStore();

     return (
       <>
         {current === 'transaction' && payload && (
           <TransactionOverlay
             isVisible={true}
             type={payload.type || 'pending'}
             title={payload.title}
             message={payload.message}
           />
         )}
         {current === 'success' && (
           <SuccessOverlay
             isVisible={true}
             transactionHash={payload?.transactionHash}
             onClose={close}
           />
         )}
       </>
     );
   }
   ```

2. Add ModalManager to App.tsx:
   ```typescript
   import { ModalManager } from './components/ModalManager';

   function App() {
     return (
       <>
         {/* Your app content */}
         <ModalManager />
       </>
     );
   }
   ```

**Expected Outcome:**
- Transaction overlay system implemented
- Pending and confirming states
- Success overlay with transaction link
- Modal manager orchestrates overlays

**Checkpoint:**
- [ ] Modal store created
- [ ] Transaction overlay component created
- [ ] Success overlay component created
- [ ] Modal manager created
- [ ] Modal manager added to App

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete overlay system implementation.

### Submission Checklist

- [ ] Modal store created with Zustand
- [ ] Transaction overlay component created
- [ ] Success overlay component created
- [ ] Modal manager created
- [ ] Modal manager added to App
- [ ] Overlays are accessible (ARIA labels)

---

## Exercise 5.2: Write to Contract (Simple)

### Objective
Send a transaction to a smart contract using Wagmi hooks with proper transaction overlay integration.

### Prerequisites
- [x] Completed Exercise 5.1
- [x] Transaction overlay system set up
- [x] Contract deployed and address available

### Instructions

#### Step 1: Create Write Component
1. Create `apps/frontend/src/components/WriteContract.tsx`:
   ```typescript
   import { useState } from 'react';
   import { useWriteContract, useWaitForTransactionReceipt } from 'wagmi';
   import { useQueryClient } from '@tanstack/react-query';
   import { useChainId } from 'wagmi';
   import { useModalStore } from '@/stores/modalStore';
   import { scopes } from '@/lib/scopes';
   import SimpleStorageABI from '@/abis/SimpleStorage.json';

   interface WriteContractProps {
     contractAddress: `0x${string}`;
   }

   export function WriteContract({ contractAddress }: WriteContractProps) {
     const [value, setValue] = useState('');
     const qc = useQueryClient();
     const chainId = useChainId();
     const { open, close } = useModalStore();

     const { 
       writeContract, 
       data: hash, 
       isPending, 
       error: writeError 
     } = useWriteContract();

     const { 
       isLoading: isConfirming, 
       isSuccess,
       error: receiptError 
     } = useWaitForTransactionReceipt({ 
       hash 
     });

     // Show overlay when pending or confirming
     if (isPending || isConfirming) {
       open('transaction', {
         type: isPending ? 'pending' : 'confirming',
       });
     }

     // Show success overlay when transaction succeeds
     if (isSuccess && hash) {
       open('success', {
         transactionHash: hash,
       });
       // Invalidate queries to refresh data
       qc.invalidateQueries({ 
         queryKey: scopes.storedValue(chainId, contractAddress),
         exact: true 
       });
     }

     const handleSubmit = async (e: React.FormEvent) => {
       e.preventDefault();
       const numValue = parseInt(value);
       if (isNaN(numValue)) {
         alert('Please enter a valid number');
         return;
       }

       writeContract({
         address: contractAddress,
         abi: SimpleStorageABI,
         functionName: 'setValue',
         args: [BigInt(numValue)],
       });
     };

     return (
       <div className="p-4 border rounded-lg space-y-4">
         <h3 className="font-bold">Set Contract Value</h3>
         <form onSubmit={handleSubmit} className="space-y-2">
           <input
             type="number"
             value={value}
             onChange={(e) => setValue(e.target.value)}
             placeholder="Enter value"
             className="border rounded px-3 py-2 w-full"
             disabled={isPending || isConfirming}
           />
           <button
             type="submit"
             disabled={isPending || isConfirming || !value}
             className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:opacity-50"
           >
             {isPending ? 'Confirming...' : isConfirming ? 'Processing...' : 'Set Value'}
           </button>
         </form>
         {(writeError || receiptError) && (
           <div className="text-red-600">
             Error: {writeError?.message || receiptError?.message}
           </div>
         )}
       </div>
     );
   }
   ```

**Expected Outcome:**
- Write component created
- Integrates with transaction overlay
- Invalidates cache after success
- Handles errors properly

**Checkpoint:**
- [ ] Uses `useWriteContract` hook
- [ ] Uses `useWaitForTransactionReceipt` hook
- [ ] Shows overlay during pending/confirming
- [ ] Shows success overlay after confirmation
- [ ] Invalidates queries after success
- [ ] Handles errors

### Common Issues & Solutions

**Issue:** Overlay doesn't show
- **Solution**: Ensure ModalManager is rendered in App and modal store is being used

**Issue:** Transaction fails silently
- **Solution**: Check error handling and display error messages to user

**Issue:** Data doesn't refresh after transaction
- **Solution**: Ensure queries are invalidated after successful transaction

### Submission Checklist

- [ ] Write component created
- [ ] Transaction overlay integrated
- [ ] Success overlay shows after confirmation
- [ ] Cache invalidation implemented
- [ ] Error handling implemented
- [ ] Form validation implemented

---

## Exercise 5.3: Gas Estimation

### Objective
Estimate gas costs before sending transactions and display them to users.

### Prerequisites
- [x] Completed Exercise 5.2
- [x] Understanding of gas and transaction costs

### Instructions

#### Step 1: Create Gas Estimation Component
1. Create `apps/frontend/src/components/GasEstimate.tsx`:
   ```typescript
   import { useEstimateGas, useGasPrice } from 'wagmi';
   import { parseUnits, formatUnits } from 'viem';
   import SimpleStorageABI from '@/abis/SimpleStorage.json';

   interface GasEstimateProps {
     contractAddress: `0x${string}`;
     value: bigint;
   }

   export function GasEstimate({ contractAddress, value }: GasEstimateProps) {
     const { data: gasPrice } = useGasPrice();
     
     const { data: gasEstimate, isLoading, isError } = useEstimateGas({
       to: contractAddress,
       account: undefined, // Will use connected account
       data: undefined, // Would need to encode function call
     });

     if (isLoading) {
       return <div className="text-sm text-gray-600">Estimating gas...</div>;
     }

     if (isError || !gasEstimate || !gasPrice) {
       return <div className="text-sm text-red-600">Could not estimate gas</div>;
     }

     // Calculate total cost (gas limit * gas price)
     const totalCost = gasEstimate * gasPrice;
     const costInEth = formatUnits(totalCost, 18);

     return (
       <div className="text-sm space-y-1">
         <p><strong>Estimated Gas:</strong> {gasEstimate.toString()}</p>
         <p><strong>Gas Price:</strong> {formatUnits(gasPrice, 9)} Gwei</p>
         <p><strong>Estimated Cost:</strong> {parseFloat(costInEth).toFixed(6)} ETH</p>
       </div>
     );
   }
   ```

**Expected Outcome:**
- Gas estimation component created
- Displays gas limit, gas price, and total cost
- Handles loading and error states

**Checkpoint:**
- [ ] Uses `useEstimateGas` hook
- [ ] Uses `useGasPrice` hook
- [ ] Calculates total cost
- [ ] Formats values correctly
- [ ] Handles errors

### Submission Checklist

- [ ] Gas estimation component created
- [ ] Displays gas limit
- [ ] Displays gas price
- [ ] Calculates and displays total cost
- [ ] Handles loading/error states

---

## Exercise 5.4: Complex Transaction with Validation

### Objective
Create a transaction with multiple parameters and proper input validation.

### Prerequisites
- [x] Completed Exercise 5.3
- [x] Understanding of form validation

### Instructions

#### Step 1: Create Complex Write Component
1. Create `apps/frontend/src/components/ComplexWrite.tsx`:
   ```typescript
   import { useState } from 'react';
   import { useWriteContract, useWaitForTransactionReceipt } from 'wagmi';
   import { useAccount, useChainId } from 'wagmi';
   import { useQueryClient } from '@tanstack/react-query';
   import { useModalStore } from '@/stores/modalStore';
   import { isAddress } from 'viem';
   import SimpleStorageABI from '@/abis/SimpleStorage.json';

   interface ComplexWriteProps {
     contractAddress: `0x${string}`;
   }

   export function ComplexWrite({ contractAddress }: ComplexWriteProps) {
     const [stringValue, setStringValue] = useState('');
     const [errors, setErrors] = useState<Record<string, string>>({});
     const { address } = useAccount();
     const chainId = useChainId();
     const qc = useQueryClient();
     const { open } = useModalStore();

     const { writeContract, data: hash, isPending } = useWriteContract();
     const { isLoading: isConfirming, isSuccess } = useWaitForTransactionReceipt({ hash });

     const validate = () => {
       const newErrors: Record<string, string> = {};
       
       if (!stringValue.trim()) {
         newErrors.stringValue = 'String value is required';
       }
       
       if (stringValue.length > 100) {
         newErrors.stringValue = 'String must be 100 characters or less';
       }

       setErrors(newErrors);
       return Object.keys(newErrors).length === 0;
     };

     const handleSubmit = async (e: React.FormEvent) => {
       e.preventDefault();
       
       if (!validate()) {
         return;
       }

       if (!address) {
         alert('Please connect your wallet');
         return;
       }

       writeContract({
         address: contractAddress,
         abi: SimpleStorageABI,
         functionName: 'setString',
         args: [stringValue],
       });
     };

     // Show overlays
     if (isPending || isConfirming) {
       open('transaction', {
         type: isPending ? 'pending' : 'confirming',
       });
     }

     if (isSuccess) {
       open('success', { transactionHash: hash });
     }

     return (
       <form onSubmit={handleSubmit} className="p-4 border rounded-lg space-y-4">
         <h3 className="font-bold">Set String Value</h3>
         <div>
           <label className="block text-sm font-medium mb-1">
             String Value
           </label>
           <input
             type="text"
             value={stringValue}
             onChange={(e) => {
               setStringValue(e.target.value);
               setErrors({});
             }}
             className={`border rounded px-3 py-2 w-full ${
               errors.stringValue ? 'border-red-500' : ''
             }`}
             disabled={isPending || isConfirming}
             maxLength={100}
           />
           {errors.stringValue && (
             <p className="text-red-600 text-sm mt-1">{errors.stringValue}</p>
           )}
         </div>
         <button
           type="submit"
           disabled={isPending || isConfirming || !stringValue.trim()}
           className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:opacity-50"
         >
           {isPending ? 'Confirming...' : isConfirming ? 'Processing...' : 'Set String'}
         </button>
       </form>
     );
   }
   ```

**Expected Outcome:**
- Complex write component with validation
- Multiple parameter support
- Error handling and display

**Checkpoint:**
- [ ] Form validation implemented
- [ ] Error messages displayed
- [ ] Transaction overlay integrated
- [ ] Handles multiple parameters
- [ ] Input sanitization

### Submission Checklist

- [ ] Complex write component created
- [ ] Input validation implemented
- [ ] Error messages displayed
- [ ] Transaction overlay integrated
- [ ] Multiple parameters handled

---

## Review Questions

1. Why is a transaction overlay system required for production UX?
2. What's the difference between `isPending` and `isConfirming`?
3. Why do we invalidate queries after successful transactions?
4. How do you estimate gas costs before sending a transaction?
5. What are best practices for transaction error handling?

---

## Next Steps

After completing this module, you should:
- Understand transaction lifecycle (pending → confirming → success)
- Be able to implement transaction overlays
- Know how to estimate gas costs
- Handle transaction errors gracefully
- Invalidate cache after transactions
- Proceed to Advanced Contract Interactions
