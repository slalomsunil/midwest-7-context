# STORY-44444: Test Feature - Implementation Plan

## Summary
Create a hello world API to validate dev readiness.

---

## Implementation Overview

### Objective
Build a simple hello world API endpoint that can be used to test the development environment and API creation workflow.

### Repositories
- **Service Repository:** `slalomsunil/midwest-7-service` (Node.js backend)
- **UI Repository:** `slalomsunil/midwest-7-ui` (React frontend) - *Optional for this story*

---

## Implementation Steps

### Phase 1: Service Layer (Backend API)

#### Step 1: Create Hello World Endpoint
**File:** `midwest-7-service/src/routes/hello.routes.js` (NEW)

```javascript
// Create new route file for hello world endpoint
const express = require('express');
const router = express.Router();

/**
 * GET /api/hello
 * Simple hello world endpoint for testing
 */
router.get('/', (req, res) => {
  res.status(200).json({
    message: 'Hello World!',
    timestamp: new Date().toISOString(),
    status: 'success'
  });
});

module.exports = router;
```

#### Step 2: Register Route in Main App
**File:** `midwest-7-service/src/app.js` or `midwest-7-service/src/server.js` (MODIFY)

```javascript
// Add this with other route imports
const helloRoutes = require('./routes/hello.routes');

// Add this with other route registrations
app.use('/api/hello', helloRoutes);
```

#### Step 3: Add Route Test
**File:** `midwest-7-service/src/routes/__tests__/hello.routes.test.js` (NEW)

```javascript
const request = require('supertest');
const app = require('../../app'); // Adjust path as needed

describe('Hello World API', () => {
  describe('GET /api/hello', () => {
    it('should return hello world message', async () => {
      const response = await request(app)
        .get('/api/hello')
        .expect(200);

      expect(response.body).toHaveProperty('message', 'Hello World!');
      expect(response.body).toHaveProperty('status', 'success');
      expect(response.body).toHaveProperty('timestamp');
    });

    it('should return valid timestamp', async () => {
      const response = await request(app)
        .get('/api/hello')
        .expect(200);

      const timestamp = new Date(response.body.timestamp);
      expect(timestamp).toBeInstanceOf(Date);
      expect(timestamp.toString()).not.toBe('Invalid Date');
    });
  });
});
```

---

### Phase 2: UI Integration (Optional)

#### Step 4: Create Hello World Component (Optional)
**File:** `midwest-7-ui/src/components/HelloWorld.tsx` (NEW)

```typescript
import React, { useState, useEffect } from 'react';

interface HelloResponse {
  message: string;
  timestamp: string;
  status: string;
}

export const HelloWorld: React.FC = () => {
  const [data, setData] = useState<HelloResponse | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchHello = async () => {
    setLoading(true);
    setError(null);
    try {
      const response = await fetch('/api/hello');
      if (!response.ok) {
        throw new Error('Failed to fetch hello world');
      }
      const json = await response.json();
      setData(json);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchHello();
  }, []);

  return (
    <div className="hello-world">
      <h2>Hello World Test</h2>
      {loading && <p>Loading...</p>}
      {error && <p className="error">Error: {error}</p>}
      {data && (
        <div>
          <p><strong>Message:</strong> {data.message}</p>
          <p><strong>Status:</strong> {data.status}</p>
          <p><strong>Timestamp:</strong> {new Date(data.timestamp).toLocaleString()}</p>
        </div>
      )}
      <button onClick={fetchHello} disabled={loading}>
        Refresh
      </button>
    </div>
  );
};
```

---

## Testing Strategy

### Manual Testing Steps

1. **Start the Service**
   ```bash
   cd midwest-7-service
   npm start
   ```

2. **Test with cURL**
   ```bash
   curl http://localhost:3000/api/hello
   ```

3. **Expected Response**
   ```json
   {
     "message": "Hello World!",
     "timestamp": "2025-11-04T12:00:00.000Z",
     "status": "success"
   }
   ```

4. **Test with Browser**
   - Navigate to: `http://localhost:3000/api/hello`
   - Verify JSON response displays correctly

### Automated Testing

1. **Run Unit Tests**
   ```bash
   cd midwest-7-service
   npm test -- hello.routes.test.js
   ```

2. **Run All Tests**
   ```bash
   npm test
   ```

3. **Coverage Check**
   ```bash
   npm run test:coverage
   ```

---

## Validation Checklist

### Backend Validation
- [ ] Route file created and properly structured
- [ ] Route registered in main app file
- [ ] Endpoint responds with correct status code (200)
- [ ] Response includes all required fields (message, timestamp, status)
- [ ] Timestamp is in valid ISO format
- [ ] Unit tests written and passing
- [ ] No linting errors
- [ ] Code follows existing project conventions

### Integration Validation (if UI component added)
- [ ] Component fetches data successfully
- [ ] Loading state displays correctly
- [ ] Error handling works properly
- [ ] Data displays correctly in UI
- [ ] Refresh functionality works
- [ ] Component tests passing

### Documentation
- [ ] API endpoint documented (in README or API docs)
- [ ] Code comments added where needed
- [ ] Any environment variables documented

---

## Dependencies

### Required Packages (should already be installed)
- `express` - Web framework
- `supertest` - HTTP testing (dev dependency)
- `jest` - Testing framework (dev dependency)

### Optional Packages (if not present)
```bash
npm install --save-dev supertest jest
```

---

## Deployment Notes

### Development Environment
- No special configuration needed
- Endpoint should be available immediately after server restart

### Staging/Production
- Endpoint will be available at: `/api/hello`
- No database migrations required
- No environment variables needed
- Can be deployed with standard deployment process

---

## Rollback Plan

If issues arise:
1. Remove route registration from main app file
2. Delete route file and test file
3. Restart server
4. Previous functionality remains intact

---

## Success Criteria

✅ **Story is complete when:**
1. `/api/hello` endpoint returns proper JSON response
2. All unit tests pass
3. Manual testing confirms expected behavior
4. Code is peer-reviewed and merged
5. Endpoint accessible in development environment

---

## Notes

- This is a simple endpoint with no authentication required
- No database interaction needed
- No external API calls
- Minimal error handling needed (basic express error handling sufficient)
- Can be used as template for future API endpoints

---

## Questions/Clarifications Needed

- [ ] Should this endpoint require authentication?
- [ ] Should it be included in API documentation?
- [ ] Any specific response format standards to follow?
- [ ] Should there be rate limiting on this endpoint?

