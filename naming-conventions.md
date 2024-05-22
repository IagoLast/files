The intention of this document is to standardise JavaScript and CSS naming conventions and syntaxes across APTO frontend repositories. Don’t shoot the messenger. Many subheadings also includes a “Why?” section to explain our rationale.

# JavaScript

In general, we follow conventions set by `eslint` and include the aforementioned linter in our repositories.

## Variables

### Use self-descriptive names

Variable names should be self-descriptive nouns – reflecting the value of the variable and negating the need for comments describing the value.

```typescript
// don't do this
const value = 2020; // current year

// do this instead
const currentYear = 2020;
```

If a variable’s value will not change, define it with `const` to make it immutable. Otherwise, unless necessary, use `let` in order to declare variables in the smallest scope possible. There are [very few cases](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Redeclarations) where you will need to use `var`.

### Avoid abbreviations

Try to keep variable names short, but don’t sacrifice descriptiveness in fear of verbosity. With this in mind, avoid abbreviating variable names.

```typescript
// don't do this
const bioDesc = "I hate puppies";

// do this instead
const biographyDescription = "I love puppies";
```

### Use camelCase

Variable names should be written in [camelCase](https://en.wikipedia.org/wiki/Camel_case).

```typescript
// don't do this
const first_name = "Luke";

// do this instead
const firstName = "Lucrecio";
```

The same goes for object properties and methods.

```typescript
// do this
const newUser = {
  firstName: "Albert",
  lastName: "Camus",
};
```

On occasion, we’ll need to handle snake_case properties in API responses, but they should be converted to camelCase before leaving their associated `service`.

**Why?**

In general, the goals with a variable declaration are to keep code clear, concise and easy to read. Historically, JavaScript developers sought short variable names because screens were narrow and minification was not a widespread practice. Neither of these concerns exists today and, instead, we want every developer to have as much information as possible with as little effort as required – allowing for quick onboarding, readable interfaces and efficient debugging.

## Domain Constants

The primary exception to the above camelCase rule is when defining constants for use throughout a component, page or the whole application. These should be defined in all caps with underscores used to separate words, what is known as [Screaming Snake Case](https://dev.to/fission/screaming-snake-case-43kj) or, as I call it, My Uncle On Facebook Case.

```typescript
// do this
const ACCOUNT_ACTIVATION_ENDPOINT = "/acccount-activation";
```

**Why?**

Just as with variables more generally, the intent with using screaming snake case is to make constant definitions as clear as possible to other developers as early as possible so they can get to solving problems and closing tickets as fast as possible.

## Booleans

In most cases – all, unless it forces you to negate the principle of self-descriptiveness – boolean definitions should begin with `is`, `has`, `can` or `are` in order to signal to other developers that the value is boolean.

```typescript
// don't do this
const redBackground = true;

// do this instead
const isRedBackground = true;
```

Also, we should avoid passing booleans as function parameters without context.

```typescript
// don't do this
function getColor(isRed: boolean) {
  return isRed ? "#913122" : "#2b6b46";
}
const color = getColor(true);

// do this instead
function getColor(color: "red" | "green") {
  return color === "red" ? "#913122" : "#2b6b46";
}
const color = getColor("red");

// or this
function getColor({ isRed }: { isRed: boolean }) {
  return isRed ? "#913122" : "#2b6b46";
}
const color = getColor({ isRed: true });
```

**Why?**

Our goal is to make it clear as quickly as possible that a boolean variable is, in fact, a boolean and remove the need for our coworkers to go hunting for value declarations.

## Functions

Like variables, function names should be self-descriptive. Unlike variables, they should be verbs instead of nouns. This means they should describe the _action the function will undertake_.

```typescript
// don't do this
function hungover(tipo: Human, drunk: number) {
  while (tipo.cuerpito < drunk) {
    drinkBeer(tipo);
  }
  return sleepUntilMorning(tipo);
}

// do this instead
function getHungover(tipo: Human, drunk: number) {
  ...
}

```

### Use function declarations instead of function expressions

```typescript
// don't do this
const removeTrailingCommas = (name: string) => name.replace(/,\s*$/, "");

// do this instead
function removeTrailingCommas(name: string) {
  return name.replace(/,\s*$/, "");
}
```

### Avoid exporting anonymous functions as components

```typescript
// don't do this
export default function (name: string) {
  return `Hello ${name}!`;
}

// don't do this either... even though it looks cool
export default (name: string) => `Hello ${name}!`;

// do this instead
export default function getGreeting(name: string) {
  return `Hello ${name}!`;
}
```

Not only will this make your functions easier to read, but it will also help with debugging errors in your stack traces.

A notable exception is lambda functions, where anonymous functions are excellently suited for callbacks.

```typescript
// do this
interface Student {
  name: string;
  age: number;
}

const studentAges = students.map(({ age }: Student): Array<number> => age);
```

**Why?**

As with our other rules, the main goal here is clarity. However, another benefit to using function declarations is [hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting), which allows you to use your function before you declare it.

### Use an object if your function takes more than 2 parameters

```typescript
// don't do this
function greet(name, surname, age, nationality) {
  return `Hello ${name} ${surname}, you're ${age} years old and from ${nationality}`;
}

// do this
function greet(args) {
  const { name, surname, age, nationality } = args;
  return `Hello ${name} ${surname}, you're ${age} years old and from ${nationality}`;
}
```

We encourage you to name the object args for clarity and destructure it on a single to make the function easier to read.

**Why?**

When calling a function with multiple parameters, it’s helpful to know what the parameters are about…

```typescript
greet("Lisa", "Simpson", 12, "Springfield");
```

For instance, in this case, is not obvious what 12 refers to. It’s a number but it refers to an age, a day of the month, an amount? Using an object, we emulate named parameters, hence every option the function operates upon on gets “labelled”:

```typescript
greet({ name: "Lisa", surname: "Simpson", age: 12 });
```

It allows us to understand better the function without having to look at its implementation details.

We can as well change the parameter order without side effects or skip some of them without the function invocation looking weird.

## Private fields/functions

JavaScript provides no native way to declare a private variable or method – though there are multiple [old school](https://www.crockford.com/javascript/private.html) and [new school](https://www.sitepoint.com/javascript-private-class-fields/) approaches to solving that problem. At APTO, we define private fields and functions as any function that will not be exported by a component or module’s interface and we use the common convention of prefacing their name with an underscore.

```typescript
// do this
function _getIsHidden() {
  return state.errorMessage === '';
}

export {
  isHidden: _getIsHidden(),
}

```

**Why?**

Again, the goal here is to achieve clarity and efficiency. Prefacing private fields with an underscore signals to other developers that any changes made that field/function will only affect a reduced scope (eg. the current module/class), which makes it easier to debug, refactor and avoid side effects.

## Services

As opposed to classes, services are files that export an object containing non-instantiable, usually pure functions. Services can be thought of as collections of utilities grouped around completing a single, often generic task. We aim to keep service functions [pure and free of](https://medium.com/@jamesjefferyuk/javascript-what-are-pure-functions-4d4d5392d49c) side effects.

```typescript
// do this
function getHasLowercaseChar(password: string) {
  return /[a-z]/.test(password);
}

function getHasMinimumLength(password: string) {
  return password.length >= MIN_REQUIRED_LENGTH;
}

function getHasNumber(password: string) {
  return /\d/.test(password);
}

function getIsValidPassword(password: string) {
  const hasLowercaseChar = getHasLowercaseChar(password);
  const hasMinimunLength = getHasMinimumLength(password);
  const hasNumber = getHasNumber(password);
  return hasLowercaseChar && hasMinimunLength && hasNumber;
}

export default {
  getHasLowercaseChar,
  getHasMinimumLength,
  getHasNumber,
  getIsValidPassword,
};
```

The naming convention for a service follows the pattern `***.service.ts`, where `***` is a self-descriptive name describing the purpose of the service. In the above example, the name might be `passwordValidation.service.ts`.

Also, services should have a default export containing every exported function.

**Why?**

Separating out service logic from components server the dual purpose of extracting logical, stateless concerns from the UI and possible side effects, while also making it easy to set up a [test double](https://martinfowler.com/bliki/TestDouble.html) on a default export.

# Typescript

## Types &amp; Interfaces

### Use self-descriptive names prefaced by “I…”

```typescript
// don't do this
interface Money {
  currency: Currency;
  amount: number;
}

// do this
interface IMoney {
  currency: ICurrency;
  amount: number;
}
```

### Why?

Being descriptive when writing a type alias allows us to understand how that type is meant to be used.

In another hand, prefacing it with I... allow us to identify quickly the variable as a type alias and prevents conflicts with class names or global variables.

- Use nouns
- Function arguments should be named `I<function-name>Args`
- Function return values should be named `I<functionName>ReturnValue`

# CSS

In general, we follow the [Block Element Modifier (BEM)](http://getbem.com/) methodology when defining CSS or SASS styles.

## Block

Blocks encapsulate standalone entities in your HTML. When working with UI frameworks, this often means a component’s outermost element, but blocks can be nested in the markup too.

Names should be self-descriptive and consist of letters, digits and dashes.

Avoid tag names or ids and, instead, use class name selectors. Blocks should not depend on other blocks or elements on a page.

```
// do this
.block {
  color: (--white);
}

```

## Element

Elements are semantically tied to the parent blocks. They have no standalone meaning.

Element names should extend the block naming conventions. Their name should include the block name, followed by two underscores and then the element name.

```
// do this
.block__elem {
  color: var(--white);
}

```

## Modifier

Modifiers are flags on blocks or elements. They represent changes in behavior and appearance. Their name should match that of the block or element they modify followed by two dashes and then a description of the intended modification.

```
// do this
.block__elem--small {
  padding: var(--spacing--s);
}

```

## Selectors

Unless necessary, avoid targeting elements by their selector directly in your CSS. Instead, add BEM-style classes to the element and target them instead.

For example, given the component:

```
<nav class="NavBar">
  <a class="NavBar__link" href=#">Link</a>
</nav>
// don't do this
.NavBar {
  a {
    text-decoration: none;
  }
}

// do this instead
.NavBar {
  &__link {
    text-decoration: none;
  }
}

```

## State

Changes in element state should be represented by a state class in the format .{verb}-{state}, following the structure defined by [the SMACCS architecture](http://smacss.com/).

```
// do this
.BulletPoint {
  color: black;
  &.is-active {
    color: red;
  }
}

```

## Custom properties

Commonly known as &quot;CSS variables&quot;.

### Defining a custom property

Custom properties _must_ be within a selector and start with two dashes

### Using a custom property

# Filenames

While variables are named using camelCase, files should be named using a [snake case](https://en.wikipedia.org/wiki/Snake_case) convention. This is because sometimes git can play some tricks on you on case-insensitive filesystems. snake_case totally avoids this problem:

- [https://stackoverflow.com/questions/6899582/i-change-the-capitalization-of-a-directory-and-git-doesnt-seem-to-pick-up-on-it/6899682](https://stackoverflow.com/questions/6899582/i-change-the-capitalization-of-a-directory-and-git-doesnt-seem-to-pick-up-on-it/6899682)

[https://stackoverflow.com/questions/10523849/changing-capitalization-of-filenames-in-git](https://stackoverflow.com/questions/10523849/changing-capitalization-of-filenames-in-git)

[https://www.reddit.com/r/git/comments/dj5b8n/lpt_check_your_uppercaselowercase_file_names/](https://www.reddit.com/r/git/comments/dj5b8n/lpt_check_your_uppercaselowercase_file_names/)

**Use an ending to indicate the content type of the file:**

- `descriptive-name.view.tsx`
- `descriptive-name.controller.ts`
- `descriptive-name.service.ts`
- `descriptive-name.spec.tsx`

# React

## Classes, components and constructor functions

Any function that will be utilized as a prototype/template to generate objects should be defined with the first letter of the name capitalized. In practice, for us this generally means React components, however, it could also mean JavaScript classes and, in case you are reading this in 2008, constructor functions.

```typescript
import React from "react";

// don't do this
export default function homePage() {
  return <div />;
}

// do this instead
export default function HomePage() {
  return <div />;
}
// do this
class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}
```

## Always declare components as functions

In React, we can declare components either through an ES6 `class` or as functions. With the introduction of hooks, most use cases for class components fell short, and even React recommends [moving away from them](https://reactjs.org/docs/hooks-intro.html#classes-confuse-both-people-and-machines) whenever possible.

```xml
// don't do this
class Button extends React.PureComponet {
  render() {
    return <button>Click!</button>
  }
}

