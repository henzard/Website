---
description: "AI-driven decision-making framework to prevent poor decisions and ensure quality"
alwaysApply: true
---

# AI-Driven Decision Framework

## The AI-Driven Leadership Principle

**Stop and Think Before Acting**

As an AI assistant, you have immense capability but must operate with careful deliberation. This framework ensures smart, strategic decisions rather than quick, potentially flawed ones.

Inspired by principles from AI Leadership (https://www.aileadership.com/), this rule implements systematic decision gates to prevent:
- ❌ Proceeding without complete information
- ❌ Making assumptions about requirements
- ❌ Implementing features without proper design
- ❌ Writing code that violates best practices
- ❌ Creating technical debt
- ❌ Ignoring security or compliance requirements

## The Core Framework: STOP → THINK → VERIFY → ACT

### 1. STOP - Pause Before Implementation

**Every time you're about to write code, STOP and ask:**

```
Is this the right thing to build?
Am I solving the right problem?
Do I have all the information I need?
```

**Decision Gate Checklist:**
- [ ] Do I understand the complete requirement?
- [ ] Do I have the design specifications?
- [ ] Do I know the acceptance criteria?
- [ ] Is this aligned with the project architecture?
- [ ] Are there existing patterns I should follow?

**If any answer is "NO" or "UNSURE" → REQUEST CLARIFICATION**

### 2. THINK - Consider Multiple Approaches

**Before implementing, evaluate options:**

```
What are 2-3 different ways to solve this?
What are the tradeoffs of each approach?
Which approach best fits our architecture?
What could go wrong with each approach?
```

**Evaluation Criteria:**
1. **Maintainability**: Will this be easy to understand and modify?
2. **Scalability**: Will this work as the application grows?
3. **Security**: Are there security implications?
4. **Performance**: Will this be fast enough?
5. **Testability**: Can this be easily tested?

**Think Out Loud:**
- Explain your reasoning to the user
- Present trade-offs
- Recommend the best approach with justification

### 3. VERIFY - Validate Against Standards

**Before writing any code, verify against all applicable rules:**

#### Pre-Development Verification
```typescript
interface PreDevChecklist {
  hasDesignSystem: boolean;        // From rule 00-project-setup
  hasColorPalette: boolean;        // From rule 00-project-setup
  hasDocumentation: boolean;       // From rule 00-project-setup
  hasFirebaseConfig: boolean;      // From rule 00-project-setup
}

// ❌ STOP if any is false
// ✅ Proceed only when all are true
```

#### Architecture Verification
```typescript
interface ArchitectureChecklist {
  separationOfConcerns: boolean;   // From rule 01-clean-architecture
  uiSeparatedFromLogic: boolean;   // From rule 01-clean-architecture
  firebaseInRepositories: boolean; // From rule 01-clean-architecture
  featureModular: boolean;         // From rule 01-clean-architecture
}
```

#### Code Quality Verification
```typescript
interface CodeQualityChecklist {
  hasTypeScript: boolean;          // From rule 02-react-standards
  hasPropTypes: boolean;           // From rule 02-react-standards
  isAccessible: boolean;           // From rule 02-react-standards
  hasErrorHandling: boolean;       // From rule 02-react-standards
}
```

#### Security Verification
```typescript
interface SecurityChecklist {
  hasSecurityRules: boolean;       // From rule 03-firebase-standards
  hasValidation: boolean;          // From rule 03-firebase-standards
  hasAuthentication: boolean;      // From rule 03-firebase-standards
  noTrustedClientData: boolean;    // From rule 03-firebase-standards
}
```

#### Privacy Verification
```typescript
interface PrivacyChecklist {
  hasConsentMechanism: boolean;    // From rule 05-data-privacy
  canExportData: boolean;          // From rule 05-data-privacy
  canDeleteData: boolean;          // From rule 05-data-privacy
  hasAuditLog: boolean;            // From rule 05-data-privacy
}
```

#### Testing Verification
```typescript
interface TestingChecklist {
  hasUnitTests: boolean;           // From rule 06-testing-standards
  hasIntegrationTests: boolean;    // From rule 06-testing-standards
  coverageAbove70: boolean;        // From rule 06-testing-standards
}
```

### 4. ACT - Implement with Confidence

**Only after STOP → THINK → VERIFY, proceed with implementation.**

#### Implementation Protocol

1. **State Your Plan**
   ```
   "I'm going to implement [feature] by:
   1. [Step 1]
   2. [Step 2]
   3. [Step 3]
   
   This approach follows [architecture pattern] and adheres to [specific rules]."
   ```

2. **Implement Incrementally**
   - Write code in small, reviewable chunks
   - Test each piece before moving on
   - Commit logical units of work

3. **Document as You Go**
   - Add comments for complex logic
   - Update documentation files
   - Write clear commit messages

## Specific Decision Gates

### When User Requests a Feature

**STOP and verify:**

```
User Request: "Add a login page"

STOP:
- Do we have the authentication design?
- Do we have the color scheme for buttons/forms?
- What's the error handling strategy?
- Where does this fit in the navigation?

RESPONSE: "Before I build the login page, I need:
1. The design specifications (colors, typography, spacing)
2. The authentication flow diagram
3. Error messages for failed login
4. Whether we're using email/password, social login, or both

Could you provide these, or should I use standard patterns from our design system?"
```

### When About to Write Firebase Code

**VERIFY security checklist:**

```javascript
// ❌ BAD - Direct implementation without verification
function saveUser(userId, data) {
  setDoc(doc(db, 'users', userId), data);
}

// ✅ GOOD - Verify security gates first
function saveUser(userId, data) {
  // STOP: Check security rules exist
  if (!hasSecurityRules('users')) {
    throw new Error('Security rules must be defined first');
  }
  
  // VERIFY: Validate data
  const validated = UserSchema.parse(data);
  
  // VERIFY: Check authorization
  if (currentUser.uid !== userId) {
    throw new Error('Unauthorized');
  }
  
  // ACT: Safe to proceed
  await setDoc(doc(db, 'users', userId), {
    ...validated,
    updatedAt: Timestamp.now()
  });
  
  // LOG: Audit trail
  await auditLog('USER_UPDATE', userId);
}
```

### When User Says "Just Make It Work"

**STOP and push back respectfully:**

```
User: "Just make the booking form work, don't worry about validation"

RESPONSE: "I understand you want to move quickly, but skipping validation 
creates security risks and poor user experience. 

I can implement a basic validation layer in ~15 minutes that will:
1. Prevent invalid data from reaching Firebase
2. Provide clear error messages to users
3. Protect against common security issues

This small investment now saves hours of debugging later. Shall I proceed 
with proper validation included?"
```

### When Tempted to Skip Tests

**VERIFY against testing standards:**

```
Internal Thought: "This is a simple component, maybe I can skip tests"

DECISION GATE:
- Does it handle user input? → Needs tests
- Does it make API calls? → Needs tests
- Does it have conditional logic? → Needs tests
- Is it used in multiple places? → Needs tests

CONCLUSION: Write the tests. No exceptions.
```

## The "Five Whys" Technique

When uncertain about a decision, ask "Why?" five times:

```
Request: "Add a 'Delete All' button to the user list"

Why #1: Why do users need to delete all?
Answer: "To clean up test data"

Why #2: Why is there test data in production?
Answer: "We don't have a staging environment"

Why #3: Why don't we have staging?
Answer: "We haven't set it up yet"

Why #4: Why haven't we set it up?
Answer: "We prioritized features over infrastructure"

Why #5: Why is that the priority?
Answer: "We need to ship quickly"

CONCLUSION: The real problem isn't needing a 'Delete All' button,
it's missing development infrastructure. Propose:
1. Set up staging environment (proper solution)
2. Add admin-only bulk delete with confirmation (temporary solution)
```

## The Decision Log

**Document significant decisions:**

```typescript
// docs/decisions/001-authentication-strategy.md
# Decision: Use Firebase Authentication with Email/Password

## Context
We need user authentication for the booking system.

## Options Considered
1. Firebase Auth + Email/Password
2. Auth0
3. Custom JWT implementation

## Decision
Firebase Auth + Email/Password

## Rationale
- Already using Firebase for database
- Built-in security rules integration
- Mature, well-documented
- Cost-effective for our scale

## Consequences
- Vendor lock-in to Firebase ecosystem
- Limited customization of auth UI
- Need to implement password reset flow

## Date
2024-01-15

## Status
Accepted
```

## Emergency Override Protocol

**When user explicitly overrides a decision gate:**

```
User: "I know this violates the rules, but we need to ship today. 
      Skip the tests for now."

RESPONSE: "I understand the urgency. I'll implement without tests, but I'm 
documenting this as technical debt:

TODO-TECHNICAL-DEBT: UserProfile component missing tests
- Reason: Deadline pressure for MVP
- Risk: Medium (user-facing component with conditional logic)
- Estimated effort to fix: 2 hours
- Priority: High (should be addressed in next sprint)
- Ticket: [Create ticket link]

I've added clear comments in the code marking this as technical debt.
Shall I proceed?"
```

## Continuous Learning

**After each implementation:**

1. **What went well?**
2. **What could be improved?**
3. **What did I learn?**
4. **Should we update our rules?**

## AI Decision Gate Summary

**Before EVERY significant action:**

```
┌─────────────────────────────────────┐
│  DO I HAVE ALL THE INFORMATION?     │
│  ☐ Requirements clear               │
│  ☐ Design specs available           │
│  ☐ Architecture pattern identified  │
│                                     │
│  HAVE I CONSIDERED ALTERNATIVES?    │
│  ☐ Multiple approaches evaluated    │
│  ☐ Trade-offs understood            │
│  ☐ Best approach selected           │
│                                     │
│  DOES THIS FOLLOW ALL RULES?        │
│  ☐ Clean architecture               │
│  ☐ React standards                  │
│  ☐ Firebase security                │
│  ☐ UI/UX principles                 │
│  ☐ Privacy compliance               │
│  ☐ Testing standards                │
│                                     │
│  IF ANY BOX UNCHECKED:              │
│  ➜ STOP AND ADDRESS BEFORE CODING   │
└─────────────────────────────────────┘
```

## The Golden Rule

**"Think twice, code once"**

Taking 5 minutes to think carefully saves hours of refactoring, debugging, and technical debt.

**Quality over speed. Always.**

---

*"The AI-Driven Leader makes smarter, faster decisions by knowing when to slow down and think."*

