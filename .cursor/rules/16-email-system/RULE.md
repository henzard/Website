---
description: "Email system, transactional emails, templates, and deliverability best practices"
alwaysApply: false
---

# Email System Standards

## 🎯 Core Principles

**"Emails must be reliable, beautiful, and compliant."**

Email requirements:
- ✅ Use dedicated email service (SendGrid, Mailgun, Amazon SES)
- ✅ Never send from personal Gmail/Yahoo
- ✅ HTML + Plain text versions
- ✅ Mobile responsive templates
- ✅ Unsubscribe link in every marketing email
- ✅ GDPR/CAN-SPAM compliant

---

## 📧 Email Service Setup

### SendGrid Integration

```bash
npm install @sendgrid/mail
```

```typescript
// functions/src/email/sendgridClient.ts

import sgMail from '@sendgrid/mail';

sgMail.setApiKey(process.env.SENDGRID_API_KEY!);

export interface EmailOptions {
  to: string;
  from?: string;
  subject: string;
  html: string;
  text?: string;
  templateId?: string;
  dynamicTemplateData?: Record<string, any>;
  attachments?: Array<{
    content: string;
    filename: string;
    type: string;
  }>;
}

export async function sendEmail(options: EmailOptions): Promise<void> {
  const msg = {
    to: options.to,
    from: options.from || process.env.FROM_EMAIL || 'noreply@yoursite.com',
    subject: options.subject,
    html: options.html,
    text: options.text || extractTextFromHtml(options.html),
    ...(options.templateId && {
      templateId: options.templateId,
      dynamicTemplateData: options.dynamicTemplateData,
    }),
    ...(options.attachments && {
      attachments: options.attachments,
    }),
  };

  try {
    await sgMail.send(msg);
    console.log(`✅ Email sent to ${options.to}`);
  } catch (error: any) {
    console.error('❌ Email send failed:', error);
    throw new Error(`Email send failed: ${error.message}`);
  }
}

function extractTextFromHtml(html: string): string {
  // Simple HTML to text conversion
  return html
    .replace(/<[^>]*>/g, '')
    .replace(/&nbsp;/g, ' ')
    .replace(/&amp;/g, '&')
    .replace(/&lt;/g, '<')
    .replace(/&gt;/g, '>')
    .trim();
}
```

---

## 📨 Transactional Emails

### Welcome Email