// do this
function Button() {
  return <button>Click!</button>
}

```

## Queries and mutations

We will always name the variable ending in query or mutation as appropriate:

```tsx
const usersQuery;
const createUserMutation;

```

When naming functions, we will follow the following structure: use | action | resource | type:

- use will be used to indicate a hook.
- action will correspond to the action we are performing on the resource (list, get, update, delete…).
- resource will refer to the resource we are working with.
- type will be according to whether it corresponds to a query or a mutation.

For more information on how it works and the advantages of using this library, see [React Query](https://react-query.tanstack.com/).

```tsx
useListUsersQuery();
CreateUserMutation();
```

**Why?**

This makes it easier to identify how the information is being treated and therefore easier to find.

## Project structure

This section is unfinished

**React applications** are generally structured as follows:

- `src/`
  - `api-client/`
    - `apiclient.service.ts`
  - `components/`
    - `Notifications/`
      - `Notifications.tsx`
      - `Notifications.spec.tsx`
      - `useNotifications.hook.ts`
      - `Notifications.module.scss`
  - `hooks/`
  - `pages`/
    - `home`
      - `components/`
        - …
      - `Home.page.tsx`
      - `Home.spec.tsx`
      - `Home.module.scss`
  - `services/`
  - `types/`
  - `utils/`

## Component structure

A component is usually made with different modules (the source code for the component, its styling, its tests, services, stories, etc…). We group all these files within a single folder named after the component:

```
MyComponent/
  MyComponent.tsx
  MyComponent.spec.tsx
  MyComponent.stories.tsx
  useMyComponent.ts

