## 1. keep components smaller and focussed follow always single responsibility principle, so that they can be easily test, debug and maintain.
```jsx
const UserAvatar = ({ user }) => (
  <img src={user.avatarUrl} alt={user.name} className="avatar" />
);

const UserInfo = ({ user }) => (
  <div className="user-info">
    <UserAvatar user={user} />
    <UserName name={user.name} />
    <UserEmail email={user.email} />
  </div>
);
```

## 2. use functional components with hooks, writing in as class components is old style.
### functional components with hooks are the react standard since 16.8. They reduce the bilerplate code, improve code reusability through custom hooks and are easier to optimize. 
```jsx
const DataList = ({ apiEndpoint }) => {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchData(apiEndpoint).then(setData).finally(() => setLoading(false));
  }, [apiEndpoint]);
  
  return loading ? <Spinner /> : <List items={data} />;
};
```

## 3. Clear naming conventions improves code readability and maintainability. React follows PascalCase for maming conventions. The name sshoiuld reveal intent-a developer should understand what the component does without reading its implementation.
```jsx
const ProductCard = ({ product }) => { /* ... */ };
const UserDashboardHeader = () => { /* ... */ };
const InvoiceDownloadButton = ({ invoiceId }) => { /* ... */ };
```

## 4.  lift the state when necessary 

```jsx
const SearchInput = () => {
  const [query, setQuery] = useState('');  // Local state
  
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
};

const ParentComponent = () => {
  const [sharedData, setSharedData] = useState(null);
  
  return (
    <>
      <ComponentA data={sharedData} />
      <ComponentB data={sharedData} onChange={setSharedData} />
    </>
  );
};
```

## 5. use states accordingly 
### useState for simple local state
```jsx
const [count, setCount] = useState(0);
```
###  useReducer for complex state logic 
```jsx
const [state, dispatch] = useReducer(formReducer, initialFormState);
```
### context for theme, auth, locale, (stable, infrequent updates)
```jsx
const { theme, toggleTheme } = useTheme();
```
### redux/zustand/jotai for global app state (use data and cache)
```jsx
const user = useSelector(state => state.user);
```
### react query/SWR for server state
```jsx
const { data, isLoading } = useQuery('products', fetchProducts);
```
## 6. Immutable state updates
```jsx
//Immutable state updates
const [user, setUser] = useState({ name: 'John', age: 30 });

// Update using spread operator
setUser(prev => ({ ...prev, age: 31 }));

// For arrays
const [items, setItems] = useState([1, 2, 3]);
setItems(prev => [...prev, 4]);  // Add
setItems(prev => prev.filter(item => item !== 2));  // Remove
setItems(prev => prev.map(item => item === 2 ? 20 : item));  // Update
```

## 7. for nested objects state update use the immer
```jsx
import { produce } from 'immer';
setComplexState(produce(draft => {
  draft.user.profile.settings.theme = 'dark';
}));
```

### 8. use functional updates for state based on previous state.
```jsx
const [count, setCount] = useState(0);

const increment = () => {
  setCount(prev => prev + 1);  // Always uses latest state
};

// Multiple updates work correctly
const incrementThrice = () => {
  setCount(prev => prev + 1);
  setCount(prev => prev + 1);
  setCount(prev => prev + 1);
  // Result: count + 3
};
```

### 9. Hooks best practices 
### don't keep states inside hooks at all 
### extract complex logic into custom hooks 
```jsx
const useUserData = (userId) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetchUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [userId]);
  
  return { user, loading, error };
};

// Usage
const UserProfile = ({ userId }) => {
  const { user, loading, error } = useUserData(userId);
  if (loading) return <Spinner />;
  if (error) return <Error error={error} />;
  return <Profile user={user} />;
};
```
## 10. use useMemo and useCallback Appropriately, for expensive calculations use useMemo

```jsx
// Memoize expensive computations
const ExpensiveComponent = ({ items }) => {
  const processedData = useMemo(() => {
    return items.map(item => expensiveTransform(item));
  }, [items]);  // Only recompute when items change
  
  return <List data={processedData} />;
};

// Memoize callbacks passed to memoized children
const Parent = () => {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);  // Stable reference
  
  return <MemoizedChild onClick={handleClick} />;
};
```
## 11. use Props destructing with default values 
```jsx
// Clear, with defaults
const Button = ({ 
  variant = 'primary',
  size = 'medium',
  disabled = false,
  onClick,
  children 
}) => (
  <button 
    className={`btn btn-${variant} btn-${size}`}
    disabled={disabled}
    onClick={onClick}
  >
    {children}
  </button>
);
```

## 12. use PropTypes or TypeScript (preferred for enterprise) for type safety
```tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  onClick: () => void;
  children: React.ReactNode;
}

const Button: React.FC<ButtonProps> = ({ variant = 'primary', ...props }) => {
  // Type-safe props
};
```

## 13. always pass known props explicity
```jsx
const CustomInput = ({ label, error, ...inputProps }) => (
  <div>
    <label>{label}</label>
    <input {...inputProps} />  {/* Only HTML input props */}
    {error && <span className="error">{error}</span>}
  </div>
);
```

## 14. use children properly 

