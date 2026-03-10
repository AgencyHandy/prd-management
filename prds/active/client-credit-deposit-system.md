# PRD: Client Credit / Deposit System

**Priority:** HIGH | **Impact:** Direct Revenue Increase | **Timeline:** Q2 2026

## Context & Problem

Agencies need a way to collect deposits upfront and manage client credit balances to improve cash flow and reduce payment friction. Currently, there's no systematic way to:
- Collect deposits before work begins
- Manage prepaid balances for ongoing services
- Apply credits automatically to future invoices
- Track client payment history and credit status

**Revenue Impact:** $15K+ MRR - improved cash flow, reduced payment delays, increased client lifetime value through prepaid balances.

## Solution Overview

Implement a comprehensive client credit/deposit system that allows agencies to:
1. Set mandatory deposits for services/packages
2. Collect prepayments that accumulate as client credits
3. Auto-apply credits to invoices with manual override options
4. Provide clear credit balance visibility to both agency and clients

## Core Features

### 1. Deposit Configuration
- Service-level deposit requirements (fixed amount or percentage)
- Package-level deposit rules
- Project-based deposit collection
- Deposit exemption settings by client tier

### 2. Credit Management System
- Client credit balance tracking
- Multiple deposit/prepayment methods
- Credit expiration policies (optional)
- Refund/withdrawal capabilities

### 3. Auto-Application Logic
- Intelligent credit application to new invoices
- Manual override controls for agency staff
- Credit hold functionality for disputed amounts
- Partial payment handling

### 4. Reporting & Visibility
- Client-facing credit balance dashboard
- Agency credit management interface
- Financial reporting integration
- Credit history and transaction logs

## Technical Requirements

### Backend Implementation
```
/client-credit/
├── models/
│   ├── ClientCredit.js      // Main credit balance model
│   ├── CreditTransaction.js // Transaction history
│   └── DepositConfig.js     // Service/package deposit rules
├── services/
│   ├── credit.service.js    // Credit balance management
│   ├── deposit.service.js   // Deposit collection logic
│   └── application.service.js // Auto-application rules
└── controllers/
    └── credit.controller.js // API endpoints
```

### Frontend Components
- Client credit dashboard widget
- Deposit payment flow integration
- Agency credit management panel
- Credit application controls in invoice creation

### Database Schema
```javascript
ClientCredit: {
  clientId: ObjectId,
  currentBalance: Number,
  totalDeposited: Number,
  totalApplied: Number,
  lastUpdated: Date
}

CreditTransaction: {
  clientId: ObjectId,
  type: 'deposit' | 'application' | 'refund' | 'adjustment',
  amount: Number,
  invoiceId: ObjectId,
  description: String,
  createdBy: ObjectId
}
```

## Success Metrics

- **Primary:** 25% improvement in cash flow (deposits collected upfront)
- **Secondary:** 15% reduction in late payments (credit auto-application)
- **User:** 90% client satisfaction with deposit/credit visibility

## Acceptance Criteria

### MVP Phase 1: Basic Credit System
- [ ] Service deposit configuration interface
- [ ] Deposit collection during order placement
- [ ] Client credit balance display
- [ ] Manual credit application to invoices
- [ ] Basic transaction history

### Phase 2: Advanced Features
- [ ] Auto-application rules engine
- [ ] Package-level deposit requirements
- [ ] Credit expiration policies
- [ ] Refund/withdrawal workflows
- [ ] Comprehensive reporting

### Phase 3: Integration & Optimization
- [ ] Payment gateway integration for deposits
- [ ] Notification system for low balances
- [ ] Credit-based service restrictions
- [ ] Advanced financial reporting

## Dependencies

- Payment gateway integration (Stripe/PayPal deposit handling)
- Invoice generation system updates
- Client portal UI enhancements
- Financial reporting module modifications

## Risks & Mitigations

**Risk:** Complex payment flow causing user confusion
**Mitigation:** Clear UX flows with progress indicators and help tooltips

**Risk:** Credit application logic conflicts with existing billing
**Mitigation:** Phased rollout with extensive testing on staging environment

## Questions for Review

1. Should credits expire after a certain period?
2. What's the minimum deposit amount threshold?
3. How should we handle partial refunds for cancelled services?
4. Should we allow negative credit balances (credit extensions)?

---
**Created:** 2026-03-10 | **Owner:** Product Team | **Reviewer:** @mehzabeen_c