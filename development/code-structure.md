# Code Structure and Conventions

This document outlines the codebase organization, naming conventions, and coding standards for the MarineMarket boat listing platform. Following these conventions ensures consistency, maintainability, and team collaboration.

## 🏗️ Project Architecture Overview

The MarineMarket platform follows a **monorepo structure** with clear separation of concerns:

```
boat_listing_project/
├── 📁 frontend/          # React TypeScript application
├── 📁 backend/           # AWS Lambda functions (Node.js TypeScript)
├── 📁 infrastructure/    # AWS CDK infrastructure code
├── 📁 docs/             # Project documentation
├── 📁 config/           # Configuration files and examples
├── 📁 scripts/          # Build and deployment scripts
└── 📄 README.md         # Project overview
```

### Architecture Principles

1. **Separation of Concerns**: Clear boundaries between frontend, backend, and infrastructure
2. **Type Safety**: TypeScript throughout the entire stack
3. **Serverless-First**: AWS Lambda for backend, CloudFront for frontend
4. **Component-Based**: Modular, reusable components and services
5. **Configuration-Driven**: Environment-specific configurations
6. **Test-Driven**: Comprehensive testing at all levels

## 📱 Frontend Structure (`frontend/`)

### Directory Organization

```
frontend/
├── 📁 src/
│   ├── 📁 components/           # Reusable UI components
│   │   ├── 📁 admin/            # Admin-specific components
│   │   ├── 📁 auth/             # Authentication components
│   │   ├── 📁 common/           # Shared UI components
│   │   ├── 📁 layout/           # Layout components
│   │   ├── 📁 listing/          # Boat listing components
│   │   └── 📁 search/           # Search and filter components
│   ├── 📁 pages/               # Page components (route handlers)
│   ├── 📁 hooks/               # Custom React hooks
│   ├── 📁 services/            # API service layer
│   ├── 📁 types/               # TypeScript type definitions
│   ├── 📁 contexts/            # React context providers
│   ├── 📁 utils/               # Utility functions
│   ├── 📁 styles/              # Global styles and CSS
│   ├── 📁 config/              # Configuration files
│   └── 📁 test/                # Test utilities and setup
├── 📁 public/                  # Static assets
├── 📄 package.json             # Dependencies and scripts
├── 📄 vite.config.ts           # Vite configuration
├── 📄 tailwind.config.js       # Tailwind CSS configuration
├── 📄 tsconfig.json            # TypeScript configuration
└── 📄 .env.example             # Environment variables template
```

### Component Organization Patterns

**1. Component Structure**
```typescript
// ✅ Good: Clear component structure
import { useState, useEffect } from 'react';
import { ComponentProps } from './types';

interface Props extends ComponentProps {
  title: string;
  onAction: (id: string) => void;
}

export default function ComponentName({ title, onAction }: Props) {
  // Hooks first
  const [state, setState] = useState<string>('');
  
  // Effects
  useEffect(() => {
    // Effect logic
  }, []);

  // Event handlers
  const handleClick = () => {
    onAction('example');
  };

  // Render
  return (
    <div className="component-container">
      <h2>{title}</h2>
      <button onClick={handleClick}>Action</button>
    </div>
  );
}
```

**2. Component Categories**

- **Pages** (`pages/`): Route-level components, one per URL
- **Layout** (`components/layout/`): Header, Footer, Sidebar, etc.
- **Common** (`components/common/`): Reusable UI components (Button, Input, Modal)
- **Feature** (`components/[feature]/`): Feature-specific components
- **Admin** (`components/admin/`): Admin dashboard components

**3. File Naming Conventions**

```
✅ Good Examples:
- Header.tsx (PascalCase for components)
- useAuth.ts (camelCase with 'use' prefix for hooks)
- api.ts (camelCase for services)
- common.ts (camelCase for types/utils)

❌ Bad Examples:
- header.tsx (should be PascalCase)
- AuthHook.ts (should start with 'use')
- API.ts (should be camelCase)
```

### TypeScript Conventions

**1. Interface Definitions**
```typescript
// ✅ Good: Clear, descriptive interfaces
export interface User {
  id: string;
  email: string;
  name: string;
  role: UserRole;
  createdAt: string;
}

export interface CreateListingRequest {
  title: string;
  description: string;
  price: number;
  location: Location;
  boatDetails: BoatDetails;
}

// ✅ Good: Use enums for constants
export enum UserRole {
  USER = 'user',
  ADMIN = 'admin',
  MODERATOR = 'moderator'
}
```

