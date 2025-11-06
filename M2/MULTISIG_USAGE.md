# Multisig System Usage Guide

## Overview

The QubicBridge contract now includes an on-chain multisig system for administrative and manager functions. This ensures that critical operations require multiple approvals before execution.

## Configuration

### Thresholds

- **Admin Threshold**: Number of admin approvals required for admin actions (e.g., 2 out of 3)
- **Manager Threshold**: Number of manager approvals required for manager actions (e.g., 2 out of 3)

Set during deployment:
```solidity
address[] memory admins = new address[](3);
admins[0] = 0x...;
admins[1] = 0x...;
admins[2] = 0x...;

QubicBridge bridge = new QubicBridge(
    tokenAddress,
    baseFee,
    admins,          // Array of admin addresses (max 3)
    adminThreshold,  // e.g., 2
    managerThreshold, // e.g., 2
    feeRecipient     // Treasury address
);
```

## Functions Requiring Multisig

### Admin Functions (require admin threshold)
1. `addAdmin(address newAdmin)` - Add a new admin (max 3)
2. `removeAdmin(address admin)` - Remove an admin
3. `addManager(address newManager)` - Add a manager (max 3)
4. `removeManager(address manager)` - Remove a manager
5. `setBaseFee(uint256 _baseFee)` - Change base fee
6. `setFeeRecipient(address newFeeRecipient)` - Change treasury address
7. `setAdminThreshold(uint256 newThreshold)` - Change admin approval threshold
8. `setManagerThreshold(uint256 newThreshold)` - Change manager approval threshold
9. `emergencyPause()` - Pause bridge
10. `emergencyUnpause()` - Unpause bridge
11. `emergencyTokenWithdraw(address tokenAddress, address recipient, uint256 amount)` - Withdraw tokens
12. `emergencyEtherWithdraw(address recipient)` - Withdraw ETH

### Manager Functions (require manager threshold)
13. `addOperator(address newOperator)` - Add an operator
14. `removeOperator(address operator)` - Remove an operator

## How to Use the Multisig System

### Step 1: Create a Proposal

An admin or manager creates a proposal for an action:

```solidity
// Example: Propose to change baseFee to 3%
uint256 newBaseFee = 3 * 100; // 300 basis points

// Encode the function call
bytes memory data = abi.encodeWithSelector(
    bridge.setBaseFee.selector,
    newBaseFee
);

// Create proposal (requires DEFAULT_ADMIN_ROLE)
bytes32 proposalId = bridge.proposeAction(
    data,
    bridge.DEFAULT_ADMIN_ROLE() // Role required to approve
);
```

### Step 2: Other Admins/Managers Approve

Other admins/managers with the required role approve the proposal:

```solidity
// Admin 2 approves
bridge.approveProposal(proposalId);

// Admin 3 approves
bridge.approveProposal(proposalId);
```

### Step 3: Execute the Proposal

Once threshold is met, anyone with the required role can execute:

```solidity
// Execute the proposal
bridge.executeProposal(proposalId);
```

## Complete Examples

### Example 1: Add a New Manager (Admin Action)

```javascript
// Admin 1 creates proposal
const newManagerAddress = "0x123...";
const data = bridge.interface.encodeFunctionData("addManager", [newManagerAddress]);
const tx1 = await bridge.proposeAction(data, await bridge.DEFAULT_ADMIN_ROLE());
const receipt1 = await tx1.wait();
const proposalId = receipt1.events[0].args.proposalId;

// Admin 2 approves
const admin2 = await bridge.connect(admin2Signer);
await admin2.approveProposal(proposalId);

// If threshold is 2, proposal can now be executed
await admin2.executeProposal(proposalId);
```

### Example 2: Add an Operator (Manager Action)

