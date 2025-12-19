---
description: "Simple gallery and product display components for showcasing images and basic products"
alwaysApply: false
globs: ["**/components/Gallery*", "**/components/Product*", "**/pages/products/**", "**/pages/gallery/**"]
---

# Gallery & Product Display Standards

## 🎯 Core Principles

**"Keep it simple. Beautiful images sell themselves."**

For small catalogs (3-10 products) or image galleries:
- ✅ Clean, visual-first layouts
- ✅ Mobile-responsive grids
- ✅ Touch-friendly interactions
- ✅ Fast loading (optimized images)
- ✅ Accessible to all users

**NOT covered here:** Shopping carts, checkout, inventory (see Rule 15 for payments if needed)

---

## 🖼️ Gallery Components

### Simple Image Gallery

```typescript
// components/Gallery.tsx

interface GalleryImage {
  id: string;
  src: string;
  alt: string;
  title?: string;
  caption?: string;
  thumbnail?: string;
}

interface GalleryProps {
  images: GalleryImage[];
  columns?: number; // 2, 3, or 4
}

export const Gallery: React.FC<GalleryProps> = ({ 
  images, 
  columns = 3 
}) => {
  const [selectedImage, setSelectedImage] = useState<GalleryImage | null>(null);

  return (
    <>
      {/* Gallery Grid */}
      <div className={`grid gap-4 ${
        columns === 2 ? 'grid-cols-1 sm:grid-cols-2' :
        columns === 3 ? 'grid-cols-1 sm:grid-cols-2 lg:grid-cols-3' :
        'grid-cols-1 sm:grid-cols-2 lg:grid-cols-4'
      }`}>
        {images.map((image) => (
          <div
            key={image.id}
            className="group cursor-pointer overflow-hidden rounded-lg"
            onClick={() => setSelectedImage(image)}
          >
            <img
              src={image.thumbnail || image.src}
              alt={image.alt}
              loading="lazy"
              className="w-full h-64 object-cover transition-transform duration-300 group-hover:scale-110"
            />
            {image.title && (
              <div className="mt-2 text-sm font-medium">{image.title}</div>
            )}
          </div>
        ))}
      </div>

      {/* Lightbox Modal */}
      {selectedImage && (
        <Lightbox
          image={selectedImage}
          onClose={() => setSelectedImage(null)}
          onNext={() => {
            const currentIndex = images.findIndex(img => img.id === selectedImage.id);
            const nextIndex = (currentIndex + 1) % images.length;
            setSelectedImage(images[nextIndex]);
          }}
          onPrev={() => {
            const currentIndex = images.findIndex(img => img.id === selectedImage.id);
            const prevIndex = (currentIndex - 1 + images.length) % images.length;
            setSelectedImage(images[prevIndex]);
          }}
        />
      )}
    </>
  );
};
```

### Lightbox Component

```typescript
// components/Lightbox.tsx

interface LightboxProps {
  image: GalleryImage;
  onClose: () => void;
  onNext?: () => void;
  onPrev?: () => void;
}

export const Lightbox: React.FC<LightboxProps> = ({
  image,
  onClose,
  onNext,
  onPrev,
}) => {
  // Close on Escape key
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'Escape') onClose();
      if (e.key === 'ArrowRight' && onNext) onNext();
      if (e.key === 'ArrowLeft' && onPrev) onPrev();
    };
    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [onClose, onNext, onPrev]);

  return (
    <div
      className="fixed inset-0 z-50 bg-black bg-opacity-90 flex items-center justify-center"
      onClick={onClose}
    >
      {/* Close Button */}
      <button
        onClick={onClose}
        className="absolute top-4 right-4 text-white text-3xl hover:text-gray-300 z-10"
        aria-label="Close lightbox"
      >
        ×
      </button>

      {/* Previous Button */}
      {onPrev && (
        <button
          onClick={(e) => { e.stopPropagation(); onPrev(); }}
          className="absolute left-4 text-white text-4xl hover:text-gray-300"
          aria-label="Previous image"
        >
          ‹
        </button>
      )}

      {/* Image */}
      <div
        className="max-w-7xl max-h-[90vh] p-4"
        onClick={(e) => e.stopPropagation()}
      >
        <img
          src={image.src}
          alt={image.alt}
          className="max-w-full max-h-[80vh] object-contain"
        />
        
        {/* Caption */}
        {(image.title || image.caption) && (
          <div className="mt-4 text-center text-white">
            {image.title && (
              <h3 className="text-xl font-semibold">{image.title}</h3>
            )}
            {image.caption && (
              <p className="text-sm text-gray-300 mt-2">{image.caption}</p>
            )}
          </div>
        )}
      </div>

      {/* Next Button */}
      {onNext && (
        <button
          onClick={(e) => { e.stopPropagation(); onNext(); }}
          className="absolute right-4 text-white text-4xl hover:text-gray-300"
          aria-label="Next image"
        >
          ›
        </button>
      )}
    </div>
  );
};
```

