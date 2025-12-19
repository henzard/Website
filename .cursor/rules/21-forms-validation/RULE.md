---
description: "Form handling, validation, and user input patterns for React applications"
alwaysApply: false
globs: ["**/components/Form*", "**/components/*Form*", "**/hooks/useForm*"]
---

# Forms & Validation Standards

## 🎯 Core Principles

**"Validate early, fail gracefully, guide users to success."**

- ✅ Use React Hook Form + Zod for validation
- ✅ Show errors inline, next to the field
- ✅ Validate on blur (first touch) then on change
- ✅ Disable submit until form is valid
- ✅ Show loading state during submission
- ✅ Handle all error cases gracefully

---

## 📦 Recommended Stack

```bash
npm install react-hook-form zod @hookform/resolvers
```

| Library | Purpose |
|---------|---------|
| `react-hook-form` | Form state management, minimal re-renders |
| `zod` | Schema validation, TypeScript inference |
| `@hookform/resolvers` | Connect Zod to React Hook Form |

---

## 🏗️ Basic Form Pattern

```typescript
// components/ContactForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// 1. Define schema with Zod
const contactSchema = z.object({
  name: z.string()
    .min(2, 'Name must be at least 2 characters')
    .max(100, 'Name is too long'),
  email: z.string()
    .email('Please enter a valid email'),
  message: z.string()
    .min(10, 'Message must be at least 10 characters')
    .max(1000, 'Message is too long'),
});

// 2. Infer TypeScript type from schema
type ContactFormData = z.infer<typeof contactSchema>;

// 3. Create form component
export const ContactForm: React.FC = () => {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting, isValid },
    reset,
  } = useForm<ContactFormData>({
    resolver: zodResolver(contactSchema),
    mode: 'onBlur', // Validate on blur first, then onChange
  });

  const onSubmit = async (data: ContactFormData) => {
    try {
      await submitContact(data);
      reset(); // Clear form on success
      toast.success('Message sent!');
    } catch (error) {
      toast.error('Failed to send message');
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      {/* Name Field */}
      <div>
        <label htmlFor="name" className="block text-sm font-medium mb-1">
          Name
        </label>
        <input
          id="name"
          type="text"
          {...register('name')}
          className={`w-full px-3 py-2 border rounded-lg ${
            errors.name ? 'border-red-500' : 'border-gray-300'
          }`}
        />
        {errors.name && (
          <p className="mt-1 text-sm text-red-600">{errors.name.message}</p>
        )}
      </div>

      {/* Email Field */}
      <div>
        <label htmlFor="email" className="block text-sm font-medium mb-1">
          Email
        </label>
        <input
          id="email"
          type="email"
          {...register('email')}
          className={`w-full px-3 py-2 border rounded-lg ${
            errors.email ? 'border-red-500' : 'border-gray-300'
          }`}
        />
        {errors.email && (
          <p className="mt-1 text-sm text-red-600">{errors.email.message}</p>
        )}
      </div>

      {/* Message Field */}
      <div>
        <label htmlFor="message" className="block text-sm font-medium mb-1">
          Message
        </label>
        <textarea
          id="message"
          rows={4}
          {...register('message')}
          className={`w-full px-3 py-2 border rounded-lg ${
            errors.message ? 'border-red-500' : 'border-gray-300'
          }`}
        />
        {errors.message && (
          <p className="mt-1 text-sm text-red-600">{errors.message.message}</p>
        )}
      </div>

      {/* Submit Button */}
      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full py-3 bg-primary-600 text-white rounded-lg hover:bg-primary-700 disabled:opacity-50"
      >
        {isSubmitting ? 'Sending...' : 'Send Message'}
      </button>
    </form>
  );
};
```

---

## 🧱 Reusable Form Components

### Input Component

