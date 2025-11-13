# JavaScript / Typescript

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

If a variable's value will not change, define it with `const` to make it immutable. Otherwise, unless necessary, use `let` in order to declare variables in the smallest scope possible. There are [very few cases](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Redeclarations) where you will need to use `var`.

### Avoid abbreviations

Try to keep variable names short, but don't sacrifice descriptiveness in fear of verbosity. With this in mind, avoid abbreviating variable names.

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

On occasion, we'll need to handle snake_case properties in API responses, but they should be converted to camelCase before leaving their associated `service`.

**Why?**

In general, the goals with a variable declaration are to keep code clear, concise and easy to read. Historically, JavaScript developers sought short variable names because screens were narrow and minification was not a widespread practice. Neither of these concerns exists today and, instead, we want every developer to have as much information as possible with as little effort as required – allowing for quick onboarding, readable interfaces and efficient debugging.

## Objects

### Organize object keys alphabetically

Object properties and export statements should be organized alphabetically to maintain consistency and improve readability.

```typescript
// don't do this
const user = {
  name: "John",
  email: "john@example.com",
  age: 30,
  address: "123 Main St",
};

export { userService, authService, apiClient };

// do this instead
const user = {
  address: "123 Main St",
  age: 30,
  email: "john@example.com",
  name: "John",
};

export { apiClient, authService, userService };
```

**Why?**

Alphabetical organization makes diffs easier to read and understand, as changes are predictable and localized. It also enables automatic formatting tools to maintain consistency across the codebase without manual intervention.

## Domain Constants

The primary exception to the above camelCase rule is when defining constants for use throughout a component, page or the whole application. These should be defined in all caps with underscores used to separate words, what is known as [Screaming Snake Case](https://dev.to/fission/screaming-snake-case-43kj) or, as I call it, My Uncle On Facebook Case.

```ts
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

When calling a function with multiple parameters, it's helpful to know what the parameters are about…

```typescript
greet("Lisa", "Simpson", 12, "Springfield");
```

For instance, in this case, is not obvious what 12 refers to. It's a number but it refers to an age, a day of the month, an amount? Using an object, we emulate named parameters, hence every option the function operates upon on gets "labelled":

```typescript
greet({ name: "Lisa", surname: "Simpson", age: 12 });
```

It allows us to understand better the function without having to look at its implementation details.

We can as well change the parameter order without side effects or skip some of them without the function invocation looking weird.

## Private fields/functions

JavaScript provides no native way to declare a private variable or method – though there are multiple [old school](https://www.crockford.com/javascript/private.html) and [new school](https://www.sitepoint.com/javascript-private-class-fields/) approaches to solving that problem. We define private fields and functions as any function that will not be exported by a component or module's interface and we use the common convention of prefacing their name with an underscore.

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

## Types & Interfaces

### Use self-descriptive names prefaced by "I..."

```ts
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

On the other hand, prefacing it with I... allows us to identify quickly the variable as a type alias and prevents conflicts with class names or global variables.

- Use nouns
- Function arguments should be named `I<function-name>Args`
- Function return values should be named `I<functionName>ReturnValue`
