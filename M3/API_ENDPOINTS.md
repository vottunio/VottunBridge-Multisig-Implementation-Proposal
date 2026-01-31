# Qubic Bridge Backend API Endpoints

**Version**: v1
**Base URL**: `http://63.33.210.165:2116/qubicbridge/v1`
**Purpose**: Knowledge Transfer (KT) guide for backend API testing and integration

---

## Testing Resources

### Postman Collection
Import the Postman collection for direct API testing:

**File**: `qubic team.postman_collection.json`
**Keys Guide**: `Admins,Managers,Operators.md`

---

## API Endpoints

### 1. General Information Endpoints

Health checks and system information queries.

| Endpoint | Method | Path | Description |
|----------|--------|------|-------------|
| **Heartbeat** | `GET` | `/ping` | Checks backend service availability |
| **TickInfo** | `GET` | `/tickInfo` | Retrieves current tick information |
| **GetProposal** | `GET` | `/get-proposal?proposalId={proposalId}` | Fetches Qubic proposal details by ID |
| **GetContractDetails** | `GET` | `/getSmartContractDetails` | Retrieves smart contract configuration |
| **GetOrder** | `GET` | `/getOrderByQubicId/{qubicOrderId}` | Retrieves order details by Qubic ID (example: `/getOrderByQubicId/13`) |

**Example**:
```bash
GET http://63.33.210.165:2116/qubicbridge/v1/ping
GET http://63.33.210.165:2116/qubicbridge/v1/get-proposal?proposalId=12345
```

---

### 2. Qubic Bridge Management Endpoints

Admin and manager operations for Qubic-side bridge management.

| Endpoint | Method | Path | Request Body | Description |
|----------|--------|------|--------------|-------------|
| **SET_ADMIN** | `POST` | `/set-admin-qubic` | `oldAdminAddress`, `newAdminAddress` | Proposes admin address change |
| **CreateOrder** | `POST` | `/createOrder` | `ethAddress`, `qubicAddress`, `amount`, `memo`, `seed`, `fromQubicToEthereum` | Creates cross-chain order |
| **AddLiquidity** | `POST` | `/add-liquidity` | `amount` | Adds liquidity to bridge |
| **ApproveProposal** | `POST` | `/approve-proposal` | `proposalId` | Approves a Qubic proposal |
| **AddManager** | `POST` | `/create-proposal-add-manager` | `managerAddress` | Creates proposal to add manager |
| **RemoveManager** | `POST` | `/create-proposal-remove-manager` | `managerAddress` | Creates proposal to remove manager |
| **WithdrawFee** | `POST` | `/withdraw-fee` | `amount` | Initiates Qubic fee withdrawal |
| **ChangeThreshold** | `POST` | `/change-threshold` | `threshold` | Changes approval threshold |

**Example - Create Order**:
```json
POST http://63.33.210.165:2116/qubicbridge/v1/createOrder
Content-Type: application/json

{
  "ethAddress": "0x1234...",
  "qubicAddress": "ABCD...",
  "amount": "1000000",
  "memo": "Bridge transfer",
  "seed": "your-seed-here",
  "fromQubicToEthereum": true
}
```

---

### 3. EVM Bridge Management Endpoints

Ethereum Virtual Machine (EVM) side bridge operations with multisig governance.

#### 3.1 Admin Management (Requires Admin Multisig)

