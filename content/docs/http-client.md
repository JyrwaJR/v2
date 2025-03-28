---
title: "HTTP Client Module"
description: "Learn how to create a robust HTTP client using Axios, with standardized response handling, error management, and React Query integration."
---

## **Overview**

This module is a custom HTTP client built using **Axios** to handle API requests in a structured manner. It includes:

- **Automatic error handling** using `handleAxiosError`
- **Consistent API responses** with a generic `ApiResponse<T>` format
- **Support for GET, POST, PUT, DELETE** requests
- **Integration with React Query** for better API data management

---

## **Code Explanation**

### **1. API Response Type Definition**

The `ApiResponse` interface defines a standard structure for API responses.

```ts
export interface ApiResponse<T> {
  success: boolean;
  message: string;
  data: T | null; // 👈 Explicitly allow `null`
  error?: string | Record<string, unknown>;
}
```

### **2. Handling API Errors**

The `handleAxiosError` function captures and standardizes error responses.

```ts
import { ApiResponse } from "@/types/apiResponse";
import { AxiosError } from "axios";

export const handleAxiosError = <T>(error: unknown): ApiResponse<T> => {
  let errorMessage = "Something went wrong. Please try again.";
  let errorDetails: string | Record<string, unknown> = "";

  if (error instanceof AxiosError) {
    if (error.response) {
      errorMessage =
        (error.response.data as { message?: string })?.message || errorMessage;
      errorDetails =
        (error.response.data as { error?: string | Record<string, unknown> })
          ?.error ||
        error.response.data ||
        "";
    } else if (error.request) {
      errorMessage = "No response from server. Please check your connection.";
    } else {
      errorMessage = error.message;
    }
  } else if (error instanceof Error) {
    errorMessage = error.message;
  }

  return {
    success: false,
    message: errorMessage,
    error: errorDetails,
    data: null,
  };
};
```

### **3. Handling API Responses**
The `handleResponse` function ensures all responses follow a consistent format.

```ts
import axiosInstance from "./api";
import { ApiResponse } from "@/types/apiResponse";
import { AxiosRequestConfig, AxiosResponse } from "axios";

const handleResponse = <T>(
  response: AxiosResponse<ApiResponse<T>>,
): ApiResponse<T> => {
  return {
    success: response.data.success,
    message: response.data.message || "Request successful",
    data: response.data.data ?? null,
  };
};
```

### **4. Creating HTTP Methods**

A **generic** HTTP client is implemented to support **GET, POST, PUT, DELETE** requests.

```ts
const http = {
  get: async <T>(
    url: string,
    config?: AxiosRequestConfig,
  ): Promise<ApiResponse<T>> => {
    try {
      const response = await axiosInstance.get(url, config);
      return handleResponse<T>(response);
    } catch (error) {
      return handleAxiosError<T>(error);
    }
  },

  post: async <T>(
    url: string,
    data?: object,
    config?: AxiosRequestConfig,
  ): Promise<ApiResponse<T>> => {
    try {
      const response = await axiosInstance.post(url, data, config);
      return handleResponse<T>(response);
    } catch (error) {
      return handleAxiosError<T>(error);
    }
  },

  put: async <T>(
    url: string,
    data?: object,
    config?: AxiosRequestConfig,
  ): Promise<ApiResponse<T>> => {
    try {
      const response = await axiosInstance.put(url, data, config);
      return handleResponse<T>(response);
    } catch (error) {
      return handleAxiosError<T>(error);
    }
  },

  delete: async <T>(
    url: string,
    config?: AxiosRequestConfig,
  ): Promise<ApiResponse<T>> => {
    try {
      const response = await axiosInstance.delete(url, config);
      return handleResponse<T>(response);
    } catch (error) {
      return handleAxiosError<T>(error);
    }
  },
};

export default http;
```

---

## **Usage in a React Application**

### **1. Using HTTP Client in a React Component**

```tsx
import http from "@/utils/http";

const fetchData = async () => {
  const response = await http.get<{ name: string; age: number }>(
    "/api/user/profile",
  );

  if (response.success) {
    console.log("User Profile:", response.data);
  } else {
    console.error("Error fetching profile:", response.message);
  }
};
```

---

## **2. Using React Query for Data Fetching**

React Query simplifies API calls with caching and automatic updates.

### **Installing React Query**

```sh
npm install @tanstack/react-query
```

### **Setting Up React Query in `App.tsx`**

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <MyComponent />
    </QueryClientProvider>
  );
}

export default App;
```

### **Fetching Data with React Query**

```tsx
import { useQuery } from "@tanstack/react-query";
import http from "@/utils/http";

type UserProfileT = {
  name: string;
  age: number;
};
const fetchUserProfile = async () => {
  return await http.get<UserProfileT>("/api/user/profile");
};

const UserProfile = () => {
  const { data, error, isLoading } = useQuery({
    queryKey: ["userProfile"],
    queryFn: fetchUserProfile,
  });

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <h2>User Profile</h2>
      <p>Name: {data?.data?.name}</p>
      <p>Age: {data?.data?.age}</p>
    </div>
  );
};

export default UserProfile;
```

---

## **Key Benefits**

- ✅ **Reusable HTTP Client**
- ✅ **Consistent API Response Format**
- ✅ **Centralized Error Handling**
- ✅ **Type Safety with TypeScript**
- ✅ **Seamless React Query Integration**

---

## **Conclusion**

This structured **HTTP client** enhances API calls, ensures **error handling**, and integrates smoothly with **React Query**, making API state management effortless. 🚀
