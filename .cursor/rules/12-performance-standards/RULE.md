---
description: "Performance optimization strategies for fast, efficient React+Firebase applications"
alwaysApply: true
---

# Performance Optimization Standards

## 🎯 Core Principles

**"Fast is better than feature-rich. Optimize for perceived performance."**

Performance targets:
- ✅ First Contentful Paint (FCP): < 1.8s
- ✅ Largest Contentful Paint (LCP): < 2.5s
- ✅ Time to Interactive (TTI): < 3.8s
- ✅ Cumulative Layout Shift (CLS): < 0.1
- ✅ First Input Delay (FID): < 100ms

---

## 📦 Bundle Size Optimization

### Code Splitting

```typescript
// ✅ GOOD - Lazy load routes
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./features/dashboard/Dashboard'));
const Profile = lazy(() => import('./features/profile/Profile'));
const Admin = lazy(() => import('./features/admin/Admin'));

function App() {
  return (
    <Suspense fallback={<LoadingScreen />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<Profile />} />
        <Route path="/admin" element={<Admin />} />
      </Routes>
    </Suspense>
  );
}

// ❌ BAD - Import everything upfront
import Dashboard from './features/dashboard/Dashboard';
import Profile from './features/profile/Profile';
import Admin from './features/admin/Admin';
```

### Dynamic Imports

```typescript
// ✅ GOOD - Load heavy libraries only when needed
const loadChartLibrary = async () => {
  const { Chart } = await import('chart.js');
  return Chart;
};

function AnalyticsPage() {
  const [Chart, setChart] = useState(null);

  useEffect(() => {
    loadChartLibrary().then(setChart);
  }, []);

  if (!Chart) return <LoadingSpinner />;
  
  return <ChartComponent Chart={Chart} />;
}

// ❌ BAD - Import 500kb library even if user never visits analytics
import { Chart } from 'chart.js';
```

### Tree Shaking

```typescript
// ✅ GOOD - Import only what you need
import { format } from 'date-fns';
import { debounce } from 'lodash-es';

// ❌ BAD - Imports entire library
import _ from 'lodash';
import * as dateFns from 'date-fns';
```

### Bundle Analysis

```bash
# Add to package.json
"scripts": {
  "build": "vite build",
  "analyze": "vite-bundle-visualizer"
}

# Run analysis
npm run analyze
```

**Bundle Size Limits:**
- Initial bundle: < 200kb (gzipped)
- Route chunks: < 100kb each
- Vendor chunk: < 150kb

---

## 🖼️ Image Optimization

### Modern Formats

```typescript
// ✅ GOOD - Use modern formats with fallbacks
<picture>
  <source srcSet="/images/hero.avif" type="image/avif" />
  <source srcSet="/images/hero.webp" type="image/webp" />
  <img src="/images/hero.jpg" alt="Hero image" loading="lazy" />
</picture>

// Use automatic optimization service
const optimizeImage = (url: string, width: number) => {
  // Using Cloudinary, Imgix, or similar
  return `${url}?w=${width}&f=auto&q=auto`;
};
```

### Lazy Loading

```typescript
// ✅ GOOD - Lazy load images below the fold
<img 
  src="/image.jpg" 
  loading="lazy" 
  alt="Description"
  width="800"
  height="600"
/>

// Component for advanced lazy loading
import { LazyLoadImage } from 'react-lazy-load-image-component';

<LazyLoadImage
  src="/image.jpg"
  alt="Description"
  effect="blur"
  placeholderSrc="/image-tiny.jpg"
/>
```

### Responsive Images

```typescript
// ✅ GOOD - Serve appropriately sized images
<img
  src="/image-800.jpg"
  srcSet="
    /image-400.jpg 400w,
    /image-800.jpg 800w,
    /image-1200.jpg 1200w
  "
  sizes="(max-width: 600px) 400px, (max-width: 1200px) 800px, 1200px"
  alt="Description"
/>
```

### Image Compression

```bash
# Optimize images before deployment
npm install -D imagemin imagemin-mozjpeg imagemin-pngquant imagemin-svgo

# optimize-images.js
const imagemin = require('imagemin');
const imageminMozjpeg = require('imagemin-mozjpeg');
const imageminPngquant = require('imagemin-pngquant');

(async () => {
  await imagemin(['public/images/*.{jpg,png}'], {
    destination: 'public/images/optimized',
    plugins: [
      imageminMozjpeg({ quality: 80 }),
      imageminPngquant({ quality: [0.6, 0.8] })
    ]
  });
})();
```

---

## ⚛️ React Performance

### Memoization

