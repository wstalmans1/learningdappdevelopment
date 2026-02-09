# Quick Reference Guide

A cheat sheet for common DApp development tasks and concepts.

> **📖 Important**: This reference is based on the setup guides in `dapp_setup_guides/`. Always refer to those guides for the **single source of truth** on configuration and best practices.
>
> - **Setup Guides**: `001_scaffolding_specs.md` and `002_setup_instructions_and_best_practices.md` are the authoritative guides
> - **Reference Document**: `bc_infrastructure_services.md` provides comparison information about blockchain infrastructure providers (useful for understanding options, but setup guides specify the chosen provider)

## 🚀 Common Commands

### Project Setup (Monorepo with pnpm)
```bash
# Create monorepo root
mkdir my-dapp && cd my-dapp
pnpm init

# Create pnpm workspace
echo "packages:\n  - 'apps/*'\n  - 'packages/*'" > pnpm-workspace.yaml

# Create frontend app
pnpm create vite@latest apps/frontend --template react-ts

# Install dependencies (from frontend)
cd apps/frontend
pnpm install

# Install React Router DOM
pnpm add react-router-dom@^6.20.0

# Run development server (from root)
cd ../..
pnpm dev

# Or from frontend
cd apps/frontend
pnpm dev

# Build for production
pnpm build

# Preview production build
pnpm preview

# Type check
pnpm type-check

# Lint code
pnpm lint
```

### Git Commands
```bash
# Initialize repository
git init

# Add files
git add .

# Commit changes
git commit -m "Your message"

# Push to remote
git push origin main

# Create branch
git checkout -b feature-name
```

## 📦 Essential Packages

### Core Web3 Packages (Correct Versions)
```bash
cd apps/frontend

# Core packages
pnpm add wagmi@^2.5.0 viem@^2.7.0 @rainbow-me/rainbowkit@^2.0.0
pnpm add @tanstack/react-query@^5.85.5 zustand@^4.4.7

# Node.js polyfills
pnpm add buffer@^6.0.3 process@^0.11.10 util@^0.12.5

# Dev tools
pnpm add -D @tanstack/react-query-devtools@^5.85.5
```

### Additional Helpful Packages
```bash
# Form handling
pnpm add react-hook-form zod

# UI components
pnpm add @radix-ui/react-dialog @radix-ui/react-dropdown-menu

# Utilities
pnpm add date-fns ethers  # if needed alongside viem
```

## 🔧 Wagmi Configuration

### Provider Configuration (`apps/frontend/src/config/providers.ts`)
```typescript
const ALCHEMY_KEY = import.meta.env.VITE_ALCHEMY_API_KEY;
if (!ALCHEMY_KEY) {
  throw new Error('VITE_ALCHEMY_API_KEY is required');
}

export const ALCHEMY_HTTP_SEPOLIA = `https://eth-sepolia.g.alchemy.com/v2/${ALCHEMY_KEY}`;
export const ALCHEMY_WS_SEPOLIA = `wss://eth-sepolia.g.alchemy.com/v2/${ALCHEMY_KEY}`;
```

### Wagmi Config (`apps/frontend/src/config/wagmi.ts`)
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
  appName: 'My DApp',
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

### Provider Setup (`apps/frontend/src/main.tsx`)
```typescript
import { Buffer } from 'buffer';
import * as util from 'util';

// Polyfill Node.js modules
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

### Static Hosting Configuration (`apps/frontend/public/_redirects`)
```
/*    /index.html   200
```

This file ensures all routes are served `index.html` for client-side routing to work on static hosting (IPFS, Vercel, Netlify).

## 🔌 Wallet Connection

### Connect Button Component
```typescript
import { ConnectButton } from '@rainbow-me/rainbowkit';

export function WalletConnect() {
  return <ConnectButton />;
}
```

### Custom Wallet Info
```typescript
import { useAccount, useBalance, useDisconnect } from 'wagmi';

export function WalletInfo() {
  const { address, isConnected } = useAccount();
  const { data: balance } = useBalance({ address });
  const { disconnect } = useDisconnect();

  if (!isConnected) return <div>Not connected</div>;

  return (
    <div>
      <p>Address: {address}</p>
      <p>Balance: {balance?.formatted} {balance?.symbol}</p>
      <button onClick={() => disconnect()}>Disconnect</button>
    </div>
  );
}
```

## 📖 Reading Contract Data

### Scopes Pattern (`apps/frontend/src/lib/scopes.ts`)
```typescript
export const scopes = {
  storedValue: (chainId: number, address: string) =>
    ['contract', 'storedValue', chainId, address.toLowerCase()] as const,
  orgData: (chainId: number, address: string) =>
    ['orgs', 'data', chainId, address.toLowerCase()] as const,
  orgCount: (chainId: number) =>
    ['orgs', 'count', chainId] as const,
} as const;
```

