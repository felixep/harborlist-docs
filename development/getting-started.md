# Getting Started Guide

Welcome to the MarineMarket boat listing platform! This comprehensive guide will help you set up your development environment and make your first contribution to the project.

## 🎯 What You'll Learn

By the end of this guide, you'll be able to:
- Set up a complete local development environment
- Understand the project structure and architecture
- Run the application locally for development
- Make your first code contribution
- Run tests and validate your changes

## 📋 Prerequisites

Before you begin, ensure you have the following installed on your system:

### Required Software

**Node.js (v18 or higher)**
```bash
# Check your Node.js version
node --version

# If you need to install or upgrade Node.js
# Visit https://nodejs.org/ or use a version manager like nvm
```

**npm (comes with Node.js)**
```bash
# Check npm version
npm --version
```

**Git**
```bash
# Check Git version
git --version

# Configure Git if not already done
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**AWS CLI (for deployment)**
```bash
# Check AWS CLI version
aws --version

# Install if needed: https://aws.amazon.com/cli/
# Configure with your credentials
aws configure
```

### Recommended Software

**Visual Studio Code** - Recommended IDE with excellent TypeScript support
- Install the following extensions:
  - TypeScript and JavaScript Language Features
  - ESLint
  - Prettier
  - Tailwind CSS IntelliSense
  - AWS Toolkit

**Docker** (optional, for local services)
```bash
# Check Docker version
docker --version
```

## 🚀 Quick Start (5 Minutes)

### 1. Clone the Repository

```bash
# Clone the repository
git clone <repository-url>
cd boat_listing_project

# Verify the project structure
ls -la
```

You should see these main directories:
- `frontend/` - React application
- `backend/` - Lambda functions
- `infrastructure/` - AWS CDK code
- `docs/` - Documentation
- `config/` - Configuration files

### 2. Install Dependencies

```bash
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

### 3. Set Up Environment Variables

```bash
# Copy environment template for frontend
cd frontend
cp .env.example .env.local

# Edit the environment file with your settings
# For local development, you can use the default values
```

**Frontend Environment Variables** (`.env.local`):
```bash
# API endpoint (will be set after infrastructure deployment)
VITE_API_URL=https://your-api-gateway-url.amazonaws.com/prod
VITE_ENVIRONMENT=development
```

### 4. Start Development Server

```bash
# Start the frontend development server
cd frontend
npm run dev
```

The application will be available at `http://localhost:5173`

## 🏗️ Project Structure Deep Dive

Understanding the project structure is crucial for effective development:

```
boat_listing_project/
├── 📁 frontend/                    # React TypeScript application
│   ├── 📁 src/
│   │   ├── 📁 components/          # Reusable React components
│   │   │   ├── 📁 admin/           # Admin dashboard components
│   │   │   ├── 📁 auth/            # Authentication components
│   │   │   ├── 📁 common/          # Shared UI components
│   │   │   ├── 📁 layout/          # Layout components
│   │   │   ├── 📁 listing/         # Boat listing components
│   │   │   └── 📁 search/          # Search and filter components
│   │   ├── 📁 pages/               # Page components (routes)
│   │   ├── 📁 hooks/               # Custom React hooks
│   │   ├── 📁 services/            # API service functions
│   │   ├── 📁 types/               # TypeScript type definitions
│   │   ├── 📁 contexts/            # React context providers
│   │   └── 📁 utils/               # Utility functions
│   ├── 📄 package.json             # Frontend dependencies and scripts
│   ├── 📄 vite.config.ts           # Vite build configuration
│   ├── 📄 tailwind.config.js       # Tailwind CSS configuration
│   └── 📄 tsconfig.json            # TypeScript configuration
├── 📁 backend/                     # AWS Lambda functions
│   ├── 📁 src/
│   │   ├── 📁 admin-service/       # Admin operations
│   │   ├── 📁 auth-service/        # User authentication
│   │   ├── 📁 listing/             # Boat listing CRUD
│   │   ├── 📁 search/              # Search functionality
│   │   ├── 📁 media/               # Image/video handling
│   │   ├── 📁 email/               # Email notifications
│   │   ├── 📁 shared/              # Shared utilities
│   │   └── 📁 types/               # Shared type definitions
│   ├── 📄 package.json             # Backend dependencies and scripts
│   ├── 📄 build.sh                 # Build script for Lambda packages
│   └── 📄 tsconfig.json            # TypeScript configuration
├── 📁 infrastructure/              # AWS CDK infrastructure code
│   ├── 📁 lib/                     # CDK stack definitions
│   ├── 📄 package.json             # Infrastructure dependencies
│   └── 📄 cdk.json                 # CDK configuration
├── 📁 docs/                        # Project documentation
├── 📁 config/                      # Configuration files
└── 📄 README.md                    # Project overview
```

