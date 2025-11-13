# Form Handling

Forms use `react-hook-form` with `arktype` validation. Controller pattern ensures all form logic is centralized.

## Pattern

- **Controller** (`*.controller.ts`): Form logic, validation schema, and submission
- **View** (`*.view.tsx`): Renders UI using `form.register` and `form.formState.errors`

## Example

```typescript
// login-page.controller.ts
import { arktypeResolver } from "@hookform/resolvers/arktype";
import { useForm } from "react-hook-form";
import { type } from "arktype";

const LoginForm = type({
  email: "string.email",
  password: "string",
});

type LoginFormData = typeof LoginForm.infer;

function useLoginPageController() {
  const form = useForm<LoginFormData>({
    resolver: arktypeResolver(LoginForm),
    defaultValues: {
      email: "",
      password: "",
    },
  });

  function onSubmit(data: LoginFormData) {
    // Handle submission
  }

  return {
    form,
    onSubmit: form.handleSubmit(onSubmit),
  };
}

export default useLoginPageController;
```

```tsx
// login-page.view.tsx
import { Box, Button, Input, Text, VStack } from "@chakra-ui/react";
import useLoginPageController from "./login-page.controller";

function LoginPage() {
  const { form, onSubmit } = useLoginPageController();

  return (
    <Box>
      <VStack as="form" onSubmit={onSubmit}>
        <Box width="full">
          <Input {...form.register("email")} />
          {form.formState.errors.email && (
            <Text color="red.500" fontSize="sm" mt={1}>
              {form.formState.errors.email.message}
            </Text>
          )}
        </Box>
        <Box width="full">
          <Input type="password" {...form.register("password")} />
          {form.formState.errors.password && (
            <Text color="red.500" fontSize="sm" mt={1}>
              {form.formState.errors.password.message}
            </Text>
          )}
        </Box>
        <Button type="submit">Submit</Button>
      </VStack>
    </Box>
  );
}

export default LoginPage;
```

## Rules

1. **No destructuring**: Use `form.register` and `form.formState.errors` directly
2. **Controller logic**: All form logic goes in the controller, view only renders
3. **Type inference**: Use `typeof Schema.infer` for form types

## Related Documentation

- [React Components Guide](./react.md) - Controller pattern and component structure
- [Architecture Guide](./architecture.md) - Separation of concerns
