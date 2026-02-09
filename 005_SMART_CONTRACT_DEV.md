# Smart Contract Development & Deployment

> **📖 Important**: This module follows the setup guides in `dapp_setup_guides/`. For the complete, authoritative setup instructions, refer to:
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Step-by-step setup guide
>
> The setup guides are the **single source of truth** for scaffolding and architecture decisions.

## Exercise 3.1: Solidity Fundamentals

### Objective
Learn the core concepts of the Solidity programming language before writing your first contract.

### Prerequisites
- [x] Completed Web3 Fundamentals & Wallet Integration
- [x] Monorepo structure set up (`apps/frontend` exists)
- [x] Basic programming knowledge (variables, functions, control flow)

### Instructions

#### Step 1: Understand Solidity Basics
1. Read the [Solidity documentation](https://docs.soliditylang.org/) introduction
2. Understand these core concepts:

   **Value types:**
   ```solidity
   bool isActive = true;
   uint256 count = 42;           // unsigned integer (256 bits, most common)
   int256 temperature = -10;     // signed integer
   address owner = msg.sender;   // 20-byte Ethereum address
   bytes32 hash = keccak256("hello"); // fixed-size byte array
   string name = "MyContract";   // dynamic-size UTF-8 string
   ```

   **Visibility modifiers:**
   ```solidity
   uint256 public storedValue;   // auto-generates a getter function
   uint256 private _secret;      // only accessible within this contract
   uint256 internal _shared;     // accessible within this contract and derived contracts
   ```

   **Function visibility and modifiers:**
   ```solidity
   function setValue(uint256 _value) public {        // anyone can call
       storedValue = _value;
   }

   function getSecret() private view returns (uint256) {  // only this contract
       return _secret;
   }

   function withdraw() external onlyOwner {          // external + custom modifier
       payable(msg.sender).transfer(address(this).balance);
   }
   ```

   **View vs Pure functions:**
   ```solidity
   function getValue() public view returns (uint256) {    // reads state, no gas when called externally
       return storedValue;
   }

   function add(uint256 a, uint256 b) public pure returns (uint256) {  // no state access
       return a + b;
   }
   ```

   **Events:**
   ```solidity
   event ValueChanged(address indexed sender, uint256 newValue);

   function setValue(uint256 _value) public {
       storedValue = _value;
       emit ValueChanged(msg.sender, _value);  // emitted in transaction logs
   }
   ```

   **Custom errors (gas-efficient):**
   ```solidity
   error Unauthorized(address caller);
   error InsufficientBalance(uint256 requested, uint256 available);

   function withdraw(uint256 amount) public {
       if (msg.sender != owner) revert Unauthorized(msg.sender);
       if (amount > balance) revert InsufficientBalance(amount, balance);
       // ...
   }
   ```

   **Mappings and arrays:**
   ```solidity
   mapping(address => uint256) public balances;
   address[] public members;

   function deposit() public payable {
       balances[msg.sender] += msg.value;
   }
   ```

   **Structs and enums:**
   ```solidity
   enum Status { Pending, Active, Closed }

   struct Proposal {
       string description;
       uint256 voteCount;
       Status status;
   }

   Proposal[] public proposals;
   ```

3. Understand `msg.sender`, `msg.value`, `block.timestamp`, and other global variables
4. Understand the difference between `storage`, `memory`, and `calldata`

**Expected Outcome:**
- Solid understanding of Solidity syntax and concepts
- Can read and understand basic Solidity contracts

**Checkpoint:**
- [ ] Understands value types (uint256, address, bool, string, bytes)
- [ ] Understands visibility modifiers (public, private, internal, external)
- [ ] Understands view vs pure functions
- [ ] Understands events and when to use them
- [ ] Understands custom errors
- [ ] Understands mappings, arrays, structs, enums
- [ ] Understands storage vs memory vs calldata

---

## Exercise 3.2: Set Up Hardhat 2 in the Monorepo

### Objective
Set up Hardhat 2 in the monorepo for smart contract compilation, testing, and deployment.

### Prerequisites
- [x] Completed Exercise 3.1
- [x] Monorepo structure set up
- [x] pnpm workspace configured

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
       "deploy:sepolia": "hardhat run scripts/deploy.js --network sepolia",
       "node": "hardhat node"
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

3. Install optional but recommended plugins:
   ```bash
   pnpm add -D hardhat-storage-layout@^0.1.7
   pnpm add -D hardhat-gas-reporter
   pnpm add -D hardhat-contract-sizer
   ```

#### Step 3: Initialize Hardhat
1. Initialize Hardhat:
   ```bash
   pnpm exec hardhat init
   ```

   Choose:
   - "Create a JavaScript project" (CommonJS)
   - "y" for .gitignore
   - "y" for installing dependencies (skip if already installed)

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
   };
   ```

2. Create `packages/contracts/.env`:
   ```bash
   SEPOLIA_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_ALCHEMY_KEY
   PRIVATE_KEY=your_private_key_here
   ```

3. Ensure `.env` is in `.gitignore`

#### Step 5: Add Root Scripts
1. Update root `package.json` to include contracts scripts:
   ```json
   {
     "scripts": {
       "contracts:compile": "pnpm --filter contracts compile",
       "contracts:test": "pnpm --filter contracts test",
       "contracts:deploy:local": "pnpm --filter contracts deploy:local",
       "contracts:deploy:sepolia": "pnpm --filter contracts deploy:sepolia",
       "contracts:node": "pnpm --filter contracts node"
     }
   }
   ```

**Expected Outcome:**
- Hardhat 2 configured in monorepo
- OpenZeppelin contracts installed
- Compilation and testing work

**Checkpoint:**
- [ ] `packages/contracts/` directory created
- [ ] Hardhat initialized and configured
- [ ] OpenZeppelin contracts installed
- [ ] `pnpm compile` works from contracts directory
- [ ] Root scripts configured

### Common Issues & Solutions

**Issue:** `require is not defined` in hardhat config
- **Solution**: Use `.cjs` extension for hardhat config (CommonJS)

**Issue:** Cannot find module `@openzeppelin/contracts`
- **Solution**: Ensure packages are installed as devDependencies in the contracts package

### Submission Checklist

- [ ] Contracts package created in monorepo
- [ ] Hardhat 2 installed and configured
- [ ] OpenZeppelin contracts installed
- [ ] Compilation works
- [ ] Environment variables set up

---

## Exercise 3.3: Write Your First Smart Contract

### Objective
Write a SimpleStorage contract from scratch, understanding each line of code.

### Prerequisites
- [x] Completed Exercise 3.2
- [x] Hardhat configured and compiling

### Instructions

#### Step 1: Create SimpleStorage Contract
1. Create `packages/contracts/contracts/SimpleStorage.sol`:
   ```solidity
   // SPDX-License-Identifier: MIT
   pragma solidity ^0.8.22;

   /// @title SimpleStorage
   /// @notice A simple contract to store and retrieve a value and a string
   /// @dev Used as a learning exercise for basic contract interactions
   contract SimpleStorage {
       // State variables
       uint256 public storedValue;
       string public storedString;

       // Events
       event ValueSet(address indexed sender, uint256 newValue);
       event StringSet(address indexed sender, string newString);

       /// @notice Set the stored value
       /// @param _value The new value to store
       function setValue(uint256 _value) public {
           storedValue = _value;
           emit ValueSet(msg.sender, _value);
       }

       /// @notice Set the stored string
       /// @param _str The new string to store
       function setString(string memory _str) public {
           storedString = _str;
           emit StringSet(msg.sender, _str);
       }

       /// @notice Get both stored values at once
       /// @return value The stored uint256 value
       /// @return str The stored string value
       function getAll() public view returns (uint256 value, string memory str) {
           return (storedValue, storedString);
       }
   }
   ```

2. Understand each part:
   - `SPDX-License-Identifier`: License declaration
   - `pragma solidity ^0.8.22`: Compiler version constraint
   - NatSpec comments (`@title`, `@notice`, `@dev`, `@param`, `@return`)
   - State variables and their automatic getters (`public`)
   - Events with `indexed` parameters for efficient filtering
   - `memory` keyword for string parameters
   - `view` functions that read but don't modify state

#### Step 2: Compile the Contract
1. Compile:
   ```bash
   cd packages/contracts
   pnpm compile
   ```

2. Verify compilation output:
   - Check `artifacts/contracts/SimpleStorage.sol/SimpleStorage.json` exists
   - This JSON file contains the **ABI** (Application Binary Interface) and **bytecode**

3. Understand the ABI -- it defines how the frontend will interact with the contract

**Expected Outcome:**
- SimpleStorage contract written with NatSpec
- Contract compiles without errors
- ABI generated in artifacts

**Checkpoint:**
- [ ] Contract written with NatSpec comments
- [ ] Events declared and emitted
- [ ] Contract compiles successfully
- [ ] Understands the generated ABI

### Common Issues & Solutions

**Issue:** Compilation errors about pragma version
- **Solution**: Ensure hardhat.config.cjs solidity version matches the pragma in your contract

**Issue:** `string memory` vs `string storage`
- **Solution**: Function parameters and local variables use `memory`; state variables are implicitly `storage`

### Submission Checklist

- [ ] SimpleStorage contract created
- [ ] NatSpec comments added
- [ ] Events declared and emitted
- [ ] Contract compiles without errors
- [ ] ABI file generated

---

## Exercise 3.4: Write Contract Tests

### Objective
Write comprehensive tests for your SimpleStorage contract using Hardhat's testing framework.

### Prerequisites
- [x] Completed Exercise 3.3
- [x] SimpleStorage contract compiles

> ### A Note on ethers.js vs Viem
>
> You're about to write `const { ethers } = require("hardhat")`. That's **ethers.js** -- a *different* Ethereum JavaScript library than the **Viem** you installed on the frontend in Web3 Fundamentals & Wallet Integration. Why two libraries?
>
> | Context | Library | Why |
> |---------|---------|-----|
> | **Hardhat** (contracts, tests, deploy scripts) | **ethers.js** | Hardhat's plugin ecosystem (`hardhat-ethers`, `hardhat-upgrades`, etc.) is built around ethers.js. It's the de-facto standard for contract tooling. |
> | **React frontend** | **Viem + Wagmi** | Viem is TypeScript-first, tree-shakeable, and designed for production frontends. Wagmi wraps it in React hooks with caching and state management. |
>
> **The good news:** the concepts are the same, only the syntax differs.
>
> | Concept | ethers.js (Hardhat) | Viem (Frontend) |
> |---------|---------------------|-----------------|
> | Format wei → ether | `ethers.formatEther(wei)` | `formatEther(wei)` |
> | Parse ether → wei | `ethers.parseEther("1.0")` | `parseEther("1.0")` |
> | Get signers / accounts | `ethers.getSigners()` | `useAccount()` (via Wagmi) |
> | Read contract | `contract.storedValue()` | `useReadContract({ ... })` (via Wagmi) |
> | Write contract | `contract.setValue(42)` | `useWriteContract()` (via Wagmi) |
>
> You don't need to memorise both APIs right now. Just be aware that when you see `ethers` in this module, it's the *Hardhat* library, and when you go back to the frontend in Reading Blockchain Data, you'll switch to Viem/Wagmi. That module covers the full library landscape in depth.

### Instructions

#### Step 1: Create Test File
1. Create `packages/contracts/test/SimpleStorage.test.js`:
   ```javascript
   const { expect } = require("chai");
   const { ethers } = require("hardhat");

   describe("SimpleStorage", function () {
     let simpleStorage;
     let owner;
     let otherAccount;

     beforeEach(async function () {
       // Get signers (test accounts)
       [owner, otherAccount] = await ethers.getSigners();

       // Deploy contract
       const SimpleStorage = await ethers.getContractFactory("SimpleStorage");
       simpleStorage = await SimpleStorage.deploy();
       await simpleStorage.waitForDeployment();
     });

     describe("Deployment", function () {
       it("Should initialize with default values", async function () {
         expect(await simpleStorage.storedValue()).to.equal(0);
         expect(await simpleStorage.storedString()).to.equal("");
       });
     });

     describe("setValue", function () {
       it("Should set the stored value", async function () {
         await simpleStorage.setValue(42);
         expect(await simpleStorage.storedValue()).to.equal(42);
       });

       it("Should emit ValueSet event", async function () {
         await expect(simpleStorage.setValue(42))
           .to.emit(simpleStorage, "ValueSet")
           .withArgs(owner.address, 42);
       });

       it("Should allow any account to set value", async function () {
         await simpleStorage.connect(otherAccount).setValue(99);
         expect(await simpleStorage.storedValue()).to.equal(99);
       });

       it("Should handle large numbers", async function () {
         const largeValue = ethers.parseUnits("1000000", 18);
         await simpleStorage.setValue(largeValue);
         expect(await simpleStorage.storedValue()).to.equal(largeValue);
       });
     });

     describe("setString", function () {
       it("Should set the stored string", async function () {
         await simpleStorage.setString("Hello, World!");
         expect(await simpleStorage.storedString()).to.equal("Hello, World!");
       });

       it("Should emit StringSet event", async function () {
         await expect(simpleStorage.setString("Test"))
           .to.emit(simpleStorage, "StringSet")
           .withArgs(owner.address, "Test");
       });

       it("Should handle empty string", async function () {
         await simpleStorage.setString("Hello");
         await simpleStorage.setString("");
         expect(await simpleStorage.storedString()).to.equal("");
       });
     });

     describe("getAll", function () {
       it("Should return both values", async function () {
         await simpleStorage.setValue(42);
         await simpleStorage.setString("Hello");

         const [value, str] = await simpleStorage.getAll();
         expect(value).to.equal(42);
         expect(str).to.equal("Hello");
       });
     });
   });
   ```

#### Step 2: Run Tests
1. Run the test suite:
   ```bash
   pnpm test
   ```

2. Verify all tests pass
3. Understand the test output format

#### Step 3: Understand Testing Patterns
1. **`beforeEach`**: Deploy a fresh contract before every test (isolation)
2. **`ethers.getSigners()`**: Get test accounts with ETH
3. **`ethers.getContractFactory()`**: Load contract for deployment
4. **`.connect(otherAccount)`**: Call as a different account
5. **`.to.emit()`**: Assert that events are emitted
6. **`.withArgs()`**: Assert event arguments

**Expected Outcome:**
- Test file written with multiple test cases
- All tests pass
- Understands testing patterns

**Checkpoint:**
- [ ] Test file created
- [ ] Tests cover deployment, setValue, setString, getAll
- [ ] Event emission tested
- [ ] All tests pass
- [ ] Understands `beforeEach`, `connect`, `emit`

### Common Issues & Solutions

**Issue:** Tests timing out
- **Solution**: Ensure `await` is used on all async calls

**Issue:** Event assertions failing
- **Solution**: Check event name matches exactly and argument types are correct

### Submission Checklist

- [ ] Test file created
- [ ] All functions tested
- [ ] Events tested
- [ ] Edge cases covered
- [ ] All tests pass

---

## Exercise 3.5: Deploy to Local Hardhat Network (Optional)

> **This exercise is optional.** Deploying to a local Hardhat node is useful for rapid iteration and debugging, but you can skip straight to Exercise 3.6 (Deploy to Sepolia Testnet) if you prefer working on a real testnet from the start. The deployment script you create in Step 1 below is reused in Exercise 3.6, so **do complete Step 1** even if you skip the local node steps.

### Objective
Deploy your contract to a local Hardhat network and interact with it, establishing the local development workflow.

### Prerequisites
- [x] Completed Exercise 3.4
- [x] All tests passing

### Instructions

#### Step 1: Create Deployment Script
1. Create `packages/contracts/scripts/deploy.js`:
   ```javascript
   const { ethers } = require("hardhat");

   async function main() {
     const [deployer] = await ethers.getSigners();
     console.log("Deploying contracts with the account:", deployer.address);

     const balance = await ethers.provider.getBalance(deployer.address);
     console.log("Account balance:", ethers.formatEther(balance), "ETH");

     // Deploy SimpleStorage
     const SimpleStorage = await ethers.getContractFactory("SimpleStorage");
     const simpleStorage = await SimpleStorage.deploy();
     await simpleStorage.waitForDeployment();

     const address = await simpleStorage.getAddress();
     console.log("SimpleStorage deployed to:", address);

     // Verify deployment by reading default values
     const value = await simpleStorage.storedValue();
     console.log("Initial storedValue:", value.toString());
   }

   main()
     .then(() => process.exit(0))
     .catch((error) => {
       console.error(error);
       process.exit(1);
     });
   ```

#### Step 2: Run Local Hardhat Node
1. In one terminal, start the local node:
   ```bash
   cd packages/contracts
   pnpm node
   ```
   This starts a local Ethereum node at `http://127.0.0.1:8545` with pre-funded accounts.

