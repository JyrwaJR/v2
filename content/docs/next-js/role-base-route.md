---
title: "Role-Based Routing"
description: "Role-based routing"
---

This document provides an overview of implementing role-based routing with user authentication in a Next.js application. The solution presented handles various use cases such as:

1. Redirecting back to the previously visited page after login.
2. Checking for a user's role before granting access to specific pages.
3. Preventing authenticated users from accessing unauthenticated-only pages.

## Overview

In this solution, we utilize a `RoleBaseRoute` component to manage user authentication and access control. This component ensures that only authenticated users with the appropriate roles can access certain pages, while also redirecting users to login pages when necessary. Additionally, it automatically redirects users back to the page they originally intended to visit after they log in.

### Key Features:

- **Redirect After Login**: If a user visits a protected page without being authenticated, they are redirected to the login page, and upon successful login, they are redirected back to the original page they were trying to access.
- **Role-Based Access Control (RBAC)**: The component checks if the authenticated user has the required roles to access specific routes. If they do not, they are redirected to a fallback page.
- **Prevention of Unauthorized Access**: Authenticated users are prevented from accessing certain routes (like `/signin`, `/signup`, etc.) that should only be accessible by unauthenticated users.

## Component Breakdown

The core of this solution is the `RoleBaseRoute` component which is responsible for handling all of the logic for route access based on authentication status and roles.

