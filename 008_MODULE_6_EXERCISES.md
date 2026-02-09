# Module 6: State Management & Data Fetching

> **📖 Important**: This module follows the setup guides in `dapp_setup_guides/`. For the complete, authoritative setup instructions, refer to:
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Step-by-step setup guide
>
> The setup guides are the **single source of truth** for scaffolding and architecture decisions.

## Exercise 6.1: Global State Management with Zustand 4

### Objective
Set up Zustand 4 for UI state management, separate from blockchain data.

### Prerequisites
- [x] Completed Module 5
- [x] Zustand installed
- [x] Understanding of state management

### Instructions

#### Step 1: Create UI Store
1. Create `apps/frontend/src/stores/uiStore.ts`:
   ```typescript
   import { create } from 'zustand';
   import { persist } from 'zustand/middleware';

   interface UIState {
     theme: 'light' | 'dark';
     setTheme: (theme: 'light' | 'dark') => void;
     sidebarOpen: boolean;
     setSidebarOpen: (open: boolean) => void;
   }

   export const useUIStore = create<UIState>()(
     persist(
       (set) => ({
         theme: 'light',
         setTheme: (theme) => set({ theme }),
         sidebarOpen: false,
         setSidebarOpen: (open) => set({ sidebarOpen: open }),
       }),
       { 
         name: 'ui-storage',
         partialize: (state) => ({ theme: state.theme }), // Only persist theme
       }
     )
   );
   ```

**Expected Outcome:**
- UI store created with Zustand
- Theme and sidebar state managed
- Theme persisted to localStorage

**Checkpoint:**
- [ ] UI store created
- [ ] Uses Zustand 4
- [ ] Persist middleware configured
- [ ] Only UI state, not blockchain data

### Common Issues & Solutions

**Issue:** Store not persisting
- **Solution**: Ensure `persist` middleware is properly configured and `name` is set

**Issue:** Using Zustand for blockchain data
- **Solution**: Remember: Zustand is for UI state only. Blockchain data comes from Wagmi + TanStack Query.

### Submission Checklist

- [ ] UI store created
- [ ] Zustand 4 configured
- [ ] Persist middleware used
- [ ] Only UI state stored

---

## Exercise 6.2: Advanced Scopes Pattern

### Objective
Expand the scopes pattern for complex data relationships and targeted cache invalidation.

### Prerequisites
- [x] Completed Exercise 6.1
- [x] Understanding of scopes pattern from Module 3

### Instructions

#### Step 1: Expand Scopes File
1. Update `apps/frontend/src/lib/scopes.ts`:
   ```typescript
   import { QueryClient } from '@tanstack/react-query';

   export const scopes = {
     // Contract scopes
     storedValue: (chainId: number, address: string) =>
       ['contract', 'storedValue', chainId, address.toLowerCase()] as const,
     
     // Organization scopes (for pagination)
     orgPage: (chainId: number, page: number, size: number) =>
       ['orgs', 'page', chainId, page, size] as const,
     
     orgData: (chainId: number, address: string) =>
       ['orgs', 'data', chainId, address.toLowerCase()] as const,
     
     orgCount: (chainId: number) =>
       ['orgs', 'count', chainId] as const,
     
     // Umbrella keys for group invalidation
     orgPageAll: (chainId: number) =>
       ['orgs', 'page', chainId] as const,
   } as const;

   // Debounced invalidation to prevent flicker
   const pendingInvalidations = new Set<string>();
   let invalidationTimeout: number | null = null;

   export function invalidateByScope(
     qc: QueryClient,
     scope: string | readonly unknown[]
   ) {
     const scopeKey = Array.isArray(scope) 
       ? scope.map(String).join('|') 
       : scope;
     
     pendingInvalidations.add(scopeKey as string);
     
     if (invalidationTimeout) {
       clearTimeout(invalidationTimeout);
     }
     
     invalidationTimeout = window.setTimeout(() => {
       const scopesToInvalidate = Array.from(pendingInvalidations);
       pendingInvalidations.clear();
       
       scopesToInvalidate.forEach(scopeStr => {
         if (scopeStr.includes('|')) {
           const keyParts = scopeStr.split('|');
           qc.invalidateQueries({ 
             queryKey: keyParts,
             exact: true 
           });
         } else {
           qc.invalidateQueries({
             predicate: (q) => {
               const k = q.queryKey as unknown[];
               return Array.isArray(k) && k.some((el) => el === scopeStr);
             },
           });
         }
       });
     }, 50); // 50ms debounce
   }

   // Precise invalidation helpers
   export function invalidateOrgData(
     qc: QueryClient,
     chainId: number,
     orgAddress: string
   ) {
     qc.invalidateQueries({ 
       queryKey: scopes.orgData(chainId, orgAddress),
       exact: true 
     });
   }

   export function invalidateOrgCount(qc: QueryClient, chainId: number) {
     qc.invalidateQueries({ 
       queryKey: scopes.orgCount(chainId),
       exact: true 
     });
   }

   export function invalidateAllOrgPages(qc: QueryClient, chainId: number) {
     qc.invalidateQueries({ 
       queryKey: scopes.orgPageAll(chainId),
       exact: false 
     });
   }
   ```

