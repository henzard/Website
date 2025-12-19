---
description: "Booking and scheduling system patterns for appointments and reservations"
alwaysApply: false
globs: ["**/booking/**", "**/calendar/**", "**/appointments/**"]
---

# Booking & Scheduling System Standards

## 🎯 Core Principles

- ✅ Show real-time availability
- ✅ Handle time zones correctly
- ✅ Prevent double bookings
- ✅ Send confirmation emails
- ✅ Allow cancellations/rescheduling
- ✅ Buffer time between appointments

---

## 🗄️ Data Structure

```typescript
// types/booking.ts
interface Booking {
  id: string;
  userId: string;
  serviceId: string;
  staffId?: string;
  
  // Time (always store in UTC)
  startTime: Timestamp;
  endTime: Timestamp;
  timezone: string; // User's timezone
  
  // Status
  status: 'pending' | 'confirmed' | 'cancelled' | 'completed' | 'no-show';
  
  // Contact
  customerName: string;
  customerEmail: string;
  customerPhone?: string;
  notes?: string;
  
  // Metadata
  createdAt: Timestamp;
  updatedAt: Timestamp;
  cancelledAt?: Timestamp;
  cancellationReason?: string;
}

interface Service {
  id: string;
  name: string;
  duration: number; // minutes
  bufferBefore: number; // minutes
  bufferAfter: number; // minutes
  price: number;
  isActive: boolean;
}

interface Availability {
  dayOfWeek: number; // 0-6 (Sunday-Saturday)
  startTime: string; // "09:00"
  endTime: string; // "17:00"
  isAvailable: boolean;
}

interface BlockedTime {
  id: string;
  staffId?: string;
  startTime: Timestamp;
  endTime: Timestamp;
  reason: string;
}
```

---

## 📅 Availability Checker

```typescript
// services/availabilityService.ts
import { 
  collection, query, where, getDocs, 
  Timestamp, orderBy 
} from 'firebase/firestore';

interface TimeSlot {
  start: Date;
  end: Date;
  available: boolean;
}

export const getAvailableSlots = async (
  date: Date,
  serviceId: string,
  staffId?: string
): Promise<TimeSlot[]> => {
  const service = await getService(serviceId);
  const dayOfWeek = date.getDay();
  
  // 1. Get business hours for this day
  const availability = await getAvailability(dayOfWeek, staffId);
  if (!availability.isAvailable) return [];
  
  // 2. Get existing bookings for this day
  const startOfDay = new Date(date);
  startOfDay.setHours(0, 0, 0, 0);
  const endOfDay = new Date(date);
  endOfDay.setHours(23, 59, 59, 999);
  
  const bookingsQuery = query(
    collection(db, 'bookings'),
    where('startTime', '>=', Timestamp.fromDate(startOfDay)),
    where('startTime', '<=', Timestamp.fromDate(endOfDay)),
    where('status', 'in', ['pending', 'confirmed']),
    ...(staffId ? [where('staffId', '==', staffId)] : []),
    orderBy('startTime')
  );
  
  const bookings = await getDocs(bookingsQuery);
  const bookedSlots = bookings.docs.map(doc => ({
    start: doc.data().startTime.toDate(),
    end: doc.data().endTime.toDate(),
  }));
  
  // 3. Get blocked times
  const blockedTimes = await getBlockedTimes(date, staffId);
  
  // 4. Generate available slots
  const slots: TimeSlot[] = [];
  const slotDuration = service.duration + service.bufferBefore + service.bufferAfter;
  
  let currentTime = new Date(date);
  const [startHour, startMin] = availability.startTime.split(':').map(Number);
  currentTime.setHours(startHour, startMin, 0, 0);
  
  const [endHour, endMin] = availability.endTime.split(':').map(Number);
  const endTime = new Date(date);
  endTime.setHours(endHour, endMin, 0, 0);
  
  while (currentTime < endTime) {
    const slotEnd = new Date(currentTime.getTime() + slotDuration * 60000);
    
    const isBooked = bookedSlots.some(booking => 
      (currentTime < booking.end && slotEnd > booking.start)
    );
    
    const isBlocked = blockedTimes.some(blocked =>
      (currentTime < blocked.end && slotEnd > blocked.start)
    );
    
    const isPast = currentTime < new Date();
    
    slots.push({
      start: new Date(currentTime),
      end: slotEnd,
      available: !isBooked && !isBlocked && !isPast,
    });
    
    // Move to next slot (30-minute intervals)
    currentTime.setMinutes(currentTime.getMinutes() + 30);
  }
  
  return slots;
};
```

---

## 🔒 Prevent Double Bookings

```typescript
// services/bookingService.ts
import { runTransaction } from 'firebase/firestore';

export const createBooking = async (
  bookingData: Omit<Booking, 'id' | 'createdAt' | 'updatedAt'>
): Promise<string> => {
  return runTransaction(db, async (transaction) => {
    // 1. Check for conflicts within transaction
    const conflictQuery = query(
      collection(db, 'bookings'),
      where('startTime', '<', bookingData.endTime),
      where('endTime', '>', bookingData.startTime),
      where('status', 'in', ['pending', 'confirmed']),
      ...(bookingData.staffId 
        ? [where('staffId', '==', bookingData.staffId)] 
        : []
      )
    );
    
    const conflicts = await getDocs(conflictQuery);
    
    if (!conflicts.empty) {
      throw new Error('This time slot is no longer available');
    }
    
    // 2. Create booking
    const bookingRef = doc(collection(db, 'bookings'));
    const now = Timestamp.now();
    
    transaction.set(bookingRef, {
      ...bookingData,
      id: bookingRef.id,
      status: 'pending',
      createdAt: now,
      updatedAt: now,
    });
    
    return bookingRef.id;
  });
};
```

