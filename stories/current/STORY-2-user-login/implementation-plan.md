# STORY-2-user-login Implementation Plan

## Overview
Implement username-only login system with separate login page, session persistence, and user display.

## Backend Implementation (midwest-7-service)

### 1. New Login Route (`/routes/auth.js`)
```javascript
POST /api/auth/login
- Input: { username }
- Logic: 
  - Check if user exists via usersDb.findByUsername(username)
  - If exists: return existing user
  - If not exists: create new user with usersDb.create(username)
  - Return: { user: {...}, success: true }
```

### 2. Database Enhancement
- The existing `users.js` database already supports `findByUsername()` - no changes needed
- The existing `create()` method supports username-only creation

### 3. Route Registration
- Create new `auth.js` route file
- Register in `app.js` as `/api/auth`

## Frontend Implementation (midwest-7-ui)

### 1. Session Management
- Create `src/utils/auth.js` for session storage utilities:
  - `saveUser(user)` - store in sessionStorage
  - `getUser()` - retrieve from sessionStorage  
  - `removeUser()` - clear session
  - `isLoggedIn()` - check if user exists

### 2. New Login Page (`src/components/LoginPage.js`)
- Simple form with username input and submit button
- Call login API endpoint
- On success: save user to session and navigate to homepage
- Handle loading states and errors
- WhatsApp-style design to match existing UI

### 3. Updated App Component (`src/App.js`)
- Check login status on app load
- Route logic:
  - If not logged in: show LoginPage
  - If logged in: show HomePage
- No router needed - simple conditional rendering

### 4. Updated HomePage (`src/components/HomePage.js`)
- Add username display in top right
- Get username from session storage
- Optional: Add logout functionality

### 5. New API Service (`src/services/auth.js`)
- `loginUser(username)` function to call login endpoint
- Handle API errors and network issues

### 6. CSS Updates
- `LoginPage.css` - WhatsApp-style login form
- `HomePage.css` - Add styles for username display area

## Implementation Steps

### Phase 1: Backend (midwest-7-service)
1. Create `/routes/auth.js` with login endpoint
2. Register auth routes in `app.js`
3. Add tests for login endpoint
4. Test manually with curl

### Phase 2: Frontend Core (midwest-7-ui)  
1. Create `src/utils/auth.js` session utilities
2. Create `src/services/auth.js` API service
3. Add tests for auth utilities and API service

### Phase 3: Frontend Components (midwest-7-ui)
1. Create `LoginPage.js` component with form
2. Update `App.js` with login flow logic
3. Update `HomePage.js` to display username
4. Add CSS styling for both components

### Phase 4: Integration & Testing
1. End-to-end testing of login flow
2. Test session persistence across page reloads
3. Test error scenarios (network issues, invalid input)
4. Cross-browser testing of sessionStorage

## API Contract

### Login Endpoint
```
POST /api/auth/login
Content-Type: application/json

Request:
{
  "username": "john_doe"
}

Response (200):
{
  "success": true,
  "user": {
    "id": 123,
    "username": "john_doe", 
    "display_name": null,
    "bio": null,
    "profile_image": null,
    "created_at": "2025-11-04T..."
  }
}

Response (400):
{
  "success": false,
  "error": "Username is required"
}
```

## Session Storage Schema
```javascript
// sessionStorage key: "chatApp_user"
{
  "id": 123,
  "username": "john_doe",
  "display_name": null,
  "bio": null, 
  "profile_image": null,
  "created_at": "2025-11-04T..."
}
```

## Testing Strategy

### Backend Tests
- Unit tests for auth route (happy path, validation, errors)
- Integration with existing user database functions
- API response format validation

### Frontend Tests  
- Auth utilities (session storage operations)
- LoginPage component (form submission, error handling)
- App component (login state routing logic)
- HomePage component (username display)
- Auth API service (network calls, error handling)

### E2E Tests
- Complete login flow: form → API → session → homepage
- Session persistence across page refreshes
- Username display after successful login

## Error Scenarios to Handle

### Backend
- Missing username in request body
- Database errors during user creation/lookup
- Malformed request data

### Frontend  
- Network connectivity issues
- API server unavailable
- Invalid server responses
- SessionStorage unavailable/disabled
- Empty/whitespace-only usernames

## Notes
- Keep existing HomePage functionality intact
- Login page should match WhatsApp visual style
- Session storage chosen over localStorage for security (clears on tab close)
- No passwords required per story requirements
- Username display in top right - simple text display initially