```typescript
// components/forms/Input.tsx
import { forwardRef } from 'react';
import { FieldError } from 'react-hook-form';

interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: FieldError;
  helpText?: string;
}

export const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, error, helpText, id, ...props }, ref) => {
    const inputId = id || props.name;
    
    return (
      <div className="space-y-1">
        <label 
          htmlFor={inputId} 
          className="block text-sm font-medium text-gray-700"
        >
          {label}
          {props.required && <span className="text-red-500 ml-1">*</span>}
        </label>
        
        <input
          id={inputId}
          ref={ref}
          className={`
            w-full px-3 py-2 border rounded-lg transition
            focus:ring-2 focus:ring-primary-500 focus:border-transparent
            ${error 
              ? 'border-red-500 bg-red-50' 
              : 'border-gray-300 hover:border-gray-400'
            }
            ${props.disabled ? 'bg-gray-100 cursor-not-allowed' : ''}
          `}
          aria-invalid={error ? 'true' : 'false'}
          aria-describedby={error ? `${inputId}-error` : undefined}
          {...props}
        />
        
        {error && (
          <p id={`${inputId}-error`} className="text-sm text-red-600" role="alert">
            {error.message}
          </p>
        )}
        
        {helpText && !error && (
          <p className="text-sm text-gray-500">{helpText}</p>
        )}
      </div>
    );
  }
);

Input.displayName = 'Input';
```

### Select Component

```typescript
// components/forms/Select.tsx
import { forwardRef } from 'react';
import { FieldError } from 'react-hook-form';

interface Option {
  value: string;
  label: string;
}

interface SelectProps extends React.SelectHTMLAttributes<HTMLSelectElement> {
  label: string;
  options: Option[];
  error?: FieldError;
  placeholder?: string;
}

export const Select = forwardRef<HTMLSelectElement, SelectProps>(
  ({ label, options, error, placeholder, id, ...props }, ref) => {
    const selectId = id || props.name;
    
    return (
      <div className="space-y-1">
        <label 
          htmlFor={selectId} 
          className="block text-sm font-medium text-gray-700"
        >
          {label}
          {props.required && <span className="text-red-500 ml-1">*</span>}
        </label>
        
        <select
          id={selectId}
          ref={ref}
          className={`
            w-full px-3 py-2 border rounded-lg transition
            focus:ring-2 focus:ring-primary-500 focus:border-transparent
            ${error 
              ? 'border-red-500 bg-red-50' 
              : 'border-gray-300 hover:border-gray-400'
            }
          `}
          aria-invalid={error ? 'true' : 'false'}
          {...props}
        >
          {placeholder && (
            <option value="" disabled>
              {placeholder}
            </option>
          )}
          {options.map((option) => (
            <option key={option.value} value={option.value}>
              {option.label}
            </option>
          ))}
        </select>
        
        {error && (
          <p className="text-sm text-red-600" role="alert">
            {error.message}
          </p>
        )}
      </div>
    );
  }
);

Select.displayName = 'Select';
```

### Textarea Component

```typescript
// components/forms/Textarea.tsx
import { forwardRef } from 'react';
import { FieldError } from 'react-hook-form';

interface TextareaProps extends React.TextareaHTMLAttributes<HTMLTextAreaElement> {
  label: string;
  error?: FieldError;
  maxLength?: number;
  showCount?: boolean;
}

export const Textarea = forwardRef<HTMLTextAreaElement, TextareaProps>(
  ({ label, error, maxLength, showCount, id, value, ...props }, ref) => {
    const textareaId = id || props.name;
    const charCount = typeof value === 'string' ? value.length : 0;
    
    return (
      <div className="space-y-1">
        <div className="flex justify-between">
          <label 
            htmlFor={textareaId} 
            className="block text-sm font-medium text-gray-700"
          >
            {label}
            {props.required && <span className="text-red-500 ml-1">*</span>}
          </label>
          
          {showCount && maxLength && (
            <span className={`text-sm ${
              charCount > maxLength ? 'text-red-600' : 'text-gray-500'
            }`}>
              {charCount}/{maxLength}
            </span>
          )}
        </div>
        
        <textarea
          id={textareaId}
          ref={ref}
          value={value}
          maxLength={maxLength}
          className={`
            w-full px-3 py-2 border rounded-lg transition resize-y
            focus:ring-2 focus:ring-primary-500 focus:border-transparent
            ${error 
              ? 'border-red-500 bg-red-50' 
              : 'border-gray-300 hover:border-gray-400'
            }
          `}
          aria-invalid={error ? 'true' : 'false'}
          {...props}
        />
        
        {error && (
          <p className="text-sm text-red-600" role="alert">
            {error.message}
          </p>
        )}
      </div>
    );
  }
);

Textarea.displayName = 'Textarea';
```

### Checkbox Component

