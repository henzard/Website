---
description: "Error handling, logging, monitoring, and graceful failure strategies for production applications"
alwaysApply: true
---

# Error Handling & Logging Standards

## 🎯 Core Principles

**"Fail gracefully, log everything, recover when possible."**

Good error handling:
- ✅ Prevents app crashes
- ✅ Provides clear feedback to users
- ✅ Logs enough info to debug issues
- ✅ Protects sensitive information
- ✅ Enables quick problem resolution

Poor error handling:
- ❌ Shows technical errors to users
- ❌ Crashes the entire app
- ❌ Gives no clue what went wrong
- ❌ Exposes security vulnerabilities
- ❌ Makes debugging impossible

---

## 🚨 Error Boundaries (React)

### Global Error Boundary

```typescript
// components/ErrorBoundary.tsx

interface Props {
  children: React.ReactNode;
  fallback?: React.ComponentType<{ error: Error; resetError: () => void }>;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log to error tracking service
    logError('React Error Boundary', {
      error: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      timestamp: new Date().toISOString(),
    });

    // Send to monitoring service (e.g., Sentry)
    if (window.Sentry) {
      window.Sentry.captureException(error, {
        contexts: {
          react: {
            componentStack: errorInfo.componentStack,
          },
        },
      });
    }
  }

  resetError = () => {
    this.setState({ hasError: false, error: null });
  };

  render() {
    if (this.state.hasError) {
      const FallbackComponent = this.props.fallback || DefaultErrorFallback;
      return (
        <FallbackComponent 
          error={this.state.error!} 
          resetError={this.resetError}
        />
      );
    }

    return this.props.children;
  }
}

// Default error fallback UI
const DefaultErrorFallback: React.FC<{
  error: Error;
  resetError: () => void;
}> = ({ error, resetError }) => {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50 px-4">
      <div className="max-w-md w-full bg-white rounded-lg shadow-lg p-6">
        <div className="flex items-center justify-center w-12 h-12 mx-auto bg-red-100 rounded-full">
          <svg className="w-6 h-6 text-red-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
          </svg>
        </div>
        
        <h2 className="mt-4 text-xl font-semibold text-center text-gray-900">
          Something went wrong
        </h2>
        
        <p className="mt-2 text-sm text-center text-gray-600">
          We're sorry, but something unexpected happened. The error has been logged and our team will look into it.
        </p>

        {process.env.NODE_ENV === 'development' && (
          <details className="mt-4 p-3 bg-gray-100 rounded text-xs">
            <summary className="cursor-pointer font-semibold text-gray-700">
              Error Details (Development Only)
            </summary>
            <pre className="mt-2 overflow-auto text-red-600">
              {error.message}
              {'\n\n'}
              {error.stack}
            </pre>
          </details>
        )}

        <div className="mt-6 flex gap-3">
          <button
            onClick={resetError}
            className="flex-1 bg-primary-600 text-white py-2 px-4 rounded-lg hover:bg-primary-700 transition"
          >
            Try Again
          </button>
          <button
            onClick={() => window.location.href = '/'}
            className="flex-1 bg-gray-200 text-gray-700 py-2 px-4 rounded-lg hover:bg-gray-300 transition"
          >
            Go Home
          </button>
        </div>
      </div>
    </div>
  );
};
```

### Feature-Specific Error Boundaries

```typescript
// App.tsx

function App() {
  return (
    <ErrorBoundary>
      <Router>
        <Header />
        
        <main>
          {/* Separate boundary for main content - won't crash header */}
          <ErrorBoundary fallback={PageErrorFallback}>
            <Routes>
              <Route path="/" element={<Home />} />
              <Route path="/dashboard" element={<Dashboard />} />
            </Routes>
          </ErrorBoundary>
        </main>

        <Footer />
      </Router>
    </ErrorBoundary>
  );
}
```

---

## 🔧 Error Types & Handling

### Define Error Classes

```typescript
// types/errors.ts

export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500,
    public isOperational: boolean = true
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class ValidationError extends AppError {
  constructor(message: string, public field?: string) {
    super(message, 'VALIDATION_ERROR', 400);
  }
}

export class AuthenticationError extends AppError {
  constructor(message: string = 'Authentication required') {
    super(message, 'AUTH_ERROR', 401);
  }
}

export class AuthorizationError extends AppError {
  constructor(message: string = 'Insufficient permissions') {
    super(message, 'AUTHORIZATION_ERROR', 403);
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 'NOT_FOUND', 404);
  }
}

export class NetworkError extends AppError {
  constructor(message: string = 'Network request failed') {
    super(message, 'NETWORK_ERROR', 503);
  }
}

export class DatabaseError extends AppError {
  constructor(message: string, public operation?: string) {
    super(message, 'DATABASE_ERROR', 500);
  }
}
```

