# React Testing Library

> Test React components the way users interact with them.

## Trigger

- User needs to test React components
- User needs to test user interactions
- User wants accessible component tests
- User needs to test forms and inputs

## Prerequisites

```bash
npm install -D @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

```typescript
// jest.setup.ts
import '@testing-library/jest-dom';
```

## Instructions

### Step 1: Basic Component Test

```tsx
// components/Button.tsx
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
}

export function Button({ children, onClick, disabled }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}
```

```tsx
// components/Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  it('renders children', () => {
    render(<Button>Click me</Button>);

    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  it('calls onClick when clicked', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();

    render(<Button onClick={handleClick}>Click me</Button>);

    await user.click(screen.getByRole('button'));

    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);

    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

### Step 2: Query Methods (Priority Order)

```tsx
describe('Query Methods', () => {
  // 1. getByRole - BEST for accessibility
  it('queries by role', () => {
    render(<button>Submit</button>);
    expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
  });

  // 2. getByLabelText - for form fields
  it('queries by label', () => {
    render(
      <label>
        Email
        <input type="email" />
      </label>
    );
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
  });

  // 3. getByPlaceholderText - if no label available
  it('queries by placeholder', () => {
    render(<input placeholder="Enter email" />);
    expect(screen.getByPlaceholderText(/enter email/i)).toBeInTheDocument();
  });

  // 4. getByText - for non-interactive elements
  it('queries by text', () => {
    render(<p>Hello World</p>);
    expect(screen.getByText(/hello world/i)).toBeInTheDocument();
  });

  // 5. getByDisplayValue - for current input value
  it('queries by display value', () => {
    render(<input value="test@example.com" readOnly />);
    expect(screen.getByDisplayValue(/test@example.com/i)).toBeInTheDocument();
  });

  // 6. getByAltText - for images
  it('queries by alt text', () => {
    render(<img alt="User avatar" src="/avatar.png" />);
    expect(screen.getByAltText(/user avatar/i)).toBeInTheDocument();
  });

  // 7. getByTitle - for title attribute
  it('queries by title', () => {
    render(<span title="Close">X</span>);
    expect(screen.getByTitle(/close/i)).toBeInTheDocument();
  });

  // 8. getByTestId - LAST RESORT
  it('queries by test id', () => {
    render(<div data-testid="custom-element">Content</div>);
    expect(screen.getByTestId('custom-element')).toBeInTheDocument();
  });
});
```

### Step 3: Query Variants

```tsx
describe('Query Variants', () => {
  // getBy - throws if not found (use for elements that should exist)
  it('getBy throws if not found', () => {
    render(<div>Hello</div>);
    expect(() => screen.getByText(/goodbye/i)).toThrow();
  });

  // queryBy - returns null if not found (use to assert absence)
  it('queryBy returns null if not found', () => {
    render(<div>Hello</div>);
    expect(screen.queryByText(/goodbye/i)).not.toBeInTheDocument();
  });

  // findBy - async, waits for element (use for async rendering)
  it('findBy waits for element', async () => {
    render(<AsyncComponent />);
    expect(await screen.findByText(/loaded/i)).toBeInTheDocument();
  });

  // getAllBy - returns array, throws if empty
  it('getAllBy returns multiple elements', () => {
    render(
      <ul>
        <li>Item 1</li>
        <li>Item 2</li>
      </ul>
    );
    expect(screen.getAllByRole('listitem')).toHaveLength(2);
  });

  // queryAllBy - returns empty array if not found
  it('queryAllBy returns empty array', () => {
    render(<ul></ul>);
    expect(screen.queryAllByRole('listitem')).toHaveLength(0);
  });

  // findAllBy - async version
  it('findAllBy waits for elements', async () => {
    render(<AsyncList />);
    const items = await screen.findAllByRole('listitem');
    expect(items).toHaveLength(3);
  });
});
```

### Step 4: User Events

