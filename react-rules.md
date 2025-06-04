# React

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