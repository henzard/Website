---
description: "Database design, naming conventions, normalization, and data integrity standards"
alwaysApply: true
globs: ["**/repositories/**", "**/services/**", "**/types/**"]
---

# Database Design & Naming Standards

## 🎯 Core Principles

1. **Data Normalization** - Structure data properly, no dumping
2. **Consistent Naming** - Clear, predictable names
3. **Type Safety** - TypeScript types match database structure
4. **Scalability** - Design for growth from day one
5. **Query Efficiency** - Structure data for fast reads

---

## 📐 Data Normalization Rules

### The Anti-Dump Rule

**❌ NEVER DO THIS:**
```typescript
// Dumping everything into one field
interface User {
  id: string;
  data: string; // JSON string with everything - WRONG!
}

// Storing arrays as comma-separated strings
interface Booking {
  id: string;
  services: string; // "yoga,pilates,massage" - WRONG!
}

// Duplicating related data everywhere
interface Booking {
  id: string;
  userId: string;
  userName: string;        // Duplicate from users
  userEmail: string;       // Duplicate from users
  userPhone: string;       // Duplicate from users
  userAddress: string;     // Duplicate from users
  // WRONG - Store reference only!
}
```

### ✅ Proper Data Structure

#### 1. One Entity, One Collection

```typescript
// ✅ GOOD - Each entity in its own collection
interface User {
  id: string;
  email: string;
  displayName: string;
  role: 'user' | 'staff' | 'admin';
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

interface Booking {
  id: string;
  userId: string; // Reference to user, not duplicate data
  serviceId: string; // Reference to service
  date: Timestamp;
  status: 'pending' | 'confirmed' | 'cancelled';
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

interface Service {
  id: string;
  name: string;
  description: string;
  duration: number; // minutes
  price: number;
  active: boolean;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

#### 2. Use Proper Arrays for Multiple Values

```typescript
// ✅ GOOD - Array of IDs
interface Booking {
  id: string;
  userId: string;
  serviceIds: string[]; // Multiple services
  addOns: string[]; // Multiple add-ons
  createdAt: Timestamp;
}

// ✅ GOOD - Array of objects for complex data
interface Product {
  id: string;
  name: string;
  variants: Array<{
    id: string;
    name: string;
    price: number;
    stock: number;
  }>;
}
```

#### 3. Acceptable Denormalization (Performance)

Sometimes duplication is okay for performance:

```typescript
// ✅ ACCEPTABLE - Commonly displayed data
interface Booking {
  id: string;
  userId: string;
  userName: string; // OK - Displayed in lists, avoid extra query
  userEmail: string; // OK - Needed for notifications
  serviceId: string;
  serviceName: string; // OK - Displayed everywhere
  servicePrice: number; // OK - Snapshot of price at booking time
  createdAt: Timestamp;
}

// But update strategy must exist:
// When user updates name → Update all their bookings (Cloud Function)
```

**Rules for acceptable denormalization:**
- Data is displayed frequently (every list view)
- Querying separate collection would be expensive
- Data rarely changes (or you have update strategy)
- It's a snapshot (like price at time of purchase)

---

## 📛 Naming Conventions

### Collection Names

```typescript
// ✅ GOOD - Plural, lowercase, no special characters
collections:
  - users
  - bookings
  - services
  - products
  - orders
  - payments
  - reviews

// ❌ BAD
collections:
  - User          // Not plural
  - booking_data  // Unnecessary suffix
  - user-profiles // Use camelCase or remove hyphen
  - Products      // Don't capitalize
  - tbl_orders    // No database prefixes
```

### Document Field Names

```typescript
// ✅ GOOD - camelCase, descriptive, no abbreviations
interface User {
  id: string;
  email: string;
  displayName: string;           // Not "name" or "dispName"
  phoneNumber: string;           // Not "phone" or "num"
  dateOfBirth: Timestamp;        // Not "dob"
  profileImageUrl: string;       // Not "img" or "pic"
  isEmailVerified: boolean;      // Boolean starts with "is"
  hasCompletedOnboarding: boolean;
  createdAt: Timestamp;
  updatedAt: Timestamp;
  lastLoginAt: Timestamp;
}

// ❌ BAD
interface User {
  ID: string;                    // Don't capitalize
  e_mail: string;                // No underscores
  name: string;                  // Too vague
  ph: string;                    // Abbreviated
  img: string;                   // Abbreviated
  verified: boolean;             // Ambiguous
  timestamp: Timestamp;          // Which timestamp?
  usr_created: Timestamp;        // No prefixes
}
```

### Boolean Field Naming

```typescript
// ✅ GOOD - Clearly indicates yes/no
interface User {
  isActive: boolean;
  isEmailVerified: boolean;
  hasCompletedProfile: boolean;
  canPublishContent: boolean;
  shouldReceiveNotifications: boolean;
}

