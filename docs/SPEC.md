# SafeSafe - Product Specification

## Overview

SafeSafe is a security-focused Safe multisig interface designed for institutions and power users. It prioritizes transaction clarity, hardware wallet UX, and risk assessment.

**Core Value Proposition**: Make it crystal clear what users are signing, with step-by-step guidance for hardware wallet users.

**Target Users**:
- Institutions managing treasury with Safe multisig
- Power users who want to understand exactly what they're signing
- Hardware wallet users who need clear guidance on what appears on their device screen

---

## Technical Stack

| Layer | Technology |
|-------|------------|
| Framework | Next.js 14+ (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS + shadcn/ui |
| State | Zustand |
| Safe Integration | @safe-global/protocol-kit, @safe-global/api-kit |
| Wallet Connection | WalletConnect v2 + wagmi + viem |
| Contract Decoding | Sourcify API + 4byte.directory |
| Simulation | Tenderly Simulation API |

## Supported Chains

**Phase 1:**
- Ethereum Mainnet
- Arbitrum One
- Optimism
- Base
- Sepolia (testnet)

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        SafeSafe UI                          │
├─────────────────────────────────────────────────────────────┤
│  Transaction Decoder  │  Risk Scorer  │  Signing Wizard     │
├─────────────────────────────────────────────────────────────┤
│                    Safe SDK Layer                           │
│         (Protocol Kit + API Kit)                            │
├─────────────────────────────────────────────────────────────┤
│  Safe Transaction    │  Sourcify  │  4byte   │  Tenderly    │
│  Service API         │  API       │  API     │  Sim API     │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Features

### Phase 1: Foundation (MVP)

**Goal**: Import existing Safes and view transactions with decoded data

#### 1.1 Safe Import & Dashboard
- Connect wallet via WalletConnect
- Import existing Safe by address
- Display Safe overview (owners, threshold, balance)
- List pending and historical transactions

#### 1.2 Transaction Decoding Engine
- Fetch verified source/ABI from Sourcify API
- Fallback to 4byte.directory for function selectors
- Parse calldata into human-readable format
- Display: function name, parameters with types, decoded values
- Show target contract info (name, verification status)

**Sourcify Integration:**
```
GET https://sourcify.dev/server/files/{chain}/{address}
```
Returns: source files, ABI, metadata for verified contracts

**4byte Fallback:**
```
GET https://www.4byte.directory/api/v1/signatures/?hex_signature=0x...
```
Returns: function signature text for unverified contracts

#### 1.3 Basic Transaction View
- Clear breakdown: FROM (Safe) → TO (contract/address)
- Value transfer amount in ETH and USD
- Gas estimation
- Raw data toggle for advanced users

---

### Phase 2: Signing Experience

**Goal**: Best-in-class hardware wallet signing UX

#### 2.1 Step-by-Step Signing Wizard
- Pre-sign checklist (verify recipient, amount, function)
- Hardware wallet detection and guidance
- Step-by-step instructions per wallet type (Ledger/Trezor via WalletConnect)
- Visual confirmation of what will appear on device screen
- Post-sign verification

#### 2.2 Transaction Simulation
- Integrate Tenderly Simulation API
- Show predicted state changes before signing
- Display token balance changes
- Highlight approvals and their implications
- Warn on failed simulations

**Tenderly API:**
- Requires API key (free tier available)
- Returns: state changes, logs, gas used, success/failure

#### 2.3 Multi-Sig Coordination
- Show who has signed / who hasn't
- Signature status timeline
- Optional notifications (email/webhook) for pending signatures

---

### Phase 3: Risk & Intelligence

**Goal**: Help users understand transaction risk

#### 3.1 Risk Scoring System
- Contract verification score (Sourcify verified = lower risk)
- Contract age and activity metrics
- Known contract labels (Uniswap, Aave, etc.)
- Interaction history (first time vs. frequent)
- Anomaly detection (unusual amounts, new recipients)

#### 3.2 Signer Reputation & History
- Track signer behavior over time
- Show past transactions approved by each signer
- Flag unusual signing patterns
- Signer activity dashboard

#### 3.3 Address Intelligence
- Address book with verification
- ENS resolution and verification
- Contract vs. EOA detection
- Known scam/phishing address warnings

---

### Phase 4: Advanced Features

**Goal**: Power user and institutional features

#### 4.1 Transaction Policies (future)
- Spending limits per time period
- Whitelist/blacklist addresses
- Required approvers for certain operations

#### 4.2 Batch Operations
- Queue multiple transactions
- Preview combined effect
- Execute in sequence

#### 4.3 Safe Creation (deferred from Phase 1)
- Create new Safe with security guidance
- Recommended configurations for institutions

---

## Repository Structure

```
safesafe/
├── docs/
│   └── SPEC.md                 # This document
├── src/
│   ├── app/                    # Next.js App Router pages
│   │   ├── page.tsx            # Landing/connect page
│   │   ├── safe/
│   │   │   └── [address]/
│   │   │       ├── page.tsx    # Safe dashboard
│   │   │       └── tx/
│   │   │           └── [id]/
│   │   │               └── page.tsx  # Transaction detail
│   │   └── layout.tsx
│   │
│   ├── components/
│   │   ├── ui/                 # shadcn/ui components
│   │   ├── safe/               # Safe-specific components
│   │   │   ├── SafeOverview.tsx
│   │   │   ├── OwnerList.tsx
│   │   │   └── TransactionList.tsx
│   │   ├── transaction/
│   │   │   ├── TransactionCard.tsx
│   │   │   ├── DecodedCalldata.tsx
│   │   │   ├── SimulationResult.tsx
│   │   │   └── RiskScore.tsx
│   │   ├── signing/
│   │   │   ├── SigningWizard.tsx
│   │   │   ├── HardwareWalletGuide.tsx
│   │   │   └── SignatureStatus.tsx
│   │   └── wallet/
│   │       └── ConnectButton.tsx
│   │
│   ├── lib/
│   │   ├── safe/               # Safe SDK wrappers
│   │   │   ├── client.ts       # Protocol Kit initialization
│   │   │   ├── api.ts          # API Kit wrappers
│   │   │   └── types.ts
│   │   ├── decoder/            # Transaction decoding
│   │   │   ├── sourcify.ts     # Sourcify API client
│   │   │   ├── fourbyte.ts     # 4byte directory client
│   │   │   ├── decoder.ts      # Main decoding logic
│   │   │   └── types.ts
│   │   ├── simulation/         # Transaction simulation
│   │   │   ├── tenderly.ts
│   │   │   └── types.ts
│   │   ├── risk/               # Risk scoring
│   │   │   ├── scorer.ts
│   │   │   ├── rules.ts
│   │   │   └── types.ts
│   │   └── chains.ts           # Chain configurations
│   │
│   ├── hooks/                  # React hooks
│   │   ├── useSafe.ts
│   │   ├── useTransaction.ts
│   │   ├── useDecoder.ts
│   │   └── useSimulation.ts
│   │
│   ├── store/                  # State management (Zustand)
│   │   ├── safe.ts
│   │   └── settings.ts
│   │
│   └── styles/
│       └── globals.css
│
├── public/
├── .env.example
├── next.config.js
├── tailwind.config.js
├── tsconfig.json
├── package.json
└── README.md
```

---

## External APIs

### 1. Safe Transaction Service
**Purpose**: Get Safe info, transactions, signatures

Endpoints per chain:
- Mainnet: `https://safe-transaction-mainnet.safe.global`
- Arbitrum: `https://safe-transaction-arbitrum.safe.global`
- Optimism: `https://safe-transaction-optimism.safe.global`
- Base: `https://safe-transaction-base.safe.global`
- Sepolia: `https://safe-transaction-sepolia.safe.global`

Used via `@safe-global/api-kit`

### 2. Sourcify API
**Purpose**: Get verified contract source code and ABI

```
GET https://sourcify.dev/server/files/{chain}/{address}
```
- Returns: source files, ABI, metadata
- Chains: Supports all major EVM chains
- Fallback needed for unverified contracts

### 3. 4byte Directory
**Purpose**: Decode function selectors when no ABI available

```
GET https://www.4byte.directory/api/v1/signatures/?hex_signature=0x...
```
- Returns: function signature text
- Contains 1.4M+ signatures
- Use for: unverified contracts, quick decoding

### 4. Tenderly Simulation API
**Purpose**: Simulate transactions before execution

- Requires API key (free tier available)
- Returns: state changes, logs, gas used, success/failure

---

## Environment Variables

```env
# WalletConnect (required)
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=

# Tenderly (optional, for simulation)
TENDERLY_ACCESS_KEY=
TENDERLY_PROJECT=
TENDERLY_ACCOUNT=

# Optional: Analytics
NEXT_PUBLIC_POSTHOG_KEY=
```

---

## Implementation Milestones

### Milestone 1: Project Setup
- [ ] Initialize Next.js project with TypeScript
- [ ] Configure Tailwind CSS and shadcn/ui
- [ ] Set up WalletConnect + wagmi
- [ ] Create basic layout and routing
- [ ] Environment configuration

### Milestone 2: Safe Integration
- [ ] Implement Safe SDK initialization
- [ ] Build Safe import flow
- [ ] Create Safe dashboard with overview
- [ ] Display owner list and threshold
- [ ] Show asset balances

### Milestone 3: Transaction Decoding
- [ ] Build Sourcify API client
- [ ] Build 4byte fallback client
- [ ] Create decoding engine with fallback chain
- [ ] Design decoded calldata component
- [ ] Display human-readable transaction details

### Milestone 4: Transaction List & Detail
- [ ] Fetch transactions from Safe Transaction Service
- [ ] Build transaction list component
- [ ] Create transaction detail page
- [ ] Show signature status
- [ ] Add execution functionality

### Milestone 5: Signing Wizard
- [ ] Design step-by-step signing flow
- [ ] Detect hardware wallet connection type
- [ ] Create pre-sign checklist UI
- [ ] Build hardware wallet guidance screens
- [ ] Implement signing with WalletConnect

### Milestone 6: Simulation
- [ ] Integrate Tenderly API
- [ ] Build simulation result component
- [ ] Show predicted state changes
- [ ] Display warnings for failed simulations

### Milestone 7: Risk Scoring
- [ ] Define risk scoring rules
- [ ] Implement contract verification checks
- [ ] Build risk score UI component
- [ ] Add address intelligence features

---

## Testing Strategy

### Manual Testing
1. **Safe Import**: Import a real Safe on mainnet/testnet
2. **Transaction Decoding**: Verify correct decoding of common contracts (Uniswap, ERC20, etc.)
3. **Signing Flow**: Test with Ledger/Trezor via WalletConnect
4. **Simulation**: Compare Tenderly results with actual execution

### Automated Testing
- Unit tests for decoder logic
- Integration tests for Safe SDK interactions
- E2E tests for critical flows (Playwright)

### Test Safes
- Create test Safe on Sepolia for development
- Use existing mainnet Safes (read-only) for decoding tests

---

## Open Questions

1. **Tenderly vs. alternatives**: Tenderly has rate limits on free tier. Consider alternatives like Alchemy Simulation API or building custom simulation.

2. **Backend for signer reputation**: Tracking signer history across sessions requires persistence. Options:
   - Local storage (simple, per-device)
   - Supabase/Firebase (free tier, cross-device)
   - Custom backend (most control)

3. **Notification system**: For multi-sig coordination, may need backend for email/push notifications.

---

## User Decisions

Based on planning session:

| Decision | Choice |
|----------|--------|
| Platform | Web application |
| Safe Integration | Build on Safe SDK (existing contracts, new UI) |
| Frontend Stack | React/Next.js |
| Hardware Wallet Support | WalletConnect for flexibility |
| Initial Chains | Ethereum + major L2s (Arbitrum, Optimism, Base) |
| Development Approach | Phased with milestones |
| Team | Solo developer |
| Design Approach | Iterate as we build |
| Project Name | SafeSafe |
| License | Closed source |

---

## Summary

SafeSafe is a phased project starting with core Safe viewing and transaction decoding, building up to a comprehensive signing experience with risk scoring. The MVP focuses on:

1. Importing existing Safes
2. Decoding transactions to human-readable format (using Sourcify + 4byte)
3. Basic transaction viewing and execution

Phase 2 adds the signature UX differentiation (step-by-step wizard), and Phase 3 adds intelligence features (risk scoring, signer reputation).

The architecture uses Safe's official SDK and leverages external APIs (Sourcify, 4byte, Tenderly) to enrich the user experience without requiring a custom backend initially.