### Key Files to Understand

**Frontend Key Files:**
- `frontend/src/App.tsx` - Main application component with routing
- `frontend/src/main.tsx` - Application entry point
- `frontend/src/services/api.ts` - API service layer
- `frontend/package.json` - Dependencies and build scripts

**Backend Key Files:**
- `backend/src/shared/database.ts` - Database connection utilities
- `backend/src/shared/auth.ts` - Authentication middleware
- `backend/src/types/common.ts` - Shared TypeScript interfaces
- `backend/package.json` - Dependencies and build scripts

**Infrastructure Key Files:**
- `infrastructure/lib/boat-listing-stack.ts` - Main CDK stack
- `infrastructure/package.json` - CDK dependencies and deployment scripts

## 🔧 Development Workflow

### 1. Frontend Development

**Start the development server:**
```bash
cd frontend
npm run dev
```

**Key development commands:**
```bash
# Type checking
npm run type-check

# Linting
npm run lint

# Run tests
npm run test

# Build for production
npm run build
```

**Frontend Architecture:**
- **React 18** with TypeScript for type safety
- **Vite** for fast development and building
- **Tailwind CSS** for styling
- **TanStack Query** for server state management
- **React Router** for client-side routing

### 2. Backend Development

**Build the backend:**
```bash
cd backend
npm run build
```

**Key development commands:**
```bash
# Watch for changes during development
npm run watch

# Run tests
npm test

# Build Lambda packages for deployment
./build.sh

# Create admin user (after deployment)
npm run create-admin
```

**Backend Architecture:**
- **AWS Lambda** functions for serverless compute
- **DynamoDB** for data storage
- **TypeScript** for type safety
- **JWT** for authentication
- **Sharp** for image processing

### 3. Infrastructure Development

**Deploy infrastructure:**
```bash
cd infrastructure

# Validate CDK code
npm run build

# Preview changes
npm run diff:dev

# Deploy to development environment
npm run deploy:dev
```

**Key infrastructure commands:**
```bash
# Synthesize CloudFormation templates
npm run synth:dev

# Deploy with custom domains (production)
npm run deploy:prod

# Destroy infrastructure (cleanup)
cdk destroy
```

## 🧪 Testing Your Setup

### 1. Verify Frontend

```bash
cd frontend

# Run type checking
npm run type-check

# Run linting
npm run lint

# Run tests
npm run test:run

# Build the application
npm run build
```

### 2. Verify Backend

```bash
cd backend

# Build TypeScript
npm run build

# Run tests
npm test

# Create deployment packages
./build.sh
```

### 3. Verify Infrastructure

```bash
cd infrastructure

# Build CDK code
npm run build

# Run infrastructure tests
npm test

# Synthesize templates (no deployment)
npm run synth:dev
```

## 🚀 Your First Contribution

### 1. Create a Feature Branch

```bash
# Create and switch to a new feature branch
git checkout -b feature/your-feature-name

# Example: git checkout -b feature/improve-search-filters
```

### 2. Make Your Changes

Start with a small change to get familiar with the codebase:

**Example: Add a new boat type to the search filters**

1. **Frontend Change** (`frontend/src/components/search/SearchFilters.tsx`):
```typescript
// Add a new boat type to the options
const boatTypes = [
  'Sailboat',
  'Motorboat', 
  'Yacht',
  'Fishing Boat',
  'Pontoon',
  'Jet Ski', // <- Add this new option
];
```

2. **Backend Change** (`backend/src/types/common.ts`):
```typescript
// Update the BoatType enum
export type BoatType = 
  | 'Sailboat'
  | 'Motorboat'
  | 'Yacht'
  | 'Fishing Boat'
  | 'Pontoon'
  | 'Jet Ski'; // <- Add this new type
```

### 3. Test Your Changes

```bash
# Test frontend changes
cd frontend
npm run test:run
npm run type-check

# Test backend changes
cd ../backend
npm test
npm run build
```

### 4. Commit Your Changes

```bash
# Add your changes
git add .

# Commit with a descriptive message
git commit -m "feat: add Jet Ski as a new boat type option

- Added Jet Ski to search filter options
- Updated TypeScript types to include new boat type
- Tested frontend and backend changes"
```

### 5. Push and Create Pull Request

```bash
# Push your branch
git push origin feature/your-feature-name

# Create a pull request on GitHub
# Include a description of your changes and any testing notes
```

## 🔍 Understanding the Data Flow

### Frontend to Backend Communication

1. **User Action** → Component event handler
2. **API Call** → Service function (`frontend/src/services/api.ts`)
3. **HTTP Request** → AWS API Gateway
4. **Lambda Function** → Backend service (`backend/src/`)
5. **Database Operation** → DynamoDB
6. **Response** → Frontend component
7. **UI Update** → React state/context update

