# Unit and Integration Testing Guide

## Overview

This guide provides comprehensive instructions for writing, running, and maintaining unit and integration tests for the MarineMarket platform. Our testing approach ensures code quality, reliability, and maintainability across both backend and frontend components.

## Backend Testing (Node.js/TypeScript)

### Testing Framework Setup

**Framework**: Jest with TypeScript support
**Configuration**: `backend/jest.config.js`
**Test Utilities**: `backend/src/shared/test-utils.ts`

```javascript
// backend/jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/*.test.ts', '**/*.integration.test.ts'],
  transform: {
    '^.+\\.ts$': 'ts-jest',
  },
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.test.ts',
    '!src/**/*.integration.test.ts',
    '!src/**/test-setup.ts',
    '!**/node_modules/**',
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  testTimeout: 10000,
  verbose: true
};
```

### Unit Testing Patterns

#### 1. Lambda Function Handler Testing

**File**: `backend/src/admin-service/admin-service.test.ts`

```typescript
import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';
import { handler } from './index';
import { UserRole, UserStatus, AdminPermission } from '../types/common';

// Mock AWS SDK
jest.mock('@aws-sdk/client-dynamodb');
jest.mock('@aws-sdk/lib-dynamodb');

describe('Admin Service', () => {
  const mockRequestContext = {
    requestId: 'test-request-id',
    identity: { sourceIp: '127.0.0.1' }
  };

  const mockAdminUser = {
    sub: 'admin-user-id',
    email: 'admin@example.com',
    role: UserRole.ADMIN,
    permissions: [AdminPermission.USER_MANAGEMENT],
    sessionId: 'session-123'
  };

  beforeEach(() => {
    jest.clearAllMocks();
    process.env.USERS_TABLE = 'test-users';
    process.env.LISTINGS_TABLE = 'test-listings';
  });

  describe('Dashboard Metrics', () => {
    it('should return dashboard metrics successfully', async () => {
      // Mock DynamoDB responses
      mockDocClient.send
        .mockResolvedValueOnce({ Count: 150 }) // total users
        .mockResolvedValueOnce({ Count: 120 }) // active users
        .mockResolvedValueOnce({ Count: 75 });  // total listings

      const event: Partial<APIGatewayProxyEvent> = {
        httpMethod: 'GET',
        path: '/admin/dashboard',
        requestContext: mockRequestContext,
        user: mockAdminUser
      };

      const result = await handler(event as any);
      
      expect(result.statusCode).toBe(200);
      const body = JSON.parse(result.body);
      expect(body.metrics.totalUsers).toBe(150);
      expect(body.metrics.activeUsers).toBe(120);
      expect(body.metrics.totalListings).toBe(75);
    });
  });
});
```

#### 2. Utility Function Testing

**File**: `backend/src/shared/utils.test.ts`

```typescript
import { createResponse, createErrorResponse, validateEmail } from './utils';

describe('Utility Functions', () => {
  describe('createResponse', () => {
    it('should create valid API Gateway response', () => {
      const response = createResponse(200, { message: 'Success' });
      
      expect(response.statusCode).toBe(200);
      expect(response.headers['Content-Type']).toBe('application/json');
      expect(response.headers['Access-Control-Allow-Origin']).toBe('*');
      expect(JSON.parse(response.body)).toEqual({ message: 'Success' });
    });
  });

  describe('validateEmail', () => {
    it('should validate correct email formats', () => {
      expect(validateEmail('user@example.com')).toBe(true);
      expect(validateEmail('test.email+tag@domain.co.uk')).toBe(true);
    });

    it('should reject invalid email formats', () => {
      expect(validateEmail('invalid-email')).toBe(false);
      expect(validateEmail('user@')).toBe(false);
      expect(validateEmail('@domain.com')).toBe(false);
    });
  });
});
```

#### 3. Middleware Testing

**File**: `backend/src/shared/middleware.test.ts`

