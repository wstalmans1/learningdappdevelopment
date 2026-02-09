# Module 8: User Experience & Best Practices

> **📖 Important**: This module follows the setup guides in `dapp_setup_guides/`. For the complete, authoritative setup instructions, refer to:
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Step-by-step setup guide
>
> The setup guides are the **single source of truth** for scaffolding and architecture decisions.

## Exercise 8.1: Connection Health Monitoring

### Objective
Implement connection health monitoring to track RPC provider health and show status to users.

### Prerequisites
- [x] Completed Module 7
- [x] Zustand installed
- [x] Understanding of RPC connections

### Instructions

#### Step 1: Create Connection Health Store
1. Create `apps/frontend/src/stores/connectionHealthStore.ts`:
   ```typescript
   import { create } from 'zustand';

   type ConnectionQuality = 'excellent' | 'good' | 'poor' | 'failed';

   interface ConnectionHealthState {
     quality: ConnectionQuality;
     consecutiveFailures: number;
     lastError?: string;
     responseTime?: number;
     updateHealth: (quality: ConnectionQuality, responseTime?: number, error?: string) => void;
     reset: () => void;
   }

   export const useConnectionHealthStore = create<ConnectionHealthState>((set) => ({
     quality: 'excellent',
     consecutiveFailures: 0,
     lastError: undefined,
     responseTime: undefined,
     updateHealth: (quality, responseTime, error) => {
       set((state) => ({
         quality,
         responseTime,
         lastError: error,
         consecutiveFailures: quality === 'failed' 
           ? state.consecutiveFailures + 1 
           : 0,
       }));
     },
     reset: () => set({
       quality: 'excellent',
       consecutiveFailures: 0,
       lastError: undefined,
       responseTime: undefined,
     }),
   }));
   ```

#### Step 2: Create Health Check Hook
1. Create `apps/frontend/src/hooks/useConnectionHealth.ts`:
   ```typescript
   import { useEffect, useRef } from 'react';
   import { usePublicClient } from 'wagmi';
   import { useConnectionHealthStore } from '@/stores/connectionHealthStore';

   export function useConnectionHealth() {
     const publicClient = usePublicClient();
     const { updateHealth } = useConnectionHealthStore();
     const intervalRef = useRef<NodeJS.Timeout>();

     useEffect(() => {
       const checkHealth = async () => {
         if (!publicClient) return;

         const startTime = Date.now();
         try {
           await publicClient.getBlockNumber();
           const responseTime = Date.now() - startTime;

           // Determine quality based on response time
           let quality: 'excellent' | 'good' | 'poor' | 'failed';
           if (responseTime < 500) {
             quality = 'excellent';
           } else if (responseTime < 1000) {
             quality = 'good';
           } else {
             quality = 'poor';
           }

           updateHealth(quality, responseTime);
         } catch (error) {
           updateHealth('failed', undefined, (error as Error).message);
         }
       };

       // Initial check
       checkHealth();

       // Check every 2 minutes
       intervalRef.current = setInterval(checkHealth, 2 * 60 * 1000);

       return () => {
         if (intervalRef.current) {
           clearInterval(intervalRef.current);
         }
       };
     }, [publicClient, updateHealth]);
   }
   ```

#### Step 3: Create Connection Indicator Component
1. Create `apps/frontend/src/components/ConnectionIndicator.tsx`:
   ```typescript
   import { useConnectionHealthStore } from '@/stores/connectionHealthStore';

   export function ConnectionIndicator() {
     const { quality, responseTime, consecutiveFailures } = useConnectionHealthStore();

     const getColor = () => {
       switch (quality) {
         case 'excellent': return 'bg-green-500';
         case 'good': return 'bg-yellow-500';
         case 'poor': return 'bg-orange-500';
         case 'failed': return 'bg-red-500';
         default: return 'bg-gray-500';
       }
     };

     const getLabel = () => {
       switch (quality) {
         case 'excellent': return 'Excellent';
         case 'good': return 'Good';
         case 'poor': return 'Poor';
         case 'failed': return 'Failed';
         default: return 'Unknown';
       }
     };

     return (
       <div className="flex items-center gap-2 px-3 py-1 rounded-full bg-gray-100">
         <div className={`w-2 h-2 rounded-full ${getColor()}`} />
         <span className="text-sm">
           {getLabel()}
           {responseTime && ` (${responseTime}ms)`}
           {consecutiveFailures > 0 && ` (${consecutiveFailures} failures)`}
         </span>
       </div>
     );
   }
   ```

#### Step 4: Integrate Health Monitoring
1. Add to `apps/frontend/src/App.tsx`:
   ```typescript
   import { useConnectionHealth } from '@/hooks/useConnectionHealth';
   import { ConnectionIndicator } from './components/ConnectionIndicator';

   function App() {
     useConnectionHealth(); // Start health monitoring

     return (
       <div>
         <header>
           <ConnectionIndicator />
         </header>
         {/* Rest of app */}
       </div>
     );
   }
   ```

**Expected Outcome:**
- Connection health monitoring implemented
- Health status displayed to users
- Automatic health checks

**Checkpoint:**
- [ ] Connection health store created
- [ ] Health check hook created
- [ ] Connection indicator component created
- [ ] Health monitoring integrated

### Submission Checklist

- [ ] Connection health store created
- [ ] Health check hook created
- [ ] Connection indicator created
- [ ] Health monitoring integrated
- [ ] Status displayed to users

---

## Exercise 8.2: Enhanced Error Handling

### Objective
Create comprehensive error handling system with user-friendly error messages.

### Prerequisites
- [x] Completed Exercise 8.1
- [x] Understanding of error types

### Instructions

