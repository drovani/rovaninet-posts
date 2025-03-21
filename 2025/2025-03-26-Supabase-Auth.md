---
title: "Implementing Supabase Auth in React Router: A Modular Approach"
category: Technical Guide
excerpt: "Learn how to implement a clean, modular authentication system using Supabase Auth with React Router v7 and TypeScript. This guide covers building a swappable auth service, creating an authentication context, and implementing sign-up/login components with shadcn UI. Perfect for projects that need robust authentication without tight coupling to a specific provider."
tags:
  - react
  - typescript
  - supabase
  - authentication
  - react-router
date: 2025-03-26
---

Authentication is a critical component for any application that manages user data. For my [Hero Wars Helper](https://herowars.rovani.net/) project, I needed to implement a login system that would secure user-specific content while maintaining a clean separation of concerns. In this post, I'll walk through implementing Supabase Auth in a React Router v7 application with TypeScript, using a modular approach that allows for easy replacement of the auth provider if needed in the future.

## Our Authentication Requirements

Before diving into implementation, let's outline what we need from our authentication system:

- Email/password authentication with optional social logins (Google, Apple, Twitch)
- Persistent sessions that survive page refreshes and browser restarts
- Protected routes that redirect unauthenticated users
- A clean separation between auth provider (Supabase) and application code
- TypeScript support throughout
- Integration with our existing UI component system (shadcn)

## Project Setup

First, let's install the necessary dependencies:

```bash
npm install @supabase/supabase-js loglevel
```

I'm assuming you already have React Router v7 and shadcn UI components set up in your project.

## Creating a Supabase Service Layer

The first step in our modular approach is creating a service layer that encapsulates all Supabase-specific code. This separation will make it easier to switch to a different auth provider in the future.

Let's create a new file called `supabaseService.ts`:

```typescript
import { createClient, SupabaseClient, User, AuthError } from '@supabase/supabase-js';
import log from 'loglevel';

// Define the authentication provider interface
export interface AuthProvider {
  login: (email: string, password: string) => Promise<{ user: User | null; error: AuthError | null }>;
  loginWithGoogle: () => Promise<void>;
  loginWithApple: () => Promise<void>;
  loginWithTwitch: () => Promise<void>;
  signUp: (email: string, password: string) => Promise<{ user: User | null; error: AuthError | null }>;
  logout: () => Promise<void>;
  getCurrentUser: () => Promise<User | null>;
  onAuthStateChange: (callback: (user: User | null) => void) => { unsubscribe: () => void };
}

class SupabaseService implements AuthProvider {
  private supabase: SupabaseClient;
  
  constructor() {
    const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
    const supabaseKey = import.meta.env.VITE_SUPABASE_ANON_KEY;
    
    if (!supabaseUrl || !supabaseKey) {
      throw new Error('Missing Supabase environment variables');
    }
    
    this.supabase = createClient(supabaseUrl, supabaseKey);
    log.debug('Supabase client initialized');
  }
  
  async login(email: string, password: string) {
    try {
      const { data, error } = await this.supabase.auth.signInWithPassword({
        email,
        password,
      });
      
      if (error) {
        log.error('Login error:', error);
        return { user: null, error };
      }
      
      log.info('User logged in successfully');
      return { user: data.user, error: null };
    } catch (error) {
      log.error('Unexpected login error:', error);
      return { user: null, error: error as AuthError };
    }
  }
  
  async loginWithGoogle() {
    try {
      await this.supabase.auth.signInWithOAuth({
        provider: 'google',
        options: {
          redirectTo: `${window.location.origin}/auth/callback`
        }
      });
    } catch (error) {
      log.error('Google login error:', error);
    }
  }
  
  async loginWithApple() {
    try {
      await this.supabase.auth.signInWithOAuth({
        provider: 'apple',
        options: {
          redirectTo: `${window.location.origin}/auth/callback`
        }
      });
    } catch (error) {
      log.error('Apple login error:', error);
    }
  }
  
  async loginWithTwitch() {
    try {
      await this.supabase.auth.signInWithOAuth({
        provider: 'twitch',
        options: {
          redirectTo: `${window.location.origin}/auth/callback`
        }
      });
    } catch (error) {
      log.error('Twitch login error:', error);
    }
  }
  
  async signUp(email: string, password: string) {
    try {
      const { data, error } = await this.supabase.auth.signUp({
        email,
        password,
      });
      
      if (error) {
        log.error('Sign up error:', error);
        return { user: null, error };
      }
      
      log.info('User signed up successfully');
      return { user: data.user, error: null };
    } catch (error) {
      log.error('Unexpected sign up error:', error);
      return { user: null, error: error as AuthError };
    }
  }
  
  async logout() {
    try {
      await this.supabase.auth.signOut();
      log.info('User logged out successfully');
    } catch (error) {
      log.error('Logout error:', error);
    }
  }
  
  async getCurrentUser() {
    try {
      const { data } = await this.supabase.auth.getUser();
      return data.user;
    } catch (error) {
      log.error('Get current user error:', error);
      return null;
    }
  }
  
  onAuthStateChange(callback: (user: User | null) => void) {
    const { data } = this.supabase.auth.onAuthStateChange((_, session) => {
      callback(session?.user || null);
    });
    
    return data;
  }
}

// Export a singleton instance
export const authService = new SupabaseService();
```

By defining an `AuthProvider` interface and implementing it with our `SupabaseService` class, we've created a clean abstraction layer. If we decide to switch to Auth0, Firebase, or any other auth provider in the future, we can create a new implementation of the `AuthProvider` interface without changing the rest of our application code.

## Setting Up the Auth Context

Now that we have our service layer, let's create an authentication context to make auth state available throughout our application:

```typescript
import { createContext, useContext, useState, useEffect, ReactNode } from 'react';
import { User } from '@supabase/supabase-js';
import { authService } from './supabaseService';
import log from 'loglevel';

interface AuthContextType {
  user: User | null;
  isLoading: boolean;
  login: (email: string, password: string) => Promise<{ success: boolean; error?: string }>;
  loginWithGoogle: () => Promise<void>;
  loginWithApple: () => Promise<void>;
  loginWithTwitch: () => Promise<void>;
  signUp: (email: string, password: string) => Promise<{ success: boolean; error?: string }>;
  logout: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    // Check for existing session on mount
    const checkSession = async () => {
      try {
        const currentUser = await authService.getCurrentUser();
        setUser(currentUser);
      } catch (error) {
        log.error('Session check error:', error);
      } finally {
        setIsLoading(false);
      }
    };

    checkSession();

    // Subscribe to auth state changes
    const { unsubscribe } = authService.onAuthStateChange((updatedUser) => {
      setUser(updatedUser);
      setIsLoading(false);
    });

    // Clean up subscription
    return unsubscribe;
  }, []);

  const login = async (email: string, password: string) => {
    try {
      const { user, error } = await authService.login(email, password);
      
      if (error) {
        return { success: false, error: error.message };
      }
      
      return { success: true };
    } catch (error) {
      log.error('Login error:', error);
      return { 
        success: false, 
        error: error instanceof Error ? error.message : 'An unknown error occurred' 
      };
    }
  };

  const signUp = async (email: string, password: string) => {
    try {
      const { user, error } = await authService.signUp(email, password);
      
      if (error) {
        return { success: false, error: error.message };
      }
      
      return { success: true };
    } catch (error) {
      log.error('Sign up error:', error);
      return { 
        success: false, 
        error: error instanceof Error ? error.message : 'An unknown error occurred' 
      };
    }
  };

  const value = {
    user,
    isLoading,
    login,
    loginWithGoogle: authService.loginWithGoogle,
    loginWithApple: authService.loginWithApple,
    loginWithTwitch: authService.loginWithTwitch,
    signUp,
    logout: authService.logout,
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  
  return context;
}
```

Our `AuthProvider` component handles:
- Checking for an existing session when the application loads
- Subscribing to auth state changes from Supabase
- Providing wrapper methods around our service layer
- Making auth state and methods available to the rest of the application

## Setting Up the Auth Callback Handler

For OAuth providers like Google, Apple, and Twitch, we need to handle the callback after the user authenticates with the third-party service:

```typescript
import { useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import { authService } from './supabaseService';
import log from 'loglevel';

export function AuthCallback() {
  const navigate = useNavigate();

  useEffect(() => {
    // Handle the OAuth callback
    const handleAuthCallback = async () => {
      try {
        // The Supabase Auth client will automatically handle the token exchange
        const user = await authService.getCurrentUser();
        
        if (user) {
          log.info('OAuth authentication successful');
          // Redirect to the dashboard or home page
          navigate('/dashboard');
        } else {
          log.warn('OAuth authentication failed');
          navigate('/login', { 
            state: { error: 'Authentication failed. Please try again.' } 
          });
        }
      } catch (error) {
        log.error('Auth callback error:', error);
        navigate('/login', { 
          state: { error: 'Authentication failed. Please try again.' } 
        });
      }
    };

    handleAuthCallback();
  }, [navigate]);

  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="text-center">
        <h2 className="text-2xl font-bold mb-2">Completing authentication...</h2>
        <p>Please wait while we complete the authentication process.</p>
      </div>
    </div>
  );
}
```

## Creating Login and Signup Components with shadcn

Now, let's create our login and signup components using shadcn UI components:

```tsx
import { useState } from 'react';
import { useNavigate, useLocation } from 'react-router-dom';
import { useAuth } from './AuthContext';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from '@/components/ui/card';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { Label } from '@/components/ui/label';

export function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const { login, loginWithGoogle, loginWithApple, loginWithTwitch } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  
  // Check if we have an error from a redirect
  const redirectError = location.state?.error;
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsLoading(true);
    setError(null);
    
    try {
      const { success, error } = await login(email, password);
      
      if (success) {
        navigate('/dashboard');
      } else {
        setError(error || 'Login failed. Please try again.');
      }
    } catch (err) {
      setError('An unexpected error occurred. Please try again.');
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <div className="flex items-center justify-center min-h-screen p-4">
      <Card className="w-full max-w-md">
        <CardHeader>
          <CardTitle>Log In</CardTitle>
          <CardDescription>
            Enter your credentials to access your account
          </CardDescription>
        </CardHeader>
        <CardContent>
          {(error || redirectError) && (
            <Alert variant="destructive" className="mb-4">
              <AlertDescription>
                {error || redirectError}
              </AlertDescription>
            </Alert>
          )}
          
          <form onSubmit={handleSubmit} className="space-y-4">
            <div className="space-y-2">
              <Label htmlFor="email">Email</Label>
              <Input
                id="email"
                type="email"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                required
                placeholder="you@example.com"
              />
            </div>
            
            <div className="space-y-2">
              <Label htmlFor="password">Password</Label>
              <Input
                id="password"
                type="password"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                required
                placeholder="••••••••"
              />
            </div>
            
            <Button
              type="submit"
              className="w-full"
              disabled={isLoading}
            >
              {isLoading ? 'Logging in...' : 'Log In'}
            </Button>
          </form>
          
          <div className="relative my-6">
            <div className="absolute inset-0 flex items-center">
              <span className="w-full border-t" />
            </div>
            <div className="relative flex justify-center text-sm">
              <span className="bg-background px-2 text-muted-foreground">
                Or continue with
              </span>
            </div>
          </div>
          
          <div className="grid grid-cols-3 gap-2">
            <Button variant="outline" onClick={loginWithGoogle}>
              Google
            </Button>
            <Button variant="outline" onClick={loginWithApple}>
              Apple
            </Button>
            <Button variant="outline" onClick={loginWithTwitch}>
              Twitch
            </Button>
          </div>
        </CardContent>
        <CardFooter className="flex justify-center">
          <p className="text-sm text-muted-foreground">
            Don't have an account?{' '}
            <a 
              href="/signup" 
              className="text-primary hover:underline"
              onClick={(e) => {
                e.preventDefault();
                navigate('/signup');
              }}
            >
              Sign up
            </a>
          </p>
        </CardFooter>
      </Card>
    </div>
  );
}
```

And similarly for the signup component:

```tsx
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from './AuthContext';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from '@/components/ui/card';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { Label } from '@/components/ui/label';

export function Signup() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const { signUp } = useAuth();
  const navigate = useNavigate();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (password !== confirmPassword) {
      setError('Passwords do not match');
      return;
    }
    
    setIsLoading(true);
    setError(null);
    
    try {
      const { success, error } = await signUp(email, password);
      
      if (success) {
        navigate('/login', { 
          state: { 
            message: 'Registration successful! Please check your email to confirm your account.' 
          } 
        });
      } else {
        setError(error || 'Registration failed. Please try again.');
      }
    } catch (err) {
      setError('An unexpected error occurred. Please try again.');
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <div className="flex items-center justify-center min-h-screen p-4">
      <Card className="w-full max-w-md">
        <CardHeader>
          <CardTitle>Create an Account</CardTitle>
          <CardDescription>
            Sign up to start using Hero Wars Helper
          </CardDescription>
        </CardHeader>
        <CardContent>
          {error && (
            <Alert variant="destructive" className="mb-4">
              <AlertDescription>{error}</AlertDescription>
            </Alert>
          )}
          
          <form onSubmit={handleSubmit} className="space-y-4">
            <div className="space-y-2">
              <Label htmlFor="email">Email</Label>
              <Input
                id="email"
                type="email"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                required
                placeholder="you@example.com"
              />
            </div>
            
            <div className="space-y-2">
              <Label htmlFor="password">Password</Label>
              <Input
                id="password"
                type="password"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                required
                placeholder="••••••••"
              />
            </div>
            
            <div className="space-y-2">
              <Label htmlFor="confirmPassword">Confirm Password</Label>
              <Input
                id="confirmPassword"
                type="password"
                value={confirmPassword}
                onChange={(e) => setConfirmPassword(e.target.value)}
                required
                placeholder="••••••••"
              />
            </div>
            
            <Button
              type="submit"
              className="w-full"
              disabled={isLoading}
            >
              {isLoading ? 'Creating account...' : 'Sign Up'}
            </Button>
          </form>
        </CardContent>
        <CardFooter className="flex justify-center">
          <p className="text-sm text-muted-foreground">
            Already have an account?{' '}
            <a 
              href="/login" 
              className="text-primary hover:underline"
              onClick={(e) => {
                e.preventDefault();
                navigate('/login');
              }}
            >
              Log in
            </a>
          </p>
        </CardFooter>
      </Card>
    </div>
  );
}
```

## Creating a Protected Route Component

Now let's create a component to handle protected routes:

```tsx
import { ReactNode } from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from './AuthContext';

interface ProtectedRouteProps {
  children: ReactNode;
  redirectTo?: string;
}

export function ProtectedRoute({ 
  children, 
  redirectTo = '/login' 
}: ProtectedRouteProps) {
  const { user, isLoading } = useAuth();
  const location = useLocation();
  
  // Show loading state while checking authentication
  if (isLoading) {
    return (
      <div className="flex items-center justify-center min-h-screen">
        <div className="text-center">
          <h2 className="text-2xl font-bold mb-2">Loading...</h2>
          <p>Please wait while we verify your authentication.</p>
        </div>
      </div>
    );
  }
  
  // Redirect to login if not authenticated
  if (!user) {
    return <Navigate to={redirectTo} state={{ from: location }} replace />;
  }
  
  // User is authenticated, render the protected content
  return <>{children}</>;
}
```

## Setting Up Routes in React Router v7

Now, let's integrate everything with React Router:

```tsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import { AuthProvider } from './AuthContext';
import { ProtectedRoute } from './ProtectedRoute';
import { Login } from './Login';
import { Signup } from './Signup';
import { AuthCallback } from './AuthCallback';
import { Dashboard } from './Dashboard';
import { HomePage } from './HomePage';
import { ErrorBoundary } from './ErrorBoundary';

// Create the router configuration
const router = createBrowserRouter([
  {
    path: '/',
    element: <HomePage />,
    errorElement: <ErrorBoundary />,
  },
  {
    path: '/login',
    element: <Login />,
  },
  {
    path: '/signup',
    element: <Signup />,
  },
  {
    path: '/auth/callback',
    element: <AuthCallback />,
  },
  {
    path: '/dashboard',
    element: (
      <ProtectedRoute>
        <Dashboard />
      </ProtectedRoute>
    ),
  },
  // Add more protected routes as needed
]);

export function App() {
  return (
    <AuthProvider>
      <RouterProvider router={router} />
    </AuthProvider>
  );
}
```

## Creating an Error Boundary Component

Let's create a simple error boundary component to handle routing errors:

```tsx
import { useRouteError } from 'react-router-dom';
import log from 'loglevel';

export function ErrorBoundary() {
  const error = useRouteError();
  
  // Log the error
  log.error('Route error:', error);
  
  return (
    <div className="flex items-center justify-center min-h-screen p-4">
      <div className="text-center">
        <h1 className="text-4xl font-bold mb-4">Oops!</h1>
        <p className="text-xl mb-6">Something went wrong.</p>
        <div className="bg-gray-100 p-4 rounded-md text-left overflow-auto max-w-2xl">
          {error instanceof Error ? (
            <>
              <p className="font-bold">{error.name}: {error.message}</p>
              <pre className="mt-2 text-sm">{error.stack}</pre>
            </>
          ) : (
            <p className="font-bold">An unknown error occurred</p>
          )}
        </div>
        <a 
          href="/"
          className="inline-block mt-6 px-4 py-2 bg-primary text-white rounded hover:bg-primary/90"
        >
          Go back home
        </a>
      </div>
    </div>
  );
}
```

## Setting Up Environment Variables

Don't forget to set up your environment variables. Create a `.env` file in the root of your project:

```
VITE_SUPABASE_URL=your-supabase-url
VITE_SUPABASE_ANON_KEY=your-supabase-anon-key
```

## Testing the Authentication Flow

Let's do a quick walkthrough of how the authentication flow works:

1. A new user visits the application and sees the home page.
2. They click "Sign Up" and are taken to the signup page.
3. After creating an account, they're redirected to the login page with a message to check their email.
4. After confirming their email, they log in with their credentials.
5. Once authenticated, they can access protected routes like the dashboard.
6. If they try to access a protected route without being authenticated, they're redirected to the login page.
7. When they're done, they can log out, which clears their session.

## Conclusion

We've successfully implemented a modular authentication system using Supabase Auth in a React Router v7 application with TypeScript. Our implementation follows these key principles:

1. **Separation of concerns** - By abstracting Supabase-specific code into a service layer, we've made it easy to swap auth providers in the future.
2. **Type safety** - TypeScript ensures that our auth operations are used correctly throughout the application.
3. **Consistent error handling** - We're using loglevel for error logging and providing user-friendly error messages.
4. **Clean UI** - Our shadcn components create a polished user experience.
5. **Protected routes** - We're using React Router's capabilities to protect sensitive routes.

By following this approach, you can implement authentication in your own React applications while maintaining flexibility and clean code architecture. The modular design ensures that if you ever need to switch from Supabase to another auth provider, you'll only need to update the service layer without changing the rest of your application.

Remember to properly secure your environment variables and follow best practices for authentication in production applications, such as implementing proper CORS settings, using HTTPS, and regularly rotating API keys.

---

*Want to see this in action? Check out the [Hero Wars Helper repository](https://github.com/yourusername/herowars-helper) for a complete implementation.*