---

## 🏷️ Product Display Components

### Product Card (Grid View)

```typescript
// components/ProductCard.tsx

interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  image: string;
  images?: string[]; // Multiple images
  category?: string;
  inStock?: boolean;
}

interface ProductCardProps {
  product: Product;
  onViewDetails?: (product: Product) => void;
}

export const ProductCard: React.FC<ProductCardProps> = ({ 
  product, 
  onViewDetails 
}) => {
  return (
    <div className="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-xl transition-shadow">
      {/* Product Image */}
      <div className="relative aspect-square bg-gray-100">
        <img
          src={product.image}
          alt={product.name}
          loading="lazy"
          className="w-full h-full object-cover"
        />
        
        {/* Stock Badge */}
        {product.inStock === false && (
          <div className="absolute top-2 right-2 bg-red-500 text-white px-3 py-1 rounded-full text-sm">
            Out of Stock
          </div>
        )}

        {/* Category Badge */}
        {product.category && (
          <div className="absolute top-2 left-2 bg-white bg-opacity-90 px-3 py-1 rounded-full text-sm">
            {product.category}
          </div>
        )}
      </div>

      {/* Product Info */}
      <div className="p-4">
        <h3 className="text-lg font-semibold mb-2 line-clamp-2">
          {product.name}
        </h3>
        
        <p className="text-gray-600 text-sm mb-3 line-clamp-2">
          {product.description}
        </p>

        {/* Price */}
        <div className="flex items-center justify-between">
          <span className="text-2xl font-bold text-primary-600">
            ${product.price.toFixed(2)}
          </span>

          {/* View Details Button */}
          <button
            onClick={() => onViewDetails?.(product)}
            className="px-4 py-2 bg-primary-600 text-white rounded-lg hover:bg-primary-700 transition"
          >
            View Details
          </button>
        </div>
      </div>
    </div>
  );
};
```

### Product Grid

```typescript
// components/ProductGrid.tsx

interface ProductGridProps {
  products: Product[];
  columns?: number;
}

export const ProductGrid: React.FC<ProductGridProps> = ({ 
  products, 
  columns = 3 
}) => {
  const navigate = useNavigate();

  return (
    <div className={`grid gap-6 ${
      columns === 2 ? 'grid-cols-1 md:grid-cols-2' :
      columns === 3 ? 'grid-cols-1 sm:grid-cols-2 lg:grid-cols-3' :
      'grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4'
    }`}>
      {products.map((product) => (
        <ProductCard
          key={product.id}
          product={product}
          onViewDetails={(product) => navigate(`/products/${product.id}`)}
        />
      ))}
    </div>
  );
};
```

---

## 📱 Product Detail Page