#### Step 1: Create Error Utilities
1. Create `apps/frontend/src/utils/errors.ts`:
   ```typescript
   export function getErrorMessage(error: unknown): string {
     if (error instanceof Error) {
       // Map common contract errors to user-friendly messages
       if (error.message.includes('user rejected')) {
         return 'Transaction was cancelled. Please try again.';
       }
       if (error.message.includes('insufficient funds')) {
         return 'Insufficient funds for transaction. Please add more ETH.';
       }
       if (error.message.includes('network')) {
         return 'Network error. Please check your connection.';
       }
       if (error.message.includes('revert')) {
         return 'Transaction failed. Please check the contract state.';
       }
       return error.message;
     }
     return 'An unknown error occurred';
   }

   export function isUserRejection(error: unknown): boolean {
     if (error instanceof Error) {
       return error.message.includes('user rejected') || 
              error.message.includes('User rejected');
     }
     return false;
   }
   ```

#### Step 2: Create Error Display Component
1. Create `apps/frontend/src/components/ErrorDisplay.tsx`:
   ```typescript
   import { getErrorMessage, isUserRejection } from '@/utils/errors';

   interface ErrorDisplayProps {
     error: unknown;
     onDismiss?: () => void;
   }

   export function ErrorDisplay({ error, onDismiss }: ErrorDisplayProps) {
     if (!error) return null;

     const message = getErrorMessage(error);
     const isRejection = isUserRejection(error);

     return (
       <div className={`p-4 rounded-lg ${
         isRejection ? 'bg-yellow-50 border-yellow-200' : 'bg-red-50 border-red-200'
       } border`}>
         <div className="flex justify-between items-start">
           <div>
             <h4 className="font-semibold mb-1">
               {isRejection ? 'Transaction Cancelled' : 'Error'}
             </h4>
             <p className="text-sm">{message}</p>
           </div>
           {onDismiss && (
             <button
               onClick={onDismiss}
               className="text-gray-500 hover:text-gray-700"
             >
               ×
             </button>
           )}
         </div>
       </div>
     );
   }
   ```

**Expected Outcome:**
- Error utilities created
- User-friendly error messages
- Error display component created

**Checkpoint:**
- [ ] Error utilities created
- [ ] Error messages mapped to user-friendly text
- [ ] Error display component created
- [ ] Handles different error types

### Submission Checklist

- [ ] Error utilities created
- [ ] Error display component created
- [ ] User-friendly error messages
- [ ] Error handling integrated

---

## Exercise 8.3: Loading States and Skeletons

### Objective
Implement loading skeletons and smooth loading states for better UX.

### Prerequisites
- [x] Completed Exercise 8.2
- [x] Understanding of loading states

### Instructions

#### Step 1: Create Skeleton Components
1. Create `apps/frontend/src/components/ui/Skeleton.tsx`:
   ```typescript
   interface SkeletonProps {
     className?: string;
   }

   export function Skeleton({ className = '' }: SkeletonProps) {
     return (
       <div className={`animate-pulse bg-gray-200 rounded ${className}`} />
     );
   }

   export function SkeletonText({ lines = 1 }: { lines?: number }) {
     return (
       <div className="space-y-2">
         {Array.from({ length: lines }).map((_, i) => (
           <Skeleton 
             key={i} 
             className={`h-4 ${i === lines - 1 ? 'w-3/4' : 'w-full'}`} 
           />
         ))}
       </div>
     );
   }
   ```

#### Step 2: Use Skeletons in Components
1. Update read components to use skeletons:
   ```typescript
   import { Skeleton, SkeletonText } from './ui/Skeleton';

   export function ReadContract({ contractAddress, abi }) {
     const { data, isLoading } = useReadContract({...});

     if (isLoading) {
       return (
         <div className="p-4 border rounded-lg space-y-2">
           <Skeleton className="h-6 w-32" />
           <SkeletonText lines={2} />
         </div>
       );
     }

     return (
       <div>
         {/* Actual content */}
       </div>
     );
   }
   ```

**Expected Outcome:**
- Skeleton components created
- Loading states use skeletons
- Smooth loading experience

**Checkpoint:**
- [ ] Skeleton components created
- [ ] Used in loading states
- [ ] Smooth loading experience

### Submission Checklist

- [ ] Skeleton components created
- [ ] Used in components
- [ ] Loading states improved

---

## Exercise 8.4: Responsive Design

### Objective
Ensure the DApp works well on mobile and desktop devices.

### Prerequisites
- [x] Completed Exercise 8.3
- [x] Tailwind CSS configured

### Instructions

#### Step 1: Make Components Responsive
1. Update components to be mobile-friendly:
   ```typescript
   // Use Tailwind responsive classes
   <div className="container mx-auto px-4 sm:px-6 lg:px-8">
     <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
       {/* Content */}
     </div>
   </div>
   ```

#### Step 2: Test on Mobile
1. Test wallet connection on mobile
2. Ensure transaction overlays work on mobile
3. Verify touch interactions work

**Expected Outcome:**
- Components are responsive
- Mobile-friendly layout
- Touch interactions work

**Checkpoint:**
- [ ] Components use responsive classes
- [ ] Mobile layout tested
- [ ] Touch interactions work

### Submission Checklist

- [ ] Components are responsive
- [ ] Mobile layout tested
- [ ] Works on different screen sizes

---

## Review Questions

1. Why is connection health monitoring important?
2. How do you map technical errors to user-friendly messages?
3. What are the benefits of loading skeletons?
4. Why is responsive design important for DApps?
5. How do you ensure good UX during transactions?

---

## Next Steps

After completing this module, you should:
- Understand connection health monitoring
- Be able to handle errors gracefully
- Know how to implement loading states
- Create responsive designs
- Follow web3 UX best practices
- Proceed to Module 9: Advanced Topics & Production Readiness