```typescript
// ✅ GOOD - Memoize expensive computations
const sortedUsers = useMemo(() => {
  return users.sort((a, b) => a.name.localeCompare(b.name));
}, [users]);

// ✅ GOOD - Memoize callbacks passed to children
const handleUserClick = useCallback((userId: string) => {
  navigate(`/users/${userId}`);
}, [navigate]);

// ✅ GOOD - Memoize components with React.memo
const UserCard = React.memo(({ user }: { user: User }) => {
  return <div>{user.name}</div>;
}, (prevProps, nextProps) => {
  // Custom comparison - only re-render if user.id changed
  return prevProps.user.id === nextProps.user.id;
});

// ❌ BAD - Unnecessary memoization
const fullName = useMemo(() => 
  `${firstName} ${lastName}`, 
  [firstName, lastName]
); // Simple concatenation doesn't need memo

// ❌ BAD - Creating new function on every render
<Button onClick={() => handleClick(user.id)} />

// ✅ GOOD - Memoized callback
const onClick = useCallback(() => handleClick(user.id), [user.id]);
<Button onClick={onClick} />
```

### Virtual Lists

```typescript
// ✅ GOOD - Virtualize long lists
import { FixedSizeList } from 'react-window';

function UserList({ users }: { users: User[] }) {
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => (
    <div style={style}>
      <UserCard user={users[index]} />
    </div>
  );

  return (
    <FixedSizeList
      height={600}
      itemCount={users.length}
      itemSize={80}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}

// ❌ BAD - Render 10,000 items at once
{users.map(user => <UserCard key={user.id} user={user} />)}
```

### Debouncing & Throttling

```typescript
// ✅ GOOD - Debounce search input
import { useDebouncedValue } from '@/hooks/useDebouncedValue';

function SearchBar() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebouncedValue(searchTerm, 300);

  useEffect(() => {
    if (debouncedSearchTerm) {
      performSearch(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);

  return (
    <input
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
    />
  );
}

// hooks/useDebouncedValue.ts
export function useDebouncedValue<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

// ✅ GOOD - Throttle scroll events
import { useThrottle } from '@/hooks/useThrottle';

function InfiniteScroll() {
  const handleScroll = useThrottle(() => {
    // Load more items
  }, 200);

  return <div onScroll={handleScroll}>...</div>;
}
```

---

## 🔥 Firebase Performance

### Efficient Queries

```typescript
// ✅ GOOD - Query only what you need
const q = query(
  collection(db, 'bookings'),
  where('userId', '==', userId),
  where('status', '==', 'active'),
  orderBy('date', 'desc'),
  limit(20) // Limit results!
);

// ❌ BAD - Fetch everything then filter in JavaScript
const allBookings = await getDocs(collection(db, 'bookings'));
const userBookings = allBookings.docs
  .filter(doc => doc.data().userId === userId)
  .filter(doc => doc.data().status === 'active');
```

### Batch Operations

```typescript
// ✅ GOOD - Batch writes
const batch = writeBatch(db);
users.forEach(user => {
  const docRef = doc(db, 'users', user.id);
  batch.set(docRef, user);
});
await batch.commit(); // Single network call

// ❌ BAD - Individual writes
for (const user of users) {
  await setDoc(doc(db, 'users', user.id), user); // Multiple network calls
}
```

### Pagination

```typescript
// ✅ GOOD - Paginated queries
async function loadNextPage(lastVisible: QueryDocumentSnapshot | null) {
  let q = query(
    collection(db, 'products'),
    orderBy('createdAt', 'desc'),
    limit(20)
  );

  if (lastVisible) {
    q = query(q, startAfter(lastVisible));
  }

  const snapshot = await getDocs(q);
  const lastDoc = snapshot.docs[snapshot.docs.length - 1];
  
  return {
    products: snapshot.docs.map(doc => doc.data()),
    lastVisible: lastDoc,
  };
}
```

### Caching Strategy

```typescript
// ✅ GOOD - Enable Firestore cache
import { enableIndexedDbPersistence } from 'firebase/firestore';

enableIndexedDbPersistence(db).catch((err) => {
  if (err.code === 'failed-precondition') {
    // Multiple tabs open
  } else if (err.code === 'unimplemented') {
    // Browser doesn't support
  }
});

// React Query for caching
import { useQuery } from '@tanstack/react-query';

function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => userService.getById(userId),
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
  });
}
```

---

## 🌐 Network Optimization

### Service Worker

