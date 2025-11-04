# Full-Stack Development Guide

## Chat Tool Feature Development Workflow

### 1. User Story Analysis and Planning
**Before starting any chat feature implementation:**
- Input user story for chat functionality (messaging, tone selection, user profiles)
- Generate detailed specifications for the feature
- Create implementation plan with clear API contracts
- Identify which layers (UI, Service, or both) need changes for chat functionality
- Plan WhatsApp-style user experience and message flow

### 2. Service-First Development Pattern
**When building chat features that require new backend functionality:**
1. **Design the messaging API contract** - Define message, user, and tone transformation structures
2. **Implement Service endpoints** - Build messaging, user management, and tone processing first
3. **Generate and run unit tests** - Verify backend messaging logic works independently
4. **Generate and run integration tests** - Test complete messaging workflows
5. **Implement WhatsApp-style UI** - Build chat interface to consume messaging APIs
6. **Update Swagger documentation** - Document all API creates/updates for messaging
7. **End-to-end testing** - Validate complete chat functionality with tone modification

### 3. Chat UI Enhancement Pattern
**When building chat features that only require UI changes:**
1. **Verify existing messaging APIs** - Confirm Service provides needed chat data
2. **Implement WhatsApp-style components** - Build new chat interface elements
3. **Integrate with messaging Service** - Connect UI to existing chat endpoints
4. **Test chat user experience** - Validate messaging behavior and tone selection
5. **Ensure real-time functionality** - Test live chat and message delivery

## Full-Stack Feature Implementation Templates

### Template: Username-Only Authentication Feature
```
1. Service Layer:
   - POST /auth/login - Accept username, create profile if not exists
   - GET /users/:username - Get user profile by username
   - PUT /users/:username - Update user profile
   - No password validation (username-only authentication)

2. UI Layer:
   - Simple username input component
   - User profile creation/display
   - WhatsApp-style user management
   - Session management without tokens
   - User discovery and search

3. Integration:
   - Simple username-based authentication
   - Automatic profile creation for new users
   - Error handling for username conflicts
   - User session management
```

### Template: Chat Messaging Feature
```
1. Service Layer:
   - POST /conversations - Create new conversation
   - GET /conversations - Get user's conversations
   - POST /messages - Send message with tone selection
   - GET /conversations/:id/messages - Retrieve conversation messages
   - PUT /messages/:id/tone - Apply tone transformation to message
   - WebSocket support for real-time messaging

2. UI Layer:
   - WhatsApp-style conversation list
   - Chat interface with message bubbles
   - Tone selector component (funny, playful, serious)
   - Message input with real-time typing indicators
   - Message delivery status indicators

3. Integration:
   - Real-time message delivery via WebSockets
   - Tone transformation with loading states
   - Error handling for message failures
   - Optimistic UI updates for sent messages
```

### Template: Tone Transformation Feature
```
1. Service Layer:
   - POST /messages/:id/transform - Transform message with selected tone
   - GET /tones - Get available tone options (funny, playful, serious)
   - PUT /users/:id/preferences - Save user's default tone preferences
   - AI/ML integration for tone transformation logic

2. UI Layer:
   - Tone selection dropdown/picker component
   - Message preview with tone transformation
   - Tone preference settings in user profile
   - Visual indicators for tone-modified messages

3. Integration:
   - Real-time tone transformation preview
   - Error handling for transformation failures
   - Caching of transformed messages
   - User preference persistence
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