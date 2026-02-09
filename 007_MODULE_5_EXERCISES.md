# Module 5: Advanced Contract Interactions

> **📖 Important**: This module follows the setup guides in `dapp_setup_guides/`. For the complete, authoritative setup instructions, refer to:
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Step-by-step setup guide
>
> The setup guides are the **single source of truth** for scaffolding and architecture decisions.

## Exercise 5.1: Set Up Hardhat 2 for Smart Contract Development

### Objective
Set up Hardhat 2 in the monorepo for smart contract development, compilation, and testing.

### Prerequisites
- [x] Completed Module 4
- [x] Monorepo structure set up
- [x] Basic Solidity knowledge

### Instructions

#### Step 1: Create Contracts Package
1. Navigate to packages directory:
   ```bash
   cd packages
   mkdir contracts && cd contracts
   pnpm init
   ```

2. Update `package.json`:
   ```json
   {
     "name": "contracts",
     "version": "0.0.0",
     "private": true,
     "scripts": {
       "compile": "hardhat compile",
       "test": "hardhat test",
       "deploy:local": "hardhat run scripts/deploy.js --network hardhat",
       "deploy:sepolia": "hardhat run scripts/deploy.js --network sepolia"
     }
   }
   ```

#### Step 2: Install Hardhat 2 and Dependencies
1. Install Hardhat 2 and tooling:
   ```bash
   pnpm add -D hardhat@^2.26.1 @nomicfoundation/hardhat-toolbox@^6.1.0
   ```

2. Install OpenZeppelin contracts:
   ```bash
   pnpm add -D @openzeppelin/contracts@^5.4.0 @openzeppelin/contracts-upgradeable@^5.4.0
   pnpm add -D @openzeppelin/hardhat-upgrades@^3.5.0
   ```

3. Install storage layout plugin:
   ```bash
   pnpm add -D hardhat-storage-layout@^0.1.7
   ```

#### Step 3: Initialize Hardhat
1. Initialize Hardhat:
   ```bash
   pnpm exec hardhat init
   ```
   
   Choose:
   - "Create a JavaScript project" (CommonJS)
   - "y" for .gitignore
   - "y" for installing dependencies (skip, already installed)

#### Step 4: Configure Hardhat
1. Update `hardhat.config.cjs`:
   ```javascript
   require("@nomicfoundation/hardhat-toolbox");
   require("hardhat-storage-layout");
   require("@openzeppelin/hardhat-upgrades");

   /** @type import('hardhat/config').HardhatUserConfig */
   module.exports = {
     solidity: {
       version: "0.8.22",
       settings: {
         optimizer: {
           enabled: true,
           runs: 200,
         },
         outputSelection: {
           "*": {
             "*": ["storageLayout"],
           },
         },
       },
     },
     networks: {
       hardhat: {
         chainId: 1337,
       },
       sepolia: {
         url: process.env.SEPOLIA_URL,
         accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
       },
     },
     paths: {
       sources: "./contracts",
       tests: "./test",
       cache: "./cache",
       artifacts: "./artifacts",
     },
     storageLayout: {
       outputDirectory: "./storage-layouts",
       flat: false,
     },
   };
   ```

2. Create `.env` file:
   ```bash
   SEPOLIA_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_ALCHEMY_KEY
   PRIVATE_KEY=your_private_key_here
   ```

**Expected Outcome:**
- Hardhat 2 configured
- OpenZeppelin upgrades plugin installed
- Storage layout plugin configured

**Checkpoint:**
- [ ] Hardhat initialized
- [ ] Configuration file created
- [ ] OpenZeppelin contracts installed
- [ ] Storage layout plugin configured
- [ ] Environment variables set up

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete Hardhat setup.

### Submission Checklist

- [ ] Contracts package created
- [ ] Hardhat 2 installed
- [ ] OpenZeppelin contracts installed
- [ ] Hardhat configured
- [ ] Environment variables set up

---

## Exercise 5.2: Deploy Upgradeable Contract

### Objective
Deploy an upgradeable contract using OpenZeppelin's upgradeable patterns.

