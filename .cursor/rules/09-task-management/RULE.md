---
description: "Task management, todo tracking, and structured work breakdown for complex features"
alwaysApply: true
---

# Task Management & Todo System

## 🎯 Core Principle

**"Big work needs a map. Small work needs focus."**

Never start complex features without breaking them down into manageable tasks. This prevents:
- ❌ Getting overwhelmed and stuck
- ❌ Forgetting critical steps
- ❌ Building features that break existing code
- ❌ Skipping testing or documentation
- ❌ Creating incomplete features

---

## 📋 The "3-Task Rule"

### When to Create a Todo List

```
IF feature requires MORE THAN 3 steps → CREATE TODO

Examples that NEED todos:
✅ "Add user authentication" (login, signup, password reset, profile, logout)
✅ "Build booking system" (calendar, form, payment, confirmation, email)
✅ "Implement data export" (gather data, format, download, audit log)
✅ "Create admin dashboard" (layout, users list, stats, actions)

Examples that DON'T need todos:
❌ "Fix button color" (single change)
❌ "Add logo to header" (single change)
❌ "Update contact email" (single change)
```

---

## 📁 Todo File Structure

### Location: `/docs/todos/`

```
project-root/
├── docs/
│   ├── todos/
│   │   ├── ACTIVE.md           # Current sprint/active work
│   │   ├── BACKLOG.md          # Future features
│   │   ├── COMPLETED.md        # Finished tasks (archive)
│   │   └── features/
│   │       ├── user-auth.md    # Specific feature breakdown
│   │       ├── booking-system.md
│   │       └── admin-panel.md
│   ├── REQUIREMENTS.md
│   ├── DESIGN_SYSTEM.md
│   └── CHANGELOG.md
```

---

## ✅ Todo Template

### ACTIVE.md Format

```markdown
# Active Tasks

**Last Updated:** [Date]
**Sprint/Week:** [Week of Dec 19, 2025]

## In Progress

### [Feature Name]
**Status:** 🟡 In Progress
**Started:** [Date]
**Assignee:** [Name]
**Priority:** High/Medium/Low
**Estimated Time:** [Hours/Days]

**Tasks:**
- [x] Task 1 (completed)
- [x] Task 2 (completed)
- [ ] Task 3 (current)
- [ ] Task 4
- [ ] Task 5

**Blockers:**
- None / [Describe blocker]

**Related Files:**
- `src/features/auth/LoginForm.tsx`
- `src/services/authService.ts`

**Notes:**
[Any important context]

---

## Up Next

### [Next Feature]
**Priority:** High
**Estimated Time:** [Hours/Days]
**Dependencies:** [What must be done first]

**Tasks:**
- [ ] Task 1
- [ ] Task 2

---

## Blocked / Waiting

### [Feature Name]
**Blocked By:** [Reason]
**Expected Resolution:** [Date/Action needed]
```

---

## 🔍 Feature Breakdown Template

### Example: `/docs/todos/features/user-auth.md`

