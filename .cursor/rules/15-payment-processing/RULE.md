---
description: "Payment processing with Stripe, security, PCI compliance, and transaction handling"
alwaysApply: false
---

# Payment Processing Standards

## 🎯 Core Principles

**"Security first. Compliance always. Never store card data."**

Payment rules:
- ✅ Use certified payment processor (Stripe, PayPal)
- ✅ Never handle raw card data
- ✅ PCI DSS compliance through processor
- ✅ Strong Customer Authentication (SCA)
- ✅ Webhook signature verification
- ✅ Idempotent operations

---

## 💳 Stripe Integration

### Setup

```bash
npm install @stripe/stripe-js @stripe/react-stripe-js stripe
```

### Firebase Cloud Function

```typescript
// functions/src/payment.ts

import { onCall, HttpsError } from 'firebase-functions/v2/https';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

/**
 * Create payment intent
 */
export const createPaymentIntent = onCall(async (request) => {
  // Authentication check
  if (!request.auth) {
    throw new HttpsError('unauthenticated', 'User must be authenticated');
  }

  const { amount, currency, metadata } = request.data;

  // Validation
  if (!amount || amount <= 0) {
    throw new HttpsError('invalid-argument', 'Invalid amount');
  }

  if (!currency) {
    throw new HttpsError('invalid-argument', 'Currency is required');
  }

  try {
    // Get customer from Firestore or create new
    const customerId = await getOrCreateStripeCustomer(request.auth.uid);

    // Create payment intent
    const paymentIntent = await stripe.paymentIntents.create({
      amount: Math.round(amount * 100), // Convert to cents
      currency: currency.toLowerCase(),
      customer: customerId,
      metadata: {
        userId: request.auth.uid,
        ...metadata,
      },
      // Enable automatic payment methods
      automatic_payment_methods: {
        enabled: true,
      },
    });

    // Log transaction
    await admin.firestore().collection('transactions').add({
      userId: request.auth.uid,
      paymentIntentId: paymentIntent.id,
      amount,
      currency,
      status: 'pending',
      createdAt: admin.firestore.FieldValue.serverTimestamp(),
    });

    return {
      clientSecret: paymentIntent.client_secret,
      paymentIntentId: paymentIntent.id,
    };
  } catch (error: any) {
    console.error('Payment intent creation failed:', error);
    throw new HttpsError('internal', error.message);
  }
});

/**
 * Get or create Stripe customer
 */
async function getOrCreateStripeCustomer(userId: string): Promise<string> {
  const userDoc = await admin.firestore().collection('users').doc(userId).get();
  const userData = userDoc.data();

  // Return existing customer ID
  if (userData?.stripeCustomerId) {
    return userData.stripeCustomerId;
  }

  // Create new customer
  const customer = await stripe.customers.create({
    email: userData?.email,
    metadata: {
      firebaseUID: userId,
    },
  });

  // Store customer ID
  await userDoc.ref.update({
    stripeCustomerId: customer.id,
  });

  return customer.id;
}
```

### Frontend Payment Form

```typescript
// components/PaymentForm.tsx

import { Elements, PaymentElement, useStripe, useElements } from '@stripe/react-stripe-js';
import { loadStripe } from '@stripe/stripe-js';

const stripePromise = loadStripe(import.meta.env.VITE_STRIPE_PUBLISHABLE_KEY);

interface PaymentFormProps {
  amount: number;
  currency: string;
  onSuccess: (paymentIntentId: string) => void;
  onError: (error: string) => void;
}

function PaymentFormContent({ amount, onSuccess, onError }: PaymentFormProps) {
  const stripe = useStripe();
  const elements = useElements();
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    if (!stripe || !elements) {
      return;
    }

    setLoading(true);

    try {
      // Confirm payment
      const { error, paymentIntent } = await stripe.confirmPayment({
        elements,
        confirmParams: {
          return_url: `${window.location.origin}/payment/success`,
        },
        redirect: 'if_required',
      });

      if (error) {
        onError(error.message || 'Payment failed');
      } else if (paymentIntent && paymentIntent.status === 'succeeded') {
        onSuccess(paymentIntent.id);
      }
    } catch (err: any) {
      onError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label className="block text-sm font-medium mb-2">
          Payment Details
        </label>
        <PaymentElement />
      </div>

      <div className="bg-gray-50 p-4 rounded">
        <div className="flex justify-between items-center">
          <span className="font-medium">Total:</span>
          <span className="text-2xl font-bold">
            ${amount.toFixed(2)}
          </span>
        </div>
      </div>

      <button
        type="submit"
        disabled={!stripe || loading}
        className="w-full bg-primary-600 text-white py-3 rounded-lg hover:bg-primary-700 disabled:opacity-50"
      >
        {loading ? 'Processing...' : `Pay $${amount.toFixed(2)}`}
      </button>

      <p className="text-xs text-gray-500 text-center">
        🔒 Secured by Stripe. We never see your card details.
      </p>
    </form>
  );
}

export const PaymentForm: React.FC<PaymentFormProps> = (props) => {
  const [clientSecret, setClientSecret] = useState<string | null>(null);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // Create payment intent on mount
    const createIntent = async () => {
      try {
        const result = await functions.httpsCallable('createPaymentIntent')({
          amount: props.amount,
          currency: props.currency,
          metadata: {
            // Add relevant metadata
          },
        });

        setClientSecret(result.data.clientSecret);
      } catch (err: any) {
        setError(err.message);
      }
    };

    createIntent();
  }, [props.amount, props.currency]);

  if (error) {
    return <ErrorMessage message={error} />;
  }

  if (!clientSecret) {
    return <LoadingSpinner />;
  }

  return (
    <Elements stripe={stripePromise} options={{ clientSecret }}>
      <PaymentFormContent {...props} />
    </Elements>
  );
};
```