### Example: Creating a New Listing

```typescript
// 1. User submits form (frontend/src/pages/CreateListing.tsx)
const handleSubmit = async (listingData: CreateListingRequest) => {
  // 2. Call API service
  const result = await createListing(listingData);
  // 3. Update UI based on result
  if (result.success) {
    navigate('/listings');
  }
};

// 4. API service makes HTTP request (frontend/src/services/api.ts)
export const createListing = async (data: CreateListingRequest) => {
  const response = await fetch(`${API_URL}/listings`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${getAuthToken()}`
    },
    body: JSON.stringify(data)
  });
  return response.json();
};

// 5. Lambda function processes request (backend/src/listing/index.ts)
export const handler = async (event: APIGatewayProxyEvent) => {
  // Validate authentication
  const user = await validateToken(event.headers.Authorization);
  
  // Validate request data
  const listingData = JSON.parse(event.body);
  
  // Save to database
  const listing = await createListingInDB(listingData, user.userId);
  
  // Return response
  return {
    statusCode: 201,
    body: JSON.stringify({ listing })
  };
};
```

## 🛠️ Development Tools and Tips

### VS Code Configuration

Create `.vscode/settings.json` in your project root:
```json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "files.associations": {
    "*.css": "tailwindcss"
  }
}
```

### Useful npm Scripts Reference

**Frontend Scripts** (`frontend/package.json`):
```json
{
  "dev": "vite",                    // Start development server
  "build": "tsc && vite build",     // Build for production
  "build:dev": "vite build --mode dev",     // Build for development
  "preview": "vite preview",        // Preview production build
  "lint": "eslint . --ext ts,tsx",  // Run ESLint
  "type-check": "tsc --noEmit",     // Type checking only
  "test": "vitest",                 // Run tests in watch mode
  "test:run": "vitest run"          // Run tests once
}
```

**Backend Scripts** (`backend/package.json`):
```json
{
  "build": "tsc && npm run package",        // Build and package
  "package": "cd dist && ...",              // Create Lambda ZIP files
  "watch": "tsc -w",                        // Watch for changes
  "test": "jest",                           // Run tests
  "lint": "eslint src --ext .ts",           // Run ESLint
  "create-admin": "ts-node scripts/create-admin-user.ts"  // Create admin user
}
```

**Infrastructure Scripts** (`infrastructure/package.json`):
```json
{
  "build": "tsc",                           // Compile TypeScript
  "synth": "cdk synth",                     // Generate CloudFormation
  "deploy": "cdk deploy",                   // Deploy infrastructure
  "deploy:dev": "cdk deploy -c environment=dev",    // Deploy to dev
  "diff": "cdk diff",                       // Show infrastructure changes
  "test": "jest"                            // Run infrastructure tests
}
```

### Debugging Tips

**Frontend Debugging:**
- Use React Developer Tools browser extension
- Use browser DevTools for network requests
- Check console for TypeScript errors
- Use `console.log()` for quick debugging

**Backend Debugging:**
- Check CloudWatch logs for Lambda functions
- Use `console.log()` in Lambda functions
- Test API endpoints with Postman or curl
- Use AWS CLI to check DynamoDB data

**Common Issues and Solutions:**

1. **CORS Errors**: Check API Gateway CORS configuration
2. **Authentication Errors**: Verify JWT token format and expiration
3. **Build Errors**: Check TypeScript types and imports
4. **Deployment Errors**: Verify AWS credentials and permissions

## 📚 Next Steps

Now that you have your development environment set up:

1. **Explore the Codebase**: 
   - Read through the main components and services
   - Understand the data models and API endpoints
   - Review the existing tests

2. **Read Additional Documentation**:
   - [Local Development Setup](local-setup.md) - Detailed environment configuration
   - [Code Structure and Conventions](code-structure.md) - Coding standards
   - [Development Workflow](development-workflow.md) - Git workflow and processes

3. **Make Your First Contribution**:
   - Pick a small issue or improvement
   - Follow the contribution workflow
   - Ask for help when needed

4. **Join the Community**:
   - Participate in code reviews
   - Share knowledge with team members
   - Contribute to documentation improvements

## 🆘 Getting Help

If you encounter issues during setup:

1. **Check the Documentation**: Review the relevant documentation sections
2. **Search Issues**: Look for similar issues in the project's issue tracker
3. **Ask for Help**: Reach out to team members or create a new issue
4. **Community Resources**: Check Stack Overflow and relevant community forums

## 🎉 Welcome to the Team!

Congratulations! You now have a fully functional development environment for the MarineMarket platform. You're ready to start contributing to this exciting project.

Remember:
- Start small and gradually take on larger features
- Don't hesitate to ask questions
- Write tests for your code
- Follow the established coding standards
- Have fun building something amazing!

Happy coding! 🚤