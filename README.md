# PRD Management System

**Product Requirements Document (PRD) tracking and management for AgencyHandy revenue optimization**

## 🎯 Purpose

Track, manage, and accelerate PRDs from identification to deployment, focusing on features that drive toward $100K MRR.

## 📁 Structure

```
/prds/active/     - Current PRDs in development
/prds/completed/  - Shipped PRDs
/prds/backlog/    - Future PRDs awaiting prioritization
/templates/       - PRD templates and standards
/analysis/        - Feature gap analysis and insights
```

## 🚀 Current Active PRDs

### Critical (Revenue Blocking)
- [Workspace Branding Stability](prds/active/workspace-branding-stability.md) - Brand colors resetting
- [Post-Payment Automation Fix](prds/active/post-payment-automation-fix.md) - Form delivery failure  
- [Team Notification System](prds/active/team-notification-system.md) - @mentions not working

### High Priority (Workflow & Scale)
- [Service Module Core Workflows](prds/active/service-module-core-workflows.md) - Roles, pricing, lifecycle & UX

## 📊 Metrics Tracking

- **Revenue Impact**: Direct/Indirect revenue effect
- **Conversion Impact**: Trial→Paid conversion blockers
- **User Experience**: Workflow and satisfaction improvements
- **Development Effort**: Engineering complexity estimation

## 🔄 Workflow

1. **Gap Identification** - Intercom analysis, user feedback, team insights
2. **PRD Creation** - Using standard templates with clear acceptance criteria  
3. **Priority Assignment** - Revenue impact-based prioritization
4. **Development Tracking** - GitHub issues linked to PRDs
5. **Release Management** - Deployment coordination and user notification

## 🧠 PRD Authoring Rules (Roles, Impact, UX)

To keep PM decisions unambiguous and implementation aligned with the product:

1. **Always define roles explicitly**  
   - For each behavior (view, edit, publish, delete, cancel, purchase, filter, sort, etc.), list which roles are allowed or blocked.  
   - Use the concrete role names used in code: `superAdmin`, `admin`, `projectManager`, `assignee`, `client`, `member`.

2. **Call out forwarding / cross‑module impact**  
   - When a flow starts in one module and finishes in another (e.g. client cancels a service → agency sees notification → lands on Order), describe:
   - Which modules are involved.
   - What data is passed between them.
   - What “good” and “bad” paths look like for each role.

3. **Describe best UX, not just “happy path”**  
   - Include empty‑states, error states, and redirects (e.g. “after cancel, redirect client to Service Board with a success message”).  
   - Prefer PM‑level UX descriptions over low‑level implementation notes.

4. **Align with existing behavior before expanding scope**  
   - Wherever possible, PRDs should reference how the feature works today (based on code) and then describe **additions/changes**, instead of redefining behavior from scratch.

These rules apply to all new PRDs added to this repo and should be treated as a checklist for PMs before handing off to engineering.

## 🎯 Success Metrics

**Primary Goal**: Drive toward $100K MRR through strategic feature development

- Conversion rate improvements
- Customer satisfaction increases  
- Support ticket reductions
- Revenue attribution per PRD

---

*This system ensures no feature gaps slip through and every PRD contributes to revenue growth.*