**2. Component Props**
```typescript
// ✅ Good: Explicit prop interfaces
interface ListingCardProps {
  listing: Listing;
  onFavorite?: (id: string) => void;
  showOwner?: boolean;
  className?: string;
}

// ✅ Good: Extend HTML element props when needed
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
}
```

### React Patterns and Best Practices

**1. Custom Hooks**
```typescript
// ✅ Good: Custom hook pattern
export const useAuth = () => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  const login = async (email: string, password: string) => {
    // Login logic
  };

  const logout = () => {
    // Logout logic
  };

  return { user, login, logout, loading, isAuthenticated: !!user };
};
```

**2. Context Providers**
```typescript
// ✅ Good: Context pattern
interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  loading: boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const auth = useAuthState();
  
  return (
    <AuthContext.Provider value={auth}>
      {children}
    </AuthContext.Provider>
  );
};
```

**3. API Service Layer**
```typescript
// ✅ Good: Service class pattern
class ApiService {
  private getAuthHeaders() {
    const token = localStorage.getItem('authToken');
    return {
      'Content-Type': 'application/json',
      ...(token && { Authorization: `Bearer ${token}` })
    };
  }

  private async request<T>(url: string, options: RequestInit = {}): Promise<T> {
    // Request implementation
  }

  async getListings(): Promise<Listing[]> {
    return this.request<Listing[]>(endpoints.listings);
  }
}

export const api = new ApiService();
```

## 🔧 Backend Structure (`backend/`)

### Directory Organization

```
backend/
├── 📁 src/
│   ├── 📁 admin-service/        # Admin operations
│   ├── 📁 auth-service/         # User authentication
│   ├── 📁 listing/              # Boat listing CRUD
│   ├── 📁 search/               # Search functionality
│   ├── 📁 media/                # Image/video handling
│   ├── 📁 email/                # Email notifications
│   ├── 📁 shared/               # Shared utilities
│   │   ├── 📄 database.ts       # Database service
│   │   ├── 📄 auth.ts           # Authentication utilities
│   │   ├── 📄 utils.ts          # Common utilities
│   │   └── 📄 middleware.ts     # Lambda middleware
│   └── 📁 types/                # Shared type definitions
├── 📁 scripts/                  # Utility scripts
├── 📄 package.json              # Dependencies and scripts
├── 📄 tsconfig.json             # TypeScript configuration
├── 📄 build.sh                  # Build script
└── 📄 jest.config.js            # Test configuration
```

### Lambda Function Structure

**1. Function Organization**
```typescript
// ✅ Good: Lambda function structure
import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';
import { db } from '../shared/database';
import { createResponse, createErrorResponse, getUserId } from '../shared/utils';

export const handler = async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
  const requestId = event.requestContext.requestId;

  try {
    const method = event.httpMethod;
    
    // Handle CORS preflight
    if (method === 'OPTIONS') {
      return createResponse(200, {});
    }

    switch (method) {
      case 'GET':
        return await handleGet(event, requestId);
      case 'POST':
        return await handlePost(event, requestId);
      default:
        return createErrorResponse(405, 'METHOD_NOT_ALLOWED', `Method ${method} not allowed`, requestId);
    }
  } catch (error) {
    console.error('Error:', error);
    return createErrorResponse(500, 'INTERNAL_ERROR', 'Internal server error', requestId);
  }
};
```

**2. Database Service Pattern**
```typescript
// ✅ Good: Database service class
export class DatabaseService {
  private docClient: DynamoDBDocumentClient;

  constructor() {
    const client = new DynamoDBClient({ region: process.env.AWS_REGION });
    this.docClient = DynamoDBDocumentClient.from(client);
  }

  async createListing(listing: Listing): Promise<void> {
    await this.docClient.send(new PutCommand({
      TableName: LISTINGS_TABLE,
      Item: listing,
      ConditionExpression: 'attribute_not_exists(id)',
    }));
  }

  async getListing(listingId: string): Promise<Listing | null> {
    const result = await this.docClient.send(new GetCommand({
      TableName: LISTINGS_TABLE,
      Key: { id: listingId },
    }));

    return result.Item as Listing || null;
  }
}
```

