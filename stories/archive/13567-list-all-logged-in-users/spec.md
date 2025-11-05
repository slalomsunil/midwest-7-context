# Spec: 13567 – List All Logged In Users

## Summary
Surface a real-time roster of currently authenticated users using a lightweight polling loop so that any signed-in chat participant can quickly choose a conversation partner. The list must never include the viewing user’s own identity and should automatically drop users shortly after they sign out.

## Goals

- Display all users who are presently logged in to the chat experience.
- Keep the list synchronized with the live session state so new logins and logouts are reflected without manual refreshes, using short-interval polling.
- Provide a clean, WhatsApp-inspired presentation that aligns with the chat application’s existing styling.

## Non-Goals

- Implementing the downstream messaging workflow once a recipient is selected (covered by other stories).
- Persisting historical online/offline activity beyond the current session window.

## Functional Requirements

- When a user signs in and navigates to the main screen, they see a roster that contains every other logged-in user.
- The current user’s own account is excluded from the roster.
- If any user logs out, their entry disappears from the list immediately.
- List updates occur without disrupting the viewer’s ongoing interactions (no full-page reloads or input loss).

## UX & Interaction Notes

- Layout should mirror familiar chat contact lists (avatar, display name, and optional status snippet if readily available).
- Empty states should communicate when no other users are online (“No one else is online right now. Check back soon.”).
- Handle rapid join/leave churn gracefully; avoid flicker by diffing updates and animating entry/exit if feasible.

## Technical Approach

### Service Layer

- Introduce or reuse a repository method that returns the currently authenticated users excluding the requester (e.g., `getActiveUsers({ excludeUserId })`).
- Expose a lightweight authenticated endpoint such as `GET /api/v1/users/online` that returns the active roster on demand and is optimized for frequent polling.
- Ensure logout flows (manual sign-out or session timeout) immediately flag the user as offline so the roster stays accurate for the next poll.

### Frontend Layer

- Add a `LoggedInUsersPanel` React component that polls the online-user endpoint and renders the roster.
- Leverage context or a dedicated hook (e.g., `useOnlineUsers(currentUserId)`) to encapsulate the polling loop, filtering, caching, and cleanup.
- When the current user logs out, clear timers and cached state to prevent unnecessary requests.

### Refresh Strategy (Polling)

- Use a modest polling cadence (e.g., every 5 seconds) tuned to balance responsiveness with server load; back off automatically on repeated errors.
- Apply diffing between responses to avoid unnecessary re-renders and provide smooth transitions (e.g., fade in/out for joins and leaves).
- Respect browser visibility by pausing or slowing polling when the tab is hidden to reduce unnecessary requests.

## Data & Integration

- Active user records should expose: `userId`, `displayName`, optional `avatarUrl`, and `lastSeen` timestamp for UI display.
- Update the authentication/session store so that `login` and `logout` events maintain an `isOnline` flag (database or in-memory cache) that the polling endpoint can read quickly.
- Coordinate with any presence service already used by the chat experience to avoid duplicating state while keeping the polling response simple.

## Testing Plan

- Follow TDD: write failing tests for service filtering (excludes requester, removes logged-out users) before implementation.
- Unit tests for the service endpoint to validate correct payloads, enforce reasonable polling cadence limits, and cover edge cases (no users online, rapid logout/login).
- Component tests ensuring the roster renders sorted user lists, handles empty state, and updates when mocked polling responses change.
- Integration test covering login → roster visibility → logout removal within the happy path, ensuring poll-driven updates propagate fast enough for a good experience.

## Open Questions

- What polling interval best balances freshness with backend load (initial recommendation: 5 seconds, adjustable via config)?
- Do we need to display additional metadata (e.g., availability status, typing indicator) in the roster for parity with other chat views?
- Should the roster support server-side pagination if the active user count becomes very large?