```typescript
// functions/src/email/templates/welcomeEmail.ts

export async function sendWelcomeEmail(userId: string): Promise<void> {
  const user = await admin.firestore().collection('users').doc(userId).get();
  const userData = user.data();

  if (!userData?.email) {
    throw new Error('User email not found');
  }

  await sendEmail({
    to: userData.email,
    subject: 'Welcome to YourApp! 🎉',
    html: generateWelcomeEmailHtml(userData.displayName),
  });

  // Log email sent
  await admin.firestore().collection('email_logs').add({
    userId,
    type: 'welcome',
    to: userData.email,
    sentAt: admin.firestore.FieldValue.serverTimestamp(),
    status: 'sent',
  });
}

function generateWelcomeEmailHtml(displayName: string): string {
  return `
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Welcome to YourApp</title>
</head>
<body style="margin: 0; padding: 0; font-family: Arial, sans-serif; background-color: #f4f4f4;">
  <table role="presentation" style="width: 100%; border-collapse: collapse;">
    <tr>
      <td align="center" style="padding: 40px 0;">
        <table role="presentation" style="width: 600px; border-collapse: collapse; background-color: #ffffff; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
          
          <!-- Header -->
          <tr>
            <td style="padding: 40px 30px; text-align: center; background-color: #3B82F6; border-radius: 8px 8px 0 0;">
              <h1 style="margin: 0; color: #ffffff; font-size: 28px;">Welcome to YourApp!</h1>
            </td>
          </tr>

          <!-- Content -->
          <tr>
            <td style="padding: 40px 30px;">
              <p style="margin: 0 0 20px; font-size: 16px; line-height: 1.6; color: #333333;">
                Hi ${displayName},
              </p>
              <p style="margin: 0 0 20px; font-size: 16px; line-height: 1.6; color: #333333;">
                Welcome to YourApp! We're excited to have you on board.
              </p>
              <p style="margin: 0 0 30px; font-size: 16px; line-height: 1.6; color: #333333;">
                Here's how to get started:
              </p>

              <!-- Getting Started Steps -->
              <table role="presentation" style="width: 100%; margin: 0 0 30px;">
                <tr>
                  <td style="padding: 15px; background-color: #f8f9fa; border-radius: 4px; margin-bottom: 10px;">
                    <strong style="color: #3B82F6;">Step 1:</strong> Complete your profile
                  </td>
                </tr>
                <tr>
                  <td style="padding: 15px; background-color: #f8f9fa; border-radius: 4px; margin-bottom: 10px;">
                    <strong style="color: #3B82F6;">Step 2:</strong> Explore features
                  </td>
                </tr>
                <tr>
                  <td style="padding: 15px; background-color: #f8f9fa; border-radius: 4px;">
                    <strong style="color: #3B82F6;">Step 3:</strong> Make your first booking
                  </td>
                </tr>
              </table>

              <!-- CTA Button -->
              <table role="presentation" style="width: 100%;">
                <tr>
                  <td align="center">
                    <a href="https://yourapp.com/dashboard" style="display: inline-block; padding: 14px 32px; background-color: #3B82F6; color: #ffffff; text-decoration: none; border-radius: 6px; font-weight: bold; font-size: 16px;">
                      Get Started
                    </a>
                  </td>
                </tr>
              </table>
            </td>
          </tr>

          <!-- Footer -->
          <tr>
            <td style="padding: 30px; background-color: #f8f9fa; border-radius: 0 0 8px 8px; text-align: center;">
              <p style="margin: 0 0 10px; font-size: 14px; color: #666666;">
                Need help? <a href="https://yourapp.com/support" style="color: #3B82F6;">Contact Support</a>
              </p>
              <p style="margin: 0; font-size: 12px; color: #999999;">
                © 2025 YourApp. All rights reserved.
              </p>
            </td>
          </tr>

        </table>
      </td>
    </tr>
  </table>
</body>
</html>
  `;
}
```

### Email Verification

```typescript
// functions/src/email/templates/verificationEmail.ts

export async function sendVerificationEmail(
  userId: string,
  verificationLink: string
): Promise<void> {
  const user = await admin.firestore().collection('users').doc(userId).get();
  const userData = user.data();

  if (!userData?.email) {
    throw new Error('User email not found');
  }

  await sendEmail({
    to: userData.email,
    subject: 'Verify your email address',
    html: `
<!DOCTYPE html>
<html>
<body style="font-family: Arial, sans-serif; line-height: 1.6; color: #333;">
  <div style="max-width: 600px; margin: 0 auto; padding: 20px;">
    <h2>Verify your email address</h2>
    <p>Hi ${userData.displayName},</p>
    <p>Please verify your email address by clicking the button below:</p>
    
    <div style="margin: 30px 0;">
      <a href="${verificationLink}" 
         style="display: inline-block; padding: 12px 24px; background-color: #3B82F6; color: white; text-decoration: none; border-radius: 6px;">
        Verify Email
      </a>
    </div>

    <p>Or copy and paste this link into your browser:</p>
    <p style="word-break: break-all; color: #666;">
      ${verificationLink}
    </p>

    <p style="margin-top: 30px; font-size: 14px; color: #666;">
      This link expires in 24 hours. If you didn't create an account, you can safely ignore this email.
    </p>
  </div>
</body>
</html>
    `,
  });
}
```

### Password Reset

```typescript
// functions/src/email/templates/passwordResetEmail.ts

export async function sendPasswordResetEmail(
  email: string,
  resetLink: string
): Promise<void> {
  await sendEmail({
    to: email,
    subject: 'Reset your password',
    html: `