### Error Handler Service

```typescript
// services/errorHandler.ts

class ErrorHandlerService {
  /**
   * Handle errors uniformly across the application
   */
  handle(error: unknown, context?: string): AppError {
    // Convert unknown errors to AppError
    const appError = this.normalizeError(error);

    // Log the error
    this.log(appError, context);

    // Send to monitoring service
    this.report(appError, context);

    return appError;
  }

  /**
   * Convert any error to AppError
   */
  private normalizeError(error: unknown): AppError {
    // Already an AppError
    if (error instanceof AppError) {
      return error;
    }

    // Firebase error
    if (this.isFirebaseError(error)) {
      return this.handleFirebaseError(error);
    }

    // Standard Error
    if (error instanceof Error) {
      return new AppError(
        error.message,
        'UNKNOWN_ERROR',
        500,
        false // Not operational
      );
    }

    // Unknown error type
    return new AppError(
      'An unexpected error occurred',
      'UNKNOWN_ERROR',
      500,
      false
    );
  }

  /**
   * Handle Firebase-specific errors
   */
  private handleFirebaseError(error: any): AppError {
    const code = error.code;
    const message = this.getFirebaseErrorMessage(code);

    switch (code) {
      case 'auth/user-not-found':
      case 'auth/wrong-password':
        return new AuthenticationError(message);
      
      case 'auth/email-already-in-use':
        return new ValidationError(message, 'email');
      
      case 'auth/weak-password':
        return new ValidationError(message, 'password');
      
      case 'permission-denied':
        return new AuthorizationError(message);
      
      case 'not-found':
        return new NotFoundError('Resource');
      
      case 'unavailable':
        return new NetworkError(message);
      
      default:
        return new AppError(message, code, 500);
    }
  }

  /**
   * Get user-friendly Firebase error messages
   */
  private getFirebaseErrorMessage(code: string): string {
    const messages: Record<string, string> = {
      'auth/user-not-found': 'No account found with this email',
      'auth/wrong-password': 'Incorrect password',
      'auth/email-already-in-use': 'An account with this email already exists',
      'auth/weak-password': 'Password should be at least 6 characters',
      'auth/invalid-email': 'Invalid email address',
      'auth/user-disabled': 'This account has been disabled',
      'auth/too-many-requests': 'Too many attempts. Please try again later',
      'permission-denied': 'You do not have permission to perform this action',
      'not-found': 'The requested resource was not found',
      'unavailable': 'Service temporarily unavailable. Please try again',
      'already-exists': 'This resource already exists',
    };

    return messages[code] || 'An unexpected error occurred. Please try again';
  }

  /**
   * Log error to console and storage
   */
  private log(error: AppError, context?: string) {
    const logData = {
      timestamp: new Date().toISOString(),
      code: error.code,
      message: error.message,
      statusCode: error.statusCode,
      context: context || 'Unknown',
      stack: error.stack,
      isOperational: error.isOperational,
    };

    if (error.statusCode >= 500) {
      console.error('🔴 Error:', logData);
    } else if (error.statusCode >= 400) {
      console.warn('🟡 Warning:', logData);
    } else {
      console.info('🔵 Info:', logData);
    }

    // Store in local storage for debugging (development only)
    if (process.env.NODE_ENV === 'development') {
      this.storeError(logData);
    }
  }

  /**
   * Report error to monitoring service
   */
  private report(error: AppError, context?: string) {
    // Only report operational errors to monitoring
    if (!error.isOperational) {
      return;
    }

    // Don't spam monitoring with client errors in development
    if (process.env.NODE_ENV === 'development') {
      return;
    }

    // Send to Sentry, LogRocket, etc.
    if (window.Sentry) {
      window.Sentry.captureException(error, {
        tags: {
          code: error.code,
          context: context || 'Unknown',
        },
        level: error.statusCode >= 500 ? 'error' : 'warning',
      });
    }
  }

  /**
   * Store error in local storage (development)
   */
  private storeError(logData: any) {
    try {
      const errors = JSON.parse(localStorage.getItem('app_errors') || '[]');
      errors.push(logData);
      
      // Keep only last 50 errors
      if (errors.length > 50) {
        errors.shift();
      }
      
      localStorage.setItem('app_errors', JSON.stringify(errors));
    } catch (e) {
      // Ignore localStorage errors
    }
  }

  private isFirebaseError(error: any): boolean {
    return error && typeof error.code === 'string' && (
      error.code.startsWith('auth/') ||
      error.code === 'permission-denied' ||
      error.code === 'not-found' ||
      error.code === 'unavailable'
    );
  }

  /**
   * Get user-friendly error message
   */
  getUserMessage(error: AppError): string {
    if (error.isOperational) {
      return error.message;
    }

    // Don't expose non-operational errors to users
    return 'An unexpected error occurred. Please try again or contact support if the problem persists.';
  }
}

export const errorHandler = new ErrorHandlerService();
```

