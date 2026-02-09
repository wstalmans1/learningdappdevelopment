# Module 8: Advanced Topics & Production Readiness

> **📖 Important**: This module follows the setup guides in `dapp_setup_guides/`. For the complete, authoritative setup instructions, refer to:
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Step-by-step setup guide
>
> The setup guides are the **single source of truth** for scaffolding and architecture decisions.

## Exercise 8.1: Contract Verification (Sourcify + Blockscout)

### Objective
Verify smart contracts on Sourcify first, then Blockscout, without using Etherscan API keys.

### Prerequisites
- [x] Completed Module 7
- [x] Contract deployed to Sepolia
- [x] Hardhat configured

### Instructions

#### Step 1: Verify on Sourcify
1. Install Sourcify plugin (optional, can use API directly):
   ```bash
   cd packages/contracts
   pnpm add -D @sourcify-dev/hardhat-sourcify
   ```

2. Update `hardhat.config.cjs`:
   ```javascript
   require("@sourcify-dev/hardhat-sourcify");

   module.exports = {
     // ... existing config
     sourcify: {
       enabled: true,
       apiUrl: "https://sourcify.dev/server",
       browserUrl: "https://sourcify.dev",
     },
   };
   ```

3. Verify contract:
   ```bash
   pnpm exec hardhat sourcify --network sepolia
   ```

#### Step 2: Verify on Blockscout
1. Update `hardhat.config.cjs`:
   ```javascript
   module.exports = {
     // ... existing config
     networks: {
       sepolia: {
         url: process.env.SEPOLIA_URL,
         accounts: [process.env.PRIVATE_KEY],
         verify: {
           apiUrl: 'https://eth-sepolia.blockscout.com/api',
           apiKey: 'NO_API_KEY_NEEDED', // Blockscout doesn't require API key
           browserUrl: 'https://eth-sepolia.blockscout.com',
         },
       },
     },
   };
   ```

2. Verify contract:
   ```bash
   pnpm exec hardhat verify --network sepolia <CONTRACT_ADDRESS>
   ```

**Expected Outcome:**
- Contract verified on Sourcify
- Contract verified on Blockscout
- No API keys required

**Checkpoint:**
- [ ] Sourcify verification successful
- [ ] Blockscout verification successful
- [ ] Contract source visible on explorers

> **📖 Reference**: See `dapp_setup_guides/001_scaffolding_specs.md` section 7 for verification details.

### Submission Checklist

- [ ] Contract verified on Sourcify
- [ ] Contract verified on Blockscout
- [ ] No API keys used

---

## Exercise 8.2: Build Versioning

### Objective
Implement build versioning system for debugging and support.

### Prerequisites
- [x] Completed Exercise 8.1
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
       console.warn('⚠️ Could not read/write build counter, using timestamp fallback');
       return Math.floor(Date.now() / 1000);
     }
   }

   const buildNumber = getBuildNumber();
   console.log(`🚀 Build #${buildNumber}`);

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

## Exercise 8.3: Security Best Practices

### Objective
Implement security best practices for DApp development.

### Prerequisites
- [x] Completed Exercise 8.2
- [x] Understanding of common vulnerabilities

### Instructions

#### Step 1: Input Validation
1. Create `apps/frontend/src/utils/validation.ts`:
   ```typescript
   import { isAddress } from 'viem';

   export function validateAddress(address: string): boolean {
     return isAddress(address);
   }

   export function validateAmount(amount: string, maxDecimals: number = 18): boolean {
     const regex = new RegExp(`^\\d+(\\.\\d{1,${maxDecimals}})?$`);
     return regex.test(amount) && parseFloat(amount) > 0;
   }

   export function sanitizeInput(input: string): string {
     return input.trim().slice(0, 1000); // Limit length
   }
   ```

#### Step 2: Rate Limiting (Frontend)
1. Create `apps/frontend/src/utils/rateLimit.ts`:
   ```typescript
   const requestTimestamps: number[] = [];
   const MAX_REQUESTS = 10;
   const WINDOW_MS = 60 * 1000; // 1 minute

   export function checkRateLimit(): boolean {
     const now = Date.now();
     const recentRequests = requestTimestamps.filter(
       (timestamp) => now - timestamp < WINDOW_MS
     );

     if (recentRequests.length >= MAX_REQUESTS) {
       return false; // Rate limit exceeded
     }

     requestTimestamps.push(now);
     return true;
   }
   ```

#### Step 3: Secure Environment Variables
1. Ensure `.env.local` is in `.gitignore`
2. Never commit private keys or API keys
3. Use environment variables for all secrets

**Expected Outcome:**
- Input validation implemented
- Rate limiting implemented
- Environment variables secured

**Checkpoint:**
- [ ] Input validation utilities created
- [ ] Rate limiting implemented
- [ ] Environment variables secured
- [ ] No secrets in code

### Submission Checklist

- [ ] Input validation implemented
- [ ] Rate limiting implemented
- [ ] Environment variables secured
- [ ] No secrets committed

---

## Exercise 8.4: Testing Strategy

### Objective
Set up testing for both frontend and smart contracts.

### Prerequisites
- [x] Completed Exercise 8.3
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

#### Step 3: Write Contract Tests
1. Create `packages/contracts/test/MyToken.test.js`:
   ```javascript
   const { expect } = require("chai");
   const { ethers, upgrades } = require("hardhat");

   describe("MyToken", function () {
     it("Should deploy and initialize", async function () {
       const MyToken = await ethers.getContractFactory("MyToken");
       const myToken = await upgrades.deployProxy(MyToken, [], {
         initializer: "initialize",
       });
       await myToken.waitForDeployment();

       expect(await myToken.name()).to.equal("MyToken");
       expect(await myToken.symbol()).to.equal("MTK");
     });
   });
   ```

**Expected Outcome:**
- Testing framework set up
- Frontend tests written
- Contract tests written

**Checkpoint:**
- [ ] Vitest configured
- [ ] Frontend tests written
- [ ] Contract tests written
- [ ] Tests pass

### Submission Checklist

- [ ] Testing framework set up
- [ ] Frontend tests written
- [ ] Contract tests written
- [ ] Tests run successfully

---

## Exercise 8.5: Production Build and Deployment

### Objective
Build the DApp for production and prepare for static hosting deployment.

### Prerequisites
- [x] Completed Exercise 8.4
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

1. Why verify contracts on Sourcify first, then Blockscout?
2. What is the purpose of build versioning?
3. What security practices should you follow in DApp development?
4. Why are manual chunks important in production builds?
5. What should be tested before deploying to production?

---

## Next Steps

After completing this module, you should:
- Understand contract verification process
- Know how to implement build versioning
- Follow security best practices
- Set up testing strategies
- Build and deploy to production
- Have a complete, production-ready DApp!

---

## 🎉 Congratulations!

You've completed the full DApp development learning path! You now have the skills to:
- Set up a complete DApp development environment
- Build production-ready DApps
- Handle complex blockchain interactions
- Create excellent user experiences
- Deploy and maintain DApps

Continue building and learning! 🚀
