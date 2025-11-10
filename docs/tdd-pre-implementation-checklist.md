# TDD Pre-Implementation Checklist

## STOP! Before Writing ANY Code

Answer these questions:

### 1. Test Existence Check
```
Question: Do I have a test for this behavior?
Answer: [ ] YES → Proceed to #2
        [ ] NO  → WRITE TEST FIRST, then proceed
```

### 2. Test Failure Check
```
Question: Have I run the test and seen it FAIL?
Answer: [ ] YES → Proceed to #3
        [ ] NO  → RUN TEST NOW, see failure, then proceed
```

### 3. Failure Reason Check
```
Question: Did the test fail for the RIGHT reason?
Answer: [ ] YES → You can write implementation code
        [ ] NO  → Fix test, run again, see correct failure
```

## Quick TDD Validation

### Before touching implementation files:
```bash
# 1. Have you written the test?
ls -la **/*test.js | grep -i "your-feature"

# 2. Does the test fail?
npm test -- --testPathPattern="YourFeature"

# 3. Did you see red (FAIL)?
# If YES: ✅ Proceed with implementation
# If NO:  ❌ Fix test or write test first
```

## Common Scenarios

### Scenario A: User Requests New Feature
```
❌ WRONG:
User: "Add feature X"
You:  *writes implementation*

✅ CORRECT:
User: "Add feature X"
You:  "Let me write a test first that describes this behavior..."
      *writes failing test*
      "Test is failing as expected. Now implementing..."
      *writes minimal code*
      "Test now passes. Feature complete."
```

### Scenario B: User Reports Bug
```
❌ WRONG:
User: "Feature Y is broken"
You:  *fixes code*

✅ CORRECT:
User: "Feature Y is broken"
You:  "Let me write a test that reproduces this bug..."
      *writes test that fails due to bug*
      "Confirmed the bug with a failing test. Fixing..."
      *fixes code*
      "Bug fixed, test now passes."
```

### Scenario C: User Wants to Change Behavior
```
❌ WRONG:
User: "Change X to work like Y instead"
You:  *modifies implementation*
      *updates tests to match new code*

✅ CORRECT:
User: "Change X to work like Y instead"
You:  "Writing test for new behavior Y..."
      *writes test for Y - should fail*
      "Test fails as expected. Implementing Y..."
      *modifies implementation*
      "Test passes. Removing old X tests..."
      *removes tests for old behavior*
      "All tests pass. Change complete."
```

## Self-Intervention Prompts

### If you catch yourself typing implementation code:
```
STOP!
Ask: "Have I written and run the failing test?"
     NO → Delete what you just typed
          Write test first
          See it fail
          THEN write implementation
```

### If you catch yourself updating test expectations:
```
STOP!
Ask: "Am I changing the test to match my code?"
     YES → This is backwards!
           Revert test changes
           Either fix code to match test
           OR write new test for new behavior
```

### If you catch yourself skipping test runs:
```
STOP!
Ask: "When did I last run the tests?"
     More than 5 minutes ago → RUN TESTS NOW
     Haven't run them → RUN TESTS NOW
     Tests might be broken → RUN TESTS NOW
```

## Emergency TDD Recovery

### If you realize you wrote code without tests:

```
Step 1: STOP writing more code
Step 2: Identify what you just implemented
Step 3: DELETE the implementation (yes, really)
Step 4: Write the test for that behavior
Step 5: Run test, see it FAIL
Step 6: Re-implement (usually faster 2nd time)
Step 7: Run test, see it PASS
Step 8: Proceed with TDD from here
```

### Why delete working code?
- You need to see the test fail to know it's testing the right thing
- Without seeing red, you don't know if test is valid
- This is the price of breaking TDD - pay it and learn

## TDD Affirmation

Before starting any work, recite:

```
I will write the test first.
I will see it fail.
I will write minimal code to pass it.
I will run all tests.
I will refactor only after green.
```

## Integration Checkpoint

### After completing any task:
```
✓ All tests written BEFORE implementation?
✓ All tests seen failing BEFORE fixing?
✓ All tests passing now?
✓ No obsolete tests remaining?
✓ No untested code added?

If ALL YES: ✅ Good work!
If ANY NO:  ❌ Fix before moving on
```

---

**Remember**: TDD is not a suggestion. It's the mandatory development process.
**Breaking TDD = Technical Debt = Fix it now before it compounds**
