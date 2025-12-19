---
description: "Internationalization (i18n), multi-language support, and localization best practices"
alwaysApply: false
---

# Internationalization (i18n) Standards

## 🎯 Core Principles

**"Build global from day one, even if you start local."**

i18n requirements:
- ✅ Separate content from code
- ✅ Support multiple languages
- ✅ Handle right-to-left (RTL) languages
- ✅ Format dates, numbers, currency by locale
- ✅ Translate all user-facing text
- ✅ Easy for translators to contribute

---

## 🌍 Setup i18n Library

### Install react-i18next

```bash
npm install react-i18next i18next i18next-http-backend i18next-browser-languagedetector
```

### Configuration

```typescript
// i18n.ts

import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';
import Backend from 'i18next-http-backend';

i18n
  // Load translations using http backend
  .use(Backend)
  // Detect user language
  .use(LanguageDetector)
  // Pass to React
  .use(initReactI18next)
  // Initialize
  .init({
    fallbackLng: 'en',
    debug: process.env.NODE_ENV === 'development',
    
    // Supported languages
    supportedLngs: ['en', 'es', 'fr', 'de', 'pt', 'zh', 'ar'],
    
    // Namespace
    ns: ['common', 'auth', 'dashboard', 'booking'],
    defaultNS: 'common',
    
    interpolation: {
      escapeValue: false, // React already escapes
    },
    
    backend: {
      // Load from public folder
      loadPath: '/locales/{{lng}}/{{ns}}.json',
    },
    
    detection: {
      // Order of detection
      order: ['querystring', 'cookie', 'localStorage', 'navigator'],
      caches: ['cookie', 'localStorage'],
      cookieMinutes: 10080, // 7 days
    },
  });

export default i18n;
```

### Initialize in App

```typescript
// main.tsx

import React from 'react';
import ReactDOM from 'react-dom/client';
import './i18n'; // Import i18n configuration
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

---

## 📝 Translation Files

### File Structure

```
public/
├── locales/
│   ├── en/
│   │   ├── common.json
│   │   ├── auth.json
│   │   ├── dashboard.json
│   │   └── booking.json
│   ├── es/
│   │   ├── common.json
│   │   ├── auth.json
│   │   ├── dashboard.json
│   │   └── booking.json
│   ├── fr/
│   │   └── ...
│   └── ar/
│       └── ...
```

### English Translations

```json
// public/locales/en/common.json
{
  "welcome": "Welcome to YourApp",
  "navigation": {
    "home": "Home",
    "about": "About",
    "contact": "Contact",
    "dashboard": "Dashboard",
    "profile": "Profile",
    "logout": "Log Out"
  },
  "actions": {
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete",
    "edit": "Edit",
    "submit": "Submit",
    "confirm": "Confirm",
    "back": "Back"
  },
  "messages": {
    "loading": "Loading...",
    "error": "Something went wrong",
    "success": "Success!",
    "noData": "No data available"
  }
}
```

```json
// public/locales/en/auth.json
{
  "login": {
    "title": "Sign In",
    "email": "Email Address",
    "password": "Password",
    "submit": "Sign In",
    "forgotPassword": "Forgot password?",
    "noAccount": "Don't have an account?",
    "signUpLink": "Sign up"
  },
  "signup": {
    "title": "Create Account",
    "firstName": "First Name",
    "lastName": "Last Name",
    "email": "Email Address",
    "password": "Password",
    "confirmPassword": "Confirm Password",
    "submit": "Create Account",
    "hasAccount": "Already have an account?",
    "signInLink": "Sign in"
  },
  "errors": {
    "invalidEmail": "Invalid email address",
    "passwordTooShort": "Password must be at least 6 characters",
    "passwordMismatch": "Passwords do not match",
    "emailInUse": "Email already in use",
    "userNotFound": "No account found with this email",
    "wrongPassword": "Incorrect password"
  }
}
```

### Spanish Translations

```json
// public/locales/es/common.json
{
  "welcome": "Bienvenido a YourApp",
  "navigation": {
    "home": "Inicio",
    "about": "Acerca de",
    "contact": "Contacto",
    "dashboard": "Panel",
    "profile": "Perfil",
    "logout": "Cerrar Sesión"
  },
  "actions": {
    "save": "Guardar",
    "cancel": "Cancelar",
    "delete": "Eliminar",
    "edit": "Editar",
    "submit": "Enviar",
    "confirm": "Confirmar",
    "back": "Atrás"
  },
  "messages": {
    "loading": "Cargando...",
    "error": "Algo salió mal",
    "success": "¡Éxito!",
    "noData": "No hay datos disponibles"
  }
}
```

---

## 💬 Using Translations in Components

### Basic Usage

```typescript
// components/Header.tsx

