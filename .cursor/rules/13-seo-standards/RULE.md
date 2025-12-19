---
description: "SEO optimization, meta tags, structured data, and search engine visibility standards"
alwaysApply: true
---

# SEO & Metadata Standards

## 🎯 Core Principles

**"Be discoverable. Be indexable. Be rankable."**

SEO checklist:
- ✅ Semantic HTML structure
- ✅ Proper meta tags on every page
- ✅ Fast page load times
- ✅ Mobile-friendly design
- ✅ Accessible to all users
- ✅ Quality, unique content

---

## 📄 Meta Tags (Every Page)

### HTML Head Template

```typescript
// components/SEO.tsx

interface SEOProps {
  title: string;
  description: string;
  image?: string;
  url?: string;
  type?: 'website' | 'article' | 'product';
  noindex?: boolean;
}

export const SEO: React.FC<SEOProps> = ({
  title,
  description,
  image = '/og-image.jpg',
  url,
  type = 'website',
  noindex = false,
}) => {
  const siteUrl = import.meta.env.VITE_SITE_URL;
  const fullUrl = url ? `${siteUrl}${url}` : siteUrl;
  const fullImage = image.startsWith('http') ? image : `${siteUrl}${image}`;

  return (
    <Helmet>
      {/* Primary Meta Tags */}
      <title>{title} | Your Site Name</title>
      <meta name="title" content={title} />
      <meta name="description" content={description} />
      
      {/* Open Graph / Facebook */}
      <meta property="og:type" content={type} />
      <meta property="og:url" content={fullUrl} />
      <meta property="og:title" content={title} />
      <meta property="og:description" content={description} />
      <meta property="og:image" content={fullImage} />
      <meta property="og:site_name" content="Your Site Name" />
      
      {/* Twitter */}
      <meta property="twitter:card" content="summary_large_image" />
      <meta property="twitter:url" content={fullUrl} />
      <meta property="twitter:title" content={title} />
      <meta property="twitter:description" content={description} />
      <meta property="twitter:image" content={fullImage} />
      <meta property="twitter:creator" content="@yourusername" />
      
      {/* Canonical URL */}
      <link rel="canonical" href={fullUrl} />
      
      {/* Robots */}
      {noindex && <meta name="robots" content="noindex,nofollow" />}
    </Helmet>
  );
};
```

### Usage

```typescript
// pages/HomePage.tsx
function HomePage() {
  return (
    <>
      <SEO
        title="Welcome to Your Site"
        description="Your site helps users do amazing things. Start today!"
        url="/"
      />
      <HomeContent />
    </>
  );
}

// pages/ProductPage.tsx
function ProductPage({ product }: { product: Product }) {
  return (
    <>
      <SEO
        title={product.name}
        description={product.description}
        image={product.images[0]}
        url={`/products/${product.slug}`}
        type="product"
      />
      <ProductDetails product={product} />
    </>
  );
}
```

---

## 🏷️ Structured Data (JSON-LD)

### Organization Schema

```typescript
// components/StructuredData.tsx

export const OrganizationSchema = () => {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: 'Your Company Name',
    url: 'https://yoursite.com',
    logo: 'https://yoursite.com/logo.png',
    description: 'Your company description',
    contactPoint: {
      '@type': 'ContactPoint',
      telephone: '+1-555-555-5555',
      contactType: 'customer service',
      email: 'support@yoursite.com',
      availableLanguage: ['English', 'Spanish'],
    },
    sameAs: [
      'https://facebook.com/yourpage',
      'https://twitter.com/yourhandle',
      'https://linkedin.com/company/yourcompany',
    ],
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
};
```

### Product Schema

