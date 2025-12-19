---
description: "Documentation standards - when to document, what to document, and how much is enough"
alwaysApply: true
---

# Documentation Standards

## 🎯 Core Principle

**"Document decisions, not details. Explain why, not what."**

Good documentation:
- ✅ Saves time for future developers
- ✅ Explains context and decisions
- ✅ Helps onboard new team members
- ✅ Prevents repeated questions

Bad documentation:
- ❌ States the obvious
- ❌ Gets outdated immediately
- ❌ Nobody reads it
- ❌ Costs more time than it saves

---

## 📚 Required Documentation

### 1. README.md (Project Root) - REQUIRED

**Purpose:** Project overview and quick start

**Must include:**
```markdown
# [Project Name]

## What This Project Does
[One paragraph explaining the purpose]

## Tech Stack
- Frontend: React + TypeScript
- Backend: Firebase (Firestore, Auth, Storage)
- Styling: Tailwind CSS
- Testing: Jest + React Testing Library

## Quick Start

### Prerequisites
- Node.js 18+
- npm or yarn
- Firebase account

### Installation
\```bash
npm install
cp .env.example .env
# Edit .env with your Firebase config
npm run dev
\```

### First-Time Setup
1. Create Firebase project at console.firebase.google.com
2. Enable Authentication and Firestore
3. Copy config to .env
4. Deploy security rules: `npm run deploy:rules`

## Project Structure
\```
src/
├── components/    # Reusable UI components
├── features/      # Feature modules
├── services/      # Business logic
└── config/        # Configuration
\```

## Available Scripts
- `npm run dev` - Development server
- `npm run build` - Production build
- `npm run test` - Run tests
- `npm run lint` - Lint code

## Documentation
- [Requirements](docs/REQUIREMENTS.md)
- [Design System](docs/DESIGN_SYSTEM.md)
- [API Documentation](docs/API_DOCUMENTATION.md)

## Contributing
[If team project]

## License
[If applicable]

## Contact
[Your name/email]
```

**❌ Don't Include:**
- How JavaScript works
- How React works
- Every single file explanation
- Code examples (those go in code comments)

---

### 2. REQUIREMENTS.md (docs/) - REQUIRED

**Purpose:** What the project needs to do

**Created:** During requirements gathering (Rule 08a)

**Structure:**
```markdown
# Requirements Document

## Project Overview
[From discovery interview]

## User Stories
### Epic: User Authentication
- As a user, I want to create an account so I can book classes
- As a user, I want to log in so I can view my bookings
- As a user, I want to reset my password if I forget it

### Epic: Booking System
[etc.]

## Functional Requirements
1. Users can create accounts with email/password
2. Users receive email verification
3. Users can book classes from available schedule
[etc.]

## Non-Functional Requirements
- Page load time < 3 seconds
- Mobile responsive
- WCAG AA compliant
- GDPR compliant

## Success Criteria
[How we measure success]
```

---

### 3. DESIGN_SYSTEM.md (docs/) - REQUIRED

**Purpose:** Visual and interaction standards

**Created:** During project setup (Rule 00, 08b)

See Rule 00 for complete template.

---

### 4. CHANGELOG.md (Project Root) - REQUIRED

**Purpose:** Track user-facing changes

**Format:** Keep-a-Changelog format

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- New feature X

### Changed
- Improved Y

### Fixed
- Bug Z

## [1.1.0] - 2025-01-15

### Added
- User can export their data
- Email verification for new accounts
- Password strength indicator

### Changed
- Improved booking form validation
- Updated color scheme for better contrast

### Fixed
- Login page crash on Safari
- Mobile menu not closing on route change

## [1.0.0] - 2025-01-01

### Added
- Initial release
- User authentication
- Booking system
- Payment integration
```

**When to update:**
- After completing any user-facing feature
- When fixing user-visible bugs
- When making breaking changes
- Before each release/deployment

**❌ Don't include:**
- Internal refactoring (unless it affects performance)
- Code quality improvements
- Test additions
- Minor code style changes

---

### 5. API_DOCUMENTATION.md (docs/) - REQUIRED IF API/Services

**Purpose:** Document all services and their methods

```markdown
# API Documentation

