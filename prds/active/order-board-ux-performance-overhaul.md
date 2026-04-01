# PRD: Order Board UX, DnD, and Performance Overhaul

## 1) Context

The Order Board (`/order?view=board`) had major usability and reliability issues:
- Drag-and-drop behavior was unstable (cards getting stuck or snapping back).
- Board totals and per-column totals were inconsistent and confusing for end users.
- Rendering many orders caused sluggish UI due to high DOM load.
- "Load more" behavior and labels were unclear for users.

This PRD defines the product and engineering requirements for a stable, scalable, and understandable board experience.

## 2) Goals

- Provide reliable drag-and-drop for cross-column moves and same-column reordering.
- Ensure board and list views display consistent total order counts.
- Show exact per-status column totals to reduce confusion.
- Improve board performance for large datasets via pagination and reduced initial render load.
- Standardize UX and i18n behavior for board headers and load-more controls.

## 3) Non-Goals

- Full virtualization rewrite of board cards in this phase.
- New board statuses or workflow redesign.
- Backend schema migrations unrelated to ordering/counting.

## 4) User Problems to Solve

1. As a user, when I drag a card, it should move predictably and persist after refresh.
2. As a user, I should not see mismatched totals between list and board views.
3. As a user, I should clearly see how many items are in each status column.
4. As a user, the board should remain responsive even with many orders.
5. As a user, action labels (e.g., load more) should be properly translated.

## 5) Functional Requirements

### 5.1 Board Interaction
- Card drag/drop MUST work for:
  - cross-column status change
  - same-column reorder
- On drop, system MUST persist:
  - `status` (when changed)
  - `positionInBoard` (always for final position)

### 5.2 Data Fetching
- Board view MUST use per-status paginated queries.
- Query params MUST include and pass through API helper:
  - `status`
  - `sortedBy=positionInBoard`
  - `sortOrder=asc`
  - `limit`
  - `skip`

### 5.3 Counts and Clarity
- Top page total MUST be consistent between list and board views.
- Each board column header MUST show exact total for that status.
- Backend list response MUST include:
  - `filteredProjectsCount` (exact filtered count, pre-pagination)
- Empty columns MUST NOT show load-more control.

### 5.4 Localization
- Load-more label MUST use existing locale keys.
- Button labels/state MUST be translated:
  - `SHOW_MORE`
  - `LOADING`

## 6) Technical Requirements

### 6.1 Frontend
- Use shared/stable `KanbanBoard` component for DnD behavior.
- Avoid custom inline DnD wiring that recreates sortable/droppable entities every render.
- Keep card ids globally unique in board list data (`_id`).
- On board update success/failure, refetch board queries to avoid stale lanes/snap-back.

### 6.2 Backend
- Orders list API MUST support `sortedBy=positionInBoard`.
- Orders list projection MUST include `positionInBoard`.
- Orders list response MUST include `filteredProjectsCount`.

## 7) Performance Requirements

- Initial board load must fetch small paginated sets per status (target: 30 per lane).
- Load-more fetches should be lane-specific (not all lanes).
- UI should remain interactive under large order volume by capping initial rendered nodes.

## 8) UX Requirements

- Header must show status title + exact status count.
- Top total should reflect global total from API source, not just currently loaded cards.
- Load-more button:
  - visible only when `hasNextPage` for that lane
  - hidden for empty lanes
  - translated label and loading state

## 9) Acceptance Criteria

1. Drag/drop works without stuck cards in all statuses.
2. Dropped card remains in new status/position after refresh.
3. List and board top totals match for same filter/search context.
4. Column counts show exact per-status totals (not `30+` approximation).
5. Empty columns do not show load-more.
6. Load-more labels are translated.
7. No linter errors in touched files.

## 10) Risks and Mitigations

- Risk: Hidden query-helper param mismatch can silently break board data.
  - Mitigation: enforce typed API helper signatures for all board params.
- Risk: Duplicate ids in board dataset break DnD internals.
  - Mitigation: ensure status-filtered queries and dedupe safeguard.
- Risk: Totals confusion from mixing "loaded count" vs "global count".
  - Mitigation: define explicit source-of-truth rules for top total and column totals.

## 11) Rollout and Verification

- Deploy backend support (`positionInBoard` sorting + `filteredProjectsCount`) before/with frontend board changes.
- Verify in staging:
  - DnD behavior
  - persistence on refresh
  - list vs board total consistency
  - exact per-status column counts
  - load-more visibility/translation

## 12) Future Enhancements

- Card virtualization for very large lanes.
- Background reorder queue for high-frequency drag updates.
- Board analytics (move frequency, lane aging, bottleneck statuses).