<!DOCTYPE html>
<html>
<body style="font-family: Arial, sans-serif; line-height: 1.6; color: #333;">
  <div style="max-width: 600px; margin: 0 auto; padding: 20px;">
    <h2>Reset your password</h2>
    <p>We received a request to reset your password.</p>
    <p>Click the button below to create a new password:</p>
    
    <div style="margin: 30px 0;">
      <a href="${resetLink}" 
         style="display: inline-block; padding: 12px 24px; background-color: #3B82F6; color: white; text-decoration: none; border-radius: 6px;">
        Reset Password
      </a>
    </div>

    <p style="margin-top: 30px; font-size: 14px; color: #666;">
      This link expires in 1 hour. If you didn't request a password reset, you can safely ignore this email.
    </p>

    <p style="font-size: 14px; color: #999;">
      For security, never share this link with anyone.
    </p>
  </div>
</body>
</html>
    `,
  });
}
```

### Booking Confirmation

```typescript
// functions/src/email/templates/bookingConfirmationEmail.ts

export async function sendBookingConfirmationEmail(
  userId: string,
  bookingId: string
): Promise<void> {
  const [user, booking] = await Promise.all([
    admin.firestore().collection('users').doc(userId).get(),
    admin.firestore().collection('bookings').doc(bookingId).get(),
  ]);

  const userData = user.data();
  const bookingData = booking.data();

  if (!userData?.email || !bookingData) {
    throw new Error('User or booking data not found');
  }

  const bookingDate = bookingData.date.toDate();

  await sendEmail({
    to: userData.email,
    subject: `Booking Confirmed - ${bookingData.serviceName}`,
    html: `
<!DOCTYPE html>
<html>
<body style="font-family: Arial, sans-serif; line-height: 1.6; color: #333;">
  <div style="max-width: 600px; margin: 0 auto; padding: 20px;">
    <h2 style="color: #10B981;">✓ Booking Confirmed!</h2>
    
    <p>Hi ${userData.displayName},</p>
    <p>Your booking has been confirmed. Here are the details:</p>

    <div style="background-color: #f8f9fa; padding: 20px; border-radius: 8px; margin: 20px 0;">
      <table style="width: 100%; border-collapse: collapse;">
        <tr>
          <td style="padding: 8px 0; font-weight: bold;">Service:</td>
          <td style="padding: 8px 0;">${bookingData.serviceName}</td>
        </tr>
        <tr>
          <td style="padding: 8px 0; font-weight: bold;">Date:</td>
          <td style="padding: 8px 0;">${bookingDate.toLocaleDateString()}</td>
        </tr>
        <tr>
          <td style="padding: 8px 0; font-weight: bold;">Time:</td>
          <td style="padding: 8px 0;">${bookingDate.toLocaleTimeString()}</td>
        </tr>
        <tr>
          <td style="padding: 8px 0; font-weight: bold;">Duration:</td>
          <td style="padding: 8px 0;">${bookingData.duration} minutes</td>
        </tr>
        <tr>
          <td style="padding: 8px 0; font-weight: bold;">Price:</td>
          <td style="padding: 8px 0;">$${bookingData.price.toFixed(2)}</td>
        </tr>
        <tr>
          <td style="padding: 8px 0; font-weight: bold;">Booking ID:</td>
          <td style="padding: 8px 0; font-family: monospace;">${bookingId}</td>
        </tr>
      </table>
    </div>

    <div style="margin: 30px 0;">
      <a href="https://yourapp.com/bookings/${bookingId}" 
         style="display: inline-block; padding: 12px 24px; background-color: #3B82F6; color: white; text-decoration: none; border-radius: 6px; margin-right: 10px;">
        View Booking
      </a>
      <a href="https://yourapp.com/calendar" 
         style="display: inline-block; padding: 12px 24px; background-color: #ffffff; color: #3B82F6; text-decoration: none; border-radius: 6px; border: 2px solid #3B82F6;">
        Add to Calendar
      </a>
    </div>

    <p style="margin-top: 30px; font-size: 14px; color: #666;">
      Need to cancel or reschedule? <a href="https://yourapp.com/bookings/${bookingId}">Manage your booking</a>
    </p>
  </div>
</body>
</html>
    `,
  });
}
```

---

## 📬 Email Template System