---

## 🔔 Webhook Handling

### Stripe Webhook Endpoint

```typescript
// functions/src/webhooks.ts

import { onRequest } from 'firebase-functions/v2/https';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

export const stripeWebhook = onRequest(async (req, res) => {
  const sig = req.headers['stripe-signature'] as string;

  let event: Stripe.Event;

  try {
    // Verify webhook signature
    event = stripe.webhooks.constructEvent(req.rawBody, sig, webhookSecret);
  } catch (err: any) {
    console.error('Webhook signature verification failed:', err.message);
    res.status(400).send(`Webhook Error: ${err.message}`);
    return;
  }

  // Handle event
  try {
    switch (event.type) {
      case 'payment_intent.succeeded':
        await handlePaymentSucceeded(event.data.object as Stripe.PaymentIntent);
        break;

      case 'payment_intent.payment_failed':
        await handlePaymentFailed(event.data.object as Stripe.PaymentIntent);
        break;

      case 'charge.refunded':
        await handleChargeRefunded(event.data.object as Stripe.Charge);
        break;

      case 'customer.subscription.created':
        await handleSubscriptionCreated(event.data.object as Stripe.Subscription);
        break;

      case 'customer.subscription.deleted':
        await handleSubscriptionCancelled(event.data.object as Stripe.Subscription);
        break;

      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    res.json({ received: true });
  } catch (err: any) {
    console.error('Webhook handler failed:', err);
    res.status(500).send('Webhook handler failed');
  }
});

async function handlePaymentSucceeded(paymentIntent: Stripe.PaymentIntent) {
  const userId = paymentIntent.metadata.userId;

  // Update transaction status
  await admin
    .firestore()
    .collection('transactions')
    .where('paymentIntentId', '==', paymentIntent.id)
    .get()
    .then((snapshot) => {
      snapshot.docs.forEach((doc) => {
        doc.ref.update({
          status: 'succeeded',
          paidAt: admin.firestore.FieldValue.serverTimestamp(),
        });
      });
    });

  // Fulfill order
  await fulfillOrder(userId, paymentIntent.id);

  // Send confirmation email
  await sendPaymentConfirmationEmail(userId, paymentIntent.id);

  // Log for analytics
  await admin.firestore().collection('analytics_events').add({
    event: 'payment_succeeded',
    userId,
    amount: paymentIntent.amount / 100,
    currency: paymentIntent.currency,
    timestamp: admin.firestore.FieldValue.serverTimestamp(),
  });
}

async function handlePaymentFailed(paymentIntent: Stripe.PaymentIntent) {
  const userId = paymentIntent.metadata.userId;

  // Update transaction status
  await admin
    .firestore()
    .collection('transactions')
    .where('paymentIntentId', '==', paymentIntent.id)
    .get()
    .then((snapshot) => {
      snapshot.docs.forEach((doc) => {
        doc.ref.update({
          status: 'failed',
          failureReason: paymentIntent.last_payment_error?.message,
        });
      });
    });

  // Notify user
  await sendPaymentFailedEmail(userId, paymentIntent.id);
}
```

---

