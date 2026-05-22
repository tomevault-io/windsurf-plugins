---
trigger: always_on
description: This document outlines the frontend data fetching patterns and best practices for the OpenFrame project.
---

# Frontend Data Fetching

This document outlines the frontend data fetching patterns and best practices for the OpenFrame project.

## Apollo Client

OpenFrame uses Apollo Client for GraphQL data fetching:

```typescript
// src/apollo/apolloClient.ts
import { ApolloClient, InMemoryCache, HttpLink, from } from '@apollo/client/core';
import { onError } from '@apollo/client/link/error';
import { logoutUser, refreshToken } from '@/services/AuthService';

// Error handling link
const errorLink = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors) {
    graphQLErrors.forEach(({ message, locations, path }) => {
      console.error(
        `[GraphQL error]: Message: ${message}, Location: ${locations}, Path: ${path}`
      );
      
      // Handle authentication errors
      if (message.includes('not authenticated') || message.includes('jwt expired')) {
        handleGraphQLAuthError();
      }
    });
  }
  
  if (networkError) {
    console.error(`[Network error]: ${networkError}`);
  }
});

// Handle authentication errors
async function handleGraphQLAuthError() {
  try {
    // Try to refresh the token
    const success = await refreshToken();
    if (!success) {
      // If refresh fails, logout the user
      logoutUser();
    }
  } catch (error) {
    logoutUser();
  }
}

// HTTP link
const httpLink = new HttpLink({
  uri: import.meta.env.VITE_API_URL + '/graphql',
  credentials: 'include',
});

// Create Apollo Client
export const apolloClient = new ApolloClient({
  link: from([errorLink, httpLink]),
  cache: new InMemoryCache(),
  defaultOptions: {
    watchQuery: {
      fetchPolicy: 'cache-and-network',
      errorPolicy: 'all',
    },
    query: {
      fetchPolicy: 'network-only',
      errorPolicy: 'all',
    },
    mutate: {
      errorPolicy: 'all',
    },
  },
});
```

## Vue Apollo Composables

Use Vue Apollo composables for GraphQL operations:

```typescript
// src/composables/useDevices.ts
import { useQuery, useMutation } from '@vue/apollo-composable';
import { gql } from '@apollo/client/core';
import { ref, computed } from 'vue';
import type { Device } from '@/types';

export function useDevices() {
  const GET_DEVICES = gql`
    query GetDevices {
      devices {
        id
        hostname
        operatingSystem
        status
        lastSeen
      }
    }
  `;
  
  const CREATE_DEVICE = gql`
    mutation CreateDevice($input: DeviceInput!) {
      createDevice(input: $input) {
        id
        hostname
        operatingSystem
        status
        lastSeen
      }
    }
  `;
  
  // Query devices
  const { result, loading, error, refetch } = useQuery(GET_DEVICES);
  
  // Create device mutation
  const { mutate: createDevice, loading: createLoading } = useMutation(CREATE_DEVICE);
  
  // Computed property for devices
  const devices = computed(() => result.value?.devices || []);
  
  // Create a new device
  const addDevice = async (device: Omit<Device, 'id'>) => {
    try {
      const response = await createDevice({
        input: device
      });
      
      await refetch();
      return response?.data?.createDevice;
    } catch (error) {
      console.error('Error creating device:', error);
      throw error;
    }
  };
  
  return {
    devices,
    loading,
    error,
    refetch,
    addDevice,
    createLoading
  };
}
```

## REST API Fetching

For REST API endpoints, use Axios:

```typescript
// src/services/ApiService.ts
import axios, { AxiosInstance, AxiosRequestConfig } from 'axios';
import { refreshToken, logoutUser } from '@/services/AuthService';

class ApiService {
  private api: AxiosInstance;
  private isRefreshing = false;
  private failedQueue: any[] = [];
  
  constructor() {
    this.api = axios.create({
      baseURL: import.meta.env.VITE_API_URL,
      headers: {
        'Content-Type': 'application/json',
      },
      withCredentials: true,
    });
    
    this.setupInterceptors();
  }
  
  private setupInterceptors() {
    // Request interceptor
    this.api.interceptors.request.use(
      (config) => {
        // Add auth token if available
        const token = localStorage.getItem('auth_token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );
    
    // Response interceptor
    this.api.interceptors.response.use(
      (response) => response,
      async (error) => {
        const originalRequest = error.config;
        
        // Handle 401 Unauthorized errors
        if (error.response?.status === 401 && !originalRequest._retry) {
          if (this.isRefreshing) {
            // If token refresh is in progress, queue the request
            return new Promise((resolve, reject) => {
              this.failedQueue.push({ resolve, reject });
            })
              .then((token) => {
                originalRequest.headers.Authorization = `Bearer ${token}`;
                return this.api(originalRequest);
              })
              .catch((err) => Promise.reject(err));
          }
          
          originalRequest._retry = true;
          this.isRefreshing = true;
          
          try {
            // Try to refresh the token

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
