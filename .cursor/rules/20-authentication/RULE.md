---
description: "Authentication, user management, roles, and access control for Firebase Auth"
alwaysApply: false
globs: ["**/auth/**", "**/components/Auth*", "**/hooks/useAuth*", "**/contexts/Auth*"]
---

# Authentication & User Management Standards

## 🎯 Core Principles

**"Security first. User experience second. Never store passwords."**

- ✅ Use Firebase Authentication (never roll your own)
- ✅ Implement proper session management
- ✅ Role-based access control (RBAC)
- ✅ Secure password reset flows
- ✅ Social login options
- ✅ Clear error messages (without leaking info)

---

## 🔐 Firebase Auth Setup

### Initialize Firebase Auth

```typescript
// lib/firebase.ts
import { initializeApp } from 'firebase/app';
import { 
  getAuth, 
  connectAuthEmulator,
  browserLocalPersistence,
  setPersistence
} from 'firebase/auth';

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);

// Set persistence (survives browser close)
setPersistence(auth, browserLocalPersistence);

// Use emulator in development
if (process.env.NODE_ENV === 'development') {
  connectAuthEmulator(auth, 'http://localhost:9099');
}
```

---

## 👤 Auth Context Provider

```typescript
// contexts/AuthContext.tsx
import { createContext, useContext, useEffect, useState } from 'react';
import { 
  User, 
  onAuthStateChanged,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  signOut,
  sendPasswordResetEmail,
  sendEmailVerification,
  GoogleAuthProvider,
  signInWithPopup,
  updateProfile
} from 'firebase/auth';
import { auth } from '@/lib/firebase';

interface AuthContextType {
  user: User | null;
  userProfile: UserProfile | null;
  loading: boolean;
  error: string | null;
  
  // Auth methods
  login: (email: string, password: string) => Promise<void>;
  register: (email: string, password: string, displayName: string) => Promise<void>;
  logout: () => Promise<void>;
  resetPassword: (email: string) => Promise<void>;
  loginWithGoogle: () => Promise<void>;
  verifyEmail: () => Promise<void>;
  
  // Role helpers
  isAdmin: boolean;
  isStaff: boolean;
  hasRole: (role: string) => boolean;
}

interface UserProfile {
  uid: string;
  email: string;
  displayName: string;
  photoURL?: string;
  roles: string[];
  createdAt: Date;
  lastLoginAt: Date;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [userProfile, setUserProfile] = useState<UserProfile | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  // Listen to auth state changes
  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, async (firebaseUser) => {
      setUser(firebaseUser);
      
      if (firebaseUser) {
        // Fetch user profile from Firestore
        const profile = await fetchUserProfile(firebaseUser.uid);
        setUserProfile(profile);
        
        // Update last login
        await updateLastLogin(firebaseUser.uid);
      } else {
        setUserProfile(null);
      }
      
      setLoading(false);
    });

    return () => unsubscribe();
  }, []);

  // Login with email/password
  const login = async (email: string, password: string) => {
    try {
      setError(null);
      await signInWithEmailAndPassword(auth, email, password);
    } catch (err) {
      setError(getAuthErrorMessage(err));
      throw err;
    }
  };

  // Register new user
  const register = async (email: string, password: string, displayName: string) => {
    try {
      setError(null);
      const { user: newUser } = await createUserWithEmailAndPassword(auth, email, password);
      
      // Update display name
      await updateProfile(newUser, { displayName });
      
      // Create user profile in Firestore
      await createUserProfile(newUser.uid, {
        email,
        displayName,
        roles: ['user'], // Default role
      });
      
      // Send verification email
      await sendEmailVerification(newUser);
    } catch (err) {
      setError(getAuthErrorMessage(err));
      throw err;
    }
  };

  // Logout
  const logout = async () => {
    try {
      await signOut(auth);
    } catch (err) {
      setError(getAuthErrorMessage(err));
      throw err;
    }
  };

  // Password reset
  const resetPassword = async (email: string) => {
    try {
      setError(null);
      await sendPasswordResetEmail(auth, email);
    } catch (err) {
      setError(getAuthErrorMessage(err));
      throw err;
    }
  };

  // Google sign-in
  const loginWithGoogle = async () => {
    try {
      setError(null);
      const provider = new GoogleAuthProvider();
      const { user: googleUser } = await signInWithPopup(auth, provider);
      
      // Create/update user profile
      const existingProfile = await fetchUserProfile(googleUser.uid);
      if (!existingProfile) {
        await createUserProfile(googleUser.uid, {
          email: googleUser.email!,
          displayName: googleUser.displayName || 'User',
          photoURL: googleUser.photoURL || undefined,
          roles: ['user'],
        });
      }
    } catch (err) {
      setError(getAuthErrorMessage(err));
      throw err;
    }
  };

  // Email verification
  const verifyEmail = async () => {
    if (user) {
      await sendEmailVerification(user);
    }
  };

  // Role checks
  const hasRole = (role: string) => userProfile?.roles?.includes(role) ?? false;
  const isAdmin = hasRole('admin');
  const isStaff = hasRole('staff') || isAdmin;

  return (
    <AuthContext.Provider value={{
      user,
      userProfile,
      loading,
      error,
      login,
      register,
      logout,
      resetPassword,
      loginWithGoogle,
      verifyEmail,
      isAdmin,
      isStaff,
      hasRole,
    }}>
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

---

## 🚪 Auth Components

### Login Form

```typescript
// components/auth/LoginForm.tsx
import { useState } from 'react';
import { useAuth } from '@/contexts/AuthContext';
import { useNavigate, Link } from 'react-router-dom';

