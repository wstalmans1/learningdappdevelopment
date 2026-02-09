# Module 1: Environment Setup & Project Scaffolding

> **📖 Important**: This module follows the setup guides in `dapp_setup_guides/`. For the complete, authoritative setup instructions, refer to:
> - **[001_scaffolding_specs.md](./dapp_setup_guides/001_scaffolding_specs.md)** - Tech stack, architecture, and patterns
> - **[002_setup_instructions_and_best_practices.md](./dapp_setup_guides/002_setup_instructions_and_best_practices.md)** - Step-by-step setup guide
>
> The setup guides are the **single source of truth** for scaffolding and architecture decisions.

## Exercise 1.1: Initial Environment Setup

### Objective
Set up your development environment with all necessary tools for DApp development.

### Prerequisites
- A computer (Mac, Windows, or Linux)
- Internet connection
- Basic familiarity with command line

### Instructions

#### Step 1: Install Node.js
1. Visit [nodejs.org](https://nodejs.org/)
2. Download and install the LTS (Long Term Support) version (v18 or higher)
3. Verify installation:
   ```bash
   node --version
   npm --version
   ```
   You should see version numbers for both commands.

**Expected Outcome:**
- Node.js and npm are installed and accessible from terminal

**Checkpoint:**
- [ ] `node --version` shows v18.x.x or higher
- [ ] `npm --version` shows 9.x.x or higher

#### Step 2: Choose a Package Manager
You have three options: npm (comes with Node.js), yarn, or pnpm. For this course, we'll use npm, but you can choose any:

**Option A: npm (Recommended for beginners)**
- Already installed with Node.js
- No additional installation needed

**Option B: Install Yarn**
```bash
npm install -g yarn
yarn --version
```

**Option C: Install pnpm**
```bash
npm install -g pnpm
pnpm --version
```

**Expected Outcome:**
- A package manager is installed and ready to use

**Checkpoint:**
- [ ] Can run `npm --version` (or yarn/pnpm equivalent)

#### Step 3: Install Git
1. **Mac**: Git may already be installed. Check with `git --version`
   - If not installed, install Xcode Command Line Tools: `xcode-select --install`
2. **Windows**: Download from [git-scm.com](https://git-scm.com/download/win)
3. **Linux**: 
   ```bash
   sudo apt-get install git  # Ubuntu/Debian
   sudo yum install git      # CentOS/RHEL
   ```

4. Configure Git:
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```

**Expected Outcome:**
- Git is installed and configured

**Checkpoint:**
- [ ] `git --version` shows a version number
- [ ] Git user name and email are configured

#### Step 4: Set Up VS Code
1. Download VS Code from [code.visualstudio.com](https://code.visualstudio.com/)
2. Install recommended extensions:
   - **ESLint** (dbaeumer.vscode-eslint)
   - **Prettier** (esbenp.prettier-vscode)
   - **Solidity** (JuanBlanco.solidity)
   - **TypeScript and JavaScript Language Features** (built-in)
   - **Tailwind CSS IntelliSense** (bradlc.vscode-tailwindcss)

3. Open VS Code and install extensions:
   - Press `Cmd+Shift+X` (Mac) or `Ctrl+Shift+X` (Windows/Linux)
   - Search for each extension and click "Install"

**Expected Outcome:**
- VS Code is installed with essential extensions

**Checkpoint:**
- [ ] VS Code opens successfully
- [ ] All recommended extensions are installed

#### Step 5: Create GitHub Account (if needed)
1. Visit [github.com](https://github.com/)
2. Sign up for a free account
3. Verify your email address

**Expected Outcome:**
- GitHub account is created and verified

**Checkpoint:**
- [ ] Can log into GitHub
- [ ] Email is verified

#### Step 6: Set Up SSH Keys (Optional but Recommended)
1. Generate SSH key:
   ```bash
   ssh-keygen -t ed25519 -C "your.email@example.com"
   ```
   - Press Enter to accept default file location
   - Optionally set a passphrase for extra security

2. Copy public key:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
   Copy the entire output

3. Add to GitHub:
   - Go to GitHub → Settings → SSH and GPG keys
   - Click "New SSH key"
   - Paste your public key
   - Give it a title (e.g., "My Laptop")
   - Click "Add SSH key"

4. Test connection:
   ```bash
   ssh -T git@github.com
   ```
   You should see: "Hi [username]! You've successfully authenticated..."

**Expected Outcome:**
- SSH keys are set up for GitHub

**Checkpoint:**
- [ ] SSH key is generated
- [ ] SSH key is added to GitHub
- [ ] Can authenticate with GitHub via SSH

### Common Issues & Solutions

**Issue:** `node: command not found`
- **Solution:** 
  - Make sure Node.js is installed
  - Restart your terminal
  - Check if Node.js is in your PATH

**Issue:** Git commands not recognized
- **Solution:**
  - Verify Git installation: `git --version`
  - Restart terminal after installation
  - On Windows, make sure Git Bash is in PATH

**Issue:** VS Code extensions not installing
- **Solution:**
  - Check internet connection
  - Try installing from VS Code marketplace website
  - Restart VS Code

### Testing Your Solution

Run these commands to verify everything is set up:

```bash
# Check Node.js
node --version
npm --version

# Check Git
git --version
git config --global user.name
git config --global user.email

# Check VS Code (if installed via command line)
code --version
```

### Extension (Optional)

- Set up a GitHub CLI (`gh`) for easier GitHub interactions
- Configure VS Code with custom settings (theme, font, etc.)
- Set up a terminal profile with custom colors

### Resources

- [Node.js Documentation](https://nodejs.org/docs/)
- [Git Documentation](https://git-scm.com/doc)
- [VS Code Documentation](https://code.visualstudio.com/docs)
- [GitHub SSH Setup Guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)

### Submission Checklist

- [ ] Node.js v18+ installed
- [ ] Package manager (npm/yarn/pnpm) working
- [ ] Git installed and configured
- [ ] VS Code installed with recommended extensions
- [ ] GitHub account created
- [ ] SSH keys set up (optional but recommended)

---

## Review Questions

1. What is the difference between Node.js and npm?
2. Why do we need Git for DApp development?
3. What is the purpose of ESLint and Prettier extensions?
4. Why is SSH authentication preferred over HTTPS for Git?

---

## Next Steps

After completing this exercise, you should:
- Have a fully configured development environment
- Be ready to create your first DApp project
- Proceed to Exercise 1.2: Project Initialization

---

## Exercise 1.2: Project Initialization

### Objective
Create a new React project with Vite, TypeScript, and React Router DOM 6, then set up the project structure for DApp development.

### Prerequisites
- [x] Completed Exercise 1.1
- [x] Node.js 18+ installed
- [x] **pnpm 8+ installed** (required - not npm/yarn)
- [x] VS Code installed

> **📖 Reference**: Follow the setup guides in `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for the complete setup process.

### Instructions

#### Step 1: Create Monorepo Root
1. Open your terminal
2. Navigate to where you want to create your project:
   ```bash
   cd ~/Projects  # or your preferred directory
   ```

3. Create the monorepo root directory:
   ```bash
   mkdir dapp-learning && cd dapp-learning
   pnpm init
   ```

**Expected Outcome:**
- Monorepo root directory created with `package.json`

**Checkpoint:**
- [ ] `dapp-learning/` directory exists
- [ ] Root `package.json` exists

#### Step 2: Create Frontend App (pnpm workspace)
1. Create the frontend app using Vite:
   ```bash
   pnpm create vite@latest apps/frontend --template react-ts
   ```
   
   **Note**: `pnpm create vite` automatically creates the `apps/frontend` directory, initializes `package.json`, and sets up the complete Vite/React/TypeScript project structure.

2. Navigate into the frontend app:
   ```bash
   cd apps/frontend
   ```

3. Install dependencies:
   ```bash
   pnpm install
   ```

**Expected Outcome:**
- Frontend app created in `apps/frontend/` with React + Vite + TypeScript

**Checkpoint:**
- [ ] `apps/frontend/` directory exists
- [ ] Can run `pnpm dev` and see the app running

#### Step 3: Setup pnpm Workspace
1. Create `pnpm-workspace.yaml` at the monorepo root:
   ```bash
   cd ../..  # Back to monorepo root
   ```
   
   Create `pnpm-workspace.yaml`:
   ```yaml
   packages:
     - 'apps/*'
     - 'packages/*'
   ```

2. Update root `package.json` with workspace scripts:
   ```json
   {
     "name": "dapp-learning-monorepo",
     "private": true,
     "version": "0.0.0",
     "type": "module",
     "scripts": {
       "dev": "pnpm --filter frontend dev",
       "build": "pnpm --filter frontend build",
       "preview": "pnpm --filter frontend preview",
       "lint": "pnpm --filter frontend lint"
     },
     "engines": {
       "node": ">=18.0.0",
       "pnpm": ">=8.0.0"
     }
   }
   ```

**Expected Outcome:**
- pnpm workspace configured
- Root scripts set up for frontend app

**Checkpoint:**
- [ ] `pnpm-workspace.yaml` exists
- [ ] Root `package.json` has workspace scripts

#### Step 4: Install Core Dependencies
1. Navigate to frontend app:
   ```bash
   cd apps/frontend
   ```

2. Install React Router DOM 6:
   ```bash
   pnpm add react-router-dom@^6.20.0
   ```

3. Install Tailwind CSS:
   ```bash
   pnpm add -D tailwindcss@^3.3.6 postcss@^8.4.32 autoprefixer@^10.4.16
   pnpm exec tailwindcss init -p
   ```

4. Open the project in VS Code:
   ```bash
   code ../..  # Open monorepo root
   ```

**Expected Outcome:**
- React Router DOM 6 installed
- Tailwind CSS configured

**Checkpoint:**
- [ ] `react-router-dom` is in `apps/frontend/package.json`
- [ ] Tailwind config files exist

#### Step 5: Explore Monorepo Structure
1. Understand the monorepo structure:
   ```
   dapp-learning/              # Monorepo root
   ├── apps/
   │   └── frontend/          # Frontend app
   │       ├── src/
   │       │   ├── App.tsx
   │       │   └── main.tsx
   │       ├── public/
   │       ├── vite.config.ts
   │       ├── tsconfig.json
   │       └── package.json
   ├── packages/              # (Will be created in later exercises)
   │   └── contracts/        # Smart contracts (Module 5+)
   ├── pnpm-workspace.yaml    # Workspace configuration
   └── package.json           # Root workspace config
   ```

2. Read through `apps/frontend/package.json` to understand:
   - Dependencies vs devDependencies
   - Available scripts (dev, build, preview, lint)

**Expected Outcome:**
- Understanding of monorepo structure
- Understanding of pnpm workspace

**Checkpoint:**
- [ ] Can explain monorepo structure
- [ ] Understands pnpm workspace concept
- [ ] Can explain difference between dependencies and devDependencies

#### Step 6: Set Up React Router DOM 6
1. Update `src/main.tsx` to include BrowserRouter:
   ```tsx
   import React from 'react'
   import ReactDOM from 'react-dom/client'
   import { BrowserRouter } from 'react-router-dom'
   import App from './App.tsx'
   import './index.css'

   ReactDOM.createRoot(document.getElementById('root')!).render(
     <React.StrictMode>
       <BrowserRouter>
         <App />
       </BrowserRouter>
     </React.StrictMode>,
   )
   ```

2. Create a basic routing structure in `src/App.tsx`:
   ```tsx
   import { Routes, Route } from 'react-router-dom'
   import Home from './pages/Home'

   function App() {
     return (
       <Routes>
         <Route path="/" element={<Home />} />
       </Routes>
     )
   }

   export default App
   ```

3. Create `src/pages/Home.tsx`:
   ```tsx
   function Home() {
     return (
       <div>
         <h1>Welcome to DApp Learning!</h1>
       </div>
     )
   }

   export default Home
   ```

**Expected Outcome:**
- React Router DOM 6 is configured with BrowserRouter
- Basic routing is set up

**Checkpoint:**
- [ ] BrowserRouter wraps the app
- [ ] Can navigate to routes
- [ ] Routes work correctly

#### Step 7: Create Custom Project Structure
1. Create the following directory structure:
   ```bash
   mkdir -p src/{components,hooks,lib,pages,types,utils}
   mkdir -p contracts
   mkdir -p docs
   ```

2. Your structure should look like:
   ```
   dapp-learning/              # Monorepo root
   ├── apps/
   │   └── frontend/
   │       ├── src/
   │       │   ├── components/   # Reusable React components
   │       │   ├── hooks/        # Custom React hooks
   │       │   ├── lib/           # Utility libraries and configs
   │       │   ├── pages/         # Page components (routes)
   │       │   ├── types/         # TypeScript type definitions
   │       │   ├── utils/         # Helper functions
   │       │   ├── config/        # Configuration files
   │       │   ├── stores/        # Zustand stores
   │       │   ├── App.tsx        # Main app with routes
   │       │   └── main.tsx       # Entry point
   │       └── public/
   │           └── _redirects     # For static hosting
   ├── packages/
   │   └── contracts/            # Smart contracts (created later)
   ├── pnpm-workspace.yaml
   └── package.json
   ```

3. Create a README.md explaining the structure:
   ```markdown
   # DApp Learning Project

   ## Monorepo Structure

   This project uses a **pnpm workspace** monorepo structure:

   - `apps/frontend/` - React + Vite frontend application
   - `packages/contracts/` - Hardhat smart contract project (created in later modules)
   - `pnpm-workspace.yaml` - Workspace configuration

   ## Frontend Structure (`apps/frontend/`)

   - `src/pages/` - Page components for routes
   - `src/components/` - Reusable React components
   - `src/hooks/` - Custom React hooks for web3 interactions
   - `src/lib/` - Utility libraries
   - `src/config/` - Configuration files (Wagmi, RainbowKit, chains, providers)
   - `src/stores/` - Zustand stores for UI state
   - `src/types/` - TypeScript type definitions
   - `src/utils/` - Helper functions and utilities
   - `public/_redirects` - Static hosting configuration (all routes → index.html)

   > **📖 Reference**: See `dapp_setup_guides/001_scaffolding_specs.md` for complete architecture details.
   ```

**Expected Outcome:**
- Organized project structure ready for DApp development

**Checkpoint:**
- [ ] All directories are created
- [ ] README.md explains the structure

#### Step 8: Create _redirects File for Static Hosting
1. Create a `_redirects` file in the `public/` directory:
   ```
   /*    /index.html   200
   ```

   This ensures that all routes are served `index.html`, which is required for client-side routing to work on static hosting (IPFS, Vercel, Netlify).

2. If using Vite, update `vite.config.ts` to copy the `_redirects` file:
   ```typescript
   import { defineConfig } from 'vite'
   import react from '@vitejs/plugin-react'

   export default defineConfig({
     plugins: [react()],
     publicDir: 'public',
   })
   ```

**Expected Outcome:**
- `_redirects` file is created for static hosting compatibility

**Checkpoint:**
- [ ] `_redirects` file exists in `public/`
- [ ] File contains the redirect rule

#### Step 9: Configure package.json Scripts
1. Open `package.json`
2. Review existing scripts:
   ```json
   {
     "scripts": {
       "dev": "next dev",
       "build": "next build",
       "start": "next start",
       "lint": "next lint"
     }
   }
   ```

3. Add helpful scripts (optional but recommended):
   ```json
   {
     "scripts": {
       "dev": "next dev",
       "build": "next build",
       "start": "next start",
       "lint": "next lint",
       "type-check": "tsc --noEmit",
       "format": "prettier --write .",
       "format:check": "prettier --check ."
     }
   }
   ```

**Expected Outcome:**
- Useful scripts for development workflow

**Checkpoint:**
- [ ] Scripts are added to package.json
- [ ] Can run `npm run type-check`

#### Step 10: Set Up .gitignore
1. Check if `.gitignore` exists (it should from Vite)
2. Verify it includes:
   ```
   # Dependencies
   node_modules/
   .pnp
   .pnp.js

   # Testing
   coverage/

   # Vite
   dist/
   dist-ssr/
   *.local

   # Production
   *.log
   npm-debug.log*

   # Environment variables
   .env
   .env.local
   .env.development.local
   .env.test.local
   .env.production.local

   # IDE
   .vscode/
   .idea/
   *.swp
   *.swo

   # OS
   .DS_Store
   Thumbs.db
   ```

3. Add DApp-specific ignores:
   ```
   # Hardhat
   cache/
   artifacts/
   
   # TypeScript
   *.tsbuildinfo
   ```

**Expected Outcome:**
- Proper `.gitignore` prevents committing unnecessary files

**Checkpoint:**
- [ ] `.gitignore` includes all necessary patterns
- [ ] `node_modules/` is ignored

### Code Snippets

**Root `package.json` example:**
```json
{
  "name": "dapp-learning-monorepo",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "pnpm --filter frontend dev",
    "build": "pnpm --filter frontend build",
    "preview": "pnpm --filter frontend preview",
    "lint": "pnpm --filter frontend lint"
  },
  "engines": {
    "node": ">=18.0.0",
    "pnpm": ">=8.0.0"
  }
}
```

**Frontend `apps/frontend/package.json` example:**
```json
{
  "name": "frontend",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "type-check": "tsc --noEmit"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.20.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.43",
    "@types/react-dom": "^18.2.17",
    "@types/node": "^24.3.0",
    "@vitejs/plugin-react": "^4.2.1",
    "typescript": "^5.2.2",
    "vite": "^5.4.0",
    "tailwindcss": "^3.3.6",
    "postcss": "^8.4.32",
    "autoprefixer": "^10.4.16"
  }
}
```

> **📖 Reference**: See `dapp_setup_guides/002_setup_instructions_and_best_practices.md` for complete dependency installation instructions with correct versions.

### Common Issues & Solutions

**Issue:** `pnpm` command not found
- **Solution:**
  - Install pnpm: `npm install -g pnpm` or `corepack enable` (Node 16.10+)
  - Verify: `pnpm --version` (should be 8.0.0+)

**Issue:** `pnpm create vite` fails
- **Solution:**
  - Check internet connection
  - Clear pnpm cache: `pnpm store prune`
  - Try with specific version: `pnpm create vite@latest`

**Issue:** Workspace not recognized
- **Solution:**
  - Ensure `pnpm-workspace.yaml` exists at monorepo root
  - Verify `packages: ['apps/*', 'packages/*']` is correct
  - Run `pnpm install` from root to link workspace

**Issue:** TypeScript errors after project creation
- **Solution:**
  - Run `pnpm install` from root to install all dependencies
  - Restart VS Code
  - Check `apps/frontend/tsconfig.json` is properly configured

**Issue:** Port 5173 already in use (Vite default)
- **Solution:**
  - Kill the process using port 5173
  - Or configure a different port in `apps/frontend/vite.config.ts`:
    ```typescript
    export default defineConfig({
      server: {
        port: 3000
      }
    })
    ```

**Issue:** Routes return 404 on refresh (static hosting)
- **Solution:**
  - Ensure `_redirects` file exists in `apps/frontend/public/` directory
  - Verify hosting platform is configured to use `_redirects` file
  - For Vercel: Create `vercel.json` with redirect rules
  - For Netlify: `_redirects` file should work automatically

### Testing Your Solution

1. Start the development server from monorepo root:
   ```bash
   pnpm dev
   ```
   
   Or from frontend directory:
   ```bash
   cd apps/frontend
   pnpm dev
   ```

2. Open [http://localhost:5173](http://localhost:5173) (Vite default port)
   - You should see the React app

3. Make a small change to `apps/frontend/src/pages/Home.tsx`:
   ```tsx
   function Home() {
     return (
       <main>
         <h1>Welcome to DApp Learning!</h1>
       </main>
     )
   }

   export default Home
   ```

4. Verify hot reload works (page updates automatically)

5. Run linting from root:
   ```bash
   pnpm lint
   ```

**Expected Result:**
- Dev server runs without errors
- Page loads in browser
- Hot reload works
- No linting errors
- Workspace commands work from root

### Extension (Optional)

- Set up a custom `tsconfig.json` with stricter settings
- Configure path aliases for cleaner imports
- Set up a pre-commit hook with Husky
- Add a `.editorconfig` file for consistent formatting

### Resources

- **[Setup Guides](./dapp_setup_guides/)** - Single source of truth for scaffolding and architecture
- [React Documentation](https://react.dev/)
- [React Router Documentation](https://reactrouter.com/)
- [Vite Documentation](https://vitejs.dev/)
- [pnpm Documentation](https://pnpm.io/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)

### Submission Checklist

- [ ] **Monorepo root created** with `pnpm-workspace.yaml`
- [ ] **Frontend app created** in `apps/frontend/` using Vite
- [ ] **pnpm workspace** configured and working
- [ ] React Router DOM 6 installed and configured
- [ ] BrowserRouter set up in `apps/frontend/src/main.tsx`
- [ ] `_redirects` file created in `apps/frontend/public/`
- [ ] Project structure organized (components, hooks, pages, etc.)
- [ ] README.md created
- [ ] Scripts configured in root and frontend `package.json`
- [ ] `.gitignore` properly set up
- [ ] Dev server runs without errors (`pnpm dev` from root)
- [ ] Can make changes and see hot reload
- [ ] Routes work correctly (no 404 on refresh)

---

## Review Questions

1. What is a monorepo and why do we use pnpm workspaces?
2. What is the difference between BrowserRouter and HashRouter in React Router?
3. Why do we need a `_redirects` file for static hosting?
4. What's the difference between `dependencies` and `devDependencies`?
5. Why is `.gitignore` important?
6. What is Vite and how does it differ from Create React App?
7. How do you run commands in a pnpm workspace (from root vs from package)?

---

## Next Steps

After completing this exercise, you should:
- Have a working **monorepo** with pnpm workspace
- Have a **frontend app** in `apps/frontend/` with React + Vite
- Understand React Router DOM 6 setup with BrowserRouter
- Have configured `_redirects` for static hosting compatibility
- Understand the monorepo structure and workspace concept
- Be ready to configure development tools
- **Read the setup guides** in `dapp_setup_guides/` for complete architecture details
- Proceed to Exercise 1.3: Development Tools Configuration
