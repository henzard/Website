---
description: "Analytics, tracking, and user behavior monitoring"
alwaysApply: false
globs: ["**/analytics/**", "**/tracking/**"]
---

# Analytics & Tracking Standards

## 🎯 Core Principles

- ✅ Track meaningful events, not everything
- ✅ Respect user privacy (GDPR/POPIA)
- ✅ Use consent-based tracking
- ✅ Combine quantitative + qualitative data
- ✅ Make data actionable

---

## 📊 Google Analytics 4 Setup

```typescript
// lib/analytics.ts
import { getAnalytics, logEvent, setUserId, setUserProperties } from 'firebase/analytics';
import { app } from './firebase';

const analytics = getAnalytics(app);

// Track page views
export const trackPageView = (pageName: string, pageLocation: string) => {
  logEvent(analytics, 'page_view', {
    page_title: pageName,
    page_location: pageLocation,
  });
};

// Track custom events
export const trackEvent = (
  eventName: string,
  params?: Record<string, any>
) => {
  logEvent(analytics, eventName, params);
};

// Set user ID after login
export const setAnalyticsUser = (userId: string) => {
  setUserId(analytics, userId);
};

// Set user properties
export const setUserProps = (properties: Record<string, string>) => {
  setUserProperties(analytics, properties);
};
```

---

## 🎯 Essential Events to Track

```typescript
// events/trackingEvents.ts

// User journey
export const trackSignUp = (method: string) => {
  trackEvent('sign_up', { method });
};

export const trackLogin = (method: string) => {
  trackEvent('login', { method });
};

// E-commerce
export const trackViewProduct = (product: Product) => {
  trackEvent('view_item', {
    currency: 'ZAR',
    value: product.price,
    items: [{
      item_id: product.id,
      item_name: product.name,
      price: product.price,
    }],
  });
};

export const trackAddToCart = (product: Product, quantity: number) => {
  trackEvent('add_to_cart', {
    currency: 'ZAR',
    value: product.price * quantity,
    items: [{
      item_id: product.id,
      item_name: product.name,
      price: product.price,
      quantity,
    }],
  });
};

export const trackPurchase = (orderId: string, total: number, items: any[]) => {
  trackEvent('purchase', {
    transaction_id: orderId,
    currency: 'ZAR',
    value: total,
    items,
  });
};

// Engagement
export const trackSearch = (searchTerm: string) => {
  trackEvent('search', { search_term: searchTerm });
};

export const trackContactFormSubmit = () => {
  trackEvent('generate_lead', { event_category: 'contact' });
};

export const trackBookingComplete = (serviceId: string, value: number) => {
  trackEvent('booking_complete', {
    service_id: serviceId,
    value,
    currency: 'ZAR',
  });
};
```

---

## 🍪 Consent Management

```typescript
// components/CookieConsent.tsx
export const CookieConsent: React.FC = () => {
  const [consent, setConsent] = useState<string | null>(
    localStorage.getItem('analytics_consent')
  );

  const handleAccept = () => {
    localStorage.setItem('analytics_consent', 'granted');
    setConsent('granted');
    // Enable analytics
    window.gtag?.('consent', 'update', {
      analytics_storage: 'granted',
    });
  };

  const handleDecline = () => {
    localStorage.setItem('analytics_consent', 'denied');
    setConsent('denied');
  };

  if (consent) return null;

  return (
    <div className="fixed bottom-0 left-0 right-0 bg-white border-t p-4 shadow-lg z-50">
      <div className="max-w-4xl mx-auto flex items-center justify-between">
        <p className="text-sm text-gray-600">
          We use cookies to improve your experience. 
          <a href="/privacy" className="underline ml-1">Learn more</a>
        </p>
        <div className="flex gap-2">
          <button
            onClick={handleDecline}
            className="px-4 py-2 text-gray-600 hover:underline"
          >
            Decline
          </button>
          <button
            onClick={handleAccept}
            className="px-4 py-2 bg-primary-600 text-white rounded-lg"
          >
            Accept
          </button>
        </div>
      </div>
    </div>
  );
};
```

---

## 📈 React Router Integration

```typescript
// hooks/usePageTracking.ts
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';
import { trackPageView } from '@/lib/analytics';

export const usePageTracking = () => {
  const location = useLocation();

  useEffect(() => {
    // Only track if consent given
    if (localStorage.getItem('analytics_consent') === 'granted') {
      trackPageView(
        document.title,
        window.location.href
      );
    }
  }, [location]);
};

// In App.tsx
function App() {
  usePageTracking();
  return <Routes>...</Routes>;
}
```

---

## 📋 Key Metrics to Monitor

| Metric | What It Tells You |
|--------|-------------------|
| Bounce Rate | Landing page effectiveness |
| Session Duration | Content engagement |
| Conversion Rate | Funnel effectiveness |
| Pages per Session | Site navigation quality |
| User Flow | Where users drop off |
| Events | Specific action completion |

---

## ✅ Checklist

```
Setup
☐ GA4 property created
☐ Firebase Analytics linked
☐ Debug mode for development
☐ Production property separate

Consent
☐ Cookie banner implemented
☐ Consent stored in localStorage
☐ Analytics respects consent
☐ Privacy policy updated

Events
☐ Page views tracked
☐ Key conversions tracked
☐ E-commerce events (if applicable)
☐ Search queries tracked

Privacy
☐ No PII in events
☐ User ID hashed
☐ IP anonymization enabled
☐ Data retention configured

IF ANY ☐ UNCHECKED → Fix before launching
```

---

**Remember: "Track to improve, not to spy."** 📊✨