import { useTranslation } from 'react-i18next';

export const Header: React.FC = () => {
  const { t } = useTranslation('common');

  return (
    <header>
      <nav>
        <Link to="/">{t('navigation.home')}</Link>
        <Link to="/about">{t('navigation.about')}</Link>
        <Link to="/contact">{t('navigation.contact')}</Link>
        <Link to="/dashboard">{t('navigation.dashboard')}</Link>
      </nav>
    </header>
  );
};
```

### With Interpolation

```typescript
// components/Greeting.tsx

import { useTranslation } from 'react-i18next';

export const Greeting: React.FC<{ userName: string }> = ({ userName }) => {
  const { t } = useTranslation('common');

  return (
    <h1>{t('greeting', { name: userName })}</h1>
  );
};

// Translation files
// en/common.json: "greeting": "Hello, {{name}}!"
// es/common.json: "greeting": "¡Hola, {{name}}!"
```

### With Pluralization

```typescript
// components/ItemCount.tsx

export const ItemCount: React.FC<{ count: number }> = ({ count }) => {
  const { t } = useTranslation('common');

  return <p>{t('itemCount', { count })}</p>;
};

// Translation files
// en/common.json:
// "itemCount_one": "{{count}} item"
// "itemCount_other": "{{count}} items"

// es/common.json:
// "itemCount_one": "{{count}} artículo"
// "itemCount_other": "{{count}} artículos"
```

---

## 🔄 Language Switcher

```typescript
// components/LanguageSwitcher.tsx

import { useTranslation } from 'react-i18next';

const languages = [
  { code: 'en', name: 'English', flag: '🇺🇸' },
  { code: 'es', name: 'Español', flag: '🇪🇸' },
  { code: 'fr', name: 'Français', flag: '🇫🇷' },
  { code: 'de', name: 'Deutsch', flag: '🇩🇪' },
  { code: 'pt', name: 'Português', flag: '🇵🇹' },
  { code: 'zh', name: '中文', flag: '🇨🇳' },
  { code: 'ar', name: 'العربية', flag: '🇸🇦' },
];

export const LanguageSwitcher: React.FC = () => {
  const { i18n } = useTranslation();

  const changeLanguage = async (languageCode: string) => {
    await i18n.changeLanguage(languageCode);
    
    // Update user preference in Firestore
    const user = auth.currentUser;
    if (user) {
      await updateDoc(doc(db, 'users', user.uid), {
        language: languageCode,
      });
    }
    
    // Update HTML lang attribute
    document.documentElement.lang = languageCode;
    
    // Update HTML dir attribute for RTL languages
    document.documentElement.dir = ['ar', 'he', 'fa'].includes(languageCode) ? 'rtl' : 'ltr';
  };

  return (
    <div className="language-switcher">
      <select
        value={i18n.language}
        onChange={(e) => changeLanguage(e.target.value)}
        className="px-3 py-2 border rounded"
      >
        {languages.map((lang) => (
          <option key={lang.code} value={lang.code}>
            {lang.flag} {lang.name}
          </option>
        ))}
      </select>
    </div>
  );
};
```

---

## 📅 Date & Time Formatting

```typescript
// utils/dateFormatter.ts

import { format, parseISO } from 'date-fns';
import { enUS, es, fr, de, ptBR, zhCN, ar } from 'date-fns/locale';

const locales: Record<string, Locale> = {
  en: enUS,
  es: es,
  fr: fr,
  de: de,
  pt: ptBR,
  zh: zhCN,
  ar: ar,
};

export function formatDate(
  date: Date | string,
  formatStr: string = 'PPP',
  language: string = 'en'
): string {
  const dateObj = typeof date === 'string' ? parseISO(date) : date;
  const locale = locales[language] || enUS;

  return format(dateObj, formatStr, { locale });
}