## 💰 Pricing & Currency

### Price Formatting

```typescript
// utils/currency.ts

export function formatCurrency(
  amount: number,
  currency: string = 'USD',
  locale: string = 'en-US'
): string {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency: currency.toUpperCase(),
  }).format(amount);
}

// Usage
formatCurrency(1999, 'USD'); // "$1,999.00"
formatCurrency(1999, 'EUR', 'de-DE'); // "1.999,00 €"
```

### Tax Calculation

```typescript
// services/taxService.ts

interface TaxCalculation {
  subtotal: number;
  tax: number;
  total: number;
}

export async function calculateTax(
  amount: number,
  country: string,
  region?: string
): Promise<TaxCalculation> {
  // Use Stripe Tax or TaxJar API
  const taxRate = await getTaxRate(country, region);

  const subtotal = amount;
  const tax = subtotal * taxRate;
  const total = subtotal + tax;

  return { subtotal, tax, total };
}

async function getTaxRate(country: string, region?: string): Promise<number> {
  // Simplified - use actual tax service in production
  const taxRates: Record<string, number> = {
    US_CA: 0.0725, // California
    US_NY: 0.04, // New York
    GB: 0.20, // UK VAT
    DE: 0.19, // Germany VAT
  };

  const key = region ? `${country}_${region}` : country;
  return taxRates[key] || 0;
}
```

---

## 📜 Invoice Generation

```typescript
// services/invoiceService.ts

export async function generateInvoice(transactionId: string): Promise<string> {
  const transaction = await getTransaction(transactionId);
  const user = await getUser(transaction.userId);

  const invoiceData = {
    invoiceNumber: `INV-${Date.now()}`,
    date: new Date().toISOString(),
    customer: {
      name: user.displayName,
      email: user.email,
      address: user.billingAddress,
    },
    items: transaction.items.map((item: any) => ({
      description: item.name,
      quantity: item.quantity,
      price: item.price,
      total: item.quantity * item.price,
    })),
    subtotal: transaction.subtotal,
    tax: transaction.tax,
    total: transaction.total,
    paymentMethod: 'Card ending in ****' + transaction.last4,
  };

  // Generate PDF (using jsPDF or similar)
  const pdfUrl = await generatePDF(invoiceData);

  // Store invoice
  await admin.firestore().collection('invoices').add({
    ...invoiceData,
    pdfUrl,
    transactionId,
    userId: transaction.userId,
    createdAt: admin.firestore.FieldValue.serverTimestamp(),
  });

  return pdfUrl;
}
```

---

## 🔄 Refunds

```typescript
// functions/src/refunds.ts

export const processRefund = onCall(async (request) => {
  if (!request.auth) {
    throw new HttpsError('unauthenticated', 'Authentication required');
  }

  // Check if user is admin
  const user = await admin.firestore().collection('users').doc(request.auth.uid).get();
  if (user.data()?.role !== 'admin') {
    throw new HttpsError('permission-denied', 'Admin access required');
  }

  const { paymentIntentId, amount, reason } = request.data;

  try {
    // Create refund
    const refund = await stripe.refunds.create({
      payment_intent: paymentIntentId,
      amount: amount ? Math.round(amount * 100) : undefined, // Partial or full refund
      reason: reason || 'requested_by_customer',
    });

    // Update transaction
    await admin
      .firestore()
      .collection('transactions')
      .where('paymentIntentId', '==', paymentIntentId)
      .get()
      .then((snapshot) => {
        snapshot.docs.forEach((doc) => {
          doc.ref.update({
            refundId: refund.id,
            refundStatus: refund.status,
            refundAmount: refund.amount / 100,
            refundedAt: admin.firestore.FieldValue.serverTimestamp(),
          });
        });
      });

    // Send refund confirmation email
    const transaction = await getTransaction(paymentIntentId);
    await sendRefundConfirmationEmail(transaction.userId, refund.id);

    return {
      success: true,
      refundId: refund.id,
      amount: refund.amount / 100,
    };
  } catch (error: any) {
    console.error('Refund failed:', error);
    throw new HttpsError('internal', error.message);
  }
});
```

---

## 🔒 Security Best Practices

### Never Store Card Data

```typescript
// ❌ NEVER DO THIS
interface User {
  cardNumber: string; // NEVER!
  cvv: string; // NEVER!
  expiryDate: string; // NEVER!
}

// ✅ GOOD - Only store Stripe customer ID
interface User {
  stripeCustomerId: string; // Stripe handles card storage
  // Last 4 digits and brand (from Stripe) are OK
  paymentMethods?: Array<{
    id: string;
    brand: string; // 'visa', 'mastercard'
    last4: string; // '4242'
    expiryMonth: number;
    expiryYear: number;
  }>;
}
```

