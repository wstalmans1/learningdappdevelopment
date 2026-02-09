# Web3 Fundamentals & Wallet Integration

> **📖 Important**: This module follows the setup guides in `dapp_setup_guides/`. For the complete, authoritative setup instructions, refer to:
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Step-by-step setup guide
>
> The setup guides are the **single source of truth** for scaffolding and architecture decisions.

## Exercise 2.1: Install Web3 Dependencies

### Objective
Install and understand the core web3 packages needed for DApp development.

### Prerequisites
- [x] Completed Environment Setup & Project Scaffolding
- [x] Monorepo structure set up with `apps/frontend/`
- [x] pnpm workspace configured

### Instructions

#### Step 1: Navigate to Frontend App
1. Open your terminal
2. Navigate to the frontend app:
   ```bash
   cd apps/frontend
   ```

**Expected Outcome:**
- You're in the `apps/frontend/` directory

**Checkpoint:**
- [ ] Current directory is `apps/frontend/`

#### Step 2: Install Core Web3 Packages
1. Install Wagmi, Viem, and RainbowKit with correct versions:
   ```bash
   pnpm add wagmi@^2.5.0 viem@^2.7.0 @rainbow-me/rainbowkit@^2.0.0
   ```

2. Install TanStack Query and Zustand:
   ```bash
   pnpm add @tanstack/react-query@^5.85.5 zustand@^4.4.7
   ```

3. Install Node.js polyfills for browser compatibility:
   ```bash
   pnpm add buffer@^6.0.3 process@^0.11.10 util@^0.12.5
   ```

4. Install TanStack Query Devtools (development only):
   ```bash
   pnpm add -D @tanstack/react-query-devtools@^5.85.5
   ```

**Expected Outcome:**
- All web3 dependencies are installed with correct versions

**Checkpoint:**
- [ ] `package.json` includes all required packages
- [ ] Versions match the setup guides (Wagmi 2.5+, Viem 2.7+, etc.)

#### Step 3: Understand Each Package
Read about what each package does:

- **Wagmi 2.x**: React hooks for Ethereum (read/write contracts, wallet state)
- **Viem 2.x**: TypeScript Ethereum library (low-level blockchain interactions)
- **RainbowKit 2.x**: Beautiful wallet connection UI components
- **TanStack Query 5.x**: Data fetching and caching for blockchain data
- **Zustand 4.x**: Lightweight state management for UI state
- **buffer/process/util**: Node.js polyfills for browser compatibility

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete dependency installation and explanations.

### Common Issues & Solutions

**Issue:** Package version conflicts
- **Solution**: Ensure you're using the exact versions specified. Check `package.json` and reinstall if needed.

**Issue:** TypeScript errors after installation
- **Solution**: Restart VS Code and ensure TypeScript server is running. Run `pnpm type-check` to verify.

### Testing Your Solution

1. Verify packages are installed:
   ```bash
   pnpm list wagmi viem @rainbow-me/rainbowkit
   ```

2. Check versions match requirements:
   ```bash
   pnpm list --depth=0
   ```

**Expected Result:**
- All packages installed successfully
- Versions match or exceed minimum requirements

### Submission Checklist

- [ ] All web3 packages installed
- [ ] Correct versions (Wagmi 2.5+, Viem 2.7+, etc.)
- [ ] Node.js polyfills installed
- [ ] TanStack Query Devtools installed (dev dependency)
- [ ] No installation errors

---

## Exercise 2.2: Configure Alchemy RPC Provider

### Objective
Set up Alchemy as the RPC provider with proper environment variable configuration.

### Prerequisites
- [x] Completed Exercise 2.1
- [x] Alchemy account created
- [x] WalletConnect Project ID obtained

### Instructions