```typescript
// vite.config.ts
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg}'],
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/fonts\.googleapis\.com\/.*/i,
            handler: 'CacheFirst',
            options: {
              cacheName: 'google-fonts-cache',
              expiration: {
                maxEntries: 10,
                maxAgeSeconds: 60 * 60 * 24 * 365, // 1 year
              },
            },
          },
          {
            urlPattern: /^https:\/\/firebasestorage\.googleapis\.com\/.*/i,
            handler: 'CacheFirst',
            options: {
              cacheName: 'firebase-storage-cache',
              expiration: {
                maxEntries: 50,
                maxAgeSeconds: 60 * 60 * 24 * 7, // 1 week
              },
            },
          },
        ],
      },
    }),
  ],
});
```

### Prefetching

```typescript
// ✅ GOOD - Prefetch next page on hover
function ProductCard({ product }: { product: Product }) {
  const prefetchProduct = () => {
    queryClient.prefetchQuery({
      queryKey: ['product', product.id],
      queryFn: () => productService.getById(product.id),
    });
  };

  return (
    <Link
      to={`/products/${product.id}`}
      onMouseEnter={prefetchProduct}
    >
      {product.name}
    </Link>
  );
}
```

### Resource Hints

```html
<!-- index.html -->
<head>
  <!-- Preconnect to required origins -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://firebasestorage.googleapis.com">
  
  <!-- DNS prefetch for external resources -->
  <link rel="dns-prefetch" href="https://www.google-analytics.com">
  
  <!-- Preload critical assets -->
  <link rel="preload" href="/fonts/main-font.woff2" as="font" type="font/woff2" crossorigin>
  <link rel="preload" href="/critical.css" as="style">
</head>
```

---

## 🎨 CSS Performance

### Critical CSS

```typescript
// Extract and inline critical CSS
import { extractCritical } from '@emotion/server';

const { html, css } = extractCritical(appHtml);

// Inline in HTML
<head>
  <style dangerouslySetInnerHTML={{ __html: css }} />
</head>
```

### CSS-in-JS Performance

```typescript
// ✅ GOOD - Use CSS Modules or Tailwind
import styles from './Button.module.css';

function Button() {
  return <button className={styles.button}>Click</button>;
}

// Or Tailwind (optimized at build time)
function Button() {
  return <button className="px-4 py-2 bg-blue-500 hover:bg-blue-600">
    Click
  </button>;
}

// ⚠️ CAUTION - Runtime CSS-in-JS has performance cost
import styled from 'styled-components';
const StyledButton = styled.button`
  padding: 1rem 2rem;
  background: blue;
`; // Creates new styles on every render
```

### Avoid Layout Thrashing

```typescript
// ❌ BAD - Causes layout thrashing
elements.forEach(el => {
  const height = el.offsetHeight; // Read
  el.style.height = height + 10 + 'px'; // Write
});

// ✅ GOOD - Batch reads, then writes
const heights = elements.map(el => el.offsetHeight); // All reads
elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + 'px'; // All writes
});
```

---

## 📊 Performance Monitoring

### Web Vitals

```typescript
// reportWebVitals.ts
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

function sendToAnalytics(metric: Metric) {
  // Send to Google Analytics, Sentry, or custom endpoint
  if (window.gtag) {
    window.gtag('event', metric.name, {
      value: Math.round(metric.value),
      event_category: 'Web Vitals',
      non_interaction: true,
    });
  }
}

export function reportWebVitals() {
  getCLS(sendToAnalytics);
  getFID(sendToAnalytics);
  getFCP(sendToAnalytics);
  getLCP(sendToAnalytics);
  getTTFB(sendToAnalytics);
}

// main.tsx
reportWebVitals();
```

### Performance Profiler

```typescript
// Use React Profiler to measure render performance
import { Profiler } from 'react';

function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
) {
  console.log(`${id} (${phase}) took ${actualDuration}ms`);
  
  if (actualDuration > 16) {
    console.warn(`Slow render detected in ${id}`);
  }
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <YourApp />
    </Profiler>
  );
}
```

### Custom Performance Marks

```typescript
// Measure custom metrics
export function measurePerformance(name: string, fn: () => void) {
  performance.mark(`${name}-start`);
  fn();
  performance.mark(`${name}-end`);
  performance.measure(name, `${name}-start`, `${name}-end`);
  
  const measure = performance.getEntriesByName(name)[0];
  console.log(`${name}: ${measure.duration}ms`);
}

// Usage
measurePerformance('data-processing', () => {
  // Your expensive operation
});
```

---

## ⚡ Loading Strategies

### Skeleton Screens

