---
description: "UI/UX design principles, accessibility standards, and component design patterns"
alwaysApply: true
globs: ["**/components/**", "**/pages/**", "**/*.css", "**/*.tsx"]
---

# UI/UX Design Standards

## Core Design Principles

### 1. Clean and Minimal Design
- **Less is more**: Remove unnecessary elements
- **White space**: Use generous spacing for breathing room
- **Visual hierarchy**: Clear distinction between primary and secondary content
- **Consistency**: Same patterns throughout the application

### 2. User-Centric Design
- **Mobile-first**: Design for mobile, enhance for desktop
- **Touch-friendly**: Minimum 44x44px touch targets
- **Progressive disclosure**: Show what's needed, hide complexity
- **Immediate feedback**: Visual response to all user actions

## Design System Implementation

### Color System

```typescript
// tailwind.config.js or theme.ts
export const colors = {
  // Must be provided by client before development
  primary: {
    50: '#...',  // Lightest
    100: '#...',
    200: '#...',
    300: '#...',
    400: '#...',
    500: '#...', // Main brand color
    600: '#...',
    700: '#...',
    800: '#...',
    900: '#...', // Darkest
  },
  secondary: {...},
  accent: {...},
  neutral: {
    white: '#FFFFFF',
    50: '#F9FAFB',
    100: '#F3F4F6',
    // ... grays
    900: '#111827',
    black: '#000000',
  },
  semantic: {
    success: '#10B981',
    warning: '#F59E0B',
    error: '#EF4444',
    info: '#3B82F6',
  }
};
```

### Typography Scale

```typescript
export const typography = {
  fonts: {
    heading: 'Client Heading Font, sans-serif', // From design system
    body: 'Client Body Font, sans-serif',
    mono: 'JetBrains Mono, monospace',
  },
  sizes: {
    xs: '0.75rem',    // 12px
    sm: '0.875rem',   // 14px
    base: '1rem',     // 16px
    lg: '1.125rem',   // 18px
    xl: '1.25rem',    // 20px
    '2xl': '1.5rem',  // 24px
    '3xl': '1.875rem', // 30px
    '4xl': '2.25rem',  // 36px
    '5xl': '3rem',     // 48px
  },
  weights: {
    normal: 400,
    medium: 500,
    semibold: 600,
    bold: 700,
  },
  lineHeights: {
    tight: 1.2,
    normal: 1.5,
    relaxed: 1.75,
  }
};
```

### Spacing System

```typescript
// Use consistent spacing scale (Tailwind default recommended)
export const spacing = {
  0: '0',
  1: '0.25rem',  // 4px
  2: '0.5rem',   // 8px
  3: '0.75rem',  // 12px
  4: '1rem',     // 16px
  5: '1.25rem',  // 20px
  6: '1.5rem',   // 24px
  8: '2rem',     // 32px
  10: '2.5rem',  // 40px
  12: '3rem',    // 48px
  16: '4rem',    // 64px
  20: '5rem',    // 80px
  24: '6rem',    // 96px
};
```

## Component Design Patterns

### Button Component

```typescript
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger';
  size: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
  fullWidth?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
  onClick?: () => void;
  type?: 'button' | 'submit' | 'reset';
  children: React.ReactNode;
}

export const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'md',
  loading = false,
  disabled = false,
  fullWidth = false,
  leftIcon,
  rightIcon,
  children,
  ...props
}) => {
  const baseStyles = "inline-flex items-center justify-center font-medium rounded-lg transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2";
  
  const variants = {
    primary: "bg-primary-600 text-white hover:bg-primary-700 focus:ring-primary-500",
    secondary: "bg-secondary-600 text-white hover:bg-secondary-700 focus:ring-secondary-500",
    outline: "border-2 border-primary-600 text-primary-600 hover:bg-primary-50",
    ghost: "text-primary-600 hover:bg-primary-50",
    danger: "bg-red-600 text-white hover:bg-red-700 focus:ring-red-500",
  };
  
  const sizes = {
    sm: "px-3 py-1.5 text-sm",
    md: "px-4 py-2 text-base",
    lg: "px-6 py-3 text-lg",
  };

  return (
    <button
      className={`
        ${baseStyles}
        ${variants[variant]}
        ${sizes[size]}
        ${fullWidth ? 'w-full' : ''}
        ${disabled || loading ? 'opacity-50 cursor-not-allowed' : ''}
      `}
      disabled={disabled || loading}
      aria-busy={loading}
      {...props}
    >
      {loading && <Spinner className="mr-2" />}
      {leftIcon && <span className="mr-2">{leftIcon}</span>}
      {children}
      {rightIcon && <span className="ml-2">{rightIcon}</span>}
    </button>
  );
};
```

### Form Input Component

