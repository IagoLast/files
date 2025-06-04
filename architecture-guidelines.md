The objective of this post is to establish clear guidelines on how applications should be structured so that they are scalable, easy to understand and easy to test.

# Global architecture schema

In essence we can summarise all frontend applications to a series of well-defined elements. The arrows indicate [coupling](https://connascence.io/) where the source of the arrow is coupled to the target element.

![](https://slabstatic.com/prod/uploads/fcn5vyb4/posts/images/dv4nVcw7bdfMx_jOGMav2fC6.svg)

# Types of files

## Types

They are the fundamental element of our application. Types are used to represent our business entities, function arguments, return objects, and in general any entity represented in our application. The types are only coupled to the language (typescript) and can depend on each other.

To avoid confusion, types are prefixed with the letter I, see [Naming conventions](https://aptopayments.slab.com/posts/xc590vpi#hgbtk-types-interfaces).

### Why Types as Entities?

Using TypeScript types as entities in our architecture brings several significant advantages to our codebase. By treating types as pure data structures without behavior, we create a more functional and maintainable codebase. This approach eliminates the need to reconstruct class instances from server data, making data flow simpler and more predictable throughout the application.

The functional approach with types as entities provides better type safety through TypeScript's compile-time type checking. This creates clear contracts between different parts of the application, making it easier to maintain and refactor as the compiler catches type mismatches early in the development process.

Business logic is cleanly separated into pure functions within services, making the code more testable and maintainable. This separation of concerns between data (types) and behavior (services) leads to better code organization and reusability. Pure functions are easier to test as they have no side effects, and the business logic can be tested independently of the UI.

State management becomes more straightforward as state is treated as pure data. This makes it easier to track changes, implement undo/redo functionality, and integrate with React's state management patterns. The approach also brings performance benefits by eliminating class instantiation overhead and improving memory usage through plain objects.

API integration is simplified with direct mapping between API responses and types, eliminating the need for complex object mapping. This approach also improves error handling by providing clear type definitions and leveraging TypeScript's type system to catch errors at compile time.

The overall maintainability of the codebase is enhanced through predictable data flow, better documentation through types, and easier onboarding for new developers. This architecture aligns perfectly with our goals of building scalable, testable, and maintainable applications while keeping the code simple and easy to understand.

## Services

Services are groups of pure functions where the business logic of our application is defined. Generally these functions will be part of our hooks, but sometimes there is some shared logic throughout the application that needs to be extracted into services.

### Understanding Services in Our Architecture

Services form the backbone of our business logic implementation, following functional programming principles. They are collections of pure functions that operate on our type-defined entities, ensuring predictable and testable behavior throughout the application.

A service is typically organized as a module that exports a set of related functions. These functions should be pure, meaning they always return the same output for the same input and have no side effects. This makes them extremely testable and maintainable.

Here's an example of a well-structured service:

```typescript
// password-validation.service.ts
interface IPasswordValidationResult {
  isValid: boolean;
  errors: string[];
}

function getHasLowercaseChar(password: string): boolean {
  return /[a-z]/.test(password);
}

function getHasMinimumLength(password: string): boolean {
  return password.length >= MIN_REQUIRED_LENGTH;
}

function getHasNumber(password: string): boolean {
  return /\d/.test(password);
}

function validatePassword(password: string): IPasswordValidationResult {
  const errors: string[] = [];
  
  if (!getHasLowercaseChar(password)) {
    errors.push('Password must contain at least one lowercase letter');
  }
  
  if (!getHasMinimumLength(password)) {
    errors.push(`Password must be at least ${MIN_REQUIRED_LENGTH} characters long`);
  }
  
  if (!getHasNumber(password)) {
    errors.push('Password must contain at least one number');
  }
  
  return {
    isValid: errors.length === 0,
    errors
  };
}

export default {
  validatePassword,
  getHasLowercaseChar,
  getHasMinimumLength,
  getHasNumber
};
```

Services can also handle more complex business logic, such as data transformations or calculations. Here's an example of a service that handles financial calculations:

```typescript
// financial-calculations.service.ts
interface ITransaction {
  amount: number;
  currency: string;
  date: Date;
}

interface IBalance {
  total: number;
  currency: string;
  lastUpdated: Date;
}

function calculateBalance(transactions: ITransaction[]): IBalance {
  const total = transactions.reduce((sum, transaction) => sum + transaction.amount, 0);
  const lastUpdated = new Date(Math.max(...transactions.map(t => t.date.getTime())));
  
  return {
    total,
    currency: transactions[0]?.currency || 'USD',
    lastUpdated
  };
}

function formatCurrency(amount: number, currency: string): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency
  }).format(amount);
}

export default {
  calculateBalance,
  formatCurrency
};
```

### Default Exports and Testing

The use of default exports in our services serves a specific purpose: it makes mocking significantly easier in our tests. When we export a service as a default object containing all its functions, we can easily mock individual functions using testing libraries like Vitest:

```typescript
// In a test file
import * as financialService from './financial-calculations.service';

// Mock a specific function
vi.spyOn(financialService.default, 'calculateBalance').mockReturnValue({
  total: 1000,
  currency: 'USD',
  lastUpdated: new Date()
});
```

This approach does come with a trade-off: it can interfere with tree shaking, as the bundler needs to include the entire service object even if only one function is used. However, for web applications, this is generally an acceptable compromise. Modern bundlers are quite efficient at tree shaking even with default exports, and the improved testability and maintainability outweigh the small performance cost. Most services are relatively small and focused, so the impact on bundle size is minimal compared to the benefits in testing.

If bundle size becomes a critical concern, we can always refactor to named exports, but the default export pattern provides the best balance of testability and maintainability for most cases.

The key benefits of this service-based approach include:

1. **Testability**: Pure functions are easy to test as they have no side effects and always return the same output for the same input.

2. **Reusability**: Services can be used across different parts of the application without modification.

3. **Maintainability**: Business logic is centralized and isolated, making it easier to update and maintain.

4. **Type Safety**: Services work with our type-defined entities, ensuring type safety throughout the application.

5. **Separation of Concerns**: Business logic is cleanly separated from UI components and data management.

When using services in our application, we should:

- Keep functions pure and side-effect free
- Use TypeScript types for all inputs and outputs
- Group related functions together in a single service
- Export all functions through a default export
- Document complex business logic with clear comments
- Write comprehensive tests for all service functions

This service-based architecture allows us to build complex applications while maintaining clean, testable, and maintainable code.

## Repositories

Repositories are specialized services that handle data operations, typically interacting with a server through CRUD operations. They act as an abstraction layer between our application and the data source, providing a clean interface for data manipulation.

The foundation of our repository pattern is the API client, a singleton that handles all HTTP communications. We typically use Axios as our HTTP client, configured with interceptors for authentication and common headers:

```typescript
// api-client.service.ts
import axios, { AxiosInstance, AxiosRequestConfig } from 'axios';

class ApiClient {
  private static instance: ApiClient;
  private axiosInstance: AxiosInstance;

  private constructor() {
    this.axiosInstance = axios.create({
      baseURL: process.env.API_BASE_URL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json'
      }
    });

    this.setupInterceptors();
  }

  public static getInstance(): ApiClient {
    if (!ApiClient.instance) {
      ApiClient.instance = new ApiClient();
    }
    return ApiClient.instance;
  }

  private setupInterceptors(): void {
    // Request interceptor for adding auth token
    this.axiosInstance.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('auth_token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor for handling common errors
    this.axiosInstance.interceptors.response.use(
      (response) => response,
      (error) => {
        if (error.response?.status === 401) {
          // Handle unauthorized access
          window.location.href = '/login';
        }
        return Promise.reject(error);
      }
    );
  }

  public async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.axiosInstance.get<T>(url, config);
    return response.data;
  }

  public async post<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.axiosInstance.post<T>(url, data, config);
    return response.data;
  }

  // Additional methods for PUT, DELETE, etc.
}

export default ApiClient.getInstance();
```

Repositories then use this API client to implement their data operations. Here's an example of a user repository:

```typescript
// user.repository.ts
import apiClient from './api-client.service';

interface IUser {
  id: string;
  name: string;
  email: string;
}

interface ICreateUserDTO {
  name: string;
  email: string;
  password: string;
}

class UserRepository {
  private readonly BASE_URL = '/users';

  async getUsers(): Promise<IUser[]> {
    return apiClient.get<IUser[]>(this.BASE_URL);
  }

  async getUserById(id: string): Promise<IUser> {
    return apiClient.get<IUser>(`${this.BASE_URL}/${id}`);
  }

  async createUser(userData: ICreateUserDTO): Promise<IUser> {
    return apiClient.post<IUser>(this.BASE_URL, userData);
  }

  async updateUser(id: string, userData: Partial<IUser>): Promise<IUser> {
    return apiClient.put<IUser>(`${this.BASE_URL}/${id}`, userData);
  }

  async deleteUser(id: string): Promise<void> {
    return apiClient.delete(`${this.BASE_URL}/${id}`);
  }
}

export default new UserRepository();
```

The API client provides several benefits:

1. **Centralized Configuration**: All HTTP-related configuration is managed in one place, including base URLs, timeouts, and default headers.

2. **Automatic Authentication**: The request interceptor automatically adds authentication tokens to requests, keeping the authentication logic separate from business logic.

3. **Error Handling**: Response interceptors can handle common error cases, such as unauthorized access or network errors, in a consistent way across the application.

4. **Type Safety**: The client is fully typed, providing compile-time type checking for API responses.

5. **Request/Response Transformation**: Interceptors can transform requests and responses, allowing for consistent data formatting across the application.

Repositories, in turn, provide these advantages:

1. **Abstraction**: They hide the details of how data is fetched or stored, allowing the rest of the application to work with clean interfaces.

2. **Type Safety**: They work with our type-defined entities, ensuring type safety throughout the data layer.

3. **Reusability**: Common data operations can be reused across different parts of the application.

4. **Testability**: The repository pattern makes it easy to mock data operations in tests.

5. **Maintainability**: Changes to the data source or API structure can be handled in one place.

This architecture creates a clean separation between our data layer and business logic, making the application more maintainable and testable.

## API-Client

The API-Client is a simple object that handles all HTTP communications with the server. It's implemented in a single file (`api-client.ts`) that all repositories use to make their requests. This centralization allows us to maintain consistent configuration and handle authentication across all requests.

Here's a simple implementation:

```ts
// api-client.ts
import authService from '@/services/auth.service';
import axios from 'axios';

// Determine baseURL based on the environment
const baseURL = import.meta.env.DEV 
  ? 'http://localhost:3000' 
  : 'https://api.example.com';

const client = axios.create({
  baseURL: baseURL,
});

// Add a request interceptor to include the token in headers
client.interceptors.request.use(
  (config) => {
    const credentials = authService.getLoginCredentials();
    if (credentials && credentials.token) {
      config.headers.Authorization = `Bearer ${credentials.token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  },
);

export default client;
```

## Queries and Mutations

Following the Command Query Responsibility Segregation (CQRS) pattern, we separate our data operations into two distinct categories: queries (read operations) and mutations (write operations). This separation is implemented using TanStack Query (formerly React Query), which provides powerful caching and state management capabilities.

### Query Layer

The query layer is responsible for managing data fetching, caching, and synchronization. It sits between our repositories and the UI components, providing a clean interface for data access while handling complex concerns like caching, background updates, and error states.

Here's an example of how we structure our queries:

```typescript
// tasks.queries.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import tasksRepository from '@/repositories/tasks.repository';

// Query for fetching tasks
export function useSearchTasksQuery() {
  return useQuery({
    queryKey: ['tasks'],
    queryFn: () => tasksRepository.search(),
    // Optional configuration
    staleTime: 5 * 60 * 1000, // Data is considered fresh for 5 minutes
    cacheTime: 30 * 60 * 1000, // Keep unused data in cache for 30 minutes
  });
}

// Query for fetching a single task
export function useTaskQuery(taskId: string) {
  return useQuery({
    queryKey: ['tasks', taskId],
    queryFn: () => tasksRepository.getById(taskId),
    enabled: !!taskId, // Only run query if taskId is provided
  });
}

// Mutation for adding a new task
export function useAddTaskMutation() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (newTask: ICreateTaskDTO) => tasksRepository.add(newTask),
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
    },
    onError: (error) => {
      // Handle error (e.g., show toast notification)
      console.error('Failed to add task:', error);
    },
  });
}