```typescript
interface ProductSchemaProps {
  product: {
    name: string;
    description: string;
    image: string;
    price: number;
    currency: string;
    availability: 'InStock' | 'OutOfStock';
    rating?: number;
    reviewCount?: number;
  };
}

export const ProductSchema: React.FC<ProductSchemaProps> = ({ product }) => {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    description: product.description,
    image: product.image,
    offers: {
      '@type': 'Offer',
      price: product.price,
      priceCurrency: product.currency,
      availability: `https://schema.org/${product.availability}`,
      url: window.location.href,
    },
    ...(product.rating && {
      aggregateRating: {
        '@type': 'AggregateRating',
        ratingValue: product.rating,
        reviewCount: product.reviewCount,
      },
    }),
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
};
```

### Article/Blog Schema

```typescript
export const ArticleSchema: React.FC<{
  article: {
    title: string;
    description: string;
    image: string;
    author: string;
    publishDate: string;
    modifyDate?: string;
  };
}> = ({ article }) => {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: article.title,
    description: article.description,
    image: article.image,
    author: {
      '@type': 'Person',
      name: article.author,
    },
    publisher: {
      '@type': 'Organization',
      name: 'Your Site Name',
      logo: {
        '@type': 'ImageObject',
        url: 'https://yoursite.com/logo.png',
      },
    },
    datePublished: article.publishDate,
    dateModified: article.modifyDate || article.publishDate,
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
};
```

### Local Business Schema

```typescript
export const LocalBusinessSchema = () => {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'LocalBusiness',
    name: 'Your Business Name',
    image: 'https://yoursite.com/storefront.jpg',
    '@id': 'https://yoursite.com',
    url: 'https://yoursite.com',
    telephone: '+1-555-555-5555',
    priceRange: '$$',
    address: {
      '@type': 'PostalAddress',
      streetAddress: '123 Main St',
      addressLocality: 'Your City',
      addressRegion: 'CA',
      postalCode: '12345',
      addressCountry: 'US',
    },
    geo: {
      '@type': 'GeoCoordinates',
      latitude: 37.7749,
      longitude: -122.4194,
    },
    openingHoursSpecification: [
      {
        '@type': 'OpeningHoursSpecification',
        dayOfWeek: ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'],
        opens: '09:00',
        closes: '17:00',
      },
    ],
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
};
```

---

## 🗺️ Sitemap Generation

### Dynamic Sitemap

```typescript
// scripts/generate-sitemap.ts

import { writeFileSync } from 'fs';
import { db } from './firebase-admin';