---

## 🌍 Time Zone Handling

```typescript
// utils/timezone.ts
import { format, utcToZonedTime, zonedTimeToUtc } from 'date-fns-tz';

// Convert local time to UTC for storage
export const toUTC = (date: Date, timezone: string): Date => {
  return zonedTimeToUtc(date, timezone);
};

// Convert UTC to local time for display
export const toLocalTime = (utcDate: Date, timezone: string): Date => {
  return utcToZonedTime(utcDate, timezone);
};

// Format for display
export const formatInTimezone = (
  date: Date,
  timezone: string,
  formatStr: string = 'PPpp'
): string => {
  const zonedDate = utcToZonedTime(date, timezone);
  return format(zonedDate, formatStr, { timeZone: timezone });
};

// Get user's timezone
export const getUserTimezone = (): string => {
  return Intl.DateTimeFormat().resolvedOptions().timeZone;
};
```

---

## 📧 Booking Notifications

```typescript
// functions/src/bookingTriggers.ts
import * as functions from 'firebase-functions';
import { sendEmail } from './emailService';

export const onBookingCreated = functions.firestore
  .document('bookings/{bookingId}')
  .onCreate(async (snap) => {
    const booking = snap.data();
    
    // Send confirmation to customer
    await sendEmail({
      to: booking.customerEmail,
      template: 'booking-confirmation',
      data: {
        customerName: booking.customerName,
        serviceName: booking.serviceName,
        dateTime: formatInTimezone(
          booking.startTime.toDate(),
          booking.timezone,
          'EEEE, MMMM d, yyyy at h:mm a'
        ),
        bookingId: snap.id,
      },
    });
    
    // Send notification to staff/admin
    await sendEmail({
      to: process.env.ADMIN_EMAIL,
      template: 'new-booking-notification',
      data: booking,
    });
  });

export const onBookingCancelled = functions.firestore
  .document('bookings/{bookingId}')
  .onUpdate(async (change) => {
    const before = change.before.data();
    const after = change.after.data();
    
    if (before.status !== 'cancelled' && after.status === 'cancelled') {
      await sendEmail({
        to: after.customerEmail,
        template: 'booking-cancelled',
        data: after,
      });
    }
  });
```

---

## 🗓️ Calendar Component

```typescript
// components/BookingCalendar.tsx
import { useState } from 'react';
import { DayPicker } from 'react-day-picker';

export const BookingCalendar: React.FC<{
  serviceId: string;
  onSelectSlot: (slot: TimeSlot) => void;
}> = ({ serviceId, onSelectSlot }) => {
  const [selectedDate, setSelectedDate] = useState<Date | undefined>();
  const [slots, setSlots] = useState<TimeSlot[]>([]);
  const [loading, setLoading] = useState(false);

  const handleDateSelect = async (date: Date | undefined) => {
    if (!date) return;
    
    setSelectedDate(date);
    setLoading(true);
    
    try {
      const availableSlots = await getAvailableSlots(date, serviceId);
      setSlots(availableSlots);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="grid md:grid-cols-2 gap-6">
      {/* Calendar */}
      <DayPicker
        mode="single"
        selected={selectedDate}
        onSelect={handleDateSelect}
        disabled={{ before: new Date() }}
        className="border rounded-lg p-4"
      />

      {/* Time Slots */}
      <div>
        <h3 className="font-semibold mb-4">
          {selectedDate 
            ? `Available times for ${format(selectedDate, 'MMMM d')}`
            : 'Select a date'
          }
        </h3>
        
        {loading ? (
          <p>Loading...</p>
        ) : (
          <div className="grid grid-cols-3 gap-2">
            {slots.map((slot) => (
              <button
                key={slot.start.toISOString()}
                onClick={() => onSelectSlot(slot)}
                disabled={!slot.available}
                className={`py-2 px-3 rounded-lg text-sm ${
                  slot.available
                    ? 'bg-primary-100 hover:bg-primary-200 text-primary-700'
                    : 'bg-gray-100 text-gray-400 cursor-not-allowed'
                }`}
              >
                {format(slot.start, 'h:mm a')}
              </button>
            ))}
          </div>
        )}
      </div>
    </div>
  );
};
```

---

## ✅ Checklist

```
Data Design
☐ Times stored in UTC
☐ User timezone captured
☐ Buffer times configured
☐ Status tracking implemented

Availability
☐ Business hours defined
☐ Real-time slot checking
☐ Blocked times supported
☐ Past times disabled

Booking Flow
☐ Double booking prevented (transactions)
☐ Confirmation email sent
☐ Reminder emails scheduled
☐ Cancellation allowed

User Experience
☐ Calendar shows availability
☐ Time slots clearly marked
☐ Loading states shown
☐ Mobile-friendly design

IF ANY ☐ UNCHECKED → Fix before launching
```

---

**Remember: "Store UTC. Display local. Prevent conflicts."** 📅✨