---

## 🎣 Error Handling in Hooks

### useAsync Hook

```typescript
// hooks/useAsync.ts

interface UseAsyncState<T> {
  data: T | null;
  loading: boolean;
  error: AppError | null;
}

export function useAsync<T>(
  asyncFunction: () => Promise<T>,
  dependencies: any[] = []
) {
  const [state, setState] = useState<UseAsyncState<T>>({
    data: null,
    loading: true,
    error: null,
  });

  useEffect(() => {
    let cancelled = false;

    setState({ data: null, loading: true, error: null });

    asyncFunction()
      .then((data) => {
        if (!cancelled) {
          setState({ data, loading: false, error: null });
        }
      })
      .catch((error) => {
        if (!cancelled) {
          const appError = errorHandler.handle(error, 'useAsync');
          setState({ data: null, loading: false, error: appError });
        }
      });

    return () => {
      cancelled = true;
    };
  }, dependencies);

  return state;
}
```

### useErrorHandler Hook

```typescript
// hooks/useErrorHandler.ts

export function useErrorHandler() {
  const showToast = useToast();

  const handleError = useCallback((error: unknown, context?: string) => {
    const appError = errorHandler.handle(error, context);
    const userMessage = errorHandler.getUserMessage(appError);

    // Show user-friendly message
    showToast({
      type: 'error',
      message: userMessage,
      duration: 5000,
    });

    return appError;
  }, [showToast]);

  return { handleError };
}
```

---

## 🔌 API Error Handling

### Axios Interceptor

```typescript
// config/axios.ts

import axios from 'axios';
import { errorHandler } from '@/services/errorHandler';

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 30000,
});

// Request interceptor
apiClient.interceptors.request.use(
  (config) => {
    // Add auth token
    const token = getAuthToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    // Network error
    if (!error.response) {
      return Promise.reject(
        new NetworkError('Network error. Please check your connection.')
      );
    }

    // HTTP error
    const { status, data } = error.response;

    switch (status) {
      case 400:
        return Promise.reject(new ValidationError(data.message || 'Invalid request'));
      
      case 401:
        // Redirect to login
        window.location.href = '/login';
        return Promise.reject(new AuthenticationError());
      
      case 403:
        return Promise.reject(new AuthorizationError(data.message));
      
      case 404:
        return Promise.reject(new NotFoundError(data.resource || 'Resource'));
      
      case 429:
        return Promise.reject(
          new AppError('Too many requests. Please try again later.', 'RATE_LIMIT', 429)
        );
      
      case 500:
      case 502:
      case 503:
        return Promise.reject(
          new AppError('Server error. Please try again later.', 'SERVER_ERROR', status)
        );
      
      default:
        return Promise.reject(
          new AppError(data.message || 'An error occurred', 'API_ERROR', status)
        );
    }
  }
);
```

---

## 📝 Logging Service

### Logger Implementation

