---
title: "Custom Axios Configuration"
description: "Learn how to set up and use a custom Axios instance with authentication tokens stored in cookies."
---

When building applications that interact with APIs, you often need to configure HTTP clients for consistent handling of requests, especially when dealing with authentication. Here, we demonstrate how to create a custom Axios instance for authenticated API calls using tokens stored in cookies.

## **Key Features**

This implementation streamlines your API calls with:

1. **Custom Axios Instance**:

   - Centralized configuration for HTTP requests.
   - Base URL defined via environment variables.

2. **Automatic Authentication**:

   - Automatically attaches `Authorization` headers to requests using tokens retrieved from cookies.

3. **Error Handling**:
   - Simplified error handling for failed requests.

## **Implementation**

Below is the code for creating the custom Axios instance:

```typescript
import axios from "axios";
import { Cookies } from "react-cookie";

const cookies = new Cookies(); // Instantiate react-cookie's Cookies

const axiosInstance = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_BASE_URL, // Replace with your base URL
});

// Add interceptor to attach cookies dynamically
axiosInstance.interceptors.request.use(
  (config) => {
    const token = cookies.get("AUTH_TOKEN"); // Retrieve the token from cookies
    if (token) {
      config.headers["Authorization"] = `Bearer ${token}`;
      config.headers["Content-Type"] = "application/json";
    }
    return config;
  },
  (error) => {
    // Handle request errors
    return Promise.reject(error);
  },
);

export default axiosInstance;
```

## **Detailed Breakdown**

### **1. Axios Instance Creation**

We start by creating a custom Axios instance using `axios.create`. This instance has a base URL fetched from environment variables (`NEXT_PUBLIC_API_BASE_URL`), allowing flexibility for different environments like development, staging, or production.

### **2. Request Interceptor**

An interceptor is attached to the Axios instance to:

- Dynamically fetch the authentication token stored in cookies.
- Add the token to the `Authorization` header in the format `Bearer <token>`.
- Set the `Content-Type` header to `application/json` for JSON-based requests.

### **3. Error Handling**

The interceptor also includes error handling to capture and handle any issues during the request configuration phase. Errors are passed back using `Promise.reject`, enabling further handling elsewhere in the application.

## **How to Use**

Simply import the configured Axios instance into your application components or services:

```typescript
import axiosInstance from "./path-to-your-axios-config";

axiosInstance
  .get("/protected-resource")
  .then((response) => {
    console.log(response.data);
  })
  .catch((error) => {
    console.error("Request failed:", error);
  });
```

## **Advantages of This Approach**

- **Centralized Configuration**: Simplifies API integration by defining headers and authentication logic in one place.
- **Improved Security**: Keeps authentication tokens out of the application state.
- **Scalability**: Easy to modify or extend for additional headers, error handling, or logging.

## **Assumptions**

- Authentication tokens are stored in cookies under the key `AUTH_TOKEN`.
- The `react-cookie` library is used for accessing cookies in a React environment.
- The base URL for the API is set in the environment variable `NEXT_PUBLIC_API_BASE_URL`.

By utilizing this setup, your application can make secure and authenticated API requests seamlessly, ensuring maintainability and scalability for future enhancements.
