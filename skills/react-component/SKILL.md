# React Component

> Build reusable, type-safe React components with proper patterns.

## Trigger

- User needs to create a new React component
- User needs a reusable UI component
- User wants to refactor a component for better reusability
- User needs a component with proper TypeScript types

## Prerequisites

- React 18+
- TypeScript 5+

## Instructions

### Step 1: Component Structure

Always start with a clear file structure:

```
components/
├── Button/
│   ├── index.ts        # Re-exports
│   ├── Button.tsx      # Main component
│   ├── Button.types.ts # TypeScript types (optional for complex types)
│   └── Button.test.tsx # Tests
```

### Step 2: Define Props with TypeScript

```tsx
// Button.tsx
import { ReactNode, ButtonHTMLAttributes } from 'react';

// Extend native HTML attributes for full compatibility
interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  /** Button visual style */
  variant?: 'primary' | 'secondary' | 'ghost';
  /** Button size */
  size?: 'sm' | 'md' | 'lg';
  /** Show loading spinner */
  isLoading?: boolean;
  /** Left icon */
  leftIcon?: ReactNode;
  /** Right icon */
  rightIcon?: ReactNode;
  /** Button content */
  children: ReactNode;
}
```

### Step 3: Implement Component

```tsx
export function Button({
  variant = 'primary',
  size = 'md',
  isLoading = false,
  leftIcon,
  rightIcon,
  children,
  disabled,
  className = '',
  ...props
}: ButtonProps) {
  // Compute styles based on variant and size
  const baseStyles = 'inline-flex items-center justify-center font-medium rounded-lg transition-colors';

  const variantStyles = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 disabled:bg-blue-300',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300 disabled:bg-gray-100',
    ghost: 'bg-transparent text-gray-700 hover:bg-gray-100 disabled:text-gray-400',
  };

  const sizeStyles = {
    sm: 'px-3 py-1.5 text-sm gap-1.5',
    md: 'px-4 py-2 text-base gap-2',
    lg: 'px-6 py-3 text-lg gap-2.5',
  };

  return (
    <button
      className={`${baseStyles} ${variantStyles[variant]} ${sizeStyles[size]} ${className}`}
      disabled={disabled || isLoading}
      {...props}
    >
      {isLoading ? (
        <Spinner size={size} />
      ) : (
        <>
          {leftIcon && <span className="shrink-0">{leftIcon}</span>}
          {children}
          {rightIcon && <span className="shrink-0">{rightIcon}</span>}
        </>
      )}
    </button>
  );
}
```

### Step 4: Composition Pattern - Compound Components

For complex components, use compound pattern:

```tsx
// Card.tsx
import { createContext, useContext, ReactNode } from 'react';

interface CardContextValue {
  variant: 'elevated' | 'outlined';
}

const CardContext = createContext<CardContextValue | null>(null);

function useCardContext() {
  const context = useContext(CardContext);
  if (!context) {
    throw new Error('Card components must be used within a Card');
  }
  return context;
}

// Main Card component
interface CardProps {
  variant?: 'elevated' | 'outlined';
  children: ReactNode;
  className?: string;
}

function Card({ variant = 'elevated', children, className = '' }: CardProps) {
  const baseStyles = 'rounded-lg overflow-hidden';
  const variantStyles = {
    elevated: 'bg-white shadow-md',
    outlined: 'bg-white border border-gray-200',
  };

  return (
    <CardContext.Provider value={{ variant }}>
      <div className={`${baseStyles} ${variantStyles[variant]} ${className}`}>
        {children}
      </div>
    </CardContext.Provider>
  );
}

// Sub-components
function CardHeader({ children, className = '' }: { children: ReactNode; className?: string }) {
  return <div className={`px-6 py-4 border-b border-gray-100 ${className}`}>{children}</div>;
}

function CardBody({ children, className = '' }: { children: ReactNode; className?: string }) {
  return <div className={`px-6 py-4 ${className}`}>{children}</div>;
}

function CardFooter({ children, className = '' }: { children: ReactNode; className?: string }) {
  return <div className={`px-6 py-4 border-t border-gray-100 bg-gray-50 ${className}`}>{children}</div>;
}

// Attach sub-components
Card.Header = CardHeader;
Card.Body = CardBody;
Card.Footer = CardFooter;

export { Card };
```