```typescript
import { withAdminAuth, withRateLimit, withAuditLog } from './middleware';
import { AdminPermission } from '../types/common';

describe('Middleware Functions', () => {
  describe('withAdminAuth', () => {
    it('should allow access with valid admin permissions', async () => {
      const mockHandler = jest.fn().mockResolvedValue({ statusCode: 200 });
      const wrappedHandler = withAdminAuth([AdminPermission.USER_MANAGEMENT])(mockHandler);

      const event = {
        user: {
          role: 'ADMIN',
          permissions: [AdminPermission.USER_MANAGEMENT]
        }
      };

      await wrappedHandler(event, {});
      expect(mockHandler).toHaveBeenCalled();
    });

    it('should deny access without required permissions', async () => {
      const mockHandler = jest.fn();
      const wrappedHandler = withAdminAuth([AdminPermission.USER_MANAGEMENT])(mockHandler);

      const event = {
        user: {
          role: 'ADMIN',
          permissions: [AdminPermission.CONTENT_MODERATION] // Wrong permission
        }
      };

      const result = await wrappedHandler(event, {});
      expect(result.statusCode).toBe(403);
      expect(mockHandler).not.toHaveBeenCalled();
    });
  });
});
```

### Integration Testing Patterns

#### 1. Full API Flow Testing

**File**: `backend/src/admin-service/admin-service.integration.test.ts`

```typescript
import { APIGatewayProxyEvent } from 'aws-lambda';
import { handler } from './index';

// Integration tests use real AWS services in test environment
describe('Admin Service Integration Tests', () => {
  beforeAll(() => {
    process.env.USERS_TABLE = 'test-users-integration';
    process.env.LISTINGS_TABLE = 'test-listings-integration';
  });

  describe('Permission-based Access Control', () => {
    it('should allow super admin to access all endpoints', async () => {
      const endpoints = [
        { path: '/admin/dashboard', method: 'GET' },
        { path: '/admin/users', method: 'GET' },
        { path: '/admin/listings', method: 'GET' }
      ];

      for (const endpoint of endpoints) {
        const event: Partial<APIGatewayProxyEvent> = {
          httpMethod: endpoint.method,
          path: endpoint.path,
          user: mockSuperAdmin
        };

        const result = await handler(event as any);
        expect(result.statusCode).not.toBe(403);
      }
    });
  });

  describe('Data Validation and Sanitization', () => {
    it('should validate user status update requests', async () => {
      const event: Partial<APIGatewayProxyEvent> = {
        httpMethod: 'PUT',
        path: '/admin/users/user-123/status',
        pathParameters: { userId: 'user-123' },
        body: JSON.stringify({
          status: 'invalid-status',
          reason: 'Test reason'
        }),
        user: mockSuperAdmin
      };

      const result = await handler(event as any);
      
      expect(result.statusCode).toBe(400);
      const body = JSON.parse(result.body);
      expect(body.error.code).toBe('VALIDATION_ERROR');
    });
  });
});
```

### Test Utilities and Helpers

**File**: `backend/src/shared/test-utils.ts`

```typescript
import { APIGatewayProxyEvent, Context } from 'aws-lambda';

export function createMockEvent(overrides: Partial<APIGatewayProxyEvent> = {}): APIGatewayProxyEvent {
  return {
    body: null,
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer mock-jwt-token',
      ...overrides.headers
    },
    httpMethod: 'GET',
    path: '/admin/test',
    pathParameters: null,
    queryStringParameters: null,
    requestContext: {
      requestId: 'test-request-id',
      identity: { sourceIp: '192.168.1.1' }
    },
    ...overrides
  } as APIGatewayProxyEvent;
}

export function createMockAdminUser(overrides: any = {}) {
  return {
    sub: 'admin-1',
    email: 'admin@test.com',
    name: 'Admin User',
    role: 'ADMIN',
    permissions: ['user_management', 'content_moderation'],
    sessionId: 'session-123',
    exp: Math.floor(Date.now() / 1000) + 3600,
    ...overrides
  };
}

export function expectValidApiResponse(response: any, expectedStatusCode: number = 200) {
  expect(response).toBeDefined();
  expect(response.statusCode).toBe(expectedStatusCode);
  expect(response.headers).toBeDefined();
  expect(response.body).toBeDefined();
  
  if (response.body) {
    expect(() => JSON.parse(response.body)).not.toThrow();
  }
}

export function expectErrorResponse(response: any, expectedCode: string, expectedStatusCode: number = 400) {
  expectValidApiResponse(response, expectedStatusCode);
  
  const body = JSON.parse(response.body);
  expect(body.error).toBeDefined();
  expect(body.error.code).toBe(expectedCode);
  expect(body.error.message).toBeDefined();
}
```