export const LoginForm: React.FC = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const { login, loginWithGoogle, error } = useAuth();
  const navigate = useNavigate();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsLoading(true);
    
    try {
      await login(email, password);
      navigate('/dashboard');
    } catch {
      // Error is handled by AuthContext
    } finally {
      setIsLoading(false);
    }
  };

  const handleGoogleLogin = async () => {
    try {
      await loginWithGoogle();
      navigate('/dashboard');
    } catch {
      // Error is handled by AuthContext
    }
  };

  return (
    <div className="max-w-md mx-auto p-6 bg-white rounded-lg shadow-md">
      <h2 className="text-2xl font-bold mb-6 text-center">Sign In</h2>
      
      {error && (
        <div className="mb-4 p-3 bg-red-100 text-red-700 rounded-lg">
          {error}
        </div>
      )}

      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label htmlFor="email" className="block text-sm font-medium mb-1">
            Email
          </label>
          <input
            id="email"
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
            autoComplete="email"
            className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-primary-500"
          />
        </div>

        <div>
          <label htmlFor="password" className="block text-sm font-medium mb-1">
            Password
          </label>
          <input
            id="password"
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            required
            autoComplete="current-password"
            className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-primary-500"
          />
        </div>

        <div className="flex justify-end">
          <Link to="/forgot-password" className="text-sm text-primary-600 hover:underline">
            Forgot password?
          </Link>
        </div>

        <button
          type="submit"
          disabled={isLoading}
          className="w-full py-3 bg-primary-600 text-white rounded-lg hover:bg-primary-700 disabled:opacity-50"
        >
          {isLoading ? 'Signing in...' : 'Sign In'}
        </button>
      </form>

      <div className="mt-6">
        <div className="relative">
          <div className="absolute inset-0 flex items-center">
            <div className="w-full border-t"></div>
          </div>
          <div className="relative flex justify-center text-sm">
            <span className="px-2 bg-white text-gray-500">Or continue with</span>
          </div>
        </div>

        <button
          onClick={handleGoogleLogin}
          className="mt-4 w-full py-3 border border-gray-300 rounded-lg hover:bg-gray-50 flex items-center justify-center gap-2"
        >
          <img src="/google-icon.svg" alt="" className="w-5 h-5" />
          Google
        </button>
      </div>

      <p className="mt-6 text-center text-sm text-gray-600">
        Don't have an account?{' '}
        <Link to="/register" className="text-primary-600 hover:underline font-medium">
          Sign up
        </Link>
      </p>
    </div>
  );
};
```

### Registration Form

```typescript
// components/auth/RegisterForm.tsx
import { useState } from 'react';
import { useAuth } from '@/contexts/AuthContext';
import { useNavigate, Link } from 'react-router-dom';