```jsx
//Flexible composition
const Card = ({ children }) => (
  <div className="card">{children}</div>
);

// Usage
<Card>
  <CardHeader title="My Card" />
  <CardBody>Content here</CardBody>
  <CardFooter />
</Card>

//Render props for advanced cases
const DataFetcher = ({ url, children }) => {
  const { data, loading } = useFetch(url);
  return children({ data, loading });
};

// Usage
<DataFetcher url="/api/users">
  {({ data, loading }) => loading ? <Spinner /> : <List items={data} />}
</DataFetcher>
```

## 15. use React.memo for pure components 

```jsx
// Memoize component that re-renders unnecessarily
const ExpensiveListItem = React.memo(({ item, onDelete }) => {
  console.log('Rendering item:', item.id);
  return (
    <div>
      <span>{item.name}</span>
      <button onClick={() => onDelete(item.id)}>Delete</button>
    </div>
  );
});

// Custom comparison function for complex props
const UserCard = React.memo(
  ({ user }) => <div>{user.name}</div>,
  (prevProps, nextProps) => prevProps.user.id === nextProps.user.id
);
```

## 16. use proper keys in the lists
```jsx
// Stable, unique keys
const UserList = ({ users }) => (
  <ul>
    {users.map(user => (
      <li key={user.id}>{user.name}</li>  // Use unique ID
    ))}
  </ul>
);
```

## 17. For dynamic lists without IDs, generate stable keys
```jsx
const generateStableId = (item) => `${item.name}-${item.email}`;

const ItemList = ({ items }) => (
  <ul>
    {items.map(item => (
      <li key={generateStableId(item)}>{item.name}</li>
    ))}
  </ul>
);
```

## 18. Lazy load components and routes

```jsx
// Code splitting with React.lazy
const Dashboard = React.lazy(() => import('./pages/Dashboard'));
const Reports = React.lazy(() => import('./pages/Reports'));
const Settings = React.lazy(() => import('./pages/Settings'));

const App = () => (
  <Suspense fallback={<LoadingSpinner />}>
    <Routes>
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/reports" element={<Reports />} />
      <Route path="/settings" element={<Settings />} />
    </Routes>
  </Suspense>
);

// Lazy load heavy components
const HeavyChart = React.lazy(() => import('./components/HeavyChart'));

const Analytics = () => {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && (
        <Suspense fallback={<Spinner />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
};
```

## 19. use debounce for expensive operations
```jsx
import { useDebouncedCallback } from 'use-debounce';

const SearchInput = () => {
  const [query, setQuery] = useState('');
  
  const debouncedSearch = useDebouncedCallback(
    (value) => {
      apiClient.search(value);  // API call only after user stops typing
    },
    500  // 500ms delay
  );
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    debouncedSearch(value);
  };
  
  return <input value={query} onChange={handleChange} />;
};
```

## 20. use Error boundaries 

```jsx
// Error boundary component
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    // Log to error tracking service
    logErrorToService(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback || <ErrorFallback error={this.state.error} />;
    }
    
    return this.props.children;
  }
}

// Usage
const App = () => (
  <ErrorBoundary fallback={<ErrorPage />}>
    <Dashboard />
  </ErrorBoundary>
);
```

## 21. use proper event handler naming 
```jsx
// Clear, consistent naming (handle* for component functions)
const Form = () => {
  const handleSubmit = (e) => {
    e.preventDefault();
    // ...
  };
  
  const handleInputChange = (e) => {
    // ...
  };
  
  const handleCancel = () => {
    // ...
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleInputChange} />
      <button type="submit">Submit</button>
      <button type="button" onClick={handleCancel}>Cancel</button>
    </form>
  );
};

// use on* for props passed to children
<CustomButton onClick={handleClick} onHover={handleHover} />
```
## 22. use appropriate conditinal rendering patterns
```jsx
// Ternary for if-else
{isLoading ? <Spinner /> : <Data />}

// && for simple conditionals
{error && <ErrorMessage error={error} />}
{user.isAdmin && <AdminPanel />}

// Early returns for complex conditions
const UserProfile = ({ user }) => {
  if (!user) return <NotFound />;
  if (!user.isActive) return <InactiveAccount />;
  if (user.isPending) return <PendingApproval />;
  
  return <ProfileContent user={user} />;
};

```
## 23. use controlled components 

```jsx
// Controlled input
const Form = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    apiClient.login({ email, password });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button type="submit">Login</button>
    </form>
  );
};
```
## 24. use react-hook-form library for complex forms

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be 8+ characters'),
  age: z.number().min(18, 'Must be 18+')
});

const RegistrationForm = () => {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema)
  });
  
  const onSubmit = (data) => {
    apiClient.register(data);
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input type="password" {...register('password')} />
      {errors.password && <span>{errors.password.message}</span>}
      
      <button type="submit">Register</button>
    </form>
  );
};
```

## 25. use refs sparingly for focus management
```jsx
const SearchInput = () => {
  const inputRef = useRef(null);
  
  useEffect(() => {
    inputRef.current?.focus();  // Auto-focus on mount
  }, []);
  
  return <input ref={inputRef} type="search" />;
};
```











