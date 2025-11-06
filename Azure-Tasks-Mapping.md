# VottunBridge Multisig - Task Mapping by Milestone for Azure

## Milestone 1: Core Multisig Infrastructure (20%)
**Duration**: Week 1 (development) + Week 2 (QA)
**Hours**: 68h development
**Payment**: $4,200 USD

### SC Qubic (32h) - Week 1
- [x] **[W1] 1.1** Design and implement multisig wallet structures (8h)
- [x] **[W1] 1.2** Modify existing order structures for approval tracking (8h)
- [x] **[W1] 1.3** Implement core multisig validation functions (16h)
  - Owner verification and validation logic
  - Approval counting and threshold checking
  - Security validations and edge case handling

### Backend Go (18h) - Week 1
- [x] **[W1] 2.1** Design and implement comprehensive DTO structures (8h)
  - Multisig wallet DTOs with validation
  - Update order DTOs for approval tracking
  - Approval history and audit trail structures
  - JSON marshaling and validation
- [x] **[W1] 2.2** Develop robust multisig business logic - COMPLETE (10h of 12h)
  - Multisig service layer with state management
  - Approval workflow orchestration

### Frontend React (6h) - Week 1
- [x] **[W1] 4.1** Develop state management and API integration - COMPLETE (6h of 15h)
  - Basic multisig wallet hooks with caching
  - Initial approval status hooks

### Testing (12h) - Week 1
- [x] **[W1] 5.1** Comprehensive unit testing - COMPLETE (12h of 16h)
  - Qubic contract multisig function testing (35 tests passing)
  - Go backend service layer testing with mocks
  - Integration testing completed

### QA - Week 2
- [x] **[W2] QA** Ecosystem Services validation of M1 deliverables

---

## Milestone 2: Full Order Lifecycle + EVM Integration (25%)
**Duration**: Week 3 (development) + Week 4 (QA)
**Hours**: 85h development
**Payment**: $5,250 USD

### SC Qubic (16h) - Week 3
- [x] **[W3] 1.4** Implement admin proposal system for critical operations (16h)
  - Add/Remove admin proposal workflow (AddAdmin function)
  - Add/Remove manager proposal workflow (CreateProposalAddManager, CreateProposalRemoveManager)
  - ApproveProposal function with validation
  - GetProposal query function with binary parsing
  - Note: User operations (completeOrder, refundOrder, transferToContract) remain single-sig for UX

### Backend Go (18h) - Week 3
- [x] **[W3] 2.2** Develop robust multisig business logic - COMPLETE (12h)
  - Integration with Qubic contract (adminClient.go, managerClient.go)
  - Error handling and retry mechanisms (WaitForTransactionToApprove goroutine)
  - Proposal validation (PENDING → PARTIAL → APPROVED → EXECUTED)
- [x] **[W3] 2.3** Implement multisig proposal endpoints (8h)
  - 14 admin/manager proposal endpoints (emergency-pause, set-basefee, add-manager, add-operator, etc.)
  - Approval/Execute endpoints (approve-proposals, execute-proposals)
  - Database persistence with status tracking
  - Note: User order operations (complete, refund, transfer) remain direct for UX - no multisig
- [x] **[W3] 2.4** Implement comprehensive multisig API endpoints - PARTIAL (8h of 16h DONE)
  - Approval submission: ApproveProposalEvm(), ExecuteProposalEvm()
  - Proposal tracking: GetAllPendingProposals(), FetchAllProposals(), FetchSingleProposalsEVM()
  - Emergency/admin proposals: EmergencyPauseEVM(), SetBaseFeeEVM(), ChangeFeeRecipientEVM()
  - Manager proposals: CreateProposalForAddingOperatorEVM()
  - 8h remaining for M3: Audit trail endpoints, batch operations

### SC EVM (40h) - Week 3
- [x] **[W3] 3.1** Design and implement comprehensive operator multisig system (12h)
  - Create sophisticated approval tracking mappings and structures
  - Implement operator role management with access controls
  - Design gas-efficient approval storage mechanisms
  - Add comprehensive event emission for approval tracking
- [x] **[W3] 3.2** Implement multisig for admin/manager functions (16h)
  - All admin functions protected by onlyProposal modifier (13 functions)
  - All manager functions protected by onlyProposal modifier (2 functions)
  - Operator functions remain single-sig (confirmOrder, revertOrder, executeOrder) - for bridge performance
  - Fixed fee recipient security vulnerability (global feeRecipient instead of per-tx parameter)
  - Implement proposal workflow (proposeAction, approveProposal, executeProposal, cancelProposal)