export const RegisterForm: React.FC = () => {
  const [formData, setFormData] = useState({
    displayName: '',
    email: '',
    password: '',
    confirmPassword: '',
  });
  const [isLoading, setIsLoading] = useState(false);
  const [validationError, setValidationError] = useState('');
  const { register, error } = useAuth();
  const navigate = useNavigate();

  const validateForm = (): boolean => {
    if (formData.password.length < 8) {
      setValidationError('Password must be at least 8 characters');
      return false;
    }
    if (formData.password !== formData.confirmPassword) {
      setValidationError('Passwords do not match');
      return false;
    }
    if (formData.displayName.trim().length < 2) {
      setValidationError('Name must be at least 2 characters');
      return false;
    }
    setValidationError('');
    return true;
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!validateForm()) return;
    
    setIsLoading(true);
    
    try {
      await register(formData.email, formData.password, formData.displayName);
      navigate('/verify-email');
    } catch {
      // Error handled by AuthContext
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="max-w-md mx-auto p-6 bg-white rounded-lg shadow-md">
      <h2 className="text-2xl font-bold mb-6 text-center">Create Account</h2>

      {(error || validationError) && (
        <div className="mb-4 p-3 bg-red-100 text-red-700 rounded-lg">
          {validationError || error}
        </div>
      )}

      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label htmlFor="displayName" className="block text-sm font-medium mb-1">
            Full Name
          </label>
          <input
            id="displayName"
            type="text"
            value={formData.displayName}
            onChange={(e) => setFormData({ ...formData, displayName: e.target.value })}
            required
            autoComplete="name"
            className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-primary-500"
          />
        </div>

        <div>
          <label htmlFor="email" className="block text-sm font-medium mb-1">
            Email
          </label>
          <input
            id="email"
            type="email"
            value={formData.email}
            onChange={(e) => setFormData({ ...formData, email: e.target.value })}
            required
            autoComplete="email"
            className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-primary-500"
          />
        </div>

        <div>
          <label htmlFor="password" className="block text-sm font-medium mb-1">
            Password
          </label>
          <input
            id="password"
            type="password"
            value={formData.password}
            onChange={(e) => setFormData({ ...formData, password: e.target.value })}
            required
            autoComplete="new-password"
            minLength={8}
            className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-primary-500"
          />
          <p className="mt-1 text-xs text-gray-500">
            At least 8 characters
          </p>
        </div>

        <div>
          <label htmlFor="confirmPassword" className="block text-sm font-medium mb-1">
            Confirm Password
          </label>
          <input
            id="confirmPassword"
            type="password"
            value={formData.confirmPassword}
            onChange={(e) => setFormData({ ...formData, confirmPassword: e.target.value })}
            required
            autoComplete="new-password"
            className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-primary-500"
          />
        </div>

        <button
          type="submit"
          disabled={isLoading}
          className="w-full py-3 bg-primary-600 text-white rounded-lg hover:bg-primary-700 disabled:opacity-50"
        >
          {isLoading ? 'Creating account...' : 'Create Account'}
        </button>
      </form>

      <p className="mt-6 text-center text-sm text-gray-600">
        Already have an account?{' '}
        <Link to="/login" className="text-primary-600 hover:underline font-medium">
          Sign in
        </Link>
      </p>
    </div>
  );
};
```

### Password Reset

```typescript
// components/auth/ForgotPasswordForm.tsx
import { useState } from 'react';
import { useAuth } from '@/contexts/AuthContext';
import { Link } from 'react-router-dom';

export const ForgotPasswordForm: React.FC = () => {
  const [email, setEmail] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [isSuccess, setIsSuccess] = useState(false);
  const { resetPassword, error } = useAuth();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsLoading(true);
    
    try {
      await resetPassword(email);
      setIsSuccess(true);
    } catch {
      // Error handled by AuthContext
    } finally {
      setIsLoading(false);
    }
  };

  if (isSuccess) {
    return (
      <div className="max-w-md mx-auto p-6 bg-white rounded-lg shadow-md text-center">
        <div className="text-5xl mb-4">📧</div>
        <h2 className="text-2xl font-bold mb-4">Check Your Email</h2>
        <p className="text-gray-600 mb-6">
          We've sent password reset instructions to <strong>{email}</strong>
        </p>
        <Link to="/login" className="text-primary-600 hover:underline">
          Back to login
        </Link>
      </div>
    );
  }

  return (
    <div className="max-w-md mx-auto p-6 bg-white rounded-lg shadow-md">
      <h2 className="text-2xl font-bold mb-2 text-center">Reset Password</h2>
      <p className="text-gray-600 text-center mb-6">
        Enter your email and we'll send you instructions
      </p>

      {error && (
        <div className="mb-4 p-3 bg-red-100 text-red-700 rounded-lg">
          {error}
        </div>
      )}

      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label htmlFor="email" className="block text-sm font-medium mb-1">
            Email
          </label>
          <input
            id="email"
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
            autoComplete="email"
            className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-primary-500"
          />
        </div>

        <button
          type="submit"
          disabled={isLoading}
          className="w-full py-3 bg-primary-600 text-white rounded-lg hover:bg-primary-700 disabled:opacity-50"
        >
          {isLoading ? 'Sending...' : 'Send Reset Link'}
        </button>
      </form>

      <p className="mt-6 text-center text-sm">
        <Link to="/login" className="text-primary-600 hover:underline">
          Back to login
        </Link>
      </p>
    </div>
  );
};
```

---

## 🛡️ Protected Routes

```typescript
// components/auth/ProtectedRoute.tsx
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '@/contexts/AuthContext';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredRole?: string;
  requireEmailVerified?: boolean;
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  children,
  requiredRole,
  requireEmailVerified = false,
}) => {
  const { user, userProfile, loading, hasRole } = useAuth();
  const location = useLocation();

  // Show loading state
  if (loading) {
    return (
      <div className="flex items-center justify-center min-h-screen">
        <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-primary-600"></div>
      </div>
    );
  }

  // Not logged in → redirect to login
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  // Email not verified → redirect to verification page
  if (requireEmailVerified && !user.emailVerified) {
    return <Navigate to="/verify-email" replace />;
  }

  // Role required but user doesn't have it → show forbidden
  if (requiredRole && !hasRole(requiredRole)) {
    return <Navigate to="/forbidden" replace />;
  }

  return <>{children}</>;
};