**3. Utility Functions**
```typescript
// ✅ Good: Utility function patterns
export const createResponse = (statusCode: number, body: any): APIGatewayProxyResult => ({
  statusCode,
  headers: {
    'Content-Type': 'application/json',
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Headers': 'Content-Type,Authorization',
    'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
  },
  body: JSON.stringify(body)
});

export const getUserId = (event: APIGatewayProxyEvent): string => {
  const authHeader = event.headers.Authorization || event.headers.authorization;
  if (!authHeader) {
    throw new Error('User not authenticated');
  }
  
  // JWT parsing logic
  const token = authHeader.replace('Bearer ', '');
  const payload = JSON.parse(Buffer.from(token.split('.')[1], 'base64').toString());
  return payload.sub;
};
```

### Error Handling Patterns

**1. Consistent Error Responses**
```typescript
// ✅ Good: Standardized error handling
export const createErrorResponse = (
  statusCode: number,
  code: string,
  message: string,
  requestId: string,
  details?: any[]
): APIGatewayProxyResult => ({
  statusCode,
  headers: {
    'Content-Type': 'application/json',
    'Access-Control-Allow-Origin': '*'
  },
  body: JSON.stringify({
    error: {
      code,
      message,
      details,
      requestId
    }
  })
});
```

**2. Validation Patterns**
```typescript
// ✅ Good: Input validation
export const validateRequired = (obj: any, fields: string[]): void => {
  const missing = fields.filter(field => {
    const value = field.split('.').reduce((o, key) => o?.[key], obj);
    return value === undefined || value === null || value === '';
  });

  if (missing.length > 0) {
    throw new Error(`Missing required fields: ${missing.join(', ')}`);
  }
};

export const validatePrice = (price: number): boolean => {
  return typeof price === 'number' && price > 0 && price <= 10000000;
};
```

## 🏗️ Infrastructure Structure (`infrastructure/`)

### Directory Organization

```
infrastructure/
├── 📁 lib/
│   ├── 📄 boat-listing-stack.ts    # Main CDK stack
│   ├── 📄 monitoring-construct.ts  # Monitoring resources
│   └── 📄 cloudflare-tunnel-construct.ts  # Cloudflare integration
├── 📁 bin/
│   └── 📄 app.ts                   # CDK app entry point
├── 📁 test/                        # Infrastructure tests
├── 📄 package.json                 # Dependencies and scripts
├── 📄 cdk.json                     # CDK configuration
└── 📄 tsconfig.json                # TypeScript configuration
```

### CDK Patterns

**1. Stack Organization**
```typescript
// ✅ Good: CDK stack structure
export class BoatListingStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // Environment configuration
    const environment = this.node.tryGetContext('environment') || 'dev';
    
    // Create resources
    const database = this.createDatabase();
    const lambdaFunctions = this.createLambdaFunctions(database);
    const api = this.createApiGateway(lambdaFunctions);
    const frontend = this.createFrontend();
    
    // Outputs
    new CfnOutput(this, 'ApiUrl', {
      value: api.url,
      description: 'API Gateway URL'
    });
  }

  private createDatabase(): Table {
    // Database creation logic
  }
}
```

**2. Construct Patterns**
```typescript
// ✅ Good: Reusable construct
export class MonitoringConstruct extends Construct {
  public readonly dashboard: Dashboard;

  constructor(scope: Construct, id: string, props: MonitoringProps) {
    super(scope, id);

    this.dashboard = new Dashboard(this, 'Dashboard', {
      dashboardName: `${props.stackName}-monitoring`
    });

    this.createAlarms(props.functions);
  }

  private createAlarms(functions: Function[]): void {
    // Alarm creation logic
  }
}
```

## 🎨 Styling Conventions

### Tailwind CSS Patterns

**1. Component Styling**
```typescript
// ✅ Good: Consistent class patterns
const Button = ({ variant, size, children, ...props }) => {
  const baseClasses = 'font-medium rounded-lg transition-colors duration-150 focus:outline-none focus:ring-2';
  
  const variantClasses = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300 focus:ring-gray-500',
    danger: 'bg-red-600 text-white hover:bg-red-700 focus:ring-red-500'
  };

  const sizeClasses = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg'
  };

  const classes = `${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]}`;

  return (
    <button className={classes} {...props}>
      {children}
    </button>
  );
};
```

