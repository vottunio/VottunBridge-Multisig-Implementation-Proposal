# Milestone 3: Technical Documentation

This document contains detailed technical documentation for Milestone 3 implementation, including component architecture, API endpoints, code examples, database schemas, and technology stack details.

**Related**: See `M3.md` for the deliverable summary.

---

## Table of Contents

1. [Detailed Component Implementation](#detailed-component-implementation)
2. [Backend Implementation Details](#backend-implementation-details)
3. [Services Layer Details](#services-layer-details)
4. [User Journey Examples](#user-journey-examples)
5. [Component Hierarchy](#component-hierarchy)
6. [API Endpoint Reference](#api-endpoint-reference)
7. [Technology Stack](#technology-stack)
8. [Database Schema](#database-schema)
9. [Security Considerations](#security-considerations)

---

## Detailed Component Implementation

### 1. Admin Dashboard

**File**: `src/pages/Admin.tsx`

**Detailed Features**:
- ✅ Dual-tab interface (EVM Bridge / Qubic Bridge)
- ✅ Role-based access control (Admin/Manager permissions)
- ✅ Automatic tab selection based on user role
- ✅ Real-time role display badge (e.g., "EVM Admin | Qubic Manager")
- ✅ Permission verification with redirect if unauthorized
- ✅ Modal management for all admin operations
- ✅ Auto-refresh proposals after operations

**User Flow**:
```
1. User connects wallet (MetaMask + Qubic Snap)
2. System checks roles on both networks
3. Dashboard shows available tabs (EVM/Qubic) based on permissions
4. Default tab = highest privilege network (Qubic > EVM)
5. If no admin/manager role → redirect to home
```

### 2. EVM Proposals Management

**File**: `src/components/admin/EvmProposalsList.tsx`

**Detailed Features**:
- ✅ Pending/Executed tabs with proposal counts
- ✅ Real-time approval tracking (X / Y approvals)
- ✅ Direct blockchain interaction (wagmi hooks)
- ✅ Approve & Execute buttons with loading states
- ✅ Role-based filtering (Managers only see operator proposals)
- ✅ Auto-refresh after transactions confirmed (2s delay)
- ✅ Proposal type badges with color coding
- ✅ Transaction status toasts (success/error)
- ✅ Address truncation for UX
- ✅ Duplicate proposal ID detection

**Supported Proposal Types**:
- Emergency Pause / Unpause
- Set Base Fee
- Set Fee Recipient
- Emergency Token/Ether Withdraw
- Add/Remove Admin
- Add/Remove Manager
- Add/Remove Operator
- Set Admin/Manager Threshold

**Workflow**:
```
Pending Tab:
├─ Show proposals with < required approvals
├─ [Approve] button → writeContract(approveProposal)
└─ [Execute] button (only if approvals >= threshold)

Executed Tab:
└─ Show executed/canceled proposals (read-only)
```

### 3. Create EVM Proposal Modal

**File**: `src/components/admin/CreateEvmProposalModal.tsx`

**Detailed Features**:
- ✅ Form-based proposal creation
- ✅ Dynamic fields based on proposal type
- ✅ Input validation (address format, amounts)
- ✅ Backend API integration
- ✅ Success callback → refresh parent list
- ✅ 2-second delay for backend indexing
- ✅ Loading states during submission
- ✅ Error handling with user-friendly messages

**Proposal Creation Flow**:
```
1. Admin clicks "Create Proposal"
2. Modal opens with proposal type selector
3. Form shows relevant fields (e.g., address for Add Manager)
4. Admin fills form + submits
5. Backend creates proposal → Smart Contract
6. Modal closes, proposals list refreshes (with delay)
```

---

## Backend Implementation Details

### Complete REST API Endpoints

**Files**:
- `src/evm/proposalEvm.go`
- `src/service/proposalsEvm.go`
- `src/controller/proposalsControllerEvm.go`
- `src/controller/RestServer.go`

**Complete REST API Endpoints**:
```go
// Role Management
POST /qubicbridge/v1/evm/add-manager-evm
POST /qubicbridge/v1/evm/remove-manager-evm
POST /qubicbridge/v1/evm/add-admin-evm
POST /qubicbridge/v1/evm/remove-admin-evm
POST /qubicbridge/v1/evm/add-operator-evm
POST /qubicbridge/v1/evm/remove-operator-evm

// Configuration Management
POST /qubicbridge/v1/evm/set-basefee-evm
POST /qubicbridge/v1/evm/set-admin-threshold-evm
POST /qubicbridge/v1/evm/set-manager-threshold-evm
POST /qubicbridge/v1/evm/set-fee-recipient-evm

// Emergency Functions
POST /qubicbridge/v1/evm/emergency-pause-evm
POST /qubicbridge/v1/evm/remove-pause-evm
POST /qubicbridge/v1/evm/emergency-token-withdraw-evm
POST /qubicbridge/v1/evm/emergency-ether-withdraw-evm

// Proposal Lifecycle Management
GET  /qubicbridge/v1/evm/pending-proposals
GET  /qubicbridge/v1/evm/fetch-proposals-evm
GET  /qubicbridge/v1/evm/fetch-single-proposals-evm?proposeId=X
GET  /qubicbridge/v1/evm/get-proposals?filter=STATUS
POST /qubicbridge/v1/evm/approve-proposals
POST /qubicbridge/v1/evm/execute-proposals
```

### Core Functions Implementation

```go
// Proposal Type Decoding (evm/proposalEvm.go:86-119)
DecodeProposalType(data []byte) string
  → Decodes function selector from proposal data
  → Supports 14 proposal types (emergency_pause, add_manager, etc.)
  → Maps function signature hash to human-readable type

// Proposal Creation Functions
EmergencyPauseEvm(adminKey) → Creates emergency pause proposal
EmergencyUnPauseEvm(adminKey) → Creates unpause proposal
SetBaseFeeEvm(baseFee, adminKey) → Creates base fee change proposal
SetAdminThresholdEvm(threshold, adminKey) → Creates admin threshold proposal
SetManagerThresholdEvm(threshold, adminKey) → Creates manager threshold proposal
ChangeFeeRecipientEVM(recipient, adminKey) → Creates fee recipient change proposal
EmergencyTokenWithdrawEVM(token, recipient, amount, adminKey) → Token withdrawal
EmergencyEtherWithdrawEVM(recipient, adminKey) → ETH withdrawal
CreateProposalForAddOperator(address) → Add operator proposal
CreateProposalForRemoveOperator(address) → Remove operator proposal

// Proposal Execution Functions
ApproveProposalEvm(proposalId, adminKey, proposalType) → Approve proposal
ExecuteProposalEvm(proposalId, proposalType) → Execute approved proposal
GetSingleProposalFromEVM(proposalId) → Fetch proposal from smart contract
GetAllPendingProposals() → Get all pending proposal IDs
```

### Advanced Features

- ✅ **Duplicate proposal prevention**: `getExistingProposalByType()` checks for active proposals
- ✅ **Asynchronous transaction monitoring**: `WaitForTransactionToApprove()` goroutine
- ✅ **Hybrid data source**: Merges database + EVM contract data
- ✅ **Auto-approval on creation**: Creator automatically approves proposal
- ✅ **Database persistence**: Proposals stored in `bridge_proposals` table
- ✅ **Event parsing**: Extracts `ProposalApproved` events from transaction logs
- ✅ **Threshold validation**: Checks admin/manager thresholds before execution
- ✅ **Role-based validation**: Verifies caller has required role (admin/manager)

### Data Layer

```go
SaveProposal(proposal) → Store proposal in database
UpdateProposalDetails(proposalId, status, executor) → Update proposal state
FetchAllProposals() → Get all proposals
FetchProposalByID(proposalId) → Get single proposal
FetchAllProposalsByStatus(status) → Filter by PENDING/APPROVED/EXECUTED
GetProposalDetails(proposalId) → Get type and status
GetProposalByRoleRequired(type) → Check for duplicate proposals
```

### DTOs Defined

```go
type BridgeProposal struct {
    ID            int64
    ProposalID    string          // Unique proposal hash
    Proposer      string          // Creator address
    ContractData  json.RawMessage // Encoded function call
    RoleRequired  string          // Required role hash
    Status        string          // PENDING/APPROVED/EXECUTED/PARTIAL
    ApprovalCount int64           // Current approvals
    Executor      string          // Executor address
    DataSource    string          // "database", "evm_contract", "merged"
    ProposalType  string          // Decoded human-readable type
}

// Request DTOs
type ApproveProposalsDTO
type BaseFeeProposalsDTO
type ChangeFeeRecipientProposalsDTO
type EmergencyTokenWithdrawDTO
type EmergencyEtherWithdrawDTO
type AddEvmOperatorRequestDTO
type AddEvmThresholdRequestDTO
```

### Proposal State Machine

```
1. CREATE → ProposeAction(data, roleRequired)
   ↓ (Creator auto-approves, ApprovalCount = 1)
2. PENDING → Wait for approvals (ApprovalCount < Threshold)
   ↓ (Admin/Manager calls ApproveProposal)
3. APPROVED → ApprovalCount >= Threshold
   ↓ (Any admin calls ExecuteProposal)
4. EXECUTED → Proposal executed, state immutable
```

### Workflow Example (Set Admin Threshold)

```go
// Step 1: Admin creates proposal
POST /evm/set-admin-threshold-evm
Body: { "evmThreshold": 3, "adminKey": "0x..." }
  ↓
SetAdminThresholdEvm() → Creates proposal with encoded data
  ↓ Keccak256("setAdminThreshold(uint256)") + padded(3)
ProposeAction() → Submits to smart contract
  ↓ ProposalCreated event emitted
WaitForTransactionToApprove() → Monitors transaction
  ↓ Stores in database: proposal_id, status=PENDING, approval_count=1

// Step 2: Admin 2 approves
POST /evm/approve-proposals
Body: { "proposalID": "0xabc123...", "adminKey": "0x..." }
  ↓
ApproveProposalEvm() → Calls bridge.ApproveProposal()
  ↓ ProposalApproved event emitted
Database updated: approval_count=2, status=APPROVED

// Step 3: Execute when threshold met (2/2)
POST /evm/execute-proposals
Body: { "proposalID": "0xabc123..." }
  ↓
ExecuteProposalEvm() → Validates threshold, calls ExecuteProposal()
  ↓ Proposal executed on-chain, admin threshold changed
Database updated: status=EXECUTED
```

---

## Services Layer Details

### EVM Admin Service

**File**: `src/services/evmAdminService.ts`

**Endpoints Integrated**:
```typescript
// Admin proposals
addAdmin(data: AddAdminRequest)
  → POST /qubicbridge/v1/evm/add-admin-evm

addManager(data: AddManagerRequest)
  → POST /qubicbridge/v1/evm/add-manager-evm

removeManager(data: RemoveManagerRequest)
  → POST /qubicbridge/v1/evm/remove-manager-evm

emergencyPause()
  → POST /qubicbridge/v1/evm/emergency-pause-evm

removePause()
  → POST /qubicbridge/v1/evm/remove-pause-evm

// Manager proposals
addOperator(data: AddOperatorRequest)
  → POST /qubicbridge/v1/evm/add-operator-evm

removeOperator(data: RemoveOperatorRequest)
  → POST /qubicbridge/v1/evm/remove-operator-evm

// Proposal management
getPendingProposals()
  → GET /qubicbridge/v1/evm/pending-proposals

getExecutedProposals()
  → GET /qubicbridge/v1/evm/all-proposals-evm (filter executed)

getSingleProposal(proposalId)
  → GET /qubicbridge/v1/evm/fetch-single-proposals-evm?proposeId=X

approveProposal(data: ApproveProposalRequest)
  → POST /qubicbridge/v1/evm/approve-proposals

executeProposal(data: ExecuteProposalRequest)
  → POST /qubicbridge/v1/evm/execute-proposals
```

**Type Mapping**:
```typescript
{
  'emergency_pause': 'Emergency Pause',
  'emergency_unpause': 'Emergency Unpause',
  'set_base_fee': 'Set Base Fee',
  'set_fee_recipient': 'Set Fee Recipient',
  'emergency_withdraw_token': 'Emergency Withdraw Token',
  'emergency_withdraw_ether': 'Emergency Withdraw Ether',
  'add_admin': 'Add Admin',
  'remove_admin': 'Remove Admin',
  'add_manager': 'Add Manager',
  'remove_manager': 'Remove Manager',
  'set_admin_threshold': 'Set Admin Threshold',
  'set_manager_threshold': 'Set Manager Threshold',
  'add_operator': 'Add Operator',
  'remove_operator': 'Remove Operator',
}
```

### Qubic Proposal Service

**File**: `src/services/qubicProposalService.ts`

**Endpoints Integrated**:
```typescript
getSmartContractDetails()
  → GET /qubicbridge/v1/getSmartContractDetails

getProposal(proposalId)
  → GET /qubicbridge/v1/get-proposal?proposalId=X

getAllProposals()
  → Fetches contract details + iterates all proposals

getActiveProposals()
  → Filters active && !executed

createSetAdminProposal(data)
  → POST /qubicbridge/v1/create-proposal-set-admin

createAddManagerProposal(data)
  → POST /qubicbridge/v1/create-proposal-add-manager

createRemoveManagerProposal(data)
  → POST /qubicbridge/v1/create-proposal-remove-manager

approveProposal(data)
  → POST /qubicbridge/v1/approve-proposal
```

**Data Model**:
```typescript
interface Proposal {
  proposalId: number;
  proposalType: ProposalType;
  proposalTypeString: "SET_ADMIN" | "ADD_MANAGER" | "REMOVE_MANAGER";
  targetAddress: string;       // Qubic address
  oldAddress: string;          // For SET_ADMIN only
  amount: number;
  approvalsCount: number;
  requiredApprovals: number;   // 2 (fixed)
  executed: boolean;
  active: boolean;
}
```

### useUserRole Hook Details

**File**: `src/hooks/useUserRole.ts`

**EVM Role Detection**:
```typescript
// Admin (hasRole check)
useReadContract({
  functionName: "hasRole",
  args: [DEFAULT_ADMIN_ROLE, address]
})

// Manager (getManagers list check)
useReadContract({
  functionName: "getManagers",
  args: []
})
// Then check if address in list

// Operator (getOperators list check)
useReadContract({
  functionName: "getOperators",
  args: []
})
```

**Return Type**:
```typescript
{
  // EVM roles
  isAdmin: boolean,
  isManager: boolean,
  isOperator: boolean,
  // Qubic roles
  isQubicAdmin: boolean,
  isQubicManager: boolean,
  isQubicOperator: boolean,
  // Combined
  hasAdminAccess: boolean,
  hasManagerAccess: boolean,
  // Utils
  isLoading: boolean,
  refetch: () => void,
}
```

---

## User Journey Examples

### Journey 1: EVM Admin Creates & Approves Proposal

```
1. Admin 1 (0x464800...) connects MetaMask
   → Dashboard shows "EVM Admin" badge
   → EVM Bridge tab active by default

2. Admin 1 clicks "Create Proposal"
   → Modal opens

3. Admin 1 selects "Add Manager"
   → Enters manager address: 0x123456...
   → Submits form

4. Backend creates proposal
   → Transaction confirmed on Base Sepolia
   → Modal closes

5. Proposals list refreshes (2s delay)
   → New proposal appears in "Pending" tab
   → Shows "Approvals: 1 / 2" (creator auto-approved)

6. Admin 2 (0x0e60B8...) connects wallet
   → Sees same proposal
   → Clicks [Approve]

7. MetaMask popup
   → Admin 2 confirms transaction
   → "Approving..." button shows loading

8. Transaction confirmed
   → Approvals: 2 / 2
   → [Execute] button appears (green)

9. Admin 1 or Admin 2 clicks [Execute]
   → Transaction submitted
   → Manager added to contract

10. Proposal moves to "Executed" tab
    → New manager can now create operator proposals
```

### Journey 2: Manager Adds Operator (Qubic)

```
1. Manager (QUBIC_ADDRESS) connects via Snap
   → Dashboard shows "Qubic Manager" badge
   → Qubic Bridge tab active

2. Manager clicks "New Proposal"
   → Modal shows proposal types
   → ADD_MANAGER option available

3. Manager selects "Add Manager"
   → Enters Qubic address (60 chars uppercase)
   → Validates format

4. Submits proposal
   → Backend calls Qubic smart contract
   → CreateProposal transaction broadcast

5. Proposal appears in "Active" tab
   → Shows "Approvals: 1 / 2"
   → Manager approval auto-counted

6. Admin 2 approves via backend API
   → /qubicbridge/v1/approve-proposal
   → Approvals: 2 / 2

7. Proposal auto-executes (threshold met)
   → Manager added to contract
   → Moves to "Completed" tab
```

### Journey 3: Emergency Pause (EVM)

```
1. Admin 1 detects exploit
   → Navigates to EVM Bridge tab
   → Clicks "Create Proposal"

2. Selects "Emergency Pause"
   → No additional fields required
   → Submits

3. Backend creates emergency_pause proposal
   → Proposal created with 1 approval

4. Admin 1 contacts Admin 2 (off-chain)
   → Admin 2 logs in
   → Approves immediately

5. Threshold met (2/2)
   → Admin 1 executes
   → Bridge paused on-chain

6. All bridge operations halt
   → Users see "Bridge Paused" message
   → Admins investigate exploit

7. After fix, Admin 1 creates "Remove Pause" proposal
   → Admin 2 approves
   → Bridge resumes operations
```

---

## Component Hierarchy

```
Admin Dashboard (src/pages/Admin.tsx)
│
├─ EVM Bridge Tab
│  ├─ EvmActionsPanel
│  │  └─ Create Proposal Button → CreateEvmProposalModal
│  └─ EvmProposalsList
│     ├─ Pending Tab
│     │  ├─ Proposal Cards (Approve/Execute buttons)
│     │  └─ Empty State
│     └─ Executed Tab
│        ├─ Proposal Cards (Read-only)
│        └─ Empty State
│
└─ Qubic Bridge Tab
   ├─ QubicRolesPanel (Contract details)
   ├─ Action Cards Grid
   │  ├─ Create Proposal → CreateProposalModal
   │  ├─ Manage Admins → AdminManagementModal
   │  ├─ Manage Orders → OrderManagementModal
   │  └─ Contract Details → ContractDetailsModal
   └─ QubicProposalsList
      ├─ Active Tab
      │  ├─ Proposal Cards (Approve button)
      │  └─ Empty State
      └─ Completed Tab
         ├─ Proposal Cards (Read-only)
         └─ Empty State
```

---

## State Flow

```
User Action → Component Event Handler → Service Call → Backend API → Smart Contract
                                                                            ↓
                                                                      Blockchain
                                                                            ↓
User sees update ← Component Re-render ← State Update ← Wagmi Hook ← Event Emission
```

---

## API Endpoint Reference

### EVM Proposal Endpoints (Complete Implementation)

```
// Role Management (6 endpoints)
POST /qubicbridge/v1/evm/add-admin-evm
POST /qubicbridge/v1/evm/remove-admin-evm
POST /qubicbridge/v1/evm/add-manager-evm
POST /qubicbridge/v1/evm/remove-manager-evm
POST /qubicbridge/v1/evm/add-operator-evm
POST /qubicbridge/v1/evm/remove-operator-evm

// Configuration Management (5 endpoints)
POST /qubicbridge/v1/evm/set-basefee-evm
POST /qubicbridge/v1/evm/set-admin-threshold-evm
POST /qubicbridge/v1/evm/set-manager-threshold-evm
POST /qubicbridge/v1/evm/set-fee-recipient-evm

// Emergency Functions (4 endpoints)
POST /qubicbridge/v1/evm/emergency-pause-evm
POST /qubicbridge/v1/evm/remove-pause-evm
POST /qubicbridge/v1/evm/emergency-token-withdraw-evm
POST /qubicbridge/v1/evm/emergency-ether-withdraw-evm

// Proposal Lifecycle (6 endpoints)
GET  /qubicbridge/v1/evm/pending-proposals
GET  /qubicbridge/v1/evm/fetch-proposals-evm
GET  /qubicbridge/v1/evm/fetch-single-proposals-evm?proposeId=X
GET  /qubicbridge/v1/evm/get-proposals?filter=STATUS
POST /qubicbridge/v1/evm/approve-proposals
POST /qubicbridge/v1/evm/execute-proposals

// TOTAL: 21 EVM endpoints fully implemented
```

### Qubic Proposal Endpoints

```
GET  /qubicbridge/v1/getSmartContractDetails
GET  /qubicbridge/v1/get-proposal?proposalId=X
POST /qubicbridge/v1/create-proposal-set-admin
POST /qubicbridge/v1/create-proposal-add-manager
POST /qubicbridge/v1/create-proposal-remove-manager
POST /qubicbridge/v1/approve-proposal
```

---

## Technology Stack

### Frontend Framework
- React 18 + TypeScript
- Build Tool: Vite
- Styling: Tailwind CSS
- UI Components: Custom components + shadcn/ui

### State Management
- Zustand (global state)
- React Query (server state)
- Wagmi (blockchain state)

### Wallet Connection
- RainbowKit + MetaMask Snap
- Notifications: Sonner (toast library)
- Routing: TanStack Router
- Type Safety: TypeScript 5.x + Zod (schema validation)

### Backend Framework
- Language: Go 1.21+
- Web Framework: Gorilla Mux (HTTP router)
- Blockchain Client: go-ethereum (geth) v1.13+
- Database: MySQL 8.0 with sqlx
- Caching: Redis (for event pub/sub)
- gRPC: For internal service communication

### Smart Contract Interaction
- ABI Binding: go-ethereum/accounts/abi/bind
- Contract: QubicBridge (custom bindings)
- Event Parsing: ProposalCreated, ProposalApproved, ProposalExecuted
- Transaction Management: Context-based with timeouts

---

## Database Schema

```sql
-- bridge_proposals table
CREATE TABLE bridge_proposals (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  proposal_id VARCHAR(66) UNIQUE NOT NULL,  -- 0x + 64 hex chars
  proposer VARCHAR(42) NOT NULL,
  contract_data JSON,
  role_required VARCHAR(66),
  status ENUM('PENDING', 'APPROVED', 'EXECUTED', 'PARTIAL'),
  approval_count INT DEFAULT 1,
  executor VARCHAR(42),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_proposal_id (proposal_id),
  INDEX idx_status (status)
);
```

---

## Security Considerations

### Implemented Protections ✅

1. **Role-Based Access Control**
   - ✅ Frontend checks roles before rendering admin panel
   - ✅ Backend validates roles before accepting proposals
   - ✅ Smart contract enforces multisig on execution

2. **Transaction Validation**
   - ✅ Wagmi validates network (must be Base Sepolia)
   - ✅ Smart contract validates caller has role
   - ✅ Backend validates addresses before creating proposals

3. **Input Validation**
   - ✅ Address format validation (EVM: 0x + 40 hex, Qubic: 60 uppercase)
   - ✅ Amount validation (positive numbers)
   - ✅ Required fields enforcement

4. **Error Handling**
   - ✅ Transaction rejections handled gracefully
   - ✅ Network errors shown as toasts
   - ✅ Invalid inputs blocked before submission
   - ✅ Timeout handling for long transactions (5 minutes)

### Frontend-Specific Risks ⚠️

**Risk**: User approves malicious proposal
- **Mitigation**: Proposal details shown clearly before approval
- **Recommendation**: Add proposal preview modal before approve

**Risk**: XSS via address input
- **Mitigation**: React escapes all user input by default
- **Status**: Safe

**Risk**: Phishing (fake admin panel)
- **Mitigation**: Domain verification required, SSL certificate mandatory, wallet signature verification
- **Recommendation**: Add domain binding in smart contract

---

## Wagmi Integration Example

**Files**:
- `src/wagmi.config.ts`
- `src/hooks/useMetaMask.ts`

**Example Usage**:
```typescript
const { writeContract, data: hash, isPending } = useWriteContract();
const { isLoading: isConfirming, isSuccess } = useWaitForTransactionReceipt({ hash });

// Approve proposal
writeContract({
  address: BRIDGE_ADDRESS,
  abi: BRIDGE_ABI,
  functionName: "approveProposal",
  args: [proposalId]
});

// Listen for success
useEffect(() => {
  if (isSuccess) {
    toast.success("Proposal approved!");
    loadProposals();
  }
}, [isSuccess]);
```

**EVM Bridge Contract**:
- Testnet: `0xbC79b4a96186b0AFE09Ee83830e2Fb30E14d5Ddc` (Base Sepolia)
- Mainnet: TBD

---

## Known Limitations

### EVM Contract Limitations:
- ⚠️ `getProposal()` returns raw bytes (no ABI decoding in SC)
  - **Workaround**: Backend stores proposal metadata in database
  - **Solution**: Frontend relies on backend for human-readable data

### Qubic Contract Limitations:
- ⚠️ Threshold hardcoded to 2 (not configurable without redeployment)
  - **Status**: As designed in M1, no change needed
- ⚠️ Max 3 admins (hardcoded array size)
  - **Status**: As designed in M1, no change needed

### UX Improvements (Post-M3):
- 🔄 Add proposal cancellation UI (function exists in SC)
- 🔄 Add proposal history timeline (created → approved → executed)
- 🔄 Add WebSocket for real-time proposal updates
- 🔄 Add CSV export for proposals
- 🔄 Add multi-language support (i18n)

---

## Error Handling & Validation

**Backend**:
- ✅ Invalid private key format detection
- ✅ Transaction revert detection (receipt.Status == 0)
- ✅ Duplicate proposal prevention
- ✅ Threshold validation before execution
- ✅ Proposal existence checks (proposer != 0x0)
- ✅ Status validation (prevent re-execution)
- ✅ Event parsing fallback (if logs missing)

**Performance Optimizations**:
- ✅ Goroutines for async transaction monitoring
- ✅ Database caching of proposal metadata
- ✅ Batch fetching from smart contract
- ✅ Indexed database queries by proposal_id and status

---

*This document contains detailed technical specifications. For the deliverable summary, see `M3.md`.*

