---
title: User Roles and Permissions System
description: A simple user roles and permissions system.
---

````markdown
# User Roles and Permissions System

This module defines a simple user roles and permissions system. It allows checking whether a user has a specific permission based on their roles. The roles are predefined with associated permissions, and the `hasPermission` function checks if a user has a particular permission.

## Types

### `User`

```typescript
export type User = { roles: Role[]; id: string };
```
````

A `User` object contains:

- `id`: A string identifier for the user.
- `roles`: An array of roles assigned to the user. Each role determines the permissions that the user can have.

### `Role`

```typescript
type Role = keyof typeof ROLES;
```

A `Role` is a key of the `ROLES` object. The predefined roles are:

- `admin`
- `moderator`
- `user`

These roles map to the permissions defined in the `ROLES` object.

### `Permission`

```typescript
type Permission = (typeof ROLES)[Role][number];
```

A `Permission` is a string that represents a specific action or operation that a user can perform. Permissions are associated with each role in the `ROLES` object, and can be:

- `view:comments`
- `create:comments`
- `update:comments`
- `delete:comments`

## Constant: ROLES

```typescript
const ROLES = {
  admin: [
    "view:comments",
    "create:comments",
    "update:comments",
    "delete:comments",
  ],
  moderator: ["view:comments", "create:comments", "delete:comments"],
  user: ["view:comments", "create:comments"],
} as const;
```

The `ROLES` constant defines the permissions associated with each role:

- `admin`: Has full permissions for comments (view, create, update, delete).
- `moderator`: Can view, create, and delete comments (cannot update).
- `user`: Can view and create comments (cannot update or delete).

The `as const` assertion ensures that the roles and permissions are treated as literal types, which helps with type safety.

## Function: `hasPermission`

```typescript
export function hasPermission(user: User, permission: Permission) {
  return user.roles.some((role) =>
    (ROLES[role] as readonly Permission[]).includes(permission),
  );
}
```

The `hasPermission` function checks if a user has a specific permission.

### Parameters:

- `user`: A `User` object that has a list of roles.
- `permission`: A `Permission` string (e.g., `view:comments`, `create:comments`, etc.).

### Returns:

- A boolean value indicating whether the user has the given permission. It returns `true` if the user has the permission, and `false` otherwise.

### Logic:

1. The function loops through the `roles` assigned to the user.
2. For each role, it checks if the permission is included in that role's list of permissions (defined in `ROLES`).
3. If the permission is found for any of the user's roles, the function returns `true`. Otherwise, it returns `false`.

## Example Usage

### Example 1: Checking if a user can create a comment

```typescript
const user: User = { id: "1", roles: ["user"] };

// Can create a comment
hasPermission(user, "create:comments");
```

In this example, the user has the `user` role, which includes the permission `create:comments`. So, the function will return `true`.

### Example 2: Checking if a user can view all comments

```typescript
const user: User = { id: "1", roles: ["user"] };

// Can view all comments
hasPermission(user, "view:comments");
```

In this example, the user has the `user` role, which includes the permission `view:comments`. So, the function will return `true`.

### Example 3: Checking if a user can update a comment

```typescript
const user: User = { id: "1", roles: ["user"] };

// Cannot update a comment
hasPermission(user, "update:comments");
```

In this example, the user has the `user` role, but the `user` role does not include the `update:comments` permission. Therefore, the function will return `false`.

### Example 4: Admin user with all permissions

```typescript
const user: User = { id: "1", roles: ["admin"] };

// Can delete a comment
hasPermission(user, "delete:comments");
```

In this case, the user has the `admin` role, which includes all permissions (`view:comments`, `create:comments`, `update:comments`, `delete:comments`). Therefore, the function will return `true`.

## Conclusion

This system provides an easy way to manage user roles and permissions, ensuring that users can only perform actions that are authorized based on their roles. The `hasPermission` function helps validate whether a user has the required permission for a specific action.

```

This documentation provides an overview of the code's types, constants, and functions, and includes example usage to clarify how the system works.
```