**2. Design System Classes**
```css
/* ✅ Good: Custom utility classes */
@layer components {
  .btn-primary {
    @apply bg-blue-600 text-white px-6 py-3 rounded-lg font-medium hover:bg-blue-700 transition-colors duration-150;
  }

  .card {
    @apply bg-white rounded-lg shadow-sm border border-gray-200 p-6;
  }

  .input-field {
    @apply w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent;
  }
}
```

## 📝 Naming Conventions

### General Principles

1. **Be Descriptive**: Names should clearly indicate purpose
2. **Be Consistent**: Follow established patterns throughout the codebase
3. **Use Standard Conventions**: Follow language/framework conventions
4. **Avoid Abbreviations**: Use full words unless widely understood

### Specific Conventions

**Files and Directories:**
```
✅ Good:
- components/listing/ListingCard.tsx
- hooks/useAuth.ts
- services/api.ts
- types/common.ts

❌ Bad:
- components/listing/LC.tsx
- hooks/auth.ts
- services/API.ts
- types/Types.ts
```

**Variables and Functions:**
```typescript
// ✅ Good: Descriptive camelCase
const userListings = await getUserListings(userId);
const isAuthenticated = checkAuthStatus();
const handleSubmitForm = () => { /* ... */ };

// ❌ Bad: Unclear or inconsistent
const ul = await getUL(uid);
const auth = checkAuth();
const HandleSubmit = () => { /* ... */ };
```

**Constants:**
```typescript
// ✅ Good: SCREAMING_SNAKE_CASE for constants
const API_BASE_URL = 'https://api.example.com';
const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
const DEFAULT_PAGE_SIZE = 20;

// ❌ Bad: Inconsistent casing
const apiBaseUrl = 'https://api.example.com';
const maxFileSize = 10485760;
```

**CSS Classes:**
```css
/* ✅ Good: kebab-case with BEM-like structure */
.listing-card { }
.listing-card__title { }
.listing-card__price { }
.listing-card--featured { }

/* ❌ Bad: Inconsistent or unclear */
.ListingCard { }
.lc-title { }
.card1 { }
```

## 🧪 Testing Conventions

### Test File Organization

```
src/
├── components/
│   ├── ListingCard.tsx
│   └── __tests__/
│       └── ListingCard.test.tsx
├── hooks/
│   ├── useAuth.ts
│   └── __tests__/
│       └── useAuth.test.ts
└── services/
    ├── api.ts
    └── __tests__/
        └── api.test.ts
```

### Test Naming Patterns

```typescript
// ✅ Good: Descriptive test names
describe('ListingCard', () => {
  it('should display listing title and price', () => {
    // Test implementation
  });

  it('should call onFavorite when favorite button is clicked', () => {
    // Test implementation
  });

  it('should show loading state when data is being fetched', () => {
    // Test implementation
  });
});

// ✅ Good: Test grouping
describe('useAuth hook', () => {
  describe('when user is not authenticated', () => {
    it('should return isAuthenticated as false', () => {
      // Test implementation
    });
  });

  describe('when login is called', () => {
    it('should set user data on successful login', () => {
      // Test implementation
    });

    it('should throw error on invalid credentials', () => {
      // Test implementation
    });
  });
});
```

## 📚 Documentation Standards

### Code Comments

**1. JSDoc for Public APIs**
```typescript
/**
 * Creates a new boat listing
 * @param listingData - The listing information
 * @param userId - The ID of the user creating the listing
 * @returns Promise that resolves to the created listing ID
 * @throws {ValidationError} When required fields are missing
 * @throws {AuthenticationError} When user is not authenticated
 */
export async function createListing(
  listingData: CreateListingRequest,
  userId: string
): Promise<string> {
  // Implementation
}
```

**2. Inline Comments for Complex Logic**
```typescript
// ✅ Good: Explain why, not what
// Calculate the monthly payment using the standard loan formula
// PMT = P * (r * (1 + r)^n) / ((1 + r)^n - 1)
const monthlyPayment = principal * 
  (monthlyRate * Math.pow(1 + monthlyRate, months)) / 
  (Math.pow(1 + monthlyRate, months) - 1);

// ❌ Bad: Obvious comments
// Set the user variable to null
const user = null;
```

### README Files

