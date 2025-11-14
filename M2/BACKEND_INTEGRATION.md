# Backend Integration Guide - Qubic Bridge Multisig System

This guide explains how to integrate the Qubic Bridge's multisig system into Go backend services that manage bridge operations (operator nodes, monitoring services, admin tools).

## Table of Contents
- [Overview](#overview)
- [Backend Service Architecture](#backend-service-architecture)
- [Smart Contract Functions Reference](#smart-contract-functions-reference)
- [Operator Node Integration](#operator-node-integration)
- [Admin Service Integration](#admin-service-integration)
- [Manager Service Integration](#manager-service-integration)
- [Event Monitoring Requirements](#event-monitoring-requirements)
- [Error Handling](#error-handling)
- [Security Considerations](#security-considerations)

---

## Overview

The Qubic Bridge uses a **multisig approval system** for administrative and manager functions:

- **Admin functions** (pause, setBaseFee, emergency withdrawals) require **N admin approvals**
- **Manager functions** (addOperator, removeOperator) require **M manager approvals**
- **Operator functions** (confirmOrder, revertOrder, executeOrder) are **single-signature** (no multisig)

### Backend Services Required

Your Go backend needs to implement three distinct services:

1. **Operator Service** - High frequency, single-signature transactions
2. **Manager Service** - Low frequency, multisig transactions for operator management
3. **Admin Service** - Very low frequency, multisig transactions for critical actions

---

## Backend Service Architecture

### Service Responsibilities

**Operator Service:**
- Monitor `OrderCreated` events
- Call `confirmOrder()` after verifying Qubic network transfer
- Call `revertOrder()` if Qubic transfer fails
- Call `executeOrder()` for push orders (Qubic → Ethereum)
- **No multisig required** - direct execution

**Manager Service:**
- Create proposals for adding/removing operators
- Monitor pending manager proposals
- Approve manager proposals from other managers
- Execute approved manager proposals
- **Requires multisig approval**

**Admin Service:**
- Create proposals for emergency actions, fee changes, role changes
- Monitor pending admin proposals
- Approve admin proposals from other admins
- Execute approved admin proposals
- **Requires multisig approval**

### Key Architecture Decisions

- **Operator nodes should NOT implement multisig** - would destroy UX and bridge speed
- **Admin/Manager services should run separately** from operator nodes
- **Multiple admin/manager wallets** must be configured across different servers/locations
- **Event monitoring is critical** - all services need real-time event subscriptions

---

## Smart Contract Functions Reference

### Core Multisig Functions (Admin & Manager)

**`proposeAction(bytes data, bytes32 roleRequired) → bytes32 proposalId`**
- Creates a new proposal
- `data`: ABI-encoded function call to execute later
- `roleRequired`: `DEFAULT_ADMIN_ROLE` (0x00...00) or `MANAGER_ROLE` (keccak256("MANAGER_ROLE"))
- Returns: `proposalId` (bytes32 hash)
- Emits: `ProposalCreated` event

**`approveProposal(bytes32 proposalId)`**
- Approves an existing proposal
- Caller must have the required role (admin or manager)
- Each account can only approve once
- Emits: `ProposalApproved` event

**`executeProposal(bytes32 proposalId) → bool success`**
- Executes a proposal that has met approval threshold
- Anyone can call this once threshold is met
- Proposal must not be executed or cancelled
- Emits: `ProposalExecuted` event on success

**`cancelProposal(bytes32 proposalId)`**
- Cancels a proposal (proposer or admin only)
- Prevents execution even if threshold is met
- Emits: `ProposalCancelled` event

### View Functions (Read-Only)

**`getPendingProposals() → bytes32[] proposalIds`**
- Returns array of all pending (not executed, not cancelled) proposals
- Use for monitoring loop to find proposals needing approval

**`getProposal(bytes32 proposalId) → Proposal`**
- Returns full proposal details:
  - `bytes data` - Encoded function call
  - `bytes32 roleRequired` - Admin or Manager role
  - `address proposer` - Who created it
  - `uint256 approvalCount` - Current approval count
  - `bool executed` - Whether executed
  - `bool cancelled` - Whether cancelled

**`hasApprovedProposal(bytes32 proposalId, address account) → bool`**
- Check if specific address has approved a proposal
- Use before attempting to approve

**`adminThreshold() → uint256`**
- Returns number of admin approvals required
- Set at deployment time

**`managerThreshold() → uint256`**
- Returns number of manager approvals required
- Set at deployment time

### Operator Functions (No Multisig)

**`confirmOrder(uint256 orderId, uint256 feePct)`**
- Confirms a pull order (Ethereum → Qubic)
- Burns tokens from bridge contract
- Marks order as done
- Fees sent to global `feeRecipient` address
- Emits: `OrderConfirmed` event

**`revertOrder(uint256 orderId, uint256 feePct)`**
- Reverts a failed pull order
- Refunds tokens to user
- Marks order as done
- Fees sent to global `feeRecipient` address
- Emits: `OrderReverted` event

**`executeOrder(uint256 originOrderId, string originAccount, address destinationAccount, uint256 amount, uint256 feePct)`**
- Executes a push order (Qubic → Ethereum)
- Mints tokens to destination account
- Stores originOrderId to prevent re-execution
- Fees sent to global `feeRecipient` address
- Emits: `OrderExecuted` event

### Admin Functions (Require Multisig via Proposals)

- `addAdmin(address newAdmin)` - Add a new admin (max 3)
- `removeAdmin(address admin)` - Remove an admin
- `addManager(address newManager)` - Add manager (max 3)
- `removeManager(address manager)` - Remove manager
- `setBaseFee(uint256 _baseFee)` - Change base fee (basis points)
- `setFeeRecipient(address newFeeRecipient)` - Change treasury address that receives all fees
- `setAdminThreshold(uint256 newThreshold)` - Change admin approval threshold
- `setManagerThreshold(uint256 newThreshold)` - Change manager approval threshold
- `emergencyPause()` - Pause all bridge operations
- `emergencyUnpause()` - Resume bridge operations
- `emergencyTokenWithdraw(address tokenAddress, address recipient, uint256 amount)` - Withdraw tokens
- `emergencyEtherWithdraw(address recipient)` - Withdraw all ETH

### Manager Functions (Require Multisig via Proposals)

- `addOperator(address newOperator)` - Add new operator
- `removeOperator(address operator)` - Remove operator

---

## Operator Node Integration

### Required Adaptations for Go Backend

Your existing Go operator service needs to **remain unchanged** for the most part. Operator functions do NOT use multisig.

### What Stays the Same

- Direct transaction signing and submission
- Event monitoring for `OrderCreated`
- Logic for confirming/reverting orders
- Push order execution

### Contract Interaction Pattern

For each operator function, your Go code should:

1. **Build transaction** - Create transaction data for `confirmOrder`, `revertOrder`, or `executeOrder`
2. **Sign transaction** - Use operator private key (single signature)
3. **Send transaction** - Submit to Ethereum network
4. **Wait for receipt** - Confirm transaction was mined
5. **Handle errors** - Retry logic for network issues

### Events to Monitor

- **`OrderCreated(uint256 orderId, address user, string qubicAddress, uint256 amount)`**
  - Triggers operator to process pull order
  - Operator should verify Qubic network, then call `confirmOrder()` or `revertOrder()`

### Error Cases to Handle

- `QubicBridge: Only operator` - Caller is not an operator (should not happen if properly configured)
- `QubicBridge: Order already done` - Order already processed (check state before calling)
- `QubicBridge: Order does not exist` - Invalid orderId (should not happen with event monitoring)
- `QubicBridge: Push order already executed` - originOrderId already used (prevent duplicate processing)

---

## Admin Service Integration

### New Service Component Required

You need to create a **separate admin service** that runs independently from operator nodes.

### Admin Service Responsibilities

1. **Propose Actions** - Create proposals for admin functions
2. **Monitor Proposals** - Poll `getPendingProposals()` for admin proposals needing approval
3. **Approve Proposals** - Call `approveProposal()` for valid proposals
4. **Execute Proposals** - Call `executeProposal()` once threshold met
5. **Notify Other Admins** - Send alerts when proposals need approval

### Workflow for Admin Actions

**Example: Change Base Fee to 3%**

1. **Admin 1 proposes:**
   - Encode function call: `setBaseFee(300)` → ABI-encoded bytes
   - Call: `proposeAction(encodedData, DEFAULT_ADMIN_ROLE)`
   - Extract `proposalId` from `ProposalCreated` event
   - Store `proposalId` in database
   - Notify other admins (email/Slack/etc)

2. **Admin 2+ approve:**
   - Monitor: `getPendingProposals()` returns list of pending proposals
   - Check: `hasApprovedProposal(proposalId, admin2Address)` → false
   - Call: `approveProposal(proposalId)`
   - Repeat for each additional admin until threshold met

3. **Anyone executes:**
   - Check: `getProposal(proposalId).approvalCount >= adminThreshold()`
   - Call: `executeProposal(proposalId)`
   - Transaction executes `setBaseFee(300)` on success

### Functions Requiring Admin Proposals

Every call to these functions must go through the proposal flow:

- **addAdmin()** - Add new admin (max 3 admins)
- **removeAdmin()** - Remove an admin
- **addManager()** - Grant manager privileges (max 3 managers)
- **removeManager()** - Revoke manager privileges
- **setBaseFee()** - Change fee percentage (max 100%)
- **emergencyPause()** - Stop all bridge operations
- **emergencyUnpause()** - Resume operations
- **emergencyTokenWithdraw()** - Withdraw tokens (recovery)
- **emergencyEtherWithdraw()** - Withdraw ETH (recovery)
- **setAdminThreshold()** - Change admin approval threshold
- **setManagerThreshold()** - Change manager approval threshold

### ABI Encoding in Go

For each admin function, you need to ABI-encode the function call:

- Use `abigen` generated Go bindings
- Call the appropriate method on the contract instance to build transaction data
- Extract the `data` field (ABI-encoded bytes)
- Pass this to `proposeAction()`

### Monitoring Loop for Admin Service

Your admin service should run a continuous loop (e.g., every 60 seconds):

1. Call `getPendingProposals()` to get all pending proposal IDs
2. For each proposal ID:
   - Call `getProposal(proposalId)` to get details
   - Check if `roleRequired == DEFAULT_ADMIN_ROLE` (admin proposal)
   - Check if `hasApprovedProposal(proposalId, thisAdminAddress)` == false
   - If not approved, decode the proposal data and decide whether to approve
   - Optionally auto-approve based on configuration
   - Or send notification for manual approval
3. For approved proposals meeting threshold:
   - Call `executeProposal(proposalId)`
   - Log the execution result

---

## Manager Service Integration

### New Service Component Required

Create a **separate manager service** similar to admin service but for operator management.

### Manager Service Responsibilities

1. **Propose Operator Changes** - Add/remove operators
2. **Monitor Manager Proposals** - Poll for manager proposals
3. **Approve Manager Proposals** - Approve proposals from other managers
4. **Execute Manager Proposals** - Execute once threshold met

### Workflow for Manager Actions

**Example: Add New Operator**

1. **Manager 1 proposes:**
   - Encode: `addOperator(newOperatorAddress)` → bytes
   - Call: `proposeAction(encodedData, MANAGER_ROLE)`
   - Store `proposalId`
   - Notify other managers

2. **Manager 2+ approve:**
   - Monitor: `getPendingProposals()`
   - Filter: `getProposal(proposalId).roleRequired == MANAGER_ROLE`
   - Call: `approveProposal(proposalId)`

3. **Execute:**
   - Check: `approvalCount >= managerThreshold()`
   - Call: `executeProposal(proposalId)`

### Functions Requiring Manager Proposals

- **addOperator(address)** - Add new operator to process orders
- **removeOperator(address)** - Remove operator access

### Manager Role Constant

`MANAGER_ROLE = keccak256("MANAGER_ROLE")`

In Go: `crypto.Keccak256Hash([]byte("MANAGER_ROLE"))`

### Why Manager Functions Need Multisig

**Critical security consideration:**

- A single compromised manager could add themselves as an operator
- Operators can mint unlimited tokens via `executeOrder()`
- Operators can manipulate fee percentages (0-100%)
- **Manager + Operator = ability to mint unlimited tokens and manipulate fees**

Therefore, adding/removing operators **must** require multiple signatures.

**Note:** The `feeRecipient` vulnerability has been fixed - fees now go to a global treasury address that can only be changed by admin multisig via `setFeeRecipient()`.

---

## Event Monitoring Requirements

### Events to Subscribe To

**Multisig Events:**

- `ProposalCreated(bytes32 indexed proposalId, address indexed proposer, bytes data, bytes32 roleRequired)`
- `ProposalApproved(bytes32 indexed proposalId, address indexed approver, uint256 approvalCount)`
- `ProposalExecuted(bytes32 indexed proposalId, address indexed executor)`
- `ProposalCancelled(bytes32 indexed proposalId, address indexed canceller)`

**Bridge Operation Events:**

- `OrderCreated(uint256 indexed orderId, address indexed user, string qubicAddress, uint256 amount)`
- `OrderConfirmed(uint256 indexed orderId, uint256 amount, uint256 transferFee, address feeRecipient)`
- `OrderReverted(uint256 indexed orderId, address indexed user, uint256 amount)`
- `OrderExecuted(string indexed originOrderId, address indexed destinationAccount, uint256 amount, uint256 transferFee, address feeRecipient)`

### Event Monitoring Strategy

**Real-time subscriptions** (WebSocket):
- Subscribe to all events for immediate notification
- Best for operator service (needs instant reaction to `OrderCreated`)

**Polling** (HTTP RPC):
- Call `getPendingProposals()` every 60 seconds
- Acceptable for admin/manager services (low frequency)

**Historical sync**:
- On startup, fetch past events to rebuild state
- Prevent missing events during downtime

### Event Handling Actions

When `ProposalCreated` is emitted:
- Send notification to admins/managers
- Log proposal details to database
- Optionally trigger auto-approval logic

When `ProposalApproved` is emitted:
- Check if threshold met
- If yes, trigger execution
- Update approval count in database

When `ProposalExecuted` is emitted:
- Mark proposal as complete in database
- Log successful execution
- Send confirmation notification

---

## Error Handling

### Common Errors by Function

**proposeAction():**
- `AccessControl: account is missing role` - Caller not admin/manager
- `QubicBridge: Invalid role` - roleRequired is neither admin nor manager role

**approveProposal():**
- `QubicBridge: Proposal does not exist` - Invalid proposalId
- `QubicBridge: Already approved` - Account already approved this proposal
- `QubicBridge: Wrong role` - Caller doesn't have required role
- `QubicBridge: Proposal cancelled` - Proposal was cancelled
- `QubicBridge: Proposal already executed` - Already executed

**executeProposal():**
- `QubicBridge: Proposal does not exist` - Invalid proposalId
- `QubicBridge: Not enough approvals` - Threshold not met
- `QubicBridge: Proposal cancelled` - Proposal was cancelled
- `QubicBridge: Proposal already executed` - Already executed
- `QubicBridge: Proposal execution failed` - The underlying function call reverted

**cancelProposal():**
- `QubicBridge: Proposal does not exist` - Invalid proposalId
- `QubicBridge: Not proposer or admin` - Caller cannot cancel
- `QubicBridge: Proposal already executed` - Cannot cancel executed proposal

### Error Handling Strategy

For your Go backend:

1. **Parse revert reasons** from transaction receipts
2. **Retry on network errors** (RPC timeout, connection issues)
3. **Do NOT retry on contract errors** (already executed, wrong role, etc.)
4. **Log all errors** with full context for debugging
5. **Alert on critical errors** (unexpected failures, role issues)

---

## Security Considerations

### Private Key Management

**Operator keys:**
- Hot wallets acceptable (high frequency usage)
- Store in encrypted environment variables or key management service
- One key per operator node

**Manager keys:**
- Cold storage or hardware wallets recommended
- Only needed for occasional operator changes
- Multiple keys across different physical locations

**Admin keys:**
- **Hardware wallets strongly recommended** (Ledger, Trezor)
- Highest security required (control over all funds)
- Multiple keys across different organizations if possible
- Never store admin keys on internet-connected servers

### Threshold Configuration

**Admin threshold recommendations:**
- Minimum: 2/3 (2 signatures required out of 3 admins)
- Recommended: 3/5 or 4/7 for production
- Never use 1-of-N (defeats purpose of multisig)

**Manager threshold recommendations:**
- Minimum: 2/3
- Recommended: 2/4 or 3/5
- Balance between security and operational flexibility

### Notification System

Implement multi-channel notifications:
- **Email** - Primary notification method
- **SMS** - Critical proposals (emergency pause, withdrawals)
- **Slack/Discord** - Team coordination
- **On-call paging** - Emergency situations

### Monitoring Best Practices

**Operator monitoring:**
- Track transaction frequency per operator
- Alert if operator submits > 1000 txs/day (potential compromise)
- Monitor fee percentages used (alert if feePct > 10% for example)
- Check for unusual patterns (many reverts, wrong addresses, excessive fees)

**Proposal monitoring:**
- Decode proposal data and display human-readable action
- Require manual approval for sensitive actions (withdrawals, role changes)
- Auto-approve routine actions if configured (fee adjustments)
- Log all proposal activity to immutable audit log

**Health checks:**
- Operator balance monitoring (alert if < 0.1 ETH)
- Bridge pause state monitoring
- Proposal queue depth monitoring
- RPC endpoint health monitoring

### Rate Limiting

Implement application-level rate limits:
- Max orders per operator per hour
- Max proposal executions per hour
- Max fee percentage per transaction
- Max withdrawal amounts per proposal

---

## Implementation Checklist

### Operator Service (Existing - Minor Updates)

- [ ] No changes needed for multisig (operator functions unchanged)
- [ ] Verify error handling for new operator-related errors
- [ ] Ensure event monitoring captures all relevant events
- [ ] Add monitoring for operator role status (could be removed via multisig)

### Manager Service (New Service Required)

- [ ] Create manager service separate from operator nodes
- [ ] Implement `proposeAction()` wrapper for `addOperator()` and `removeOperator()`
- [ ] Implement `approveProposal()` for manager proposals
- [ ] Implement `executeProposal()` for approved manager proposals
- [ ] Add polling loop for `getPendingProposals()` filtered by `MANAGER_ROLE`
- [ ] Implement notification system for manager proposals
- [ ] Configure manager threshold and wallet addresses
- [ ] Set up secure key storage for manager wallets

### Admin Service (New Service Required)

- [ ] Create admin service separate from operator nodes
- [ ] Implement `proposeAction()` wrappers for all admin functions
- [ ] Implement `approveProposal()` for admin proposals
- [ ] Implement `executeProposal()` for approved admin proposals
- [ ] Add polling loop for `getPendingProposals()` filtered by `DEFAULT_ADMIN_ROLE`
- [ ] Implement notification system for admin proposals
- [ ] Configure admin threshold and wallet addresses
- [ ] Set up hardware wallet integration for admin keys
- [ ] Create emergency runbook for critical proposals

### Event Monitoring (Enhanced)

- [ ] Subscribe to `ProposalCreated`, `ProposalApproved`, `ProposalExecuted` events
- [ ] Handle event parsing for multisig events
- [ ] Store proposal state in database
- [ ] Implement event replay on service restart
- [ ] Add monitoring dashboards for proposal activity

### Testing (Critical)

- [ ] Test complete proposal flow (propose → approve → execute)
- [ ] Test threshold enforcement (N-1 approvals should fail execution)
- [ ] Test proposal cancellation
- [ ] Test duplicate approval rejection
- [ ] Test wrong role rejection (manager approving admin proposal)
- [ ] Test operator functions still work (no multisig impact)
- [ ] Load test proposal system

---

## Function Selectors Reference

For Go backend integration, you may need the 4-byte function selectors (first 4 bytes of keccak256(signature)):

### Admin Function Selectors (require multisig via proposeAction)
```
addAdmin(address)                                    → 0x70480275
removeAdmin(address)                                 → 0x1785f53c
addManager(address)                                  → 0x2d06177a
removeManager(address)                               → 0xac18de43
setBaseFee(uint256)                                  → 0x46860698
setFeeRecipient(address)                             → 0xe74b981b
setAdminThreshold(uint256)                           → 0x5af28cf9
setManagerThreshold(uint256)                         → 0x4fdfb1ad
emergencyPause()                                     → 0x51858e27
emergencyUnpause()                                   → 0x4a4e3bd5
emergencyTokenWithdraw(address,address,uint256)     → 0xfef860b8
emergencyEtherWithdraw(address)                      → 0x77964ad1
```

### Manager Function Selectors (require multisig via proposeAction)
```
addOperator(address)                                 → 0x9870d7fe
removeOperator(address)                              → 0xac8a584a
```

### Operator Function Selectors (direct call, no multisig)
```
confirmOrder(uint256,uint256)                        → 0xf38aa74d
revertOrder(uint256,uint256)                         → 0x20e9c21c
executeOrder(uint256,string,address,uint256,uint256) → 0x6ead4f0c
```

### Public Function Selectors
```
createOrder(string,uint256,bool)                     → 0x72354d92
```

### Multisig Function Selectors
```
proposeAction(bytes,bytes32)                         → 0xf4cf8576
approveProposal(bytes32)                             → 0xf20b5b2c
executeProposal(bytes32)                             → 0x980ff6c6
cancelProposal(bytes32)                              → 0x37376ca8
```

### Role Constants
```
DEFAULT_ADMIN_ROLE = 0x0000000000000000000000000000000000000000000000000000000000000000
MANAGER_ROLE       = keccak256("MANAGER_ROLE")
                   = 0x241ecf16d79d0f8dbfb92cbc07fe17840425976cf0667f022fe9877caa831b08
OPERATOR_ROLE      = keccak256("OPERATOR_ROLE")
                   = 0x97667070c54ef182b0f5858b034beac1b6f3089aa2d3188bb1e8929f4fa9b929
```

**Note:** When using `abigen` to generate Go bindings, these selectors are automatically included in the generated code. You typically don't need to manually compute them.

---

## Support and Resources

- **Smart Contract Source:** `src/QubicBridge.sol`
- **Frontend Guide:** `FRONTEND_INTEGRATION.md`
- **Multisig Usage Guide:** `MULTISIG_USAGE.md`
- **Project Overview:** `CLAUDE.md`
- **Deployed Contract (Base Sepolia):** `0x50B0A6391BB06cc0C7a228D41352EE91d496aE78`
- **Verified Contract:** https://sepolia.basescan.org/address/0xbC79b4a96186b0AFE09Ee83830e2Fb30E14d5Ddc

For questions about specific functions, refer to the NatSpec documentation in `QubicBridge.sol`.


QubicToken ✅
Address: 0x5438615E84178C951C0EB84Ec9Af1045eA2A7C78
Explorer: https://base-sepolia.blockscout.com/address/0x5438615e84178c951c0eb84ec9af1045ea2a7c78
QubicBridge (con Multisig) ✅
Address: 0x50B0A6391BB06cc0C7a228D41352EE91d496aE78 
Explorer: https://sepolia.basescan.org/address/0xbC79b4a96186b0AFE09Ee83830e2Fb30E14d5Ddc
Configuración:
Network: Base Sepolia (Chain ID: 84532)
Base Fee: 2% (200 basis points)
Admin Threshold: 2 aprobaciones requeridas
Manager Threshold: 2 aprobaciones requeridas