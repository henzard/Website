---
description: "Unit testing, integration testing, and quality assurance standards for React+Firebase"
alwaysApply: true
globs: ["**/*.test.ts", "**/*.test.tsx", "**/*.spec.ts"]
---

# Testing Standards

## Testing Philosophy

### Testing Pyramid
```
    /\
   /E2E\          <- Few (5%)
  /------\
 /Integration\   <- Some (15%)
/----------\
/   Unit    \    <- Many (80%)
/------------\
```

### Core Principles
1. **Test behavior, not implementation**
2. **Write tests that give confidence**
3. **Fast feedback loop**
4. **Tests should be maintainable**

## Test Configuration

### Jest Configuration

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/main.tsx',
    '!src/**/*.stories.tsx',
  ],
  coverageThresholds: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
};
```

### Test Setup

```typescript
// tests/setup.ts
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach, vi } from 'vitest';

// Cleanup after each test
afterEach(() => {
  cleanup();
});

// Mock Firebase
vi.mock('firebase/app', () => ({
  initializeApp: vi.fn(),
}));

vi.mock('firebase/firestore', () => ({
  getFirestore: vi.fn(),
  collection: vi.fn(),
  doc: vi.fn(),
  getDoc: vi.fn(),
  getDocs: vi.fn(),
  setDoc: vi.fn(),
  updateDoc: vi.fn(),
  deleteDoc: vi.fn(),
  query: vi.fn(),
  where: vi.fn(),
}));

vi.mock('firebase/auth', () => ({
  getAuth: vi.fn(),
  signInWithEmailAndPassword: vi.fn(),
  signOut: vi.fn(),
  onAuthStateChanged: vi.fn(),
}));
```

## Unit Testing

### Testing Components

```typescript
// components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('renders with label', () => {
    render(<Button label="Click me" onClick={() => {}} />);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = vi.fn();
    render(<Button label="Click me" onClick={handleClick} />);
    
    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when disabled prop is true', () => {
    render(<Button label="Click me" onClick={() => {}} disabled />);
    expect(screen.getByRole('button')).toBeDisabled();
  });

  it('shows loading state', () => {
    render(<Button label="Click me" onClick={() => {}} loading />);
    expect(screen.getByRole('button')).toHaveAttribute('aria-busy', 'true');
    expect(screen.getByRole('button')).toBeDisabled();
  });

  it('applies correct variant styles', () => {
    const { rerender } = render(
      <Button label="Test" onClick={() => {}} variant="primary" />
    );
    expect(screen.getByRole('button')).toHaveClass('btn-primary');

    rerender(<Button label="Test" onClick={() => {}} variant="secondary" />);
    expect(screen.getByRole('button')).toHaveClass('btn-secondary');
  });
});
```

### Testing Custom Hooks

```typescript
// hooks/useUser.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { useUser } from './useUser';
import { userService } from '@/services/userService';

vi.mock('@/services/userService');