```tsx
import userEvent from '@testing-library/user-event';

describe('User Events', () => {
  it('types in input', async () => {
    const user = userEvent.setup();

    render(<input aria-label="Email" />);
    const input = screen.getByLabelText(/email/i);

    await user.type(input, 'test@example.com');

    expect(input).toHaveValue('test@example.com');
  });

  it('clears and types', async () => {
    const user = userEvent.setup();

    render(<input aria-label="Email" defaultValue="old@example.com" />);
    const input = screen.getByLabelText(/email/i);

    await user.clear(input);
    await user.type(input, 'new@example.com');

    expect(input).toHaveValue('new@example.com');
  });

  it('clicks button', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();

    render(<button onClick={handleClick}>Click</button>);

    await user.click(screen.getByRole('button'));

    expect(handleClick).toHaveBeenCalled();
  });

  it('double clicks', async () => {
    const user = userEvent.setup();
    const handleDblClick = jest.fn();

    render(<div onDoubleClick={handleDblClick}>Double click me</div>);

    await user.dblClick(screen.getByText(/double click/i));

    expect(handleDblClick).toHaveBeenCalled();
  });

  it('selects option', async () => {
    const user = userEvent.setup();

    render(
      <select aria-label="Color">
        <option value="red">Red</option>
        <option value="blue">Blue</option>
      </select>
    );

    await user.selectOptions(screen.getByLabelText(/color/i), 'blue');

    expect(screen.getByRole('option', { name: 'Blue' })).toBeSelected();
  });

  it('checks checkbox', async () => {
    const user = userEvent.setup();

    render(<input type="checkbox" aria-label="Accept terms" />);
    const checkbox = screen.getByLabelText(/accept terms/i);

    await user.click(checkbox);

    expect(checkbox).toBeChecked();
  });

  it('hovers element', async () => {
    const user = userEvent.setup();

    render(<Tooltip text="Hello">Hover me</Tooltip>);

    await user.hover(screen.getByText(/hover me/i));

    expect(screen.getByText('Hello')).toBeVisible();
  });

  it('tabs through elements', async () => {
    const user = userEvent.setup();

    render(
      <>
        <input aria-label="First" />
        <input aria-label="Second" />
      </>
    );

    await user.tab();
    expect(screen.getByLabelText(/first/i)).toHaveFocus();

    await user.tab();
    expect(screen.getByLabelText(/second/i)).toHaveFocus();
  });

  it('uses keyboard shortcuts', async () => {
    const user = userEvent.setup();

    render(<Editor />);

    await user.keyboard('{Control>}s{/Control}');

    expect(screen.getByText(/saved/i)).toBeInTheDocument();
  });
});
```

### Step 5: Testing Forms

```tsx
// components/LoginForm.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('submits form with valid data', async () => {
    const user = userEvent.setup();
    const handleSubmit = jest.fn();

    render(<LoginForm onSubmit={handleSubmit} />);

    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /sign in/i }));

    await waitFor(() => {
      expect(handleSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123',
      });
    });
  });

  it('shows validation errors', async () => {
    const user = userEvent.setup();

    render(<LoginForm onSubmit={jest.fn()} />);

    await user.click(screen.getByRole('button', { name: /sign in/i }));

    expect(await screen.findByText(/email is required/i)).toBeInTheDocument();
    expect(await screen.findByText(/password is required/i)).toBeInTheDocument();
  });

  it('disables submit button while loading', async () => {
    const user = userEvent.setup();
    const handleSubmit = jest.fn(() => new Promise(() => {})); // Never resolves

    render(<LoginForm onSubmit={handleSubmit} />);

    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /sign in/i }));

    expect(screen.getByRole('button', { name: /signing in/i })).toBeDisabled();
  });
});
```

### Step 6: Testing Async Components

