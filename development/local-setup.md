# Local Development Setup

This guide provides detailed instructions for setting up a complete local development environment for the MarineMarket boat listing platform. Follow these steps to configure your development environment for optimal productivity.

## 🎯 Overview

The MarineMarket platform uses a serverless architecture that can be developed locally in two ways:

1. **Hybrid Development** (Recommended): Frontend runs locally, backend deployed to AWS
2. **Full Local Development**: All services running locally with mock AWS services

## 📋 Prerequisites Verification

Before proceeding, verify you have all required software installed:

### System Requirements

**Operating System Support:**
- macOS 10.15+ (Catalina or later)
- Windows 10/11 with WSL2
- Ubuntu 18.04+ or equivalent Linux distribution

**Hardware Requirements:**
- 8GB RAM minimum (16GB recommended)
- 10GB free disk space
- Stable internet connection for AWS services

### Required Software Installation

**1. Node.js (v18 or higher)**
```bash
# Check current version
node --version

# Install using Node Version Manager (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 18
nvm use 18
nvm alias default 18

# Verify installation
node --version  # Should show v18.x.x
npm --version   # Should show 9.x.x or higher
```

**2. Git Configuration**
```bash
# Check Git version (2.20+ recommended)
git --version

# Configure Git (if not already done)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**3. AWS CLI (v2.0+)**
```bash
# Install AWS CLI v2
# macOS
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Windows (PowerShell)
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi

# Verify installation
aws --version  # Should show aws-cli/2.x.x
```

**4. AWS CDK CLI**
```bash
# Install CDK CLI globally
npm install -g aws-cdk

# Verify installation
cdk --version  # Should show 2.100.0 or higher
```

**5. Code Editor Setup (VS Code Recommended)**
```bash
# Install VS Code extensions for optimal development experience
code --install-extension ms-vscode.vscode-typescript-next
code --install-extension bradlc.vscode-tailwindcss
code --install-extension esbenp.prettier-vscode
code --install-extension ms-vscode.vscode-json
code --install-extension amazonwebservices.aws-toolkit-vscode
```

## 🚀 Project Setup

### 1. Repository Clone and Initial Setup
```bash
# Navigate to your development directory
cd ~/Documents/Projects  # or your preferred location

# Clone the repository (replace with actual repository URL)
git clone <repository-url> boat_listing_project
cd boat_listing_project

# Verify project structure
ls -la
# Should show: backend/, frontend/, infrastructure/, docs/, etc.
```

### 2. Install Dependencies

**Frontend Dependencies:**
```bash
cd frontend
npm install

# Verify installation
npm list --depth=0
# Should show React, TypeScript, Tailwind CSS, and other dependencies
```

**Available Scripts (from [`frontend/package.json`](frontend/src/../../frontend/package.json)):**
```bash
npm run dev              # Start development server with hot reload
npm run build            # Production build with TypeScript compilation
npm run build:dev        # Development build
npm run build:staging    # Staging build
npm run build:prod       # Production build
npm run preview          # Preview production build locally
npm run lint             # ESLint code quality check
npm run type-check       # TypeScript type checking
npm run test             # Run unit tests with Vitest
npm run test:run         # Run tests once (CI mode)
```

**Backend Dependencies:**
```bash
cd ../backend
npm install

# Verify installation
npm list --depth=0
# Should show AWS SDK, TypeScript, and Lambda dependencies
```

**Available Scripts (from [`backend/package.json`](frontend/src/../../backend/package.json)):**
```bash
npm run build            # TypeScript compilation + Lambda packaging
npm run package          # Create Lambda deployment packages
npm run watch            # TypeScript compilation in watch mode
npm run test             # Run Jest unit tests
npm run lint             # ESLint code quality check
npm run clean            # Remove build artifacts
npm run create-admin     # Create admin user (requires AWS credentials)
```

**Build Process Details:**
```bash
# The build script performs:
# 1. TypeScript compilation (tsc)
# 2. Lambda function packaging (zip files in dist/packages/)
#    - auth-service.zip
#    - listing.zip
#    - search.zip
#    - media.zip
#    - email.zip
#    - stats-service.zip
#    - admin-service.zip
```

**Infrastructure Dependencies:**
```bash
cd ../infrastructure
npm install