## Authentication Service

### authService.login()
**Purpose:** Authenticate user with email and password

**Parameters:**
- `email` (string) - User email address
- `password` (string) - User password

**Returns:** `Promise<User>`

**Throws:**
- `auth/invalid-email` - Email format is invalid
- `auth/user-not-found` - No user with this email
- `auth/wrong-password` - Password is incorrect

**Example:**
\```typescript
const user = await authService.login('user@example.com', 'password123');
\```

**Related:**
- Uses Firebase Authentication
- See: src/services/authService.ts
- Security rules: firebase/firestore.rules

---

### authService.signup()
[Similar format]

## Booking Service
[etc.]
```

**Include:**
- All public methods
- Parameters and types
- Return types
- Possible errors
- Usage examples
- Related files/rules

**❌ Don't include:**
- Private helper methods
- Implementation details
- Full code listings

---

### 6. ARCHITECTURE.md (docs/) - RECOMMENDED FOR COMPLEX PROJECTS

**Purpose:** Explain high-level architecture decisions

```markdown
# Architecture Decisions

## Overview
This project follows clean architecture with strict separation of concerns.

## Layer Structure

### Presentation Layer (UI)
- React components
- Only handles UI rendering
- No business logic
- No direct Firebase calls

### Business Logic Layer
- Services (business rules)
- Custom hooks (state management)
- Validation (zod schemas)

### Data Access Layer
- Repositories (Firebase operations)
- All database calls isolated here

## Key Decisions

### Decision: Use Repository Pattern
**Date:** 2025-01-10
**Decided by:** Team

**Context:**
We need to isolate Firebase from business logic for testability.

**Decision:**
Implement repository pattern with all Firebase calls in repository classes.

**Consequences:**
- Easier to test (mock repositories)
- Can swap Firebase for another DB
- More boilerplate code
- Extra abstraction layer

**Alternatives considered:**
- Direct Firebase calls (rejected: hard to test)
- Services with Firebase (rejected: tight coupling)

### Decision: Use Context API not Redux
[etc.]
```

**When to create:**
- Team projects
- Complex applications
- When architecture isn't obvious
- When training new developers

---

## 💬 Code Comments - When and How

### ✅ DO Comment

**1. Why, Not What**

```typescript
// ✅ GOOD - Explains WHY
// Using setTimeout to prevent race condition where auth state
// updates after component unmounts, causing React warning
setTimeout(() => {
  setAuthState(newState);
}, 0);

// ❌ BAD - States obvious WHAT
// Set auth state
setAuthState(newState);
```

**2. Business Logic**

```typescript
// ✅ GOOD - Explains business rule
// Bookings within 24 hours cannot be cancelled per policy
// Updated: 2025-01-15 - Changed from 48h to 24h
if (hoursUntilBooking < 24) {
  throw new Error('Cannot cancel within 24 hours');
}
```

**3. Non-Obvious Code**

```typescript
// ✅ GOOD - Explains complexity
// Convert Firestore Timestamp to JS Date, handling null case
// Firestore returns null for unset timestamp fields
const loginDate = user.lastLoginAt?.toDate() ?? null;

// Using bitwise OR for rounding instead of Math.floor
// Performance optimization for large datasets
const rounded = (value | 0);
```

**4. Workarounds and TODOs**

```typescript
// WORKAROUND: Firebase SDK bug with Safari
// Related issue: https://github.com/firebase/firebase-js-sdk/issues/1234
// Remove when SDK v10.2.0 is released
if (isSafari()) {
  // Alternative implementation
}

// TODO(john, 2025-02-01): Refactor to use new payment API
// Priority: High
// Related: docs/todos/features/payment-v2.md
function processPayment() {
  // Current implementation
}
```

**5. Complex Algorithms**

```typescript
/**
 * Calculate booking price with dynamic pricing
 * 
 * Pricing rules:
 * - Base price from service
 * - +50% for weekend bookings
 * - -20% for bookings >2 weeks in advance
 * - +$10 for each add-on
 * - Round to nearest $5
 * 
 * @param service - Service being booked
 * @param date - Booking date
 * @param addOns - Selected add-ons
 * @returns Final price in cents
 */
function calculateBookingPrice(
  service: Service,
  date: Date,
  addOns: AddOn[]
): number {
  // Implementation with step-by-step comments
}
```