- [x] **[W3] 3.3** Add comprehensive multisig management functions - COMPLETE (12h)
  - Approval submission with validation and security checks
  - Admin functions: addAdmin, removeAdmin, addManager, removeManager, setBaseFee, setFeeRecipient, setAdminThreshold, setManagerThreshold
  - Emergency functions: emergencyPause, emergencyUnpause, emergencyTokenWithdraw, emergencyEtherWithdraw
  - Manager functions: addOperator, removeOperator
  - Deployed to Base Sepolia testnet: 0xbC79b4a96186b0AFE09Ee83830e2Fb30E14d5Ddc

### Testing (11h) - Week 3
- [x] **[W3] 5.1** Comprehensive unit testing - COMPLETE (16h)
  - EVM contract testing with complex approval workflows (53 tests passing)
  - test/QubicBridgeMultisig.t.sol: 35 tests covering multisig system
  - test/QubicBridge.t.sol: 18 tests covering operator functions and fee recipient fix
  - Coverage: Multisig proposals, admin functions, manager functions, operator functions, security validations
- [ ] **[W3] 5.2** End-to-end integration testing - PARTIAL (7h of 12h)
  - Cross-platform multisig workflow testing
  - Real-world scenario simulation with multiple participants

### QA - Week 4
- [ ] **[W4] QA** Ecosystem Services validation of M2 deliverables

---

## Milestone 3: Wallet Management + Testing (25%)
**Duration**: Week 5 (development) + Week 6 (QA)
**Hours**: 85h development
**Payment**: $5,250 USD

### SC Qubic (16h) - Week 5
- [ ] **[W5] 1.5** Add comprehensive wallet management functions - PARTIAL (12h of 16h DONE, 4h remaining)
  - [x] Owner management with proper validations (PROPOSAL_SET_ADMIN replaces admins - sufficient for Qubic design)
  - [x] Threshold management with safety checks (PROPOSAL_CHANGE_THRESHOLD implemented)
  - [ ] Emergency procedures and fallback mechanisms (4h remaining - NEEDS: emergency pause/unpause bridge operations, emergency token recovery)
  - [x] Event logging and audit trail implementation (EthBridgeLogger, AddressChangeLogger present)

### Backend Go (8h) - Week 5
- [ ] **[W5] 2.4** Implement comprehensive multisig API endpoints - COMPLETE (8h remaining of 16h)
  - Approval history and audit trail endpoints
  - Real-time approval status and notification systems
  - Batch approval operations for efficiency

### Frontend React (9h) - Week 5
- [x] **[W5] 4.1** Develop state management and API integration - MOSTLY COMPLETE (7h of 9h)
  - [x] State management with qubicProposalService and evmAdminService
  - [x] Error handling and retry mechanisms for multisig operations
  - [x] Integration with existing authentication and authorization systems (useUserRole, useQubicRole)
  - [x] Complete admin UI with QubicProposalsList, EvmProposalsList, CreateProposalModal, CreateEvmProposalModal
  - [x] AdminManagementModal, OrderManagementModal, QubicRolesPanel, EvmActionsPanel components
  - [x] Full Admin page (/admin) with tab system for EVM and Qubic proposals
  - [ ] Real-time approval status WebSocket integration (1h remaining)
  - [ ] Optimistic UI updates for better UX (1h remaining)

### Testing (52h) - Week 5
- [ ] **[W5] 5.2** End-to-end integration testing - COMPLETE (5h remaining of 12h)
  - Performance testing under high approval load
  - Error recovery and resilience testing
- [ ] **[W5] 5.3** Specialized multisig security and edge case testing (8h)
  - Concurrent approval submission testing
  - Threshold manipulation and security breach scenarios
  - Owner management edge cases and attack vectors
  - Gas optimization and cost analysis for EVM operations
- [ ] **[W5] 5.4** User acceptance and scenario-based testing (8h)
  - Complete approval workflow simulation with real users
  - Mobile and desktop interface testing across browsers
  - Accessibility testing for multisig interfaces
  - Performance benchmarking and optimization
  - Load testing with multiple simultaneous approvals

### QA - Week 6
- [ ] **[W6] QA** Ecosystem Services validation of M3 deliverables

---

## Milestone 4: Production Deployment + Audit (30%)
**Duration**: Weeks 7-8 (development + audit prep) + Weeks 9-10 (external audit)
**Hours**: 102h development + audit coordination
**Payment**: $6,300 USD

### Documentation (8h) - Week 7
- [ ] **[W7] 6.1** Comprehensive technical documentation (8h)
  - Detailed multisig architecture documentation
  - API documentation with approval workflow examples
  - Smart contract documentation with security considerations

