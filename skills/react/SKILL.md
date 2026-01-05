---
name: react
description: React best practices for components, state, effects, and project structure. Use when writing React code, reviewing React components, or architecting React applications.
---

# React Best Practices

> Based on [bulletproof-react](https://github.com/alan2207/bulletproof-react) by Alan Alickovic and [react.dev](https://github.com/reactjs/react.dev)

## Project Structure

Organize by feature, not by file type:

```
src/
├── app/                  # Application layer (routes, providers)
├── components/           # Shared UI components
├── features/             # Feature-based modules
│   └── users/
│       ├── api/          # API requests and hooks
│       ├── components/   # Feature-specific components
│       ├── hooks/        # Feature-specific hooks
│       ├── stores/       # Feature state
│       └── types/        # Feature types
├── hooks/                # Shared hooks
├── lib/                  # Preconfigured libraries
└── utils/                # Shared utilities
```

**Rules:**
- Code flows: `shared → features → app` (unidirectional)
- No cross-feature imports — compose at app level
- Colocate code with where it's used
- Avoid barrel files (breaks tree-shaking)
- Use **kebab-case** for all file and folder names
- Use **absolute imports** (`@/`) instead of relative paths

```tsx
// BAD: Relative imports
import { Button } from '../../../components/ui/button';

// GOOD: Absolute imports
import { Button } from '@/components/ui/button';
```

Configure in `tsconfig.json`:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

---

## Components

### Extract, Don't Nest

```tsx
// BAD: Nested render functions
function Component() {
  function renderItems() {
    return <ul>...</ul>;
  }
  return <div>{renderItems()}</div>;
}

// GOOD: Separate components
function Items() {
  return <ul>...</ul>;
}

function Component() {
  return (
    <div>
      <Items />
    </div>
  );
}
```

### Limit Props — Use Composition

```tsx
// BAD: Too many props
<Dialog
  title="Confirm"
  body="Are you sure?"
  confirmText="Yes"
  cancelText="No"
  onConfirm={...}
  onCancel={...}
  icon={...}
/>

// GOOD: Composition pattern
<Dialog>
  <Dialog.Title>Confirm</Dialog.Title>
  <Dialog.Body>Are you sure?</Dialog.Body>
  <Dialog.Footer>
    <Button onClick={onCancel}>No</Button>
    <Button onClick={onConfirm}>Yes</Button>
  </Dialog.Footer>
</Dialog>
```

### Wrap Third-Party Components

```tsx
// GOOD: Abstraction allows future changes
import { Button as ShadcnButton } from '@/components/ui/button';

export function Button({ children, ...props }) {
  return <ShadcnButton {...props}>{children}</ShadcnButton>;
}
```

---

## State Management

### Choose the Right State Type

| Type | Use For | Tools |
|------|---------|-------|
| Component | Local UI state | `useState`, `useReducer` |
| Server Cache | API data | React Query, SWR |
| Form | Form inputs/validation | React Hook Form + Zod |
| URL | Filters, pagination | react-router params |
| Global | Themes, modals, toasts | Zustand, Jotai, Context |

### State Rules

```tsx
// BAD: Expensive initialization runs every render
const [state, setState] = useState(computeExpensiveValue());

// GOOD: Lazy initialization runs once
const [state, setState] = useState(() => computeExpensiveValue());
```

---

## Refs — Values Without Re-renders

Use `useRef` for values that shouldn't trigger re-renders:

```tsx
// GOOD: Store timeout/interval IDs
function Stopwatch() {
  const intervalRef = useRef(null);

  function handleStart() {
    intervalRef.current = setInterval(() => {
      // ...
    }, 1000);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }
}

// GOOD: Store previous values
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref.current;
}
```

### DOM Manipulation with Refs

```tsx
// GOOD: Focus, scroll, measure
function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus input</button>
    </>
  );
}

// GOOD: Scroll into view
function ItemList({ items, selectedId }) {
  const selectedRef = useRef(null);

  useEffect(() => {
    selectedRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [selectedId]);

  return (
    <ul>
      {items.map(item => (
        <li
          key={item.id}
          ref={item.id === selectedId ? selectedRef : null}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

### Ref vs State

| useRef | useState |
|--------|----------|
| `ref.current` is mutable | State is immutable |
| Changes don't trigger re-render | Changes trigger re-render |
| For values outside render flow | For values shown in UI |

---

## When to Use Effects

Use `useEffect` for:

1. **Synchronizing with external systems** (not React)
2. **Subscriptions** (WebSockets, event listeners)
3. **Analytics/logging** on mount or state change

```tsx
// GOOD: External system synchronization
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => connection.disconnect();
}, [serverUrl, roomId]);

