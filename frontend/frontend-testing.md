# Frontend testing

- Test should be as close as a real use case as possible.
  - Do not test implementation.
- Only server calls are mocked (avoid side effects).
- 

```ts
// Use descrptive test descriptions
test('should display a list of todos when the server returns a valid todo list"', await () => {
  // Emulate server responses
  stubJsonResponse({
    path: 'api/v1/todos',
    method: 'get',
    response: dummy_list_todos_response,
  });
  // Emulate a login
  authService.login(dummy_credentials);
  // Navigate to the path
  router.history.navigate(PATH_DASHBOARD_TODO_LIST);
  // Render the entire app
  render(<App />);
  // Interact with the page if required
  await userEvent.click(/List todos/);
  // Assert like a human would do
  return expect(screen.getByText("This is an example text")).toBeVisible();
});
```