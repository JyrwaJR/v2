---
title: "Typesafe Endpoints"
description: "Learn more about the API endpoints used in the application."
---

# API Endpoints Documentation

## Overview

This document outlines the API endpoints used in the application with a focus on type safety and structured endpoint definitions.

## **Base URL**

```ts
const API_URL = "https://api.example.com"; // Replace with actual base URL
```

## **Endpoint Definitions**

```ts
export const ENDPOINTS = {
  AUTH: {
    LOGIN: `${API_URL}/auth/login`,
    REGISTER: `${API_URL}/auth/register`,
  },
  USERS: {
    GET: (id: string) => `${API_URL}/users/${id}`,
    CREATE: `${API_URL}/users`,
    UPDATE: (id: string) => `${API_URL}/users/${id}`,
    DELETE: (id: string) => `${API_URL}/users/${id}`,
  },
  POSTS: {
    GET_ALL: `${API_URL}/posts`,
    GET: (id: string) => `${API_URL}/posts/${id}`,
    CREATE: `${API_URL}/posts`,
    UPDATE: (id: string) => `${API_URL}/posts/${id}`,
    DELETE: (id: string) => `${API_URL}/posts/${id}`,
  },
};
```

## **Type-Safe Endpoint Usage**

To ensure type safety in API calls, define TypeScript types for API responses and requests.

### **User Types**

```ts
type User = {
  id: string;
  name: string;
  email: string;
};
```

### **Post Types**

```ts
type Post = {
  id: string;
  title: string;
  content: string;
  authorId: string;
};
```

## **Example API Calls with Type Safety**

Using `fetch` with TypeScript:

### **Fetching a User**

```ts
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(ENDPOINTS.USERS.GET(id));
  if (!response.ok) {
    throw new Error("Failed to fetch user");
  }
  return response.json();
}
```

### **Creating a Post**

```ts
async function createPost(post: Omit<Post, "id">): Promise<Post> {
  const response = await fetch(ENDPOINTS.POSTS.CREATE, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(post),
  });
  if (!response.ok) {
    throw new Error("Failed to create post");
  }
  return response.json();
}
```

## **Conclusion**

By defining API endpoints in a structured and type-safe manner, we ensure better maintainability, prevent runtime errors, and improve code readability in the application.
