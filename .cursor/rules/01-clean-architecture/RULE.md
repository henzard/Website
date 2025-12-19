---
description: "Clean architecture principles and separation of concerns for React+Firebase applications"
alwaysApply: true
---

# Clean Architecture Standards

## Core Principles

1. **Separation of Concerns**: UI, business logic, and data access are separate
2. **Dependency Rule**: Dependencies point inward (UI → Services → Data)
3. **Single Responsibility**: Each module has one reason to change
4. **DRY (Don't Repeat Yourself)**: Reusable logic goes into utilities/hooks

## Layer Architecture

### 1. Presentation Layer (`/components`, `/pages`)
**Responsibility**: UI rendering and user interaction only

```typescript
// ✅ GOOD - Component only handles UI
const UserProfile = ({ userId }: Props) => {
  const { user, loading, error } = useUser(userId);
  
  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  
  return <ProfileCard user={user} />;
};

// ❌ BAD - Component directly accessing Firebase
const UserProfile = ({ userId }: Props) => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    getDoc(doc(db, 'users', userId)).then(setUser); // Direct Firebase call
  }, [userId]);
  
  return <ProfileCard user={user} />;
};
```

### 2. Business Logic Layer (`/hooks`, `/services`)
**Responsibility**: Application logic, data transformation, validation

```typescript
// /hooks/useUser.ts
export const useUser = (userId: string) => {
  return useQuery(['user', userId], () => userService.getById(userId));
};

// /services/userService.ts
export const userService = {
  async getById(userId: string): Promise<User> {
    const userData = await userRepository.findById(userId);
    return transformUserData(userData); // Business logic
  },
  
  async updateProfile(userId: string, data: UpdateUserDto) {
    validateUserData(data); // Validation
    return userRepository.update(userId, data);
  }
};
```

### 3. Data Access Layer (`/repositories`)
**Responsibility**: Database operations only

```typescript
// /repositories/userRepository.ts
import { db } from '@/config/firebase';
import { collection, doc, getDoc, updateDoc } from 'firebase/firestore';

export const userRepository = {
  async findById(userId: string) {
    const docRef = doc(db, 'users', userId);
    const snapshot = await getDoc(docRef);
    return snapshot.exists() ? snapshot.data() : null;
  },
  
  async update(userId: string, data: any) {
    const docRef = doc(db, 'users', userId);
    await updateDoc(docRef, data);
  }
};
```

## Feature-Based Organization

Organize code by feature, not by type:

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   └── RegisterForm.tsx
│   │   ├── hooks/
│   │   │   └── useAuth.ts
│   │   ├── services/
│   │   │   └── authService.ts
│   │   ├── types/
│   │   │   └── auth.types.ts
│   │   └── index.ts
│   ├── booking/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── types/
│   └── profile/
└── shared/
    ├── components/
    ├── hooks/
    └── utils/
```

## Dependency Injection Pattern

Use configuration objects and dependency injection:

```typescript
// config/firebase.ts
export const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  // ...
};

export const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);
export const auth = getAuth(app);

// services/baseService.ts
import { db } from '@/config/firebase';

export class BaseService {
  constructor(protected collectionName: string) {}
  
  protected getCollection() {
    return collection(db, this.collectionName);
  }
}

// services/userService.ts
export class UserService extends BaseService {
  constructor() {
    super('users');
  }
  
  async getById(id: string) {
    // Implementation using this.getCollection()
  }
}
```

## Error Boundaries

Always implement error boundaries for resilient UIs:

```typescript
// components/ErrorBoundary.tsx
class ErrorBoundary extends React.Component<Props, State> {
  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    logErrorToService(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }
    return this.props.children;
  }
}
```

## AI Decision Gate

**Before implementing any feature, ask:**
1. ✅ Is this component only handling UI?
2. ✅ Is business logic in services/hooks?
3. ✅ Are Firebase calls isolated in repositories?
4. ✅ Is the feature organized in its own module?
5. ✅ Can this be tested without Firebase?

**If unsure, stop and refactor before continuing.**