// Usage in components
import { useTranslation } from 'react-i18next';
import { formatDate } from '@/utils/dateFormatter';

function EventDate({ date }: { date: Date }) {
  const { i18n } = useTranslation();
  
  return <span>{formatDate(date, 'PPP', i18n.language)}</span>;
}

// Output examples:
// English: January 15, 2025
// Spanish: 15 de enero de 2025
// French: 15 janvier 2025
```

---

## 💰 Number & Currency Formatting

```typescript
// utils/numberFormatter.ts

export function formatNumber(
  value: number,
  locale: string = 'en-US'
): string {
  return new Intl.NumberFormat(locale).format(value);
}

export function formatCurrency(
  value: number,
  currency: string = 'USD',
  locale: string = 'en-US'
): string {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency: currency,
  }).format(value);
}

// Usage
formatNumber(1234567.89, 'en-US'); // "1,234,567.89"
formatNumber(1234567.89, 'de-DE'); // "1.234.567,89"

formatCurrency(1999, 'USD', 'en-US'); // "$1,999.00"
formatCurrency(1999, 'EUR', 'de-DE'); // "1.999,00 €"
formatCurrency(1999, 'GBP', 'en-GB'); // "£1,999.00"
```

---

## 🔄 RTL (Right-to-Left) Support

### CSS for RTL

```css
/* styles/rtl.css */

[dir="rtl"] {
  direction: rtl;
}

[dir="rtl"] .text-left {
  text-align: right;
}

[dir="rtl"] .text-right {
  text-align: left;
}

[dir="rtl"] .ml-4 {
  margin-left: 0;
  margin-right: 1rem;
}

[dir="rtl"] .mr-4 {
  margin-right: 0;
  margin-left: 1rem;
}

/* Use logical properties for better RTL support */
.padding-inline-start {
  padding-inline-start: 1rem;
}

.margin-inline-end {
  margin-inline-end: 1rem;
}
```

### Tailwind CSS RTL Plugin

```bash
npm install tailwindcss-rtl
```

```javascript
// tailwind.config.js
module.exports = {
  plugins: [
    require('tailwindcss-rtl'),
  ],
};

// Usage in components
<div className="ms-4 me-8"> {/* margin-start and margin-end */}
  <p className="text-start">Text</p> {/* text-align start (left in LTR, right in RTL) */}
</div>
```

---

## 🗂️ Translation Management

### Extract Translation Keys

```bash
# Install extraction tool
npm install -D i18next-parser

# i18next-parser.config.js
module.exports = {
  locales: ['en', 'es', 'fr', 'de', 'pt', 'zh', 'ar'],
  output: 'public/locales/$LOCALE/$NAMESPACE.json',
  input: ['src/**/*.{ts,tsx}'],
  keySeparator: '.',
  namespaceSeparator: ':',
};

# Add script to package.json
{
  "scripts": {
    "extract-translations": "i18next-parser"
  }
}
```

### Translation Workflow

```
1. Developer writes code with t('key')
2. Run extraction: npm run extract-translations
3. Send translation files to translators
4. Translators fill in translations
5. Commit translated files
6. Deploy with new translations
```

---

## 📱 SEO for i18n

### Hreflang Tags

```typescript
// components/SEO.tsx

import { Helmet } from 'react-helmet-async';
import { useTranslation } from 'react-i18next';

interface SEOProps {
  title: string;
  description: string;
  path: string;
}

export const SEO: React.FC<SEOProps> = ({ title, description, path }) => {
  const { i18n } = useTranslation();
  const baseUrl = 'https://yourapp.com';

  const languages = ['en', 'es', 'fr', 'de', 'pt', 'zh', 'ar'];

  return (
    <Helmet>
      <html lang={i18n.language} dir={['ar', 'he'].includes(i18n.language) ? 'rtl' : 'ltr'} />
      <title>{title}</title>
      <meta name="description" content={description} />
      
      {/* Hreflang tags for each language */}
      {languages.map((lang) => (
        <link
          key={lang}
          rel="alternate"
          hrefLang={lang}
          href={`${baseUrl}/${lang}${path}`}
        />
      ))}
      
      {/* Default language */}
      <link rel="alternate" hrefLang="x-default" href={`${baseUrl}/en${path}`} />
    </Helmet>
  );
};
```

---

## 🔧 Best Practices

### ✅ DO

```typescript
// ✅ GOOD - Use translation keys
<Button>{t('actions.save')}</Button>