```typescript
// services/logger.ts

type LogLevel = 'debug' | 'info' | 'warn' | 'error';

interface LogEntry {
  level: LogLevel;
  message: string;
  context?: string;
  data?: any;
  timestamp: string;
  userId?: string;
  sessionId?: string;
}

class LoggerService {
  private sessionId: string;

  constructor() {
    this.sessionId = this.generateSessionId();
  }

  debug(message: string, data?: any, context?: string) {
    this.log('debug', message, data, context);
  }

  info(message: string, data?: any, context?: string) {
    this.log('info', message, data, context);
  }

  warn(message: string, data?: any, context?: string) {
    this.log('warn', message, data, context);
  }

  error(message: string, error?: Error | any, context?: string) {
    this.log('error', message, error, context);
  }

  private log(level: LogLevel, message: string, data?: any, context?: string) {
    const entry: LogEntry = {
      level,
      message,
      context,
      data: this.sanitizeData(data),
      timestamp: new Date().toISOString(),
      userId: this.getUserId(),
      sessionId: this.sessionId,
    };

    // Console output with colors
    this.logToConsole(entry);

    // Send to backend in production
    if (process.env.NODE_ENV === 'production') {
      this.sendToBackend(entry);
    }

    // Store locally for debugging
    if (process.env.NODE_ENV === 'development') {
      this.storeLocally(entry);
    }
  }

  private logToConsole(entry: LogEntry) {
    const colors = {
      debug: '\x1b[36m', // Cyan
      info: '\x1b[32m',  // Green
      warn: '\x1b[33m',  // Yellow
      error: '\x1b[31m', // Red
    };

    const reset = '\x1b[0m';
    const color = colors[entry.level];

    const prefix = `${color}[${entry.level.toUpperCase()}]${reset}`;
    const timestamp = `[${entry.timestamp}]`;
    const context = entry.context ? `[${entry.context}]` : '';

    console.log(`${prefix} ${timestamp} ${context} ${entry.message}`);

    if (entry.data) {
      console.log(entry.data);
    }
  }

  private async sendToBackend(entry: LogEntry) {
    try {
      // Only send warn and error in production
      if (entry.level !== 'warn' && entry.level !== 'error') {
        return;
      }

      // Batch logs and send periodically
      await fetch('/api/logs', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(entry),
      });
    } catch (e) {
      // Silently fail - don't let logging break the app
    }
  }

  private storeLocally(entry: LogEntry) {
    try {
      const logs = JSON.parse(localStorage.getItem('app_logs') || '[]');
      logs.push(entry);

      // Keep only last 100 logs
      if (logs.length > 100) {
        logs.shift();
      }

      localStorage.setItem('app_logs', JSON.stringify(logs));
    } catch (e) {
      // Ignore localStorage errors
    }
  }

  private sanitizeData(data: any): any {
    if (!data) return data;

    // Remove sensitive fields
    const sensitiveKeys = ['password', 'token', 'apiKey', 'secret', 'creditCard'];
    
    const sanitize = (obj: any): any => {
      if (typeof obj !== 'object' || obj === null) {
        return obj;
      }

      if (Array.isArray(obj)) {
        return obj.map(sanitize);
      }

      const sanitized: any = {};
      for (const [key, value] of Object.entries(obj)) {
        if (sensitiveKeys.some(sensitive => 
          key.toLowerCase().includes(sensitive.toLowerCase())
        )) {
          sanitized[key] = '[REDACTED]';
        } else {
          sanitized[key] = sanitize(value);
        }
      }

      return sanitized;
    };

    return sanitize(data);
  }

  private getUserId(): string | undefined {
    // Get from auth context or localStorage
    try {
      const user = JSON.parse(localStorage.getItem('user') || '{}');
      return user.id;
    } catch {
      return undefined;
    }
  }

  private generateSessionId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}

export const logger = new LoggerService();
```

---

## 🎨 User-Facing Error Messages

### Toast Notifications

```typescript
// components/Toast.tsx

interface ToastProps {
  type: 'success' | 'error' | 'warning' | 'info';
  message: string;
  onClose: () => void;
}

export const Toast: React.FC<ToastProps> = ({ type, message, onClose }) => {
  useEffect(() => {
    const timer = setTimeout(onClose, 5000);
    return () => clearTimeout(timer);
  }, [onClose]);

  const icons = {
    success: '✓',
    error: '✕',
    warning: '⚠',
    info: 'ℹ',
  };

  const colors = {
    success: 'bg-green-500',
    error: 'bg-red-500',
    warning: 'bg-yellow-500',
    info: 'bg-blue-500',
  };

  return (
    <div className={`${colors[type]} text-white px-6 py-4 rounded-lg shadow-lg flex items-center gap-3`}>
      <span className="text-xl">{icons[type]}</span>
      <p className="flex-1">{message}</p>
      <button onClick={onClose} className="text-white hover:text-gray-200">
        ✕
      </button>
    </div>
  );
};
```

