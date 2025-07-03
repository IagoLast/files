# React & Frontend Components

## Component Principles
- Single responsibility
- Composition over inheritance  
- Prop typing with TypeScript
- Meaningful component names
- Presentation components (UI only)
- Container components (logic + data)
- Custom hooks (reusable logic)

## Classes, components and constructor functions

Any function that will be utilized as a prototype/template to generate objects should be defined with the first letter of the name capitalized. In practice, for us this generally means React components, however, it could also mean JavaScript classes and, in case you are reading this in 2008, constructor functions.

```typescript
import React from 'react';

// don't do this
// home-page.view.tsx
export default function homePage() {
  return (<div />);
}

// do this instead
// home-page.view.tsx
export default function HomePage() {
  return (<div />);
}
// rectangle.ts
class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}

```

## Always declare components as functions

In React, we can declare components either through an ES6 `class` or as functions. With the introduction of hooks, most use cases for class components fell short, and even React recommends [moving away from them](https://reactjs.org/docs/hooks-intro.html#classes-confuse-both-people-and-machines) whenever possible.

```tsx
// don't do this
// button.view.tsx
class Button extends React.PureComponet {
  render() {
    return <button>Click!</button>
  }
}

// do this
// button.view.tsx
function Button() {
  return <button>Click!</button>
}

```

## Queries and mutations

We will always name the variable ending in query or mutation as appropriate:

```tsx
const listUsersQuery = useListUsersQuery();
const createUserMutation = useCreateUserMutation();
```

When naming functions, we will follow the following structure: use | action | resource | type:

- use will be used to indicate a hook.
- action will correspond to the action we are performing on the resource (list, get, update, delete…).
- resource will refer to the resource we are working with.
- type will be according to whether it corresponds to a query or a mutation.

For more information on how it works and the advantages of using this library, see [React Query](https://react-query.tanstack.com/).

```typescript
useListUsersQuery();  
useCreateUserMutation(); 

```

**Why?**

This makes it easier to identify how the information is being treated and therefore easier to find.

## Project structure

This section is unfinished

**React applications** are generally structured as follows:

- `src/`
    - `api-client/`
        - `api-client.service.ts`
    - `components/`
        - `notifications/`
            - `notifications.view.tsx`
            - `notifications.spec.tsx`
            - `notifications.controller.ts`
            - `notifications.module.scss`
    - `hooks/`
    - `pages/`
        - `home/`
            - `components/`
                - …
            - `home.view.tsx`
            - `home.spec.tsx`
            - `home.module.scss`
    - `services/`
    - `types/`
    - `utils/`

## Component structure

A component is usually made with different modules (the source code for the component, its styling, its tests, services, etc…). We group all these files within a single folder named after the component:

```
my-component/
  my-component.view.tsx
  my-component.spec.tsx
  my-component.controller.ts
```

Each of the files should be named with the component name so that when searching them is easy to identify to which component they relate. Here are the naming conventions for each of the files:

- View → `{component-name}.view.tsx` or `{component-name}.component.tsx`
- Business logic → `{component-name}.controller.ts`
- Styles → `{component-name}.module.scss`
- Tests → `{component-name}.spec.tsx`

## Separation of concerns

When writing a new component, keep the view logic within the main component file and move the business logic to a controller:

```tsx
//❗Don't do this
// user-page.view.tsx
export default function UserPage() {
  const session = useSession();
  const userQuery = useUserQuery();
  const navigate = useNavigate();
  // ... 
  return (
    <View>
     <Button onClick={() => navigate(TRANSACTIONS_ROUTE)}>View transactions</View>
  // ...
}

// ✅ Do this
// user-page.view.tsx
export default function UserPage() {
  const { companyName, onViewTransactions, userDetails } = useUserPage();
  // ... 
  return (
    <View>
     <Button onClick={onViewTransactions}>View transactions</View>
  // ...
}
// user-page.controller.ts
export default function useUserPage() {
  const session = useSession();
  const userQuery = useUserQuery();
  const navigate = useNavigate();
  // ... 
  return {
    companyName: session.companyName,
    onViewTransactions: () => navigate(TRANSACTIONS_ROUTE),
    userDetails: userQuery.data?.details
  // ...
}
```

This allows us to refactor and share code across the project easier. It also sticks to the convention that component file is only in charge to render the view with the parameters injected by a controller which acts as a "controller".

## Sub-components

Whenever a component has sub-components, place them in a `components` folder relative to the current component folder following a recursive architecture.

```
my-component/
  components/
    component-foo/
      component-foo.view.tsx
    component-bar/
      component-bar.view.tsx
      component-bar.controller.ts
  my-component.view.tsx
  my-component.spec.tsx
  my-component.controller.ts
```

## Architecture Overview

### Types as Entities

Using TypeScript types as entities in our architecture brings several significant advantages to our codebase. By treating types as pure data structures without behavior, we create a more functional and maintainable codebase. This approach eliminates the need to reconstruct class instances from server data, making data flow simpler and more predictable throughout the application.

### Services

Services are groups of pure functions where the business logic of our application is defined. Generally these functions will be part of our hooks, but sometimes there is some shared logic throughout the application that needs to be extracted into services.

Services should:
- Be collections of pure functions
- Follow naming convention `*.service.ts`
- Have a default export containing all functions
- Operate on type-defined entities
- Be easily testable and mockable

Example:
```typescript
// password-validation.service.ts
interface IPasswordValidationResult {
  isValid: boolean;
  errors: string[];
}

function validatePassword(password: string): IPasswordValidationResult {
  const errors: string[] = [];
  
  if (!getHasLowercaseChar(password)) {
    errors.push('Password must contain at least one lowercase letter');
  }
  
  return {
    isValid: errors.length === 0,
    errors
  };
}

export default {
  validatePassword
};
```

### Repositories

Repositories are specialized services that handle data operations, typically interacting with a server through CRUD operations. They act as an abstraction layer between our application and the data source.