// ❌ BAD
interface User {
  active: boolean;               // Could be string/enum
  verified: boolean;             // Verified what?
  profile: boolean;              // Unclear
  notifications: boolean;        // Too vague
}
```

### Date/Time Fields

```typescript
// ✅ GOOD - Consistent suffix "At"
interface Document {
  createdAt: Timestamp;          // When created
  updatedAt: Timestamp;          // When last modified
  deletedAt: Timestamp | null;  // Soft delete timestamp
  publishedAt: Timestamp | null; // When published
  scheduledAt: Timestamp;        // When scheduled for
  expiresAt: Timestamp;          // When expires
}

// ❌ BAD
interface Document {
  created: Timestamp;            // No "At"
  modified_date: Timestamp;      // Inconsistent
  deleteTime: Timestamp;         // Use "At" not "Time"
  pub_date: Timestamp;           // Abbreviated
}
```

### Reference Fields

```typescript
// ✅ GOOD - Consistent "Id" or "Ids" suffix
interface Booking {
  id: string;
  userId: string;                // References users collection
  serviceId: string;             // References services collection
  organizationId: string;        // References organizations collection
  createdById: string;           // References users collection (who created)
  
  // Multiple references
  participantIds: string[];      // References users collection
  tagIds: string[];              // References tags collection
}

// ❌ BAD
interface Booking {
  user: string;                  // No "Id" suffix
  service_ref: string;           // No "Id", has underscore
  org: string;                   // Abbreviated
  creator: string;               // Ambiguous
  participants: string[];        // Not clear it's IDs
}
```

### Status/Enum Fields

```typescript
// ✅ GOOD - Descriptive name, clear values
interface Order {
  id: string;
  status: 'pending' | 'processing' | 'shipped' | 'delivered' | 'cancelled';
  paymentStatus: 'unpaid' | 'paid' | 'refunded' | 'failed';
  fulfillmentStatus: 'unfulfilled' | 'partially_fulfilled' | 'fulfilled';
}

// ❌ BAD
interface Order {
  status: 0 | 1 | 2 | 3;         // Use strings, not magic numbers
  payment: 'p' | 'u';            // Too abbreviated
  state: string;                 // No specific type
}
```

---

## 🗂️ Collection Structure Standards

### Root Collections

```
firestore/
├── users/                      # User accounts
├── organizations/              # Business entities
├── products/                   # Product catalog
├── orders/                     # Customer orders
├── bookings/                   # Service bookings
├── payments/                   # Payment records
├── reviews/                    # Product/service reviews
├── notifications/              # User notifications
└── analytics/                  # Analytics events
```

### Subcollections

```typescript
// ✅ GOOD - Use for one-to-many relationships
users/{userId}/
  ├── private/                  # Private user data
  │   └── {docId}
  ├── preferences/              # User settings
  │   └── {docId}
  ├── sessions/                 # Active sessions
  │   └── {sessionId}
  └── activityLog/              # User activity
      └── {logId}

// ✅ GOOD - Use for organizational structure
organizations/{orgId}/
  ├── members/                  # Org members
  │   └── {userId}
  ├── teams/                    # Teams within org
  │   └── {teamId}
  └── settings/                 # Org settings
      └── {settingKey}
```

**When to use subcollections:**
- Data belongs to specific document (user's preferences)
- Data is accessed only through parent (user's private data)
- Helps with security rules (inherit parent permissions)
- Large dataset that would bloat parent document

**When NOT to use subcollections:**
- Data needs to be queried across all users (use root collection)
- Data is referenced by multiple parents (use root collection)
- Need collection group queries (use root or specific structure)

---

## 🔢 Field Types & Validation

### String Fields

```typescript
interface User {
  // ✅ GOOD - Specific, validated strings
  email: string;                 // Validated email format
  phoneNumber: string;           // Format: +1234567890
  postalCode: string;            // Not number (can start with 0)
  countryCode: string;           // ISO 3166-1 alpha-2 (US, GB, ZA)
  languageCode: string;          // ISO 639-1 (en, es, fr)
  timezone: string;              // IANA timezone (America/New_York)
  
  // ❌ BAD
  phone: number;                 // Use string for phone numbers!
  country: string;               // Use code, not full name
  lang: string;                  // Too abbreviated
}
```

### Number Fields

```typescript
interface Product {
  // ✅ GOOD - Clear units and constraints
  price: number;                 // In cents (599 = $5.99)
  weight: number;                // In grams
  stock: number;                 // Integer, non-negative
  rating: number;                // 0.0 to 5.0
  discountPercentage: number;    // 0 to 100
  