**Usage:**
```tsx
<Card variant="elevated">
  <Card.Header>Title</Card.Header>
  <Card.Body>Content here</Card.Body>
  <Card.Footer>Actions</Card.Footer>
</Card>
```

### Step 5: Hooks - Custom Hook Extraction

Extract reusable logic into custom hooks:

```tsx
// hooks/useToggle.ts
import { useState, useCallback } from 'react';

export function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return { value, toggle, setTrue, setFalse };
}

// hooks/useDisclosure.ts - for modals, dropdowns
export function useDisclosure(initialOpen = false) {
  const { value: isOpen, setTrue: open, setFalse: close, toggle } = useToggle(initialOpen);
  return { isOpen, open, close, toggle };
}

// Usage in component
function Modal() {
  const { isOpen, open, close } = useDisclosure();

  return (
    <>
      <Button onClick={open}>Open Modal</Button>
      {isOpen && <ModalContent onClose={close} />}
    </>
  );
}
```

### Step 6: Controlled vs Uncontrolled Pattern

Support both controlled and uncontrolled usage:

```tsx
// Input.tsx
interface InputProps extends Omit<InputHTMLAttributes<HTMLInputElement>, 'onChange'> {
  /** Controlled value */
  value?: string;
  /** Default value for uncontrolled mode */
  defaultValue?: string;
  /** Change handler */
  onChange?: (value: string) => void;
}

export function Input({ value, defaultValue, onChange, ...props }: InputProps) {
  // Internal state for uncontrolled mode
  const [internalValue, setInternalValue] = useState(defaultValue ?? '');

  // Use controlled value if provided, otherwise internal
  const isControlled = value !== undefined;
  const inputValue = isControlled ? value : internalValue;

  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    const newValue = e.target.value;

    if (!isControlled) {
      setInternalValue(newValue);
    }

    onChange?.(newValue);
  };

  return (
    <input
      value={inputValue}
      onChange={handleChange}
      {...props}
    />
  );
}
```

### Step 7: forwardRef for DOM Access

When components need ref forwarding:

```tsx
import { forwardRef, InputHTMLAttributes } from 'react';

interface TextFieldProps extends InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
}

export const TextField = forwardRef<HTMLInputElement, TextFieldProps>(
  ({ label, error, className = '', ...props }, ref) => {
    return (
      <div className="flex flex-col gap-1">
        <label className="text-sm font-medium text-gray-700">{label}</label>
        <input
          ref={ref}
          className={`
            px-3 py-2 border rounded-lg
            focus:outline-none focus:ring-2 focus:ring-blue-500
            ${error ? 'border-red-500' : 'border-gray-300'}
            ${className}
          `}
          {...props}
        />
        {error && <span className="text-sm text-red-500">{error}</span>}
      </div>
    );
  }
);

TextField.displayName = 'TextField';
```

### Step 8: Polymorphic Components

Create components that can render as different elements:

```tsx
import { ElementType, ComponentPropsWithoutRef, ReactNode } from 'react';

type BoxProps<T extends ElementType> = {
  as?: T;
  children: ReactNode;
} & ComponentPropsWithoutRef<T>;

export function Box<T extends ElementType = 'div'>({
  as,
  children,
  ...props
}: BoxProps<T>) {
  const Component = as || 'div';
  return <Component {...props}>{children}</Component>;
}

// Usage
<Box as="section" className="p-4">Content</Box>
<Box as="article">Article content</Box>
<Box as="a" href="/link">Link content</Box>
```

## Component Patterns Summary

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Props Extension** | Wrap native elements | Button, Input |
| **Compound** | Related components | Card, Tabs, Accordion |
| **Controlled/Uncontrolled** | Form inputs | Input, Select |
| **Render Props** | Flexible rendering | List, Virtualized |
| **forwardRef** | DOM access needed | TextField, Modal |
| **Polymorphic** | Multi-element support | Box, Text, Heading |

## Success Criteria

- [ ] Component has TypeScript props interface
- [ ] Props extend relevant HTML attributes
- [ ] Default values provided for optional props
- [ ] Component handles edge cases (empty, loading, error)
- [ ] Styles are composable via className prop
- [ ] Component is accessible (keyboard, ARIA)

## Common Pitfalls

- Don't define inline objects/functions in JSX (causes re-renders)
- Don't forget displayName when using forwardRef
- Don't mix controlled and uncontrolled patterns
- Don't skip memoization for expensive renders
- Don't forget to spread remaining props (...props)
- Don't hardcode styles - allow className override
