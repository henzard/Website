---
description: "Progressive Web App and mobile optimization standards"
alwaysApply: false
globs: ["**/sw.js", "**/service-worker.ts", "**/manifest.json"]
---

# PWA & Mobile Standards

## 🎯 Core Principles

- ✅ Works offline (or gracefully degrades)
- ✅ Installable on home screen
- ✅ Fast on slow networks
- ✅ Touch-friendly (44x44px min targets)
- ✅ Responsive to all screen sizes

---

## 📱 Web App Manifest

```json
// public/manifest.json
{
  "name": "My App",
  "short_name": "App",
  "description": "A great web application",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#3b82f6",
  "orientation": "portrait-primary",
  "icons": [
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ]
}
```

```html
<!-- index.html -->
<link rel="manifest" href="/manifest.json" />
<meta name="theme-color" content="#3b82f6" />
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="default" />
<link rel="apple-touch-icon" href="/icons/icon-192.png" />
```

---

## ⚡ Service Worker (Vite PWA)

```bash
npm install vite-plugin-pwa -D
```

```typescript
// vite.config.ts
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      registerType: 'autoUpdate',
      includeAssets: ['favicon.ico', 'robots.txt', 'icons/*.png'],
      manifest: {
        name: 'My App',
        short_name: 'App',
        theme_color: '#3b82f6',
        icons: [
          { src: '/icons/icon-192.png', sizes: '192x192', type: 'image/png' },
          { src: '/icons/icon-512.png', sizes: '512x512', type: 'image/png' },
        ],
      },
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}'],
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/firestore\.googleapis\.com\/.*/i,
            handler: 'NetworkFirst',
            options: {
              cacheName: 'firebase-cache',
              expiration: { maxEntries: 100, maxAgeSeconds: 60 * 60 * 24 },
            },
          },
          {
            urlPattern: /^https:\/\/.*\.(?:png|jpg|jpeg|svg|gif|webp)$/i,
            handler: 'CacheFirst',
            options: {
              cacheName: 'image-cache',
              expiration: { maxEntries: 50, maxAgeSeconds: 60 * 60 * 24 * 30 },
            },
          },
        ],
      },
    }),
  ],
});
```

---

## 📴 Offline Detection

```typescript
// hooks/useOnlineStatus.ts
export const useOnlineStatus = () => {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
};

// components/OfflineBanner.tsx
export const OfflineBanner: React.FC = () => {
  const isOnline = useOnlineStatus();

  if (isOnline) return null;

  return (
    <div className="fixed bottom-0 left-0 right-0 bg-yellow-500 text-yellow-900 py-2 px-4 text-center z-50">
      You're offline. Some features may be unavailable.
    </div>
  );
};
```

---

## 📲 Install Prompt

```typescript
// hooks/useInstallPrompt.ts
export const useInstallPrompt = () => {
  const [deferredPrompt, setDeferredPrompt] = useState<any>(null);
  const [canInstall, setCanInstall] = useState(false);

  useEffect(() => {
    const handler = (e: Event) => {
      e.preventDefault();
      setDeferredPrompt(e);
      setCanInstall(true);
    };

    window.addEventListener('beforeinstallprompt', handler);
    return () => window.removeEventListener('beforeinstallprompt', handler);
  }, []);

  const install = async () => {
    if (!deferredPrompt) return;
    
    deferredPrompt.prompt();
    const { outcome } = await deferredPrompt.userChoice;
    
    if (outcome === 'accepted') {
      setCanInstall(false);
    }
    setDeferredPrompt(null);
  };

  return { canInstall, install };
};

// components/InstallButton.tsx
export const InstallButton: React.FC = () => {
  const { canInstall, install } = useInstallPrompt();

  if (!canInstall) return null;

  return (
    <button
      onClick={install}
      className="fixed bottom-4 right-4 bg-primary-600 text-white px-4 py-2 rounded-lg shadow-lg"
    >
      📲 Install App
    </button>
  );
};
```

---

## 👆 Touch-Friendly Design

```css
/* Touch targets */
button, a, input[type="checkbox"], input[type="radio"] {
  min-height: 44px;
  min-width: 44px;
}

/* Prevent zoom on input focus (iOS) */
input, select, textarea {
  font-size: 16px;
}

/* Safe areas for notched phones */
.container {
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
  padding-bottom: env(safe-area-inset-bottom);
}

/* Disable pull-to-refresh where not wanted */
.no-pull-refresh {
  overscroll-behavior-y: contain;
}
```

---

## ✅ PWA Checklist

```
Manifest
☐ manifest.json created
☐ Icons (192x192 and 512x512)
☐ Theme color set
☐ Start URL configured

Service Worker
☐ Registered and active
☐ Caching strategy defined
☐ Offline fallback page
☐ Updates handled gracefully

Mobile UX
☐ Touch targets 44x44px min
☐ No horizontal scroll
☐ Fast tap response
☐ Safe area padding

Performance
☐ First paint < 1.5s
☐ Time to interactive < 3s
☐ Works on 3G networks

Lighthouse PWA Score
☐ Installable ✓
☐ Optimized ✓
☐ PWA badge earned

IF ANY ☐ UNCHECKED → Fix before launching
```

---

**Remember: "PWAs are websites that feel like native apps."** 📱✨

