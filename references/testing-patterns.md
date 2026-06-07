# Testing Patterns Reference

Quick reference for common testing patterns. Use alongside `testing` and `test-driven-development` skills.

## Test Structure (Arrange-Act-Assert)

```typescript
it('describes expected behavior', () => {
  const input = { title: 'Test Task', priority: 'high' };
  const result = createTask(input);
  expect(result.title).toBe('Test Task');
  expect(result.status).toBe('pending');
});
```

## Test Naming

```typescript
describe('TaskService.createTask', () => {
  it('creates a task with default pending status', () => {});
  it('throws ValidationError when title is empty', () => {});
  it('trims whitespace from title', () => {});
});
```

## Common Assertions

```typescript
expect(result).toBe(value);                 // strict equality
expect(result).toEqual(obj);                // deep equality
expect(result).toBeNull();
expect(result).toBeDefined();
expect(result).toBeGreaterThan(5);
expect(result).toMatch(/pattern/);
expect(array).toContain(item);
expect(() => fn()).toThrow(ValidationError);
await expect(asyncFn()).resolves.toBe(value);
await expect(asyncFn()).rejects.toThrow(Error);
```

## Mocking Patterns

```typescript
const mockFn = jest.fn();
mockFn.mockReturnValue(42);
mockFn.mockResolvedValue({ data: 'test' });
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
```

Mock at boundaries only: database calls, HTTP requests, file system, external APIs. Don't mock: internal utilities, business logic, pure functions.

## React Testing

```typescript
import { render, screen, fireEvent } from '@testing-library/react';

it('submits form with entered data', async () => {
  const onSubmit = jest.fn();
  render(<TaskForm onSubmit={onSubmit} />);
  fireEvent.change(screen.getByRole('textbox', { name: /title/i }), {
    target: { value: 'New Task' },
  });
  fireEvent.click(screen.getByRole('button', { name: /create/i }));
  expect(onSubmit).toHaveBeenCalledWith({ title: 'New Task' });
});
```

## API Integration Testing

```typescript
import request from 'supertest';
const response = await request(app)
  .post('/api/tasks')
  .send({ title: 'Test' })
  .set('Authorization', `Bearer ${token}`)
  .expect(201);

expect(response.body).toMatchObject({
  id: expect.any(String),
  title: 'Test',
});
```

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| Testing implementation details | Breaks on refactor | Test inputs/outputs |
| Shared mutable state | Tests pollute each other | Setup/teardown per test |
| Snapshot everything | No one reviews diffs | Assert specific values |
| Skipping tests to pass CI | Hides real bugs | Fix or delete the test |
| No async error handling | Swallowed failures | Always await async tests |