// Usage in router:
// <Route path="/admin" element={
//   <ProtectedRoute requiredRole="admin">
//     <AdminDashboard />
//   </ProtectedRoute>
// } />
```

### Admin Only Route

```typescript
// components/auth/AdminRoute.tsx
import { ProtectedRoute } from './ProtectedRoute';

export const AdminRoute: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <ProtectedRoute requiredRole="admin" requireEmailVerified>
    {children}
  </ProtectedRoute>
);
```

---

## 👥 Role-Based Access Control (RBAC)

### Firestore User Profile Structure

```typescript
// types/user.ts
export interface UserProfile {
  uid: string;
  email: string;
  displayName: string;
  photoURL?: string;
  roles: UserRole[];
  permissions: string[];
  createdAt: Timestamp;
  updatedAt: Timestamp;
  lastLoginAt: Timestamp;
  emailVerified: boolean;
  isActive: boolean;
}

export type UserRole = 'user' | 'staff' | 'admin' | 'superadmin';

export const ROLE_HIERARCHY: Record<UserRole, number> = {
  user: 1,
  staff: 2,
  admin: 3,
  superadmin: 4,
};

export const ROLE_PERMISSIONS: Record<UserRole, string[]> = {
  user: ['read:own', 'write:own'],
  staff: ['read:own', 'write:own', 'read:all'],
  admin: ['read:own', 'write:own', 'read:all', 'write:all', 'manage:users'],
  superadmin: ['*'], // All permissions
};
```

### Firestore Security Rules

```javascript
// firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }
    
    function isOwner(userId) {
      return request.auth.uid == userId;
    }
    
    function getUserRoles() {
      return get(/databases/$(database)/documents/users/$(request.auth.uid)).data.roles;
    }
    
    function hasRole(role) {
      return isAuthenticated() && role in getUserRoles();
    }
    
    function isAdmin() {
      return hasRole('admin') || hasRole('superadmin');
    }
    
    function isStaff() {
      return hasRole('staff') || isAdmin();
    }

    // User profiles
    match /users/{userId} {
      // Users can read their own profile
      allow read: if isOwner(userId) || isAdmin();
      
      // Users can update their own profile (but not roles)
      allow update: if isOwner(userId) && 
        !request.resource.data.diff(resource.data).affectedKeys().hasAny(['roles', 'permissions']);
      
      // Only admins can create users or modify roles
      allow create: if isAdmin();
      allow update: if isAdmin();
      allow delete: if hasRole('superadmin');
    }
    
    // Protected resources
    match /orders/{orderId} {
      allow read: if isOwner(resource.data.userId) || isStaff();
      allow create: if isAuthenticated();
      allow update: if isStaff();
      allow delete: if isAdmin();
    }
  }
}
```

### Permission Hook

```typescript
// hooks/usePermission.ts
import { useAuth } from '@/contexts/AuthContext';

