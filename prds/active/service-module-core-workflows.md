# PRD: Service Module Core Workflows & Access Control

**Priority**: CRITICAL  
**Impact**: Revenue / Service Operations / Client Experience  
**Effort**: High  
**Owner**: Product + Backend + Frontend  
**Status**: Draft

---

## 1) Problem Statement

The Service Module is a core revenue surface (service listing, purchase, cancellation, filtering, publishing, and pricing). Current behavior must be standardized across roles and order lifecycle states so agencies can operate safely while clients can reliably discover and purchase services.

This PRD defines the required service behaviors, role-based access, filtering/sorting UX, and 3‑decimal‑friendly pricing rules (per local currency), aligned with existing backend capabilities.

---

## 2) Goals

- Ensure reliable service edit/publish/unpublish flows with explicit role-based permissions.
- Enforce service deletion rules tied to order lifecycle.
- Deliver complete client-side service purchase/cancellation/proposal workflows.
- Provide robust filtering and sorting experience in the service listing.
- Respect local currencies where the minimum is “1 unit of that currency,” not 1.000 USD equivalent.

---

## 3) Roles in Scope (As Implemented Today)

- **SuperAdmin** (`role.role.name === 'superAdmin'`)
- **Admin** (`'admin'`)
- **Project Manager (PM)** (`'projectManager'`)
- **Assignee** (`'assignee'`)
- **Client** (`'client'`)
- **Member** (`'member'` – generic internal; no special service privileges)

All PRDs touching permissions must **explicitly list which of these roles** can view, edit, publish, delete, or trigger each behavior.

---

## 4) Functional Requirements

### 4.1 Service Edit & Publish Controls

- **Edit description**
  - A user with permission (SuperAdmin / Admin / PM) can edit an existing service description and save without error.
  - After saving a description, the same user can mark or unmark any checkbox‑type fields on the service and save successfully.

- **Minimum price per currency**
  - Minimum service/package price must be **1 unit of the selected currency** (e.g. 1 BDT, 1 USD, 1 EUR), not a USD‑converted equivalent.

- **Header image**
  - A user with edit permission can upload a service header image.

- **Visibility by role**
  - SuperAdmin, Admin, Client, PM can view published services.
  - Admin, SuperAdmin, PM can change publish/unpublish status of a service.
  - Clients must **not** see unpublished services anywhere in the client‑side experience.
  - Admin, SuperAdmin, PM can see all services (published + unpublished) in the service listing and details.
  - Assignee:
    - **Must not** see the general service listing/catalog page inside the portal.
    - May see service context only when attached to specific orders/tasks, as implemented today.
  - Member:
    - Cannot edit services.
    - Must not gain any additional service permissions beyond the current permission model.

### 4.2 Service Deletion Rules

- **Active vs. completed/cancelled orders**
  - A service **cannot** be deleted if it has any related order in an “active” state (e.g., Ongoing / Review).
  - A service **can** be deleted if all related orders are in a terminal state:
    - Completed
    - Cancelled
  - A service **can** be **unpublished** even when it has active orders (so new clients cannot purchase it, but existing orders continue).

### 4.3 Service Purchase & Cancellation Flows (Client)

- **Purchase**
  - Client can purchase a published service from the catalog.

- **Cancellation**
  - Client can cancel a purchased service **only** when the related order status is `Pending`.
  - After cancelling a purchased service, client is redirected to the **Service Board** (service listing area of the client experience).

- **Notifications**
  - When a client cancels a purchased service:
    - The agency receives a notification of the cancellation.
    - When an agency user (SuperAdmin/Admin/PM) clicks that notification, they are redirected to the corresponding order details page.

### 4.4 Proposal Flow

- Client can send a proposal for a service.
- When a client sends a proposal, it appears in the agency’s **Received Proposals** list in the proposal module.

### 4.5 Filtering

- **Filter UI**
  - Clicking `Filter` reveals add‑filter controls.

- **Filter by Status**
  - User can filter services by:
    - Published
    - Unpublished

- **Filter by Payment Type**
  - User can filter by payment type:
    - One‑time
    - Recurring

- **Filter by Price Range**
  - User can open a price‑range modal to select a minimum and maximum price.

- **Filter behavior**
  - Multiple filters can be applied at the same time.
  - User can delete an individual filter chip.
  - A `Clear all` action removes all active filters and resets the listing.

### 4.6 Sorting

- **Sort UI**
  - Clicking `Sort` reveals add‑sort controls.

- **Alphabetical**
  - User can sort A–Z and Z–A.
  - For A–Z/Z–A, user can choose to sort by:
    - Service name
    - Duration
    - Active order count

- **Other sort options**
  - Sort by date (Newest first).
  - Sort by date (Oldest first).
  - Sort by budget (Highest first).
  - Sort by budget (Lowest first).

