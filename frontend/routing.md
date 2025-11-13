# React Router Architecture Guide

## Overview

This application uses React Router with a well-structured routing architecture that emphasizes:

- **Hierarchical routing** with nested routes
- **Centralized route constants** for maintainability
- **Loader-based authentication** and data fetching
- **Consistent naming conventions** and file organization

## Project Structure

### Route Configuration Files

```
src/
├── router.tsx                    # Main router configuration
├── constants/
│   ├── routes.ts                # Route path constants
│   └── namespaces.ts            # Translation namespace constants
└── pages/
    ├── page-name/
    │   ├── page-name.route.tsx  # Route definition (exports RouteObject)
    │   ├── page-name.view.tsx   # Page component
    │   └── page-name.spec.tsx   # Page tests
    └── example-page/
        ├── example.route.tsx    # Main example route
        ├── example.view.tsx     # Example component
        ├── example.spec.tsx     # Example tests
        └── sub-page/
            ├── sub-page.route.tsx # Nested route
            ├── sub-page.view.tsx  # Sub-page component
            └── sub-page.spec.tsx  # Sub-page tests
```

### Key Files

- **`src/router.tsx`**: Main router configuration that imports and uses route objects from pages
- **`src/constants/routes.ts`**: All route path constants defined here
- **`src/constants/namespaces.ts`**: Translation namespace constants
- **`*.route.tsx`**: Individual route definitions that export a `RouteObject` - these live alongside their page components

### Page Structure Convention

Each page folder should contain three files:

1. **`[page-name].route.tsx`**: Exports the route configuration object (`RouteObject`)
2. **`[page-name].view.tsx`**: The page component/view
3. **`[page-name].spec.tsx`**: Tests for the page

The route file exports everything needed for the router, making it easy to import and use in the main router configuration.

## Route Constants Architecture

### Hierarchical Constant Definition

All routes are defined as constants in `src/constants/routes.ts`:

```typescript
// Base routes
export const ROUTE_LOGIN = "/login";
export const ROUTE_DASHBOARD = "/d";

// Hierarchical routes using base constants
export const ROUTE_DASHBOARD_CUSTOMER_LIST = `${ROUTE_DASHBOARD}/customers`;
export const ROUTE_CUSTOMER_DETAILS = `${ROUTE_DASHBOARD}/customers/:id`;

// Nested routes with parameters
export const ROUTE_CUSTOMER_DETAILS_PHOTOS = `${ROUTE_DASHBOARD}/customers/:id/photos`;
export const ROUTE_CUSTOMER_DETAILS_METRICS = `${ROUTE_DASHBOARD}/customers/:id/metrics`;
```

### Route Naming Convention

Routes follow a consistent naming pattern:

- **Base routes**: `ROUTE_[PAGE_NAME]` (e.g., `ROUTE_LOGIN`, `ROUTE_DASHBOARD`)
- **Nested routes**: `ROUTE_[PARENT]_[CHILD]` (e.g., `ROUTE_DASHBOARD_CUSTOMER_LIST`)
- **Parameterized routes**: Use React Router parameter syntax (e.g., `:id`, `:conversationId`)

## Router Configuration

### Main Router Setup (`src/router.tsx`)

The main router file imports route objects from their respective page folders and assembles them:

```typescript
import { createBrowserRouter, Outlet } from "react-router-dom";
import type { RouteObject } from "react-router-dom";
import { ROUTE_ROOT, ROUTE_DASHBOARD } from "@/constants/routes";
import { homePageRoute } from "@/pages/home-page/home-page.route";
import { loginRoute } from "@/pages/login-page/login-page.route";
import { dashboardRoute } from "@/pages/dashboard-page/dashboard.route";
import { registerRoute } from "@/pages/register-page/register-page.route";
// ... other route imports

export const routes: RouteObject[] = [
  {
    path: ROUTE_ROOT,
    element: <Outlet />,
    children: [
      homePageRoute,
      loginRoute,
      dashboardRoute,
      registerRoute,
      // ... other routes
    ],
    loader(args) {
      // Root loader with default redirect
      const url = new URL(args.request.url);
      if (url.pathname === ROUTE_ROOT) {
        return redirect(ROUTE_DASHBOARD);
      }
      return null;
    },
  },
];

export const router = createBrowserRouter(routes);
```

### Route Configuration in App Component

