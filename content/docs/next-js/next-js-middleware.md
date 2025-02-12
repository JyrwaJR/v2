---
title: "Next JS Custon Middleware"
description: "Next JS Custon Middleware"
---

# Middleware Chaining Utility for Next.js

## Overview

This utility provides a flexible way to chain multiple middleware functions in a Next.js application. The `chainMiddleware` function enables the composition of middleware factories to process requests in a sequential manner.

## Code Explanation

### **Types Defined**

#### `CustomMiddleware`

```typescript
export type CustomMiddleware = (
  request: NextRequest,
  event: NextFetchEvent,
  response: NextResponse,
) => NextMiddlewareResult | Promise<NextMiddlewareResult>;
```

- Represents a middleware function that takes `NextRequest`, `NextFetchEvent`, and `NextResponse` as parameters.
- Returns either `NextMiddlewareResult` or a Promise resolving to `NextMiddlewareResult`.

#### `MiddlewareFactory`

```typescript
type MiddlewareFactory = (middleware: CustomMiddleware) => CustomMiddleware;
```

- A factory function that wraps another middleware function, allowing middleware composition.

### **Middleware Chaining Function**

#### `chainMiddleware`

```typescript
export function chainMiddleware(
  functions: MiddlewareFactory[],
  index = 0,
): CustomMiddleware {
  const current = functions[index];

  if (current) {
    const next = chainMiddleware(functions, index + 1);
    return current(next);
  }

  return (
    request: NextRequest,
    event: NextFetchEvent,
    response: NextResponse,
  ) => {
    return response;
  };
}
```

- Recursively constructs a middleware chain.
- Calls the next middleware in sequence until no middleware remains.
- The final function in the chain simply returns the `NextResponse` object.

## **Usage**

### **Creating Custom Middleware**

Define individual middleware functions using the `CustomMiddleware` type:

```typescript
const loggerMiddleware: MiddlewareFactory =
  (next) => async (request, event, response) => {
    console.log("Incoming request:", request.url);
    return next(request, event, response);
  };

const authMiddleware: MiddlewareFactory =
  (next) => async (request, event, response) => {
    const isAuthenticated = Boolean(request.cookies.get("authToken"));
    if (!isAuthenticated) {
      return NextResponse.redirect("/login");
    }
    return next(request, event, response);
  };
```

### **Applying Middleware in a Next.js API Route**

```typescript
import { chainMiddleware } from "@/middleware/chainMiddleware";
import type { NextRequest, NextFetchEvent } from "next/server";
import { NextResponse } from "next/server";

const middleware = chainMiddleware([loggerMiddleware, authMiddleware]);

export async function middleware(request: NextRequest, event: NextFetchEvent) {
  return middleware(request, event, NextResponse.next());
}
```

## **Key Benefits**

- **Modular**: Easily add, remove, or rearrange middleware functions.
- **Reusability**: Middleware functions can be used in multiple routes.
- **Scalability**: Encourages a clean separation of concerns for request handling.

## **Conclusion**

This middleware chaining utility enhances Next.js request handling by allowing middleware functions to be composed dynamically. This approach improves maintainability, code reusability, and scalability in your Next.js applications.