```typescript
// components/forms/Checkbox.tsx
import { forwardRef } from 'react';
import { FieldError } from 'react-hook-form';

interface CheckboxProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: FieldError;
}

export const Checkbox = forwardRef<HTMLInputElement, CheckboxProps>(
  ({ label, error, id, ...props }, ref) => {
    const checkboxId = id || props.name;
    
    return (
      <div className="space-y-1">
        <label className="flex items-start gap-3 cursor-pointer">
          <input
            id={checkboxId}
            type="checkbox"
            ref={ref}
            className={`
              mt-1 h-4 w-4 rounded border-gray-300 
              text-primary-600 focus:ring-primary-500
              ${error ? 'border-red-500' : ''}
            `}
            {...props}
          />
          <span className="text-sm text-gray-700">{label}</span>
        </label>
        
        {error && (
          <p className="text-sm text-red-600 ml-7" role="alert">
            {error.message}
          </p>
        )}
      </div>
    );
  }
);

Checkbox.displayName = 'Checkbox';
```

---

## 📋 Common Validation Schemas

```typescript
// schemas/commonSchemas.ts
import { z } from 'zod';

// Email
export const emailSchema = z.string()
  .email('Please enter a valid email address');

// Password
export const passwordSchema = z.string()
  .min(8, 'Password must be at least 8 characters')
  .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
  .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
  .regex(/[0-9]/, 'Password must contain at least one number');

// Phone (South African format)
export const phoneSchema = z.string()
  .regex(/^(\+27|0)[6-8][0-9]{8}$/, 'Please enter a valid phone number');

// URL
export const urlSchema = z.string()
  .url('Please enter a valid URL')
  .optional()
  .or(z.literal(''));

// Price/Amount
export const priceSchema = z.number()
  .min(0, 'Price cannot be negative')
  .multipleOf(0.01, 'Price can only have 2 decimal places');

// Date (future only)
export const futureDateSchema = z.string()
  .refine((date) => new Date(date) > new Date(), {
    message: 'Date must be in the future',
  });

// File upload
export const fileSchema = z.custom<File>()
  .refine((file) => file?.size <= 5 * 1024 * 1024, {
    message: 'File must be less than 5MB',
  })
  .refine((file) => ['image/jpeg', 'image/png', 'image/webp'].includes(file?.type), {
    message: 'Only JPEG, PNG, and WebP images are allowed',
  });

// Confirm password
export const confirmPasswordSchema = (passwordField: string) => 
  z.string().refine((val, ctx) => {
    if (val !== ctx.parent[passwordField]) {
      return false;
    }
    return true;
  }, { message: 'Passwords do not match' });
```

### Registration Schema Example

```typescript
// schemas/registrationSchema.ts
import { z } from 'zod';

export const registrationSchema = z.object({
  firstName: z.string()
    .min(2, 'First name must be at least 2 characters')
    .max(50, 'First name is too long'),
  lastName: z.string()
    .min(2, 'Last name must be at least 2 characters')
    .max(50, 'Last name is too long'),
  email: z.string()
    .email('Please enter a valid email'),
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Must contain uppercase letter')
    .regex(/[0-9]/, 'Must contain a number'),
  confirmPassword: z.string(),
  agreeToTerms: z.boolean()
    .refine((val) => val === true, {
      message: 'You must agree to the terms and conditions',
    }),
}).refine((data) => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword'],
});

export type RegistrationFormData = z.infer<typeof registrationSchema>;
```

---

## 📁 File Upload

### File Upload Component

