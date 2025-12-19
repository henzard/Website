---
description: "Firebase security rules, best practices, and data structure standards"
alwaysApply: true
---

# Firebase Standards & Security

## Firestore Security Rules

### Core Security Principles

1. **Never trust client data** - Always validate on the server
2. **Principle of least privilege** - Users access only what they need
3. **Defense in depth** - Multiple layers of security
4. **Audit everything** - Log all sensitive operations

### Security Rules Template

```javascript
// firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }
    
    function isOwner(userId) {
      return isAuthenticated() && request.auth.uid == userId;
    }
    
    function hasRole(role) {
      return isAuthenticated() && 
             get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == role;
    }
    
    function isValidEmail(email) {
      return email.matches('^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$');
    }
    
    // Users collection
    match /users/{userId} {
      // Read: User can read own profile, admins can read all
      allow read: if isOwner(userId) || hasRole('admin');
      
      // Create: Only during authentication, must match auth.uid
      allow create: if isAuthenticated() && 
                      request.auth.uid == userId &&
                      request.resource.data.keys().hasAll(['email', 'createdAt']) &&
                      isValidEmail(request.resource.data.email);
      
      // Update: Users can update own profile (except role)
      allow update: if isOwner(userId) && 
                      !request.resource.data.diff(resource.data).affectedKeys().hasAny(['role', 'createdAt']);
      
      // Delete: Never allow from client
      allow delete: if false;
      
      // User's private data subcollection
      match /private/{document=**} {
        allow read, write: if isOwner(userId);
      }
    }
    
    // Bookings collection
    match /bookings/{bookingId} {
      allow read: if isAuthenticated() && 
                    (resource.data.userId == request.auth.uid || hasRole('admin'));
      
      allow create: if isAuthenticated() &&
                      request.resource.data.userId == request.auth.uid &&
                      request.resource.data.keys().hasAll(['userId', 'date', 'status']) &&
                      request.resource.data.status == 'pending';
      
      allow update: if isOwner(resource.data.userId) || hasRole('admin');
      allow delete: if hasRole('admin');
    }
    
    // Staff-only collection
    match /staff/{staffId} {
      allow read, write: if hasRole('admin') || hasRole('staff');
    }
    
    // Public data (read-only for all)
    match /public/{document=**} {
      allow read: if true;
      allow write: if hasRole('admin');
    }
  }
}
```

## Firebase Storage Security Rules

```javascript
// storage.rules
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    
    // User profile pictures
    match /users/{userId}/profile/{fileName} {
      allow read: if true; // Public profile pictures
      allow write: if request.auth != null && 
                     request.auth.uid == userId &&
                     request.resource.size < 5 * 1024 * 1024 && // 5MB max
                     request.resource.contentType.matches('image/.*');
    }
    
    // Private user documents
    match /users/{userId}/documents/{fileName} {
      allow read, write: if request.auth != null && 
                           request.auth.uid == userId &&
                           request.resource.size < 10 * 1024 * 1024; // 10MB max
    }
    
    // Booking attachments
    match /bookings/{bookingId}/{fileName} {
      allow read: if request.auth != null;
      allow write: if request.auth != null &&
                     request.resource.size < 10 * 1024 * 1024;
    }
  }
}
```

## Data Structure Standards

### Document Structure

```typescript
// ✅ GOOD - Flat, indexed, with timestamps
interface User {
  id: string;
  email: string;
  displayName: string;
  role: 'user' | 'staff' | 'admin';
  createdAt: Timestamp;
  updatedAt: Timestamp;
  // Store references, not nested objects
  organizationId?: string;
  profileImageUrl?: string;
}

// ❌ BAD - Deeply nested, no timestamps
interface User {
  id: string;
  info: {
    personal: {
      email: string;
      name: string;
    };
    settings: {
      notifications: {...}
    }
  };
}
```

### Collection Naming

- Use lowercase, plural nouns: `users`, `bookings`, `products`
- Subcollections: `users/{userId}/preferences`
- Never use special characters or spaces

### Indexing Strategy

Create indexes for common queries:

```javascript
// firestore.indexes.json
{
  "indexes": [
    {
      "collectionGroup": "bookings",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "userId", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    },
    {
      "collectionGroup": "users",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "role", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    }
  ]
}
```

## Firebase Service Implementation

### Repository Pattern