```typescript
// ✅ GOOD - Show skeleton while loading
function ProductPage() {
  const { data: product, loading } = useProduct(productId);

  if (loading) {
    return <ProductSkeleton />;
  }

  return <ProductDetails product={product} />;
}

function ProductSkeleton() {
  return (
    <div className="animate-pulse">
      <div className="h-64 bg-gray-200 rounded" />
      <div className="mt-4 h-8 bg-gray-200 rounded w-3/4" />
      <div className="mt-2 h-4 bg-gray-200 rounded w-1/2" />
    </div>
  );
}

// ❌ BAD - Blank screen while loading
if (loading) return null;
```

### Progressive Enhancement

```typescript
// ✅ GOOD - Load critical content first
function Dashboard() {
  return (
    <>
      {/* Load immediately */}
      <DashboardHeader />
      
      {/* Load with slight delay - not critical */}
      <Suspense fallback={<SkeletonStats />}>
        <DashboardStats />
      </Suspense>
      
      {/* Load last - below the fold */}
      <Suspense fallback={<SkeletonChart />}>
        <DashboardChart />
      </Suspense>
    </>
  );
}
```

---

## 🧪 Performance Testing

### Lighthouse CI

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI
on: [push]
jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run Lighthouse
        uses: treosh/lighthouse-ci-action@v9
        with:
          urls: |
            https://your-site.com
            https://your-site.com/dashboard
          uploadArtifacts: true
          temporaryPublicStorage: true
```

### Bundle Size Monitoring

```json
// package.json
{
  "scripts": {
    "build": "vite build",
    "size": "size-limit"
  },
  "size-limit": [
    {
      "path": "dist/assets/index-*.js",
      "limit": "200 KB"
    },
    {
      "path": "dist/assets/vendor-*.js",
      "limit": "150 KB"
    }
  ]
}
```

---

## ✅ Performance Checklist

```
Bundle Size
☐ Initial bundle < 200KB (gzipped)
☐ Code splitting implemented for routes
☐ Tree shaking configured
☐ Lazy loading for heavy components
☐ Bundle analysis run and reviewed

Images
☐ All images optimized (< 100KB each)
☐ Modern formats used (WebP/AVIF)
☐ Lazy loading below the fold
☐ Responsive images with srcset
☐ Proper width/height attributes

React Performance
☐ useMemo for expensive calculations
☐ useCallback for event handlers
☐ React.memo for pure components
☐ Virtual scrolling for long lists
☐ No unnecessary re-renders

Firebase
☐ Queries use limit()
☐ Indexes created for all queries
☐ Batch operations used
☐ Pagination implemented
☐ Offline persistence enabled

Network
☐ Service worker configured
☐ Caching strategy implemented
☐ Resource hints (preconnect, prefetch)
☐ CDN used for static assets
☐ Compression enabled (Gzip/Brotli)

CSS
☐ Critical CSS inlined
☐ Unused CSS removed
☐ CSS Modules or Tailwind used
☐ No layout thrashing
☐ Animations use transform/opacity only

Monitoring
☐ Web Vitals tracked
☐ Performance monitoring setup
☐ Lighthouse CI configured
☐ Bundle size monitoring
☐ Error tracking includes performance

Loading
☐ Skeleton screens for loading states
☐ Progressive enhancement
☐ No blank screens
☐ Perceived performance optimized

IF ANY ☐ UNCHECKED → Performance improvements needed
```

---

## 🚨 AI Decision Gate

```
┌─────────────────────────────────────────────┐
│  PERFORMANCE CHECKLIST                      │
├─────────────────────────────────────────────┤
│  Before adding dependencies:                │
│  ☐ Check bundle size impact                │
│  ☐ Look for smaller alternatives            │
│  ☐ Consider lazy loading                    │
│  ☐ Verify tree-shaking support              │
│                                             │
│  Before adding images:                      │
│  ☐ Optimize and compress                    │
│  ☐ Use modern formats                       │
│  ☐ Add lazy loading                         │
│  ☐ Include dimensions                       │
│                                             │
│  Before Firebase queries:                   │
│  ☐ Add limit() clause                       │
│  ☐ Create required indexes                  │
│  ☐ Consider pagination                      │
│  ☐ Implement caching                        │
│                                             │
│  Before rendering lists:                    │
│  ☐ Check list size (>100 items?)           │
│  ☐ Implement virtualization if needed       │
│  ☐ Add pagination/infinite scroll           │
│  ☐ Memoize list items                       │
│                                             │
│  IF ANY ☐ UNCHECKED:                        │
│  ➜ ADDRESS PERFORMANCE CONCERNS             │
└─────────────────────────────────────────────┘
```

---

**Remember: "Performance is a feature. Fast apps feel more reliable and professional."** ⚡✨