// ✅ GOOD - Provide context
// en: "delete": "Delete {{itemType}}"
<Button>{t('actions.delete', { itemType: 'booking' })}</Button>

// ✅ GOOD - Use namespaces
const { t } = useTranslation('auth');
<h1>{t('login.title')}</h1>

// ✅ GOOD - Handle missing translations
<span>{t('key', { defaultValue: 'Fallback text' })}</span>
```

### ❌ DON'T

```typescript
// ❌ BAD - Hardcoded text
<Button>Save</Button>

// ❌ BAD - English in code
const message = 'Your booking has been confirmed';

// ❌ BAD - String concatenation
const fullName = firstName + ' ' + lastName; // Word order differs in some languages

// ✅ GOOD - Use interpolation
// en: "fullName": "{{firstName}} {{lastName}}"
// zh: "fullName": "{{lastName}}{{firstName}}"
const fullName = t('user.fullName', { firstName, lastName });
```

---

## ✅ i18n Checklist

```
Setup
☐ i18next library installed
☐ Configuration initialized
☐ Language detector configured
☐ Supported languages defined
☐ Fallback language set

Translation Files
☐ Translation files organized by namespace
☐ All supported languages have files
☐ Keys are descriptive (not generic)
☐ No hardcoded text in code
☐ Pluralization rules defined

Components
☐ useTranslation hook used everywhere
☐ All user-facing text translated
☐ Dynamic text uses interpolation
☐ Namespaces used appropriately
☐ Loading states while translations load

Formatting
☐ Date formatting by locale
☐ Number formatting by locale
☐ Currency formatting by locale
☐ Relative time formatting
☐ Unit formatting (distance, weight, etc.)

RTL Support
☐ RTL languages supported (if needed)
☐ CSS logical properties used
☐ Layout adjusts for RTL
☐ Icons flip for RTL (if directional)
☐ Text alignment correct in RTL

Language Switcher
☐ Language switcher component
☐ Persists user preference
☐ Updates HTML lang attribute
☐ Updates HTML dir attribute for RTL
☐ Stores preference in user profile

SEO
☐ Hreflang tags for each language
☐ Language in URL structure
☐ Canonical URLs per language
☐ Meta tags translated
☐ Sitemap includes all languages

Testing
☐ Tested in all supported languages
☐ Layout works in all languages
☐ No text overflow issues
☐ RTL languages display correctly
☐ Date/number formats correct

Translation Management
☐ Translation extraction automated
☐ Workflow for translators defined
☐ Translation files version controlled
☐ Missing translations logged
☐ Translation review process

IF ANY ☐ UNCHECKED → i18n not complete
```

---

## 🚨 AI Decision Gate

```
┌─────────────────────────────────────────────┐
│  INTERNATIONALIZATION CHECKLIST             │
├─────────────────────────────────────────────┤
│  Before adding user-facing text:            │
│  ☐ Text is in translation file             │
│  ☐ Translation key is descriptive          │
│  ☐ Interpolation used for dynamic content  │
│  ☐ Pluralization rules defined (if needed) │
│  ☐ Available in all supported languages    │
│                                             │
│  Before formatting data:                    │
│  ☐ Dates formatted with locale             │
│  ☐ Numbers formatted with locale           │
│  ☐ Currency uses correct locale/currency   │
│  ☐ Units appropriate for locale            │
│                                             │
│  Before supporting new language:            │
│  ☐ Translation files created               │
│  ☐ All keys translated                     │
│  ☐ RTL adjustments made (if RTL)           │
│  ☐ Hreflang tags added                     │
│  ☐ Language added to switcher              │
│  ☐ Tested thoroughly                       │
│                                             │
│  IF ANY ☐ UNCHECKED:                        │
│  ➜ ADDRESS BEFORE DEPLOYING                │
└─────────────────────────────────────────────┘
```

---

**Remember: "Think global from day one. Adding i18n later is 10x harder than starting with it."** 🌍✨