### Error Message Guidelines

```typescript
// ✅ GOOD - Clear, actionable, friendly
"Your password must be at least 8 characters long"
"This email is already registered. Try logging in instead?"
"We couldn't process your payment. Please check your card details"
"Your session has expired. Please log in again"

// ❌ BAD - Technical, scary, unhelpful
"ValidationError: password.length < 8"
"FirebaseError: auth/email-already-exists"
"Payment gateway returned status 402"
"JWT token expired at 2025-01-15T10:30:00Z"
```

---

## 🛡️ Production Error Monitoring

### Sentry Integration

```typescript
// config/sentry.ts

import * as Sentry from '@sentry/react';
import { BrowserTracing } from '@sentry/tracing';

if (process.env.NODE_ENV === 'production') {
  Sentry.init({
    dsn: import.meta.env.VITE_SENTRY_DSN,
    integrations: [new BrowserTracing()],
    
    // Performance monitoring
    tracesSampleRate: 0.1, // 10% of transactions
    
    // Release tracking
    release: `${import.meta.env.VITE_APP_NAME}@${import.meta.env.VITE_APP_VERSION}`,
    
    // Environment
    environment: import.meta.env.VITE_ENV,
    
    // Filter out sensitive data
    beforeSend(event, hint) {
      // Don't send if user navigated away
      if (event.exception?.values?.[0]?.value?.includes('Navigation')) {
        return null;
      }
      
      // Sanitize URLs
      if (event.request?.url) {
        event.request.url = event.request.url.replace(/token=[^&]+/, 'token=[REDACTED]');
      }
      
      return event;
    },
  });
}
```

---

## ✅ Error Handling Checklist

**Before deploying to production:**

```
Error Boundaries
☐ Global error boundary wraps entire app
☐ Feature-specific boundaries for critical sections
☐ Error fallback UIs are user-friendly
☐ Development mode shows error details

Error Classes
☐ Custom error classes defined
☐ All error types covered
☐ Firebase errors mapped to app errors
☐ User-friendly messages for all errors

Logging
☐ Logger service implemented
☐ Sensitive data sanitized
☐ Logs include context and timestamp
☐ Production logs sent to backend

Monitoring
☐ Error monitoring service integrated (Sentry, etc.)
☐ Error notifications configured
☐ Performance monitoring enabled
☐ Source maps uploaded for stack traces

User Experience
☐ Toast notifications for non-critical errors
☐ Error messages are clear and actionable
☐ No technical jargon shown to users
☐ Retry and recovery options provided
☐ Loading states during async operations

Network Errors
☐ Axios interceptors configured
☐ Network timeouts handled
☐ Retry logic for failed requests
☐ Offline mode messaging

Authentication Errors
☐ 401 redirects to login
☐ 403 shows permission denied message
☐ Token refresh implemented
☐ Session expiry handled gracefully

Database Errors
☐ Firestore errors caught and logged
☐ Permission denied errors handled
☐ Network errors retried
☐ Optimistic updates with rollback

IF ANY ☐ UNCHECKED → Not production-ready!
```

---

## 🚨 AI Decision Gate

```
┌─────────────────────────────────────────────┐
│  ERROR HANDLING CHECKLIST                   │
├─────────────────────────────────────────────┤
│  Before any async operation:                │
│  ☐ Wrapped in try-catch                     │
│  ☐ Error logged with context               │
│  ☐ User-friendly error message              │
│  ☐ Loading state implemented                │
│  ☐ Error state implemented                  │
│                                             │
│  Before any user input:                     │
│  ☐ Validation implemented                   │
│  ☐ Error messages defined                   │
│  ☐ Field-level errors shown                 │
│  ☐ Form-level errors shown                  │
│                                             │
│  Before production:                         │
│  ☐ Error boundaries in place                │
│  ☐ Error monitoring configured              │
│  ☐ All errors have user messages            │
│  ☐ Sensitive data not exposed               │
│  ☐ Logging service active                   │
│                                             │
│  IF ANY ☐ UNCHECKED:                        │
│  ➜ IMPLEMENT BEFORE PROCEEDING              │
└─────────────────────────────────────────────┘
```

---

**Remember: "Users forgive bugs, but not poor error handling. Always fail gracefully."** 🛡️✨

