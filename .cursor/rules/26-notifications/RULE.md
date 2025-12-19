---
description: "Push notifications and in-app notification systems"
alwaysApply: false
globs: ["**/notifications/**", "**/components/Notification*"]
---

# Notifications Standards

## 🎯 Core Principles

- ✅ Ask permission at the right time
- ✅ Provide value with every notification
- ✅ Allow granular preferences
- ✅ Support both push and in-app notifications
- ✅ Never spam users

---

## 🔔 Firebase Cloud Messaging Setup

```typescript
// lib/fcm.ts
import { getMessaging, getToken, onMessage } from 'firebase/messaging';
import { app } from './firebase';

const messaging = getMessaging(app);

export const requestNotificationPermission = async (): Promise<string | null> => {
  try {
    const permission = await Notification.requestPermission();
    if (permission !== 'granted') return null;
    
    const token = await getToken(messaging, {
      vapidKey: import.meta.env.VITE_FIREBASE_VAPID_KEY,
    });
    
    return token;
  } catch (error) {
    console.error('FCM token error:', error);
    return null;
  }
};

// Listen for foreground messages
export const onForegroundMessage = (callback: (payload: any) => void) => {
  return onMessage(messaging, callback);
};
```

---

## 📬 In-App Notifications

```typescript
// types/notification.ts
interface AppNotification {
  id: string;
  userId: string;
  type: 'info' | 'success' | 'warning' | 'error';
  title: string;
  message: string;
  link?: string;
  read: boolean;
  createdAt: Timestamp;
}

// hooks/useNotifications.ts
export const useNotifications = () => {
  const { user } = useAuth();
  const [notifications, setNotifications] = useState<AppNotification[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);

  useEffect(() => {
    if (!user) return;

    const q = query(
      collection(db, 'notifications'),
      where('userId', '==', user.uid),
      orderBy('createdAt', 'desc'),
      limit(20)
    );

    const unsubscribe = onSnapshot(q, (snapshot) => {
      const notifs = snapshot.docs.map(doc => ({
        id: doc.id,
        ...doc.data()
      })) as AppNotification[];
      
      setNotifications(notifs);
      setUnreadCount(notifs.filter(n => !n.read).length);
    });

    return unsubscribe;
  }, [user]);

  const markAsRead = async (id: string) => {
    await updateDoc(doc(db, 'notifications', id), { read: true });
  };

  const markAllAsRead = async () => {
    const batch = writeBatch(db);
    notifications.filter(n => !n.read).forEach(n => {
      batch.update(doc(db, 'notifications', n.id), { read: true });
    });
    await batch.commit();
  };

  return { notifications, unreadCount, markAsRead, markAllAsRead };
};
```

---

## 🔔 Notification Bell Component

```typescript
// components/NotificationBell.tsx
export const NotificationBell: React.FC = () => {
  const { notifications, unreadCount, markAsRead, markAllAsRead } = useNotifications();
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="relative">
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="relative p-2 rounded-full hover:bg-gray-100"
      >
        <BellIcon className="w-6 h-6" />
        {unreadCount > 0 && (
          <span className="absolute top-0 right-0 w-5 h-5 bg-red-500 text-white text-xs rounded-full flex items-center justify-center">
            {unreadCount > 9 ? '9+' : unreadCount}
          </span>
        )}
      </button>

      {isOpen && (
        <div className="absolute right-0 mt-2 w-80 bg-white rounded-lg shadow-lg border z-50">
          <div className="flex justify-between items-center p-3 border-b">
            <h3 className="font-semibold">Notifications</h3>
            {unreadCount > 0 && (
              <button
                onClick={markAllAsRead}
                className="text-sm text-primary-600 hover:underline"
              >
                Mark all read
              </button>
            )}
          </div>

          <div className="max-h-96 overflow-y-auto">
            {notifications.length === 0 ? (
              <p className="p-4 text-gray-500 text-center">No notifications</p>
            ) : (
              notifications.map((notif) => (
                <div
                  key={notif.id}
                  onClick={() => markAsRead(notif.id)}
                  className={`p-3 border-b cursor-pointer hover:bg-gray-50 ${
                    !notif.read ? 'bg-blue-50' : ''
                  }`}
                >
                  <p className="font-medium text-sm">{notif.title}</p>
                  <p className="text-gray-600 text-sm">{notif.message}</p>
                  <p className="text-xs text-gray-400 mt-1">
                    {formatDistanceToNow(notif.createdAt.toDate())} ago
                  </p>
                </div>
              ))
            )}
          </div>
        </div>
      )}
    </div>
  );
};
```

---

## 👤 User Preferences

```typescript
// types/preferences.ts
interface NotificationPreferences {
  email: {
    marketing: boolean;
    bookingReminders: boolean;
    orderUpdates: boolean;
  };
  push: {
    enabled: boolean;
    bookingReminders: boolean;
    orderUpdates: boolean;
  };
  inApp: {
    enabled: boolean;
  };
}

// components/NotificationSettings.tsx
export const NotificationSettings: React.FC = () => {
  const [prefs, setPrefs] = useState<NotificationPreferences>(defaultPrefs);

  return (
    <div className="space-y-6">
      <h2 className="text-xl font-semibold">Notification Preferences</h2>

      <section>
        <h3 className="font-medium mb-3">Email Notifications</h3>
        <div className="space-y-2">
          <Toggle
            label="Booking reminders"
            checked={prefs.email.bookingReminders}
            onChange={(v) => updatePref('email.bookingReminders', v)}
          />
          <Toggle
            label="Order updates"
            checked={prefs.email.orderUpdates}
            onChange={(v) => updatePref('email.orderUpdates', v)}
          />
        </div>
      </section>

      <section>
        <h3 className="font-medium mb-3">Push Notifications</h3>
        <Toggle
          label="Enable push notifications"
          checked={prefs.push.enabled}
          onChange={handlePushToggle}
        />
      </section>
    </div>
  );
};
```

---

## ✅ Checklist

```
Push Notifications
☐ FCM configured
☐ VAPID key set
☐ Permission requested contextually
☐ Service worker registered

In-App Notifications
☐ Real-time with Firestore
☐ Mark as read functionality
☐ Unread count badge
☐ Link to relevant content

User Preferences
☐ Granular controls
☐ Easy to disable
☐ Preferences saved to Firestore

IF ANY ☐ UNCHECKED → Fix before launching
```

---

**Remember: "Every notification should provide value or it's spam."** 🔔✨