---

## 5) Currency Behavior (3‑Decimal Friendly)

The Service Module must be compatible with the global 3‑decimal currency behavior, but **minimum price is expressed in local currency units**, not USD.

Key expectations:

- Display and formatting:
  - Amounts show correct decimal precision for the currency (including 3‑decimal currencies where applicable).
  - Currency codes and symbols are correct.
  - Zero and negative amounts display correctly.

- Payments and refunds:
  - Successful payments preserve full precision (no unexpected rounding).
  - Refunds and partial refunds maintain the same precision.

- Conversion:
  - Conversion between 3‑decimal and 2‑decimal currencies does not introduce rounding drift in billing, payment, or reporting.

- Validation:
  - Inputs with more decimal places than supported by the currency are rejected with clear, user‑friendly errors.

The detailed 3‑decimal QA matrix lives in a separate payment PRD and QA checklist; Service Module must not introduce behavior that conflicts with that specification.

---

## 6) Acceptance Criteria

### A) Service CRUD / Permission / Lifecycle

- [ ] Existing service description can be edited and saved without errors (by SuperAdmin/Admin/PM).
- [ ] Checkbox‑type flags on a service can be toggled and saved after description edits.
- [ ] Minimum price validation enforces **1 unit of service currency** as the floor, not 1 USD equivalent.
- [ ] Header image upload works and image appears in the service view.
- [ ] SuperAdmin/Admin/Client/PM can view published services.
- [ ] Admin/SuperAdmin/PM can publish/unpublish services.
- [ ] Clients cannot see unpublished services in any client‑facing listing or detail view.
- [ ] Admin/SuperAdmin/PM can see both published and unpublished services.
- [ ] Assignee cannot access the main service listing page, but can still operate on services where context is provided via orders/tasks.
- [ ] Member cannot edit any service.
- [ ] Attempting to delete a service with an active order is blocked with a clear error.
- [ ] Services with only completed/cancelled orders can be deleted successfully.
- [ ] Unpublishing a service with active orders is allowed (no new purchases; existing orders unaffected).

### B) Client Purchase / Cancellation / Proposal

- [ ] Client can purchase a published service successfully from the catalog.
- [ ] Client can cancel a purchased service only while the order is in `Pending` status.
- [ ] After cancellation, client is redirected to the Service Board view.
- [ ] Agency receives a cancellation notification.
- [ ] Clicking the cancellation notification opens the related order details page for the agency user.
- [ ] Proposals sent by the client against a service appear in the agency’s **Received Proposals** list.

### C) Filter & Sort UX

- [ ] Filter panel appears on `Filter` click.
- [ ] Status filter supports Published and Unpublished options.
- [ ] Payment type filter supports One‑time and Recurring.
- [ ] Price range modal allows selecting valid min/max values for the current currency.
- [ ] Multiple filters can be combined and applied together.
- [ ] Individual filters can be removed.
- [ ] `Clear all` removes all filters and resets the listing.
- [ ] Sort controls appear on `Sort` click.
- [ ] Sorting by A–Z and Z–A works and supports Service, Duration, and Active order count.
- [ ] Sorting by date (Newest/Oldest) works.
- [ ] Sorting by budget (Highest/Lowest) works.

---

## 7) PM Expectations for Cross‑Module Impact & UX

For every Service‑related PRD:

- **Always specify roles**:
  - For each behavior (view, edit, publish, delete, cancel, purchase, filter, sort), list which roles are allowed or explicitly blocked.
- **Forwarding / impact analysis**:
  - PM must identify which other modules are impacted (e.g., Orders, Proposals, Payments, Notifications, Catalog, Redirects) and list:
    - Data dependencies (e.g., order status, payment status).
    - UX flows that start in Service but finish elsewhere (like order details or proposal inbox).
- **Best‑experience defaults**:
  - Default client flows should be fast, obvious, and prevent dead‑ends (e.g., always redirect to a meaningful page after cancel, purchase, or error).
  - Error states must be described with clear copy suggestions (not just “show error”).

These expectations should be followed for this PRD and any future Service Module expansions.

---

## 8) Dependencies

- Role and permission system (company roles as implemented in `role.service.js`).
- Order lifecycle and status model.
- Notification system and deep‑link routing to order details.
- Proposal module (Received Proposals list).
- Payment and currency infrastructure, including 3‑decimal support.
- Catalog/service listing UI and API filtering/sorting endpoints.

---

## 9) Risks

- Permission mismatches between backend checks and frontend visibility.
- Lifecycle races (status changes while a delete or cancel is in progress).
- Currency rounding issues if services make assumptions that differ from payment layer rules.
- UX regressions where clients or PMs land on dead‑end screens after actions.

---

