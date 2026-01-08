## 1. define explicit types for props and state

```tsx
// Explicit prop types
interface UserCardProps {
  user: {
    id: string;
    name: string;
    email: string;
    avatar?: string;
  };
  onDelete: (userId: string) => void;
  className?: string;
}

const UserCard: React.FC<UserCardProps> = ({ user, onDelete, className }) => {
  return (
    <div className={className}>
      {user.avatar && <img src={user.avatar} alt={user.name} />}
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onDelete(user.id)}>Delete</button>
    </div>
  );
};

// Type for complex state
interface FormState {
  values: Record<string, string>;
  errors: Record<string, string>;
  touched: Record<string, boolean>;
  isSubmitting: boolean;
}

const [formState, setFormState] = useState<FormState>({
  values: {},
  errors: {},
  touched: {},
  isSubmitting: false
});
```