| Endpoint | Method | Path | Request Body | Description |
|----------|--------|------|--------------|-------------|
| **Add Admin** | `POST` | `/evm/add-admin-evm` | `evmAdminAddress` | Creates proposal to add EVM admin |
| **Remove Admin** | `POST` | `/evm/remove-admin-evm` | `evmAdminAddress` | Creates proposal to remove EVM admin |
| **Add Manager** | `POST` | `/evm/add-manager-evm` | `evmManagerAddress` | Creates proposal to add EVM manager |
| **Remove Manager** | `POST` | `/evm/remove-manager-evm` | `evmManagerAddress` | Creates proposal to remove EVM manager |
| **Set Base Fee** | `POST` | `/evm/set-basefee-evm` | `adminKey`, `baseFee` | Creates proposal to change base fee |
| **Set Fee Recipient** | `POST` | `/evm/set-fee-recipient-evm` | `recipientAddress`, `adminKey` | Creates proposal to change fee recipient |
| **Set Admin Threshold** | `POST` | `/evm/set-admin-threshold-evm` | `evmThreshold` | Creates proposal to change admin threshold |
| **Set Manager Threshold** | `POST` | `/evm/set-manager-threshold-evm` | `evmThreshold` | Creates proposal to change manager threshold |

#### 3.2 Manager Operations (Requires Manager Multisig)

| Endpoint | Method | Path | Request Body | Description |
|----------|--------|------|--------------|-------------|
| **Add Operator** | `POST` | `/evm/add-operator-evm` | `evmOperatorAddress` | Creates proposal to add EVM operator |
| **Remove Operator** | `POST` | `/evm/remove-operator-evm` | `evmOperatorAddress` | Creates proposal to remove EVM operator |

#### 3.3 Emergency Functions (Requires Admin Multisig)

| Endpoint | Method | Path | Request Body | Description |
|----------|--------|------|--------------|-------------|
| **Emergency Pause** | `POST` | `/evm/emergency-pause-evm` | `adminKey` | Creates proposal to pause bridge |
| **Emergency Unpause** | `POST` | `/evm/remove-pause-evm` | `adminKey` | Creates proposal to unpause bridge |
| **Emergency Token Withdraw** | `POST` | `/evm/emergency-token-withdraw-evm` | `tokenAddress`, `recipientAddress`, `amountTokens` | Emergency token recovery |
| **Emergency Ether Withdraw** | `POST` | `/evm/emergency-ether-withdraw-evm` | `recipientAddress` | Emergency ETH recovery |

#### 3.4 Proposal Management

| Endpoint | Method | Path | Query Parameters | Description |
|----------|--------|------|------------------|-------------|
| **Fetch All Proposals** | `GET` | `/evm/fetch-proposals-evm` | - | Retrieves all EVM proposals from database |
| **Fetch Single Proposal** | `GET` | `/evm/fetch-single-proposals-evm` | `proposeId={proposalId}` | Retrieves specific proposal details |
| **Get Pending Proposals** | `GET` | `/evm/pending-proposals` | - | Fetches pending proposals from smart contract |
| **Get Filtered Proposals** | `GET` | `/evm/get-proposals` | `filter={STATUS}` | Filters proposals (PENDING, APPROVED, EXECUTED) |
| **Approve Proposal** | `POST` | `/evm/approve-proposals` | `proposalID`, `adminKey` (optional) | Approves EVM proposal (blockchain tx) |
| **Execute Proposal** | `POST` | `/evm/execute-proposals` | `proposalID`, `adminKey` | Executes approved proposal |
| **Update Proposal Direct** | `POST` | `/evm/proposals-direct` | `proposalID`, `status`, `transaction_hash` | Updates proposal status (database only) |

**Example - Approve Proposal**:
```json
POST http://63.33.210.165:2116/qubicbridge/v1/evm/approve-proposals
Content-Type: application/json

{
  "proposalID": "0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385",
  "adminKey": "0x1234...abcd"
}
```

**Example - Fetch Filtered Proposals**:
```bash
GET http://63.33.210.165:2116/qubicbridge/v1/evm/get-proposals?filter=PENDING
GET http://63.33.210.165:2116/qubicbridge/v1/evm/get-proposals?filter=EXECUTED
```

---

## Key Management

### Qubic Key Management

**Tool**: Key Security Manager (KSM)

- **Purpose**: Manages all private Qubic keys
- **Security**: Employs encryption to protect sensitive keys
- **Storage**: Encrypted secure storage preventing unauthorized access

### EVM Key Management

**Method**: Self-Custody Wallet (e.g., MetaMask)

