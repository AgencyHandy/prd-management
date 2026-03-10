# PRD: Package-Based Task Limits System

**Priority**: HIGH (Service Delivery Control)  
**Impact**: Revenue - Proper limits = sustainable pricing  
**Effort**: Medium  

## Problem Statement

Task request limits exist in the frontend but appear to be hardcoded or order-level configured rather than inherited from packages as designed. This prevents proper service delivery control and sustainable pricing models.

**Evidence**: `taskHelper` and `useTaskList` check limits, but no dedicated backend model for package-level limit configuration found.

## Business Impact

### Revenue Impact
- **Sustainable service delivery**: Prevent scope creep and overdelivery
- **Pricing model integrity**: Packages deliver exactly what's promised
- **Resource planning**: Predictable task volumes per package type
- **Upsell opportunities**: Clear upgrade path when limits reached

### Operational Impact
- **Service consistency**: All packages of same type have identical limits
- **Team planning**: Predictable workload based on package sales
- **Client expectations**: Clear limits communicated upfront
- **Scope management**: Automatic enforcement of agreed boundaries

## Success Metrics

### Primary KPIs
- **Limit adherence**: 95%+ of packages respect configured task limits
- **Upgrade conversion**: 15%+ of users hitting limits upgrade packages
- **Service delivery consistency**: <10% variance in task volume per package type
- **Client satisfaction**: No increase in scope-related disputes

### Tracking Events
- `task_limit_reached` (package_id, client_id, limit_type)
- `task_limit_exceeded_attempt` (package_id, reason, action_taken)
- `package_upgrade_prompted` (from_package, to_package, trigger_reason)
- `task_limit_reset` (package_id, reset_cycle, tasks_processed)

## Technical Implementation

### Backend Package Configuration
1. **Package limit model**: Max tasks per cycle (monthly/bi-weekly/quarterly)
2. **Reset cycle configuration**: Automated limit resets based on package terms
3. **Limit inheritance**: Orders automatically inherit package limits
4. **Override capabilities**: Admin ability to adjust individual order limits
5. **Grace period handling**: Soft limits with warnings before hard stops

### Database Schema
```sql
-- Package Task Limits
PackageTaskLimit {
  packageId: ObjectId,
  maxTasks: Number,
  resetCycle: Enum['monthly', 'biweekly', 'quarterly', 'annual'],
  resetDay: Number, // Day of month/cycle for reset
  gracePeriod: Number, // Extra tasks allowed before hard stop
  isActive: Boolean
}

-- Order Task Tracking
OrderTaskUsage {
  orderId: ObjectId,
  packageId: ObjectId,
  currentCycle: Date,
  tasksUsed: Number,
  tasksRemaining: Number,
  graceTasksUsed: Number,
  nextResetDate: Date,
  lastUpdated: Date
}
```

### Frontend Integration
1. **Package setup UI**: Configure limits during package creation
2. **Task limit dashboard**: Show usage/remaining per order
3. **Limit warnings**: Alert clients approaching limits
4. **Upgrade prompts**: Suggest package upgrades when limits hit
5. **Admin overrides**: Manual limit adjustments for special cases

## Current System Analysis

### What Works ✅
- Task creation checks limits via `taskHelper` and `useTaskList`
- Order details show task request limits
- `allClientRequests/` page with bulk actions exists
- Client task request flow functional

### What's Missing ❌
- **Package-level limit configuration**: No admin UI to set limits per package type
- **Automated reset cycles**: No background job to reset limits monthly/bi-weekly
- **Usage tracking**: No persistent tracking of tasks used vs limits
- **Upgrade flow**: No system to prompt upgrades when limits reached
- **Grace period handling**: No soft limit warnings before hard stops

## Enhanced Features

### Package Limit Configuration
- **Admin package editor**: Set max tasks, reset cycles, grace periods per package
- **Template limits**: Common limit configurations for quick setup
- **Bulk limit updates**: Apply same limits across multiple packages
- **Limit scheduling**: Future effective dates for limit changes

### Client Limit Visibility
- **Usage dashboard**: Show tasks used/remaining in current cycle
- **Limit warnings**: Proactive notifications at 75%, 90%, 100% usage
- **Upgrade suggestions**: Compare current package vs available upgrades
- **Usage history**: Track limit usage patterns over time