export const usePermission = () => {
  const { userProfile, hasRole } = useAuth();

  const can = (action: string, resource?: string): boolean => {
    if (!userProfile) return false;
    
    // Superadmin can do anything
    if (hasRole('superadmin')) return true;
    
    const permission = resource ? `${action}:${resource}` : action;
    
    // Check explicit permissions
    if (userProfile.permissions?.includes(permission)) return true;
    if (userProfile.permissions?.includes(`${action}:all`)) return true;
    
    // Check role-based permissions
    for (const role of userProfile.roles) {
      const rolePermissions = ROLE_PERMISSIONS[role as UserRole] || [];
      if (rolePermissions.includes(permission) || rolePermissions.includes('*')) {
        return true;
      }
    }
    
    return false;
  };

  return { can };
};

// Usage:
// const { can } = usePermission();
// if (can('write', 'posts')) { ... }
// if (can('manage:users')) { ... }
```

---

## 🔑 Social Authentication

### Multiple Providers

```typescript
// lib/authProviders.ts
import { 
  GoogleAuthProvider, 
  FacebookAuthProvider,
  GithubAuthProvider,
  OAuthProvider,
  signInWithPopup,
  linkWithPopup,
  AuthProvider
} from 'firebase/auth';
import { auth } from './firebase';

// Provider configurations
const googleProvider = new GoogleAuthProvider();
googleProvider.addScope('profile');
googleProvider.addScope('email');

const facebookProvider = new FacebookAuthProvider();
facebookProvider.addScope('email');

const githubProvider = new GithubAuthProvider();
githubProvider.addScope('user:email');

const microsoftProvider = new OAuthProvider('microsoft.com');
microsoftProvider.addScope('user.read');

export const signInWithProvider = async (providerName: string) => {
  let provider: AuthProvider;
  
  switch (providerName) {
    case 'google':
      provider = googleProvider;
      break;
    case 'facebook':
      provider = facebookProvider;
      break;
    case 'github':
      provider = githubProvider;
      break;
    case 'microsoft':
      provider = microsoftProvider;
      break;
    default:
      throw new Error(`Unknown provider: ${providerName}`);
  }
  
  return signInWithPopup(auth, provider);
};

// Link additional provider to existing account
export const linkProvider = async (providerName: string) => {
  if (!auth.currentUser) throw new Error('No user logged in');
  
  const provider = getProvider(providerName);
  return linkWithPopup(auth.currentUser, provider);
};
```

### Social Login Buttons

```typescript
// components/auth/SocialLoginButtons.tsx
interface SocialLoginButtonsProps {
  onSuccess?: () => void;
  onError?: (error: Error) => void;
}

export const SocialLoginButtons: React.FC<SocialLoginButtonsProps> = ({
  onSuccess,
  onError,
}) => {
  const [isLoading, setIsLoading] = useState<string | null>(null);

  const handleSocialLogin = async (provider: string) => {
    setIsLoading(provider);
    try {
      await signInWithProvider(provider);
      onSuccess?.();
    } catch (error) {
      onError?.(error as Error);
    } finally {
      setIsLoading(null);
    }
  };

  return (
    <div className="space-y-3">
      <button
        onClick={() => handleSocialLogin('google')}
        disabled={isLoading !== null}
        className="w-full py-3 border border-gray-300 rounded-lg hover:bg-gray-50 flex items-center justify-center gap-2"
      >
        {isLoading === 'google' ? (
          <Spinner size="sm" />
        ) : (
          <>
            <GoogleIcon className="w-5 h-5" />
            Continue with Google
          </>
        )}
      </button>

      <button
        onClick={() => handleSocialLogin('facebook')}
        disabled={isLoading !== null}
        className="w-full py-3 bg-[#1877F2] text-white rounded-lg hover:bg-[#166FE5] flex items-center justify-center gap-2"
      >
        {isLoading === 'facebook' ? (
          <Spinner size="sm" />
        ) : (
          <>
            <FacebookIcon className="w-5 h-5" />
            Continue with Facebook
          </>
        )}
      </button>
    </div>
  );
};
```

---

## 🔒 Session Management

### Session Timeout

```typescript
// hooks/useSessionTimeout.ts
import { useEffect, useCallback } from 'react';
import { useAuth } from '@/contexts/AuthContext';

const SESSION_TIMEOUT = 30 * 60 * 1000; // 30 minutes
const WARNING_TIME = 5 * 60 * 1000; // 5 minutes before