- **Purpose**: Sign EVM transactions for admin/bridge operations
- **Authorization**: Decentralized user-controlled transaction signing
- **Security**: Private keys never leave user's wallet

---

## Request/Response Examples

### Create EVM Proposal (Admin Function)

**Request**:
```bash
POST http://63.33.210.165:2116/qubicbridge/v1/evm/emergency-pause-evm
Content-Type: application/json

{
  "adminKey": "0x1234567890abcdef..."
}
```

**Response**:
```json
{
  "message": "proposal created successfully for emergency pause",
  "tx_hash": "0xabcd1234..."
}
```

### Approve Qubic Proposal

**Request**:
```bash
POST http://63.33.210.165:2116/qubicbridge/v1/approve-proposal
Content-Type: application/json

{
  "proposalId": "12345"
}
```

**Response**:
```json
{
  "message": "proposal approved successfully",
  "status": "APPROVED",
  "approvalCount": 2,
  "requiredApprovals": 2
}
```

### Fetch Order by Qubic ID

**Request**:
```bash
GET http://63.33.210.165:2116/qubicbridge/v1/getOrderByQubicId/13
```

**Response**:
```json
{
  "orderId": "uuid-1234...",
  "qubicId": 13,
  "originAccount": "QUBIC_ADDRESS...",
  "destinationAccount": "0xETH_ADDRESS...",
  "amount": 1000000,
  "status": "COMPLETED",
  "sourceChainId": 1
}
```

---

## Endpoint Summary

### By Category

| Category | Count | Description |
|----------|-------|-------------|
| **General Info** | 5 | Health checks, system info |
| **Qubic Management** | 8 | Qubic-side bridge operations |
| **EVM Admin** | 8 | Admin-level EVM operations |
| **EVM Manager** | 2 | Manager-level EVM operations |
| **EVM Emergency** | 4 | Emergency controls |
| **EVM Proposals** | 7 | Proposal lifecycle management |
| **Total** | **34** | Complete API surface |

### By Milestone

| Milestone | Endpoints Added | Category |
|-----------|----------------|----------|
| **M1** | 13 | General + Qubic base |
| **M2** | 17 | EVM multisig core |
| **M3** | 4 | EVM filters + direct updates |

---

## Authentication & Authorization

### Qubic Operations
- **Method**: Seed-based signing
- **Storage**: KSM encrypted storage
- **Validation**: Signature verification on-chain

### EVM Operations
- **Method**: Private key signing
- **Storage**: User wallet (MetaMask, etc.)
- **Validation**: Smart contract role checks (Admin/Manager/Operator)

### Proposal Approval Flow
1. **Create Proposal**: Requires admin/manager role
2. **Approve Proposal**: Requires M-of-N signatures
3. **Execute Proposal**: Automatic when threshold met, or manual call

---

## Error Responses

### Standard Error Format
```json
{
  "error": "Error description",
  "code": "ERROR_CODE",
  "status": 400
}
```

### Common Error Codes

| HTTP Code | Meaning | Example |
|-----------|---------|---------|
| `400` | Bad Request | Invalid parameters, missing required fields |
| `401` | Unauthorized | Invalid admin key, insufficient permissions |
| `404` | Not Found | Proposal not found, order not found |
| `500` | Internal Server Error | Database error, blockchain interaction failure |

---

## Rate Limiting & Performance

- **Rate Limit**: Not enforced (testnet)
- **Recommended**: Max 10 requests/second per IP
- **Response Time**:
  - GET queries: < 200ms
  - POST proposals: 1-5 seconds (blockchain confirmation)

---

## Notes

- All timestamps in responses are Unix epoch (seconds)
- Amounts are in smallest unit (wei for ETH, smallest Qubic unit)
- Addresses are case-sensitive
- Proposal IDs are hex strings (0x...)
- Base URL subject to change in production

---

**Last Updated**: 2026-01-31
**API Version**: v1
**Documentation**: For full integration guide, see M2.md and M3.md in evidences/