// Mutation for updating a task
export function useUpdateTaskMutation() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<ITask> }) => 
      tasksRepository.update(id, data),
    onSuccess: (updatedTask) => {
      // Update cache optimistically
      queryClient.setQueryData(['tasks', updatedTask.id], updatedTask);
      // Invalidate list query to ensure consistency
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
    },
  });
}
```

### Caching and Invalidation

TanStack Query provides a sophisticated caching system that helps optimize performance and user experience:

1. **Automatic Caching**: Queries are automatically cached based on their queryKey. The cache is used to:
   - Show data immediately while fetching fresh data in the background
   - Avoid unnecessary network requests
   - Provide offline support for previously fetched data

2. **Stale-While-Revalidate**: The library implements a stale-while-revalidate pattern:
   - Shows cached data immediately if available
   - Fetches fresh data in the background
   - Updates the UI when new data arrives
   - This provides a snappy user experience while ensuring data freshness

3. **Automatic Background Refetching**: The cache can be configured to:
   - Refetch data when the window is refocused
   - Refetch when the network is reconnected
   - Refetch at regular intervals
   - Refetch when the query is remounted

4. **Cache Invalidation**: Mutations can automatically invalidate related queries:
   - After a successful mutation, related queries are marked as stale
   - Stale queries are automatically refetched in the background
   - This ensures data consistency across the application

### Error and Loading States

The query layer also handles loading and error states, making it easy to build robust UIs:

```typescript
function TaskList() {
  const { data: tasks, isLoading, error } = useSearchTasksQuery();
  const addTaskMutation = useAddTaskMutation();

  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <div>
      {tasks.map(task => (
        <TaskItem key={task.id} task={task} />
      ))}
      <AddTaskButton 
        onClick={() => addTaskMutation.mutate(newTask)}
        disabled={addTaskMutation.isPending}
      />
    </div>
  );
}
```

### Benefits of This Approach

1. **Separation of Concerns**: Clear separation between read (queries) and write (mutations) operations
2. **Automatic Cache Management**: No need to manually manage cache invalidation
3. **Optimistic Updates**: Support for optimistic UI updates during mutations
4. **Automatic Background Refetching**: Keeps data fresh without manual intervention
5. **Type Safety**: Full TypeScript support for queries and mutations
6. **DevTools**: Built-in development tools for debugging and monitoring
7. **Offline Support**: Automatic handling of offline scenarios
8. **Pagination and Infinite Scroll**: Built-in support for complex data fetching patterns

This query layer completes our architecture by providing a robust data synchronization layer between our repositories and UI components, while handling complex concerns like caching and background updates.

## Pages / Components

Its function is to display data coming from the controller to the users and to capture events to send them to the controller. They also contain all the presentation logic and are in charge of organising the view-components.

- When a component is rendered on some navigation event is called a page.

## Controllers

They are responsible for maintaining the internal state of each page or component. They expose an api to the pages from which they can react to events and provide it with values computed from the state.

- Each controller will have a unique state using usePureState
- Each controller defines event handlers and exposes them to the view.
- The controller derives properties from the state and exposes them to the view.

```typescript
function useDemoController(props: IUseDemoControllerProps) {
  // Internal state
  const {state, dispatch} = usePureState({
    cardholders: [],
    isLoading: false,
  });
  
  // Helper functions connecting with services and repository
  function handleFetchCardholdersClick() {
    dispatch({isLoading: true});
    repository.listCardholers().then(cardholders => {
      dispatch({cardholders, isLoading: false});
    });
  }

  return {
    // Exposed event handler
    handleFetchCardholdersClick
    // Forwarded state props
    cardholders: state.cardholders,
    isLoading: state.isLoading,
    // Derived properties computed from the state
    numberOfCardholders: state.cardholders.length,
  }
}
```

## View components / UI components

View components, or UI components, are reusable "pieces" of UI that can be used across the entire application. These components live under `src/components` (similar to libraries like shadcn) and are designed to be generic, stateless, and composable. Because they are not tied to any specific feature or page, they can be imported and used from anywhere in the app.

For example:

```tsx
import { Card } from '@/ui/card.component.tsx';
```

Their only function is to render properties and communicate user interactions through events. They should receive information through attributes and communicate information through events. If they have state, it must be an internal and hidden variable.

# Recursive architecture

It is generally recommended to divide the software into small pieces, easy to understand, test, reusable...etc and the frontend is no exception.

## What is Recursive Architecture?

**Recursive architecture** in frontend development means structuring your application as a tree of components, where each component can itself contain smaller components, and this pattern repeats down to the smallest building blocks (leaf nodes). This approach:

- Makes the codebase modular, easy to understand, and maintain.
- Reduces coupling: changes in one part of the tree rarely affect unrelated parts.
- Encourages reusability and testability.

Each component has its own folder, and if it contains subcomponents, those go into a nested `components` folder. Each significant subcomponent (like `header`) should have its own folder under its parent's `components` directory, containing all related files (view, controller, tests, etc). This structure mirrors the component tree in your UI and keeps all related logic together.

### Pages as Special Components

Files ending with `.page.tsx` (or `.page.jsx`) are a special type of React component that are mapped to a URL fragment (route) in your application. Each page represents a top-level route and acts as the entry point for a section of your UI. Pages follow the same recursive architecture: they can have their own `components` folder, and their subcomponents are organized recursively in the same way as any other component. This ensures consistency and scalability throughout the codebase.

### Visual Tree Representation

```
dashboard.page.tsx
└── components/
    ├── drawer/
    │   ├── drawer.component.tsx
    │   ├── drawer.controller.ts
    │   └── components/
    │       ├── header/
    │       │   ├── header.view.tsx
    │       │   ├── header.controller.ts
    │       │   └── header.spect.tsx
    │       └── links/
    │           ├── links.component.tsx
    │           └── links.controller.ts
    ├── nav/
    │   ├── nav.component.tsx
    │   └── nav.controller.ts
    └── content/
        ├── content.component.tsx
        ├── content.controller.ts
        └── components/
            ├── filters/
            │   ├── filters.component.tsx
            │   └── filters.controller.ts
            └── list/
                ├── list.component.tsx
                └── list.controller.ts
