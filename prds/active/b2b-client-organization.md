# PRD: B2B Client Organization & Multi-Stakeholder Access

**Priority:** HIGH | **Impact:** Revenue (ACV expansion) + Retention | **Timeline:** Q2–Q3 2026

## Context & Problem

Many agencies serve **business clients** with multiple stakeholders: procurement, legal, marketing leads, and finance. The product already supports a **client company** record and **multiple client logins** linked to it (see [As-built baseline](#as-built-baseline-codebase)), but B2B buyers still expect:

- **Stronger company identity** (legal entity, richer tax / billing metadata) aligned with how **invoices and subscriptions** are stored
- **Per-user roles** (not just “another member of the company”) so collaborators do not see billing by default
- **Structured approvals** before orders are confirmed or invoices are issued
- **Procurement fields** (PO number, cost center) on **orders and invoices** and on PDFs

Without closing these gaps, agencies still rely on shared logins, email-side approvals, and manual PO tracking—hurting win rate for mid-market B2B and expansion within accounts.

**Revenue impact (directional):** Higher ACV and lower churn in B2B-heavy segments; target **+$10K–$20K MRR** over 12 months via plan tiering and fewer competitive losses.

## Solution Overview

**Evolve** the existing **`ClientCompany` / `ClientCompanyMember`** model (do not fork a parallel “organization” stack unless a hard migration is explicitly approved). Add **roles and permissions**, **approval workflows**, and **procurement fields** while preserving today’s **`clientCompany` pointers** across projects, tasks, quotes, orders, invoices, and activity.

Agencies keep **individual** clients unchanged; **company** clients gain optional B2B depth.

---

## As-built baseline (codebase)

These pieces **already exist** and any PRD work must stay consistent with them.

### Core entities (backend)

| Model | Path | Role |
|--------|------|------|
| `ClientCompany` | `agency-backend/server/clientCompany/clientCompany.model.js` | Company profile: `name`, `taxId`, address fields, `type` (`client` \| `lead`), `owner` → `Member`, scoped to workspace `Company` |
| `ClientCompanyMember` | Same file | Links `clientCompany` ↔ `client` (`Member`) — multiple portal users per company |
| `Member.parentClient` | `agency-backend/server/company/member/member.model.js` | Used for **quotation** “additional external users” (leads under a primary client); documented as reserved for future B2B |

### HTTP API

- **Base path:** `GET/POST /client-companies`, `GET/PUT/DELETE /client-companies/:clientCompanyId`, `GET/POST /client-companies/members`, `DELETE /client-companies/members/:clientCompanyMemberId`  
- **Mount:** `agency-backend/index.route.js` → `router.use('/client-companies', …)`

### Email / notifications

- Template keys include **`client-company-member-added`** and **`client-company-created-owner`** (`agency-backend/server/services/email.services.js`)

### Frontend (representative)

- `clientSelectionType` / `clientCompanyId` on client shapes (`agency-client/main/src/models/client.interface.ts`, `project.interface.ts`)
- Company client UX and member APIs: e.g. `pages/client/details/companyDetails/`, forms `forms/order/`, `forms/companyClient/`, `forms/clientGroup/` (`clientCompanyIds`)
- Tickets / order-task activity: `clientCompanyId` for loading company members for @mentions (`ticket/details/activitySection.tsx`, `order/.../activitySection.tsx`)

### Product / agent rules

- **Tickets, orders, B2B mentions:** `agency-client/.ai/rules/TICKETS_ORDERS_B2B.md` — prefer `clientCompanyId` on the ticket; fallback `memberId`; `GET /client-companies/members` accepts either query param

- **Member `type` (backend):** `individual` \| `company` \| `leadCompanyMember` — `agency-backend/server/company/member/member.model.js`, `member.validation.js`

---

## Client account type: Business vs Myself — conversion, catalog, signup, workspace visibility

This section captures **product requirements** that were missing from earlier drafts, plus an **as-built audit** (FE/BE) so engineering can implement without rediscovering touchpoints.

### Terminology (target UX ↔ data)

| UX label | Intended meaning | Map to existing concepts |
|----------|-------------------|---------------------------|
| **Myself** | Person is acting in a **personal** capacity | Align with **`individual`** client / solo `Member`; optional `companyName` empty or ignored |
| **Business** | Person represents an **organization** | Align with **business identity**: collect `companyName`, and/or tie to **`ClientCompany`** + `clientSelectionType: 'company'` on downstream orders/invoices where applicable |

Exact enum names (`clientAccountType`, reuse `Member.type`, etc.) are an **implementation decision**; the PRD requires **consistent behavior** across catalog, signup, and agency tools.

---

### 1. Convert individual client → company (business)

**Product requirement:** An agency user can **convert** or **re-associate** an existing **individual** client (`Member`) into a **company** context: e.g. create a `ClientCompany` (or select existing), attach the member via `ClientCompanyMember`, and **optionally** backfill `Project.clientCompany` / open orders per migration rules.

**As-built (gap):**

- **FE:** Individual profile (`pages/client/details/clientProfile/`) supports **edit/delete** via `apiHelper.company.memberUpdate` / `memberBulkDelete` — **no** first-class “Convert to company” or “Link to ClientCompany” wizard. Companies are created separately (`pages/client/create` → `apiHelper.clientCompany.create`) and members managed under **company details** (`pages/client/details/companyDetails/`).
- **BE:** Lead → client and lead-company flows exist (`clientCompany.service.js` e.g. lead company member conversion; `member.services.js` `convertLeadToClient`; `account.service.js` guards when converting leads in lead companies). **No** dedicated API named for “individual client → company migration.”

**Cross-module impact if built:** `ClientCompany`, `ClientCompanyMember`, `Member`, `Project`, `OrderSubscription`, `projectInvoice`, activity logs, client portal login context.

---

### 2. Catalog checkout — client information: type Business / Myself (**duplicates allowed**)

**Product requirement:** On **public catalog checkout**, the **client information** step exposes **Business** | **Myself**. **Duplicate policy (catalog):** **allow duplicates** per product definition — e.g. same email may submit multiple catalog purchases without being blocked as “already a client,” or duplicate business labels allowed where it does not create conflicting **canonical** CRM rows (spell out in implementation: *checkout identity* vs *workspace Member record*).

**As-built (gap):**

- **FE:** `pages/catalogCheckout/steps/clientInformation/index.tsx` + `common.ts` — merged schema includes optional **`companyName`** in billing address merge; **no** Business/Myself selector. `CompanyNameContent` is always shown for sign-up branch (non–in-app purchase).
- **BE:** Catalog → order / client creation pipeline must be traced per feature ticket; today there is **no** `clientAccountType` in checkout payload.

**Cross-module:** `pages/catalogCheckout/index.tsx`, purchase flow `pages/service/purchase/form/index.tsx`, any API that creates `Member` or order from catalog.

---

### 3. Public client portal signup — type Business / Myself (**unique; duplicates not allowed**)

**Product requirement:** **Public signup** (`/signup` on client portal) includes the same **Business** | **Myself** field. **Uniqueness:** **do not allow** duplicate **canonical** client identity — at minimum **email unique per workspace** for an active client account (aligned with current signup behavior).

**As-built:**

- **FE:** `pages/publicSignup/index.tsx` → `forms/signup` with `publicSignupSchema` in `forms/signup/common.tsx` — fields: firstName, lastName, email, password, confirmPassword only (**no** client type, **no** company name).
- **BE:** `account.validation.js` → `publicSignUp` body: `email`, `password`, `confirmPassword`, `firstName`, `lastName`, `roleName: 'client'`. `account.service.js` → `publicSignUp`: if `existingMember && existingMember.isConvertedClient === true` → **`Email already exists`** (duplicate blocked).

**Gap:** Persist type (and when **Business**, require **company name** and/or create/link `ClientCompany`) in `publicSignUp` + validation + `createMember` payload.

---

### 4. Workspace configuration — hide Business / Myself (and related fields)

**Product requirement:** Workspace admin can **hide** the client-type control (and optionally dependent fields, e.g. company name when Myself) for:

- **Catalog** checkout client-information step  
- **Public** signup portal  
- *(Optional)* **Internal** order / invoice **individual vs company** switch

**As-built (gap):**

- **BE:** `Company` branding/signup fields include `signUpPortalHeading`, `signUpPortalImage`, `signUpPortalText` (`company.validation.js`, `services/select-fields.js`) — **no** boolean to hide client-type UI.
- **FE:** `pages/settings/companySettingAndBranding/signupPortal/` edits portal copy/image only. **Order form:** `enableClientTypeSwitch` is **`true` hardcoded** on order create and edit (`pages/order/create/index.tsx`, `pages/order/index.tsx` edit dialog) — **not** read from workspace settings.

**Proposed settings (names indicative):**

- `showClientAccountTypeOnCatalogCheckout` (default `true` when feature ships)  
- `showClientAccountTypeOnPublicSignup` (default `true`)  
- `showClientTypeSwitchOnInternalOrder` (default `true`) — replaces hardcoded `enableClientTypeSwitch` when implemented  

---

### Summary matrix (requirements vs codebase)

| Capability | Catalog | Public signup | Agency: convert individual → company | Workspace hide toggle |
|------------|---------|---------------|--------------------------------------|------------------------|
| Business / Myself UI | Required; **dup allowed** | Required; **unique** | N/A (conversion is separate action) | Required |
| Implemented today | ❌ (optional company name only) | ❌ | ❌ | ❌ |

---

## Cross-module reference map

When changing B2B behavior, assume **all** of the following may need updates, QA, or explicit “no change” sign-off.

### Backend — documents & money

| Module | File(s) (indicative) | `clientCompany` usage |
|--------|----------------------|------------------------|
| Project | `project/project.model.js`, `project.controller.js`, `project.service.js` | `clientCompany` on project; ties work to company |
| Task | `task/task.model.js`, `task/task.controller.js`, `task/task.services.js` | Task-level `clientCompany`; resolution from project / membership; **visibility** for company members |
| Task list / aggregation | `services/task/task.service.js` | Populate / pipeline joins `ClientCompanyMember` for effective company on tasks |
| Quotation | `quotation/quotation.model.js`, `quotation.service.js`, `quotation.helper.js` | `clientCompany` on quote; **`parentClient`** on added leads |
| Order subscription | `orderSubscription/orderSubscription.model.js`, `orderSubscription.service.js` | `clientCompany` on subscription |
| Project invoice | `projectInvoice/projectInvoice.model.js`, `projectInvoice.service.js`, `projectInvoice.controller.js` | `clientCompany` on invoice |
| Invoice subscription | `invoiceSubscription/invoiceSubscription.model.js`, `invoiceSubscription.service.js` | `clientCompany` carried from invoice context |
| Voucher | `voucher/voucher.service.js` | Resolves access via `ClientCompanyMember` |

### Backend — collaboration & UX

| Module | File(s) (indicative) | Role |
|--------|----------------------|------|
| Comments | `comment/comment.controller.js` | Same membership / company resolution pattern as tasks (mentions, access) |
| Files | `file/file.controller.js` | Client-company–aware access path aligned with tasks |
| Activity log | `activity/activity.model.js` | Optional `clientCompany` on `ActivityLog` |
| Embed | `embed/embed.services.js`, `embed/embed.controller.js`, `embed.validation.js` | `clientCompanyIds` on embeds; validation and member ↔ company mapping |
| Client group | `clientGroup/clientGroup.model.js`, `clientGroup.service.js` | `clientCompanies[]` alongside `clients[]` |
| Sidebar | `sidebar/sidebar.services.js` | Filters navigation using companies the member belongs to |
| Service / catalog | `service/service.service.js` | Client-group logic intersects `clientCompanies` |

### Backend — identity & workspace

| Module | File(s) | Note |
|--------|---------|------|
| Client company CRUD | `clientCompany/*.js` | Source of truth for company profile and membership list |
| Member services | `services/company/member/member.services.js` | Invites, `parentClient`, conversions |
| Auth / role middleware | `middlewares/authenticationMiddleware.js`, `middlewares/companyRoleMiddleware.js` | Any new **client-side** roles must align with portal auth |

### Frontend — touchpoints for B2B changes

| Area | Location pattern | Why it matters |
|------|------------------|----------------|
| Order create / edit | `forms/order/`, `forms/companyClient/` | `clientCompanyId` on submit |
| Proposals | `pages/proposal/details/`, proposal stores | Company vs lead targeting |
| Tickets | `pages/ticket/details/*` | `clientCompanyId` in mapper + activity (mentions) |
| Order board / tasks | `pages/order/details/.../activitySection.tsx` | Company member queries |
| Client company detail | `pages/client/details/companyDetails/` | Members, invites |
| Client groups | `forms/clientGroup/` | `clientCompanyIds` |
| Embeds | `pages/embeddedApps/*` | Company-scoped embeds |
| API helper | `api/helper.ts` | `clientCompanyId` query params for members and related calls |
| **Catalog checkout** | `pages/catalogCheckout/steps/clientInformation/*`, `forms/catalogCheckout/` | Client capture; **extend** for Business/Myself + company policy |
| **Public signup** | `pages/publicSignup/`, `forms/signup/` | **Extend** schema + API body |
| **Public signup API** | `account.validation.js` (`publicSignUp`), `account.service.js` (`publicSignUp`), `account.route.js` (`/public-signup`) | Validation + persistence + uniqueness |
| **Workspace branding** | `company.validation.js`, `company.model.js`, `settings/companySettingAndBranding/signupPortal/` | New toggles for showing/hiding client type |
| **Order form client mode** | `pages/order/create/index.tsx`, `pages/order/index.tsx`, `forms/order/index.tsx` | Wire `enableClientTypeSwitch` to **Company** setting when added |

---

## Core Features (target state)

### 1. Client company profile (extend existing)

- Keep **`ClientCompany`** as the canonical record; add fields as needed (e.g. legal name vs display name, jurisdiction-specific tax labels) without breaking existing `taxId` / address fields
- **Invoice delivery** preferences at company level where they differ from today’s per-invoice `sentToClients`

### 2. Client users & roles (extend `ClientCompanyMember`)

- Today: junction only (`clientCompany` + `client` `Member`)
- **Add:** role enum (and optionally project scope) on `ClientCompanyMember` or a dedicated subdocument — **Admin**, **Collaborator**, **Billing**, **Read-only**, **Approver** (Approver can be phased)
- Invites: reuse existing member invite flows where possible; ensure email templates (`client-company-member-added`) reflect role

### 3. Permissions & visibility

- **Portal routes:** enforce billing vs. non-billing by role (new middleware or centralized guard)
- **Tasks / tickets / files / comments:** today’s logic uses **membership in `ClientCompanyMember`**; extending with roles must not regress `task.controller.js` / `comment.controller.js` / `file.controller.js` paths
- **Audit:** role changes and approval actions logged (extend `ActivityLog` or dedicated collection)

### 4. Approvals (net new)

- Workspace setting: require client-side approval before **order confirmation** and/or **invoice send** (pick one for MVP)
- Notifications + in-app state; agency sees status on order / invoice timeline
- **Do not confuse** with `review` module **`approveReviews`** (testimonial-style reviews) — naming and UI must stay distinct

### 5. Procurement fields

- **PO number** / **reference** / **cost center** on entities that already carry `clientCompany`: at minimum **order subscription** and **project invoice** (and PDFs / portal views)
- Respect `select-fields` / serialization used by invoice APIs (`services/select-fields.js`)

### 6. Client account type on intake surfaces (Business / Myself)

- **Catalog:** selector + validation; **duplicate-friendly** rules (document whether duplicates apply to email, company name, or checkout-only identity)
- **Public signup:** same selector; **enforce uniqueness** for workspace client records (email; extend if business tax ID, etc.)
- **Conversion:** agency flow to move an **individual** `Member` into a **company** model (`ClientCompany` + `ClientCompanyMember` + optional project/order backfill)

### 7. Workspace visibility of client type

- Company-level settings to **show or hide** the client-type control on catalog, public signup, and (optionally) internal order **individual vs company** switch — see **§4. Workspace configuration** under [Client account type: Business vs Myself](#client-account-type-business-vs-myself--conversion-catalog-signup-workspace-visibility) above

---

## User Stories

- **As an agency owner**, I want company-level settings and multiple logins with roles so our client’s real org structure is reflected.
- **As a client collaborator**, I want project and ticket access without seeing invoices or payment methods unless I have billing access.
- **As a client billing contact**, I want invoices and subscriptions clearly under our **ClientCompany** record for ERP reconciliation.
- **As an agency PM**, I want a named approver’s decision recorded before commitment, reducing scope disputes.
- **As a catalog buyer**, I want to say whether I’m **buying as myself or as a business**, without being blocked when I’ve bought before (**catalog duplicate policy**).
- **As a workspace admin**, I want to **turn off** the client-type question on signup or catalog when it doesn’t fit my positioning.

---

## Current vs. Required Behavior

### Current (as implemented)

| Area | Status |
|------|--------|
| Company record + tax ID + address | ✅ `ClientCompany` |
| Multiple client `Member` accounts per company | ✅ `ClientCompanyMember` |
| `clientCompany` on project, task, quotation, order subscription, invoices, invoice subscription | ✅ |
| Company-aware task access, mentions, files, comments | ✅ (membership-based, not role-based) |
| Client groups & embeds scoped by `clientCompanies` | ✅ |
| `Member.parentClient` for quote additional contacts | ✅ |
| Per-member **roles** (billing vs collaborator) | ❌ |
| **Approval** gates on order / invoice | ❌ |
| **PO / cost center** on commercial documents | ❌ (or verify field-by-field before claiming done) |
| **Business / Myself** on catalog client info | ❌ (optional `companyName` only) |
| **Business / Myself** on public signup | ❌ |
| **Convert individual client → company** (guided) | ❌ |
| **Workspace toggles** to hide client-type on catalog / signup / order | ❌ (`enableClientTypeSwitch` hardcoded on order UI) |

### Required

1. Role model on top of existing membership ✅  
2. Permission matrix in client portal ✅  
3. Optional approval workflow ✅  
4. Procurement fields on chosen documents + PDFs ✅  
5. Business/Myself on **catalog** (duplicates allowed) and **public signup** (unique) + persistence ✅  
6. Individual → company conversion flow + data migration rules ✅  
7. Workspace settings to hide client-type where needed ✅  

---

## Technical Requirements

### Strategy

1. **Prefer extending** `ClientCompany` and `ClientCompanyMember` over introducing parallel `ClientOrganization` collections unless migration is explicitly scoped.
2. Any new field on `ClientCompany` / commercial documents must be threaded through **populate** and **select-fields** patterns used by invoice/order APIs.
3. Changes to **who can see a task** must be regression-tested against `task.controller.js`, `services/task/task.service.js`, `comment.controller.js`, and `file.controller.js`.

### Data model (delta — conceptual)

```javascript
// Extend ClientCompanyMember (or equivalent)
ClientCompanyMember: {
  clientCompany: ObjectId,
  client: ObjectId,              // Member
  roles: ['admin' | 'collaborator' | 'billing' | 'read_only' | 'approver'],
  projectAccess: [ObjectId],     // optional Phase 2+
  invitedAt: Date,
  acceptedAt: Date
}

// New or extended settings
ClientCompany: {
  // ...existing fields...
  b2bSettings: {
    approvalRequired: Boolean,
    approvalTrigger: 'order' | 'invoice'
  }
}

// Procurement (where clientCompany already exists)
OrderSubscription / projectInvoice: {
  purchaseOrderNumber: String,
  costCenter: String,
  // optional externalReference: String
}

// Workspace (Company) — client-type visibility on intake surfaces
Company: {
  showClientAccountTypeOnCatalogCheckout: Boolean,
  showClientAccountTypeOnPublicSignup: Boolean,
  showClientTypeSwitchOnInternalOrder: Boolean
}

// Optional: checkout / signup payload (names indicative)
// clientAccountType: 'myself' | 'business'
```

### Backend layout (evolutionary)

- Primary work under existing `server/clientCompany/` plus targeted edits in `task/`, `projectInvoice/`, `orderSubscription/`, `middlewares/`, and portal auth
- New `approval` submodule **only if** state machine does not fit cleanly in order/invoice services

### Frontend

- Extend **company detail** and **portal settings** for roles and approvals
- Order / invoice forms: PO fields; approver UX when `approvalTrigger` applies
- Keep **`clientCompanyId`** contract stable for ticket/order activity and `api/helper.ts` callers
- Catalog `clientInformation` + public `SignupForm`: Business/Myself; respect **Company** visibility flags
- Replace hardcoded `enableClientTypeSwitch` with **company setting** when `showClientTypeSwitchOnInternalOrder` (or equivalent) is implemented

### Non-functional

- **Backward compatibility:** Members without roles default to current behavior (e.g. treat as full access or “collaborator + billing” until migrated)
- **Security:** Role changes audited; tests for billing isolation

---

## Success Metrics

- % workspaces with ≥1 `ClientCompany` having ≥2 `ClientCompanyMember` rows **and** at least one non-default role
- Attach rate of B2B / higher plan among new mid-market signups
- Support tickets down for “wrong person saw invoice” / “shared login”
- Median client approval time < 48h when feature enabled

---

## Acceptance Criteria

### Phase 0 — Client type, signup, catalog, workspace toggles, conversion

- [ ] **Catalog:** Business/Myself (or equivalent) in client-information step; **duplicate policy** documented and covered by tests (allowed vs what exactly may duplicate)
- [ ] **Public signup:** same control; **email** (and any other agreed key) **unique** per workspace for active clients; **Business** path persists `companyName` and/or `ClientCompany` linkage per spec
- [ ] **BE:** `publicSignUp` validation + `publicSignUp` service accept and persist client type; catalog endpoints pass through type to order/member creation
- [ ] **Workspace:** Company settings + API expose booleans to **hide** client-type on catalog and public signup; **FE** respects flags on `publicSignup` and catalog checkout
- [ ] **Internal order:** `enableClientTypeSwitch` driven by workspace setting (remove hardcode in `pages/order/create/index.tsx` / edit modal)
- [ ] **Conversion:** Documented flow from **individual** client detail (or bulk) to **company** membership; migration rules for `Project.clientCompany` / subscriptions agreed and implemented or explicitly out-of-scope with reason

### Phase 1 — Roles + procurement

- [ ] `ClientCompanyMember` supports roles; defaults preserve legacy behavior
- [ ] Client portal enforces billing vs. non-billing (and admin manage-users) against real routes
- [ ] PO / reference (and optional cost center) on **order subscription** and **project invoice**; visible agency + client (billing role); PDFs updated
- [ ] Regression pass: **tasks**, **tickets**, **comments**, **files**, **embeds**, **sidebar**, **vouchers** for company-linked clients

### Phase 2 — Approvals

- [ ] Workspace + company-level toggles; trigger = **order** XOR **invoice** for v1 (product picks)
- [ ] Approver notification; approve/reject with reason; timeline on order/invoice
- [ ] Agency override (permissioned + audited)

### Phase 3 — Scale

- [ ] Project-scoped access for collaborators (if not in Phase 1)
- [ ] Exports / webhooks for accounting (optional)
- [ ] Client portal SSO — **separate** security PRD if pursued

---

## Dependencies

- `ClientCompany` / `ClientCompanyMember` APIs and services
- `Member` invite and portal authentication
- **`publicSignUp`** stack: `account.route.js`, `account.validation.js`, `account.service.js`, `forms/signup/common.tsx`, `pages/publicSignup/index.tsx`
- **Catalog checkout** client step: `pages/catalogCheckout/steps/clientInformation/*`, service purchase / checkout API chain
- **`Company`** model + update APIs + `select-fields` / company fetch used by portal and app shell
- Project / task / comment / file access control paths listed in [Cross-module reference map](#cross-module-reference-map)
- Invoice PDF generation and `projectInvoice` / `orderSubscription` pipelines
- Email / notification infrastructure (`email.services.js` template keys)

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Permission regression on tasks/files/comments | Matrix tests per role; QA checklist from cross-module table |
| Invoice/order pipelines miss new fields | Thread through model, validation, select-fields, PDF |
| Confusion with review “approve” | Distinct copy and API naming from `approveReviews` |

---

## Open Questions

1. Default role mapping for **existing** `ClientCompanyMember` rows?
2. Should approvals block **agency** actions or only client-facing confirmation?
3. One `Member` in multiple `ClientCompany` records (holding companies) — supported in v1?
4. Pricing: included vs. B2B add-on?
5. Tax ID: format validation only vs. external verification?
6. **Catalog “duplicates allowed”** — exact rule: duplicate **email** on checkout only, duplicate **business name**, or multiple **guest** checkouts without `Member` creation until payment?
7. When **Myself** on public signup, force `Member.type = 'individual'`; when **Business**, always create `ClientCompany` or only when company name present?
8. **Conversion:** auto-link all historical projects for that `Member` to the new `ClientCompany`, or only from a cutoff date?

---

**Created:** 2026-04-05 | **Updated:** 2026-04-05 | **Owner:** Product Team | **Reviewer:** TBD  

**Revenue context:** Supports $100K MRR via B2B positioning, expansion, and reduced workaround churn.