### Base Template

```typescript
// functions/src/email/templates/baseTemplate.ts

interface EmailTemplateOptions {
  title: string;
  preheader?: string;
  content: string;
  ctaText?: string;
  ctaLink?: string;
}

export function generateBaseEmailTemplate(options: EmailTemplateOptions): string {
  return `
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="x-apple-disable-message-reformatting">
  <title>${options.title}</title>
  <!--[if mso]>
  <style type="text/css">
    body, table, td {font-family: Arial, Helvetica, sans-serif !important;}
  </style>
  <![endif]-->
</head>
<body style="margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; background-color: #f4f4f4;">
  <!-- Preheader Text -->
  ${
    options.preheader
      ? `<div style="display: none; max-height: 0; overflow: hidden;">${options.preheader}</div>`
      : ''
  }

  <!-- Email Container -->
  <table role="presentation" style="width: 100%; border-collapse: collapse;">
    <tr>
      <td align="center" style="padding: 40px 20px;">
        
        <!-- Email Content -->
        <table role="presentation" style="max-width: 600px; width: 100%; border-collapse: collapse; background-color: #ffffff; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
          
          <!-- Logo Header -->
          <tr>
            <td style="padding: 40px 30px; text-align: center;">
              <img src="https://yourapp.com/logo.png" alt="YourApp Logo" style="width: 120px; height: auto;">
            </td>
          </tr>

          <!-- Main Content -->
          <tr>
            <td style="padding: 0 30px 40px;">
              ${options.content}
            </td>
          </tr>

          <!-- CTA Button (if provided) -->
          ${
            options.ctaText && options.ctaLink
              ? `
          <tr>
            <td align="center" style="padding: 0 30px 40px;">
              <a href="${options.ctaLink}" style="display: inline-block; padding: 14px 32px; background-color: #3B82F6; color: #ffffff; text-decoration: none; border-radius: 6px; font-weight: bold; font-size: 16px;">
                ${options.ctaText}
              </a>
            </td>
          </tr>
          `
              : ''
          }

          <!-- Footer -->
          <tr>
            <td style="padding: 30px; background-color: #f8f9fa; border-radius: 0 0 8px 8px;">
              <table role="presentation" style="width: 100%;">
                <tr>
                  <td style="text-align: center; padding-bottom: 20px;">
                    <!-- Social Links -->
                    <a href="https://twitter.com/yourapp" style="display: inline-block; margin: 0 10px;">
                      <img src="https://yourapp.com/icons/twitter.png" alt="Twitter" style="width: 24px; height: 24px;">
                    </a>
                    <a href="https://facebook.com/yourapp" style="display: inline-block; margin: 0 10px;">
                      <img src="https://yourapp.com/icons/facebook.png" alt="Facebook" style="width: 24px; height: 24px;">
                    </a>
                  </td>
                </tr>
                <tr>
                  <td style="text-align: center; font-size: 14px; color: #666666; padding-bottom: 10px;">
                    <p style="margin: 0 0 10px;">
                      <a href="https://yourapp.com/help" style="color: #3B82F6; text-decoration: none;">Help Center</a> •
                      <a href="https://yourapp.com/contact" style="color: #3B82F6; text-decoration: none;">Contact Us</a>
                    </p>
                  </td>
                </tr>
                <tr>
                  <td style="text-align: center; font-size: 12px; color: #999999;">
                    <p style="margin: 0 0 10px;">© 2025 YourApp. All rights reserved.</p>
                    <p style="margin: 0;">123 Main St, City, State 12345</p>
                  </td>
                </tr>
              </table>
            </td>
          </tr>

        </table>

      </td>
    </tr>
  </table>
</body>
</html>
  `;
}
```

---

## 🔕 Unsubscribe Management