```markdown
# Feature: User Authentication

**Overview:** Complete authentication system with login, signup, and password management

**Requirements Reference:** REQUIREMENTS.md Section 3
**Design Reference:** DESIGN_SYSTEM.md - Forms & Buttons
**Related Rules:** 03-firebase-standards, 05-data-privacy

---

## Task Breakdown

### Phase 1: Foundation (Estimated: 4 hours)
**Status:** ✅ Completed

- [x] Set up Firebase Authentication
- [x] Create auth service layer
- [x] Define User type in TypeScript
- [x] Write security rules for users collection
- [x] Create auth context provider

**Files Created:**
- `src/config/firebase.ts`
- `src/services/authService.ts`
- `src/types/user.ts`
- `src/contexts/AuthContext.tsx`
- `firebase/firestore.rules` (users section)

**Tests Written:**
- `src/services/authService.test.ts`
- `src/contexts/AuthContext.test.tsx`

---

### Phase 2: Login UI (Estimated: 3 hours)
**Status:** 🟡 In Progress

- [x] Create LoginForm component
- [x] Add form validation (email, password)
- [x] Add loading states
- [x] Add error handling
- [ ] Add "Remember Me" functionality
- [ ] Add "Forgot Password" link
- [ ] Write component tests
- [ ] Accessibility audit

**Files:**
- `src/features/auth/components/LoginForm.tsx`
- `src/features/auth/components/LoginForm.test.tsx`

**Current Blocker:** None

---

### Phase 3: Signup Flow (Estimated: 4 hours)
**Status:** ⚪ Not Started

- [ ] Create SignupForm component
- [ ] Add email validation
- [ ] Add password strength checker
- [ ] Add terms acceptance checkbox
- [ ] Create user document in Firestore after signup
- [ ] Send welcome email
- [ ] Add email verification
- [ ] Write tests

**Dependencies:** Phase 2 must be complete

---

### Phase 4: Password Management (Estimated: 3 hours)
**Status:** ⚪ Not Started

- [ ] Create ForgotPassword component
- [ ] Implement password reset email
- [ ] Create ResetPassword component
- [ ] Add password change in profile
- [ ] Write tests

---

### Phase 5: Profile Management (Estimated: 2 hours)
**Status:** ⚪ Not Started

- [ ] Create Profile component
- [ ] Allow name/avatar updates
- [ ] Implement logout functionality
- [ ] Add session management
- [ ] Write tests

---

## Definition of Done

Before marking this feature complete:

☐ All phases completed
☐ All tests written and passing
☐ Security rules tested
☐ Accessibility audit passed
☐ Documentation updated
☐ Code reviewed
☐ No console errors/warnings
☐ Works on mobile and desktop
☐ Privacy policy covers auth data
☐ CHANGELOG.md updated

---

## Impact Analysis

### What This Touches
- Auth context (entire app)
- User profile storage
- Protected routes
- Header/navigation (login status)

### Potential Breaking Changes
- None (new feature)

### Migration Needed
- None

---

## Testing Checklist

- [ ] Unit tests for authService
- [ ] Integration tests for auth flow
- [ ] Component tests for forms
- [ ] E2E test for complete signup/login/logout flow
- [ ] Security rules tests
- [ ] Accessibility tests

---

## Notes & Decisions

**2025-01-15:** Decided to use email/password instead of social login initially
**2025-01-16:** Added email verification requirement per privacy requirements
```

---

## 🚨 Before Starting ANY Feature

Run through this checklist:

```
BEFORE YOU START:

Feature Understanding
☐ I understand what this feature does
☐ I know why it's needed (user story/requirement)
☐ I have acceptance criteria
☐ I know what "done" looks like

Impact Assessment
☐ I've reviewed existing code this will touch
☐ I've checked if similar functionality exists
☐ I know what this might break
☐ I've identified all affected files
☐ I've checked BACKLOG.md for related planned features

Planning
☐ I've broken the work into ≤8 sub-tasks
☐ Each sub-task is ≤2 hours of work
☐ I've identified dependencies
☐ I've estimated total time
☐ I've created a feature todo file (if complex)

Technical Preparation
☐ I've reviewed relevant rules (architecture, security, etc.)
☐ I know what services/repositories to use
☐ I've checked if new types/interfaces are needed
☐ I've planned the testing approach

Documentation
☐ ACTIVE.md is updated with this feature
☐ Feature todo file created (if needed)
☐ Related requirements documented

IF ANY ☐ UNCHECKED → Don't start coding yet!
```

---

## 🔄 Todo Workflow

### 1. Feature Requested/Planned

```markdown
**Add to BACKLOG.md:**

### [Feature Name]
**Priority:** [High/Medium/Low]
**Requested By:** [Client/User]
**Business Value:** [Why we need this]
**Estimated Effort:** [S/M/L or hours]
**Dependencies:** [What's needed first]

**Description:**
[What the feature does]

**Status:** 📝 Planned
```

### 2. Ready to Start

```markdown
**Move from BACKLOG.md to ACTIVE.md:**

1. Create feature breakdown in `/docs/todos/features/[name].md`
2. Break into phases and tasks
3. Identify all files that will be touched
4. Run impact assessment
5. Update ACTIVE.md with first phase tasks
```

### 3. During Development

```markdown
**Update ACTIVE.md daily:**

- Mark completed tasks with [x]
- Note any blockers
- Update time estimates if needed
- Document key decisions
- List files created/modified
```