```typescript
// repositories/baseRepository.ts
import { 
  collection, 
  doc, 
  getDoc, 
  getDocs, 
  setDoc, 
  updateDoc,
  deleteDoc,
  query,
  where,
  orderBy,
  limit,
  Timestamp
} from 'firebase/firestore';

export class BaseRepository<T> {
  constructor(
    protected db: Firestore,
    protected collectionName: string
  ) {}

  async findById(id: string): Promise<T | null> {
    const docRef = doc(this.db, this.collectionName, id);
    const snapshot = await getDoc(docRef);
    return snapshot.exists() ? { id: snapshot.id, ...snapshot.data() } as T : null;
  }

  async findAll(constraints: QueryConstraint[] = []): Promise<T[]> {
    const collectionRef = collection(this.db, this.collectionName);
    const q = query(collectionRef, ...constraints);
    const snapshot = await getDocs(q);
    return snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() } as T));
  }

  async create(id: string, data: Omit<T, 'id'>): Promise<void> {
    const docRef = doc(this.db, this.collectionName, id);
    await setDoc(docRef, {
      ...data,
      createdAt: Timestamp.now(),
      updatedAt: Timestamp.now()
    });
  }

  async update(id: string, data: Partial<T>): Promise<void> {
    const docRef = doc(this.db, this.collectionName, id);
    await updateDoc(docRef, {
      ...data,
      updatedAt: Timestamp.now()
    });
  }

  async delete(id: string): Promise<void> {
    const docRef = doc(this.db, this.collectionName, id);
    await deleteDoc(docRef);
  }
}
```

### Service Layer with Validation

```typescript
// services/userService.ts
import { z } from 'zod';

const UserSchema = z.object({
  email: z.string().email(),
  displayName: z.string().min(2).max(50),
  role: z.enum(['user', 'staff', 'admin'])
});

export class UserService {
  constructor(private userRepository: BaseRepository<User>) {}

  async createUser(data: CreateUserDto): Promise<User> {
    // Validate
    const validated = UserSchema.parse(data);
    
    // Business logic
    const userId = generateUserId();
    
    // Save
    await this.userRepository.create(userId, validated);
    
    // Return
    const user = await this.userRepository.findById(userId);
    if (!user) throw new Error('Failed to create user');
    
    return user;
  }
}
```

## Real-time Listeners

```typescript
// hooks/useRealtimeDocument.ts
export const useRealtimeDocument = <T>(
  collectionName: string,
  documentId: string
) => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const docRef = doc(db, collectionName, documentId);
    
    const unsubscribe = onSnapshot(
      docRef,
      (snapshot) => {
        if (snapshot.exists()) {
          setData({ id: snapshot.id, ...snapshot.data() } as T);
        } else {
          setData(null);
        }
        setLoading(false);
      },
      (err) => {
        setError(err as Error);
        setLoading(false);
      }
    );

    return () => unsubscribe();
  }, [collectionName, documentId]);

  return { data, loading, error };
};
```

## Cloud Functions Best Practices

```typescript
// functions/src/index.ts
import { onCall, HttpsError } from 'firebase-functions/v2/https';
import { onDocumentCreated } from 'firebase-functions/v2/firestore';

// Callable function with authentication
export const createBooking = onCall(async (request) => {
  // Check authentication
  if (!request.auth) {
    throw new HttpsError('unauthenticated', 'User must be authenticated');
  }

  // Validate input
  const { date, serviceId } = request.data;
  if (!date || !serviceId) {
    throw new HttpsError('invalid-argument', 'Missing required fields');
  }

  // Business logic with error handling
  try {
    // Process booking
    const bookingId = await processBooking(request.auth.uid, date, serviceId);
    
    return { success: true, bookingId };
  } catch (error) {
    console.error('Booking creation failed:', error);
    throw new HttpsError('internal', 'Booking creation failed');
  }
});

// Background trigger
export const onUserCreated = onDocumentCreated('users/{userId}', async (event) => {
  const userData = event.data?.data();
  
  // Send welcome email, create default settings, etc.
  await sendWelcomeEmail(userData.email);
});
```

## AI Decision Gate

**Before implementing Firebase features:**
1. ✅ Are security rules written and tested?
2. ✅ Is user data properly validated?
3. ✅ Are all queries indexed?
4. ✅ Is authentication required where needed?
5. ✅ Are file uploads size-limited?
6. ✅ Is sensitive data in private subcollections?
7. ✅ Are Cloud Functions properly secured?

**Never proceed without proper security rules. Never trust client input.**

