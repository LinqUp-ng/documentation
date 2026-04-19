# Authentication Management

This section documents the primary authentication flows for LinqUp, including session management and password recovery.

---

## 🔑 Login
Users can authenticate using their registered email and password. All authentication requests are handled by the Supabase Auth service.

```typescript
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'your-secure-password',
});
```

---

## 🔄 Password Recovery

### Forget Password
When a user forgets their password, they can request a recovery link. This sends an email with a secure token to the user's registered address.

```typescript
const { data, error } = await supabase.auth.resetPasswordForEmail(email, {
  redirectTo: 'https://app.linqup.co/reset-password',
});
```

### Reset Password
Once the user clicks the recovery link, they are redirected back to the app where they can set a new password.

```typescript
const { data, error } = await supabase.auth.updateUser({ 
  password: new_password 
});
```

---

## 🛠️ Session Lifecycle
The `useAuthStore` (powered by Zustand) listens to auth state changes and updates the global application state automatically.

- **`SIGNED_IN`**: Loads the user's profile and combined metadata.
- **`SIGNED_OUT`**: Clears all local state and redirects to the login screen.