### 4. Feature Complete

```markdown
**Move to COMPLETED.md:**

### [Feature Name]
**Completed:** [Date]
**Total Time:** [Actual time spent]
**Files Changed:** [Number of files]

**Summary:**
[Brief description of what was built]

**Lessons Learned:**
[What went well, what to improve]

**Update CHANGELOG.md with user-facing changes**
```

---

## 📊 Impact Assessment Template

Before implementing any feature that touches existing code:

```markdown
# Impact Assessment: [Feature Name]

**Date:** [Date]
**Assessed By:** [Name]

## Files That Will Be Modified

### Modified Files
- `src/components/Header.tsx` - Add user menu
- `src/App.tsx` - Add auth provider
- `src/types/user.ts` - Add new fields

### New Files
- `src/features/auth/LoginForm.tsx`
- `src/services/authService.ts`
- `src/contexts/AuthContext.tsx`

### Deleted Files
- None

## Impact on Existing Features

### Affected Features
1. **Navigation** - Will now show login status
   - Risk Level: Low
   - Breaking: No
   - Requires testing: Yes

2. **User Profile** - Will integrate with auth
   - Risk Level: Medium
   - Breaking: No
   - Requires migration: No

### Not Affected
- Booking system
- Payment processing
- Admin panel

## Database Changes

### New Collections
- `users` - User profiles and auth data

### Modified Collections
- None

### Firestore Rules Changes
- Add rules for `users` collection
- Update rules for `bookings` to check auth

## API Changes

### New Endpoints
- `authService.login()`
- `authService.signup()`
- `authService.logout()`

### Modified Endpoints
- None

### Deprecated Endpoints
- None

## Backwards Compatibility

☐ No breaking changes
☐ Breaking changes (requires migration)
☐ New feature (no compatibility concerns)

**If breaking:** [Describe migration plan]

## Testing Requirements

- [ ] Unit tests for new services
- [ ] Integration tests for auth flow
- [ ] Regression tests for affected features
- [ ] E2E tests for user journeys
- [ ] Security rule tests

## Rollback Plan

**If this feature breaks something:**
1. Revert commits: [commit hashes]
2. Restore files: [backup location]
3. Database rollback: [Not needed / Migration script]
4. User communication: [If users affected]

## Deployment Considerations

- [ ] Environment variables needed
- [ ] Firebase config changes
- [ ] Database migrations needed
- [ ] Third-party service setup (email, etc.)

## Approval

☐ Impact understood and documented
☐ Risks identified and mitigated
☐ Testing plan defined
☐ Rollback plan ready
☐ Ready to implement

**Approved By:** [Name]
**Date:** [Date]
```

---

## 🛡️ AI Protection Rules

### Rule 1: No Big Features Without Breakdown

**❌ DON'T:**
```
User: "Build a complete booking system"
AI: *starts coding immediately*
```

**✅ DO:**
```
User: "Build a complete booking system"
AI: "This is a complex feature with multiple components. 
Before I start coding, let me break this down:

1. Database schema for bookings
2. Calendar UI component
3. Booking form with validation
4. Payment integration
5. Confirmation emails
6. User's bookings list
7. Admin booking management

This is about 20-30 hours of work. Should I create a detailed 
todo breakdown in /docs/todos/features/booking-system.md first?"
```

### Rule 2: Check Before Building

**Before writing any code:**
```
AI must check:
1. Does this functionality already exist?
2. Is there similar code I can reuse?
3. What files will this touch?
4. What might this break?
5. Is this in BACKLOG.md already?
6. Are there related planned features?
```

### Rule 3: Update Todos in Real-Time

**As work progresses:**
```
After completing each task:
- Update ACTIVE.md with [x]
- Note any issues encountered
- Update time estimates
- Flag blockers immediately
```

### Rule 4: No "TODO" Comments Without Context

**❌ BAD:**
```typescript
// TODO: Fix this
function getData() {
  // TODO: Add validation
  return data;
}
```

