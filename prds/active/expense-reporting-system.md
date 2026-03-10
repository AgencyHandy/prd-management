# PRD: Expense Reporting System

**Priority**: MEDIUM (Business Intelligence Gap)  
**Impact**: Operations - Complete project profitability analysis  
**Effort**: Medium  

## Problem Statement

Time tracking exists with billable/non-billable categorization, but expense tracking is completely missing. This prevents accurate project profitability analysis and proper cost allocation to clients.

**Evidence**: No `expense` module, model, or routes found in backend. Time tracking exists but lacks the expense component needed for full cost analysis.

## Business Impact

### Business Intelligence Impact
- **Project profitability**: Cannot calculate true project margins
- **Client pricing**: No data for accurate service pricing
- **Cost allocation**: Unable to track project-specific expenses
- **Tax reporting**: Missing expense documentation for deductions

### Operational Impact
- **Resource planning**: No expense forecasting for future projects
- **Budget management**: Cannot track project expenses vs budgets
- **Client billing**: Unable to pass through legitimate project costs
- **Financial reporting**: Incomplete picture of project economics

## Success Metrics

### Primary KPIs
- **Expense tracking adoption**: 60%+ of agencies log project expenses
- **Profitability visibility**: 100% of projects have cost data available
- **Billable expense recovery**: 40%+ of billable expenses invoiced to clients
- **Reporting accuracy**: Monthly P&L reports include expense allocation

### Tracking Events
- `expense_created` (project_id, category, amount, billable)
- `expense_approved` (expense_id, approver_id, approval_time)
- `billable_expense_invoiced` (expense_id, invoice_id, amount)
- `expense_report_generated` (date_range, project_count, total_expenses)

## Technical Implementation

### Backend Expense System
1. **Expense model**: Amount, category, description, receipts, billable flag
2. **Expense categories**: Predefined + custom categories per company
3. **Approval workflow**: Optional expense approval before client billing
4. **Receipt storage**: File attachments for expense documentation
5. **Project allocation**: Link expenses to specific orders/tasks

### Database Schema
```sql
Expense {
  _id: ObjectId,
  companyId: ObjectId,
  userId: ObjectId, // Who logged the expense
  orderId: ObjectId, // Which project
  taskId: ObjectId, // Optional specific task
  amount: Number,
  currency: String,
  category: String, // Travel, Software, Materials, etc.
  description: String,
  expenseDate: Date,
  isBillable: Boolean,
  isApproved: Boolean,
  approvedBy: ObjectId,
  approvedAt: Date,
  receipts: [String], // File paths
  invoiceId: ObjectId, // If billed to client
  createdAt: Date,
  updatedAt: Date
}

ExpenseCategory {
  _id: ObjectId,
  companyId: ObjectId,
  name: String,
  isDefault: Boolean, // System vs custom categories
  isBillableDefault: Boolean,
  isActive: Boolean
}
```

### Frontend Components
1. **Expense logging form**: Quick entry with receipt upload
2. **Expense list/filter**: View and manage expenses by project
3. **Approval interface**: Review and approve team expenses
4. **Reporting dashboard**: Project profitability with time + expenses
5. **Client billing integration**: Add approved expenses to invoices

## User Flows

### Team Member Logs Expense
1. Goes to project/task detail page
2. Clicks "Add Expense" 
3. Fills form:
   - Amount: $47.99
   - Category: Software License
   - Description: "Figma subscription for client mockups"
   - Billable: Yes
   - Receipt: Upload screenshot
4. Expense saved, approval notification sent to manager

### Manager Approves Expenses
1. Receives notification of pending expenses
2. Goes to Expenses → Pending Approval
3. Reviews expense details and receipts
4. Approves with note: "Billable to client on next invoice"
5. Expense status updated, available for client billing

### Agency Creates Invoice with Expenses
1. Goes to create invoice for project
2. Invoice form shows:
   - Time entries: 40 hours @ $75/hr = $3,000
   - Approved expenses: $147.99 (3 items)
3. Selects expenses to include
4. Invoice total: $3,147.99
5. Client receives invoice with itemized time + expenses

### Project Profitability Analysis
1. Agency views project dashboard
2. Sees complete cost breakdown:
   - Revenue: $5,000 (invoice total)
   - Time cost: $2,000 (40hrs @ $50/hr internal rate)
   - Expenses: $147.99
   - Profit: $2,852.01 (57% margin)
