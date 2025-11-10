# Online Users Feature

## Overview

The Online Users feature displays a real-time roster of currently authenticated users using a lightweight polling mechanism. The list automatically updates as users login and logout, providing a WhatsApp-inspired contact list UI.

## Architecture

### Backend (Service Layer)

**Database Schema:**
- Added `is_online` column to `users` table (INTEGER, default 0)
- Created index on `is_online` for efficient querying
- Updated `last_active` timestamp on user activity

**API Endpoint:**
- `GET /api/users/online?excludeUserId={id}`
- Returns JSON with list of online users
- Cache-Control header: `public, max-age=3`
- Excludes requesting user from results

**Presence Management:**
- Login endpoint marks user online (`is_online=1`)
- Logout endpoint marks user offline (`is_online=0`)
- Database methods: `markUserOnline()`, `markUserOffline()`, `getActiveUsers()`

### Frontend (UI Layer)

**Custom Hook: `useOnlineUsers`**

Located: `src/hooks/useOnlineUsers.js`

Features:
- Polling with configurable interval (default 5 seconds)
- Exponential backoff on errors (max 30 seconds)
- Visibility-based throttling (pauses when tab hidden)
- AbortController for request cancellation
- Manual refresh capability

State Management:
```javascript
{
  users: Array<User>,  // List of online users
  loading: boolean,    // Initial load state
  error: Error|null,   // Error if fetch fails
  refresh: Function    // Manual refresh trigger
}
```

**Component: `LoggedInUsersPanel`**

Located: `src/components/LoggedInUsersPanel.js`

UI Features:
- WhatsApp-inspired styling
- Avatar with initials or profile image
- Online indicator (pulsing green dot)
- User display name or username
- Last active timestamp
- Empty, loading, and error states
- Click handler for future chat initiation

Styling:
- Two-column layout (sidebar + main chat)
- Responsive design (stacks on mobile)
- Smooth animations for user join/leave
- Custom scrollbar styling

## Configuration

### Polling Interval

Default: 5000ms (5 seconds)

Can be customized when calling the hook:
```javascript
const { users, loading, error } = useOnlineUsers(userId, 10000); // 10 second interval
```

### Backend Caching

Cache headers allow browser/proxy caching for 3 seconds to reduce server load during rapid polling.

### Error Handling

**Backend:**
- Returns 500 on database errors
- Validates query parameters

**Frontend:**
- Exponential backoff: interval × 2^(error_count)
- Maximum backoff: 30 seconds
- Displays error state in UI
- Continues polling after errors

## Usage

### Integration

In `HomePage.js`:

```javascript
import { useOnlineUsers } from '../hooks/useOnlineUsers';
import LoggedInUsersPanel from './LoggedInUsersPanel';

const { users, loading, error } = useOnlineUsers(currentUser?.id);

<LoggedInUsersPanel 
  users={users}
  loading={loading}
  error={error}
  onUserClick={handleUserClick}
/>
```

### User Click Handler

Currently logs user selection. Future stories will implement chat initiation:

```javascript
const handleUserClick = (selectedUser) => {
  
  // Future: Initiate chat with selected user
};
```

## Testing

### Backend Tests

**Unit Tests (`__tests__/db/users.test.js`):**
- `markUserOnline()` - Sets is_online flag
- `markUserOffline()` - Clears is_online flag
- `getActiveUsers()` - Returns only online users
- `getActiveUsers({ excludeUserId })` - Filters out specified user
- Edge cases: empty results, rapid login/logout

**Integration Tests (`__tests__/integration/users-online-integration.test.js`):**
- GET /api/users/online returns correct users
- Exclude parameter filters properly
- Cache headers present
- Handles rapid polling efficiently
- Updates immediately on logout

### Frontend Tests

**Component Tests (`__tests__/LoggedInUsersPanel.test.js`):**
- Renders loading state
- Renders error state
- Renders empty state
- Displays user list correctly
- Click handler invoked
- Avatar initials calculated
- Online indicators rendered

**Hook Tests (`__tests__/useOnlineUsers.test.js`):**
- Initial fetch on mount
- Polling at specified interval
- Exponential backoff on errors
- Manual refresh function
- AbortController cleanup
- Visibility-based pausing
- Resume on tab visible

## Performance Considerations

### Server Load

- Database index on `is_online` for fast queries
- Cache headers reduce duplicate requests
- Filtering in SQL reduces payload size

### Client Performance

- Visibility API prevents background polling
- AbortController cancels pending requests
- Diff-based updates minimize re-renders
- CSS animations use GPU acceleration

### Network Optimization

- 3-second browser cache reduces requests
- Exponential backoff on errors
- Payload contains only necessary fields
- GZip compression (if enabled on server)

## Future Enhancements

1. **WebSocket Support**: Replace polling with real-time push notifications
2. **Pagination**: Handle large numbers of online users
3. **Status Messages**: Allow users to set custom status
4. **Typing Indicators**: Show when users are typing
5. **Read Receipts**: Track message read status
6. **User Search**: Filter online users list
7. **Favorites**: Pin frequently contacted users to top

## Troubleshooting

**Users not showing as online:**
- Verify login endpoint marks user online
- Check database `is_online` column value
- Confirm polling is active (check Network tab)

**Polling not working:**
- Check browser console for errors
- Verify API endpoint accessible
- Confirm no CORS issues
- Check visibility API not pausing

**High server load:**
- Increase polling interval
- Enable server-side caching
- Consider WebSocket migration
- Add rate limiting

## Related Documentation

- Service API: `midwest-7-service/docs/architecture.md`
- Component Patterns: `midwest-7-ui/docs/react-development-guide.md`
- Testing Guidelines: `midwest-7-ui/docs/testing-guidelines.md`