# Verify installation
npm list --depth=0
# Should show AWS CDK constructs and dependencies
```

### 3. Environment Configuration

**Frontend Environment Setup:**
```bash
cd ../frontend

# Copy environment template
cp .env.example .env.local

# Edit environment variables
nano .env.local  # or use your preferred editor
```

**Frontend Environment Variables (.env.local):**
```bash
```bash
# Backend API Configuration (choose appropriate environment)
# For development environment:
VITE_API_BASE_URL=https://dev-api.harborlist.com
VITE_ADMIN_API_BASE_URL=https://dev-api.harborlist.com/admin

# For staging environment:
# VITE_API_BASE_URL=https://staging-api.harborlist.com
# VITE_ADMIN_API_BASE_URL=https://staging-api.harborlist.com/admin

# For production environment:
# VITE_API_BASE_URL=https://api.harborlist.com
# VITE_ADMIN_API_BASE_URL=https://api.harborlist.com/admin

# Environment
VITE_ENVIRONMENT=development

# Feature Flags
VITE_ENABLE_ANALYTICS=false
VITE_ENABLE_DEBUG_MODE=true

# Optional: Local development overrides
# VITE_API_BASE_URL=http://localhost:3001  # For local backend development
```

**AWS Configuration:**
```bash
# Configure AWS credentials
aws configure
# Enter your AWS Access Key ID
# Enter your AWS Secret Access Key
# Default region: us-east-1
# Default output format: json

# Verify AWS configuration
aws sts get-caller-identity
# Should return your AWS account information
```

## 🏃‍♂️ Development Workflow

### Option 1: Hybrid Development (Recommended)

This approach runs the frontend locally while using the deployed AWS backend services.

**1. Start Frontend Development Server:**
```bash
cd frontend
npm run dev
```

The frontend will be available at `http://localhost:3000` with hot reloading enabled.

**2. Backend Development:**
For backend changes, you'll need to deploy to AWS:
```bash
cd ../backend
npm run build

cd ../infrastructure
cdk deploy
```

### Option 2: Full Local Development

For complete local development, you can use AWS SAM (Serverless Application Model) to run Lambda functions locally.

**1. Install AWS SAM CLI:**
```bash
# macOS
brew install aws-sam-cli

# Linux
pip install aws-sam-cli

# Windows
# Download and install from AWS SAM CLI releases
```

**2. Start Local Backend Services:**
```bash
cd backend
sam local start-api --port 3001
```

**3. Update Frontend Configuration:**
```bash
# In frontend/.env.local
VITE_API_BASE_URL=http://localhost:3001
```

**4. Start Frontend:**
```bash
cd frontend
npm run dev
```

## 🧪 Testing Setup

### Frontend Testing
```bash
cd frontend

# Run unit tests
npm run test

# Run E2E tests (requires backend to be running)
npm run test:e2e

# Run tests in watch mode
npm run test:watch
```

### Backend Testing
```bash
cd backend

# Run unit tests
npm run test

# Run integration tests (requires AWS credentials)
npm run test:integration

# Run all tests with coverage
npm run test:coverage
```

## 🔧 Development Tools Configuration

### VS Code Workspace Settings

Create `.vscode/settings.json` in the project root:
```json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "tailwindCSS.includeLanguages": {
    "typescript": "javascript",
    "typescriptreact": "javascript"
  },
  "files.associations": {
    "*.css": "tailwindcss"
  }
}
```

### Prettier Configuration

The project includes a `.prettierrc` file with standardized formatting rules:
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

### ESLint Configuration