### Administrative Controls
- **Limit override**: Temporary or permanent limit adjustments
- **Reset management**: Manual reset triggers for special circumstances
- **Usage reports**: Aggregate limit usage across all packages
- **Limit forecasting**: Predict upgrade needs based on usage trends

## User Flows

### Package Setup (Admin)
1. Admin creates/edits package
2. Sets task limit configuration:
   - Max tasks per cycle: 10
   - Reset cycle: Monthly (1st of month)
   - Grace period: 2 extra tasks
3. System validates configuration and saves
4. All future orders of this package inherit these limits

### Client Task Request (Within Limits)
1. Client requests task via portal
2. System checks current usage vs package limits
3. Usage: 7/10 tasks → request approved
4. Task created, usage counter incremented to 8/10
5. Client sees "2 tasks remaining this month" notification

### Client Task Request (At Limit)
1. Client requests task (current usage: 10/10)
2. System checks grace period: 2 extra tasks available
3. Shows warning: "Using 1 of 2 extra tasks. Consider upgrading."
4. Task approved with upgrade suggestion
5. Usage becomes 11/10 (1 grace task used)

### Limit Reset (Automated)
1. Monthly cron job runs on 1st of month
2. Finds all orders with monthly reset cycles
3. Resets task counters to 0/max_tasks
4. Sends "Monthly task allowance renewed" notifications
5. Clears grace task usage

## Implementation Plan

### Phase 1: Backend Infrastructure (1 week)
- Create PackageTaskLimit and OrderTaskUsage models
- Build package limit configuration API endpoints
- Implement limit checking logic in task creation
- Set up automated reset cycle background jobs

### Phase 2: Admin Configuration UI (3 days)
- Add task limit fields to package creation/edit forms
- Build limit configuration validation
- Create bulk limit management for existing packages
- Add limit override capabilities for individual orders

### Phase 3: Client Dashboard Integration (3 days)
- Show task usage/remaining in client portal
- Add limit warning notifications at key thresholds
- Build upgrade suggestion prompts when limits approached
- Create usage history tracking and display

### Phase 4: Advanced Features (1 week)
- Implement grace period handling with warnings
- Build comprehensive limit usage reporting
- Add limit forecasting based on usage patterns
- Create automated upgrade recommendation system

## Acceptance Criteria

### Must Have
- [ ] Package-level task limit configuration in admin panel
- [ ] Automated reset cycles (monthly, bi-weekly, quarterly)
- [ ] Real-time limit checking during task creation
- [ ] Client visibility of current usage vs limits
- [ ] Background job for automated limit resets

### Should Have
- [ ] Grace period handling with soft limit warnings
- [ ] Upgrade prompts when limits reached
- [ ] Admin override capabilities for special cases
- [ ] Usage reporting and analytics dashboard

### Could Have
- [ ] Limit forecasting and upgrade recommendations
- [ ] Template limit configurations for quick setup
- [ ] Advanced reset scheduling (specific days, custom cycles)
- [ ] Integration with billing system for automatic upgrades

## Dependencies & Risks

### Technical Dependencies
- **Background job scheduler**: Cron system for automated resets
- **Package configuration**: Enhanced package management system
- **Notification system**: Limit warnings and reset notifications

### Business Risks
- **Client pushback**: Strict limits may frustrate existing clients
- **Support burden**: Questions about limits and reset cycles
- **Grandfathering**: Existing orders may need special handling

### Technical Risks
- **Reset timing**: Timezone handling for global reset schedules
- **Limit synchronization**: Race conditions during concurrent task creation
- **Data migration**: Converting existing orders to new limit system

## Migration Strategy

### Existing Orders
- **Audit current limits**: Identify how limits are currently enforced
- **Default configurations**: Apply reasonable defaults based on historical usage
- **Grandfathering**: Option to maintain current behavior for existing clients
- **Gradual enforcement**: Phase in strict limits over 30-60 days

### Communication Plan
- **Client notifications**: Explain new limit visibility and benefits
- **Agency education**: Train agencies on configuring package limits
- **Documentation updates**: Update help docs with limit explanations

---

**Owner**: Backend + Frontend Teams  
**Reviewer**: @mehzabeen_c  
**Est. Completion**: 2.5 weeks  
**Revenue Impact**: Medium-High (service delivery optimization)