**6. Security Considerations**

```typescript
// SECURITY: Never send password reset token via query param
// Tokens must only be in POST body to prevent logging
async function resetPassword(token: string, newPassword: string) {
  // Implementation
}
```

### ❌ DON'T Comment

**1. Obvious Code**

```typescript
// ❌ BAD - Obvious
// Get user by ID
const user = await userService.getById(userId);

// ❌ BAD - Obvious
// Increment counter
counter++;

// ❌ BAD - Obvious
// Check if user is admin
if (user.role === 'admin') {
```

**2. Explaining Bad Code**

```typescript
// ❌ BAD - Instead of commenting bad code, fix it!
// This function does a lot of things. It fetches the user,
// validates the booking, checks availability, processes payment,
// sends emails, and logs everything.
function processBooking() {
  // 500 lines of code...
}

// ✅ GOOD - Split into focused functions
async function processBooking() {
  const user = await fetchUser();
  validateBooking(booking);
  await checkAvailability(booking);
  await processPayment(booking);
  await sendConfirmationEmail(user, booking);
  await logBooking(booking);
}
```

**3. Commented-Out Code**

```typescript
// ❌ BAD - Delete it, don't comment it (use git)
// const oldImplementation = () => {
//   // ...
// };

// ✅ GOOD - Just delete it (git history preserves it)
const newImplementation = () => {
  // ...
};
```

**4. Obvious Variable Names**

```typescript
// ❌ BAD - Name is self-explanatory
// User's email address
const userEmail = user.email;

// ✅ GOOD - Let TypeScript types document
const userEmail: string = user.email;
```

---

## 📝 JSDoc for Public APIs

### For Shared Functions/Services

```typescript
/**
 * Sends a booking confirmation email to the user
 * 
 * This function is called after successful booking creation.
 * Email template is defined in Cloud Functions.
 * 
 * @param userId - ID of user receiving email
 * @param bookingId - ID of confirmed booking
 * @param options - Optional email configuration
 * @param options.language - Email language (default: user's preference)
 * @param options.sendCopy - Send copy to admin (default: false)
 * 
 * @returns Promise resolving to email send status
 * 
 * @throws {Error} If user email is not verified
 * @throws {Error} If booking not found
 * 
 * @example
 * ```typescript
 * await sendBookingConfirmation('user123', 'booking456', {
 *   language: 'en',
 *   sendCopy: true
 * });
 * ```
 * 
 * @see {@link bookingService.create} for booking creation
 * @see {@link docs/API_DOCUMENTATION.md} for full API docs
 */
export async function sendBookingConfirmation(
  userId: string,
  bookingId: string,
  options?: EmailOptions
): Promise<EmailStatus> {
  // Implementation
}
```

**Use JSDoc for:**
- All exported functions
- All service methods
- Complex utilities
- Public APIs

**Skip JSDoc for:**
- Private helper functions (inline comments instead)
- Obvious getters/setters
- Simple UI components (TypeScript types are enough)

---

## 🗂️ File Headers

### For Complex Files

```typescript
/**
 * User Authentication Service
 * 
 * Handles all user authentication operations including login,
 * signup, password reset, and session management.
 * 
 * @module services/authService
 * 
 * Security notes:
 * - Passwords are never logged
 * - All errors are sanitized before returning to client
 * - Rate limiting applied via Cloud Functions
 * 
 * Related:
 * - Security rules: firebase/firestore.rules (users collection)
 * - Email templates: functions/src/templates/auth/
 * - Documentation: docs/API_DOCUMENTATION.md#authentication
 * 
 * @author John Doe
 * @since 1.0.0
 */

import { getAuth, signInWithEmailAndPassword } from 'firebase/auth';
// ... rest of file
```

**Use file headers for:**
- Service files
- Repository files
- Complex utilities
- Security-critical code

**Skip for:**
- Simple UI components
- Type definition files
- Test files