Both frontend and backend include ESLint configurations for code quality:
```bash
# Check linting
npm run lint

# Fix linting issues automatically
npm run lint:fix
```

# Configure Git (if not already done)
git config --global user.name "Your Full Name"
git config --global user.email "your.email@example.com"
git config --global init.defaultBranch main

# Verify configuration
git config --list --global
```

**3. AWS CLI (v2)**
```bash
# Check AWS CLI version
aws --version

# Install AWS CLI v2 (if needed)
# macOS
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Windows (PowerShell)
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi

# Verify installation
aws --version  # Should show aws-cli/2.x.x
```

**4. AWS CDK CLI**
```bash
# Install CDK CLI globally
npm install -g aws-cdk

# Verify installation
cdk --version  # Should show 2.x.x
```

## 🚀 Environment Setup

### 1. Repository Setup

```bash
# Clone the repository
git clone <repository-url>
cd boat_listing_project

# Verify project structure
ls -la
# You should see: frontend/, backend/, infrastructure/, docs/, config/
```

### 2. AWS Configuration

**Configure AWS Credentials:**
```bash
# Configure AWS CLI with your credentials
aws configure

# Enter your AWS credentials:
# AWS Access Key ID: [Your Access Key]
# AWS Secret Access Key: [Your Secret Key]
# Default region name: us-east-1
# Default output format: json

# Verify configuration
aws sts get-caller-identity
```

**Bootstrap CDK (First-time setup):**
```bash
# Bootstrap CDK in your AWS account (one-time setup)
cd infrastructure
cdk bootstrap

# This creates necessary CDK resources in your AWS account
```

### 3. Environment Variables Configuration

**Frontend Environment Setup:**
```bash
cd frontend

# Copy environment template
cp .env.example .env.local

# Edit the environment file
nano .env.local  # or use your preferred editor
```

**Frontend Environment Variables** (`.env.local`):
```bash
# API Gateway URL (will be set after infrastructure deployment)
VITE_API_URL=https://your-api-gateway-url.amazonaws.com/prod
VITE_ENVIRONMENT=development

# Optional: Enable development features
VITE_DEBUG_MODE=true
VITE_MOCK_API=false
```

**Backend Environment Variables:**
The backend uses AWS Lambda environment variables set during deployment. For local testing, create a `.env` file in the backend directory:

```bash
cd backend

# Create local environment file for testing
cat > .env << EOF
# Local development environment variables
AWS_REGION=us-east-1
NODE_ENV=development

# These will be set automatically during deployment
# LISTINGS_TABLE=BoatListingStack-dev-ListingsTable
# USERS_TABLE=BoatListingStack-dev-UsersTable
# MEDIA_BUCKET=boatlistingstack-dev-mediabucket
EOF
```

## 🏗️ Development Environment Options

### Option 1: Hybrid Development (Recommended)

This approach runs the frontend locally while using deployed AWS backend services.

**Advantages:**
- Fastest development cycle
- Real AWS services for testing
- No complex local service setup
- Matches production environment

**Setup Steps:**

**1. Deploy Backend Infrastructure:**
```bash
cd infrastructure

# Install dependencies
npm install

# Deploy development environment
npm run deploy:dev

# Note the API Gateway URL from the output
# Example: https://abc123def4.execute-api.us-east-1.amazonaws.com/prod
```

**2. Configure Frontend:**
```bash
cd frontend

# Update .env.local with the deployed API URL
echo "VITE_API_URL=https://your-api-gateway-url.amazonaws.com/prod" > .env.local
echo "VITE_ENVIRONMENT=development" >> .env.local

# Install dependencies
npm install
```

**3. Start Development:**
```bash
# Start frontend development server
npm run dev

# The application will be available at http://localhost:5173
```

### Option 2: Full Local Development

This approach runs all services locally using mock AWS services.

**Prerequisites:**
- Docker and Docker Compose
- LocalStack for AWS service mocking

**Setup Steps:**

**1. Install Docker:**
```bash
# Verify Docker installation
docker --version
docker-compose --version