```

Each of the files should be named with the component name so that when searching them is easy to identify to which component they relate. Here are the naming conventions for each of the files:

- View → `{ComponentName}.tsx`
- Business logic → `use{ComponentName}.tsx`
- Styles → `{ComponentName}.module.scss`
- Tests → `{ComponentName}.spec.tsx`
- Storybook stories → `{ComponentName}.stories.tsx`

## Separation of concerns

When writing a new component, keep the view logic within the main component file and move the business logic to a hook:

```javascript
//❗Don't do this
// UserPage.tsx
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
// UserPage.tsx
export default function UserPage() {
  const { companyName, onViewTransactions, userDetails } = useUserPage();
  // ...
  return (
    <View>
     <Button onClick={onViewTransactions}>View transactions</View>
  // ...
}
// useUserPage.ts
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

This allows us to refactor and share code across the project easier. It also sticks to the convention that component file is only in charge to render the view with the parameters injected by a hook which acts as a “controller”.

## Sub-components

Whenever a component has sub-components, place them in a `components` folder relative to the current component folder following a recursive architecture.

```
MyComponent/
  components/
    ComponentFoo/
      ComponentFoo.tsx
    ComponentBar/
      ComponentBar.tsx
      useComponentBar.ts
  MyComponent.tsx
  MyComponent.spec.tsx
  useMyComponent.ts

```
