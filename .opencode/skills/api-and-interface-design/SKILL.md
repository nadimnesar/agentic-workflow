---
name: api-and-interface-design
description: Design stable interfaces with clear contracts before implementing internals. Define inputs, outputs, errors, and invariants first — then build the implementation against them.
license: MIT
compatibility: opencode
metadata:
  phase: build
  agent: build
---

# API and Interface Design

## What I Do

I force interface design to happen before implementation. The contract is written first, agreed upon, and then the
internals are built to satisfy it. This prevents the most expensive kind of rework: refactoring a correct implementation
because the interface was wrong.

## When to Use Me

Use me when creating or changing:

- Public functions or modules (called from more than one place)
- HTTP/REST/GraphQL API endpoints
- Component props interfaces (in UI work)
- Event schemas or message formats
- Service boundaries between modules
- Any contract that, once in use, is expensive to change

Do not use me for internal helpers used only once in a single file.

## The Interface-First Protocol

### Step 1: Name the interface

Give the interface a name before designing it. Naming forces clarity.

Bad: "the function that does the user stuff"
Good: `getUserPermissions(userId: string): Promise<Permission[]>`

### Step 2: Define the contract

Write the contract as a type signature + doc comment before any implementation:

```typescript
/**
 * Returns the resolved permission set for a user in a given context.
 *
 * @param userId - The authenticated user's ID (not the session ID)
 * @param context - The resource context to resolve permissions against
 * @returns Array of resolved Permission objects. Empty array if no permissions.
 * @throws AuthError if userId doesn't correspond to an active user
 * @throws ContextError if context is malformed
 *
 * Invariants:
 * - Result is always an array (never null/undefined)
 * - Order of permissions in the result is not guaranteed
 * - Calling this twice with the same args returns equivalent results (pure)
 */
async function getUserPermissions(
  userId: string,
  context: PermissionContext,
): Promise<Permission[]>;
```

### Step 3: Write the contract checklist

Before implementing, answer:

**Inputs:**

- [ ] What types are accepted? Are they validated, or is that the caller's job?
- [ ] What are the valid ranges / formats? (empty string? null? negative number?)
- [ ] Who is responsible for sanitizing inputs?

**Outputs:**

- [ ] What is returned on success? Always the same shape?
- [ ] What is returned on "not found"? (null, empty array, or an error?)
- [ ] Is the return value stable? (can callers cache it?)

**Errors:**

- [ ] What can go wrong, and how is each case signaled? (exception type, error code, null, union type)
- [ ] Are errors recoverable? Should the caller retry?
- [ ] Is the error message safe to show to end users, or internal-only?

**Side Effects:**

- [ ] Does this mutate state? What state, and under what conditions?
- [ ] Does this have network calls? Can it fail silently?
- [ ] Is it idempotent? Can it be called twice safely?

### Step 4: Design for the caller, not the implementer

The interface is used from the outside. Design from the outside in:

- What does the caller have available? Use those types.
- What does the caller need back? Return exactly that.
- What does the caller not care about? Hide it.

Common anti-pattern: leaking implementation types into the public interface.

```typescript
// Bad: caller now depends on your DB schema
getUserById(id: string): Promise<DatabaseRow>

// Good: caller gets a stable domain type
getUserById(id: string): Promise<User | null>
```

### Step 5: Version for change

For interfaces that will be used externally (API endpoints, npm packages, event schemas):

- Consider what a breaking change would look like
- Add a version field to event schemas
- Use additive changes (new optional fields) before breaking changes
- Document the stability guarantee: "stable", "beta", "internal"

## HTTP API Design

When designing HTTP endpoints, use these conventions:

```
GET    /resources           → list (paginated if large)
GET    /resources/:id       → get one
POST   /resources           → create (returns 201 + created resource)
PUT    /resources/:id       → replace (idempotent)
PATCH  /resources/:id       → partial update
DELETE /resources/:id       → delete (idempotent, returns 204)
```

Status code contract:

- `200` — success with body
- `201` — created; include `Location` header
- `204` — success, no body
- `400` — client error (bad input); include `{ error: string, field?: string }`
- `401` — not authenticated
- `403` — authenticated but not authorized
- `404` — resource not found
- `409` — conflict (duplicate, optimistic lock)
- `422` — validation failure; include field-level errors
- `500` — server error; never expose internal detail

## Interface Stability Signals

Before finalizing an interface, ask: "How painful would a breaking change be?"

| Stability Level | Used by                 | Change policy                    |
|-----------------|-------------------------|----------------------------------|
| Internal        | 1 file                  | Change freely                    |
| Module          | 1 module/package        | Coordinate within the module     |
| Service         | Multiple services/teams | Deprecate before remove; version |
| Public/External | Third parties           | Never break; only extend         |

Match the rigor of the design process to the stability level.

## Output: Interface Doc

For each new interface in a slice, produce:

```
## Interface: [name]
Signature: [type signature]
Contract: [preconditions, postconditions, errors]
Stability: internal / module / service / public
Breaking change risk: low / medium / high
```