// GOOD: Event listener cleanup
useEffect(() => {
  const handler = (e) => setPosition({ x: e.clientX, y: e.clientY });
  window.addEventListener('pointermove', handler);
  return () => window.removeEventListener('pointermove', handler);
}, []);
```

---

## When NOT to Use Effects

### Transforming Data

```tsx
// BAD: Effect for derived state
const [items, setItems] = useState([]);
const [filteredItems, setFilteredItems] = useState([]);

useEffect(() => {
  setFilteredItems(items.filter(item => item.active));
}, [items]);

// GOOD: Calculate during render
const [items, setItems] = useState([]);
const filteredItems = items.filter(item => item.active);

// GOOD: Memoize if expensive
const filteredItems = useMemo(
  () => items.filter(item => item.active),
  [items]
);
```

### Handling User Events

```tsx
// BAD: Effect to respond to state change from event
const [query, setQuery] = useState('');

useEffect(() => {
  if (query) {
    fetchResults(query);
  }
}, [query]);

function handleSubmit() {
  setQuery(inputValue);
}

// GOOD: Fetch directly in event handler
function handleSubmit() {
  setQuery(inputValue);
  fetchResults(inputValue);
}
```

### Resetting State on Prop Change

```tsx
// BAD: Effect to reset state
function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  useEffect(() => {
    setComment('');
  }, [userId]);
}

// GOOD: Key prop forces remount
function ProfilePage({ userId }) {
  return <Profile userId={userId} key={userId} />;
}

function Profile({ userId }) {
  const [comment, setComment] = useState(''); // Fresh state per userId
}
```

### Initializing from Props

```tsx
// BAD: Syncing state with props
function List({ items }) {
  const [selection, setSelection] = useState(null);

  useEffect(() => {
    setSelection(null);
  }, [items]);
}

// GOOD: Derive or compute during render
function List({ items }) {
  const [selectedId, setSelectedId] = useState(null);
  const selection = items.find(item => item.id === selectedId) ?? null;
}
```

### Fetching Data

```tsx
// BAD: Manual fetch in effect
useEffect(() => {
  let ignore = false;
  fetchData().then(data => {
    if (!ignore) setData(data);
  });
  return () => { ignore = true; };
}, []);

// GOOD: Use React Query
const { data, isLoading, error } = useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
});
```

### Chained Effects

```tsx
// BAD: Effect chains
useEffect(() => { setA(computeA()); }, []);
useEffect(() => { setB(computeB(a)); }, [a]);
useEffect(() => { setC(computeC(b)); }, [b]);

// GOOD: Compute together or use reducer
const [state, dispatch] = useReducer(reducer, initialState);
// Or calculate derived values during render
const a = computeA();
const b = computeB(a);
const c = computeC(b);
```

---

## Effect Dependencies

### Move Objects Inside Effects

```tsx
// BAD: Object recreated every render, Effect runs every render
function ChatRoom({ roomId }) {
  const options = { serverUrl: 'https://localhost', roomId };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // 🔴 Object is new every render!
}

// GOOD: Move object inside Effect
function ChatRoom({ roomId }) {
  useEffect(() => {
    const options = { serverUrl: 'https://localhost', roomId };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Only re-runs when roomId changes
}
```

### Move Functions Inside Effects

```tsx
// BAD: Function recreated every render
function ChatRoom({ roomId }) {
  function createOptions() {
    return { serverUrl: 'https://localhost', roomId };
  }

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // 🔴 Function is new every render!
}

// GOOD: Move function inside Effect
function ChatRoom({ roomId }) {
  useEffect(() => {
    function createOptions() {
      return { serverUrl: 'https://localhost', roomId };
    }
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Only re-runs when roomId changes
}
```

### Separate Reactive from Non-Reactive Logic

```tsx
// BAD: Reconnects when theme changes (unnecessary)
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, theme]); // 🔴 Reconnects on theme change!
}

// GOOD: Use a ref for non-reactive values
function ChatRoom({ roomId, theme }) {
  const themeRef = useRef(theme);
  useEffect(() => {
    themeRef.current = theme;
  }, [theme]);

  useEffect(() => {
    const connection = createConnection(roomId);
    connection.on('connected', () => {
      showNotification('Connected!', themeRef.current);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Only reconnects when roomId changes
}
```

---

## Custom Hooks

Extract reusable logic into custom hooks:

```tsx
// GOOD: Extract mouse position tracking
function useMousePosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, []);

  return position;
}

// Usage
function Tooltip() {
  const { x, y } = useMousePosition();
  return <div style={{ left: x, top: y }}>...</div>;
}
```

```tsx
// GOOD: Extract local storage sync
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}

// Usage
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
}
```

### Custom Hook Rules

- Name must start with `use`
- Can call other hooks
- Share logic, not state (each call gets its own state)
- Keep hooks focused on one thing

---

## Performance

### Children as Optimization

```tsx
// BAD: PureComponent re-renders on count change
const Counter = () => {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      <PureComponent /> {/* Re-renders! */}
    </div>
  );
};