### Read Single Value (Wagmi 2)
```typescript
import { useReadContract } from 'wagmi';
import { useChainId } from 'wagmi';
import { scopes } from '@/lib/scopes';

export function ReadValue({ contractAddress, abi }) {
  const chainId = useChainId();
  
  const { data, isLoading, isError } = useReadContract({
    address: contractAddress,
    abi,
    functionName: 'storedValue',
    query: {
      scopeKey: scopes.storedValue(chainId, contractAddress),
      staleTime: 30_000,
    },
  });

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error reading value</div>;

  return <div>Value: {data?.toString()}</div>;
}
```

### Read Multiple Values (Wagmi 2)
```typescript
import { useReadContracts } from 'wagmi';
import { useChainId } from 'wagmi';

export function ReadMultiple({ contractAddress, abi }) {
  const chainId = useChainId();
  
  const { data, isLoading } = useReadContracts({
    contracts: [
      {
        address: contractAddress,
        abi,
        functionName: 'storedValue',
      },
      {
        address: contractAddress,
        abi,
        functionName: 'storedString',
      },
    ],
    query: {
      scopeKey: ['contract', 'batch', chainId, contractAddress],
    },
  });

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <p>Value: {data?.[0]?.result?.toString()}</p>
      <p>String: {data?.[1]?.result?.toString()}</p>
    </div>
  );
}
```

### Read ERC-20 Balance
```typescript
import { useReadContract } from 'wagmi';
import { erc20Abi } from 'viem';

export function TokenBalance({ tokenAddress, userAddress }) {
  const { data: balance, isLoading } = useReadContract({
    address: tokenAddress,
    abi: erc20Abi,
    functionName: 'balanceOf',
    args: [userAddress],
  });

  if (isLoading) return <div>Loading...</div>;

  return <div>Balance: {balance?.toString()}</div>;
}
```

## ✍️ Writing to Contracts

### Simple Write (Wagmi 2)
```typescript
import { useWriteContract, useWaitForTransactionReceipt } from 'wagmi';
import { useQueryClient } from '@tanstack/react-query';
import { useChainId } from 'wagmi';
import { scopes } from '@/lib/scopes';

export function SetValue({ contractAddress, abi }) {
  const qc = useQueryClient();
  const chainId = useChainId();
  
  const { writeContract, data: hash, isPending, error } = useWriteContract();
  const { isLoading: isConfirming, isSuccess } = useWaitForTransactionReceipt({
    hash,
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
    <div>
      <button 
        onClick={handleSetValue}
        disabled={isPending || isConfirming}
      >
        {isPending ? 'Confirming...' : 'Set Value'}
      </button>
      {isSuccess && <p>Transaction confirmed!</p>}
    </div>
  );
}
```

### Transaction Overlay (REQUIRED for Production)
```typescript
import { TransactionOverlay } from '@/components/ui/TransactionOverlay';

export function SetValueWithOverlay({ contractAddress, abi }) {
  const { writeContract, data: hash, isPending } = useWriteContract();
  const { isLoading: isConfirming } = useWaitForTransactionReceipt({ hash });

  return (
    <div className="relative">
      <button onClick={handleSetValue}>Set Value</button>
      
      {/* Transaction Overlay */}
      {(isPending || isConfirming) && (
        <TransactionOverlay
          isVisible={true}
          type={isPending ? 'pending' : 'confirming'}
        />
      )}
    </div>
  );
}
```

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete overlay system implementation.

## 🎧 Listening to Events

### Basic Event Listening (Wagmi 2)
```typescript
import { useWatchContractEvent } from 'wagmi';
import { useQueryClient } from '@tanstack/react-query';
import { useChainId } from 'wagmi';
import { scopes } from '@/lib/scopes';

export function EventListener({ contractAddress, abi }) {
  const qc = useQueryClient();
  const chainId = useChainId();
  
  useWatchContractEvent({
    address: contractAddress,
    abi,
    eventName: 'ValueSet',
    onLogs(logs) {
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

### Real-Time Event System (Production - Separate WebSocket Client)
```typescript
// apps/frontend/src/realtime/wsClient.ts
import { createPublicClient, webSocket } from 'viem';
import { APP_CHAIN } from '@/config/chain';
import { ALCHEMY_WS_SEPOLIA } from '@/config/providers';

export const wsClient = createPublicClient({
  chain: APP_CHAIN,
  transport: webSocket(ALCHEMY_WS_SEPOLIA),
});

// apps/frontend/src/realtime/useGlobalEvents.ts
import { wsClient } from './wsClient';

