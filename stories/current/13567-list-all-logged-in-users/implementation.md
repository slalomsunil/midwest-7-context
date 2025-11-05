# Implementation Plan: 13567 – List All Logged In Users

## Overview

Deliver a real-time roster of active users that excludes the current user, stays synchronized via a lightweight polling loop, and presents a polished, WhatsApp-inspired UI.

## Milestones

1. **Presence Data Contract** – Confirm data shape and availability of `isOnline` / presence signals from the service layer.
2. **Backend Endpoint** – Provide `GET /api/v1/users/online` optimized for frequent polling and filtered by requester.
3. **Frontend Hook & Context** – Create `useOnlineUsers(currentUserId)` hook plus context wiring for reuse across chat surfaces.
4. **Roster UI Component** – Implement `LoggedInUsersPanel` with graceful empty/loading/error states.
5. **Polling Loop & Cleanup** – Ensure interval management, backoff on errors, visibility handling, and teardown.
6. **Testing & Validation** – TDD service and UI, verify behavior for join/leave churn and absence of current user.

## Detailed Tasks

- **Service Layer**
  - Extend session store / repository with `getActiveUsers({ excludeUserId })`.
  - Implement controller + route: `GET /api/v1/users/online`.
  - Integrate presence events to mark users offline immediately on logout/timeout.
  - Add unit/integration tests covering filtering, no-online-users edge case, and logout removal.

- **Frontend Layer**
  - Introduce hook `useOnlineUsers(currentUserId)` that:
    - Polls `/api/v1/users/online` on a configurable interval (default ~5s) with an immediate leading request.
    - Maintains stable state, filtering out `currentUserId` and diffing results to minimize re-renders.
    - Handles transient errors with exponential backoff, pauses when the tab is hidden, and exposes `loading`, `error`, `users`.
  - Build `LoggedInUsersPanel` component:
    - Displays avatar, display name, optional status.
    - Supports empty state messaging and subtle animations for join/leave.
    - Offers optional click handler to initiate chat (stub/prop for future stories).
  - Wire component into main chat screen, ensuring layout matches design guidance.
  - Add component tests (React Testing Library) for rendering, empty state, and dynamic updates via mocked hook.

- **Polling & Refresh**
  - Centralize interval creation in the hook and allow tuning via config/environment.
  - Diff successive payloads to drive minimal UI updates and optional entry/exit animations.
  - Ensure all timers/backoff handles are cleared on logout, unmount, or visibility changes.

- **Cross-Cutting Concerns**
  - Update shared TypeScript types / service DTOs for online user payload.
  - Document API contract in backend README or swagger.
  - Add developer notes for presence dev setup (e.g., environment variables).

## Testing Strategy

- **Backend**: Jest unit tests for repository filtering and response shaping; supertest integration for `/users/online` covering polling cadence guards and caching headers.
- **Frontend**: RTL tests for component states plus hook behavior with mocked timers to validate interval, backoff, and visibility pause.
- **End-to-End**: Simulate login/logout flow ensuring roster updates within the expected polling window and UI remains responsive.

## Risks & Mitigations

- **Polling load**: Frequent requests could add backend pressure—mitigate with caching headers, interval tuning, and visibility-based throttling.
- **Perceived latency**: Intervals that are too long may feel stale—ensure UX communicates freshness and default to ~5s (configurable).
- **Race conditions**: Multiple rapid logouts/logins could cause flicker—use stable keys, optimistic diffing, and animation guards.

## Definition of Done Alignment

- Acceptance criteria satisfied (live roster, excludes current user, auto-removes logouts).
- Tests (unit, integration) written and passing.
- Documentation and API contract updated.
- Code reviewed, deployed to staging, PO sign-off secured.