// GOOD: Children are isolated VDOM
const Counter = ({ children }) => {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      {children} {/* Won't re-render */}
    </div>
  );
};

// Usage
<Counter>
  <PureComponent />
</Counter>
```

### Code Splitting

```tsx
// GOOD: Lazy load routes
const Dashboard = lazy(() => import('./features/dashboard'));
const Settings = lazy(() => import('./features/settings'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

### Context Performance

- Use Context for low-velocity data (themes, user, locale)
- Split contexts by update frequency
- Consider Zustand/Jotai for high-velocity state
- Try lifting state or composition before reaching for Context

---

## API Layer Pattern

```tsx
// lib/api-client.ts — Single configured instance
export const api = axios.create({ baseURL: env.API_URL });

// features/users/api/get-users.ts — Colocated with feature
export const getUsers = () => api.get('/users');

export const getUsersQueryOptions = () => ({
  queryKey: ['users'],
  queryFn: getUsers,
});

export const useUsers = () => useQuery(getUsersQueryOptions());
```

---

## Error Handling

### API Errors — Use Interceptors

```tsx
// lib/api-client.ts
api.interceptors.response.use(
  (response) => response,
  (error) => {
    const message = error.response?.data?.message || 'An error occurred';

    // Show toast notification
    toast.error(message);

    // Handle 401 — logout user
    if (error.response?.status === 401) {
      logout();
    }

    return Promise.reject(error);
  }
);
```

### Error Boundaries — Use Multiple, Not One

```tsx
// BAD: Single app-wide error boundary
<ErrorBoundary>
  <App /> {/* One error crashes everything */}
</ErrorBoundary>

// GOOD: Granular error boundaries
<Layout>
  <ErrorBoundary fallback={<SidebarError />}>
    <Sidebar />
  </ErrorBoundary>
  <ErrorBoundary fallback={<ContentError />}>
    <MainContent />
  </ErrorBoundary>
</Layout>
```

Place error boundaries around:
- Route components
- Independent features/widgets
- Third-party components
- Anything that can fail independently

---

## Security

### XSS Prevention — Sanitize User Content

```tsx
// BAD: Dangerous innerHTML
function Comment({ content }) {
  return <div dangerouslySetInnerHTML={{ __html: content }} />;
}

// GOOD: Sanitize with DOMPurify
import DOMPurify from 'dompurify';

function Comment({ content }) {
  const sanitized = DOMPurify.sanitize(content);
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}

// BEST: Use a markdown renderer with built-in sanitization
import ReactMarkdown from 'react-markdown';

function Comment({ content }) {
  return <ReactMarkdown>{content}</ReactMarkdown>;
}
```

### Auth Token Storage

```tsx
// BAD: localStorage (vulnerable to XSS)
localStorage.setItem('token', token);

// GOOD: HttpOnly cookies (set by server, inaccessible to JS)
// Token is automatically sent with requests via credentials: 'include'
```

### Authorization — RBAC Pattern

```tsx
// Authorization wrapper component
function RBAC({ allowedRoles, children }) {
  const { user } = useAuth();

  if (!allowedRoles.includes(user.role)) {
    return null;
  }

  return children;
}

// Usage
<RBAC allowedRoles={['ADMIN']}>
  <DeleteUserButton />
</RBAC>
```

### Authorization — PBAC (Permission-Based)

```tsx
// For granular permissions (e.g., only owner can delete)
function CanDelete({ resource, children }) {
  const { user } = useAuth();

  const canDelete =
    user.role === 'ADMIN' ||
    resource.authorId === user.id;

  if (!canDelete) return null;

  return children;
}

// Usage
<CanDelete resource={comment}>
  <DeleteButton onClick={() => deleteComment(comment.id)} />
</CanDelete>
```