**✅ GOOD:**
```typescript
// TODO(john, 2025-01-15): Implement rate limiting before production
// Related: docs/todos/features/security-hardening.md
// Priority: High
function getData() {
  // TODO(jane, 2025-01-15): Add zod validation for API responses
  // See: src/types/api.ts for schema definitions
  return data;
}
```

---

## 📝 Daily Todo Review

At the start of each development session:

```
DAILY CHECKLIST:

☐ Read ACTIVE.md - What's in progress?
☐ Check for blockers - Anything stuck?
☐ Review yesterday's progress - Mark completed tasks
☐ Identify today's focus - What's next?
☐ Check for conflicts - Anyone working on related code?
☐ Review any new requirements - Anything changed?

At end of session:
☐ Update ACTIVE.md with progress
☐ Mark completed tasks
☐ Note any blockers
☐ Commit code with meaningful messages
☐ Update feature todo file if applicable
```

---

## 🚀 Sprint Planning Template

For larger projects with multiple features:

```markdown
# Sprint: [Week of Date]

**Goals:** [What we want to accomplish this sprint]
**Team:** [Who's working on this]

## Sprint Backlog

### High Priority
1. [ ] Feature A - Phase 2 (8h) - @john
2. [ ] Feature B - Phase 1 (5h) - @jane
3. [ ] Bug fix: Login timeout (2h) - @john

### Medium Priority
4. [ ] Feature C - Research (3h) - @jane
5. [ ] Update documentation (2h) - @john

### Low Priority (If time permits)
6. [ ] Refactor user service (4h)
7. [ ] Add loading animations (2h)

## Carry Over from Last Sprint
- Feature A - Phase 1 (was 90% complete)

## Blocked Items
- Feature D - Waiting for API access from third party

## Sprint Capacity
- John: 30 hours
- Jane: 25 hours
- Total: 55 hours
- Planned: 45 hours (buffer for unexpected issues)

## Sprint Review Date
[Date and time]

## Definition of Done (Sprint Level)
- [ ] All high priority items complete
- [ ] All tests passing
- [ ] Documentation updated
- [ ] Code reviewed
- [ ] Deployed to staging
- [ ] Client/stakeholder demo completed
```

---

## 🎯 Success Metrics

This system is working when:

✅ No one is stuck wondering what to work on next
✅ Complex features are completed without missing pieces
✅ Nothing breaks unexpectedly
✅ Everyone knows the project status at a glance
✅ New team members can understand what's been done and what's next
✅ Features are consistently completed (not left 80% done)
✅ Time estimates become more accurate
✅ Less rework needed

---

## 💡 Pro Tips

### Tip 1: Time-Box Tasks
```
If a task takes >2 hours → Break it down further
If stuck >30 minutes → Add to blockers, move to next task
```

### Tip 2: Write Tomorrow's Todo Today
```
Before ending work, write first task for tomorrow:
"Tomorrow: Complete validation for SignupForm email field"
```

### Tip 3: Celebrate Completions
```
When moving feature to COMPLETED.md:
- Note total time
- Document learnings
- Appreciate the work done
```

### Tip 4: Review Weekly
```
Every Friday or start of week:
- Review ACTIVE.md vs reality
- Archive completed items
- Re-prioritize backlog
- Adjust estimates based on learnings
```

---

## 🚨 AI Decision Gate

**Before implementing complex features (>3 steps):**

```
┌─────────────────────────────────────────────┐
│  COMPLEX FEATURE CHECKLIST                  │
├─────────────────────────────────────────────┤
│  ☐ Feature broken into ≤8 tasks             │
│  ☐ Each task is ≤2 hours                    │
│  ☐ Todo file created in docs/todos/         │
│  ☐ ACTIVE.md updated                        │
│  ☐ Impact assessment completed              │
│  ☐ Existing code reviewed                   │
│  ☐ Related planned features checked         │
│  ☐ All affected files identified            │
│  ☐ Testing plan documented                  │
│  ☐ Definition of done defined               │
│                                             │
│  IF ANY ☐ UNCHECKED:                        │
│  ➜ CREATE TODO BREAKDOWN FIRST              │
│  ➜ DO NOT START CODING                      │
└─────────────────────────────────────────────┘
```

---

**Remember: "Hours spent planning saves days spent debugging."** 📋✅