### Deployment (20h) - Weeks 7-8
- [ ] **[W7] 6.3** Migration and deployment preparation (4h)
  - Production-ready migration scripts with rollback procedures
  - Environment configuration and security hardening guides
  - Monitoring and alerting setup for multisig operations
- [ ] **[W8] 6.4** Deployment, validation, and go-live support (16h)
  - Staged deployment across test and production environments
  - Comprehensive validation testing in production-like environment
  - Go-live support and monitoring during initial deployment
  - Post-deployment optimization and performance tuning
  - Emergency response procedures and rollback planning

### External Audit - Weeks 9-10
- [ ] **[W9-10] External Audit** External smart contract audit
  - Coordination with audit firm (TBD)
  - Review and fix vulnerabilities found
  - Final pre-mainnet validation

---

## Summary of Hours by Component and Milestone

### By Milestone:
| Milestone | SC Qubic | Backend Go | SC EVM | Frontend React | Testing | Docs/Deploy | Total |
|-----------|----------|------------|--------|----------------|---------|-------------|-------|
| M1 (W1-2) | 32h | 18h | 0h | 6h | 12h | 0h | 68h |
| M2 (W3-4) | 16h | 18h | 40h | 0h | 11h | 0h | 85h |
| M3 (W5-6) | 16h | 8h | 0h | 9h | 52h | 0h | 85h |
| M4 (W7-10) | 0h | 0h | 0h | 0h | 0h | 28h | 28h |
| **Total** | **64h** | **44h** | **40h** | **15h** | **75h** | **28h** | **266h** |

*Note: 74h additional buffer/contingency distributed as needed*

### By Week:
| Week | Focus | Dev Hours | Activities |
|------|-------|-----------|------------|
| W1 | M1 Development | 68h | SC Qubic (32h), Backend Go (18h), Frontend React (6h), Testing (12h) |
| W2 | M1 QA | 0h | Ecosystem Services validation |
| W3 | M2 Development | 85h | SC Qubic (16h), Backend Go (18h), SC EVM (40h), Testing (11h) |
| W4 | M2 QA | 0h | Ecosystem Services validation |
| W5 | M3 Development | 85h | SC Qubic (16h), Backend Go (8h), Frontend React (9h), Testing (52h) |
| W6 | M3 QA | 0h | Ecosystem Services validation |
| W7 | M4 Dev + Prep | 12h | Documentation (8h), Deployment prep (4h) |
| W8 | M4 Deployment | 16h | Deployment and validation (16h) |
| W9-10 | M4 External Audit | 0h | External audit coordination and fixes |

---

## Success Criteria by Milestone

### M1 Success Criteria:
- ✅ Multisig structure deployed and functional
- ✅ Basic approval validation working
- ✅ API endpoints operational

### M2 Success Criteria:
- ✅ All admin/manager operations require multisig approval (15 EVM functions + Qubic admin/manager proposals)
- ✅ EVM multisig functional (2-of-3 approval system with proposeAction → approveProposal → executeProposal)
- ✅ Backend Go API complete (14 endpoints, database tracking, validation logic)
- ✅ Qubic proposal system complete (AddAdmin, CreateProposalAddManager, ApproveProposal, GetProposal)
- ✅ 53 tests passing (EVM) + Qubic integration tested
- ✅ Deployed to Base Sepolia testnet: 0xbC79b4a96186b0AFE09Ee83830e2Fb30E14d5Ddc
- ✅ Documentation complete (M2.md with EVM + Backend sections)
- ✅ User operations (completeOrder, refundOrder, transferToContract) remain single-sig for UX

### M3 Success Criteria:
- [ ] Owner management operational
- [ ] Threshold adjustment working
- [ ] Emergency procedures functional
- [ ] Comprehensive test coverage

### M4 Success Criteria:
- [ ] Zero critical vulnerabilities in audit
- [ ] Production deployment successful
- [ ] All existing bridge functionality preserved
- [ ] Complete documentation delivered

---

## Task Dependencies

### M1 Dependencies:
- All M1 tasks can start immediately
- Testing depends on SC Qubic and Backend Go completion

### M2 Dependencies:
- M2 requires M1 completion (core multisig infrastructure)
- SC EVM work can start in parallel with SC Qubic
- Backend Go 2.3 and 2.4 depend on SC Qubic 1.4 completion
- Testing depends on all development tasks

### M3 Dependencies:
- M3 requires M1 and M2 completion
- Frontend React work depends on Backend Go API endpoints
- All testing depends on development completion

### M4 Dependencies:
- M4 requires M1, M2, and M3 completion
- External audit depends on complete code freeze
- Deployment depends on documentation and audit approval