```tsx
// components/UserProfile.test.tsx
import { render, screen, waitFor, waitForElementToBeRemoved } from '@testing-library/react';
import { UserProfile } from './UserProfile';

// Mock the fetch
global.fetch = jest.fn();

describe('UserProfile', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('shows loading state', () => {
    (fetch as jest.Mock).mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ name: 'John' }),
    });

    render(<UserProfile userId="123" />);

    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });

  it('shows user data after loading', async () => {
    (fetch as jest.Mock).mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ name: 'John Doe', email: 'john@example.com' }),
    });

    render(<UserProfile userId="123" />);

    expect(await screen.findByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  it('shows error state', async () => {
    (fetch as jest.Mock).mockRejectedValue(new Error('Failed to fetch'));

    render(<UserProfile userId="123" />);

    expect(await screen.findByText(/error/i)).toBeInTheDocument();
  });

  it('removes loading indicator after data loads', async () => {
    (fetch as jest.Mock).mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ name: 'John' }),
    });

    render(<UserProfile userId="123" />);

    await waitForElementToBeRemoved(() => screen.queryByText(/loading/i));

    expect(screen.getByText('John')).toBeInTheDocument();
  });
});
```

### Step 7: Testing with Context/Providers

```tsx
// test-utils.tsx
import { render, RenderOptions } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ThemeProvider } from './ThemeContext';
import { AuthProvider } from './AuthContext';

const createTestQueryClient = () =>
  new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });

interface WrapperProps {
  children: React.ReactNode;
}

function AllProviders({ children }: WrapperProps) {
  const queryClient = createTestQueryClient();

  return (
    <QueryClientProvider client={queryClient}>
      <AuthProvider>
        <ThemeProvider>{children}</ThemeProvider>
      </AuthProvider>
    </QueryClientProvider>
  );
}

const customRender = (
  ui: React.ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>
) => render(ui, { wrapper: AllProviders, ...options });

export * from '@testing-library/react';
export { customRender as render };
```

```tsx
// components/Dashboard.test.tsx
import { render, screen } from '../test-utils';
import { Dashboard } from './Dashboard';

describe('Dashboard', () => {
  it('renders with providers', () => {
    render(<Dashboard />);

    expect(screen.getByRole('heading')).toBeInTheDocument();
  });
});
```

### Step 8: Common Matchers (jest-dom)

```tsx
describe('jest-dom matchers', () => {
  it('checks visibility', () => {
    render(<div style={{ display: 'none' }}>Hidden</div>);
    expect(screen.getByText('Hidden')).not.toBeVisible();
  });

  it('checks disabled state', () => {
    render(<button disabled>Submit</button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });

  it('checks enabled state', () => {
    render(<button>Submit</button>);
    expect(screen.getByRole('button')).toBeEnabled();
  });

  it('checks focus', () => {
    render(<input autoFocus />);
    expect(screen.getByRole('textbox')).toHaveFocus();
  });

  it('checks class', () => {
    render(<div className="active highlighted">Content</div>);
    expect(screen.getByText('Content')).toHaveClass('active', 'highlighted');
  });

  it('checks style', () => {
    render(<div style={{ color: 'red' }}>Red text</div>);
    expect(screen.getByText('Red text')).toHaveStyle({ color: 'red' });
  });

  it('checks attribute', () => {
    render(<a href="/home">Home</a>);
    expect(screen.getByRole('link')).toHaveAttribute('href', '/home');
  });

  it('checks text content', () => {
    render(<div>Hello World</div>);
    expect(screen.getByText(/hello/i)).toHaveTextContent('Hello World');
  });

  it('checks form validity', () => {
    render(<input required value="" />);
    expect(screen.getByRole('textbox')).toBeInvalid();
  });
});
```

## Query Priority

| Priority | Query | Use Case |
|----------|-------|----------|
| 1 | getByRole | Accessible elements |
| 2 | getByLabelText | Form fields |
| 3 | getByPlaceholderText | When no label |
| 4 | getByText | Non-interactive text |
| 5 | getByDisplayValue | Current input value |
| 6 | getByAltText | Images |
| 7 | getByTitle | Title attribute |
| 8 | getByTestId | Last resort |

## Success Criteria

- [ ] Tests use accessible queries (getByRole first)
- [ ] User events are awaited
- [ ] Async rendering is handled with findBy/waitFor
- [ ] Forms test validation and submission
- [ ] Loading and error states are tested

## Common Pitfalls

- Don't use getByTestId as first choice
- Don't forget to await userEvent actions
- Don't use waitFor for things that appear immediately
- Don't query by class names or implementation details
- Don't forget to wrap with necessary providers
- Don't use fireEvent when userEvent is available
