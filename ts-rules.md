# Typescript

## Types & Interfaces

### Use self-descriptive names prefaced by "I..."

```ts
// don't do this
interface Money {
  currency: Currency;
  amount: number;
};

// do this
interface IMoney {
  currency: ICurrency;
  amount: number;
};

```

### Why?

Being descriptive when writing a type alias allows us to understand how that type is meant to be used.

In another hand, prefacing it with I... allow us to identify quickly the variable as a type alias and prevents conflicts with class names or global variables.

- Use nouns
- Function arguments should be named `I<function-name>Args`
- Function return values should be named `I<functionName>ReturnValue` 