```

#### Key Points

- **Recursion:** Each component can have its own `components` folder, repeating the pattern as deep as needed.
- **Isolation:** Changes in a folder never affect sibling folders. All logic, views, and tests for a component are grouped together.
- **No Parent Imports:** To enforce this isolation, parent imports (e.g., `../../`) are strictly banned. Components should only import from their own folder or their own `components` subfolders.
- **Predictability:** The folder structure mirrors the UI/component tree, making it easy to navigate and reason about.

This architecture ensures that changes in one folder cannot impact siblings, making the codebase safer and more maintainable as it grows.

Our page components will be at the same time made up of other smaller components that are at the same time made up of components forming a tree structure.

Sharing application state through this tree is one of the fundamental challenges that all frontend developers face.

![](https://slabstatic.com/prod/uploads/fcn5vyb4/posts/images/FnCJQQELu2QeqcVu_ZbjOSEL.svg)

One of the main rules in our architecture is to follow the [one-way-data-flow](https://tkssharma.gitbook.io/react-training/day-01/react-js-3-principles/one-way-data-flow),  keeping the state as far down the tree as possible, moving it up when strictly necessary.

Read more:

- [https://kentcdodds.com/blog/application-state-management-with-react](https://kentcdodds.com/blog/application-state-management-with-react)
- [https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster)
- [https://egghead.io/lessons/react-lifting-and-colocating-react-state](https://egghead.io/lessons/react-lifting-and-colocating-react-state)

## Folder structure

These architectural limitations are reflected in the folder structure. Making it predictable by perfectly reflecting the architecture of our component tree.

Each component will in turn have a "components" folder where all the components that are part of the parent component are contained. This rule is followed recursively until reaching the leaf nodes.

In this way we reduce the coupling of the application. Because we constrain the coupling of a node to its /components folder. We know that any change in a certain folder can only affect its direct relatives, but never its sibling nodes, or nodes that are part of another tree.

## Shared state

To share global state, session/localStorage, singletons, and React contexts will be used depending on the use case.

Each case must be discussed and evaluated independently.

## Shared components

In the event that a component has to be reused in several trees. It will extract to the Components folder. The components of this folder have to follow a very clear policy about their status.

- They must receive information through attributes.
- They must communicate information through events.
- If they have state, it must be an internal and hidden variable.
- They must be generic.

## Shared business logic

Shared business logic is performed by services. These services are pure functions and must be properly tested.

# State management

We use TanStack Query (formerly React Query) as our primary state management solution for server state, and React's built-in hooks for local UI state. This approach provides several benefits:

TanStack Query handles all server state management, providing a robust solution for data fetching and synchronization. It automatically manages caching, background updates, loading and error states, optimistic updates, and automatic retries. This ensures that our server state remains consistent across components while providing a great user experience with immediate feedback and smooth updates.

For local UI state, we follow the principle of state colocation. This means keeping state as close as possible to where it's used, only lifting it up when absolutely necessary. We leverage React's built-in hooks like useState and useReducer for managing local component state, ensuring that our components remain focused and maintainable.

## State Colocation Principles

Our state management strategy is built on several key principles. First, we keep state local to the component that uses it, unless there's a clear need to share it. This reduces complexity and makes components more predictable and easier to test. When state needs to be shared between components, we lift it to the lowest common ancestor that needs access to it, rather than immediately reaching for global state solutions.

Global state should be used sparingly and only for truly global concerns. This includes authentication state, theme preferences, user settings, and application-wide notifications. For all other cases, we prefer local state or TanStack Query's built-in state management capabilities. TanStack Query provides a comprehensive solution for server state, handling automatic caching, background updates, loading and error states, optimistic updates, and automatic retries out of the box.

## Further Reading

For a deeper understanding of these concepts, we recommend exploring the following resources. Kent C. Dodds provides excellent insights into application state management with React and how state colocation can improve your application's performance. The TanStack Query documentation offers comprehensive guidance on server state management, while TkDodo's blog post comparing React Query to Redux provides valuable context for choosing the right state management solution.

- [Application State Management with React](https://kentcdodds.com/blog/application-state-management-with-react)
- [State Colocation Will Make Your React App Faster](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster)
- [Lifting and Colocating React State](https://egghead.io/lessons/react-lifting-and-colocating-react-state)
- [TanStack Query Documentation](https://tanstack.com/query/latest)
- [React Query vs Redux](https://tkdodo.eu/blog/react-query-vs-redux)

# Routing

We use [react-router](https://reactrouter.com/) to manage the routes.

A router is nothing more than a special component that will conditionally render a child component of the tree to the URL.

- All the routes are listed as constants in the right folder.

# Forms

For form management we use [react-hook-form](https://react-hook-form.com/).

# Patterns

## Container components

Container components are a pattern that separates data fetching and state management from presentation logic. This pattern follows the Single Responsibility Principle by having one component handle data fetching and loading states, while delegating the actual rendering to a presentational component.

The container component is responsible for fetching data using TanStack Query, handling loading and error states, managing any necessary data transformations, and passing the prepared data to the presentational component. This separation of concerns allows the container to focus on data management while keeping the presentation logic clean and focused.

The presentational component, on the other hand, focuses solely on rendering the UI based on the props it receives. This separation makes our components more maintainable, testable, and reusable. By keeping the presentation logic pure and independent of data fetching concerns, we can more easily test and modify our UI components.

Here's a typical example of this pattern:

```typescript
// Container component
function UserProfileContainer() {
  const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => userRepository.getById(userId)
  });

  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;

  return <UserProfile user={user} />;
}

// Presentational component
function UserProfile({ user }) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

This pattern is particularly valuable in several scenarios. When dealing with complex data fetching logic, it helps to isolate and manage that complexity in a dedicated container component. The same applies when the same data needs to be used by multiple components, as the container can handle the data fetching once and distribute it to different presentational components. This approach also ensures consistent handling of loading and error states across the application, while making our presentational components pure and easily testable.

By following this pattern, we achieve several important benefits. Our presentational components remain focused on UI concerns, making them easier to understand and modify. The data fetching logic can be reused across different parts of the application, reducing code duplication. Loading and error states are handled consistently, providing a better user experience. Most importantly, this separation makes our components easier to test and maintain, as each component has a single, well-defined responsibility.