### Running Backend Tests

```bash
# Navigate to backend directory
cd backend

# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage

# Run only unit tests
npm test -- --testPathPattern="\.test\.ts$"

# Run only integration tests
npm test -- --testPathPattern="\.integration\.test\.ts$"

# Run specific test file
npm test -- admin-service.test.ts

# Run tests matching pattern
npm test -- --testNamePattern="Dashboard"
```

## Frontend Testing (React/TypeScript)

### Testing Framework Setup

**Framework**: Vitest with React Testing Library
**Configuration**: `frontend/vite.config.ts`
**Test Setup**: `frontend/src/test/setup.ts`

```typescript
// frontend/vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
  },
})
```

```typescript
// frontend/src/test/setup.ts
import '@testing-library/jest-dom';
```

### Unit Testing Patterns

#### 1. React Component Testing

**File**: `frontend/src/components/admin/__tests__/AdminHeader.test.tsx`

```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { AdminHeader } from '../AdminHeader';
import { AdminAuthProvider } from '../AdminAuthProvider';

const renderWithProviders = (component: React.ReactElement) => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  });

  return render(
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        <AdminAuthProvider>
          {component}
        </AdminAuthProvider>
      </BrowserRouter>
    </QueryClientProvider>
  );
};

describe('AdminHeader', () => {
  const mockUser = {
    id: 'admin-1',
    email: 'admin@test.com',
    name: 'Admin User',
    role: 'admin'
  };

  it('should display user information when authenticated', () => {
    renderWithProviders(<AdminHeader user={mockUser} />);
    
    expect(screen.getByText('Admin User')).toBeInTheDocument();
    expect(screen.getByText('admin@test.com')).toBeInTheDocument();
  });

  it('should show logout button and handle click', () => {
    const mockLogout = vi.fn();
    renderWithProviders(<AdminHeader user={mockUser} onLogout={mockLogout} />);
    
    const logoutButton = screen.getByRole('button', { name: /logout/i });
    expect(logoutButton).toBeInTheDocument();
    
    fireEvent.click(logoutButton);
    expect(mockLogout).toHaveBeenCalled();
  });

  it('should display navigation menu', () => {
    renderWithProviders(<AdminHeader user={mockUser} />);
    
    expect(screen.getByRole('link', { name: /dashboard/i })).toBeInTheDocument();
    expect(screen.getByRole('link', { name: /users/i })).toBeInTheDocument();
    expect(screen.getByRole('link', { name: /listings/i })).toBeInTheDocument();
  });
});
```

#### 2. Custom Hook Testing

**File**: `frontend/src/hooks/__tests__/useAdminAuth.test.ts`