```typescript format:prettier

type RoleRoute = {
  url: string;
  role: string[];
  redirect?: string;
  needAuth?: boolean;
};

const routeRoles: RoleRoute[] = [
  {
    url: "/admin/*",
    role: ["superadmin"],
    needAuth: true,
  },
  {
    url: "/contact/*",
    role: ["anonymous", "user", "admin", "superadmin"],
  },
];

type PropsT = {
  children: React.ReactNode;
};

const pageAccessOnlyIfUnAuthenticated: string[] = [
  "/signin",
  "/signup",
  "/reset-password",
  "/verify-email",
];


export const RoleBaseRoute = ({ children }: PropsT) => {
  const searchParams = useSearchParams();
  const redirectTo = searchParams.get("redirect");
  const router = useRouter();
  const pathName = usePathname();
  const [isLoading, setIsLoading] = useState(false);
  const { user, isLoading: isAuthLoading } = useAuth();
  const [cookies] = useCookies(["AUTH_TOKEN"]);// Get the AUTH_TOKEN cookie
  const userRoles = useMemo(() => user?.role || [], [user]);// Get the user's roles
  const isAuthenticated = !!(user && cookies.AUTH_TOKEN);//

  // Show loader during route changes or delays
  useEffect(() => {
    if (isAuthLoading) return;
    setIsLoading(true);
    const timer = setTimeout(() => setIsLoading(false), 1000);// delay in ms
    return () => clearTimeout(timer); // Cleanup the timer
  }, [isAuthLoading, pathName]);

  // Handle authentication and role-based redirects
  useEffect(() => {
    // Wait until authentication loading is complete to proceed
    if (isAuthLoading) return;

    // Step 1: Identify the current route from the `routeRoles` configuration
    const currentRoute = routeRoles.find((route) => {
      if (route.url === pathName) return true; // Direct match for the route
      if (route.url.endsWith("/*")) {
        const basePath = route.url.replace("/*", ""); // Handle wildcard route (e.g., `/dashboard/*`)
        return pathName.startsWith(basePath); // Check if the current path starts with the base path
      }
      return false; // No match found
    });

    // Step 2: Handle authentication-based redirection
    if (currentRoute) {
      // If the route requires authentication and the user is not authenticated
      if (currentRoute.needAuth && !isAuthenticated) {
        // Redirect the user to the signin page and include the current path as a `redirect` query parameter
        router.replace(`/signin?redirect=${encodeURIComponent(pathName)}`);
        return; // Exit the logic as redirection is in progress
      }

      // Step 3: Handle role-based access control
      if (isAuthenticated) {
        // Check if the user has at least one of the required roles for the current route
        const hasRequiredRole = currentRoute.role.some((role) =>
          userRoles.includes(role),
        );

        // If the user does not have the required role(s)
        if (!hasRequiredRole) {
          // Redirect the user to a fallback page specified in the route's configuration or to the homepage
          router.replace(currentRoute.redirect || "/");
          return; // Exit the logic as redirection is in progress
        }
      }
    }
  }, [pathName, isAuthenticated, userRoles, router, isAuthLoading]);

  // Prevent authenticated users from accessing unauthenticated-only pages
  useEffect(() => {
    if (isAuthLoading || isLoading) return;
    if (isAuthenticated && pageAccessOnlyIfUnAuthenticated.includes(pathName)) {
      router.push(redirectTo || "/");
    }
  }, [isAuthenticated, pathName, redirectTo, router, isAuthLoading, isLoading]);

  // Display preloader if authentication or loading is in progress
  if (isAuthLoading || isLoading) {
    return <PreLoader />;
  }

  return <>{children}</>;
};
```

### `PropsT`

The `RoleBaseRoute` component expects a `children` prop, which represents the content that will be displayed when the route is accessible.

### `pageAccessOnlyIfUnAuthenticated`

This array holds the routes that should only be accessible to unauthenticated users. If an authenticated user tries to visit one of these routes, they are redirected to the page they were trying to visit before login.

## Key Concepts

### 1. **User Authentication (`isAuthenticated`)**

The `isAuthenticated` variable determines whether the user is logged in or not. It's derived from a combination of the user's state and the presence of an authentication token (`AUTH_TOKEN`) stored in cookies.

### 2. **User Roles**

The `userRoles` variable contains the roles of the authenticated user. This is important for determining whether a user has the necessary permissions to access certain routes.

### 3. **Protected Routes and Role Validation**

Each route that requires authentication and/or role validation is matched by comparing the `pathName` with the configuration in `routeRoles`. If the route matches, the system checks whether the user is authenticated and has the required roles.

### 4. **Redirection Logic**

- If the user is not authenticated and tries to access a protected route, they are redirected to the login page (`/signin`) with the current page URL as a `redirect` query parameter.
- After login, the user is redirected back to the original page.
- If the user is authenticated but lacks the required roles, they are redirected to a fallback page (either a specified `redirect` or the homepage).

## How It Works

### 1. **Authentication and Role Checks**

The component uses `useEffect` hooks to handle redirections and role checks:

- The first effect ensures that if the route requires authentication and the user is not logged in, the user is redirected to the login page (`/signin?redirect=...`).
- The second effect prevents authenticated users from accessing pages that are meant for unauthenticated users (e.g., `/signin`, `/signup`).

### 2. **Loader State**

The component displays a preloader (`<PreLoader />`) when the authentication status or route data is loading, providing a smooth user experience.

### 3. **Role Validation**

The component compares the user's roles to the required roles of the route and redirects the user if they lack the necessary permissions.

## Code Explanation

```typescript
const { user, isLoading: isAuthLoading } = useAuth();
const [cookies] = useCookies(["AUTH_TOKEN"]);
const userRoles = useMemo(() => user?.role || [], [user]);
const isAuthenticated = !!(user && cookies.AUTH_TOKEN);
```

- `useAuth()` provides the current user state and authentication status.
- `useCookies(["AUTH_TOKEN"])` checks for the presence of an authentication token in the user's cookies.
- `userRoles` is an array containing the roles of the authenticated user.

```typescript
useEffect(() => {
  if (isAuthLoading) return;
  setIsLoading(true);
  const timer = setTimeout(() => setIsLoading(false), 500);
  return () => clearTimeout(timer);
}, [isAuthLoading, pathName]);
```

- This effect sets a loading state while waiting for authentication to complete.

```typescript
useEffect(() => {
  if (isAuthLoading) return;
  const currentRoute = routeRoles.find((route) => {
    if (route.url === pathName) return true;
    if (route.url.endsWith("/*")) {
      const basePath = route.url.replace("/*", "");
      return pathName.startsWith(basePath);
    }
    return false;
  });

  if (currentRoute) {
    if (currentRoute.needAuth && !isAuthenticated) {
      router.replace(`/signin?redirect=${encodeURIComponent(pathName)}`);
      return;
    }

    if (isAuthenticated) {
      const hasRequiredRole = currentRoute.role.some((role) =>
        userRoles.includes(role),
      );

      if (!hasRequiredRole) {
        router.replace(currentRoute.redirect || "/");
        return;
      }
    }
  }
}, [pathName, isAuthenticated, userRoles, router, isAuthLoading]);
```

- This effect handles the logic for route protection based on authentication and roles. If a route requires authentication and the user is not logged in, the user is redirected to the login page. If the user does not have the required roles, they are redirected to a fallback page.

```typescript
useEffect(() => {
  if (isAuthLoading || isLoading) return;
  if (isAuthenticated && pageAccessOnlyIfUnAuthenticated.includes(pathName)) {
    router.push(redirectTo || "/");
  }
}, [isAuthenticated, pathName, redirectTo, router, isAuthLoading, isLoading]);
```

- This effect prevents authenticated users from accessing unauthenticated-only routes like `/signin` or `/signup`.

## Considerations

- **Fallback Routing**: Always ensure that fallback routes (like `/`) are defined in case of unauthorized access.
- **Token Expiry Handling**: Consider implementing a mechanism to handle expired tokens or session timeouts.
- **User Roles & Permissions**: Define a clear structure for user roles and associated permissions to ensure the correct handling of access rights.

---