```typescript
// src/App.tsx
export function App() {
  return (
    <Provider>
      <QueryClientProvider client={queryClient}>
        <RouterProvider router={createBrowserRouter(routes)} />
      </QueryClientProvider>
    </Provider>
  );
}
```

## Route Definition Patterns

### Basic Route Pattern

Each route is defined in its own `*.route.tsx` file **located alongside the page component**. The route file exports a `RouteObject` that contains all the route configuration:

```typescript
// src/pages/login-page/login-page.route.tsx
import type { RouteObject } from "react-router-dom";
import { ROUTE_LOGIN } from "@/constants/routes";
import LoginPage from "./login-page.view";

export const loginRoute: RouteObject = {
  path: ROUTE_LOGIN,
  element: <LoginPage />,
  loader() {
    return i18next.loadNamespaces([NS_LOGIN]).then(() => null);
  },
};
```

Then in the main router, simply import and use the route object:

```typescript
// src/router.tsx
import { createBrowserRouter, Outlet } from "react-router-dom";
import type { RouteObject } from "react-router-dom";
import { ROUTE_ROOT } from "@/constants/routes";
import { loginRoute } from "@/pages/login-page/login-page.route";
import { homePageRoute } from "@/pages/home-page/home-page.route";

export const routes: RouteObject[] = [
  {
    path: ROUTE_ROOT,
    element: <Outlet />,
    children: [
      homePageRoute,
      loginRoute,
      // ... other routes
    ],
  },
];
```

### Complete Page Structure Example

A complete page folder structure looks like this:

```
pages/
└── home-page/
    ├── home-page.route.tsx    # Exports homePageRoute: RouteObject
    ├── home-page.view.tsx     # HomePage component
    └── home-page.spec.tsx     # Tests for HomePage
```

**Benefits of this approach:**

- Routes live next to their components, keeping related code together
- Easy to find and maintain route definitions
- Clean router file that just imports and assembles routes
- Each page is self-contained with its route, view, and tests

### Protected Route Pattern

Routes requiring authentication use loaders for protection. The route file exports the complete route configuration:

```typescript
// src/pages/dashboard-page/dashboard.route.tsx
import type { RouteObject } from "react-router-dom";
import { ROUTE_DASHBOARD, ROUTE_LOGIN } from "@/constants/routes";
import DashboardPage from "./dashboard-page.view";
import { customerListRoute } from "../customer-list/customer-list.route";
import { customerDetailsRoute } from "../customer-details/customer-details.route";

export const dashboardRoute: RouteObject = {
  path: ROUTE_DASHBOARD,
  element: <DashboardPage />,
  children: [
    customerListRoute,
    customerDetailsRoute,
    // ... other nested routes
  ],
  async loader() {
    // Load required namespaces
    await Promise.all([i18next.loadNamespaces([NS_DASHBOARD_INDEX, NS_PROFILE])]);

    // Authentication check
    if (!authService.isAuthenticated()) {
      return redirect(ROUTE_LOGIN);
    }

    return null;
  },
};
```

### Nested Route Pattern

Nested routes are defined as children of parent routes:

```typescript
// Parent route with children
export const customerDetailsRoute: RouteObject = {
  path: ROUTE_CUSTOMER_DETAILS,
  element: <CustomerDetailsPage />,
  children: [
    customerDetailsPlannerRoute,
    customerDetailsPhotosRoute,
    customerDetailsMetricsRoute,
    // ... other nested routes
  ],
  loader() {
    if (!authService.isAuthenticated()) {
      return redirect(ROUTE_LOGIN);
    }
    return i18next.loadNamespaces([NS_CUSTOMER_DETAILS]).then(() => null);
  },
};
```

## Authentication Patterns

### Authentication Check in Loaders

```typescript
loader() {
  // Standard auth check pattern
  if (!authService.isAuthenticated()) {
    return redirect(ROUTE_LOGIN);
  }
  return null;
}
```

### Combined Authentication and Data Loading

```typescript
async loader() {
  // Load translations first
  await i18next.loadNamespaces([NS_DASHBOARD_INDEX, NS_PROFILE]);

  // Then check authentication
  if (!authService.isAuthenticated()) {
    return redirect(ROUTE_LOGIN);
  }

  return null;
}
```

## Translation Namespace Integration

The routing system integrates with the internationalization system to preload translation namespaces before rendering components. This ensures all texts are available immediately when components mount.

For detailed information about how internationalization works in this application, including namespace organization, translation loading strategies, and best practices, see the [Internationalization Guide](./i18n.md).

