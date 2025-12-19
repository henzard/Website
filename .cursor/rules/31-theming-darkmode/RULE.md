---
description: "Theming system and dark mode implementation"
alwaysApply: false
globs: ["**/theme/**", "**/contexts/Theme*"]
---

# Theming & Dark Mode Standards

## 🎯 Core Principles

- ✅ Respect system preference by default
- ✅ Allow manual override
- ✅ Persist user preference
- ✅ Smooth transition between themes
- ✅ Accessible in all themes

---

## 🎨 CSS Custom Properties

```css
/* styles/theme.css */
:root {
  /* Light theme (default) */
  --color-bg: #ffffff;
  --color-bg-secondary: #f3f4f6;
  --color-text: #111827;
  --color-text-secondary: #6b7280;
  --color-border: #e5e7eb;
  --color-primary: #3b82f6;
  --color-primary-hover: #2563eb;
}

[data-theme="dark"] {
  --color-bg: #111827;
  --color-bg-secondary: #1f2937;
  --color-text: #f9fafb;
  --color-text-secondary: #9ca3af;
  --color-border: #374151;
  --color-primary: #60a5fa;
  --color-primary-hover: #93c5fd;
}

/* Apply theme colors */
body {
  background-color: var(--color-bg);
  color: var(--color-text);
  transition: background-color 0.2s, color 0.2s;
}
```

---

## 🔄 Theme Context

```typescript
// contexts/ThemeContext.tsx
type Theme = 'light' | 'dark' | 'system';

interface ThemeContextType {
  theme: Theme;
  resolvedTheme: 'light' | 'dark';
  setTheme: (theme: Theme) => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setThemeState] = useState<Theme>(() => {
    return (localStorage.getItem('theme') as Theme) || 'system';
  });

  const [resolvedTheme, setResolvedTheme] = useState<'light' | 'dark'>('light');

  useEffect(() => {
    const root = document.documentElement;
    
    const updateTheme = () => {
      let resolved: 'light' | 'dark';
      
      if (theme === 'system') {
        resolved = window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
      } else {
        resolved = theme;
      }
      
      root.setAttribute('data-theme', resolved);
      setResolvedTheme(resolved);
    };

    updateTheme();

    // Listen for system theme changes
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    mediaQuery.addEventListener('change', updateTheme);

    return () => mediaQuery.removeEventListener('change', updateTheme);
  }, [theme]);

  const setTheme = (newTheme: Theme) => {
    setThemeState(newTheme);
    localStorage.setItem('theme', newTheme);
  };

  return (
    <ThemeContext.Provider value={{ theme, resolvedTheme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
};
```

---

## 🌙 Theme Toggle

```typescript
// components/ThemeToggle.tsx
export const ThemeToggle: React.FC = () => {
  const { theme, setTheme } = useTheme();

  return (
    <div className="flex items-center gap-2">
      <button
        onClick={() => setTheme('light')}
        className={`p-2 rounded ${theme === 'light' ? 'bg-primary-100' : ''}`}
        aria-label="Light mode"
      >
        ☀️
      </button>
      <button
        onClick={() => setTheme('dark')}
        className={`p-2 rounded ${theme === 'dark' ? 'bg-primary-100' : ''}`}
        aria-label="Dark mode"
      >
        🌙
      </button>
      <button
        onClick={() => setTheme('system')}
        className={`p-2 rounded ${theme === 'system' ? 'bg-primary-100' : ''}`}
        aria-label="System theme"
      >
        💻
      </button>
    </div>
  );
};

// Simple toggle version
export const SimpleThemeToggle: React.FC = () => {
  const { resolvedTheme, setTheme } = useTheme();

  return (
    <button
      onClick={() => setTheme(resolvedTheme === 'dark' ? 'light' : 'dark')}
      className="p-2 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-800"
      aria-label={`Switch to ${resolvedTheme === 'dark' ? 'light' : 'dark'} mode`}
    >
      {resolvedTheme === 'dark' ? '☀️' : '🌙'}
    </button>
  );
};
```

---

## 🖼️ Image Theming

```typescript
// components/ThemedImage.tsx
export const ThemedImage: React.FC<{
  lightSrc: string;
  darkSrc: string;
  alt: string;
}> = ({ lightSrc, darkSrc, alt }) => {
  const { resolvedTheme } = useTheme();
  
  return (
    <img
      src={resolvedTheme === 'dark' ? darkSrc : lightSrc}
      alt={alt}
      className="transition-opacity"
    />
  );
};

// Or with CSS
<picture>
  <source srcset="/logo-dark.png" media="(prefers-color-scheme: dark)" />
  <img src="/logo-light.png" alt="Logo" />
</picture>
```

---

## 🎨 Tailwind Dark Mode

```javascript
// tailwind.config.js
module.exports = {
  darkMode: 'class', // or 'media' for system-only
  theme: {
    extend: {
      colors: {
        // Your custom colors
      },
    },
  },
};
```

```tsx
// Usage in components
<div className="bg-white dark:bg-gray-900">
  <p className="text-gray-900 dark:text-white">
    Adaptive text
  </p>
</div>
```

---

## 🚫 Avoid Flash of Wrong Theme

```html
<!-- Add to <head> before CSS loads -->
<script>
  (function() {
    const theme = localStorage.getItem('theme');
    if (theme === 'dark' || (!theme && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
      document.documentElement.setAttribute('data-theme', 'dark');
    }
  })();
</script>
```

---

## ✅ Checklist

```
Theme System
☐ CSS custom properties defined
☐ Light and dark values set
☐ Smooth transitions

User Preference
☐ System preference detected
☐ Manual override available
☐ Preference persisted
☐ Toggle component created

Accessibility
☐ Sufficient contrast in both themes
☐ Focus indicators visible
☐ No flash of wrong theme

Assets
☐ Logo works in both themes
☐ Icons adapt to theme
☐ Images have dark variants if needed

IF ANY ☐ UNCHECKED → Fix before launching
```

---

**Remember: "Dark mode isn't just inverting colors. It's a complete design consideration."** 🌙✨

