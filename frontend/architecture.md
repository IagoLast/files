# Recursive Architecture

Recursive architectures provide a structured and repeatable pattern for organizing frontend applications. Rooted in the principles of Clean Architecture, they emphasize **simplicity**, **low coupling**, **high cohesion**, **testability**, and **developer onboarding**. This system enforces a strict separation of concerns, layered dependencies, and a predictable folder structure that recursively applies the same conventions at every level of the application.

## 📐 Core Principles

- **Simplicity above all**: Every part of the application adheres to a small set of rules that repeat recursively.
- **Dependency Rule**: Dependencies must always flow **inward**, from the outer layers (UI, frameworks) toward the core (domain logic).
- **Recursion**: Each page or component recursively contains its own subcomponents, controller, tests, and internal logic using the same structure.

---

## 🧱 Architectural Layers

### 1. **Core Layer**

This layer contains business logic and is framework-agnostic.

#### Domain Entities

- Represent business concepts as strongly-typed plain objects or interfaces.
- Must not depend on any external code (UI, framework, API).
- Example:

```typescript
interface Booking {
  id: string;
  hotelId: string;
  customerId: string;
  start: Date;
  end: Date;
  status: "pending" | "confirmed" | "cancelled";
}
```

#### Constants

- Represent meaningful business values (e.g., minimum password length).
- Follow naming convention: UPPER_SNAKE_CASE.

```typescript
export const MIN_REQUIRED_PASSWORD_LENGTH = 8;
```

#### Services

- Contain pure functions only.
- Depend only on entities and constants.
- Recommended: Export a single default object with all public service functions.

```typescript
// password.service.ts
function checkPasswordLength(password: string): boolean {
  return password.length >= MIN_REQUIRED_PASSWORD_LENGTH;
}

export default { checkPasswordLength };
```

**Benefits:**
- Easy to test and mock.
- Enforces namespace clarity in imports (passwordService.checkLength() vs ambiguous checkLength()).

**Note:** Some services may need to interact with browser APIs (like `localStorage` for authentication). These should still be organized as services, but may have side effects. Example:

```typescript
// auth.service.ts
const TOKEN_STORAGE_KEY = 'auth_token';

function getToken(): string | null {
  return localStorage.getItem(TOKEN_STORAGE_KEY);
}

function setToken(token: string): void {
  localStorage.setItem(TOKEN_STORAGE_KEY, token);
}

function clearToken(): void {
  localStorage.removeItem(TOKEN_STORAGE_KEY);
}

function isAuthenticated(): boolean {
  return getToken() !== null;
}

export default { getToken, setToken, clearToken, isAuthenticated };
```

---

### 2. **Application Layer**

Implementation details such as networking, data access, and UI logic.

#### Client

- Singleton object for interacting with the backend.
- Handles HTTP configuration, headers, and interceptors.
- **Authentication logic should be in `auth.service.ts`, not in the client.**

```typescript
// client/index.ts
import axios from 'axios';
import authService from '../services/auth.service';

export const client = axios.create({ baseURL: BASE_URL });

function setAuthHeader(token: string): void {
  client.defaults.headers.common["Authorization"] = `Bearer ${token}`;
}

function initializeAuth(): void {
  const token = authService.getToken();
  if (token) {
    setAuthHeader(token);
  }
}

initializeAuth();

export function setAuthToken(token: string): void {
  authService.setToken(token);
  setAuthHeader(token);
}

export function clearAuthToken(): void {
  authService.clearToken();
  delete client.defaults.headers.common["Authorization"];
}
```

#### Repositories

- Abstract CRUD operations on external data sources.
- Transform raw server data into internal business entities.

```typescript
async function list(): Promise<Booking[]> {
  const response = await client.get("/bookings");
  return response.data.map(raw => ({
    id: raw.id,
    start: new Date(raw.init),
    end: new Date(raw.finish_date),
    // ...
  }));
}
```

#### Queries & Mutations

- Follow Command Query Separation:
  - **Queries**: Return values, no side effects.
  - **Commands**: Cause side effects, return void or success indicators.
- Use tanstack-query to manage async state, loading, success, error.

```typescript
export function useListBookings() {
  return useQuery({
    queryKey: ["bookings", "list"],
    queryFn: bookingsRepository.list,
  });
}

export function useCreateBookingMutation() {
  return useMutation({
    mutationFn: bookingsRepository.create,
    onSuccess: () => {
      queryClient.invalidateQueries(["bookings", "list"]);
    },
  });
}
```

#### Controllers

- Bridge between UI and application logic.
- Use hooks to expose state and handler functions to views.

```typescript
export function useBookingController() {
  const { id } = useParams();
  const query = useGetBookingQuery({ id });
  const { cancelBooking } = useCancelBookingMutation();

  function handleCancelClick() {
    return cancelBooking({ id });
  }

  return { query, handleCancelClick };
}
```

---

### 3. **Presentation Layer**

The outermost layer responsible for rendering the UI.

#### Views

- Stateless UI components.
- Get all data and handlers via props or controllers.
- No business logic inside views.

```typescript
function BookingView() {
  const { query, handleCancelClick } = useBookingController();

  return (
    <div>
      <h1>Booking {query.booking.id}</h1>
      <span>{query.booking.start}</span>
      <span>{query.booking.end}</span>
      <span>{query.booking.status}</span>
      <button onClick={handleCancelClick}>Cancel</button>
    </div>
  );
}
```

#### Stateful vs Pure Components

**Pure Components:**
- Output depends solely on props.
- Easier to test and reuse.

```typescript
function Greet({ name }: { name: string }) {
  return <div>Hello {name}</div>;
}
```

**Stateful Components:**
- Have internal state (e.g. using useState).
- Should keep state as close as possible to where it's used.

To transform a stateful component into pure:
1. Extract state into a container component.
2. Pass props to the stateless presentational component.

---

## 📁 Folder Structure

Recommended root structure:

```
/src
├── client            # Axios or Apollo singleton
├── components        # Global reusable components
├── constants         # Business constants
├── pages             # Route-based entry points
├── queries           # Async hooks
├── repositories      # Data access logic
├── services          # Pure business logic
├── types             # TS interfaces & types
```

---

## 🔁 Recursive Component Structure

Every page or component has a dedicated folder containing:

```
booking-list/
├── booking-list.view.tsx
├── booking-list.controller.ts
├── booking-list.spec.tsx
├── components/
│   └── booking-item/
│       ├── booking-item.view.tsx
│       ├── booking-item.controller.ts
│       └── components/
│           ├── booker-data/
│           ├── booking-dates/
│           └── booking-actions/
```

**Key Rule:**

A component can only import:
- Its own subcomponents (in `/components`)
- Global components

Relative imports to parent or sibling folders (`../..`) are strictly forbidden.

This guarantees:
- Clear dependency direction
- Predictable impact scope (i.e. changes in children don't affect siblings)

---

## 🌐 Routing & Pages

- Pages are components mapped to routes (like in Next.js or React Router).
- Each route corresponds to a folder under `/pages`.
- Supports nested routes recursively.

```typescript
const router = createBrowserRouter([
  {
    path: "/",
    element: <RootPage />,
    children: [
      { path: "events/:id", element: <EventPage /> },
      { path: "profile", element: <ProfilePage /> },
    ],
  },
]);
```

## Architecture key 

- Keep business logic pure and central in the core layer.
- Use controllers to manage data and async flows.
- Write views as stateless as possible.
- Avoid cross-dependencies between components.
- Use recursive folder structure for consistency and clarity.
- Prefer testable and composable components.
- Maintain clear data flow and dependency direction.




```