In route loaders, namespaces are loaded using:

```typescript
loader() {
  return i18next.loadNamespaces([NS_CUSTOMER_LIST]).then(() => null);
}
```

## Special Route Patterns

### Index Routes

Routes that should render when the parent path is matched exactly:

```typescript
export const routeDashboardIndex: RouteObject = {
  index: true,
  element: <DashboardPageView />,
  loader() {
    if (!authService.isAuthenticated()) {
      return redirect(ROUTE_LOGIN);
    }
    return redirect(ROUTE_DASHBOARD_CUSTOMER_LIST);
  },
};
```

### Dynamic Routes with Parameters

```typescript
// Route with parameter
export const customerDetailsRoute: RouteObject = {
  path: ROUTE_CUSTOMER_DETAILS, // "/d/customers/:id"
  element: <CustomerDetailsPage />,
  // ...
};

// Accessing parameters in components
const { id } = useParams<{ id: string }>();
```

### Layout Routes with Outlet

```typescript
export const libraryRoute: RouteObject = {
  path: ROUTE_DASHBOARD_LIBRARY,
  element: <Outlet />, // Renders child routes
  children: [
    exerciseListRoute,
    exerciseDetailsRoute,
    // ... other library routes
  ],
  loader() {
    return i18next.loadNamespaces([NS_LIBRARY]).then(() => null);
  },
};
```

## Legacy Route Compatibility

The application handles legacy routes for backward compatibility:

```typescript
// Legacy Angular route compatibility
const legacyResetPasswordRoute: RouteObject = {
  path: "/reset-pwd/:token",
  loader(args) {
    const token = args.params.token;
    return redirect(`/reset-password?token=${token}`);
  },
};
```

## Best Practices

### 1. Route Organization

- **Routes live with pages**: Each route is defined in a `*.route.tsx` file **alongside** the page component (`*.view.tsx`) and tests (`*.spec.tsx`)
- **Consistent naming**: Follow the `[page-name].route.tsx` naming convention
- **Export route objects**: Each `*.route.tsx` file exports a `RouteObject` that can be imported in the main router
- **Self-contained pages**: Each page folder contains route, view, and test files together

### 2. Route Constants

- **Centralized definition**: All routes defined in `src/constants/routes.ts`
- **Hierarchical structure**: Use base constants to build nested routes
- **Parameter naming**: Use clear, descriptive parameter names (`:id`, `:conversationId`)

### 3. Authentication

- **Loader-based protection**: Use loaders for route-level authentication
- **Consistent redirects**: Always redirect to `ROUTE_LOGIN` for unauthenticated users
- **Early authentication checks**: Check authentication before loading data

### 4. Data Loading

- **Namespace preloading**: Load required translation namespaces in route loaders
- **Async loading**: Use `Promise.all()` for parallel loading of multiple namespaces
- **Error handling**: Handle loading failures gracefully

### 5. Route Structure

```typescript
// Recommended route structure in [page-name].route.tsx
import type { RouteObject } from "react-router-dom";
import { ROUTE_PAGE_NAME } from "@/constants/routes";
import PageNameComponent from "./page-name.view";

export const pageNameRoute: RouteObject = {
  path: ROUTE_PAGE_NAME, // Use constant from routes.ts
  element: <PageNameComponent />, // Import component from same folder
  children: [], // Child routes if needed
  loader() {
    // Authentication & data loading
    // 1. Load namespaces
    // 2. Check authentication
    // 3. Return data or redirect
  },
};
```

Then in `router.tsx`:

```typescript
import { createBrowserRouter, Outlet } from "react-router-dom";
import type { RouteObject } from "react-router-dom";
import { pageNameRoute } from "@/pages/page-name/page-name.route";

export const routes: RouteObject[] = [
  {
    path: ROUTE_ROOT,
    element: <Outlet />,
    children: [
      pageNameRoute, // Simply import and use
      // ... other routes
    ],
  },
];
```

## Common Patterns Summary

1. **Public Routes**: Simple path + element + namespace loading
2. **Protected Routes**: Add authentication check in loader
3. **Nested Routes**: Parent route with children array
4. **Index Routes**: Use `index: true` for default child route
5. **Parameterized Routes**: Use `:parameter` syntax in path
6. **Layout Routes**: Use `<Outlet />` element for child route rendering

This architecture provides a scalable, maintainable routing system that handles authentication, internationalization, and complex nested navigation patterns effectively.