```typescript
import { renderHook, act } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useAdminAuth } from '../useAdminAuth';
import * as adminApi from '../../services/adminApi';

// Mock the API
vi.mock('../../services/adminApi');
const mockAdminApi = vi.mocked(adminApi);

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  });
  
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

describe('useAdminAuth', () => {
  beforeEach(() => {
    vi.clearAllMocks();
    localStorage.clear();
  });

  it('should initialize with no user when not authenticated', () => {
    const { result } = renderHook(() => useAdminAuth(), {
      wrapper: createWrapper(),
    });

    expect(result.current.user).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
    expect(result.current.isLoading).toBe(false);
  });

  it('should login successfully with valid credentials', async () => {
    const mockUser = {
      id: 'admin-1',
      email: 'admin@test.com',
      name: 'Admin User',
      role: 'admin'
    };

    mockAdminApi.login.mockResolvedValue({
      token: 'mock-token',
      user: mockUser,
      permissions: ['user_management']
    });

    const { result } = renderHook(() => useAdminAuth(), {
      wrapper: createWrapper(),
    });

    await act(async () => {
      await result.current.login('admin@test.com', 'password123');
    });

    expect(result.current.user).toEqual(mockUser);
    expect(result.current.isAuthenticated).toBe(true);
    expect(localStorage.getItem('admin-token')).toBe('mock-token');
  });

  it('should handle login errors', async () => {
    mockAdminApi.login.mockRejectedValue(new Error('Invalid credentials'));

    const { result } = renderHook(() => useAdminAuth(), {
      wrapper: createWrapper(),
    });

    await act(async () => {
      try {
        await result.current.login('admin@test.com', 'wrong-password');
      } catch (error) {
        expect(error.message).toBe('Invalid credentials');
      }
    });

    expect(result.current.user).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
  });

  it('should logout and clear stored data', async () => {
    localStorage.setItem('admin-token', 'mock-token');
    localStorage.setItem('admin-user', JSON.stringify({ id: 'admin-1' }));

    const { result } = renderHook(() => useAdminAuth(), {
      wrapper: createWrapper(),
    });

    act(() => {
      result.current.logout();
    });

    expect(result.current.user).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
    expect(localStorage.getItem('admin-token')).toBeNull();
    expect(localStorage.getItem('admin-user')).toBeNull();
  });
});
```

#### 3. Service/API Testing

**File**: `frontend/src/services/__tests__/adminApi.test.ts`

```typescript
import { adminApi } from '../adminApi';
import { apiClient } from '../apiClient';

// Mock the API client
vi.mock('../apiClient');
const mockApiClient = vi.mocked(apiClient);

describe('adminApi', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe('login', () => {
    it('should login successfully with valid credentials', async () => {
      const mockResponse = {
        token: 'mock-jwt-token',
        user: {
          id: 'admin-1',
          email: 'admin@test.com',
          name: 'Admin User',
          role: 'admin'
        },
        permissions: ['user_management', 'content_moderation']
      };

      mockApiClient.post.mockResolvedValue({ data: mockResponse });

      const result = await adminApi.login('admin@test.com', 'password123');

      expect(mockApiClient.post).toHaveBeenCalledWith('/admin/auth/login', {
        email: 'admin@test.com',
        password: 'password123'
      });
      expect(result).toEqual(mockResponse);
    });

    it('should throw error for invalid credentials', async () => {
      mockApiClient.post.mockRejectedValue({
        response: {
          status: 401,
          data: { error: { message: 'Invalid credentials' } }
        }
      });

      await expect(adminApi.login('admin@test.com', 'wrong-password'))
        .rejects.toThrow('Invalid credentials');
    });
  });

  describe('getDashboardMetrics', () => {
    it('should fetch dashboard metrics successfully', async () => {
      const mockMetrics = {
        totalUsers: 150,
        activeUsers: 120,
        totalListings: 75,
        systemHealth: 'healthy'
      };

      mockApiClient.get.mockResolvedValue({ data: mockMetrics });

      const result = await adminApi.getDashboardMetrics();

      expect(mockApiClient.get).toHaveBeenCalledWith('/admin/dashboard');
      expect(result).toEqual(mockMetrics);
    });
  });
});
```

### Integration Testing Patterns

#### 1. Component Integration Testing

**File**: `frontend/src/pages/admin/__tests__/AdminDashboard.integration.test.tsx`