async function generateSitemap() {
  const baseUrl = 'https://yoursite.com';
  const staticPages = [
    { url: '/', priority: 1.0, changefreq: 'daily' },
    { url: '/about', priority: 0.8, changefreq: 'monthly' },
    { url: '/contact', priority: 0.8, changefreq: 'monthly' },
    { url: '/blog', priority: 0.9, changefreq: 'daily' },
  ];

  // Fetch dynamic pages from Firestore
  const products = await db.collection('products').where('status', '==', 'published').get();
  const productPages = products.docs.map(doc => ({
    url: `/products/${doc.data().slug}`,
    priority: 0.7,
    changefreq: 'weekly',
    lastmod: doc.data().updatedAt.toDate().toISOString(),
  }));

  const blogPosts = await db.collection('posts').where('published', '==', true).get();
  const blogPages = blogPosts.docs.map(doc => ({
    url: `/blog/${doc.data().slug}`,
    priority: 0.6,
    changefreq: 'monthly',
    lastmod: doc.data().updatedAt.toDate().toISOString(),
  }));

  const allPages = [...staticPages, ...productPages, ...blogPages];

  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${allPages
  .map(
    (page) => `  <url>
    <loc>${baseUrl}${page.url}</loc>
    <changefreq>${page.changefreq}</changefreq>
    <priority>${page.priority}</priority>
    ${page.lastmod ? `<lastmod>${page.lastmod}</lastmod>` : ''}
  </url>`
  )
  .join('\n')}
</urlset>`;

  writeFileSync('public/sitemap.xml', sitemap);
  console.log('✅ Sitemap generated');
}

generateSitemap();
```

### Add to package.json

```json
{
  "scripts": {
    "generate-sitemap": "tsx scripts/generate-sitemap.ts",
    "prebuild": "npm run generate-sitemap"
  }
}
```

---

## 🤖 Robots.txt

```
# public/robots.txt

User-agent: *
Allow: /
Disallow: /admin/
Disallow: /api/
Disallow: /dashboard/
Disallow: /*.json$
Disallow: /*?*

# Allow specific bots
User-agent: Googlebot
Allow: /

# Sitemap
Sitemap: https://yoursite.com/sitemap.xml

# Crawl delay (optional)
Crawl-delay: 10
```

---

## 📱 Mobile SEO

### Viewport Meta Tag

```html
<!-- index.html -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

### Mobile-Friendly Test

```typescript
// Ensure all interactive elements are touch-friendly
const buttonStyles = {
  minHeight: '44px', // Minimum touch target size
  minWidth: '44px',
  padding: '12px 24px',
};
```

---

## 🌐 International SEO (Hreflang)

```typescript
// For multi-language sites
export const HreflangTags: React.FC<{ slug: string }> = ({ slug }) => {
  return (
    <Helmet>
      <link rel="alternate" hrefLang="en" href={`https://yoursite.com/en/${slug}`} />
      <link rel="alternate" hrefLang="es" href={`https://yoursite.com/es/${slug}`} />
      <link rel="alternate" hrefLang="fr" href={`https://yoursite.com/fr/${slug}`} />
      <link rel="alternate" hrefLang="x-default" href={`https://yoursite.com/en/${slug}`} />
    </Helmet>
  );
};
```

---

## 🔗 URL Structure

### SEO-Friendly URLs

```typescript
// ✅ GOOD - Descriptive, readable URLs
/products/yoga-mat-blue
/blog/beginners-guide-to-meditation
/services/personal-training

// ❌ BAD - Unclear, unfriendly URLs
/products/12345
/blog/post?id=abc123
/services/srv1
```

### Slug Generation

```typescript
export function generateSlug(title: string): string {
  return title
    .toLowerCase()
    .replace(/[^\w\s-]/g, '') // Remove special chars
    .replace(/\s+/g, '-') // Replace spaces with hyphens
    .replace(/-+/g, '-') // Replace multiple hyphens with single
    .trim();
}

// Usage
const product = {
  name: 'Premium Yoga Mat - Blue',
  slug: generateSlug('Premium Yoga Mat - Blue'), // 'premium-yoga-mat-blue'
};
```

---

## 📊 Google Analytics & Search Console

### Google Analytics 4

```typescript
// utils/analytics.ts

export const pageview = (url: string) => {
  if (window.gtag) {
    window.gtag('config', 'G-XXXXXXXXXX', {
      page_path: url,
    });
  }
};

export const event = (action: string, params?: any) => {
  if (window.gtag) {
    window.gtag('event', action, params);
  }
};

// Track route changes
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';

export function usePageTracking() {
  const location = useLocation();

  useEffect(() => {
    pageview(location.pathname + location.search);
  }, [location]);
}
```

### Search Console Verification

```html
<!-- index.html -->
<meta name="google-site-verification" content="your-verification-code" />
```

---

## 🏗️ Semantic HTML

### Proper Structure

```typescript
// ✅ GOOD - Semantic HTML
<header>
  <nav aria-label="Main navigation">
    <ul>
      <li><a href="/">Home</a></li>
      <li><a href="/about">About</a></li>
    </ul>
  </nav>
</header>

<main>
  <article>
    <header>
      <h1>Article Title</h1>
      <time dateTime="2025-01-15">January 15, 2025</time>
    </header>
    <section>
      <h2>Section Title</h2>
      <p>Content here...</p>
    </section>
  </article>
  
  <aside>
    <h2>Related Articles</h2>
    <ul>...</ul>
  </aside>
</main>

<footer>
  <p>&copy; 2025 Your Company</p>
</footer>

// ❌ BAD - Div soup
<div class="header">
  <div class="nav">
    <div class="nav-item">Home</div>
  </div>
</div>
<div class="content">...</div>
<div class="footer">...</div>
```

### Heading Hierarchy

```typescript
// ✅ GOOD - Logical heading structure
<h1>Page Title</h1>
  <h2>Section 1</h2>
    <h3>Subsection 1.1</h3>
    <h3>Subsection 1.2</h3>
  <h2>Section 2</h2>
    <h3>Subsection 2.1</h3>

// ❌ BAD - Skipping levels, multiple h1s
<h1>Page Title</h1>
<h3>Section</h3>
<h1>Another Title</h1>
```

---

## 🖼️ Image SEO

### Alt Text

```typescript
// ✅ GOOD - Descriptive alt text
<img 
  src="/yoga-mat.jpg" 
  alt="Blue yoga mat rolled out on wooden floor with water bottle beside it"
  width="800"
  height="600"
/>

// ❌ BAD - Generic or missing alt
<img src="/img1.jpg" alt="image" />
<img src="/product.jpg" /> {/* No alt attribute */}
```

### Image File Names

```typescript
// ✅ GOOD - Descriptive file names
yoga-mat-blue-premium.jpg
guide-beginners-meditation.jpg

// ❌ BAD - Generic file names
IMG_1234.jpg
photo.jpg
```

---

## 📈 Content Optimization

### Title Tag Guidelines

```
✅ GOOD
- Length: 50-60 characters
- Include primary keyword
- Unique for each page
- Descriptive and compelling

Example: "Premium Yoga Mats | Non-Slip, Eco-Friendly | YourBrand"

❌ BAD
- "Home" (too short, not descriptive)
- "Buy the Best Premium Yoga Mats for Yoga Online at Great Prices" (too long)
- Same title on multiple pages
```

### Meta Description Guidelines

```
✅ GOOD
- Length: 150-160 characters
- Include primary and secondary keywords
- Compelling call-to-action
- Unique for each page

Example: "Shop our premium non-slip yoga mats. Made from eco-friendly materials. Free shipping over $50. Start your practice today!"

❌ BAD
- Too short: "Yoga mats"
- Too long: "We sell yoga mats that are really good and... [300 characters]"
- Generic: "Welcome to our website where we sell products"
```

---

## ⚡ Core Web Vitals (SEO Factor)

```
Performance is a ranking factor. Ensure:
✅ LCP < 2.5s (Largest Contentful Paint)
✅ FID < 100ms (First Input Delay)
✅ CLS < 0.1 (Cumulative Layout Shift)

See Rule 12 (Performance) for optimization techniques.
```

---

## ✅ SEO Checklist

```
Technical SEO
☐ Sitemap.xml generated and submitted
☐ Robots.txt configured
☐ SSL certificate (HTTPS)
☐ Mobile-friendly (responsive design)
☐ Fast page load times (< 3s)
☐ No broken links (404 errors)
☐ Canonical URLs set
☐ Proper URL structure (no parameters)

On-Page SEO
☐ Unique title tag on every page
☐ Unique meta description on every page
☐ Proper heading hierarchy (H1-H6)
☐ Alt text on all images
☐ Descriptive image file names
☐ Internal linking strategy
☐ Keyword optimization (not stuffing)
☐ Quality, original content

Schema Markup
☐ Organization schema
☐ Product schema (if applicable)
☐ Article schema (for blog posts)
☐ Local business schema (if applicable)
☐ Breadcrumb schema
☐ FAQ schema (if applicable)

Social Media
☐ Open Graph tags
☐ Twitter Card tags
☐ Social sharing images (1200x630px)
☐ Social media profiles linked

Analytics & Monitoring
☐ Google Analytics installed
☐ Google Search Console verified
☐ Tracking page views
☐ Tracking conversions/goals
☐ Monitoring search rankings
☐ Monitoring Core Web Vitals

Content
☐ Unique content on every page
☐ Proper keyword research
☐ Content addresses user intent
☐ Regular content updates
☐ No duplicate content
☐ Content is valuable and engaging

IF ANY ☐ UNCHECKED → SEO improvements needed
```

---

## 🚨 AI Decision Gate

```
┌─────────────────────────────────────────────┐
│  SEO CHECKLIST                              │
├─────────────────────────────────────────────┤
│  Before creating any page:                  │
│  ☐ SEO component added with meta tags      │
│  ☐ Title is unique and descriptive         │
│  ☐ Meta description is compelling          │
│  ☐ URL is SEO-friendly                     │
│  ☐ Proper heading hierarchy (H1-H6)        │
│                                             │
│  Before adding images:                      │
│  ☐ Descriptive file name                   │
│  ☐ Meaningful alt text                     │
│  ☐ Proper dimensions specified             │
│  ☐ Optimized file size                     │
│                                             │
│  Before launch:                             │
│  ☐ Sitemap generated                       │
│  ☐ Robots.txt configured                   │
│  ☐ Analytics installed                     │
│  ☐ Search Console verified                 │
│  ☐ All pages have meta tags                │
│  ☐ Schema markup added                     │
│                                             │
│  IF ANY ☐ UNCHECKED:                        │
│  ➜ ADDRESS SEO ISSUES                      │
└─────────────────────────────────────────────┘
```

---

**Remember: "SEO is not a one-time task. It's an ongoing process of optimization and content creation."** 🔍✨

