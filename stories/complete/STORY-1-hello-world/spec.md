# STORY-1 Hello World - Technical Specification

## Overview
Implement a full-stack "Hello World" feature to demonstrate AI-driven development across all three repositories.

## User Story
As a user, I want to see "Hello World" displayed on the homepage so that I can verify the application is working end-to-end.

## Acceptance Criteria
1. The UI homepage displays "Hello World" retrieved from the backend
2. The message is styled like WhatsApp interface
3. The backend serves the data from an in-memory database
4. Full test coverage for both UI and Service layers

---

## Backend Implementation (midwest-7-service)

### Database Schema
**Table: `temp`**
```sql
CREATE TABLE temp (
  id INTEGER PRIMARY KEY,
  key VARCHAR(50),
  value TEXT
);

-- Initial data
INSERT INTO temp (id, key, value) VALUES (1, 'greeting', 'Hello World');
```

### API Endpoint
**GET /api/hello**
- **Purpose**: Retrieve the greeting message from the database
- **Response**: JSON object with the greeting
- **Success Response (200)**:
```json
{
  "message": "Hello World"
}
```
- **Error Response (500)**:
```json
{
  "error": "Internal server error",
  "message": "Failed to retrieve greeting"
}
```

### Backend Files to Modify/Create
1. **`db/temp.js`** - Database operations for temp table
2. **`routes/hello.js`** - API endpoint handler
3. **`app.js`** - Register the new route
4. **`__tests__/db/temp.test.js`** - Unit tests for database operations
5. **`__tests__/routes/hello.test.js`** - Unit tests for API endpoint

---

## Frontend Implementation (midwest-7-ui)

### Component Structure
**Homepage Component**
- Replace entire homepage content
- WhatsApp-style design with:
  - Clean, minimal interface
  - Green accent colors (#25D366)
  - Rounded message bubble styling
  - Clean typography
  - Centered layout

### React Component Design
```jsx
// src/components/HomePage.js
- Fetch data from /api/hello on component mount
- Display loading state while fetching
- Show error state if API fails
- Display greeting in WhatsApp-style bubble
```

### Styling Approach
- WhatsApp-inspired design system:
  - Background: Light gray (#F7F8FA)
  - Message bubble: White with subtle shadow
  - Accent color: WhatsApp green (#25D366)
  - Typography: Clean, readable fonts
  - Responsive design for mobile-first

### Frontend Files to Modify/Create
1. **`src/components/HomePage.js`** - Main homepage component
2. **`src/components/HomePage.css`** - WhatsApp-style styling
3. **`src/App.js`** - Update to use new HomePage component
4. **`src/services/api.js`** - API service layer for backend calls
5. **`src/components/__tests__/HomePage.test.js`** - Component unit tests
6. **`src/services/__tests__/api.test.js`** - API service tests

---

## Error Handling & Best Practices

### Backend Error Handling
- Database connection failures
- Query execution errors
- Proper HTTP status codes
- Structured error responses
- Input validation (if applicable)

### Frontend Error Handling
- Network failures (API unavailable)
- Loading states during API calls
- Error boundaries for component failures
- Fallback UI for error states
- Retry mechanisms where appropriate

### Performance Considerations
- Efficient database queries
- Proper React rendering optimization
- Caching strategies if needed
- Mobile-responsive design

---

## Testing Strategy

### Backend Tests (Jest)
**Unit Tests:**
1. **Database Layer (`temp.test.js`)**
   - Test retrieving greeting from temp table
   - Test database connection handling
   - Test error scenarios (empty table, connection failure)

2. **API Layer (`hello.test.js`)**
   - Test GET /api/hello success response
   - Test error handling and status codes
   - Test response format validation

**Integration Tests:**
- End-to-end API flow with real database
- Test complete request/response cycle

### Frontend Tests (Jest + React Testing Library)
**Unit Tests:**
1. **Component Tests (`HomePage.test.js`)**
   - Test component renders correctly
   - Test loading state display
   - Test successful data display
   - Test error state handling
   - Test WhatsApp styling application

2. **Service Tests (`api.test.js`)**
   - Test API call functions
   - Test error handling in service layer
   - Mock HTTP responses

**Integration Tests:**
- Test complete user flow from UI to API
- Test error scenarios end-to-end

---

## Implementation Order

### Phase 1: Backend Foundation
1. Create `temp` table and initial data
2. Implement database operations (`db/temp.js`)
3. Create API endpoint (`routes/hello.js`)
4. Register route in `app.js`
5. Write and run backend unit tests

### Phase 2: Frontend Implementation  
1. Create API service layer (`services/api.js`)
2. Implement HomePage component with WhatsApp styling
3. Update App.js to use new component
4. Write and run frontend unit tests

### Phase 3: Integration & Testing
1. Run integration tests across both layers
2. Test error scenarios end-to-end
3. Validate WhatsApp styling and responsiveness
4. Performance testing and optimization

---

## Definition of Done
- [ ] Backend API endpoint returns greeting from database
- [ ] Frontend displays greeting with WhatsApp-style design
- [ ] All unit tests pass (>90% coverage)
- [ ] Integration tests pass
- [ ] Error handling works correctly
- [ ] Mobile-responsive design verified
- [ ] Code review completed
- [ ] Documentation updated

---

## Technical Dependencies
- **Backend**: Express.js, existing in-memory database setup
- **Frontend**: React 19, CSS modules or styled-components
- **Testing**: Jest, React Testing Library, Supertest (for API testing)
- **Styling**: CSS with WhatsApp color scheme and design patterns