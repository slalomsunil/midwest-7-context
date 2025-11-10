# TDD Enforcement Checklist for AI Agents

## CRITICAL RULE
**NEVER write implementation code before writing failing tests - NO EXCEPTIONS**

## Pre-Implementation Checklist

Before writing ANY implementation code, ask yourself:

- [ ] Have I written a test that describes the expected behavior?
- [ ] Have I run the test to confirm it fails?
- [ ] Have I seen the specific failure message that confirms what I need to implement?

## TDD Workflow - MANDATORY ORDER

### Phase 1: RED (Write Failing Test)
1. **Create test file** (if it doesn't exist)
2. **Write test describing desired behavior**
   - Use descriptive test names
   - Test ONE specific behavior
   - Include edge cases
3. **Run the test**
   - Confirm it fails
   - Read the failure message
   - Verify it fails for the RIGHT reason

### Phase 2: GREEN (Make Test Pass)
4. **Write minimal implementation**
   - Only write code to make THIS test pass
   - Don't add extra features
   - Keep it simple
5. **Run the test again**
   - Confirm it passes
   - Check for unintended side effects
6. **Run ALL tests**
   - Ensure no regressions
   - Fix any broken tests immediately

### Phase 3: REFACTOR (Improve Code)
7. **Clean up implementation**
   - Remove duplication
   - Improve names
   - Optimize if needed
8. **Run ALL tests again**
   - Ensure refactoring didn't break anything
   - All tests should still pass

## When Changing Existing Features

### Scenario: User Requests Feature Change
**Example**: "Change from badge to row highlighting"

#### ❌ WRONG Approach (What I Did)
```
1. Modify LoggedInUsersPanel.js
2. Update CSS
3. Update tests to match new implementation
```

#### ✅ CORRECT TDD Approach
```
1. Write NEW test: "should apply notification class to user row"
2. Run test → See it FAIL
3. Modify LoggedInUsersPanel.js to add getUserItemClass()
4. Run test → See it PASS
5. Write test: "should apply blinking class when isBlinking is true"
6. Run test → See it FAIL  
7. Update getUserItemClass() to handle blinking
8. Run test → See it PASS
9. Update CSS (no tests needed for styles, but verify visually)
10. Run ALL tests → All should pass
11. Remove old badge-related tests
12. Run ALL tests → All should still pass
```

## Red Flags - When You're Breaking TDD

### 🚨 STOP if you catch yourself:
- Writing implementation before tests
- Saying "I'll write tests after this works"
- Running the app to "see if it works" instead of running tests
- Modifying test expectations to match implementation
- Skipping test run between changes
- Adding multiple features in one test cycle

### ✅ Good Signs You're Doing TDD:
- You see red (failing) tests before green
- You write one small test at a time
- Each test focuses on ONE behavior
- You run tests frequently (after every small change)
- Tests drive your implementation decisions
- You're comfortable with small, incremental changes

## TDD for Different Scenarios

### Adding New Feature
```
1. Test: Feature doesn't exist → FAIL
2. Code: Add minimal feature → PASS
3. Test: Edge case → FAIL
4. Code: Handle edge case → PASS
5. Refactor: Clean up
6. Test: All pass
```

### Fixing Bug
```
1. Test: Reproduce bug → FAIL (proves bug exists)
2. Code: Fix bug → PASS
3. Test: Related scenarios → All pass
4. Code: No further changes needed
```

### Refactoring
```
1. Test: All existing tests → PASS (baseline)
2. Code: Refactor implementation
3. Test: All existing tests → PASS (prove behavior unchanged)
```

### Changing Behavior (like your request)
```
1. Test: New behavior → FAIL
2. Code: Implement new behavior → PASS
3. Test: Old behavior should be removed → Update/remove old tests
4. Test: All tests → PASS
5. Code: Remove old implementation code
6. Test: All tests → STILL PASS
```

## Quick Decision Tree

```
┌─────────────────────────────────┐
│ User asks for feature/change    │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│ Do tests exist for current      │
│ behavior?                       │
└────┬────────────────────┬───────┘
     │ NO                 │ YES
     ▼                    ▼
┌─────────────┐    ┌──────────────────┐
│ Write test  │    │ Write test for   │
│ for NEW     │    │ NEW behavior     │
│ behavior    │    │ (should fail)    │
└──────┬──────┘    └────────┬─────────┘
       │                    │
       └────────┬───────────┘
                ▼
       ┌─────────────────┐
       │ Run test        │
       │ (should FAIL)   │
       └────────┬────────┘
                ▼
       ┌─────────────────┐
       │ Implement code  │
       └────────┬────────┘
                ▼
       ┌─────────────────┐
       │ Run test        │
       │ (should PASS)   │
       └────────┬────────┘
                ▼
       ┌─────────────────┐
       │ Run ALL tests   │
       │ Fix failures    │
       └────────┬────────┘
                ▼
       ┌─────────────────┐
       │ Clean up old    │
       │ tests/code      │
       └────────┬────────┘
                ▼
       ┌─────────────────┐
       │ Run ALL tests   │
       │ (all pass)      │
       └─────────────────┘
```

## Self-Audit Questions

After completing work, ask:

1. **Did I write tests before implementation?**
   - If NO: Go back and retrofit tests, then re-implement

2. **Did I see each test fail before making it pass?**
   - If NO: Delete implementation, run test to see failure, then re-implement

3. **Did I run tests after each small change?**
   - If NO: Run full suite now, fix any issues

4. **Are all tests passing?**
   - If NO: Fix implementation or tests until all pass

5. **Did I remove obsolete tests?**
   - If NO: Clean up test suite

## TDD Commitment Statement

**I commit to:**
- ✅ Write tests BEFORE implementation code
- ✅ Run tests to see failures BEFORE writing code
- ✅ Write minimal code to make tests pass
- ✅ Run ALL tests frequently
- ✅ Refactor only after tests pass
- ✅ Never skip TDD "to save time"
- ✅ Treat RED → GREEN → REFACTOR as sacred

**If I break this commitment:**
- 🔄 Stop immediately
- 🔄 Delete untested implementation
- 🔄 Write the test first
- 🔄 Follow proper TDD cycle

## Integration with Copilot Instructions

This checklist should be referenced BEFORE starting any implementation work, especially when:
- User requests new features
- User reports bugs
- User requests behavior changes
- Refactoring existing code

**Remember: TDD is not optional. It's the MANDATORY development process for this project.**