```typescript
// components/forms/FileUpload.tsx
import { useCallback, useState } from 'react';
import { useDropzone } from 'react-dropzone';
import { FieldError } from 'react-hook-form';

interface FileUploadProps {
  label: string;
  accept?: Record<string, string[]>;
  maxSize?: number; // bytes
  multiple?: boolean;
  error?: FieldError;
  onChange: (files: File[]) => void;
  value?: File[];
}

export const FileUpload: React.FC<FileUploadProps> = ({
  label,
  accept = { 'image/*': ['.jpeg', '.jpg', '.png', '.webp'] },
  maxSize = 5 * 1024 * 1024, // 5MB
  multiple = false,
  error,
  onChange,
  value = [],
}) => {
  const [previews, setPreviews] = useState<string[]>([]);

  const onDrop = useCallback((acceptedFiles: File[]) => {
    onChange(multiple ? [...value, ...acceptedFiles] : acceptedFiles);
    
    // Generate previews for images
    const newPreviews = acceptedFiles.map(file => URL.createObjectURL(file));
    setPreviews(prev => multiple ? [...prev, ...newPreviews] : newPreviews);
  }, [onChange, multiple, value]);

  const { getRootProps, getInputProps, isDragActive, fileRejections } = useDropzone({
    onDrop,
    accept,
    maxSize,
    multiple,
  });

  const removeFile = (index: number) => {
    const newFiles = [...value];
    newFiles.splice(index, 1);
    onChange(newFiles);
    
    // Revoke preview URL
    URL.revokeObjectURL(previews[index]);
    const newPreviews = [...previews];
    newPreviews.splice(index, 1);
    setPreviews(newPreviews);
  };

  return (
    <div className="space-y-2">
      <label className="block text-sm font-medium text-gray-700">
        {label}
      </label>
      
      <div
        {...getRootProps()}
        className={`
          border-2 border-dashed rounded-lg p-6 text-center cursor-pointer transition
          ${isDragActive 
            ? 'border-primary-500 bg-primary-50' 
            : 'border-gray-300 hover:border-gray-400'
          }
          ${error ? 'border-red-500 bg-red-50' : ''}
        `}
      >
        <input {...getInputProps()} />
        
        <div className="text-4xl mb-2">📁</div>
        
        {isDragActive ? (
          <p className="text-primary-600">Drop files here...</p>
        ) : (
          <>
            <p className="text-gray-600">
              Drag & drop files here, or click to select
            </p>
            <p className="text-sm text-gray-500 mt-1">
              Max size: {Math.round(maxSize / 1024 / 1024)}MB
            </p>
          </>
        )}
      </div>

      {/* File Rejections */}
      {fileRejections.length > 0 && (
        <div className="text-sm text-red-600">
          {fileRejections.map(({ file, errors }) => (
            <p key={file.name}>
              {file.name}: {errors.map(e => e.message).join(', ')}
            </p>
          ))}
        </div>
      )}

      {/* Error */}
      {error && (
        <p className="text-sm text-red-600">{error.message}</p>
      )}

      {/* Previews */}
      {previews.length > 0 && (
        <div className="grid grid-cols-3 gap-2 mt-4">
          {previews.map((preview, index) => (
            <div key={preview} className="relative">
              <img
                src={preview}
                alt={`Preview ${index + 1}`}
                className="w-full h-24 object-cover rounded-lg"
              />
              <button
                type="button"
                onClick={() => removeFile(index)}
                className="absolute -top-2 -right-2 w-6 h-6 bg-red-500 text-white rounded-full text-sm"
              >
                ×
              </button>
            </div>
          ))}
        </div>
      )}
    </div>
  );
};
```

### With React Hook Form

```typescript
// Using FileUpload with react-hook-form
import { Controller } from 'react-hook-form';

<Controller
  name="profileImage"
  control={control}
  render={({ field: { onChange, value }, fieldState: { error } }) => (
    <FileUpload
      label="Profile Image"
      accept={{ 'image/*': ['.jpeg', '.jpg', '.png'] }}
      maxSize={2 * 1024 * 1024}
      error={error}
      onChange={(files) => onChange(files[0])}
      value={value ? [value] : []}
    />
  )}
/>
```

---

## 📝 Multi-Step Forms