2. In another terminal, deploy to the local node:
   ```bash
   cd packages/contracts
   pnpm exec hardhat run scripts/deploy.js --network localhost
   ```

3. Note the deployed contract address from the output

#### Step 3: Interact with the Deployed Contract
1. Use Hardhat console to interact:
   ```bash
   pnpm exec hardhat console --network localhost
   ```

2. In the console:
   ```javascript
   const SimpleStorage = await ethers.getContractFactory("SimpleStorage");
   const contract = SimpleStorage.attach("0x...YOUR_ADDRESS...");

   // Read current value
   await contract.storedValue();

   // Set a new value
   const tx = await contract.setValue(42);
   await tx.wait();

   // Read updated value
   await contract.storedValue();
   ```

**Expected Outcome:**
- Deployment script created
- Contract deployed to local Hardhat node
- Can interact via Hardhat console

**Checkpoint:**
- [ ] Deployment script created
- [ ] Local Hardhat node running
- [ ] Contract deployed successfully
- [ ] Can read and write via console

### Common Issues & Solutions

**Issue:** `Network localhost not found`
- **Solution**: Add localhost network to hardhat.config.cjs:
  ```javascript
  networks: {
    localhost: {
      url: "http://127.0.0.1:8545",
    },
    // ...
  }
  ```