**Expected Outcome:**
- Advanced scopes pattern implemented
- Debounced invalidation to prevent flicker
- Helper functions for common invalidation patterns

**Checkpoint:**
- [ ] Scopes expanded with pagination support
- [ ] Debounced invalidation implemented
- [ ] Helper functions created
- [ ] Umbrella keys for group invalidation

### Submission Checklist

- [ ] Scopes file expanded
- [ ] Debounced invalidation implemented
- [ ] Helper functions created
- [ ] Pagination scopes added

---

## Exercise 6.3: Prevent UI Flicker with placeholderData

### Objective
Configure TanStack Query to prevent UI flicker when refetching data.

### Prerequisites
- [x] Completed Exercise 6.2
- [x] Understanding of TanStack Query configuration

### Instructions

#### Step 1: Update QueryClient Configuration
1. Update `apps/frontend/src/main.tsx`:
   ```typescript
   const queryClient = new QueryClient({
     defaultOptions: {
       queries: {
         // Keep previous data while refetching to prevent flicker
         placeholderData: (previousData: any) => previousData,
         // Don't refetch on window focus
         refetchOnWindowFocus: false,
         // Fail faster
         retry: 1,
         // Keep data fresh for 30 seconds
         staleTime: 30_000,
         // Keep unused data for 5 minutes
         gcTime: 5 * 60 * 1000,
       },
     },
   });
   ```

#### Step 2: Use placeholderData in Components
1. Update read components to use placeholderData:
   ```typescript
   const { data, isLoading } = useReadContract({
     address: contractAddress,
     abi,
     functionName: 'storedValue',
     query: {
       scopeKey: scopes.storedValue(chainId, contractAddress),
       placeholderData: (previousData) => previousData, // Keep previous data
       staleTime: 30_000,
     },
   });
   ```

**Expected Outcome:**
- QueryClient configured to prevent flicker
- Components use placeholderData
- Smooth UI updates without flicker

**Checkpoint:**
- [ ] QueryClient configured with placeholderData
- [ ] Components use placeholderData
- [ ] No UI flicker on refetch

### Submission Checklist

- [ ] QueryClient configured
- [ ] placeholderData used
- [ ] No UI flicker observed

---

## Review Questions

1. When should you use Zustand vs TanStack Query?
2. What is the purpose of debounced cache invalidation?
3. How does placeholderData prevent UI flicker?
4. What are umbrella keys used for?
5. Why is state separation important (Wagmi vs TanStack Query vs Zustand)?

---

## Next Steps

After completing this module, you should:
- Understand when to use Zustand vs TanStack Query
- Know how to implement advanced scopes patterns
- Be able to prevent UI flicker
- Understand debounced invalidation
- Proceed to Module 7: User Experience & Best Practices