#### Step 1: Get Alchemy API Key
1. Visit [Alchemy](https://www.alchemy.com/)
2. Sign up for a free account
3. Create a new app (select Sepolia testnet)
4. Copy your API key from the dashboard

> **📖 Reference**: See `dapp_setup_guides/bc_infrastructure_services.md` for a comprehensive comparison of blockchain infrastructure providers. This document helps understand provider options, but the setup guides specify Alchemy as the chosen provider.

#### Step 2: Get WalletConnect Project ID
1. Visit [WalletConnect Cloud](https://cloud.walletconnect.com/)
2. Sign up or log in
3. Create a new project
4. Copy your Project ID

#### Step 3: Create Environment Variables File
1. Create `apps/frontend/.env.local`:
   ```bash
   # WalletConnect Project ID
   VITE_WALLETCONNECT_PROJECT_ID=your_project_id_here
   
   # Alchemy API Key (required)
   VITE_ALCHEMY_API_KEY=your_alchemy_key_here
   
   # Chain ID (Sepolia testnet)
   VITE_APP_CHAIN_ID=11155111
   ```

2. Replace `your_project_id_here` and `your_alchemy_key_here` with your actual values

**Expected Outcome:**
- Environment variables file created with required values

**Checkpoint:**
- [ ] `.env.local` file exists in `apps/frontend/`
- [ ] All three variables are set
- [ ] Values are not empty

#### Step 4: Create Provider Configuration
1. Create `apps/frontend/src/config/providers.ts`:
   ```typescript
   /**
    * Centralized provider configuration
    * Single source of truth for all Alchemy URLs and API keys
    */

   // Normalize environment variables (handle empty strings)
   const normalize = (v: string | undefined) => (v && v.trim() !== '' ? v : undefined);

   // Get API key from environment - required for all environments
   const ALCHEMY_KEY = normalize(import.meta.env.VITE_ALCHEMY_API_KEY);

   // Fail fast if no key is provided
   if (!ALCHEMY_KEY) {
     const errorMessage = '[providers] Missing VITE_ALCHEMY_API_KEY. Please set it in your .env.local file. Get your key from https://www.alchemy.com/';
     throw new Error(errorMessage);
   }

   export const ALCHEMY_API_KEY = ALCHEMY_KEY;

   // Derive HTTP and WebSocket URLs from the API key
   export const ALCHEMY_HTTP_SEPOLIA = `https://eth-sepolia.g.alchemy.com/v2/${ALCHEMY_API_KEY}`;
   export const ALCHEMY_WS_SEPOLIA = `wss://eth-sepolia.g.alchemy.com/v2/${ALCHEMY_API_KEY}`;

   // Log configuration for debugging (dev only)
   if (import.meta.env.DEV) {
     console.log('[providers] Alchemy configuration:', {
       apiKey: ALCHEMY_API_KEY.substring(0, 10) + '...',
       httpUrl: ALCHEMY_HTTP_SEPOLIA,
       wsUrl: ALCHEMY_WS_SEPOLIA,
     });
   }
   ```

**Expected Outcome:**
- Provider configuration file created
- Centralized Alchemy URL management

**Checkpoint:**
- [ ] `src/config/providers.ts` exists
- [ ] File exports HTTP and WebSocket URLs
- [ ] Error handling for missing API key

### Common Issues & Solutions

**Issue:** `VITE_ALCHEMY_API_KEY is not defined`
- **Solution**: 
  - Ensure `.env.local` is in `apps/frontend/` directory
  - Restart the dev server after creating `.env.local`
  - Check that variable name starts with `VITE_`

**Issue:** Environment variables not loading
- **Solution**: 
  - Vite only loads variables prefixed with `VITE_`
  - Restart dev server: `pnpm dev`
  - Check `.env.local` is not in `.gitignore` (it should be ignored)

### Testing Your Solution

1. Start the dev server:
   ```bash
   pnpm dev
   ```

2. Check browser console for provider configuration log (should show masked API key)

3. Verify no errors about missing API key

**Expected Result:**
- Dev server starts without errors
- Console shows provider configuration (in dev mode)
- No missing API key errors

### Submission Checklist

- [ ] Alchemy API key obtained
- [ ] WalletConnect Project ID obtained
- [ ] `.env.local` file created with all variables
- [ ] `src/config/providers.ts` created
- [ ] Provider URLs correctly derived from API key
- [ ] Error handling for missing API key implemented

---

## Exercise 2.3: Configure Wagmi with RainbowKit

### Objective
Set up Wagmi configuration with RainbowKit, including chain configuration and transport selection.

### Prerequisites
- [x] Completed Exercise 2.2
- [x] Provider configuration created
- [x] Environment variables set

### Instructions

#### Step 1: Create Chain Configuration
1. Create `apps/frontend/src/config/chain.ts`:
   ```typescript
   import { sepolia } from 'wagmi/chains';
   import type { Chain } from 'wagmi/chains';

   const SUPPORTED_CHAINS = [sepolia] as const;
   const DEFAULT_ID = sepolia.id;

   // Get environment variable, handle undefined gracefully
   const envChainId = import.meta.env.VITE_APP_CHAIN_ID;
   const parsed = envChainId ? Number(envChainId) : NaN;
   const ENV_ID = Number.isFinite(parsed) ? parsed : DEFAULT_ID;

   export const APP_CHAIN_ID = SUPPORTED_CHAINS.some(c => c.id === ENV_ID) 
     ? ENV_ID 
     : DEFAULT_ID;
   
   export const APP_CHAIN: Chain = SUPPORTED_CHAINS.find(c => c.id === APP_CHAIN_ID)!;
   export const APP_CHAIN_NAME = APP_CHAIN.name;
   ```

**Expected Outcome:**
- Chain configuration file created
- Sepolia testnet configured as default chain

**Checkpoint:**
- [ ] `src/config/chain.ts` exists
- [ ] Exports `APP_CHAIN_ID` and `APP_CHAIN`
- [ ] Uses environment variable with fallback

#### Step 2: Create Wagmi Configuration
1. Create `apps/frontend/src/config/wagmi.ts`:
   ```typescript
   import { http, webSocket } from 'wagmi';
   import { sepolia } from 'wagmi/chains';
   import { getDefaultConfig } from '@rainbow-me/rainbowkit';
   import { ALCHEMY_HTTP_SEPOLIA, ALCHEMY_WS_SEPOLIA } from './providers';

   const projectId = import.meta.env.VITE_WALLETCONNECT_PROJECT_ID || 'fallback_id';

   if (!projectId || projectId === 'fallback_id') {
     console.warn('VITE_WALLETCONNECT_PROJECT_ID not found, using fallback');
   }

   // Mobile detection for performance tuning
   const isMobile = typeof window !== 'undefined' && 
     /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);

   // Build transport with Alchemy (SEPOLIA ONLY)
   function buildSepoliaTransport() {
     console.log('🚀 Using Alchemy Sepolia as RPC provider');
     console.log('🔌 WebSocket URL:', ALCHEMY_WS_SEPOLIA);
     console.log('🌐 HTTP URL:', ALCHEMY_HTTP_SEPOLIA);
     console.log('📱 Mobile device:', isMobile ? 'Yes' : 'No');
     
     if (isMobile) {
       // Mobile: HTTP for reliable polling
       console.log('📱 Mobile detected: HTTP transport for reliable polling');
       return http(ALCHEMY_HTTP_SEPOLIA);
     } else {
       // Desktop: WebSocket ONLY - NO HTTP to prevent eth_getFilterChanges traffic
       console.log('🖥️ Desktop detected: WebSocket-ONLY transport (NO HTTP)');
       return webSocket(ALCHEMY_WS_SEPOLIA);
     }
   }

   // Use RainbowKit's default config with SEPOLIA ONLY transports
   export const config = getDefaultConfig({
     appName: 'DApp Learning',
     projectId,
     chains: [sepolia], // SEPOLIA ONLY
     transports: {
       [sepolia.id]: buildSepoliaTransport(),
     },
     ssr: false, // Client-side routing only (React Router DOM)
   });
   ```

**Expected Outcome:**
- Wagmi configuration created
- Mobile/desktop transport selection implemented
- WebSocket for desktop, HTTP for mobile

**Checkpoint:**
- [ ] `src/config/wagmi.ts` exists
- [ ] Uses `getDefaultConfig` from RainbowKit
- [ ] Mobile detection implemented
- [ ] Transport selection based on device type
- [ ] `ssr: false` for client-side only

#### Step 3: Set Up Providers in main.tsx
1. Update `apps/frontend/src/main.tsx`:
   ```typescript
   // Polyfill Node.js modules for browser compatibility
   import { Buffer } from 'buffer';
   import * as util from 'util';

   // Set global polyfills
   if (typeof window !== 'undefined') {
     (window as any).Buffer = Buffer;
     (window as any).util = util;
     (window as any).global = window;
   }

   import ReactDOM from 'react-dom/client';
   import { WagmiProvider } from 'wagmi';
   import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
   import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
   import { RainbowKitProvider } from '@rainbow-me/rainbowkit';
   import { BrowserRouter } from 'react-router-dom';
   import { config } from './config/wagmi';
   import App from './App';
   import './index.css';
   import '@rainbow-me/rainbowkit/styles.css';

   // Create QueryClient with anti-flicker configuration
   const queryClient = new QueryClient({
     defaultOptions: {
       queries: {
         // Prevent UI flicker by keeping previous data while refetching
         placeholderData: (previousData: any) => previousData,
         // Don't refetch on window focus to prevent unnecessary updates
         refetchOnWindowFocus: false,
         // Fail faster to prevent long loading states
         retry: 1,
         // Keep data fresh for 30 seconds to prevent redundant requests
         staleTime: 30_000,
         // Keep unused data in cache for 5 minutes
         gcTime: 5 * 60 * 1000,
       },
     },
   });

   ReactDOM.createRoot(document.getElementById('root')!).render(
     <WagmiProvider config={config}>
       <QueryClientProvider client={queryClient}>
         <RainbowKitProvider>
           <BrowserRouter>
             <App />
           </BrowserRouter>
           <ReactQueryDevtools initialIsOpen={false} />
         </RainbowKitProvider>
       </QueryClientProvider>
     </WagmiProvider>
   );
   ```

**Expected Outcome:**
- App wrapped with all required providers
- Provider order: BrowserRouter → WagmiProvider → QueryClientProvider → RainbowKitProvider
- TanStack Query configured to prevent UI flicker

**Checkpoint:**
- [ ] All providers imported
- [ ] Correct provider nesting order
- [ ] Polyfills set up
- [ ] TanStack Query configured with anti-flicker settings
- [ ] React Query Devtools included (dev only)

### Common Issues & Solutions

**Issue:** `getDefaultConfig is not a function`
- **Solution**: Ensure you're using RainbowKit 2.x. Check version: `pnpm list @rainbow-me/rainbowkit`

**Issue:** WebSocket connection fails
- **Solution**: 
  - Verify Alchemy API key is correct
  - Check WebSocket URL format
  - Ensure `.env.local` is loaded (restart dev server)

**Issue:** Provider order errors
- **Solution**: Ensure BrowserRouter wraps everything, then WagmiProvider, then QueryClientProvider, then RainbowKitProvider

### Testing Your Solution

1. Start the dev server:
   ```bash
   pnpm dev
   ```

2. Check browser console for:
   - Provider configuration logs
   - No WebSocket connection errors
   - Transport type (WebSocket or HTTP) based on device

3. Verify the app loads without errors

**Expected Result:**
- Dev server starts successfully
- Console shows provider configuration
- No connection errors
- App renders without errors

### Submission Checklist

- [ ] Chain configuration created (`src/config/chain.ts`)
- [ ] Wagmi configuration created (`src/config/wagmi.ts`)
- [ ] Mobile/desktop transport selection implemented
- [ ] Providers set up in `main.tsx` with correct order
- [ ] Node.js polyfills configured
- [ ] TanStack Query configured with anti-flicker settings
- [ ] React Query Devtools included
- [ ] No console errors

---

## Exercise 2.4: Create Wallet Connection Component

### Objective
Create a wallet connection component using RainbowKit and display wallet information.

### Prerequisites
- [x] Completed Exercise 2.3
- [x] Wagmi and RainbowKit configured
- [x] MetaMask or other wallet installed in browser

### Instructions

#### Step 1: Create Wallet Connect Component
1. Create `apps/frontend/src/components/WalletConnect.tsx`:
   ```typescript
   import { ConnectButton } from '@rainbow-me/rainbowkit';

   export function WalletConnect() {
     return <ConnectButton />;
   }
   ```

**Expected Outcome:**
- Basic wallet connection button component created

**Checkpoint:**
- [ ] Component file created
- [ ] Uses RainbowKit's `ConnectButton`

#### Step 2: Create Wallet Info Component
1. Create `apps/frontend/src/components/WalletInfo.tsx`:
   ```typescript
   import { useAccount, useBalance, useChainId } from 'wagmi';

   export function WalletInfo() {
     const { address, isConnected } = useAccount();
     const chainId = useChainId();
     const { data: balance, isLoading: balanceLoading } = useBalance({ 
       address 
     });

     if (!isConnected) {
       return (
         <div className="p-4 border rounded-lg">
           <p className="text-gray-600">Wallet not connected</p>
         </div>
       );
     }

     return (
       <div className="p-4 border rounded-lg space-y-2">
         <h3 className="font-bold">Wallet Information</h3>
         <p><strong>Address:</strong> {address}</p>
         <p><strong>Chain ID:</strong> {chainId}</p>
         {balanceLoading ? (
           <p>Loading balance...</p>
         ) : (
           <p>
             <strong>Balance:</strong> {balance?.formatted} {balance?.symbol}
           </p>
         )}
       </div>
     );
   }
   ```

**Expected Outcome:**
- Wallet information component created
- Displays address, chain ID, and balance

**Checkpoint:**
- [ ] Component uses Wagmi hooks (`useAccount`, `useBalance`, `useChainId`)
- [ ] Handles disconnected state
- [ ] Shows loading state for balance
- [ ] Displays wallet information when connected

#### Step 3: Integrate Components in App
1. Update `apps/frontend/src/App.tsx`:
   ```typescript
   import { Routes, Route } from 'react-router-dom';
   import { WalletConnect } from './components/WalletConnect';
   import { WalletInfo } from './components/WalletInfo';
   import Home from './pages/Home';

   function App() {
     return (
       <div className="min-h-screen bg-gray-50">
         <header className="bg-white shadow-sm p-4">
           <div className="container mx-auto flex justify-between items-center">
             <h1 className="text-xl font-bold">DApp Learning</h1>
             <WalletConnect />
           </div>
         </header>
         
         <main className="container mx-auto p-4">
           <Routes>
             <Route path="/" element={<Home />} />
           </Routes>
           
           <div className="mt-8">
             <WalletInfo />
           </div>
         </main>
       </div>
     );
   }

   export default App;
   ```

**Expected Outcome:**
- Wallet connection button in header
- Wallet info displayed on home page
- Clean, organized layout

**Checkpoint:**
- [ ] Components imported and used
- [ ] WalletConnect in header
- [ ] WalletInfo displayed on page
- [ ] Layout is clean and organized

### Common Issues & Solutions

**Issue:** ConnectButton not showing
- **Solution**: 
  - Ensure RainbowKitProvider wraps the app
  - Check that `@rainbow-me/rainbowkit/styles.css` is imported
  - Verify WagmiProvider is configured correctly

**Issue:** Balance shows as undefined
- **Solution**: 
  - Ensure wallet is connected
  - Check that `address` is defined before calling `useBalance`
  - Verify RPC connection is working

**Issue:** Wallet not connecting
- **Solution**: 
  - Check WalletConnect Project ID is set correctly
  - Ensure wallet extension is installed
  - Check browser console for errors

### Testing Your Solution

1. Start the dev server:
   ```bash
   pnpm dev
   ```

2. Test wallet connection:
   - Click "Connect Wallet" button
   - Select a wallet (MetaMask, WalletConnect, etc.)
   - Approve connection in wallet

3. Verify wallet information displays:
   - Address is shown
   - Chain ID is correct (11155111 for Sepolia)
   - Balance loads and displays

4. Test disconnect:
   - Click wallet button
   - Select "Disconnect"
   - Verify wallet info disappears

**Expected Result:**
- Connect button works
- Wallet connects successfully
- Wallet information displays correctly
- Disconnect works
- No console errors

### Submission Checklist

- [ ] WalletConnect component created
- [ ] WalletInfo component created
- [ ] Components integrated in App
- [ ] Can connect wallet successfully
- [ ] Wallet information displays correctly
- [ ] Can disconnect wallet
- [ ] No console errors
- [ ] UI is clean and organized

---

## Exercise 2.5: First Taste of Viem Utilities

### Objective
Get a first practical taste of Viem by creating helper functions you'll use throughout the project. A deeper dive into the web3 library landscape and the full Viem/Wagmi API comes in Reading Blockchain Data, once you have real contracts to work with.

### Prerequisites
- [x] Completed Exercise 2.4
- [x] Wallet connection working

### Instructions

#### Step 1: Understand the Big Picture (Brief)
You installed three core libraries in Exercise 2.1. Here's how they relate:

- **Viem** -- the low-level TypeScript Ethereum library. Handles raw blockchain communication, data encoding, and provides utility functions.
- **Wagmi** -- a React hooks layer built *on top of* Viem. Every Wagmi hook uses Viem under the hood, but adds React state management, caching, and automatic re-renders.
- **RainbowKit** -- a UI layer built *on top of* Wagmi. Provides the wallet connection modal and buttons.

```
RainbowKit  (UI components)
    ↓ uses
  Wagmi      (React hooks + caching)
    ↓ uses
  Viem       (low-level Ethereum library)
    ↓ talks to
  Blockchain (via RPC provider)
```

> You'll also encounter **ethers.js** in Smart Contract Development & Deployment when you set up Hardhat. Hardhat's tooling is built on ethers.js, while the frontend uses Viem/Wagmi. Reading Blockchain Data explains this split in detail and covers the full Viem/Wagmi API. For now, just know the names.

#### Step 2: Create Viem Helper Functions
Viem provides pure utility functions that work anywhere -- React components, scripts, tests, or plain TypeScript files. Create a helpers file you'll reuse throughout the project.

1. Create `apps/frontend/src/utils/viemHelpers.ts`:
   ```typescript
   import { formatUnits, parseUnits, isAddress, formatEther, parseEther } from 'viem';

   // Format wei to ether (human-readable)
   export function formatWeiToEther(wei: bigint): string {
     return formatEther(wei);
   }

   // Parse ether string to wei
   export function parseEtherToWei(ether: string): bigint {
     return parseEther(ether);
   }

   // Format token amount with decimals
   export function formatTokenAmount(amount: bigint, decimals: number): string {
     return formatUnits(amount, decimals);
   }

   // Parse token amount string to bigint
   export function parseTokenAmount(amount: string, decimals: number): bigint {
     return parseUnits(amount, decimals);
   }

   // Validate Ethereum address
   export function validateAddress(address: string): boolean {
     return isAddress(address);
   }

   // Format address (shorten for display)
   export function formatAddress(address: string, chars = 4): string {
     if (!isAddress(address)) return address;
     return `${address.slice(0, chars + 2)}...${address.slice(-chars)}`;
   }
   ```

#### Step 3: Use Helpers in WalletInfo
1. Update `apps/frontend/src/components/WalletInfo.tsx` to use the formatter:
   ```typescript
   import { useAccount, useBalance, useChainId } from 'wagmi';
   import { formatAddress } from '@/utils/viemHelpers';

   export function WalletInfo() {
     const { address, isConnected } = useAccount();
     const chainId = useChainId();
     const { data: balance, isLoading: balanceLoading } = useBalance({ address });

     if (!isConnected) {
       return (
         <div className="p-4 border rounded-lg">
           <p className="text-gray-600">Wallet not connected</p>
         </div>
       );
     }

     return (
       <div className="p-4 border rounded-lg space-y-2">
         <h3 className="font-bold">Wallet Information</h3>
         <p><strong>Address:</strong> {formatAddress(address!)}</p>
         <p><strong>Full Address:</strong> <span className="text-xs">{address}</span></p>
         <p><strong>Chain ID:</strong> {chainId}</p>
         {balanceLoading ? (
           <p>Loading balance...</p>
         ) : (
           <p><strong>Balance:</strong> {balance?.formatted} {balance?.symbol}</p>
         )}
       </div>
     );
   }
   ```

**Expected Outcome:**
- Viem helper functions ready for reuse
- WalletInfo displays shortened address using Viem

**Checkpoint:**
- [ ] `src/utils/viemHelpers.ts` created
- [ ] WalletInfo uses `formatAddress`
- [ ] Understands that Viem utilities are pure functions, not React hooks

### Submission Checklist

- [ ] Viem helper functions file created
- [ ] `formatAddress` used in WalletInfo component
- [ ] Understands the Viem → Wagmi → RainbowKit stack

---

## Review Questions

1. What is the relationship between Viem, Wagmi, and RainbowKit?
2. Why do we use WebSocket transport on desktop but HTTP on mobile?
3. What is the purpose of TanStack Query in a DApp?
4. Why do we need Node.js polyfills (buffer, process, util) in the browser?
5. What is the correct order for wrapping providers in React?
6. Name two Viem utility functions and explain what they do.

---

## Next Steps

After completing this module, you should:
- Understand how to configure Wagmi and RainbowKit
- Know how to set up RPC providers (Alchemy)
- Be able to connect and disconnect wallets
- Display wallet information (address, balance, chain)
- Understand provider configuration and transport selection
- Have a first practical taste of Viem utility functions
- Proceed to Smart Contract Development & Deployment
