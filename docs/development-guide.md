# Full-Stack Development Guide

## Cross-Layer Feature Development Workflow

### 1. Feature Analysis and Planning
**Before starting any feature implementation:**
- Identify which layers (UI, Service, or both) need changes
- Define the data flow between UI and Service
- Check for existing Service APIs that meet the requirement
- Plan the user experience and API contract

### 2. Service-First Development Pattern
**When building features that require new backend functionality:**
1. **Design the API contract** - Define request/response structures
2. **Implement Service endpoint** - Build backend functionality first
3. **Test Service endpoint** - Verify backend works independently
4. **Implement UI integration** - Build frontend to consume the API
5. **End-to-end testing** - Validate complete feature functionality

### 3. UI Enhancement Pattern
**When building features that only require UI changes:**
1. **Verify existing APIs** - Confirm Service provides needed data
2. **Implement UI components** - Build new interface elements
3. **Integrate with Service** - Connect UI to existing endpoints
4. **Test user experience** - Validate UI behavior and error handling

## Full-Stack Feature Implementation Templates

### Template: User Authentication Feature
```
1. Service Layer:
   - POST /auth/login - Accept credentials, return JWT token
   - POST /auth/register - Create new user account
   - GET /auth/verify - Validate JWT token
   - Implement password hashing and JWT generation

2. UI Layer:
   - Login form component with validation
   - Registration form component
   - Authentication context/state management
   - Protected route components
   - Token storage and management

3. Integration:
   - API client with token attachment
   - Error handling for auth failures
   - Redirect logic after login/logout
```

### Template: Social Media Post Feature
```
1. Service Layer:
   - POST /posts - Create new post
   - GET /posts - Retrieve posts feed
   - GET /posts/:id - Get specific post
   - DELETE /posts/:id - Delete user's post
   - In-memory storage with user associations

2. UI Layer:
   - Post creation form component
   - Posts feed display component
   - Individual post component
   - Post actions (like, delete, edit)
   - Real-time feed updates

3. Integration:
   - API calls with proper error handling
   - Optimistic UI updates
   - Loading states for all operations
```

### Template: User Profile Feature
```
1. Service Layer:
   - GET /users/:id - Get user profile data
   - PUT /users/:id - Update user profile
   - GET /users/:id/posts - Get user's posts
   - File upload handling for profile images

2. UI Layer:
   - Profile display component
   - Profile edit form component
   - User posts display
   - Profile image upload component

3. Integration:
   - Form validation matching Service requirements
   - Image upload with progress indicators
   - Profile caching and updates
```

## Cross-Repository Coordination Patterns

### API-First Development
1. **Document API contracts** before implementation
2. **Share endpoint specifications** between UI and Service teams
3. **Version APIs** when making breaking changes
4. **Test API endpoints independently** before UI integration

### Data Contract Consistency
```javascript
// Example: Consistent data structures across layers

// Service Response
{
  "user": {
    "id": "string",
    "username": "string", 
    "email": "string",
    "createdAt": "ISO date string",
    "profileImage": "url string or null"
  }
}

// UI Component Props
interface UserProps {
  id: string;
  username: string;
  email: string;
  createdAt: string;
  profileImage: string | null;
}
```

### Error Handling Coordination
```javascript
// Service Error Response Format
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "User-friendly error message",
    "details": ["Specific validation errors"]
  }
}

// UI Error Handling Pattern  
try {
  const response = await apiCall();
  // Handle success
} catch (error) {
  if (error.code === 'VALIDATION_ERROR') {
    showFormErrors(error.details);
  } else {
    showGenericError(error.message);
  }
}
```

## Development Best Practices for AI-Assisted Coding

### When Working with AI Agents

1. **Provide complete context** - Include relevant API contracts and data structures
2. **Specify layer requirements** - Be clear about UI vs Service implementation needs
3. **Reference existing patterns** - Point to similar implementations in the codebase
4. **Request end-to-end thinking** - Ask AI to consider both UI and Service implications
5. **Validate cross-layer consistency** - Review generated code for proper integration

### Code Organization Standards

**Service Repository Structure:**
```
src/
├── controllers/     # Request handling logic
├── models/         # Data models and in-memory storage
├── routes/         # API endpoint definitions  
├── middleware/     # Authentication, validation, error handling
├── services/       # Business logic
└── utils/          # Helper functions
```

**UI Repository Structure:**
```
src/
├── components/     # Reusable UI components
├── pages/         # Route-level page components
├── hooks/         # Custom React hooks
├── services/      # API client and external services
├── context/       # React context providers
├── utils/         # Helper functions
└── types/         # TypeScript type definitions
```

### Testing Strategy

**Service Testing:**
- Unit tests for business logic functions
- Integration tests for API endpoints
- API contract validation tests
- In-memory database operation tests

**UI Testing:**
- Component unit tests with React Testing Library
- Integration tests for user workflows
- API mocking for frontend tests
- End-to-end tests for critical user paths

**Cross-Layer Testing:**
- API contract compliance tests
- Full-stack feature integration tests
- Error handling validation across layers
- Performance testing with realistic data volumes

## Troubleshooting Common Integration Issues

### API Integration Problems
1. **CORS Issues** - Ensure Service allows UI domain in CORS configuration
2. **Authentication Failures** - Verify JWT token format and expiration handling
3. **Data Format Mismatches** - Check request/response structure alignment
4. **Network Error Handling** - Implement proper timeout and retry logic

### Development Environment Setup
1. **Service Setup** - Start backend server with in-memory database
2. **UI Setup** - Configure frontend to point to local Service API
3. **Hot Reloading** - Ensure both UI and Service support development reloading
4. **Port Configuration** - Avoid port conflicts between UI and Service development servers