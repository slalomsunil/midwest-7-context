# CRITICAL: Content Separation Requirements
# This file contains ONLY behavioral rules for AI agents
# Project-specific information (language, frameworks, architecture) belongs in docs/
# DO NOT add: project descriptions, version info, or technical specifications here
# See docs/ for all project-specific documentation

# GitHub Copilot Instructions - AI Instruction Orchestration Hub

## Project Context
This is a React/Node.js chat tool with tone modification capabilities and cross-repository coordination requirements.

## Core AI Behavior Rules

### Communication Standards
- Provide clear, concise explanations for all code suggestions
- Ask clarifying questions when requirements are ambiguous
- Explain the reasoning behind architectural or design decisions
- Use professional tone while remaining approachable and helpful

### Session Management
- Break complex tasks into logical, manageable steps
- Clearly indicate when a task requires multiple steps or files
- Provide progress updates for multi-step implementations
- Summarize what was accomplished at the end of each session

### Code Quality Standards
- Follow established patterns and conventions found in the codebase
- Prioritize readability and maintainability over brevity
- Include appropriate error handling and edge case considerations
- Suggest improvements to existing code when relevant
- Write self-documenting code with meaningful variable and function names

### Collaboration Guidelines
- Respect existing code style and architectural decisions
- Suggest refactoring only when it provides clear benefits
- Consider the impact of changes on team members and existing functionality
- Document any breaking changes or migration requirements clearly

### Source Control Best Practices
- Write clear, descriptive commit messages
- Suggest appropriate file organization and structure
- Consider the impact of changes on version control and team workflows
- Recommend when changes should be split across multiple commits

## Cross-Repository Coordination Rules

### Full-Stack Feature Development
- Always consider both UI and Service layer implications when implementing features
- Check for existing Service APIs before suggesting new endpoint creation
- Ensure data structures align between React components and Node.js API responses
- Coordinate authentication and error handling patterns across layers

### API-First Development
- Design API contracts before implementation when building new features
- Validate that UI requirements match available Service capabilities
- Suggest Service enhancements when UI needs cannot be met with existing APIs
- Maintain consistency in request/response formats across all endpoints

### Cross-Layer Validation
- Verify that UI components properly handle all possible API responses
- Ensure error states are handled consistently across UI and Service layers
- Check that authentication flows work correctly across both repositories
- Validate that data validation rules match between frontend and backend

## Technology-Specific Guidance

For React development patterns, see: docs/technology-guide.md
For Node.js development standards, see: docs/technology-guide.md
For system architecture details, see: docs/architecture.md
For full-stack development workflows, see: docs/development-guide.md

## Implementation Priorities

1. **Service Layer First**: When building new features, implement Service APIs before UI integration
2. **Data Contract Alignment**: Ensure UI and Service agree on data structures and validation rules
3. **Error Handling Consistency**: Maintain consistent error handling patterns across layers
4. **Authentication Coordination**: Coordinate auth implementation between UI and Service repositories

## Context Organization

This repository serves as a centralized AI instruction hub for coordinating chat tool development across:
- UI Repository: React-based WhatsApp-style chat frontend (slalomsunil/midwest-7-ui)
- Service Repository: Node.js-based chat backend with tone transformation (slalomsunil/midwest-7-service)

Key chat tool features:
- Username-only authentication (no passwords)
- WhatsApp-style chat interface
- Message tone transformation (funny, playful, serious)
- Real-time messaging capabilities
- User discovery and profile management

All detailed project information is organized in the docs/ directory to maintain clear separation between behavioral rules (this file) and project-specific documentation.


## Storied work
Stories are stored under the `stories` directory.  There are three sub-directories that hold stories in various states.  New stories that have not been worked yet live in `todo`.  The current story being worked on is under `current`.  There will be a directory for under `current` to hold all files related to a given story, including the plan and spec.  Once the story is complete, the story directory is moved to the `archive` directory.
Ignore files under `stories/todo` and `stories/complete`

