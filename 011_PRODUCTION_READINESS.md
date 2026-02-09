# Advanced Topics & Production Readiness

> **📖 Important**: This module follows the setup guides in `dapp_setup_guides/`. For the complete, authoritative setup instructions, refer to:
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Step-by-step setup guide
>
> The setup guides are the **single source of truth** for scaffolding and architecture decisions.

## Exercise 9.1: Build Versioning

### Objective
Implement build versioning system for debugging and support.

### Prerequisites
- [x] Completed User Experience & Best Practices
- [x] Vite configured

### Instructions

#### Step 1: Update Vite Config
1. Update `apps/frontend/vite.config.ts`:
   ```typescript
   import { defineConfig } from 'vite';
   import react from '@vitejs/plugin-react';
   import { readFileSync, writeFileSync, existsSync } from 'fs';
   import { join } from 'path';

   function getBuildNumber(): number {
     const counterFile = join(process.cwd(), '.build-counter');
     
     try {
       if (existsSync(counterFile)) {
         const current = parseInt(readFileSync(counterFile, 'utf8')) || 0;
         const next = current + 1;
         writeFileSync(counterFile, next.toString());
         return next;
       } else {
         writeFileSync(counterFile, '1');
         return 1;
       }
     } catch (error) {
       console.warn('Could not read/write build counter, using timestamp fallback');
       return Math.floor(Date.now() / 1000);
     }
   }

   const buildNumber = getBuildNumber();
   console.log(`Build #${buildNumber}`);

   export default defineConfig({
     plugins: [react()],
     define: {
       __BUILD_ID__: JSON.stringify(`build-${buildNumber}`),
     },
     // ... rest of config
   });
   ```

#### Step 2: Display Build ID
1. Create `apps/frontend/src/components/BuildInfo.tsx`:
   ```typescript
   export function BuildInfo() {
     const buildId = (window as any).__BUILD_ID__ || 'unknown';

     if (import.meta.env.DEV) {
       return (
         <div className="fixed bottom-2 right-2 text-xs text-gray-500 bg-white px-2 py-1 rounded border">
           Build: {buildId}
         </div>
       );
     }

     return null;
   }
   ```

2. Add to App.tsx (dev only)

**Expected Outcome:**
- Build counter system implemented
- Build ID displayed in dev mode
- Available for debugging

**Checkpoint:**
- [ ] Build counter implemented
- [ ] Build ID injected at build time
- [ ] Build ID displayed (dev mode)

### Submission Checklist

- [ ] Build versioning implemented
- [ ] Build ID displayed
- [ ] Counter increments on each build

---

## Exercise 9.2: Testing Strategy

### Objective
Set up testing for both frontend and smart contracts.

### Prerequisites
- [x] Completed Exercise 9.1
- [x] Understanding of testing concepts

### Instructions

#### Step 1: Set Up Vitest for Frontend
1. Install Vitest:
   ```bash
   cd apps/frontend
   pnpm add -D vitest @testing-library/react @testing-library/jest-dom
   ```

2. Create `apps/frontend/vitest.config.ts`:
   ```typescript
   import { defineConfig } from 'vitest/config';
   import react from '@vitejs/plugin-react';

   export default defineConfig({
     plugins: [react()],
     test: {
       environment: 'jsdom',
       setupFiles: ['./src/test/setup.ts'],
     },
   });
   ```

3. Create test setup file:
   ```typescript
   // apps/frontend/src/test/setup.ts
   import '@testing-library/jest-dom';
   ```

#### Step 2: Write Frontend Tests
1. Create `apps/frontend/src/components/__tests__/WalletConnect.test.tsx`:
   ```typescript
   import { describe, it, expect } from 'vitest';
   import { render, screen } from '@testing-library/react';
   import { WalletConnect } from '../WalletConnect';

   describe('WalletConnect', () => {
     it('renders connect button', () => {
       render(<WalletConnect />);
       expect(screen.getByText(/connect/i)).toBeInTheDocument();
     });
   });
   ```

#### Step 3: Verify Contract Tests
1. Ensure contract tests from Smart Contract Development still pass:
   ```bash
   cd packages/contracts
   pnpm test
   ```

2. Add more advanced contract tests as needed (e.g., for upgradeable contracts, edge cases)

**Expected Outcome:**
- Testing framework set up
- Frontend tests written
- Contract tests verified

**Checkpoint:**
- [ ] Vitest configured
- [ ] Frontend tests written
- [ ] Contract tests passing
- [ ] Tests pass

### Submission Checklist

- [ ] Testing framework set up
- [ ] Frontend tests written
- [ ] Contract tests verified
- [ ] Tests run successfully

---

## Exercise 9.3: Production Build and Deployment

### Objective
Build the DApp for production and prepare for static hosting deployment.

### Prerequisites
- [x] Completed Exercise 9.2
- [x] All features implemented

### Instructions

#### Step 1: Configure Production Build
1. Ensure `apps/frontend/vite.config.ts` has manual chunks:
   ```typescript
   build: {
     rollupOptions: {
       output: {
         manualChunks: {
           vendor: ['react', 'react-dom'],
           wagmi: ['wagmi', '@tanstack/react-query'],
           rainbowkit: ['@rainbow-me/rainbowkit'],
         },
       },
     },
   },
   ```

#### Step 2: Build for Production
1. Build the frontend:
   ```bash
   cd apps/frontend
   pnpm build
   ```

2. Verify build output:
   ```bash
   ls -la dist/
   ```

3. Check that `_redirects` file is in `dist/`:
   ```bash
   ls dist/_redirects
   ```

#### Step 3: Test Production Build
1. Preview production build:
   ```bash
   pnpm preview
   ```

2. Test:
   - Wallet connection
   - Contract interactions
   - Routing (refresh on deep routes)
   - All features work

**Expected Outcome:**
- Production build created
- Manual chunks configured
- `_redirects` file included
- Build tested locally

**Checkpoint:**
- [ ] Production build successful
- [ ] Manual chunks created
- [ ] `_redirects` file in dist
- [ ] Build tested

### Submission Checklist

- [ ] Production build configured
- [ ] Build successful
- [ ] Manual chunks created
- [ ] `_redirects` file included
- [ ] Production build tested

---

## Review Questions

1. What is the purpose of build versioning?
2. Why are manual chunks important in production builds?
3. What should be tested before deploying to production?
4. Why is the `_redirects` file needed for static hosting?

---

## Next Steps

After completing this module, you should:
- Know how to implement build versioning
- Set up testing strategies
- Build and deploy to production
- Have a complete, production-ready DApp!

---

## Congratulations!

You've completed the full DApp development learning path! You now have the skills to:
- Set up a complete DApp development environment
- Write, test, and deploy smart contracts
- Build production-ready DApps
- Handle complex blockchain interactions
- Create excellent user experiences
- Deploy and maintain DApps

Continue building and learning!