---

## 📄 Additional Documentation (Optional)

### When Needed

**DEPLOYMENT.md** - For complex deployment
```markdown
# Deployment Guide

## Environments
- Development: http://localhost:5173
- Staging: https://staging.example.com
- Production: https://example.com

## Pre-Deployment Checklist
[...]

## Deployment Steps
[...]

## Rollback Procedure
[...]
```

**CONTRIBUTING.md** - For team/open source
```markdown
# Contributing Guide

## Code Style
[...]

## Git Workflow
[...]

## Pull Request Process
[...]
```

**SECURITY.md** - If handling sensitive data
```markdown
# Security Policy

## Reporting Vulnerabilities
[...]

## Security Practices
[...]
```

**TROUBLESHOOTING.md** - For common issues
```markdown
# Troubleshooting

## Firebase Connection Issues
**Symptom:** [...]
**Cause:** [...]
**Solution:** [...]
```

---

## 🚫 Documentation Anti-Patterns

### 1. The Novel

```markdown
❌ BAD - 50 pages explaining every line of code

✅ GOOD - High-level overview with links to code
```

### 2. The Fossil

```markdown
❌ BAD - Last updated 2 years ago, completely wrong now

✅ GOOD - Update as you change code, or delete if outdated
```

### 3. The Obvious

```markdown
❌ BAD:
## Button Component
This is a button. It can be clicked.

✅ GOOD:
## Button Component
Supports 5 variants for different contexts.
Automatically disabled during async operations.
```

### 4. The Cryptic

```markdown
❌ BAD:
"Uses pattern X for Y"

✅ GOOD:
"Uses repository pattern to isolate Firebase calls,
making the code testable without Firebase emulator"
```

### 5. The Scattered

```markdown
❌ BAD - Documentation spread across 20 random files

✅ GOOD - Centralized in docs/ with clear index
```

---

## ✅ Documentation Checklist

**Before considering project "done":**

```
Essential Documentation
☐ README.md exists and is accurate
☐ REQUIREMENTS.md documents all features
☐ DESIGN_SYSTEM.md has colors and styles
☐ CHANGELOG.md is up to date
☐ API_DOCUMENTATION.md covers all services (if applicable)

Code Documentation
☐ Complex functions have comments explaining why
☐ Business logic is commented
☐ Workarounds are documented with tickets
☐ TODOs have owner and date
☐ Public APIs have JSDoc

User Documentation (if needed)
☐ User guide for clients/end-users
☐ Admin panel documentation
☐ FAQ for common questions

Team Documentation (if team project)
☐ Deployment guide
☐ Contributing guidelines
☐ Troubleshooting guide

Quality Checks
☐ No commented-out code
☐ No obvious comments removed
☐ All links work
☐ Code examples are accurate
☐ No placeholder text ("TODO: Fill this in")

IF ANY ESSENTIAL ☐ UNCHECKED → Project not ready
```

---

## 🎯 AI Decision Gate

**The AI will check:**

```
┌─────────────────────────────────────────────┐
│  DOCUMENTATION CHECKLIST                    │
├─────────────────────────────────────────────┤
│  When creating features:                    │
│  ☐ Complex logic has "why" comments         │
│  ☐ TODOs have owner, date, context          │
│  ☐ Workarounds documented with links        │
│  ☐ Public methods have JSDoc                │
│  ☐ Business rules explained                 │
│                                             │
│  When feature complete:                     │
│  ☐ CHANGELOG.md updated                     │
│  ☐ API_DOCUMENTATION.md updated (if API)    │
│  ☐ No commented-out code                    │
│  ☐ README still accurate                    │
│                                             │
│  Red flags (AI will stop):                  │
│  ⚠️ Creating 10+ page docs for simple feature│
│  ⚠️ Documenting obvious code                │
│  ⚠️ Copying code into docs                  │
│  ⚠️ Not documenting complex logic           │
│                                             │
│  Balance: Document decisions, not details   │
└─────────────────────────────────────────────┘
```

---

**Remember: "Good documentation is like a good joke - if you have to explain it, it's not that good. Focus on the 'why', not the 'what'."** 📚✨