```typescript
import { render, screen, waitFor } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { AdminDashboard } from '../AdminDashboard';
import * as adminApi from '../../../services/adminApi';

vi.mock('../../../services/adminApi');
const mockAdminApi = vi.mocked(adminApi);

const renderWithProviders = (component: React.ReactElement) => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  });

  return render(
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        {component}
      </BrowserRouter>
    </QueryClientProvider>
  );
};

describe('AdminDashboard Integration', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should load and display dashboard data', async () => {
    const mockMetrics = {
      totalUsers: 150,
      activeUsers: 120,
      newUsersToday: 5,
      totalListings: 75,
      activeListings: 60,
      systemHealth: {
        status: 'healthy',
        services: {
          database: 'healthy',
          api: 'healthy'
        }
      }
    };

    mockAdminApi.getDashboardMetrics.mockResolvedValue(mockMetrics);

    renderWithProviders(<AdminDashboard />);

    // Should show loading state initially
    expect(screen.getByText(/loading/i)).toBeInTheDocument();

    // Wait for data to load
    await waitFor(() => {
      expect(screen.getByText('150')).toBeInTheDocument(); // Total users
      expect(screen.getByText('120')).toBeInTheDocument(); // Active users
      expect(screen.getByText('75')).toBeInTheDocument();  // Total listings
    });

    // Should display system health
    expect(screen.getByText(/system healthy/i)).toBeInTheDocument();
  });

  it('should handle API errors gracefully', async () => {
    mockAdminApi.getDashboardMetrics.mockRejectedValue(
      new Error('Failed to fetch metrics')
    );

    renderWithProviders(<AdminDashboard />);

    await waitFor(() => {
      expect(screen.getByText(/error loading dashboard/i)).toBeInTheDocument();
    });
  });
});
```

### Running Frontend Tests

```bash
# Navigate to frontend directory
cd frontend

# Run all tests
npm test

# Run tests in watch mode (for development)
npm run test:watch

# Run tests once (for CI)
npm run test:run

# Run tests with coverage
npm run test:coverage

# Run specific test file
npm test -- AdminHeader.test.tsx

# Run tests matching pattern
npm test -- --reporter=verbose --run --testNamePattern="login"
```

## Testing Best Practices

### 1. Test Structure and Organization

**Follow the AAA Pattern**:
- **Arrange**: Set up test data and mocks
- **Act**: Execute the function or interaction
- **Assert**: Verify the expected outcome

```typescript
describe('User Management', () => {
  it('should update user status successfully', async () => {
    // Arrange
    const userId = 'user-123';
    const newStatus = UserStatus.SUSPENDED;
    const mockUser = createMockUser({ id: userId });
    mockDocClient.send.mockResolvedValue({ Item: mockUser });

    // Act
    const result = await updateUserStatus(userId, newStatus, 'Policy violation');

    // Assert
    expect(result.success).toBe(true);
    expect(result.user.status).toBe(newStatus);
  });
});
```

### 2. Mocking Strategies

**Mock External Dependencies**:
```typescript
// Mock AWS SDK
jest.mock('@aws-sdk/lib-dynamodb');

// Mock API clients
vi.mock('../services/apiClient');

// Mock React Router
vi.mock('react-router-dom', () => ({
  ...vi.importActual('react-router-dom'),
  useNavigate: () => vi.fn(),
}));
```

**Use Test Utilities for Consistent Mocks**:
```typescript
// Create reusable mock factories
export const createMockUser = (overrides = {}) => ({
  id: 'user-123',
  email: 'user@test.com',
  name: 'Test User',
  status: 'ACTIVE',
  ...overrides
});
```

### 3. Test Data Management

**Use Factories for Test Data**:
```typescript
class UserFactory {
  static create(overrides = {}) {
    return {
      id: `user-${Date.now()}`,
      email: 'test@example.com',
      name: 'Test User',
      role: UserRole.USER,
      status: UserStatus.ACTIVE,
      createdAt: new Date().toISOString(),
      ...overrides
    };
  }

  static createAdmin(overrides = {}) {
    return this.create({
      role: UserRole.ADMIN,
      permissions: [AdminPermission.USER_MANAGEMENT],
      ...overrides
    });
  }
}
```

### 4. Async Testing Patterns