  // Add validation
  minimumOrderQuantity: number;  // Min: 1
  maximumOrderQuantity: number;  // Max: depends on stock
  
  // ❌ BAD
  price: number;                 // Is this dollars? cents?
  weight: number;                // Is this kg? lbs? grams?
  rating: number;                // 1-5? 1-10? 0-100?
  discount: number;              // Percentage? Fixed amount?
}
```

### Array Fields

```typescript
interface Article {
  // ✅ GOOD - Typed arrays
  tags: string[];                // Array of tag names
  tagIds: string[];              // Array of tag IDs
  authorIds: string[];           // Multiple authors
  
  images: Array<{                // Array of structured objects
    url: string;
    caption: string;
    width: number;
    height: number;
  }>;
  
  // ❌ BAD
  tags: string;                  // "tag1,tag2,tag3" comma-separated
  images: any[];                 // Untyped array
}
```

### Timestamps

```typescript
import { Timestamp } from 'firebase/firestore';

interface Document {
  // ✅ GOOD - Use Firestore Timestamp
  createdAt: Timestamp;
  updatedAt: Timestamp;
  publishedAt: Timestamp | null;
  
  // ❌ BAD
  createdAt: number;             // Unix timestamp - use Timestamp
  updatedAt: string;             // ISO string - use Timestamp
  publishedAt: Date;             // JS Date - use Timestamp
}

// Setting timestamps
await setDoc(docRef, {
  ...data,
  createdAt: Timestamp.now(),
  updatedAt: Timestamp.now(),
});
```

### GeoPoint

```typescript
import { GeoPoint } from 'firebase/firestore';

interface Location {
  // ✅ GOOD - Use Firestore GeoPoint
  coordinates: GeoPoint;
  address: string;
  
  // ❌ BAD
  lat: number;                   // Use GeoPoint instead
  lng: number;
  location: { lat: number; lng: number }; // Use GeoPoint
}

// Setting geopoint
await setDoc(docRef, {
  coordinates: new GeoPoint(37.7749, -122.4194),
  address: "San Francisco, CA"
});
```

---

## 📊 Data Relationships

### One-to-Many (Reference IDs)

```typescript
// ✅ GOOD - Store reference ID
interface Post {
  id: string;
  authorId: string;              // Reference to users collection
  title: string;
  content: string;
  createdAt: Timestamp;
}

// Query posts by author
const postsQuery = query(
  collection(db, 'posts'),
  where('authorId', '==', userId)
);
```

### Many-to-Many (Junction Collection or Arrays)

**Option A: Array of IDs (Small scale)**

```typescript
// ✅ GOOD - For limited relationships (<100)
interface Student {
  id: string;
  name: string;
  courseIds: string[];           // Enrolled courses
}

interface Course {
  id: string;
  name: string;
  studentIds: string[];          // Enrolled students
}

// BUT: Firestore limit is 10 items in array for queries
// Use Option B for larger datasets
```

**Option B: Junction Collection (Scalable)**

```typescript
// ✅ BETTER - For many relationships
interface Student {
  id: string;
  name: string;
  // No course list here
}

interface Course {
  id: string;
  name: string;
  // No student list here
}

interface Enrollment {
  id: string;
  studentId: string;
  courseId: string;
  enrolledAt: Timestamp;
  status: 'active' | 'completed' | 'dropped';
}

// Query all courses for a student
const enrollmentsQuery = query(
  collection(db, 'enrollments'),
  where('studentId', '==', studentId),
  where('status', '==', 'active')
);

// Query all students in a course
const enrollmentsQuery = query(
  collection(db, 'enrollments'),
  where('courseId', '==', courseId),
  where('status', '==', 'active')
);
```

---

## 🎯 TypeScript Type Definitions

### Base Types

```typescript
// types/base.ts

// Every document has these fields
export interface BaseDocument {
  id: string;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

// Soft delete support
export interface SoftDeletable {
  deletedAt: Timestamp | null;
  isDeleted: boolean;
}

// Audit trail
export interface Auditable extends BaseDocument {
  createdById: string;
  updatedById: string;
}
```

### Entity Types

```typescript
// types/user.ts
import { BaseDocument, SoftDeletable } from './base';

export type UserRole = 'user' | 'staff' | 'admin';

export interface User extends BaseDocument, SoftDeletable {
  // Authentication
  email: string;
  isEmailVerified: boolean;
  phoneNumber: string | null;
  isPhoneVerified: boolean;
  
