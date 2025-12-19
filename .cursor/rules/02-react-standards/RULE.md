---
description: "React best practices, TypeScript standards, and component design patterns"
alwaysApply: true
---

# React Best Practices & Standards

## TypeScript Configuration

Always use TypeScript with strict mode:

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUncheckedIndexedAccess": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "jsx": "react-jsx"
  }
}
```

## Component Standards

### 1. Functional Components with TypeScript

```typescript
// ✅ GOOD - Clear props interface, named export
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  loading?: boolean;
}

export const Button: React.FC<ButtonProps> = ({ 
  label, 
  onClick, 
  variant = 'primary',
  disabled = false,
  loading = false 
}) => {
  return (
    <button 
      onClick={onClick}
      disabled={disabled || loading}
      className={`btn btn-${variant}`}
      aria-busy={loading}
    >
      {loading ? <Spinner /> : label}
    </button>
  );
};

// ❌ BAD - No types, unclear props
export default function Button(props) {
  return <button onClick={props.onClick}>{props.label}</button>;
}
```

### 2. Component File Structure

```typescript
// ComponentName.tsx

// 1. Imports (group by: React, third-party, internal)
import React, { useState, useEffect } from 'react';
import { useQuery } from '@tanstack/react-query';
import { Button } from '@/components/Button';
import { userService } from '@/services/userService';
import type { User } from '@/types/user';

// 2. Types/Interfaces
interface ComponentNameProps {
  userId: string;
  onUpdate?: (user: User) => void;
}

// 3. Component
export const ComponentName: React.FC<ComponentNameProps> = ({ 
  userId, 
  onUpdate 
}) => {
  // Hooks first
  const [isEditing, setIsEditing] = useState(false);
  const { data: user, isLoading } = useQuery(['user', userId], 
    () => userService.getById(userId)
  );

  // Effects
  useEffect(() => {
    // Side effects
  }, []);

  // Event handlers
  const handleEdit = () => {
    setIsEditing(true);
  };

  // Early returns
  if (isLoading) return <LoadingSpinner />;
  if (!user) return <ErrorMessage />;

  // Main render
  return (
    <div>
      {/* Component JSX */}
    </div>
  );
};

// 4. Helper components (if needed)
const SubComponent: React.FC = () => {
  return <div>Helper</div>;
};
```

## Custom Hooks Standards

### Hook Naming and Structure

```typescript
// hooks/useUser.ts

interface UseUserOptions {
  enabled?: boolean;
  onSuccess?: (user: User) => void;
}

interface UseUserReturn {
  user: User | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
  updateUser: (data: Partial<User>) => Promise<void>;
}

export const useUser = (
  userId: string, 
  options: UseUserOptions = {}
): UseUserReturn => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    if (!options.enabled) return;
    
    fetchUser();
  }, [userId, options.enabled]);

  const fetchUser = async () => {
    try {
      setLoading(true);
      const userData = await userService.getById(userId);
      setUser(userData);
      options.onSuccess?.(userData);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  };

  const updateUser = async (data: Partial<User>) => {
    await userService.update(userId, data);
    await fetchUser();
  };

  return { user, loading, error, refetch: fetchUser, updateUser };
};
```

## State Management

### Context API Pattern

```typescript
// contexts/AuthContext.tsx

interface AuthContextValue {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
  loading: boolean;
}

const AuthContext = createContext<AuthContextValue | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ 
  children 
}) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  const value = useMemo(() => ({
    user,
    login,
    logout,
    loading
  }), [user, loading]);

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};
```

## Performance Optimization

### Memoization Rules

```typescript
// ✅ GOOD - Memoize expensive computations
const sortedUsers = useMemo(() => {
  return users.sort((a, b) => a.name.localeCompare(b.name));
}, [users]);

// ✅ GOOD - Memoize callbacks passed to children
const handleUserClick = useCallback((userId: string) => {
  navigate(`/users/${userId}`);
}, [navigate]);

// ❌ BAD - Unnecessary memoization
const fullName = useMemo(() => `${firstName} ${lastName}`, [firstName, lastName]);
```

### Code Splitting

```typescript
// Lazy load routes and heavy components
const Dashboard = lazy(() => import('./features/dashboard/Dashboard'));
const UserProfile = lazy(() => import('./features/profile/UserProfile'));

function App() {
  return (
    <Suspense fallback={<LoadingScreen />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<UserProfile />} />
      </Routes>
    </Suspense>
  );
}
```

## Accessibility Standards

Every component must be accessible:

```typescript
// ✅ GOOD - Accessible form
<form onSubmit={handleSubmit} aria-label="Login form">
  <label htmlFor="email">Email</label>
  <input 
    id="email"
    type="email"
    aria-required="true"
    aria-invalid={!!errors.email}
    aria-describedby={errors.email ? "email-error" : undefined}
  />
  {errors.email && (
    <span id="email-error" role="alert">{errors.email}</span>
  )}
  
  <button type="submit" aria-busy={loading}>
    {loading ? 'Signing in...' : 'Sign In'}
  </button>
</form>
```

## Naming Conventions

```typescript
// Components: PascalCase
export const UserProfile = () => {};

// Hooks: camelCase with 'use' prefix
export const useUserData = () => {};

// Constants: UPPER_SNAKE_CASE
export const MAX_FILE_SIZE = 5 * 1024 * 1024;

// Functions: camelCase
export const formatDate = (date: Date) => {};

// Types/Interfaces: PascalCase
export interface UserProfile {}
export type Status = 'active' | 'inactive';

// Boolean variables: is/has/should prefix
const isLoading = true;
const hasPermission = false;
const shouldRender = true;
```

## AI Decision Gate

**Before creating a component, verify:**
1. ✅ Is there a clear TypeScript interface for props?
2. ✅ Does the component have a single responsibility?
3. ✅ Are effects properly cleaned up?
4. ✅ Is the component accessible (ARIA labels, keyboard nav)?
5. ✅ Is expensive logic memoized?
6. ✅ Are loading and error states handled?

**If any check fails, refactor before proceeding.**