```typescript
// components/MultiStepForm.tsx
import { useState } from 'react';
import { useForm, FormProvider } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

interface MultiStepFormProps {
  steps: {
    title: string;
    schema: z.ZodSchema;
    component: React.ComponentType;
  }[];
  onComplete: (data: any) => Promise<void>;
}

export const MultiStepForm: React.FC<MultiStepFormProps> = ({
  steps,
  onComplete,
}) => {
  const [currentStep, setCurrentStep] = useState(0);
  const [formData, setFormData] = useState({});
  
  const currentSchema = steps[currentStep].schema;
  const CurrentStepComponent = steps[currentStep].component;
  
  const methods = useForm({
    resolver: zodResolver(currentSchema),
    defaultValues: formData,
    mode: 'onBlur',
  });

  const isLastStep = currentStep === steps.length - 1;

  const handleNext = async (stepData: any) => {
    const newFormData = { ...formData, ...stepData };
    setFormData(newFormData);
    
    if (isLastStep) {
      await onComplete(newFormData);
    } else {
      setCurrentStep(prev => prev + 1);
    }
  };

  const handleBack = () => {
    setCurrentStep(prev => prev - 1);
  };

  return (
    <div className="max-w-2xl mx-auto">
      {/* Progress Indicator */}
      <div className="mb-8">
        <div className="flex justify-between">
          {steps.map((step, index) => (
            <div
              key={index}
              className={`flex items-center ${
                index !== steps.length - 1 ? 'flex-1' : ''
              }`}
            >
              <div
                className={`w-10 h-10 rounded-full flex items-center justify-center ${
                  index <= currentStep
                    ? 'bg-primary-600 text-white'
                    : 'bg-gray-200 text-gray-600'
                }`}
              >
                {index < currentStep ? '✓' : index + 1}
              </div>
              
              {index !== steps.length - 1 && (
                <div
                  className={`flex-1 h-1 mx-2 ${
                    index < currentStep ? 'bg-primary-600' : 'bg-gray-200'
                  }`}
                />
              )}
            </div>
          ))}
        </div>
        
        <div className="flex justify-between mt-2">
          {steps.map((step, index) => (
            <span
              key={index}
              className={`text-sm ${
                index <= currentStep ? 'text-primary-600' : 'text-gray-500'
              }`}
            >
              {step.title}
            </span>
          ))}
        </div>
      </div>

      {/* Form */}
      <FormProvider {...methods}>
        <form onSubmit={methods.handleSubmit(handleNext)} className="space-y-6">
          <CurrentStepComponent />

          <div className="flex justify-between pt-4">
            {currentStep > 0 && (
              <button
                type="button"
                onClick={handleBack}
                className="px-6 py-2 border border-gray-300 rounded-lg hover:bg-gray-50"
              >
                Back
              </button>
            )}
            
            <button
              type="submit"
              disabled={methods.formState.isSubmitting}
              className="ml-auto px-6 py-2 bg-primary-600 text-white rounded-lg hover:bg-primary-700 disabled:opacity-50"
            >
              {methods.formState.isSubmitting
                ? 'Processing...'
                : isLastStep
                ? 'Complete'
                : 'Next'}
            </button>
          </div>
        </form>
      </FormProvider>
    </div>
  );
};
```

---

## ⚡ Async Validation

```typescript
// Checking if email is already registered
const registrationSchema = z.object({
  email: z.string()
    .email('Invalid email')
    .refine(async (email) => {
      const exists = await checkEmailExists(email);
      return !exists;
    }, { message: 'This email is already registered' }),
  // ... other fields
});

// Using with debounce for live validation
import { useDebouncedCallback } from 'use-debounce';

const EmailInput: React.FC = () => {
  const [isChecking, setIsChecking] = useState(false);
  const [emailError, setEmailError] = useState('');
  
  const checkEmail = useDebouncedCallback(async (email: string) => {
    if (!email || !email.includes('@')) return;
    
    setIsChecking(true);
    try {
      const exists = await checkEmailExists(email);
      if (exists) {
        setEmailError('This email is already registered');
      } else {
        setEmailError('');
      }
    } finally {
      setIsChecking(false);
    }
  }, 500);

  return (
    <div>
      <Input
        label="Email"
        onChange={(e) => {
          register('email').onChange(e);
          checkEmail(e.target.value);
        }}
      />
      {isChecking && <span className="text-gray-500">Checking...</span>}
      {emailError && <span className="text-red-600">{emailError}</span>}
    </div>
  );
};
```

---

## 🛡️ Form Security

### Honeypot Field (Anti-Spam)

```typescript
// Hidden field to catch bots
<input
  type="text"
  name="website" // Bots often fill this
  style={{ display: 'none' }}
  tabIndex={-1}
  autoComplete="off"
  {...register('honeypot')}
/>

// In submit handler
const onSubmit = async (data: FormData) => {
  if (data.honeypot) {
    // Bot detected, silently ignore
    console.warn('Bot submission detected');
    return;
  }
  // Process real submission
};
```

### Rate Limiting Submissions