export const useSessionTimeout = () => {
  const { logout } = useAuth();
  const [showWarning, setShowWarning] = useState(false);
  let timeoutId: NodeJS.Timeout;
  let warningId: NodeJS.Timeout;

  const resetTimer = useCallback(() => {
    clearTimeout(timeoutId);
    clearTimeout(warningId);
    setShowWarning(false);
    
    // Set warning timer
    warningId = setTimeout(() => {
      setShowWarning(true);
    }, SESSION_TIMEOUT - WARNING_TIME);
    
    // Set logout timer
    timeoutId = setTimeout(() => {
      logout();
    }, SESSION_TIMEOUT);
  }, [logout]);

  useEffect(() => {
    // Activity events
    const events = ['mousedown', 'keydown', 'scroll', 'touchstart'];
    
    events.forEach(event => {
      window.addEventListener(event, resetTimer);
    });
    
    resetTimer();
    
    return () => {
      events.forEach(event => {
        window.removeEventListener(event, resetTimer);
      });
      clearTimeout(timeoutId);
      clearTimeout(warningId);
    };
  }, [resetTimer]);

  const extendSession = () => {
    resetTimer();
  };

  return { showWarning, extendSession };
};
```

---

## ⚠️ Error Messages

```typescript
// utils/authErrors.ts

export const getAuthErrorMessage = (error: unknown): string => {
  const firebaseError = error as { code?: string };
  
  switch (firebaseError.code) {
    case 'auth/invalid-email':
      return 'Please enter a valid email address';
    case 'auth/user-disabled':
      return 'This account has been disabled. Please contact support.';
    case 'auth/user-not-found':
    case 'auth/wrong-password':
      // Don't reveal which one is wrong (security)
      return 'Invalid email or password';
    case 'auth/email-already-in-use':
      return 'An account with this email already exists';
    case 'auth/weak-password':
      return 'Password is too weak. Use at least 8 characters';
    case 'auth/too-many-requests':
      return 'Too many failed attempts. Please try again later.';
    case 'auth/popup-closed-by-user':
      return 'Sign-in was cancelled';
    case 'auth/account-exists-with-different-credential':
      return 'An account already exists with this email. Try a different sign-in method.';
    case 'auth/network-request-failed':
      return 'Network error. Please check your connection.';
    default:
      console.error('Auth error:', error);
      return 'An error occurred. Please try again.';
  }
};
```

---

## ✅ Checklist

```
Setup
☐ Firebase Auth initialized
☐ Auth emulator configured for development
☐ Persistence set (browserLocalPersistence)

Auth Context
☐ AuthProvider wraps app
☐ useAuth hook created
☐ User state managed
☐ Error state managed
☐ Loading state managed

Authentication Flows
☐ Email/password login
☐ User registration
☐ Password reset flow
☐ Email verification
☐ Social login (Google at minimum)
☐ Logout functionality

Protected Routes
☐ ProtectedRoute component
☐ Role-based route protection
☐ Email verification requirement
☐ Redirect to login with return URL

User Profiles
☐ Firestore user profile structure
☐ Create profile on registration
☐ Update last login timestamp
☐ Sync with Firebase Auth user

Role-Based Access
☐ Roles stored in Firestore
☐ hasRole() helper function
☐ Firestore security rules use roles
☐ UI shows/hides based on roles

Security
☐ Error messages don't leak info
☐ Rate limiting on login attempts
☐ Session timeout implemented
☐ Secure password requirements

IF ANY ☐ UNCHECKED → Address before launch
```

---

## 🚨 AI Decision Gate

```
┌─────────────────────────────────────────────┐
│  AUTHENTICATION CHECKLIST                   │
├─────────────────────────────────────────────┤
│  Before implementing auth:                  │
│  ☐ Using Firebase Auth (not custom)        │
│  ☐ Firestore rules use auth                │
│  ☐ Error messages are user-friendly        │
│  ☐ Error messages don't leak info          │
│  ☐ Password requirements enforced          │
│                                             │
│  For protected routes:                      │
│  ☐ Loading state shown while checking      │
│  ☐ Redirect preserves intended URL         │
│  ☐ Role checks use Firestore roles         │
│  ☐ Admin routes require admin role         │
│                                             │
│  For social login:                          │
│  ☐ Handle account-exists-with-different    │
│  ☐ Create user profile if new user         │
│  ☐ Handle popup-closed-by-user             │
│                                             │
│  NEVER:                                     │
│  ✗ Store passwords yourself                │
│  ✗ Reveal if email exists in system        │
│  ✗ Skip email verification for sensitive   │
│  ✗ Trust client-side role checks alone     │
│                                             │
│  IF ANY ☐ UNCHECKED:                        │
│  ➜ ADDRESS BEFORE IMPLEMENTING              │
└─────────────────────────────────────────────┘
```

---

**Remember: "Firebase Auth handles security. You handle user experience."** 🔐✨