describe('useUser', () => {
  const mockUser = {
    id: '123',
    email: 'test@example.com',
    displayName: 'Test User',
  };

  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('fetches user data on mount', async () => {
    vi.mocked(userService.getById).mockResolvedValue(mockUser);

    const { result } = renderHook(() => useUser('123'));

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.user).toEqual(mockUser);
    expect(userService.getById).toHaveBeenCalledWith('123');
  });

  it('handles errors', async () => {
    const error = new Error('Failed to fetch');
    vi.mocked(userService.getById).mockRejectedValue(error);

    const { result } = renderHook(() => useUser('123'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.error).toEqual(error);
    expect(result.current.user).toBeNull();
  });

  it('refetches data when refetch is called', async () => {
    vi.mocked(userService.getById).mockResolvedValue(mockUser);

    const { result } = renderHook(() => useUser('123'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    vi.mocked(userService.getById).mockClear();
    
    await result.current.refetch();

    expect(userService.getById).toHaveBeenCalledTimes(1);
  });
});
```

### Testing Services

```typescript
// services/userService.test.ts
import { userService } from './userService';
import { userRepository } from '@/repositories/userRepository';

vi.mock('@/repositories/userRepository');

describe('UserService', () => {
  describe('createUser', () => {
    it('creates a user with valid data', async () => {
      const userData = {
        email: 'test@example.com',
        displayName: 'Test User',
        role: 'user' as const,
      };

      vi.mocked(userRepository.create).mockResolvedValue(undefined);
      vi.mocked(userRepository.findById).mockResolvedValue({
        id: '123',
        ...userData,
        createdAt: new Date(),
      });

      const user = await userService.createUser(userData);

      expect(user).toMatchObject(userData);
      expect(userRepository.create).toHaveBeenCalled();
    });

    it('throws error for invalid email', async () => {
      const userData = {
        email: 'invalid-email',
        displayName: 'Test User',
        role: 'user' as const,
      };

      await expect(userService.createUser(userData)).rejects.toThrow();
    });

    it('throws error for short display name', async () => {
      const userData = {
        email: 'test@example.com',
        displayName: 'T',
        role: 'user' as const,
      };

      await expect(userService.createUser(userData)).rejects.toThrow();
    });
  });

  describe('updateUser', () => {
    it('updates user data', async () => {
      const updateData = { displayName: 'Updated Name' };
      
      vi.mocked(userRepository.update).mockResolvedValue(undefined);

      await userService.updateUser('123', updateData);

      expect(userRepository.update).toHaveBeenCalledWith('123', updateData);
    });
  });
});
```

## Integration Testing

### Testing Firebase Integration

```typescript
// tests/firebase.test.ts
import { 
  initializeTestEnvironment, 
  RulesTestEnvironment 
} from '@firebase/rules-unit-testing';

let testEnv: RulesTestEnvironment;

beforeAll(async () => {
  testEnv = await initializeTestEnvironment({
    projectId: 'test-project',
    firestore: {
      rules: fs.readFileSync('firestore.rules', 'utf8'),
    },
  });
});

afterAll(async () => {
  await testEnv.cleanup();
});

describe('Firestore Security Rules', () => {
  it('allows user to read own profile', async () => {
    const alice = testEnv.authenticatedContext('alice');
    const aliceDoc = alice.firestore().doc('users/alice');
    
    await assertSucceeds(aliceDoc.get());
  });

  it('denies user from reading another user profile', async () => {
    const alice = testEnv.authenticatedContext('alice');
    const bobDoc = alice.firestore().doc('users/bob');
    
    await assertFails(bobDoc.get());
  });

  it('allows user to create own profile', async () => {
    const alice = testEnv.authenticatedContext('alice');
    const aliceDoc = alice.firestore().doc('users/alice');
    
    await assertSucceeds(aliceDoc.set({
      email: 'alice@example.com',
      createdAt: new Date(),
    }));
  });

  it('denies user from creating profile with invalid email', async () => {
    const alice = testEnv.authenticatedContext('alice');
    const aliceDoc = alice.firestore().doc('users/alice');
    
    await assertFails(aliceDoc.set({
      email: 'invalid-email',
      createdAt: new Date(),
    }));
  });
});
```

### Testing Forms

```typescript
// components/LoginForm.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';
import { authService } from '@/services/authService';

vi.mock('@/services/authService');

describe('LoginForm', () => {
  it('submits form with valid credentials', async () => {
    const user = userEvent.setup();
    const onSuccess = vi.fn();
    
    vi.mocked(authService.login).mockResolvedValue({ id: '123', email: 'test@example.com' });

    render(<LoginForm onSuccess={onSuccess} />);

    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /sign in/i }));

    await waitFor(() => {
      expect(authService.login).toHaveBeenCalledWith('test@example.com', 'password123');
    });

    expect(onSuccess).toHaveBeenCalled();
  });

  it('shows validation errors for empty fields', async () => {
    const user = userEvent.setup();
    
    render(<LoginForm onSuccess={() => {}} />);

    await user.click(screen.getByRole('button', { name: /sign in/i }));

    expect(await screen.findByText(/email is required/i)).toBeInTheDocument();
    expect(await screen.findByText(/password is required/i)).toBeInTheDocument();
  });

  it('shows error message on login failure', async () => {
    const user = userEvent.setup();
    
    vi.mocked(authService.login).mockRejectedValue(
      new Error('Invalid credentials')
    );

    render(<LoginForm onSuccess={() => {}} />);

    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'wrong-password');
    await user.click(screen.getByRole('button', { name: /sign in/i }));

    expect(await screen.findByText(/invalid credentials/i)).toBeInTheDocument();
  });
});
```

## E2E Testing (Playwright)

```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication Flow', () => {
  test('user can sign up', async ({ page }) => {
    await page.goto('/signup');

    await page.fill('input[name="email"]', 'newuser@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.fill('input[name="confirmPassword"]', 'password123');
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('text=Welcome')).toBeVisible();
  });

  test('user can log in', async ({ page }) => {
    await page.goto('/login');

    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL('/dashboard');
  });

  test('user can log out', async ({ page }) => {
    // Assuming user is logged in
    await page.goto('/dashboard');
    await page.click('button:has-text("Logout")');

    await expect(page).toHaveURL('/login');
  });
});
```

## Test Coverage Requirements

### Minimum Coverage
- **Unit Tests**: 80% coverage
- **Integration Tests**: Critical paths covered
- **E2E Tests**: Major user flows covered

### What to Test
✅ User interactions
✅ Form validation
✅ Error handling
✅ Edge cases
✅ Security rules
✅ Business logic

### What NOT to Test
❌ Third-party libraries
❌ Implementation details
❌ Trivial code (getters/setters)

## Test Documentation

```typescript
/**
 * Test naming convention:
 * describe('ComponentName or functionName', () => {
 *   describe('specific scenario', () => {
 *     it('should do something specific', () => {
 *       // test implementation
 *     });
 *   });
 * });
 */
```

## AI Decision Gate

**Before marking a feature complete:**
1. ✅ Are there unit tests for all business logic?
2. ✅ Are components tested with different props/states?
3. ✅ Are error cases tested?
4. ✅ Is test coverage above 70%?
5. ✅ Do tests pass consistently?
6. ✅ Are Firebase rules tested?
7. ✅ Are critical user flows covered by E2E tests?

**No feature is complete without tests. Tests are not optional.**

