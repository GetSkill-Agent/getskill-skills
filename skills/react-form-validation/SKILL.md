# React Form Validation

> Build forms with client-side validation using React Hook Form and Zod.

## Trigger

- User needs to create a form with validation
- User needs a signup/login form
- User wants to handle user input with validation

## Prerequisites

```bash
npm install react-hook-form zod @hookform/resolvers
```

## Instructions

### Step 1: Define Validation Schema

```typescript
// schemas/auth.ts
import { z } from 'zod';

export const loginSchema = z.object({
  email: z
    .string()
    .min(1, 'Email is required')
    .email('Invalid email address'),
  password: z
    .string()
    .min(1, 'Password is required')
    .min(8, 'Password must be at least 8 characters'),
});

export type LoginFormData = z.infer<typeof loginSchema>;

export const signupSchema = z
  .object({
    name: z
      .string()
      .min(1, 'Name is required')
      .min(2, 'Name must be at least 2 characters'),
    email: z
      .string()
      .min(1, 'Email is required')
      .email('Invalid email address'),
    password: z
      .string()
      .min(1, 'Password is required')
      .min(8, 'Password must be at least 8 characters')
      .regex(
        /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
        'Password must contain uppercase, lowercase, and number'
      ),
    confirmPassword: z.string().min(1, 'Please confirm your password'),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: 'Passwords do not match',
    path: ['confirmPassword'],
  });

export type SignupFormData = z.infer<typeof signupSchema>;
```

### Step 2: Create Reusable Input Component

```tsx
// components/FormInput.tsx
import { forwardRef } from 'react';
import { FieldError } from 'react-hook-form';

interface FormInputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: FieldError;
}

export const FormInput = forwardRef<HTMLInputElement, FormInputProps>(
  ({ label, error, ...props }, ref) => {
    return (
      <div className="mb-4">
        <label className="block text-sm font-medium mb-1">
          {label}
          {props.required && <span className="text-red-500 ml-1">*</span>}
        </label>
        <input
          ref={ref}
          className={`
            w-full px-3 py-2 border rounded-lg
            focus:outline-none focus:ring-2 focus:ring-blue-500
            ${error ? 'border-red-500' : 'border-gray-300'}
          `}
          {...props}
        />
        {error && (
          <p className="mt-1 text-sm text-red-500">{error.message}</p>
        )}
      </div>
    );
  }
);

FormInput.displayName = 'FormInput';
```

### Step 3: Create Form Component

```tsx
// components/LoginForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { loginSchema, LoginFormData } from '@/schemas/auth';
import { FormInput } from './FormInput';

interface LoginFormProps {
  onSubmit: (data: LoginFormData) => Promise<void>;
}

export function LoginForm({ onSubmit }: LoginFormProps) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    setError,
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  });

  const handleFormSubmit = async (data: LoginFormData) => {
    try {
      await onSubmit(data);
    } catch (error) {
      // Handle API errors
      setError('root', {
        message: 'Invalid email or password',
      });
    }
  };

  return (
    <form onSubmit={handleSubmit(handleFormSubmit)} className="space-y-4">
      <FormInput
        label="Email"
        type="email"
        {...register('email')}
        error={errors.email}
        required
      />

      <FormInput
        label="Password"
        type="password"
        {...register('password')}
        error={errors.password}
        required
      />

      {errors.root && (
        <p className="text-sm text-red-500">{errors.root.message}</p>
      )}

      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full py-2 px-4 bg-blue-600 text-white rounded-lg
          hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed"
      >
        {isSubmitting ? 'Signing in...' : 'Sign In'}
      </button>
    </form>
  );
}
```

### Step 4: Handle Form Submission

```tsx
// pages/login.tsx or app/login/page.tsx
'use client';

import { useRouter } from 'next/navigation';
import { LoginForm } from '@/components/LoginForm';
import { LoginFormData } from '@/schemas/auth';

export default function LoginPage() {
  const router = useRouter();

  const handleLogin = async (data: LoginFormData) => {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });

    if (!response.ok) {
      throw new Error('Login failed');
    }

    router.push('/dashboard');
  };

  return (
    <div className="max-w-md mx-auto mt-10 p-6">
      <h1 className="text-2xl font-bold mb-6">Sign In</h1>
      <LoginForm onSubmit={handleLogin} />
    </div>
  );
}
```

### Step 5: Advanced - Multi-step Form

```tsx
// components/MultiStepForm.tsx
import { useState } from 'react';
import { useForm, FormProvider } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

const steps = [
  { id: 'account', title: 'Account' },
  { id: 'profile', title: 'Profile' },
  { id: 'confirm', title: 'Confirm' },
];

export function MultiStepForm() {
  const [currentStep, setCurrentStep] = useState(0);

  const methods = useForm({
    resolver: zodResolver(fullSchema),
    mode: 'onChange',
  });

  const nextStep = async () => {
    // Validate current step fields
    const fieldsToValidate = getFieldsForStep(currentStep);
    const isValid = await methods.trigger(fieldsToValidate);

    if (isValid) {
      setCurrentStep((prev) => Math.min(prev + 1, steps.length - 1));
    }
  };

  const prevStep = () => {
    setCurrentStep((prev) => Math.max(prev - 1, 0));
  };

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(onSubmit)}>
        {/* Step indicators */}
        <div className="flex justify-between mb-8">
          {steps.map((step, index) => (
            <div
              key={step.id}
              className={`flex-1 text-center ${
                index <= currentStep ? 'text-blue-600' : 'text-gray-400'
              }`}
            >
              {step.title}
            </div>
          ))}
        </div>

        {/* Step content */}
        {currentStep === 0 && <AccountStep />}
        {currentStep === 1 && <ProfileStep />}
        {currentStep === 2 && <ConfirmStep />}

        {/* Navigation */}
        <div className="flex justify-between mt-6">
          <button
            type="button"
            onClick={prevStep}
            disabled={currentStep === 0}
          >
            Back
          </button>

          {currentStep < steps.length - 1 ? (
            <button type="button" onClick={nextStep}>
              Next
            </button>
          ) : (
            <button type="submit">Submit</button>
          )}
        </div>
      </form>
    </FormProvider>
  );
}
```

## Common Validation Patterns

```typescript
// Common Zod patterns
const patterns = {
  // Email
  email: z.string().email(),

  // Password (strong)
  password: z
    .string()
    .min(8)
    .regex(/[A-Z]/, 'Must contain uppercase')
    .regex(/[a-z]/, 'Must contain lowercase')
    .regex(/[0-9]/, 'Must contain number'),

  // Phone (US)
  phone: z.string().regex(/^\d{10}$/, 'Must be 10 digits'),

  // URL
  url: z.string().url(),

  // Optional with transform
  age: z.string().transform(Number).pipe(z.number().min(18).max(120)),

  // Enum
  role: z.enum(['admin', 'user', 'guest']),

  // Array with min/max
  tags: z.array(z.string()).min(1).max(5),

  // Date
  birthDate: z.coerce.date().max(new Date(), 'Must be in the past'),
};
```

## Success Criteria

- [ ] Schema validates all inputs
- [ ] Error messages are user-friendly
- [ ] Loading state during submission
- [ ] Server errors are displayed
- [ ] Form is accessible (labels, ARIA)

## Common Pitfalls

- Don't forget to handle server-side validation errors
- Don't skip loading states during submission
- Don't use onChange validation for long forms (use onBlur)
- Don't forget to reset form after successful submission
- Don't skip TypeScript types