```typescript
interface InputProps {
  label: string;
  name: string;
  type?: 'text' | 'email' | 'password' | 'number' | 'tel';
  placeholder?: string;
  error?: string;
  helperText?: string;
  required?: boolean;
  disabled?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

export const Input = forwardRef<HTMLInputElement, InputProps>(({
  label,
  name,
  type = 'text',
  error,
  helperText,
  required = false,
  disabled = false,
  leftIcon,
  rightIcon,
  ...props
}, ref) => {
  const inputId = `input-${name}`;
  const errorId = `${inputId}-error`;
  const helperId = `${inputId}-helper`;

  return (
    <div className="w-full">
      <label 
        htmlFor={inputId}
        className="block text-sm font-medium text-gray-700 mb-1"
      >
        {label}
        {required && <span className="text-red-500 ml-1">*</span>}
      </label>
      
      <div className="relative">
        {leftIcon && (
          <div className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400">
            {leftIcon}
          </div>
        )}
        
        <input
          ref={ref}
          id={inputId}
          name={name}
          type={type}
          className={`
            w-full px-4 py-2 border rounded-lg
            ${leftIcon ? 'pl-10' : ''}
            ${rightIcon ? 'pr-10' : ''}
            ${error 
              ? 'border-red-500 focus:ring-red-500 focus:border-red-500' 
              : 'border-gray-300 focus:ring-primary-500 focus:border-primary-500'
            }
            disabled:bg-gray-100 disabled:cursor-not-allowed
            transition-colors duration-200
          `}
          aria-invalid={!!error}
          aria-describedby={error ? errorId : helperText ? helperId : undefined}
          aria-required={required}
          disabled={disabled}
          {...props}
        />
        
        {rightIcon && (
          <div className="absolute right-3 top-1/2 -translate-y-1/2 text-gray-400">
            {rightIcon}
          </div>
        )}
      </div>
      
      {error && (
        <p id={errorId} className="mt-1 text-sm text-red-600" role="alert">
          {error}
        </p>
      )}
      
      {helperText && !error && (
        <p id={helperId} className="mt-1 text-sm text-gray-500">
          {helperText}
        </p>
      )}
    </div>
  );
});
```

## Loading States

### Skeleton Screens

```typescript
export const SkeletonCard: React.FC = () => (
  <div className="animate-pulse">
    <div className="h-48 bg-gray-200 rounded-t-lg" />
    <div className="p-4 space-y-3">
      <div className="h-4 bg-gray-200 rounded w-3/4" />
      <div className="h-4 bg-gray-200 rounded w-1/2" />
    </div>
  </div>
);

export const SkeletonList: React.FC = () => (
  <div className="space-y-4">
    {[...Array(5)].map((_, i) => (
      <div key={i} className="flex items-center space-x-4 animate-pulse">
        <div className="w-12 h-12 bg-gray-200 rounded-full" />
        <div className="flex-1 space-y-2">
          <div className="h-4 bg-gray-200 rounded w-1/4" />
          <div className="h-3 bg-gray-200 rounded w-1/3" />
        </div>
      </div>
    ))}
  </div>
);
```

## Responsive Design

### Breakpoints

```typescript
export const breakpoints = {
  sm: '640px',   // Mobile landscape
  md: '768px',   // Tablet
  lg: '1024px',  // Desktop
  xl: '1280px',  // Large desktop
  '2xl': '1536px', // Extra large
};

// Usage in components
<div className="
  grid 
  grid-cols-1 
  sm:grid-cols-2 
  lg:grid-cols-3 
  xl:grid-cols-4 
  gap-4
">
  {/* Responsive grid */}
</div>
```

## Accessibility Requirements

### WCAG 2.1 AA Compliance

```typescript
// ✅ GOOD - Accessible modal
export const Modal: React.FC<ModalProps> = ({ isOpen, onClose, title, children }) => {
  useEffect(() => {
    if (isOpen) {
      document.body.style.overflow = 'hidden';
      // Focus trap implementation
    }
    return () => {
      document.body.style.overflow = 'unset';
    };
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      className="fixed inset-0 z-50 flex items-center justify-center"
    >
      <div 
        className="fixed inset-0 bg-black bg-opacity-50"
        onClick={onClose}
        aria-hidden="true"
      />
      
      <div className="relative bg-white rounded-lg p-6 max-w-md w-full">
        <h2 id="modal-title" className="text-xl font-semibold mb-4">
          {title}
        </h2>
        
        <button
          onClick={onClose}
          className="absolute top-4 right-4"
          aria-label="Close modal"
        >
          <XIcon />
        </button>
        
        {children}
      </div>
    </div>
  );
};
```

### Keyboard Navigation

- All interactive elements must be keyboard accessible
- Tab order must be logical
- Focus indicators must be visible
- Escape key closes modals/dialogs

### Screen Reader Support

- Use semantic HTML (`<nav>`, `<main>`, `<article>`)
- Provide ARIA labels where needed
- Alt text for all images
- Announce dynamic content changes

## Animation Guidelines

```typescript
// Subtle, purposeful animations only
export const animations = {
  fast: '150ms',
  normal: '250ms',
  slow: '350ms',
  
  // Respect user preferences
  respectMotion: {
    '@media (prefers-reduced-motion: reduce)': {
      animation: 'none',
      transition: 'none',
    }
  }
};
```

## AI Decision Gate

**Before implementing UI components:**
1. ✅ Is the design system documentation referenced?
2. ✅ Are colors from the approved palette?
3. ✅ Is the component accessible (ARIA, keyboard, screen reader)?
4. ✅ Are loading and error states designed?
5. ✅ Is the component responsive (mobile-first)?
6. ✅ Are animations subtle and respect user preferences?
7. ✅ Does it follow the naming conventions?

**Never proceed without approved design system. Never guess colors or spacing.**