**Handle Promises Properly**:
```typescript
// Use async/await
it('should handle async operations', async () => {
  const result = await asyncFunction();
  expect(result).toBeDefined();
});

// Use waitFor for React components
await waitFor(() => {
  expect(screen.getByText('Loaded')).toBeInTheDocument();
});

// Test error handling
await expect(failingFunction()).rejects.toThrow('Expected error');
```

### 5. Coverage Guidelines

**Aim for High Coverage in Critical Areas**:
- Authentication and authorization: 95%+
- Business logic: 90%+
- API endpoints: 85%+
- UI components: 80%+
- Utility functions: 90%+

**Focus on Meaningful Tests**:
- Test behavior, not implementation
- Cover edge cases and error scenarios
- Ensure tests are maintainable and readable

### 6. Performance Testing in Unit Tests

**Test Performance-Critical Functions**:
```typescript
describe('Performance Tests', () => {
  it('should process large datasets efficiently', () => {
    const largeDataset = Array(10000).fill(null).map((_, i) => ({ id: i }));
    
    const startTime = performance.now();
    const result = processLargeDataset(largeDataset);
    const endTime = performance.now();
    
    expect(result).toBeDefined();
    expect(endTime - startTime).toBeLessThan(100); // Should complete in <100ms
  });
});
```

## Continuous Integration

### GitHub Actions Integration

**Test Workflow** (`.github/workflows/test.yml`):
```yaml
name: Test Suite

on: [push, pull_request]

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: cd backend && npm ci
      - name: Run tests
        run: cd backend && npm test -- --coverage
      - name: Upload coverage
        uses: codecov/codecov-action@v3

  frontend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: cd frontend && npm ci
      - name: Run tests
        run: cd frontend && npm run test:run -- --coverage
```

### Local Development Workflow

**Pre-commit Testing**:
```bash
# Run quick tests before committing
npm run test:quick

# Run full test suite
npm run test:all

# Check coverage
npm run test:coverage
```

**Test-Driven Development**:
1. Write failing test
2. Implement minimal code to pass
3. Refactor while keeping tests green
4. Repeat cycle

## Troubleshooting Common Issues

### Backend Testing Issues

**DynamoDB Mocking Problems**:
```typescript
// Ensure proper mock setup
const mockDocClient = {
  send: jest.fn()
};

jest.mock('@aws-sdk/lib-dynamodb', () => ({
  DynamoDBDocumentClient: {
    from: jest.fn(() => mockDocClient)
  },
  ScanCommand: jest.fn(),
  QueryCommand: jest.fn()
}));
```

**Environment Variable Issues**:
```typescript
beforeEach(() => {
  process.env.USERS_TABLE = 'test-users';
  process.env.JWT_SECRET = 'test-secret';
});

afterEach(() => {
  delete process.env.USERS_TABLE;
  delete process.env.JWT_SECRET;
});
```

### Frontend Testing Issues

**React Query Testing**:
```typescript
// Disable retries in tests
const queryClient = new QueryClient({
  defaultOptions: {
    queries: { retry: false },
    mutations: { retry: false },
  },
});
```

**Router Testing**:
```typescript
// Mock useNavigate for component tests
const mockNavigate = vi.fn();
vi.mock('react-router-dom', () => ({
  ...vi.importActual('react-router-dom'),
  useNavigate: () => mockNavigate,
}));
```

**Async Component Testing**:
```typescript
// Wait for async operations
await waitFor(() => {
  expect(screen.getByText('Expected text')).toBeInTheDocument();
});

// Use findBy queries for async elements
const element = await screen.findByText('Async content');
```

## Conclusion

This comprehensive testing guide provides the foundation for maintaining high-quality code in the MarineMarket platform. By following these patterns and practices, developers can ensure reliable, maintainable, and well-tested code that meets our quality standards.

Remember to:
- Write tests first when possible (TDD)
- Focus on testing behavior, not implementation
- Keep tests simple and focused
- Maintain good test coverage
- Regularly review and refactor tests
- Use the provided utilities and patterns for consistency