**Issue:** `Nonce too high` errors
- **Solution**: Reset MetaMask account or restart the Hardhat node

### Submission Checklist

- [ ] Deployment script created
- [ ] Local node started
- [ ] Contract deployed locally
- [ ] Interacted via console
- [ ] Understands the local dev workflow

---

## Exercise 3.6: Deploy to Sepolia Testnet

### Objective
Deploy the SimpleStorage contract to the Sepolia testnet and verify it works on a real network.

### Prerequisites
- [x] Completed Exercise 3.5
- [x] Alchemy API key configured (from Web3 Fundamentals & Wallet Integration)
- [x] Sepolia testnet ETH (from a faucet)

### Instructions

#### Step 1: Get Testnet ETH
1. Get Sepolia ETH from a faucet:
   - [Alchemy Faucet](https://www.alchemy.com/faucets/ethereum-sepolia)
   - [Sepolia Faucet](https://sepoliafaucet.com)
2. Verify your wallet has Sepolia ETH

#### Step 2: Configure Environment
1. Ensure `packages/contracts/.env` has:
   ```bash
   SEPOLIA_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_ALCHEMY_KEY
   PRIVATE_KEY=your_wallet_private_key
   ```

2. Install dotenv to load env vars:
   ```bash
   pnpm add -D dotenv
   ```

3. Update `hardhat.config.cjs` to load env:
   ```javascript
   require("dotenv").config();
   require("@nomicfoundation/hardhat-toolbox");
   require("@openzeppelin/hardhat-upgrades");

   module.exports = {
     // ... existing config
     networks: {
       hardhat: { chainId: 1337 },
       localhost: { url: "http://127.0.0.1:8545" },
       sepolia: {
         url: process.env.SEPOLIA_URL || "",
         accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
       },
     },
   };
   ```

#### Step 3: Deploy to Sepolia
1. Deploy:
   ```bash
   pnpm deploy:sepolia
   ```

2. Save the deployed contract address -- you will use it in Reading Blockchain Data and beyond

3. View your contract on a block explorer:
   - [Sepolia Etherscan](https://sepolia.etherscan.io) -- paste the contract address

#### Step 4: Interact on Sepolia
1. Use Hardhat console connected to Sepolia:
   ```bash
   pnpm exec hardhat console --network sepolia
   ```

2. Interact with the deployed contract (same as local, but real transactions)

**Expected Outcome:**
- Contract deployed to Sepolia
- Contract visible on block explorer
- Can interact on testnet

**Checkpoint:**
- [ ] Sepolia ETH obtained
- [ ] Environment configured for Sepolia
- [ ] Contract deployed to Sepolia
- [ ] Contract address saved
- [ ] Contract visible on block explorer

### Submission Checklist

- [ ] Testnet ETH obtained
- [ ] Contract deployed to Sepolia
- [ ] Contract address recorded
- [ ] Verified on block explorer
- [ ] Interaction works on testnet

---

## Exercise 3.7: Share ABIs with the Frontend

### Objective
Set up the workflow for sharing contract ABIs between the contracts package and the frontend app.

### Prerequisites
- [x] Completed Exercise 3.6
- [x] Contract compiled (ABI generated)

### Instructions

#### Step 1: Understand the ABI
1. Open `packages/contracts/artifacts/contracts/SimpleStorage.sol/SimpleStorage.json`
2. The `abi` field contains the contract interface -- this is what the frontend needs
3. The `bytecode` field contains the compiled contract -- only needed for deployment

#### Step 2: Create ABI Directory in Frontend
1. Create `apps/frontend/src/contracts/` directory
2. Create `apps/frontend/src/contracts/SimpleStorage.ts`:
   ```typescript
   // Contract address on Sepolia (update after deployment)
   export const SIMPLE_STORAGE_ADDRESS = '0x...' as const;

   // ABI extracted from compilation artifacts
   export const SIMPLE_STORAGE_ABI = [
     {
       inputs: [],
       name: "storedValue",
       outputs: [{ internalType: "uint256", name: "", type: "uint256" }],
       stateMutability: "view",
       type: "function",
     },
     {
       inputs: [],
       name: "storedString",
       outputs: [{ internalType: "string", name: "", type: "string" }],
       stateMutability: "view",
       type: "function",
     },
     {
       inputs: [{ internalType: "uint256", name: "_value", type: "uint256" }],
       name: "setValue",
       outputs: [],
       stateMutability: "nonpayable",
       type: "function",
     },
     {
       inputs: [{ internalType: "string", name: "_str", type: "string" }],
       name: "setString",
       outputs: [],
       stateMutability: "nonpayable",
       type: "function",
     },
     {
       inputs: [],
       name: "getAll",
       outputs: [
         { internalType: "uint256", name: "value", type: "uint256" },
         { internalType: "string", name: "str", type: "string" },
       ],
       stateMutability: "view",
       type: "function",
     },
     {
       anonymous: false,
       inputs: [
         { indexed: true, internalType: "address", name: "sender", type: "address" },
         { indexed: false, internalType: "uint256", name: "newValue", type: "uint256" },
       ],
       name: "ValueSet",
       type: "event",
     },
     {
       anonymous: false,
       inputs: [
         { indexed: true, internalType: "address", name: "sender", type: "address" },
         { indexed: false, internalType: "string", name: "newString", type: "string" },
       ],
       name: "StringSet",
       type: "event",
     },
   ] as const;
   ```

#### Step 3: Create a Script to Copy ABIs (Optional)
1. Create `packages/contracts/scripts/copy-abis.js`:
   ```javascript
   const fs = require("fs");
   const path = require("path");

   const artifactPath = path.join(
     __dirname,
     "../artifacts/contracts/SimpleStorage.sol/SimpleStorage.json"
   );
   const artifact = JSON.parse(fs.readFileSync(artifactPath, "utf8"));

   const outputDir = path.join(__dirname, "../../../apps/frontend/src/contracts");
   if (!fs.existsSync(outputDir)) fs.mkdirSync(outputDir, { recursive: true });

   const output = `// Auto-generated from contract compilation
   export const SIMPLE_STORAGE_ABI = ${JSON.stringify(artifact.abi, null, 2)} as const;
   `;

   fs.writeFileSync(path.join(outputDir, "SimpleStorageABI.ts"), output);
   console.log("ABI copied to frontend");
   ```

2. Add script to contracts `package.json`:
   ```json
   "scripts": {
     "copy-abis": "node scripts/copy-abis.js"
   }
   ```

**Expected Outcome:**
- ABIs available in the frontend
- Contract address configured
- Frontend can import contract info

**Checkpoint:**
- [ ] ABI extracted from artifacts
- [ ] Contract info file created in frontend
- [ ] Contract address configured
- [ ] Frontend can import ABI and address

### Common Issues & Solutions

**Issue:** ABI out of sync after contract changes
- **Solution**: Re-compile and re-copy ABIs whenever the contract changes

**Issue:** TypeScript type errors with ABI
- **Solution**: Use `as const` assertion for proper type inference with Wagmi

### Submission Checklist

- [ ] ABI directory created in frontend
- [ ] Contract info file created (address + ABI)
- [ ] ABI matches compiled contract
- [ ] Frontend can import contract info

---

## Exercise 3.8: OpenZeppelin Contracts & Upgradeable Patterns

### Objective
Learn to use OpenZeppelin's audited contract libraries and understand upgradeable contract patterns.

### Prerequisites
- [x] Completed Exercise 3.7
- [x] OpenZeppelin contracts installed

### Instructions

#### Step 1: Write an ERC-20 Token Contract
1. Create `packages/contracts/contracts/MyToken.sol`:
   ```solidity
   // SPDX-License-Identifier: MIT
   pragma solidity ^0.8.22;

   import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
   import "@openzeppelin/contracts/access/Ownable.sol";

   /// @title MyToken
   /// @notice A simple ERC-20 token with owner-only minting
   contract MyToken is ERC20, Ownable {
       constructor() ERC20("MyToken", "MTK") Ownable(msg.sender) {
           // Mint 1,000,000 tokens to deployer
           _mint(msg.sender, 1_000_000 * 10 ** decimals());
       }

       /// @notice Mint new tokens (owner only)
       /// @param to The address to mint to
       /// @param amount The amount to mint (in smallest unit)
       function mint(address to, uint256 amount) public onlyOwner {
           _mint(to, amount);
       }
   }
   ```

2. Understand what OpenZeppelin provides:
   - `ERC20`: Full ERC-20 implementation (transfer, approve, balanceOf, etc.)
   - `Ownable`: Access control with a single owner
   - Inheritance: `is ERC20, Ownable` -- your contract inherits all functionality

#### Step 2: Write an Upgradeable Version
1. Create `packages/contracts/contracts/MyTokenUpgradeable.sol`:
   ```solidity
   // SPDX-License-Identifier: MIT
   pragma solidity ^0.8.22;

   import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
   import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
   import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

   /// @title MyTokenUpgradeable
   /// @notice An upgradeable ERC-20 token using the proxy pattern
   contract MyTokenUpgradeable is Initializable, ERC20Upgradeable, OwnableUpgradeable {
       /// @custom:oz-upgrades-unsafe-allow constructor
       constructor() {
           _disableInitializers();
       }

       function initialize() public initializer {
           __ERC20_init("MyToken", "MTK");
           __Ownable_init(msg.sender);
           _mint(msg.sender, 1_000_000 * 10 ** decimals());
       }

       function mint(address to, uint256 amount) public onlyOwner {
           _mint(to, amount);
       }
   }
   ```

2. Understand the key differences:
   - **No constructor logic**: Uses `initialize()` instead (proxy pattern)
   - **`_disableInitializers()`**: Prevents the implementation from being initialized directly
   - **`Initializable`**: Ensures `initialize()` can only be called once
   - **`__ERC20_init()`**: Initialization functions replace constructors

#### Step 3: Create Upgrade Deployment Script
1. Create `packages/contracts/scripts/deploy-upgradeable.js`:
   ```javascript
   const { ethers, upgrades } = require("hardhat");

   async function main() {
     const [deployer] = await ethers.getSigners();
     console.log("Deploying with:", deployer.address);

     const MyToken = await ethers.getContractFactory("MyTokenUpgradeable");
     const myToken = await upgrades.deployProxy(MyToken, [], {
       initializer: "initialize",
     });
     await myToken.waitForDeployment();

     const proxyAddress = await myToken.getAddress();
     console.log("MyToken proxy deployed to:", proxyAddress);

     const implAddress = await upgrades.erc1967.getImplementationAddress(proxyAddress);
     console.log("Implementation address:", implAddress);
   }

   main()
     .then(() => process.exit(0))
     .catch((error) => {
       console.error(error);
       process.exit(1);
     });
   ```

#### Step 4: Test the Upgradeable Contract
1. Create `packages/contracts/test/MyTokenUpgradeable.test.js`:
   ```javascript
   const { expect } = require("chai");
   const { ethers, upgrades } = require("hardhat");

   describe("MyTokenUpgradeable", function () {
     let myToken;
     let owner;

     beforeEach(async function () {
       [owner] = await ethers.getSigners();
       const MyToken = await ethers.getContractFactory("MyTokenUpgradeable");
       myToken = await upgrades.deployProxy(MyToken, [], {
         initializer: "initialize",
       });
       await myToken.waitForDeployment();
     });

     it("Should initialize with correct name and symbol", async function () {
       expect(await myToken.name()).to.equal("MyToken");
       expect(await myToken.symbol()).to.equal("MTK");
     });

     it("Should mint initial supply to deployer", async function () {
       const supply = await myToken.totalSupply();
       const balance = await myToken.balanceOf(owner.address);
       expect(balance).to.equal(supply);
     });

     it("Should allow owner to mint", async function () {
       const amount = ethers.parseUnits("1000", 18);
       await myToken.mint(owner.address, amount);
       // Balance should be initial supply + 1000
     });
   });
   ```

2. Run tests:
   ```bash
   pnpm test
   ```

**Expected Outcome:**
- Standard and upgradeable ERC-20 tokens written
- Understands OpenZeppelin inheritance
- Understands proxy pattern basics
- Tests pass for upgradeable contract

**Checkpoint:**
- [ ] Standard ERC-20 contract written
- [ ] Upgradeable ERC-20 contract written
- [ ] Understands difference between standard and upgradeable
- [ ] Proxy deployment script created
- [ ] Tests pass

### Submission Checklist

- [ ] Both contract versions created
- [ ] Understands OpenZeppelin inheritance
- [ ] Understands proxy pattern
- [ ] Deployment script for upgradeable contract
- [ ] Tests pass

---

## Exercise 3.9: Contract Verification (Sourcify + Blockscout)

### Objective
Verify deployed contracts on Sourcify and Blockscout so their source code is publicly readable.

> **Why not Etherscan?** Etherscan is the most well-known block explorer and supports contract verification, but it **requires a free API key** that you have to sign up for. This curriculum uses **Sourcify** and **Blockscout** instead -- both are free, open, and require no API keys or accounts. Your verified source code will still be publicly visible on Blockscout's explorer (and any other explorer that reads from Sourcify). You can always add Etherscan verification later if you want; it's just one more API key in your Hardhat config.

### Prerequisites
- [x] Completed Exercise 3.8
- [x] Contract deployed to Sepolia

### Instructions

#### Step 1: Install Sourcify Plugin
1. Install the plugin:
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

3. Verify on Sourcify:
   ```bash
   pnpm exec hardhat sourcify --network sepolia
   ```

#### Step 2: Verify on Blockscout
1. Update `hardhat.config.cjs` to add Blockscout verification:
   ```javascript
   module.exports = {
     // ... existing config
     etherscan: {
       apiKey: {
         sepolia: "NO_API_KEY_NEEDED",
       },
       customChains: [
         {
           network: "sepolia",
           chainId: 11155111,
           urls: {
             apiURL: "https://eth-sepolia.blockscout.com/api",
             browserURL: "https://eth-sepolia.blockscout.com",
           },
         },
       ],
     },
   };
   ```

2. Verify on Blockscout:
   ```bash
   pnpm exec hardhat verify --network sepolia <CONTRACT_ADDRESS>
   ```

> **Note**: The `etherscan` config key above is just the name Hardhat's verify plugin uses internally -- it doesn't mean we're verifying on Etherscan. The `customChains` section redirects verification to Blockscout's API instead. The `"NO_API_KEY_NEEDED"` placeholder works because Blockscout doesn't require one.

**Expected Outcome:**
- Contract verified on Sourcify
- Contract verified on Blockscout
- Source code publicly readable

**Checkpoint:**
- [ ] Sourcify verification successful
- [ ] Blockscout verification successful
- [ ] Source code visible on explorers

### Submission Checklist

- [ ] Contract verified on Sourcify
- [ ] Contract verified on Blockscout
- [ ] No API keys required

---

## Review Questions

1. What is the difference between `storage`, `memory`, and `calldata` in Solidity?
2. Why do upgradeable contracts use `initialize()` instead of a constructor?
3. What is the purpose of NatSpec comments?
4. Why do we test contracts before deploying to a testnet?
5. What does the ABI contain, and why does the frontend need it?
6. What is the difference between a proxy contract and an implementation contract?
7. Why verify contracts on Sourcify/Blockscout?

---

## Next Steps

After completing this module, you should:
- Understand Solidity fundamentals (types, functions, events, modifiers)
- Be able to write, test, and deploy smart contracts
- Know how to use OpenZeppelin's contract libraries
- Understand upgradeable contract patterns
- Know how to share ABIs with the frontend
- Know how to verify contracts (Sourcify + Blockscout)
- Proceed to Reading Blockchain Data