```typescript
// hooks/useSubmitThrottle.ts
import { useState, useRef } from 'react';

export const useSubmitThrottle = (cooldownMs: number = 3000) => {
  const [isThrottled, setIsThrottled] = useState(false);
  const lastSubmitRef = useRef<number>(0);

  const canSubmit = (): boolean => {
    const now = Date.now();
    if (now - lastSubmitRef.current < cooldownMs) {
      setIsThrottled(true);
      return false;
    }
    return true;
  };

  const recordSubmit = () => {
    lastSubmitRef.current = Date.now();
    setIsThrottled(false);
  };

  return { canSubmit, recordSubmit, isThrottled };
};
```

### Input Sanitization

```typescript
// utils/sanitize.ts
import DOMPurify from 'dompurify';

// Remove HTML tags and dangerous content
export const sanitizeInput = (input: string): string => {
  return DOMPurify.sanitize(input, { ALLOWED_TAGS: [] });
};

// In form submission
const onSubmit = async (data: FormData) => {
  const sanitizedData = {
    ...data,
    name: sanitizeInput(data.name),
    message: sanitizeInput(data.message),
  };
  
  await submitForm(sanitizedData);
};
```

---

## ♿ Accessibility

### Required Labels

```typescript
// ✅ GOOD - Always associate labels
<label htmlFor="email">Email</label>
<input id="email" type="email" {...register('email')} />

// ❌ BAD - Missing htmlFor/id association
<label>Email</label>
<input type="email" {...register('email')} />
```

### Error Announcements

```typescript
// ✅ GOOD - Screen readers announce errors
<input
  aria-invalid={errors.email ? 'true' : 'false'}
  aria-describedby={errors.email ? 'email-error' : undefined}
  {...register('email')}
/>
{errors.email && (
  <p id="email-error" role="alert" className="text-red-600">
    {errors.email.message}
  </p>
)}
```

### Focus Management

```typescript
// Focus first error field on submit failure
const onSubmit = async (data: FormData) => {
  try {
    await submitForm(data);
  } catch {
    // Focus first error field
    const firstError = Object.keys(errors)[0];
    if (firstError) {
      document.getElementById(firstError)?.focus();
    }
  }
};
```

---

## ✅ Checklist

```
Setup
☐ react-hook-form installed
☐ zod installed
☐ @hookform/resolvers installed

Form Structure
☐ Every input has a label
☐ Labels use htmlFor matching input id
☐ Required fields are marked
☐ Error messages display inline

Validation
☐ Schema defined with Zod
☐ Validation mode set (onBlur recommended)
☐ Custom error messages are user-friendly
☐ Async validation debounced

User Experience
☐ Submit button shows loading state
☐ Form clears on success
☐ Success feedback shown
☐ Error feedback shown
☐ Focus moves to first error on failure

Accessibility
☐ aria-invalid on error fields
☐ aria-describedby links to error messages
☐ role="alert" on error messages
☐ Required fields indicated visually and in aria

Security
☐ Honeypot field for anti-spam
☐ Rate limiting on submissions
☐ Input sanitization before saving
☐ Server-side validation (never trust client only)

IF ANY ☐ UNCHECKED → Fix before deploying
```

---

## 🚨 AI Decision Gate

```
┌─────────────────────────────────────────────┐
│  FORM IMPLEMENTATION CHECKLIST              │
├─────────────────────────────────────────────┤
│  Before building form:                      │
│  ☐ Zod schema defined with all fields      │
│  ☐ TypeScript type inferred from schema    │
│  ☐ Error messages are user-friendly        │
│  ☐ Required fields clearly marked          │
│                                             │
│  For each input:                            │
│  ☐ Has associated label (htmlFor)          │
│  ☐ Has unique id                           │
│  ☐ Shows error message inline              │
│  ☐ aria-invalid when errored               │
│                                             │
│  For submission:                            │
│  ☐ Loading state shown                     │
│  ☐ Button disabled while submitting        │
│  ☐ Success feedback displayed              │
│  ☐ Error handling for failures             │
│                                             │
│  For security:                              │
│  ☐ Server also validates (not just client) │
│  ☐ Input sanitized before storage          │
│  ☐ Spam protection considered              │
│                                             │
│  IF ANY ☐ UNCHECKED:                        │
│  ➜ ADDRESS BEFORE IMPLEMENTING              │
└─────────────────────────────────────────────┘
```

---

**Remember: "Good forms guide users to success, not frustrate them into leaving."** 📝✨

