---
description: "Search implementation with Algolia or Firebase full-text search"
alwaysApply: false
globs: ["**/search/**", "**/components/Search*"]
---

# Search Implementation Standards

## 🎯 Core Principles

- ✅ Fast, responsive search (< 200ms)
- ✅ Typo tolerance
- ✅ Relevant results ranking
- ✅ Autocomplete suggestions
- ✅ Filter and facet support

---

## 🔍 Option 1: Algolia (Recommended)

### Setup

```bash
npm install algoliasearch react-instantsearch
```

```typescript
// lib/algolia.ts
import algoliasearch from 'algoliasearch';

export const searchClient = algoliasearch(
  import.meta.env.VITE_ALGOLIA_APP_ID,
  import.meta.env.VITE_ALGOLIA_SEARCH_KEY
);

export const INDEX_NAME = 'products';
```

### Search Component

```typescript
// components/Search.tsx
import { InstantSearch, SearchBox, Hits, Highlight } from 'react-instantsearch';
import { searchClient, INDEX_NAME } from '@/lib/algolia';

export const Search: React.FC = () => {
  return (
    <InstantSearch searchClient={searchClient} indexName={INDEX_NAME}>
      <SearchBox
        placeholder="Search products..."
        classNames={{
          input: 'w-full px-4 py-2 border rounded-lg',
          submitIcon: 'hidden',
          resetIcon: 'hidden',
        }}
      />
      
      <Hits
        hitComponent={({ hit }) => (
          <a href={`/products/${hit.objectID}`} className="block p-3 hover:bg-gray-50">
            <Highlight attribute="name" hit={hit} className="font-medium" />
            <p className="text-sm text-gray-600">{hit.description}</p>
            <p className="text-primary-600 font-bold">${hit.price}</p>
          </a>
        )}
      />
    </InstantSearch>
  );
};
```

### Sync Firebase to Algolia

```typescript
// functions/src/algoliaSync.ts
import * as functions from 'firebase-functions';
import algoliasearch from 'algoliasearch';

const client = algoliasearch(
  process.env.ALGOLIA_APP_ID!,
  process.env.ALGOLIA_ADMIN_KEY!
);
const index = client.initIndex('products');

export const syncProductToAlgolia = functions.firestore
  .document('products/{productId}')
  .onWrite(async (change, context) => {
    const { productId } = context.params;

    if (!change.after.exists) {
      // Deleted
      await index.deleteObject(productId);
      return;
    }

    const data = change.after.data();
    await index.saveObject({
      objectID: productId,
      name: data.name,
      description: data.description,
      price: data.price,
      category: data.category,
      inStock: data.inStock,
    });
  });
```

---

## 🔍 Option 2: Simple Firebase Search

```typescript
// For small datasets (< 1000 items)
export const searchProducts = async (query: string): Promise<Product[]> => {
  const searchTerms = query.toLowerCase().split(' ');
  
  const snapshot = await getDocs(collection(db, 'products'));
  
  return snapshot.docs
    .map(doc => ({ id: doc.id, ...doc.data() } as Product))
    .filter(product => {
      const searchableText = `${product.name} ${product.description}`.toLowerCase();
      return searchTerms.every(term => searchableText.includes(term));
    })
    .slice(0, 20);
};
```

---

## 🎯 Autocomplete

```typescript
// components/SearchAutocomplete.tsx
import { useState, useEffect } from 'react';
import { useDebouncedCallback } from 'use-debounce';

export const SearchAutocomplete: React.FC = () => {
  const [query, setQuery] = useState('');
  const [suggestions, setSuggestions] = useState<string[]>([]);
  const [isOpen, setIsOpen] = useState(false);

  const fetchSuggestions = useDebouncedCallback(async (q: string) => {
    if (q.length < 2) {
      setSuggestions([]);
      return;
    }
    
    const results = await searchProducts(q);
    setSuggestions(results.map(r => r.name).slice(0, 5));
  }, 300);

  useEffect(() => {
    fetchSuggestions(query);
  }, [query]);

  return (
    <div className="relative">
      <input
        type="search"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        onFocus={() => setIsOpen(true)}
        placeholder="Search..."
        className="w-full px-4 py-2 border rounded-lg"
      />
      
      {isOpen && suggestions.length > 0 && (
        <ul className="absolute w-full bg-white border rounded-lg mt-1 shadow-lg z-50">
          {suggestions.map((suggestion, i) => (
            <li
              key={i}
              onClick={() => {
                setQuery(suggestion);
                setIsOpen(false);
              }}
              className="px-4 py-2 hover:bg-gray-100 cursor-pointer"
            >
              {suggestion}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
};
```

---

## ✅ Checklist

```
Search Quality
☐ Results appear < 200ms
☐ Typo tolerance enabled
☐ Relevant ranking
☐ No results message shown

UX
☐ Debounced input
☐ Loading indicator
☐ Keyboard navigation
☐ Mobile-friendly

Data Sync
☐ Real-time sync to search index
☐ Deletes handled
☐ Only searchable fields indexed

IF ANY ☐ UNCHECKED → Fix before launching
```

---

**Remember: "Good search is invisible. Bad search loses customers."** 🔍✨