```typescript
// pages/ProductDetail.tsx

export const ProductDetail: React.FC = () => {
  const { productId } = useParams();
  const [selectedImage, setSelectedImage] = useState(0);
  const [product, setProduct] = useState<Product | null>(null);

  useEffect(() => {
    // Fetch product details
    loadProduct(productId);
  }, [productId]);

  if (!product) return <LoadingSpinner />;

  const images = product.images || [product.image];

  return (
    <div className="max-w-7xl mx-auto px-4 py-8">
      <div className="grid md:grid-cols-2 gap-8">
        
        {/* Image Gallery Side */}
        <div>
          {/* Main Image */}
          <div className="aspect-square bg-gray-100 rounded-lg overflow-hidden mb-4">
            <img
              src={images[selectedImage]}
              alt={`${product.name} - Image ${selectedImage + 1}`}
              className="w-full h-full object-cover"
            />
          </div>

          {/* Thumbnail Navigation */}
          {images.length > 1 && (
            <div className="grid grid-cols-4 gap-2">
              {images.map((image, index) => (
                <button
                  key={index}
                  onClick={() => setSelectedImage(index)}
                  className={`aspect-square rounded-lg overflow-hidden border-2 transition ${
                    selectedImage === index
                      ? 'border-primary-600'
                      : 'border-transparent hover:border-gray-300'
                  }`}
                >
                  <img
                    src={image}
                    alt={`${product.name} thumbnail ${index + 1}`}
                    className="w-full h-full object-cover"
                  />
                </button>
              ))}
            </div>
          )}
        </div>

        {/* Product Info Side */}
        <div>
          {/* Category */}
          {product.category && (
            <div className="text-sm text-gray-600 mb-2">
              {product.category}
            </div>
          )}

          {/* Title */}
          <h1 className="text-3xl font-bold mb-4">{product.name}</h1>

          {/* Price */}
          <div className="text-4xl font-bold text-primary-600 mb-6">
            ${product.price.toFixed(2)}
          </div>

          {/* Stock Status */}
          <div className="mb-6">
            {product.inStock !== false ? (
              <span className="inline-flex items-center text-green-600">
                <span className="w-2 h-2 bg-green-600 rounded-full mr-2"></span>
                In Stock
              </span>
            ) : (
              <span className="inline-flex items-center text-red-600">
                <span className="w-2 h-2 bg-red-600 rounded-full mr-2"></span>
                Out of Stock
              </span>
            )}
          </div>

          {/* Description */}
          <div className="prose mb-8">
            <h2 className="text-xl font-semibold mb-3">Description</h2>
            <p className="text-gray-700">{product.description}</p>
          </div>

          {/* Action Buttons */}
          <div className="space-y-3">
            <button
              className="w-full py-3 bg-primary-600 text-white rounded-lg hover:bg-primary-700 transition font-semibold"
              disabled={product.inStock === false}
            >
              {product.inStock !== false ? 'Contact for Purchase' : 'Out of Stock'}
            </button>

            <button
              className="w-full py-3 border-2 border-primary-600 text-primary-600 rounded-lg hover:bg-primary-50 transition font-semibold"
            >
              Ask a Question
            </button>
          </div>

          {/* Additional Info */}
          <div className="mt-8 pt-8 border-t">
            <h3 className="font-semibold mb-3">Product Information</h3>
            <dl className="space-y-2 text-sm">
              <div className="flex justify-between">
                <dt className="text-gray-600">SKU:</dt>
                <dd className="font-medium">{product.id}</dd>
              </div>
              <div className="flex justify-between">
                <dt className="text-gray-600">Category:</dt>
                <dd className="font-medium">{product.category || 'General'}</dd>
              </div>
            </dl>
          </div>
        </div>
      </div>
    </div>
  );
};
```

---

## 📐 Layout Patterns

### 2-Column Grid (Mobile: 1, Desktop: 2)

```tsx
<div className="grid grid-cols-1 md:grid-cols-2 gap-4">
  {items.map(item => <Card key={item.id} {...item} />)}
</div>
```

### 3-Column Grid (Mobile: 1, Tablet: 2, Desktop: 3)

```tsx
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
  {items.map(item => <Card key={item.id} {...item} />)}
</div>
```

### 4-Column Grid (for many small items)

```tsx
<div className="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 gap-4">
  {items.map(item => <Card key={item.id} {...item} />)}
</div>
```

---

## 🎨 Image Optimization

### Image Sizes for Products/Gallery

```typescript
// Image dimensions for different uses

export const IMAGE_SIZES = {
  thumbnail: { width: 200, height: 200 },
  card: { width: 400, height: 400 },
  detail: { width: 800, height: 800 },
  full: { width: 1200, height: 1200 },
};

// Responsive images
<img
  src={product.image}
  srcSet={`
    ${product.thumbnail} 200w,
    ${product.image} 400w,
    ${product.imageHD} 800w
  `}
  sizes="(max-width: 640px) 200px, (max-width: 1024px) 400px, 800px"
  alt={product.name}
/>
```

### Lazy Loading

```tsx
// ✅ GOOD - Images below fold load lazily
<img src={image.src} alt={image.alt} loading="lazy" />

// ✅ GOOD - First 3 images load immediately
{images.map((img, index) => (
  <img
    key={img.id}
    src={img.src}
    alt={img.alt}
    loading={index < 3 ? 'eager' : 'lazy'}
  />
))}
```

---

## 🎭 Simple Animations

```css
/* Hover zoom on images */
.gallery-item {
  overflow: hidden;
}

.gallery-item img {
  transition: transform 0.3s ease;
}

.gallery-item:hover img {
  transform: scale(1.1);
}

/* Fade in on load */
.product-card {
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
```

---

## ♿ Accessibility

### Gallery Accessibility

```tsx
<div role="group" aria-label="Photo gallery">
  {images.map((image, index) => (
    <button
      key={image.id}
      onClick={() => setSelected(image)}
      aria-label={`View image ${index + 1}: ${image.title}`}
    >
      <img src={image.src} alt={image.alt} />
    </button>
  ))}
</div>
```

### Product Card Accessibility