### Environment Variables

```env
# .env.production (NEVER commit!)
STRIPE_SECRET_KEY=sk_live_...
STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

### Idempotency

```typescript
// Use idempotency keys to prevent duplicate charges
const paymentIntent = await stripe.paymentIntents.create(
  {
    amount: 2000,
    currency: 'usd',
  },
  {
    idempotencyKey: `payment-${userId}-${orderId}`,
  }
);
```

---

## 💳 Subscription Billing

```typescript
// Create subscription
export const createSubscription = onCall(async (request) => {
  const { priceId, userId } = request.data;

  const customerId = await getOrCreateStripeCustomer(userId);

  const subscription = await stripe.subscriptions.create({
    customer: customerId,
    items: [{ price: priceId }],
    payment_behavior: 'default_incomplete',
    payment_settings: { save_default_payment_method: 'on_subscription' },
    expand: ['latest_invoice.payment_intent'],
  });

  // Store subscription info
  await admin.firestore().collection('subscriptions').doc(subscription.id).set({
    userId,
    customerId,
    subscriptionId: subscription.id,
    status: subscription.status,
    currentPeriodStart: subscription.current_period_start,
    currentPeriodEnd: subscription.current_period_end,
    createdAt: admin.firestore.FieldValue.serverTimestamp(),
  });

  return {
    subscriptionId: subscription.id,
    clientSecret: (subscription.latest_invoice as any).payment_intent.client_secret,
  };
});
```

---

## ✅ Payment Checklist

```
Setup
☐ Stripe account created
☐ API keys configured (test & live)
☐ Webhook endpoint set up
☐ Webhook secret configured
☐ Payment methods enabled

Security
☐ NEVER store card data directly
☐ All payments through Stripe/PayPal
☐ Webhook signatures verified
☐ HTTPS enforced
☐ Environment variables secured
☐ PCI compliance through processor

Frontend
☐ Stripe Elements integrated
☐ Payment form with validation
☐ Loading states during processing
☐ Error handling and display
☐ Success confirmation page
☐ 3D Secure (SCA) supported

Backend
☐ Cloud Function for payment intents
☐ Cloud Function for webhooks
☐ Transaction logging to Firestore
☐ Idempotency keys implemented
☐ Customer creation/retrieval
☐ Email confirmations

Features
☐ Tax calculation (if applicable)
☐ Multiple currencies supported
☐ Invoice generation
☐ Refund functionality
☐ Subscription billing (if applicable)
☐ Payment history for users

Testing
☐ Test mode payments working
☐ Webhook testing complete
☐ Error scenarios tested
☐ Refund testing complete
☐ Edge cases handled

Compliance
☐ Terms of service includes payment terms
☐ Privacy policy covers payment data
☐ Refund policy documented
☐ Tax handling compliant
☐ GDPR/POPIA compliance for payment data

IF ANY ☐ UNCHECKED → Not production-ready!
```

---

## 🚨 AI Decision Gate

```
┌─────────────────────────────────────────────┐
│  PAYMENT PROCESSING CHECKLIST               │
├─────────────────────────────────────────────┤
│  Before implementing payments:              │
│  ☐ Stripe/PayPal account set up            │
│  ☐ Never storing card data                 │
│  ☐ Using certified payment processor       │
│  ☐ Webhooks configured                     │
│  ☐ HTTPS enforced                          │
│                                             │
│  Before processing payments:                │
│  ☐ User authenticated                      │
│  ☐ Amount validated (> 0)                  │
│  ☐ Currency specified                      │
│  ☐ Idempotency key used                    │
│  ☐ Transaction logged                      │
│                                             │
│  Before going live:                         │
│  ☐ Test mode thoroughly tested             │
│  ☐ Error handling complete                 │
│  ☐ Email confirmations working             │
│  ☐ Refund process tested                   │
│  ☐ Security audit passed                   │
│  ☐ Terms of service include payment terms  │
│                                             │
│  IF ANY ☐ UNCHECKED:                        │
│  ➜ DO NOT PROCESS REAL PAYMENTS            │
└─────────────────────────────────────────────┘
```

---

**Remember: "Payment security is not negotiable. Use certified processors and never handle card data directly."** 💳🔒