# If not installed, visit: https://docs.docker.com/get-docker/
```

**2. Install LocalStack:**
```bash
# Install LocalStack CLI
pip install localstack

# Or using Docker
docker pull localstack/localstack
```

**3. Create Docker Compose Configuration:**
```bash
# Create docker-compose.yml in project root
cat > docker-compose.yml << EOF
version: '3.8'

services:
  localstack:
    image: localstack/localstack:latest
    ports:
      - "4566:4566"
    environment:
      - SERVICES=dynamodb,s3,apigateway,lambda,iam,sts
      - DEBUG=1
      - DATA_DIR=/tmp/localstack/data
      - DOCKER_HOST=unix:///var/run/docker.sock
    volumes:
      - "/tmp/localstack:/tmp/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"

  postgres:
    image: postgres:14
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=boat_listings_dev
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
EOF
```

**4. Start Local Services:**
```bash
# Start all local services
docker-compose up -d

# Verify services are running
docker-compose ps
```

**5. Configure Local AWS Endpoints:**
```bash
# Configure AWS CLI for LocalStack
aws configure set aws_access_key_id test
aws configure set aws_secret_access_key test
aws configure set region us-east-1

# Set LocalStack endpoint
export AWS_ENDPOINT_URL=http://localhost:4566
```

## 🔧 Development Workflow Setup

### 1. Install Project Dependencies

**Install all dependencies:**
```bash
# Frontend dependencies
cd frontend
npm install

# Backend dependencies
cd ../backend
npm install

# Infrastructure dependencies
cd ../infrastructure
npm install

# Return to project root
cd ..
```

### 2. Build and Test Setup

**Frontend Build Verification:**
```bash
cd frontend

# Type checking
npm run type-check

# Linting
npm run lint

# Run tests
npm run test:run

# Build for development
npm run build:dev
```

**Backend Build Verification:**
```bash
cd backend

# Compile TypeScript
npm run build

# Run tests
npm test

# Create deployment packages
chmod +x build.sh
./build.sh
```

**Infrastructure Verification:**
```bash
cd infrastructure

# Compile CDK code
npm run build

# Run infrastructure tests
npm test

# Synthesize CloudFormation (no deployment)
npm run synth:dev
```

### 3. IDE Configuration

**Visual Studio Code Setup:**

Create `.vscode/settings.json`:
```json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "files.associations": {
    "*.css": "tailwindcss"
  },
  "eslint.workingDirectories": [
    "frontend",
    "backend",
    "infrastructure"
  ],
  "typescript.preferences.includePackageJsonAutoImports": "on"
}
```

Create `.vscode/extensions.json`:
```json
{
  "recommendations": [
    "ms-vscode.vscode-typescript-next",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "bradlc.vscode-tailwindcss",
    "ms-vscode.vscode-json",
    "amazonwebservices.aws-toolkit-vscode"
  ]
}
```

**Recommended VS Code Extensions:**
- TypeScript and JavaScript Language Features
- ESLint
- Prettier - Code formatter
- Tailwind CSS IntelliSense
- AWS Toolkit
- GitLens
- Thunder Client (for API testing)

## 🧪 Testing Your Setup

### 1. Frontend Development Test

```bash
cd frontend

# Start development server
npm run dev

# In another terminal, run tests
npm run test:run

# Check type safety
npm run type-check

# Verify linting
npm run lint
```

**Expected Results:**
- Development server starts at `http://localhost:5173`
- All tests pass
- No TypeScript errors
- No linting errors

### 2. Backend Development Test

```bash
cd backend

# Build the backend
npm run build

# Run tests
npm test

# Create deployment packages
./build.sh

# Verify packages were created
ls -la dist/packages/
```

**Expected Results:**
- TypeScript compiles without errors
- All tests pass
- ZIP packages created in `dist/packages/`