Each major directory should have a README.md explaining:
- Purpose and scope
- Key files and their roles
- Setup and usage instructions
- Common patterns and conventions
- Links to related documentation

## 🔧 Configuration Management

### Environment Variables

**Frontend (.env files):**
```bash
# ✅ Good: Descriptive variable names with VITE_ prefix
VITE_API_URL=https://api.example.com
VITE_ENVIRONMENT=development
VITE_DEBUG_MODE=true

# ❌ Bad: Unclear or missing prefix
API_URL=https://api.example.com
ENV=dev
DEBUG=1
```

**Backend (Lambda environment variables):**
```typescript
// ✅ Good: Validate environment variables
const LISTINGS_TABLE = process.env.LISTINGS_TABLE;
if (!LISTINGS_TABLE) {
  throw new Error('LISTINGS_TABLE environment variable is required');
}

// ✅ Good: Use constants for environment variables
const config = {
  listingsTable: process.env.LISTINGS_TABLE!,
  usersTable: process.env.USERS_TABLE!,
  region: process.env.AWS_REGION || 'us-east-1'
};
```

### Configuration Files

**TypeScript Configuration:**
```json
// tsconfig.json - Strict configuration
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

## 🚀 Performance Considerations

### Code Splitting

```typescript
// ✅ Good: Lazy load heavy components
const AdminDashboard = lazy(() => import('./pages/admin/AdminDashboard'));
const Analytics = lazy(() => import('./pages/admin/Analytics'));

// ✅ Good: Route-based code splitting
<Route 
  path="/admin/analytics" 
  element={
    <Suspense fallback={<LoadingSpinner />}>
      <Analytics />
    </Suspense>
  } 
/>
```

### Bundle Optimization

```typescript
// ✅ Good: Tree-shakable imports
import { debounce } from 'lodash/debounce';
import { format } from 'date-fns/format';

// ❌ Bad: Full library imports
import _ from 'lodash';
import * as dateFns from 'date-fns';
```

## 🔍 Code Quality Tools

### ESLint Configuration

```json
{
  "extends": [
    "@typescript-eslint/recommended",
    "react-hooks/recommended"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

### Prettier Configuration

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2
}
```

## 📋 Code Review Checklist

### Before Submitting

- [ ] Code follows naming conventions
- [ ] TypeScript types are properly defined
- [ ] Error handling is implemented
- [ ] Tests are written and passing
- [ ] Documentation is updated
- [ ] No console.log statements in production code
- [ ] Performance implications considered
- [ ] Security best practices followed

### During Review

- [ ] Code is readable and maintainable
- [ ] Logic is sound and efficient
- [ ] Edge cases are handled
- [ ] Consistent with existing patterns
- [ ] No code duplication
- [ ] Proper error messages
- [ ] Accessibility considerations (frontend)

## 🎯 Best Practices Summary

### Do's ✅

- Use TypeScript strictly with proper typing
- Follow established naming conventions
- Write descriptive commit messages
- Keep functions small and focused
- Use consistent error handling patterns
- Document complex logic
- Write tests for critical functionality
- Use environment variables for configuration
- Follow the principle of least privilege
- Keep dependencies up to date

### Don'ts ❌

- Don't use `any` type unless absolutely necessary
- Don't commit sensitive information
- Don't ignore TypeScript errors
- Don't write overly complex functions
- Don't skip error handling
- Don't hardcode configuration values
- Don't ignore accessibility requirements
- Don't write tests that don't add value
- Don't break existing APIs without versioning
- Don't optimize prematurely

## 📚 Additional Resources

### Internal Documentation
- [Getting Started Guide](getting-started.md) - Setup and first contribution
- [Local Development Setup](local-setup.md) - Environment configuration
- [Development Workflow](development-workflow.md) - Git and collaboration

### External Resources
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [React Best Practices](https://react.dev/learn)
- [AWS CDK Best Practices](https://docs.aws.amazon.com/cdk/v2/guide/best-practices.html)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)

### Tools and Extensions
- [VS Code TypeScript Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-next)
- [ESLint Extension](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- [Prettier Extension](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
- [Tailwind CSS IntelliSense](https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss)

---

Following these conventions ensures that our codebase remains maintainable, scalable, and accessible to all team members. When in doubt, prioritize clarity and consistency over cleverness.

Happy coding! 🚤