  // Profile
  displayName: string;
  firstName: string;
  lastName: string;
  avatarUrl: string | null;
  dateOfBirth: Timestamp | null;
  
  // Authorization
  role: UserRole;
  permissions: string[];
  
  // Settings
  languageCode: string;
  timezone: string;
  notificationPreferences: {
    email: boolean;
    sms: boolean;
    push: boolean;
  };
  
  // Activity
  lastLoginAt: Timestamp | null;
  loginCount: number;
}

// Validation schema (Zod)
import { z } from 'zod';

export const UserSchema = z.object({
  email: z.string().email(),
  displayName: z.string().min(2).max(50),
  firstName: z.string().min(1).max(50),
  lastName: z.string().min(1).max(50),
  role: z.enum(['user', 'staff', 'admin']),
  // ...
});

// Type derived from validation
export type CreateUserDto = z.infer<typeof UserSchema>;
```

---

## 🔍 Indexing Strategy

### Composite Indexes

```json
// firestore.indexes.json

{
  "indexes": [
    {
      "collectionGroup": "bookings",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "userId", "order": "ASCENDING" },
        { "fieldPath": "status", "order": "ASCENDING" },
        { "fieldPath": "date", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "products",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "categoryId", "order": "ASCENDING" },
        { "fieldPath": "isActive", "order": "ASCENDING" },
        { "fieldPath": "price", "order": "ASCENDING" }
      ]
    }
  ]
}
```

**Index for every query pattern:**
- List user's active bookings by date
- List products in category, active only, sorted by price
- List orders by customer, filtered by status, sorted by date

---

## 🚫 Anti-Patterns to Avoid

### 1. The God Object

```typescript
// ❌ BAD - Everything in one document
interface User {
  id: string;
  // 50+ fields here...
  bookingHistory: Booking[];     // Could be 1000s of items!
  orders: Order[];               // Could be 1000s of items!
  activityLog: Activity[];       // Could be 10,000s of items!
  // Document size limit is 1MB!
}

// ✅ GOOD - Separate collections
interface User {
  id: string;
  // Only essential user fields
}
// bookings in separate collection
// orders in separate collection
// activity in separate collection or subcollection
```

### 2. The String Dump

```typescript
// ❌ BAD - JSON string in field
interface Order {
  id: string;
  itemsJson: string;             // '{"items":[...]}'
}

// ✅ GOOD - Proper structure
interface Order {
  id: string;
  items: Array<{
    productId: string;
    quantity: number;
    price: number;
  }>;
}
```

### 3. The Magic Number

```typescript
// ❌ BAD - Meaningless numbers
interface User {
  status: number;                // 0=active, 1=inactive, 2=banned ???
}

// ✅ GOOD - String enums
interface User {
  status: 'active' | 'inactive' | 'banned';
}
```

### 4. The Mega Array

```typescript
// ❌ BAD - Unbounded arrays
interface User {
  notifications: Notification[]; // Could have 10,000 items!
}

// ✅ GOOD - Subcollection for large lists
users/{userId}/notifications/{notificationId}
```

---

## ✅ AI Decision Gate

**Before creating/modifying any collection:**

```
┌─────────────────────────────────────────────┐
│  DATABASE DESIGN CHECKLIST                  │
├─────────────────────────────────────────────┤
│  Naming                                     │
│  ☐ Collection name is plural, lowercase    │
│  ☐ Field names are camelCase               │
│  ☐ Boolean fields start with is/has/can    │
│  ☐ Date fields end with "At"               │
│  ☐ Reference fields end with "Id/Ids"      │
│                                             │
│  Structure                                  │
│  ☐ No data dumped into string fields       │
│  ☐ No arrays stored as comma-separated     │
│  ☐ Data is normalized (no unnecessary dupes)│
│  ☐ TypeScript interface defined            │
│  ☐ Zod schema for validation               │
│                                             │
│  Performance                                │
│  ☐ Indexes planned for query patterns      │
│  ☐ Document size will stay under 1MB       │
│  ☐ Arrays won't exceed reasonable size     │
│  ☐ Subcollections for large related data   │
│                                             │
│  Security                                   │
│  ☐ Firestore rules written                 │
│  ☐ Sensitive data in private subcollection │
│  ☐ Audit fields included (if needed)       │
│                                             │
│  IF ANY ☐ UNCHECKED:                        │
│  ➜ FIX BEFORE IMPLEMENTING                  │
└─────────────────────────────────────────────┘
```

---

**Remember: "A well-designed database is a joy to work with. A poorly designed one is technical debt from day one."** 🗄️✨

