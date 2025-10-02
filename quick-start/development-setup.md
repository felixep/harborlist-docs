# Development Setup Guide

## 🚀 Quick Start

### Prerequisites

Before starting development, ensure you have the following installed:

- **Node.js** v18+ ([Download](https://nodejs.org/)) (development environment) (development environment) (development environment)
- **npm** v8+ (comes with Node.js)
- **Git** ([Download](https://git-scm.com/)) (development environment) (development environment) (development environment)
- **AWS CLI** ([Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)) (development environment) (development environment) (development environment)
- **Code Editor** (VS Code recommended)

### Initial Setup

```bash
# Clone the repository
git clone <your-repository-url>
cd boat_listing_project

# Install frontend dependencies
cd frontend
npm install

# Install backend dependencies
cd ../backend
npm install

# Install infrastructure dependencies
cd ../infrastructure
npm install

# Return to project root
cd ..
```

### Start Development Environment

```bash
# Start frontend development server
cd frontend
npm run dev
# Frontend will be available at http://localhost:3000

# In a new terminal, build backend (for API integration)
cd backend
npm run build
npm run watch  # Optional: watch for changes
```

## 📁 Project Structure

```
boat_listing_project/
├── 📚 docs/                    # Documentation
│   ├── quick-start/            # Getting started guides
│   ├── architecture/           # System architecture
│   ├── operations/             # Operational procedures
│   ├── administration/         # Admin documentation
│   ├── development/            # Development guides
│   └── reference/              # Reference materials
├── 🎨 frontend/                # React application
│   ├── src/
│   │   ├── components/         # React components
│   │   ├── pages/              # Page components
│   │   ├── services/           # API services
│   │   ├── hooks/              # Custom React hooks
│   │   └── types/              # TypeScript types
│   ├── public/                 # Static assets
│   └── dist/                   # Build output
├── ⚙️ backend/                 # Lambda functions
│   ├── src/
│   │   ├── admin-service/      # Admin functionality
│   │   ├── auth-service/       # Authentication
│   │   ├── listing-service/    # Listing CRUD
│   │   ├── search/             # Search functionality
│   │   ├── media/              # Media handling
│   │   ├── email/              # Email services
│   │   └── shared/             # Shared utilities
│   └── dist/                   # Build output
├── 🏗️ infrastructure/          # AWS CDK templates
│   ├── lib/                    # CDK stack definitions
│   ├── bin/                    # CDK app entry point
│   └── test/                   # Infrastructure tests
└── 📋 README.md               # Project overview
```

## 🎨 Frontend Development

### Available Scripts

```bash
# Development server with hot reload
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Type checking
npm run type-check

# Linting and formatting
npm run lint
npm run lint:fix
```

### Development Workflow

#### 1. Component Development

Components are organized by feature:

```bash
frontend/src/components/
├── common/          # Shared components (Header, LoadingSpinner, etc.)
├── auth/           # Authentication components
├── listing/        # Boat listing components
├── search/         # Search functionality
└── admin/          # Admin panel components
```

#### 2. Adding New Components

```typescript
// Example: Create new component
// frontend/src/components/common/NewComponent.tsx

import React from 'react';

interface NewComponentProps {
  title: string;
  description?: string;
}

export default function NewComponent({ title, description }: NewComponentProps) {
  return (
    <div className="bg-white rounded-lg p-4 shadow-md">
      <h2 className="text-xl font-semibold text-gray-900">{title}</h2>
      {description && (
        <p className="mt-2 text-gray-600">{description}</p>
      )}
    </div>
  );
}
```

#### 3. Adding New Pages

```typescript
// 1. Create page component
// frontend/src/pages/NewPage.tsx

import React from 'react';

export default function NewPage() {
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold">New Page</h1>
      {/* Page content */}
    </div>
  );
}

// 2. Add route to App.tsx
import NewPage from './pages/NewPage';

// In Routes component:
<Route path="/new-page" element={<NewPage />} />
```

#### 4. API Integration

```typescript
// Add new API functions to services/
export async function createListing(listingData: CreateListingRequest): Promise<Listing> {
  return apiRequest('/listings', {
    method: 'POST',
    body: JSON.stringify(listingData),
  });
}

// Use in components with React Query
import { useMutation, useQueryClient } from '@tanstack/react-query';

const queryClient = useQueryClient();

const createListingMutation = useMutation({
  mutationFn: createListing,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['listings'] });
  },
});
```

### Styling Guidelines

The project uses **Tailwind CSS** for styling. Here are common patterns:

```css
/* Layout Classes */
.container-responsive  /* max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 */
.grid-responsive      /* grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 */

/* Component Classes */
.btn-primary         /* bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded */
.btn-secondary       /* bg-gray-200 hover:bg-gray-300 text-gray-900 px-4 py-2 rounded */
.form-input          /* border border-gray-300 rounded-md px-3 py-2 focus:ring-2 focus:ring-blue-500 */
.card               /* bg-white rounded-lg shadow-md p-6 */
```

#### Common Component Patterns

```typescript
// Loading State Pattern
{isLoading ? (
  <div className="animate-pulse bg-gray-200 rounded h-48" />
) : (
  <ActualContent />
)}

// Error State Pattern
{error ? (
  <div className="bg-red-50 border border-red-200 text-red-700 px-4 py-3 rounded">
    <p className="font-medium">Error</p>
    <p className="text-sm">{error.message}</p>
  </div>
) : (
  <Content />
)}

// Empty State Pattern
{items.length === 0 ? (
  <div className="text-center py-12">
    <div className="text-gray-400 mb-4">
      <svg className="mx-auto h-12 w-12" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        {/* Icon SVG */}
      </svg>
    </div>
    <p className="text-gray-500">No items found</p>
  </div>
) : (
  <ItemsList items={items} />
)}
```

## 🔧 Backend Development

### Available Scripts

```bash
# Build TypeScript to JavaScript
npm run build

# Watch for changes and rebuild
npm run watch

# Run unit tests
npm test

# Lint TypeScript code
npm run lint

# Clean build directory
npm run clean

# Build deployment packages
./build.sh
```

### Development Workflow

#### 1. Lambda Function Structure

```bash
backend/src/
├── admin-service/       # Admin panel functionality
├── auth-service/        # User authentication
├── listing-service/     # Listing CRUD operations
├── search/              # Advanced search
├── media/               # Image/video processing
├── email/               # Contact and notifications
└── shared/              # Common utilities
    ├── auth.ts          # Authentication helpers
    ├── database.ts      # DynamoDB operations
    ├── middleware.ts    # Common middleware
    ├── test-utils.ts    # Testing utilities
    └── utils.ts         # General utilities
```

#### 2. Creating New Lambda Functions

```typescript
// Example: backend/src/newservice/index.ts
import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';
import { createResponse, createErrorResponse } from '../shared/utils';
import { validateJWT } from '../shared/auth';

export const handler = async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
  const requestId = event.requestContext.requestId;

  try {
    // Authentication for protected endpoints
    const userId = validateJWT(event);
    
    // Parse request body
    const body = JSON.parse(event.body || '{}');
    
    // Validate input
    if (!body.requiredField) {
      return createErrorResponse(400, 'VALIDATION_ERROR', 'Required field missing', requestId);
    }

    // Business logic
    const result = await processRequest(body);
    
    return createResponse(200, result);
  } catch (error) {
    console.error('Error:', error);
    return createErrorResponse(500, 'INTERNAL_ERROR', 'Operation failed', requestId);
  }
};
```

#### 3. Database Operations

```typescript
// Use shared database service
import { db } from '../shared/database';

// Create record
const newListing = await db.createListing({
  title: 'Beautiful Sailboat',
  price: 50000,
  ownerId: userId,
  // ... other fields
});

// Get record
const listing = await db.getListing(listingId);

// Update record
await db.updateListing(listingId, {
  price: 45000,
  updatedAt: Date.now()
});

// Query with filters
const listings = await db.queryListings({
  location: 'Florida',
  priceRange: { min: 20000, max: 100000 }
});
```

#### 4. Testing Lambda Functions Locally

```bash
# Install SAM CLI for local testing
npm install -g @aws-sam/cli

# Create test events
mkdir test-events
echo '{"body": "{\"title\":\"Test Listing\"}"}' > test-events/create-listing.json

# Test specific function
sam local invoke ListingFunction --event test-events/create-listing.json

# Start local API Gateway
sam local start-api --template template.yaml
# API available at http://localhost:3000
```

### Backend Architecture Patterns

#### Error Handling Pattern

```typescript
try {
  const result = await someOperation();
  return createResponse(200, result);
} catch (error) {
  console.error('Operation failed:', error);
  
  if (error instanceof ValidationError) {
    return createErrorResponse(400, 'VALIDATION_ERROR', error.message, requestId);
  }
  
  if (error instanceof AuthenticationError) {
    return createErrorResponse(401, 'UNAUTHORIZED', 'Authentication failed', requestId);
  }
  
  return createErrorResponse(500, 'INTERNAL_ERROR', 'Operation failed', requestId);
}
```

#### Authentication Pattern

```typescript
// For protected endpoints
import { validateJWT } from '../shared/auth';

const userId = validateJWT(event);
if (!userId) {
  return createErrorResponse(401, 'UNAUTHORIZED', 'Invalid token', requestId);
}

// Check user permissions
const user = await db.getUser(userId);
if (!user || user.status !== 'active') {
  return createErrorResponse(403, 'FORBIDDEN', 'User not authorized', requestId);
}
```

## ⚙️ Infrastructure Development

### AWS CDK Setup

```bash
# Install CDK CLI globally
npm install -g aws-cdk

# Bootstrap CDK (first time only)
cd infrastructure
cdk bootstrap

# Deploy development environment
cdk deploy --context environment=dev
```

### Environment Configuration

The infrastructure supports multiple environments:

```bash
# Development (without custom domains)
cdk deploy --context environment=dev --context useCustomDomains=false

# Staging (with custom domains)
cdk deploy --context environment=staging --context useCustomDomains=true

# Production (with custom domains)
cdk deploy --context environment=prod --context useCustomDomains=true
```

### Local Infrastructure Testing

```bash
# Validate CDK templates
cdk synth

# Check for differences
cdk diff

# Run infrastructure tests
npm test
```

## 🧪 Testing

### Frontend Testing

```bash
# Run unit tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage

# Run E2E tests (if configured)
npm run test:e2e
```

### Backend Testing

```bash
# Run unit tests
npm test

# Run integration tests
npm run test:integration

# Run tests with coverage
npm run test:coverage
```

### Test Structure

```bash
# Frontend tests
frontend/src/
├── components/
│   └── __tests__/           # Component tests
├── hooks/
│   └── __tests__/           # Hook tests
└── services/
    └── __tests__/           # Service tests

# Backend tests
backend/src/
├── listing-service/
│   └── __tests__/           # Service tests
└── shared/
    └── __tests__/           # Utility tests
```

## 🔍 Debugging & Development Tools

### Browser DevTools

- **React DevTools**: Component inspection and state debugging
- **TanStack Query DevTools**: API state and cache debugging
- **Network Tab**: Monitor API requests and responses
- **Console**: View application logs and errors

### VS Code Extensions (Recommended)

Create `.vscode/extensions.json`:

```json
{
  "recommendations": [
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "formulahendry.auto-rename-tag",
    "christian-kohler.path-intellisense",
    "ms-vscode.vscode-json",
    "amazonwebservices.aws-toolkit-vscode"
  ]
}
```

### Environment Variables

Create environment files for different configurations:

```bash
# frontend/.env.local (for local development)
VITE_API_URL=http://localhost:3001
VITE_ENVIRONMENT=development
VITE_DEBUG=true

# frontend/.env.development
VITE_API_URL=https://api-dev.harborlist.com (development environment) (development environment) (development environment)
VITE_ENVIRONMENT=development

# frontend/.env.production
VITE_API_URL=https://api.harborlist.com (development environment) (development environment) (development environment)
VITE_ENVIRONMENT=production
```

## 🚨 Common Issues & Solutions

### Development Issues

#### Port Already in Use

```bash
# Find process using port 3000
lsof -ti:3000

# Kill process
lsof -ti:3000 | xargs kill -9

# Or use different port
npm run dev -- --port 3001
```

#### TypeScript Errors

```bash
# Clear TypeScript cache
rm -rf node_modules/.cache
rm -rf .tsbuildinfo

# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install

# Run type check
npm run type-check
```

#### Build Issues

```bash
# Clear all build artifacts
npm run clean

# Rebuild everything
npm run build

# For frontend build issues
cd frontend
rm -rf dist node_modules/.vite
npm run build
```

### API Integration Issues

#### CORS Errors

```typescript
// Ensure API service includes proper headers
const response = await fetch(url, {
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`,
  },
  mode: 'cors',
});
```

#### Authentication Errors

```typescript
// Debug authentication issues
const token = localStorage.getItem('authToken');
console.log('Current token:', token);

// Check token expiration
if (token) {
  const payload = JSON.parse(atob(token.split('.')[1]));
  console.log('Token expires:', new Date(payload.exp * 1000));
}
```

### AWS/Infrastructure Issues

#### CDK Deployment Errors

```bash
# Check AWS credentials
aws sts get-caller-identity

# Verify CDK bootstrap
cdk bootstrap --show-template

# Check for stack drift
cdk diff
```

#### Lambda Function Errors

```bash
# View CloudWatch logs
aws logs filter-log-events \
  --log-group-name "/aws/lambda/BoatListingStack-ListingFunction" \
  --start-time $(date -d '1 hour ago' +%s)000

# Test function locally
sam local invoke ListingFunction --event test-events/sample.json
```

## 📊 Performance Monitoring

### Frontend Performance

- **Lighthouse Scores**: Run regular performance audits
- **Core Web Vitals**: Monitor LCP, FID, CLS metrics
- **Bundle Analysis**: Use `npm run build -- --analyze` to check bundle size
- **React DevTools Profiler**: Profile component render performance

### Backend Performance

- **CloudWatch Metrics**: Monitor Lambda duration, memory usage, error rates
- **X-Ray Tracing**: Enable for detailed request tracing
- **DynamoDB Metrics**: Monitor read/write capacity and throttling

### Development Performance Tips

```bash
# Frontend optimization
npm run build -- --analyze  # Analyze bundle size
npm run lighthouse          # Run Lighthouse audit

# Backend optimization
npm run test:load           # Run load tests
npm run profile            # Profile function performance
```

## 🔒 Security Best Practices

### Code Security

- **Input Validation**: Always validate and sanitize user inputs
- **Authentication**: Use JWT tokens with proper expiration
- **Authorization**: Implement role-based access control
- **Secrets Management**: Never commit secrets to version control

### Development Security

```bash
# Audit dependencies for vulnerabilities
npm audit

# Fix vulnerabilities
npm audit fix

# Check for security issues
npm run security-check
```

### Environment Security

```bash
# Use environment variables for sensitive data
export JWT_SECRET="your-secret-key"
export DATABASE_URL="your-database-url"

# Never commit .env files with real secrets
echo ".env.local" >> .gitignore
```

## 📈 Development Best Practices

### Code Quality

- **TypeScript**: Use strict mode for better type safety
- **ESLint**: Maintain consistent code style
- **Prettier**: Automatic code formatting
- **Git Hooks**: Pre-commit validation (husky + lint-staged)

### Component Guidelines

- **Single Responsibility**: One purpose per component
- **Props Interface**: Always define TypeScript interfaces
- **Error Boundaries**: Wrap components in error handling
- **Accessibility**: Include ARIA labels and keyboard navigation

### State Management

- **Server State**: Use TanStack Query for API data
- **Global State**: Use React Context sparingly
- **Local State**: Use useState for component-specific state
- **URL State**: Use React Router for navigation state

## 🚀 Development Workflow

### Daily Development Process

1. **Pull Latest Changes**
   ```bash
   git pull origin main
   npm install  # Update dependencies if needed
   ```

2. **Create Feature Branch**
   ```bash
   git checkout -b feature/new-feature-name
   ```

3. **Start Development Environment**
   ```bash
   # Terminal 1: Frontend
   cd frontend && npm run dev
   
   # Terminal 2: Backend (if needed)
   cd backend && npm run watch
   ```

4. **Make Changes and Test**
   ```bash
   # Run tests frequently
   npm test
   
   # Check types
   npm run type-check
   
   # Lint code
   npm run lint
   ```

5. **Commit and Push**
   ```bash
   git add .
   git commit -m "feat: add new feature"
   git push origin feature/new-feature-name
   ```

### Code Review Process

1. Create pull request with clear description
2. Ensure all tests pass
3. Request review from team members
4. Address feedback and update PR
5. Merge after approval

### Release Process

1. **Create Release Branch**
   ```bash
   git checkout -b release/v1.2.0
   ```

2. **Update Version Numbers**
   ```bash
   npm version patch  # or minor/major
   ```

3. **Build and Test**
   ```bash
   npm run build
   npm run test:all
   ```

4. **Deploy to Staging**
   ```bash
   cd infrastructure
   cdk deploy --context environment=staging
   ```

5. **Merge to Main and Tag**
   ```bash
   git checkout main
   git merge release/v1.2.0
   git tag v1.2.0
   git push origin main --tags
   ```

This development setup provides everything needed to efficiently develop, test, and deploy the boat listing platform.