```typescript
// functions/src/email/unsubscribe.ts

export const handleUnsubscribe = onCall(async (request) => {
  const { userId, emailType } = request.data;

  if (!userId || !emailType) {
    throw new HttpsError('invalid-argument', 'Missing required fields');
  }

  // Update user preferences
  await admin
    .firestore()
    .collection('users')
    .doc(userId)
    .update({
      [`emailPreferences.${emailType}`]: false,
      updatedAt: admin.firestore.FieldValue.serverTimestamp(),
    });

  // Log unsubscribe
  await admin.firestore().collection('unsubscribes').add({
    userId,
    emailType,
    timestamp: admin.firestore.FieldValue.serverTimestamp(),
  });

  return { success: true };
});

// Add unsubscribe link to marketing emails
function addUnsubscribeLink(userId: string, emailType: string): string {
  const unsubscribeUrl = `https://yourapp.com/unsubscribe?userId=${userId}&type=${emailType}`;
  
  return `
    <p style="margin: 20px 0 0; font-size: 12px; color: #999999; text-align: center;">
      Don't want to receive these emails?
      <a href="${unsubscribeUrl}" style="color: #3B82F6;">Unsubscribe</a>
    </p>
  `;
}
```

---

## 📊 Email Analytics

```typescript
// Track email opens and clicks

export async function trackEmailOpen(emailId: string): Promise<void> {
  await admin
    .firestore()
    .collection('email_logs')
    .doc(emailId)
    .update({
      openedAt: admin.firestore.FieldValue.serverTimestamp(),
      opens: admin.firestore.FieldValue.increment(1),
    });
}

export async function trackEmailClick(emailId: string, linkUrl: string): Promise<void> {
  await admin.firestore().collection('email_clicks').add({
    emailId,
    linkUrl,
    clickedAt: admin.firestore.FieldValue.serverTimestamp(),
  });
}
```

---

## ✅ Email System Checklist

```
Setup
☐ Email service configured (SendGrid/Mailgun)
☐ Domain verified
☐ SPF record set up
☐ DKIM configured
☐ DMARC policy set
☐ FROM email configured

Templates
☐ Welcome email
☐ Email verification
☐ Password reset
☐ Booking confirmation
☐ Base template with branding
☐ HTML + plain text versions

Transactional Emails
☐ Welcome email on signup
☐ Email verification on signup
☐ Password reset on request
☐ Booking/order confirmations
☐ Payment receipts
☐ Account notifications

Compliance
☐ Unsubscribe link in marketing emails
☐ Physical address in footer
☐ CAN-SPAM compliance
☐ GDPR compliance (if EU users)
☐ User email preferences stored

Deliverability
☐ Sender reputation monitored
☐ Bounce rate < 5%
☐ Complaint rate < 0.1%
☐ Email warming strategy (if new)
☐ Test emails before sending to lists

Testing
☐ Test in major email clients
☐ Test on mobile devices
☐ Test all links work
☐ Test unsubscribe works
☐ Test with spam checkers

Analytics
☐ Track email sends
☐ Track email opens
☐ Track link clicks
☐ Monitor bounce rates
☐ Monitor unsubscribe rates

IF ANY ☐ UNCHECKED → Email system not ready
```

---

## 🚨 AI Decision Gate

```
┌─────────────────────────────────────────────┐
│  EMAIL SYSTEM CHECKLIST                     │
├─────────────────────────────────────────────┤
│  Before sending emails:                     │
│  ☐ Email service configured                │
│  ☐ Domain verified with provider           │
│  ☐ SPF/DKIM/DMARC records set              │
│  ☐ FROM address is professional            │
│  ☐ Template is mobile-responsive           │
│                                             │
│  For transactional emails:                  │
│  ☐ HTML + plain text versions              │
│  ☐ Subject line is clear                   │
│  ☐ All links work                          │
│  ☐ Branding is consistent                  │
│  ☐ Email is logged to Firestore            │
│                                             │
│  For marketing emails:                      │
│  ☐ Unsubscribe link included               │
│  ☐ Physical address in footer              │
│  ☐ User has opted in                       │
│  ☐ Not sent too frequently                 │
│                                             │
│  IF ANY ☐ UNCHECKED:                        │
│  ➜ FIX BEFORE SENDING                      │
└─────────────────────────────────────────────┘
```

---

**Remember: "Email is still one of the most effective communication channels. Do it right."** 📧✨