```javascript
// Manager 1 creates proposal
const newOperatorAddress = "0x456...";
const data = bridge.interface.encodeFunctionData("addOperator", [newOperatorAddress]);
const tx1 = await bridge.proposeAction(data, await bridge.MANAGER_ROLE());
const receipt1 = await tx1.wait();
const proposalId = receipt1.events[0].args.proposalId;

// Manager 2 approves
const manager2 = await bridge.connect(manager2Signer);
await manager2.approveProposal(proposalId);

// Execute
await manager2.executeProposal(proposalId);
```

### Example 3: Emergency Pause (Admin Action)

```javascript
// Admin 1 proposes emergency pause
const data = bridge.interface.encodeFunctionData("emergencyPause", []);
const tx1 = await bridge.proposeAction(data, await bridge.DEFAULT_ADMIN_ROLE());
const receipt1 = await tx1.wait();
const proposalId = receipt1.events[0].args.proposalId;

// Admins approve (assuming threshold = 2)
await bridge.connect(admin2Signer).approveProposal(proposalId);

// Execute
await bridge.connect(admin2Signer).executeProposal(proposalId);
```

## Query Functions

### Get Pending Proposals

```solidity
bytes32[] memory pendingIds = bridge.getPendingProposals();
```

### Get Proposal Details

```solidity
(
    address proposer,
    bytes memory data,
    uint256 approvalCount,
    bool executed,
    uint256 createdAt,
    bytes32 roleRequired
) = bridge.getProposal(proposalId);
```

### Check if Address Has Approved

```solidity
bool hasApproved = bridge.hasApprovedProposal(proposalId, adminAddress);
```

### Get Current Thresholds

```solidity
uint256 adminThreshold = bridge.adminThreshold();
uint256 managerThreshold = bridge.managerThreshold();
```

## Frontend Integration

### Display Pending Proposals

```javascript
const pendingProposals = await bridge.getPendingProposals();

for (const proposalId of pendingProposals) {
    const proposal = await bridge.getProposal(proposalId);
    console.log({
        id: proposalId,
        proposer: proposal.proposer,
        approvalCount: proposal.approvalCount.toString(),
        executed: proposal.executed,
        createdAt: new Date(proposal.createdAt.toNumber() * 1000)
    });
}
```

### Check if Current User Can Approve

```javascript
const isAdmin = await bridge.hasRole(await bridge.DEFAULT_ADMIN_ROLE(), userAddress);
const isManager = await bridge.hasRole(await bridge.MANAGER_ROLE(), userAddress);
const proposal = await bridge.getProposal(proposalId);

const canApprove = (
    (proposal.roleRequired === await bridge.DEFAULT_ADMIN_ROLE() && isAdmin) ||
    (proposal.roleRequired === await bridge.MANAGER_ROLE() && isManager)
);
```

### Check if User Has Already Approved

```javascript
const hasApproved = await bridge.hasApprovedProposal(proposalId, userAddress);
```

## Events to Monitor

```solidity
event ProposalCreated(
    bytes32 indexed proposalId,
    address indexed proposer,
    bytes data,
    bytes32 roleRequired
);

event ProposalApproved(
    bytes32 indexed proposalId,
    address indexed approver,
    uint256 approvalCount
);

event ProposalExecuted(
    bytes32 indexed proposalId,
    address indexed executor
);

event ProposalCancelled(
    bytes32 indexed proposalId,
    address indexed canceller
);
```

## Security Considerations

1. **Threshold Configuration**: Set thresholds appropriately:
   - Too low: Risk of single compromised account
   - Too high: Risk of being unable to respond to emergencies

2. **Role Distribution**: Distribute admin/manager roles across multiple trusted parties

3. **Proposal Monitoring**: Monitor all proposals and approvals

4. **Timeouts**: Consider implementing proposal expiration in production

5. **Emergency Procedures**: Have clear procedures for emergency situations

## Best Practices

1. **Always verify proposals** before approving
2. **Use hardware wallets** for admin/manager accounts
3. **Keep proposal IDs** recorded externally
4. **Monitor events** for unauthorized proposals
5. **Test thoroughly** on testnet before mainnet deployment