3. Can drill down to expense categories and trends

## Expense Categories

### Default Categories
- **Travel**: Transportation, lodging, meals
- **Software**: Tools, licenses, subscriptions
- **Materials**: Physical supplies, equipment
- **Communication**: Phone, internet, video calls
- **Marketing**: Advertising, promotion costs
- **Legal**: Contracts, compliance, consultation
- **Training**: Courses, certification, conferences

### Custom Categories
- Agencies can create project-specific categories
- Category-level billable defaults
- Archive unused categories vs delete (data preservation)

## Reporting & Analytics

### Project Reports
- **Cost breakdown**: Time vs expenses by category
- **Profitability trends**: Margin analysis over time
- **Budget tracking**: Planned vs actual project costs
- **Client comparison**: Profitability across different clients

### Business Reports
- **Monthly P&L**: Revenue, time costs, expenses, profit
- **Expense trends**: Category spending patterns
- **Team utilization**: Time + expense allocation per person
- **Tax reporting**: Deductible expenses with receipt documentation

## Integration Points

### Time Tracking Integration
- **Combined reporting**: Time + expenses in unified project view
- **Task-level costs**: Expenses allocated to specific tasks
- **Billable consolidation**: Both time and expenses in client invoices

### Invoice System Integration
- **Expense line items**: Approved expenses added to invoices automatically
- **Receipt attachments**: Client can view expense receipts
- **Tax handling**: Expense taxes calculated correctly

### Accounting Integration (Future)
- **QuickBooks export**: Expense transactions with proper categorization
- **Receipt management**: Automated document organization
- **Audit trail**: Complete expense history with approvals

## Implementation Plan

### Phase 1: Core Expense System (1.5 weeks)
- Build expense model and API endpoints
- Create expense logging form and basic list view
- Implement receipt upload and storage
- Set up default expense categories

### Phase 2: Approval Workflow (3 days)
- Build approval interface for managers
- Add approval notifications via email/in-app
- Implement expense status tracking
- Create bulk approval capabilities

### Phase 3: Reporting Integration (1 week)
- Add expenses to existing project dashboards
- Build profitability analysis with time + expense data
- Create expense-specific reports and filtering
- Integrate with invoice generation workflow

### Phase 4: Advanced Features (3 days)
- Add custom expense categories
- Build expense forecasting and budgeting
- Create expense analytics and trending
- Implement bulk expense operations

## Acceptance Criteria

### Must Have
- [ ] Expense creation with receipt upload
- [ ] Expense approval workflow for team expenses
- [ ] Project-level expense tracking and reporting
- [ ] Integration with invoice generation
- [ ] Basic expense categories and filtering

### Should Have
- [ ] Custom expense categories per company
- [ ] Expense profitability analysis integrated with time tracking
- [ ] Bulk expense operations and management
- [ ] Expense budget tracking per project

### Could Have
- [ ] Mobile expense logging with photo capture
- [ ] Automated expense categorization using ML
- [ ] Integration with accounting software (QuickBooks)
- [ ] Expense forecasting based on historical patterns

## Dependencies & Risks

### Technical Dependencies
- **File storage**: Receipt upload using existing file system
- **Notification system**: Expense approval notifications
- **Invoice integration**: Adding expenses to existing invoice workflow

### Business Risks
- **Adoption resistance**: Teams may not want to log expenses
- **Approval bottlenecks**: Slow approval may frustrate team members
- **Client disputes**: Clients questioning billable expenses

### Technical Risks
- **Receipt storage**: File organization and long-term retention
- **Currency handling**: Multi-currency expense tracking
- **Audit requirements**: Expense modification tracking and history

## Rollout Strategy

### Soft Launch (Month 1)
- Enable for agencies already using time tracking actively
- Focus on project-based expenses (software, materials)
- Gather feedback on expense categories and workflow

### Education Phase (Month 2)
- Create expense tracking tutorials and best practices
- Showcase profitability analysis benefits
- Train agencies on client billing with expenses

### Full Rollout (Month 3)
- Enable for all agencies with project management features
- Announce expense tracking as premium feature differentiator
- Monitor adoption rates and optimize based on usage patterns

---

**Owner**: Backend + Frontend Teams  
**Reviewer**: @mehzabeen_c  
**Est. Completion**: 3 weeks  
**Revenue Impact**: Medium (better profitability analysis + client billing)