export function useGlobalEvents(watchAddresses: `0x${string}`[]) {
  useEffect(() => {
    const unsubscribe = wsClient.watchContractEvent({
      address: watchAddresses,
      abi: CONTRACT_ABI,
      eventName: 'ValueSet',
      poll: false,  // WebSocket-only, no HTTP polling
      onLogs: (logs) => {
        // Invalidate queries
      },
    });
    
    return () => unsubscribe();
  }, [watchAddresses]);
}
```

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete event system.

## 🔄 ERC-20 Token Operations

### Approve Token Spending
```typescript
import { useWriteContract } from 'wagmi';
import { erc20Abi, parseUnits } from 'viem';

export function ApproveToken() {
  const { writeContract } = useWriteContract();

  const handleApprove = () => {
    writeContract({
      address: tokenAddress,
      abi: erc20Abi,
      functionName: 'approve',
      args: [
        spenderAddress, // Address to approve
        parseUnits('1000', 18), // Amount (with decimals)
      ],
    });
  };

  return <button onClick={handleApprove}>Approve</button>;
}
```

### Check Allowance
```typescript
import { useReadContract } from 'wagmi';
import { erc20Abi } from 'viem';

export function CheckAllowance({ tokenAddress, owner, spender }) {
  const { data: allowance } = useReadContract({
    address: tokenAddress,
    abi: erc20Abi,
    functionName: 'allowance',
    args: [owner, spender],
  });

  return <div>Allowance: {allowance?.toString()}</div>;
}
```

## 🎨 Common UI Patterns

### Loading State
```typescript
export function LoadingButton({ isLoading, children, ...props }) {
  return (
    <button disabled={isLoading} {...props}>
      {isLoading ? 'Loading...' : children}
    </button>
  );
}
```

### Error Display
```typescript
export function ErrorMessage({ error }) {
  if (!error) return null;

  return (
    <div className="text-red-500">
      {error.message || 'An error occurred'}
    </div>
  );
}
```

### Transaction Status
```typescript
import { useWaitForTransaction } from 'wagmi';

export function TransactionStatus({ hash }) {
  const { isLoading, isSuccess, isError } = useWaitForTransaction({
    hash,
  });

  if (isLoading) return <div>Confirming transaction...</div>;
  if (isError) return <div>Transaction failed</div>;
  if (isSuccess) return <div>Transaction confirmed!</div>;

  return null;
}
```

## 🔗 Useful Links

### Block Explorers
- **Ethereum Mainnet**: https://etherscan.io
- **Sepolia Testnet**: https://sepolia.etherscan.io
- **Goerli Testnet**: https://goerli.etherscan.io

### Testnet Faucets
- **Sepolia Faucet**: https://sepoliafaucet.com
- **Alchemy Faucet**: https://www.alchemy.com/faucets/ethereum-sepolia

### Tools
- **Remix IDE**: https://remix.ethereum.org
- **Hardhat**: https://hardhat.org
- **Foundry**: https://book.getfoundry.sh

## 📝 TypeScript Types

### Common Types
```typescript
import { Address } from 'viem';

// Contract address
const contractAddress: Address = '0x...';

// User address
const userAddress: Address = '0x...';

// BigInt for large numbers
const amount: bigint = BigInt('1000000000000000000'); // 1 ETH in wei
```

## 🐛 Common Errors & Solutions

### "User rejected the request"
- User clicked "Reject" in wallet
- Solution: Handle gracefully, show message

### "Insufficient funds"
- Not enough ETH for gas
- Solution: Check balance, show helpful message

### "Contract call reverted"
- Transaction failed on-chain
- Solution: Check contract logic, validate inputs

### "Network mismatch"
- Wrong network selected
- Solution: Prompt user to switch networks

## 💡 Best Practices

1. **Monorepo Structure**: Use pnpm workspace with `apps/frontend` and `packages/contracts`
2. **Package Versions**: Use correct versions (Wagmi 2.5+, Viem 2.7+, TanStack Query 5.85+)
3. **RPC Provider**: Use Alchemy with WebSocket (desktop) / HTTP (mobile) transport selection
4. **Scopes Pattern**: Use centralized scope keys for TanStack Query invalidation
5. **Transaction Overlays**: REQUIRED for production UX (full-screen feedback)
6. **Real-Time Events**: Use separate WebSocket client to prevent HTTP polling
7. **Connection Health**: Monitor RPC health and show status to users
8. **Cache Invalidation**: Invalidate targeted scopes, not entire cache
9. **Prevent Flicker**: Use `placeholderData` to keep previous data while refetching
10. **State Separation**: Wagmi for blockchain state, TanStack Query for caching, Zustand for UI
11. **Contract Verification**: Sourcify first, then Blockscout (no Etherscan API keys)
12. **Build Versioning**: Use build counter system for debugging
13. **Always check wallet connection** before contract interactions
14. **Handle loading states** for all async operations
15. **Validate inputs** before sending transactions
16. **Use TypeScript** for type safety (strict mode)
17. **Test on testnets** before mainnet
18. **Handle network switching** gracefully

---

*Keep this reference handy while working through the exercises!*