### 3. Infrastructure Test

```bash
cd infrastructure

# Build CDK code
npm run build

# Run tests
npm test

# Synthesize templates (no deployment)
npm run synth:dev

# Check generated CloudFormation
ls -la cdk.out/
```

**Expected Results:**
- CDK code compiles without errors
- Infrastructure tests pass
- CloudFormation templates generated

### 4. End-to-End Integration Test

**Deploy and Test Full Stack:**
```bash
# 1. Deploy infrastructure
cd infrastructure
npm run deploy:dev

# 2. Update frontend configuration with API URL
cd ../frontend
# Update .env.local with the API Gateway URL from deployment output

# 3. Start frontend
npm run dev

# 4. Test the application
# - Visit http://localhost:5173
# - Try creating an account
# - Test listing creation
# - Verify API connectivity
```

## 🔍 Development Tools and Utilities

### 1. Database Management

**DynamoDB Local (for full local development):**
```bash
# Install DynamoDB Local
npm install -g dynamodb-local

# Start DynamoDB Local
dynamodb-local

# Access DynamoDB Admin (optional)
npm install -g dynamodb-admin
dynamodb-admin
```

**AWS DynamoDB (for hybrid development):**
```bash
# List tables
aws dynamodb list-tables

# Describe table
aws dynamodb describe-table --table-name BoatListingStack-dev-ListingsTable

# Query items (example)
aws dynamodb scan --table-name BoatListingStack-dev-ListingsTable --limit 5
```

### 2. API Testing Tools

**Using curl:**
```bash
# Test API health
curl https://your-api-gateway-url.amazonaws.com/prod/health

# Test listings endpoint
curl https://your-api-gateway-url.amazonaws.com/prod/listings
```

**Using Thunder Client (VS Code extension):**
1. Install Thunder Client extension
2. Create new request
3. Set URL to your API Gateway endpoint
4. Test various endpoints

### 3. Log Monitoring

**CloudWatch Logs (for deployed backend):**
```bash
# View Lambda function logs
aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/BoatListingStack"

# Tail logs in real-time
aws logs tail /aws/lambda/BoatListingStack-dev-ListingFunction --follow
```

**Local Development Logs:**
```bash
# Frontend logs (in browser console)
# Backend logs (in terminal where services are running)
# Infrastructure logs (CDK deployment output)
```

## 🚨 Troubleshooting Common Issues

### 1. Node.js and npm Issues

**Problem: Node version conflicts**
```bash
# Solution: Use Node Version Manager
nvm install 18
nvm use 18
nvm alias default 18
```

**Problem: npm permission errors**
```bash
# Solution: Fix npm permissions
npm config set prefix ~/.npm-global
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### 2. AWS Configuration Issues

**Problem: AWS credentials not configured**
```bash
# Solution: Configure AWS CLI
aws configure
# Or set environment variables
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key
export AWS_DEFAULT_REGION=us-east-1
```

**Problem: CDK bootstrap required**
```bash
# Solution: Bootstrap CDK
cd infrastructure
cdk bootstrap
```

### 3. Frontend Development Issues

**Problem: CORS errors**
```bash
# Solution: Check API Gateway CORS configuration
# Ensure your local development URL is in the allowed origins
```

**Problem: Environment variables not loading**
```bash
# Solution: Verify .env.local file
cat frontend/.env.local
# Ensure variables start with VITE_
# Restart development server after changes
```

### 4. Backend Development Issues

**Problem: Lambda function errors**
```bash
# Solution: Check CloudWatch logs
aws logs tail /aws/lambda/your-function-name --follow

# Check function configuration
aws lambda get-function --function-name your-function-name
```

**Problem: DynamoDB access errors**
```bash
# Solution: Check IAM permissions
aws iam list-attached-role-policies --role-name your-lambda-role