### Prerequisites
- [x] Completed Exercise 5.1
- [x] Hardhat configured
- [x] Testnet ETH for deployment

### Instructions

#### Step 1: Create Upgradeable Contract
1. Create `packages/contracts/contracts/MyToken.sol`:
   ```solidity
   // SPDX-License-Identifier: MIT
   pragma solidity ^0.8.22;

   import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
   import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
   import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

   contract MyToken is Initializable, ERC20Upgradeable, OwnableUpgradeable {
       /// @custom:oz-upgrades-unsafe-allow constructor
       constructor() {
           _disableInitializers();
       }

       function initialize() public initializer {
           __ERC20_init("MyToken", "MTK");
           __Ownable_init(msg.sender);
           _mint(msg.sender, 1000000 * 10**decimals());
       }

       function mint(address to, uint256 amount) public onlyOwner {
           _mint(to, amount);
       }
   }
   ```

#### Step 2: Create Deployment Script
1. Create `packages/contracts/scripts/deploy.js`:
   ```javascript
   const { ethers, upgrades } = require("hardhat");

   async function main() {
     const [deployer] = await ethers.getSigners();
     console.log("Deploying contracts with the account:", deployer.address);

     const MyToken = await ethers.getContractFactory("MyToken");
     const myToken = await upgrades.deployProxy(MyToken, [], { 
       initializer: "initialize" 
     });
     await myToken.waitForDeployment();

     const proxyAddress = await myToken.getAddress();
     console.log("MyToken proxy deployed to:", proxyAddress);

     // Get implementation address
     const implementationAddress = await upgrades.erc1967.getImplementationAddress(proxyAddress);
     console.log("Implementation address:", implementationAddress);
   }

   main()
     .then(() => process.exit(0))
     .catch((error) => {
       console.error(error);
       process.exit(1);
     });
   ```

#### Step 3: Deploy to Sepolia
1. Compile contracts:
   ```bash
   pnpm compile
   ```

2. Deploy to Sepolia:
   ```bash
   pnpm deploy:sepolia
   ```

3. Save contract addresses for frontend use

**Expected Outcome:**
- Upgradeable contract deployed
- Proxy and implementation addresses obtained

**Checkpoint:**
- [ ] Contract compiles without errors
- [ ] Deployment script created
- [ ] Contract deployed to Sepolia
- [ ] Proxy address saved

### Submission Checklist

- [ ] Upgradeable contract created
- [ ] Deployment script created
- [ ] Contract deployed successfully
- [ ] Proxy and implementation addresses obtained

---

## Exercise 5.3: ERC-20 Token Transfer with Approval

### Objective
Implement ERC-20 token transfer with the approval pattern (two-step transaction).

### Prerequisites
- [x] Completed Exercise 5.2
- [x] Token contract deployed
- [x] Understanding of ERC-20 standard

### Instructions

#### Step 1: Create Approval Component
1. Create `apps/frontend/src/components/TokenApproval.tsx`:
   ```typescript
   import { useState } from 'react';
   import { useWriteContract, useReadContract } from 'wagmi';
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
- Two-step transaction flow (approve → transfer)
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

## Exercise 5.4: Real-Time Event System (Production)

### Objective
Set up a real-time event system using a separate WebSocket client to prevent HTTP polling.

### Prerequisites
- [x] Completed Exercise 5.3
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
   import SimpleStorageABI from '@/abis/SimpleStorage.json';

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
               abi: SimpleStorageABI,
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

1. Why do we use OpenZeppelin's upgradeable contracts?
2. What is the difference between a proxy and an implementation contract?
3. Why do ERC-20 transfers require approval first?
4. Why use a separate WebSocket client for event watching?
5. What is the purpose of storage layout validation?

---

## Next Steps

After completing this module, you should:
- Understand how to set up Hardhat 2
- Know how to deploy upgradeable contracts
- Be able to implement approval patterns
- Understand real-time event systems
- Know how to verify contracts (Sourcify/Blockscout)
- Proceed to Module 6: State Management & Data Fetching
