## Google Firebase Login Implementation for Nextjs App

To integrate Firebase login with Google Sign-In in a Next.js application, we’ll go through the following steps:

1. **Set up Firebase Project** and configure Firebase in Next.js.
2. **Install Firebase SDK**.
3. **Set up Google Sign-In with Firebase in the Firebase Console**.
4. **Implement Google Sign-In using Firebase Authentication**.
5. **Protect pages by creating a custom authentication provider**.

### Step 1: Set Up Firebase Project

1. Go to the [Firebase Console](https://console.firebase.google.com/), create a new project, or use an existing one.
2. Register your web application within this project.
3. Once registered, Firebase will provide a configuration object (API keys and other settings) that will be used in your Next.js application.

### Step 2: Enable Google Sign-In in Firebase Console

1. Go to **Authentication** in the Firebase Console.
2. Under **Sign-in method**, enable **Google** as a sign-in provider.

### Step 3: Install Firebase SDK in Next.js

Install the Firebase JavaScript SDK via npm.

```bash
npm install firebase
```

### Step 4: Configure Firebase in Next.js

1. Create a Firebase configuration file in your project to initialize Firebase.
2. Add your Firebase credentials in an `.env.local` file to keep them secure.

#### Directory Structure

Here's a suggested structure for implementing Firebase Authentication in Next.js:

```plaintext
my-nextjs-app/
│
├── public/
├── src/
│   ├── components/
│   │   ├── AuthProvider.js       # Custom Firebase Auth Context
│   │   ├── LoginButton.js        # Google Login Button Component
│   │   └── ProtectedRoute.js     # Component to protect routes
│   ├── pages/
│   │   ├── index.js              # Home page (public)
│   │   ├── dashboard.js          # Example protected page
│   ├── firebase/
│   │   └── firebaseConfig.js     # Firebase configuration and initialization
├── .env.local                    # Environment variables
└── package.json
```

#### .env.local

```plaintext
NEXT_PUBLIC_FIREBASE_API_KEY=your_api_key
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_auth_domain
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_storage_bucket
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_messaging_sender_id
NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id
```

#### src/firebase/firebaseConfig.js

```javascript
import { initializeApp, getApps, getApp } from "firebase/app";
import { getAuth, GoogleAuthProvider } from "firebase/auth";

// Check if Firebase is already initialized to avoid reinitializing
const firebaseConfig = {
    apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
    authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
    projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
    storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
    messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
    appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
};

const app = !getApps().length ? initializeApp(firebaseConfig) : getApp();
const auth = getAuth(app);
const provider = new GoogleAuthProvider();

export { auth, provider };
```

### Step 5: Create Authentication Context Provider

In Next.js, creating a custom authentication context helps to keep track of user login status and manage session data across the app.

#### src/components/AuthProvider.js

```javascript
import { createContext, useContext, useEffect, useState } from "react";
import { auth } from "../firebase/firebaseConfig";
import { onAuthStateChanged, signOut } from "firebase/auth";

const AuthContext = createContext();

export function AuthProvider({ children }) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        const unsubscribe = onAuthStateChanged(auth, (user) => {
            setUser(user);
            setLoading(false);
        });
        return () => unsubscribe();
    }, []);

    const logout = () => signOut(auth);

    return (
        <AuthContext.Provider value={{ user, logout }}>
            {!loading && children}
        </AuthContext.Provider>
    );
}

export function useAuth() {
    return useContext(AuthContext);
}
```

### Step 6: Implement Google Sign-In Button

#### src/components/LoginButton.js

This component will handle the Google Sign-In action.

```javascript
import { signInWithPopup } from "firebase/auth";
import { auth, provider } from "../firebase/firebaseConfig";

export default function LoginButton() {
    const handleLogin = async () => {
        try {
            await signInWithPopup(auth, provider);
            console.log("User signed in");
        } catch (error) {
            console.error("Error during sign-in:", error);
        }
    };

    return (
        <button onClick={handleLogin}>
            Sign in with Google
        </button>
    );
}
```

### Step 7: Protect Routes

Create a Protected Route wrapper component to restrict access to certain pages based on authentication.

#### src/components/ProtectedRoute.js

```javascript
import { useRouter } from "next/router";
import { useAuth } from "./AuthProvider";
import { useEffect } from "react";

export default function ProtectedRoute({ children }) {
    const { user } = useAuth();
    const router = useRouter();

    useEffect(() => {
        if (!user) {
            router.push("/");
        }
    }, [user, router]);

    if (!user) return null; // Optionally show a loading spinner

    return children;
}
```

### Step 8: Use Authentication Provider in the Application

Wrap the app in the `AuthProvider` context in `pages/_app.js` to make the authentication state available across the app.

#### pages/_app.js

```javascript
import { AuthProvider } from "../components/AuthProvider";

function MyApp({ Component, pageProps }) {
    return (
        <AuthProvider>
            <Component {...pageProps} />
        </AuthProvider>
    );
}

export default MyApp;
```

### Step 9: Use the ProtectedRoute Component

Now you can wrap any page with `ProtectedRoute` to make it accessible only to authenticated users.

#### pages/dashboard.js

```javascript
import ProtectedRoute from "../components/ProtectedRoute";
import { useAuth } from "../components/AuthProvider";

export default function Dashboard() {
    const { user, logout } = useAuth();

    return (
        <ProtectedRoute>
            <div>
                <h1>Dashboard</h1>
                <p>Welcome, {user?.displayName}</p>
                <button onClick={logout}>Logout</button>
            </div>
        </ProtectedRoute>
    );
}
```

#### pages/index.js

Display the login button on the home page (or wherever you want the user to log in).

```javascript
import LoginButton from "../components/LoginButton";
import { useAuth } from "../components/AuthProvider";

export default function Home() {
    const { user } = useAuth();

    return (
        <div>
            <h1>Welcome to My Next.js App</h1>
            {user ? (
                <p>Logged in as: {user.displayName}</p>
            ) : (
                <LoginButton />
            )}
        </div>
    );
}
```

### Final Considerations

- **Environment Variables**: Always keep Firebase configuration variables in `.env.local` for security.
- **Route Protection**: You can use the `ProtectedRoute` component on any page that requires login.
- **Session Handling**: Firebase manages sessions natively, but logging out clears session data for a clean experience.

This setup gives you a fully functional Google login in Next.js using Firebase, and you can easily extend it to support other OAuth providers as needed.