# Verify table exists
aws dynamodb describe-table --table-name your-table-name
```

### 5. Infrastructure Issues

**Problem: CDK deployment fails**
```bash
# Solution: Check CDK version compatibility
cdk --version
npm list aws-cdk-lib

# Clean and rebuild
rm -rf node_modules cdk.out
npm install
npm run build
```

**Problem: Resource conflicts**
```bash
# Solution: Use unique resource names
# Check existing resources
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE
```

## 🔄 Development Workflow

### Daily Development Routine

**1. Start Development Session:**
```bash
# Pull latest changes
git pull origin main

# Update dependencies (if needed)
cd frontend && npm install
cd ../backend && npm install
cd ../infrastructure && npm install

# Start development server
cd ../frontend && npm run dev
```

**2. Make Changes:**
- Edit code in your preferred IDE
- Save files (auto-formatting should apply)
- Check browser for immediate feedback

**3. Test Changes:**
```bash
# Run type checking
npm run type-check

# Run tests
npm run test:run

# Run linting
npm run lint
```

**4. Commit Changes:**
```bash
# Stage changes
git add .

# Commit with descriptive message
git commit -m "feat: add new search filter functionality"

# Push to feature branch
git push origin feature/your-feature-name
```

### Hot Reload and Development Speed

**Frontend Hot Reload:**
- Vite provides instant hot module replacement
- Changes appear immediately in browser
- State is preserved during component updates

**Backend Development:**
- Use `npm run watch` for automatic TypeScript compilation
- Deploy changes with `npm run deploy:dev`
- Use CloudWatch logs for debugging

**Infrastructure Changes:**
- Use `cdk diff` to preview changes
- Deploy with `npm run deploy:dev`
- Use `cdk destroy` to clean up resources

## 📊 Performance Optimization

### Development Environment Performance

**1. Optimize Node.js Performance:**
```bash
# Increase Node.js memory limit (if needed)
export NODE_OPTIONS="--max-old-space-size=4096"

# Use npm ci for faster installs
npm ci  # instead of npm install
```

**2. Optimize Build Performance:**
```bash
# Use parallel builds
npm run build -- --parallel

# Enable TypeScript incremental compilation
# (already configured in tsconfig.json)
```

**3. Optimize Development Server:**
```bash
# Use Vite's fast refresh
# (already configured in vite.config.ts)

# Optimize bundle analysis
npm run build && npx vite-bundle-analyzer dist
```

## 🔐 Security Considerations

### Local Development Security

**1. Environment Variables:**
- Never commit `.env.local` files
- Use different credentials for development
- Rotate development credentials regularly

**2. AWS Security:**
- Use IAM roles with minimal permissions
- Enable MFA on your AWS account
- Use separate AWS accounts for dev/prod

**3. Code Security:**
- Run security audits: `npm audit`
- Keep dependencies updated: `npm update`
- Use secure coding practices

## 📚 Additional Resources

### Documentation Links
- [Getting Started Guide](getting-started.md) - Quick start for new developers
- [Code Structure Guide](code-structure.md) - Codebase organization
- [Development Workflow](development-workflow.md) - Git and collaboration workflow

### External Resources
- [React Documentation](https://react.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [AWS CDK Documentation](https://docs.aws.amazon.com/cdk/)
- [Vite Documentation](https://vitejs.dev/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)

### Community and Support
- Project GitHub Issues
- Team Slack/Discord channels
- Stack Overflow (use project tags)
- AWS Developer Forums

## 🎉 You're Ready to Develop!

Congratulations! You now have a fully configured local development environment. You can:

- ✅ Run the frontend locally with hot reload
- ✅ Deploy and test backend changes
- ✅ Manage infrastructure with CDK
- ✅ Debug issues effectively
- ✅ Follow best practices for security and performance

**Next Steps:**
1. Explore the codebase structure
2. Make your first contribution
3. Join the development community
4. Share your knowledge with others

Happy coding! 🚤