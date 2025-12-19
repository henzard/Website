---
description: "Contact form implementation with spam protection and lead capture"
alwaysApply: false
globs: ["**/components/Contact*", "**/pages/contact/**"]
---

# Contact Forms & Lead Capture Standards

## 🎯 Core Principles

- ✅ Simple, focused forms (fewer fields = more conversions)
- ✅ Spam protection (reCAPTCHA or honeypot)
- ✅ Instant confirmation feedback
- ✅ Store leads in Firestore
- ✅ Send notification emails
- ✅ Auto-response to user

---

## 📝 Basic Contact Form

```typescript
// components/ContactForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useGoogleReCaptcha } from 'react-google-recaptcha-v3';

const contactSchema = z.object({
  name: z.string().min(2, 'Name is required'),
  email: z.string().email('Valid email required'),
  phone: z.string().optional(),
  message: z.string().min(10, 'Please provide more details'),
  honeypot: z.string().max(0), // Should be empty
});

type ContactFormData = z.infer<typeof contactSchema>;

export const ContactForm: React.FC = () => {
  const { executeRecaptcha } = useGoogleReCaptcha();
  const [isSubmitted, setIsSubmitted] = useState(false);
  
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    reset,
  } = useForm<ContactFormData>({
    resolver: zodResolver(contactSchema),
  });

  const onSubmit = async (data: ContactFormData) => {
    // Honeypot check
    if (data.honeypot) return;
    
    // Get reCAPTCHA token
    const token = await executeRecaptcha?.('contact_form');
    
    try {
      await submitContactForm({ ...data, recaptchaToken: token });
      setIsSubmitted(true);
      reset();
    } catch (error) {
      alert('Something went wrong. Please try again.');
    }
  };

  if (isSubmitted) {
    return (
      <div className="text-center p-8 bg-green-50 rounded-lg">
        <div className="text-4xl mb-4">✅</div>
        <h3 className="text-xl font-semibold text-green-800">Message Sent!</h3>
        <p className="text-green-600 mt-2">
          We'll get back to you within 24 hours.
        </p>
      </div>
    );
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      {/* Honeypot - hidden from users */}
      <input
        type="text"
        {...register('honeypot')}
        style={{ display: 'none' }}
        tabIndex={-1}
        autoComplete="off"
      />

      <div>
        <label htmlFor="name" className="block font-medium mb-1">Name *</label>
        <input
          id="name"
          {...register('name')}
          className={`w-full px-4 py-2 border rounded-lg ${
            errors.name ? 'border-red-500' : 'border-gray-300'
          }`}
        />
        {errors.name && (
          <p className="text-red-500 text-sm mt-1">{errors.name.message}</p>
        )}
      </div>

      <div>
        <label htmlFor="email" className="block font-medium mb-1">Email *</label>
        <input
          id="email"
          type="email"
          {...register('email')}
          className={`w-full px-4 py-2 border rounded-lg ${
            errors.email ? 'border-red-500' : 'border-gray-300'
          }`}
        />
        {errors.email && (
          <p className="text-red-500 text-sm mt-1">{errors.email.message}</p>
        )}
      </div>

      <div>
        <label htmlFor="phone" className="block font-medium mb-1">Phone</label>
        <input
          id="phone"
          type="tel"
          {...register('phone')}
          className="w-full px-4 py-2 border border-gray-300 rounded-lg"
        />
      </div>

      <div>
        <label htmlFor="message" className="block font-medium mb-1">Message *</label>
        <textarea
          id="message"
          rows={4}
          {...register('message')}
          className={`w-full px-4 py-2 border rounded-lg ${
            errors.message ? 'border-red-500' : 'border-gray-300'
          }`}
        />
        {errors.message && (
          <p className="text-red-500 text-sm mt-1">{errors.message.message}</p>
        )}
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full py-3 bg-primary-600 text-white rounded-lg hover:bg-primary-700 disabled:opacity-50"
      >
        {isSubmitting ? 'Sending...' : 'Send Message'}
      </button>

      <p className="text-xs text-gray-500 text-center">
        Protected by reCAPTCHA. 
        <a href="https://policies.google.com/privacy" className="underline">Privacy</a> · 
        <a href="https://policies.google.com/terms" className="underline">Terms</a>
      </p>
    </form>
  );
};
```

---

## 🔒 Backend Processing

```typescript
// functions/src/contactForm.ts
import * as functions from 'firebase-functions';
import { verifyRecaptcha } from './recaptcha';
import { sendEmail } from './email';

export const submitContact = functions.https.onCall(async (data, context) => {
  const { name, email, phone, message, recaptchaToken } = data;

  // 1. Verify reCAPTCHA
  const recaptchaValid = await verifyRecaptcha(recaptchaToken);
  if (!recaptchaValid || recaptchaValid.score < 0.5) {
    throw new functions.https.HttpsError('permission-denied', 'Spam detected');
  }

  // 2. Save to Firestore
  const contactRef = await admin.firestore().collection('contacts').add({
    name,
    email,
    phone: phone || null,
    message,
    status: 'new',
    source: 'contact_form',
    createdAt: admin.firestore.FieldValue.serverTimestamp(),
    recaptchaScore: recaptchaValid.score,
  });

  // 3. Send notification to admin
  await sendEmail({
    to: process.env.ADMIN_EMAIL!,
    subject: `New Contact: ${name}`,
    template: 'admin-contact-notification',
    data: { name, email, phone, message, id: contactRef.id },
  });

  // 4. Send auto-response to user
  await sendEmail({
    to: email,
    subject: 'Thanks for contacting us!',
    template: 'contact-auto-response',
    data: { name },
  });

  return { success: true };
});
```

---

## 🛡️ reCAPTCHA v3 Setup

```typescript
// app.tsx - Wrap your app
import { GoogleReCaptchaProvider } from 'react-google-recaptcha-v3';

<GoogleReCaptchaProvider reCaptchaKey={import.meta.env.VITE_RECAPTCHA_SITE_KEY}>
  <App />
</GoogleReCaptchaProvider>

// functions - Verify token
export const verifyRecaptcha = async (token: string) => {
  const response = await fetch(
    `https://www.google.com/recaptcha/api/siteverify?secret=${process.env.RECAPTCHA_SECRET}&response=${token}`,
    { method: 'POST' }
  );
  return response.json();
};
```

---

## 📊 Lead Tracking

```typescript
// types/contact.ts
interface Contact {
  id: string;
  name: string;
  email: string;
  phone?: string;
  message: string;
  
  // Tracking
  status: 'new' | 'contacted' | 'converted' | 'closed';
  source: string;
  utm?: {
    source?: string;
    medium?: string;
    campaign?: string;
  };
  
  // Timestamps
  createdAt: Timestamp;
  contactedAt?: Timestamp;
  notes?: string;
}
```

---

## ✅ Checklist

```
Form Design
☐ Minimal required fields
☐ Clear labels and placeholders
☐ Error messages inline
☐ Success confirmation shown

Spam Protection
☐ reCAPTCHA v3 implemented
☐ Honeypot field added
☐ Rate limiting on backend

Lead Management
☐ Contacts stored in Firestore
☐ Admin notification sent
☐ Auto-response to user
☐ UTM parameters captured

IF ANY ☐ UNCHECKED → Fix before launching
```

---

**Remember: "Every contact is a potential customer. Treat them well."** 📧✨