```tsx
<article aria-label={`Product: ${product.name}`}>
  <img src={product.image} alt={`${product.name} product photo`} />
  <h3>{product.name}</h3>
  <p>{product.description}</p>
  <div aria-label="Price">${product.price}</div>
  <button aria-label={`View details for ${product.name}`}>
    View Details
  </button>
</article>
```

### Keyboard Navigation

```typescript
// Lightbox keyboard controls
useEffect(() => {
  const handleKeyDown = (e: KeyboardEvent) => {
    if (e.key === 'Escape') closeLightbox();
    if (e.key === 'ArrowRight') nextImage();
    if (e.key === 'ArrowLeft') previousImage();
  };
  
  window.addEventListener('keydown', handleKeyDown);
  return () => window.removeEventListener('keydown', handleKeyDown);
}, []);
```

---

## 📱 Mobile-Friendly Touches

### Touch Gestures for Gallery

```typescript
// components/TouchGallery.tsx
import { useSwipeable } from 'react-swipeable';

const handlers = useSwipeable({
  onSwipedLeft: () => nextImage(),
  onSwipedRight: () => previousImage(),
  preventScrollOnSwipe: true,
  trackMouse: true,
});

<div {...handlers}>
  <img src={currentImage.src} alt={currentImage.alt} />
</div>
```

### Mobile Grid Adjustments

```tsx
// Smaller gaps on mobile
<div className="grid grid-cols-2 gap-2 sm:gap-4 lg:gap-6">
  {/* Items */}
</div>

// Larger touch targets on mobile
<button className="p-4 sm:p-3">
  View Details
</button>
```

---

## 🗄️ Firestore Structure for Products

```typescript
// firestore structure
products/
  {productId}/
    name: string
    description: string
    price: number
    images: string[]  // Array of image URLs
    category: string
    inStock: boolean
    featured: boolean
    order: number  // For manual sorting
    createdAt: Timestamp
    updatedAt: Timestamp

// Query for products
const productsQuery = query(
  collection(db, 'products'),
  where('inStock', '==', true),
  orderBy('order', 'asc'),
  limit(10)
);
```

---

## ✅ Checklist

```
Gallery Components
☐ Image grid is responsive (1/2/3 columns)
☐ Images are lazy loaded
☐ Images are optimized (< 100KB)
☐ Lightbox has keyboard navigation
☐ Lightbox has touch/swipe support
☐ Alt text on all images
☐ Loading states shown

Product Display
☐ Product cards show key info (name, price, image)
☐ Products are in responsive grid
☐ "Out of stock" clearly indicated
☐ Product detail page has image gallery
☐ Multiple product images supported
☐ Price is prominently displayed
☐ Contact/inquiry button provided

Mobile Experience
☐ Touch-friendly buttons (min 44x44px)
☐ Grid adjusts for small screens
☐ Images load fast on mobile
☐ Swipe gestures work in gallery
☐ Text is readable (min 16px)

Accessibility
☐ All images have alt text
☐ Keyboard navigation works
☐ Screen reader labels present
☐ Focus indicators visible
☐ ARIA labels where needed

Performance
☐ Images optimized and compressed
☐ Lazy loading implemented
☐ No layout shift on image load
☐ Fast initial page load

IF ANY ☐ UNCHECKED → Fix before launch
```

---

## 🚨 AI Decision Gate

```
┌─────────────────────────────────────────────┐
│  GALLERY/PRODUCT DISPLAY CHECKLIST          │
├─────────────────────────────────────────────┤
│  Before creating gallery:                   │
│  ☐ Images are optimized (< 100KB each)     │
│  ☐ Alt text for all images                 │
│  ☐ Responsive grid layout                  │
│  ☐ Lazy loading enabled                    │
│  ☐ Lightbox has keyboard controls          │
│                                             │
│  Before creating product display:           │
│  ☐ Product has name, price, description    │
│  ☐ Image is optimized                      │
│  ☐ Stock status indicated                  │
│  ☐ Mobile-responsive layout                │
│  ☐ Contact/inquiry method provided         │
│                                             │
│  Before product detail page:                │
│  ☐ Multiple images if available            │
│  ☐ Thumbnail navigation                    │
│  ☐ Clear CTA button                        │
│  ☐ Product info complete                   │
│  ☐ Accessible to screen readers            │
│                                             │
│  IF ANY ☐ UNCHECKED:                        │
│  ➜ ADDRESS BEFORE IMPLEMENTING              │
└─────────────────────────────────────────────┘
```

---

**Remember: "For small catalogs, simplicity wins. Focus on beautiful images and clear information."** 🖼️✨

