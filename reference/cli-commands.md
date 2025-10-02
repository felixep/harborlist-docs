# CLI and Command Reference

## Overview

This document provides a comprehensive reference for all command-line interfaces, scripts, and development commands available in the MarineMarket platform. Commands are organized by category and include usage examples, options, and troubleshooting information.

## Quick Reference

### Essential Commands

```bash
# Development Setup
./scripts/development/setup-dev-environment.sh

# Build All Components
npm run build                    # Backend
npm run build:prod              # Frontend (production)
npm run build                   # Infrastructure

# Development Servers
npm run dev                     # Frontend development server
npm run watch                   # Backend watch mode

# Testing
npm test                        # Run all tests
npm run test:run               # Run tests once (no watch)

# Deployment
./scripts/deployment/deploy-dev.sh        # Development
./scripts/deployment/deploy-staging.sh    # Staging
./scripts/deployment/deploy-production.sh # Production

# Admin Operations
npm run create-admin           # Create admin user
```

## Package.json Scripts

### Backend Scripts (`backend/package.json`)

#### Build Commands
```bash
# Build TypeScript and create deployment packages
npm run build
```
**Description:** Compiles TypeScript to JavaScript and creates ZIP packages for Lambda deployment.
**Output:** `dist/` directory with compiled code and `dist/packages/` with deployment packages.
**Implementation:** [`backend/package.json:6`](../backend/package.json#L6)

```bash
# Package Lambda functions for deployment
npm run package
```
**Description:** Creates ZIP files for each Lambda function with dependencies.
**Dependencies:** Requires `npm run build` to be run first.
**Output:** ZIP files in `dist/packages/` directory.

```bash
# Watch TypeScript files for changes
npm run watch
```
**Description:** Continuously compiles TypeScript files when changes are detected.
**Usage:** Development only - keeps compiled code up to date.

#### Testing Commands
```bash
# Run Jest test suite
npm test
```
**Description:** Executes all unit and integration tests using Jest.
**Configuration:** `backend/jest.config.js`
**Coverage:** Includes code coverage reporting.

```bash
# Lint TypeScript code
npm run lint
```
**Description:** Runs ESLint on TypeScript source files.
**Configuration:** `.eslintrc.js`
**Auto-fix:** Use `npm run lint -- --fix` to automatically fix issues.

#### Utility Commands
```bash
# Clean build artifacts
npm run clean
```
**Description:** Removes `dist/` directory and all build artifacts.
**Usage:** Run before fresh builds or when troubleshooting build issues.

```bash
# Create admin user
npm run create-admin
```
**Description:** Interactive script to create admin users in the system.
**Implementation:** [`backend/scripts/create-admin-user.ts`](frontend/../backend/scripts/create-admin-user.ts)
**Usage:** 
```bash
npm run create-admin -- --email admin@example.com --name "Admin User" --role super_admin
```

### Frontend Scripts (`frontend/package.json`)

#### Development Commands
```bash
# Start development server
npm run dev
```
**Description:** Starts Vite development server with hot module replacement.
**Port:** Default 5173 (configurable)
**Environment:** Uses `.env.local` for configuration.

```bash
# Preview production build
npm run preview
```
**Description:** Serves production build locally for testing.
**Usage:** Run after `npm run build` to test production bundle.

#### Build Commands
```bash
# Build for production
npm run build
```
**Description:** Creates optimized production build.
**Output:** `dist/` directory with static assets.
**Process:** TypeScript compilation + Vite bundling + optimization.

```bash
# Build for development
npm run build:dev
```
**Description:** Creates development build with source maps.
**Environment:** Uses development configuration.

```bash
# Build for staging
npm run build:staging
```
**Description:** Creates staging build with staging configuration.
**Environment:** Uses staging environment variables.

```bash
# Build for production
npm run build:prod
```
**Description:** Creates production build with production configuration.
**Environment:** Uses production environment variables.

#### Quality Assurance Commands
```bash
# Run ESLint
npm run lint
```
**Description:** Lints TypeScript and TSX files.
**Configuration:** `.eslintrc.cjs`
**Auto-fix:** Use `npm run lint -- --fix`

```bash
# Type checking
npm run type-check
```
**Description:** Runs TypeScript compiler for type checking without emitting files.
**Usage:** Validates TypeScript types across the project.

```bash
# Run tests
npm test
```
**Description:** Runs Vitest test suite in watch mode.
**Configuration:** `frontend/vite.config.ts`

```bash
# Run tests once
npm run test:run
```
**Description:** Runs tests once without watch mode.
**Usage:** CI/CD pipelines and pre-deployment validation.

### Infrastructure Scripts (`infrastructure/package.json`)

#### Build Commands
```bash
# Compile TypeScript
npm run build
```
**Description:** Compiles CDK TypeScript code to JavaScript.
**Output:** Compiled files for CDK operations.

#### CDK Commands
```bash
# Synthesize CloudFormation templates
npm run synth
npm run synth:dev
npm run synth:staging
npm run synth:prod
```
**Description:** Generates CloudFormation templates from CDK code.
**Environment-specific:** Each command targets specific environment configuration.
**Output:** `cdk.out/` directory with CloudFormation templates.

```bash
# Deploy infrastructure
npm run deploy
npm run deploy:dev
npm run deploy:staging
npm run deploy:prod
```
**Description:** Deploys infrastructure to AWS using CDK.
**Requirements:** AWS credentials configured, CDK bootstrapped.
**Environment-specific:** Each command deploys to specific environment.

```bash
# Deploy without custom domains
npm run deploy:dev:no-domains
npm run deploy:staging:no-domains
npm run deploy:prod:no-domains
```
**Description:** Deploys infrastructure without custom domain configuration.
**Usage:** Development environments or when domains aren't configured.

```bash
# Show infrastructure differences
npm run diff
npm run diff:dev
npm run diff:staging
npm run diff:prod
```
**Description:** Shows differences between deployed and local infrastructure.
**Usage:** Review changes before deployment.

#### Validation Commands
```bash
# Run infrastructure tests
npm test
npm run test:watch
```
**Description:** Runs CDK unit tests using Jest.
**Coverage:** Tests CDK construct configurations.

```bash
# Validate infrastructure
npm run validate
```
**Description:** Builds, tests, and synthesizes for development environment.
**Usage:** Pre-deployment validation.

```bash
# Validate all environments
npm run validate:all
```
**Description:** Validates infrastructure for all environments.
**Usage:** Comprehensive validation before releases.

### Configuration Scripts (`config/package.json`)

#### Validation Commands
```bash
# Validate all configurations
npm run validate
```
**Description:** Validates all configuration files against schema.
**Implementation:** [`config/validation/validate-config.js`](frontend/../config/validation/validate-config.js)

```bash
# Validate specific environment
npm run validate:dev
npm run validate:staging
npm run validate:prod
```
**Description:** Validates configuration for specific environment.
**Schema:** Uses JSON Schema validation.

#### Configuration Management
```bash
# Load configuration
npm run load
npm run load:dev
npm run load:staging
npm run load:prod
```
**Description:** Loads and displays configuration for environment.
**Implementation:** [`config/config-loader.js`](frontend/../config/config-loader.js)

## Deployment Scripts

### Development Deployment

#### `./scripts/deployment/deploy-dev.sh`

**Purpose:** Builds and prepares application for development deployment.

**Usage:**
```bash
./scripts/deployment/deploy-dev.sh [options]
```

**Options:**
- `--no-domains`: Disable custom domains (development only)
- `--dry-run`: Preview changes without execution
- `--verbose`: Enable detailed logging
- `--config FILE`: Specify custom configuration file

**Examples:**
```bash
# Standard development deployment
./scripts/deployment/deploy-dev.sh --verbose

# Development without custom domains
./scripts/deployment/deploy-dev.sh --no-domains

# Preview deployment changes
./scripts/deployment/deploy-dev.sh --dry-run
```

**Process:**
1. Validates prerequisites (Node.js, npm, directories)
2. Builds backend Lambda functions
3. Builds frontend assets for development
4. Prepares infrastructure for deployment
5. Displays next steps and deployment commands

**Implementation:** [`scripts/deployment/deploy-dev.sh`](../scripts/deployment/deploy-dev.sh)

### Staging Deployment

#### `./scripts/deployment/deploy-staging.sh`

**Purpose:** Builds and deploys application to staging environment.

**Usage:**
```bash
./scripts/deployment/deploy-staging.sh [options]
```

**Options:**
- `--no-domains`: Disable custom domains (not recommended)
- `--skip-build`: Skip build process (use existing artifacts)
- `--skip-tests`: Skip test execution before deployment
- `--dry-run`: Preview changes without execution
- `--verbose`: Enable detailed logging
- `--config FILE`: Specify custom configuration file

**Examples:**
```bash
# Full staging deployment with tests
./scripts/deployment/deploy-staging.sh --verbose

# Quick deployment skipping tests
./scripts/deployment/deploy-staging.sh --skip-tests

# Preview staging deployment
./scripts/deployment/deploy-staging.sh --dry-run
```

**Process:**
1. Validates prerequisites including AWS credentials
2. Runs pre-deployment tests (unless skipped)
3. Builds all components for staging
4. Deploys infrastructure to AWS
5. Verifies deployment health
6. Displays deployment summary

**Implementation:** [`scripts/deployment/deploy-staging.sh`](../scripts/deployment/deploy-staging.sh)

### Production Deployment

#### `./scripts/deployment/deploy-production.sh`

**Purpose:** Builds and deploys application to production environment with safety checks.

**Usage:**
```bash
./scripts/deployment/deploy-production.sh [options]
```

**Options:**
- `--no-domains`: Disable custom domains (NOT recommended)
- `--skip-build`: Skip build process (use existing artifacts)
- `--skip-tests`: Skip test execution (NOT recommended)
- `--force`: Skip safety confirmations (use with extreme caution)
- `--dry-run`: Preview changes without execution
- `--verbose`: Enable detailed logging
- `--config FILE`: Specify custom configuration file

**Examples:**
```bash
# Standard production deployment
./scripts/deployment/deploy-production.sh --verbose

# Preview production deployment
./scripts/deployment/deploy-production.sh --dry-run

# Emergency deployment (NOT RECOMMENDED)
./scripts/deployment/deploy-production.sh --force --skip-tests
```

**Safety Features:**
- Interactive confirmation required
- Git branch validation (main/master recommended)
- Uncommitted changes detection
- Comprehensive test execution
- Deployment backup creation
- Post-deployment verification

**Process:**
1. Production deployment confirmation
2. Validates prerequisites and Git status
3. Runs comprehensive tests
4. Builds all components for production
5. Creates deployment backup
6. Deploys infrastructure to AWS
7. Verifies production deployment
8. Displays deployment summary and monitoring info

**Implementation:** [`scripts/deployment/deploy-production.sh`](../scripts/deployment/deploy-production.sh)

## Development Scripts

### Environment Setup

#### `./scripts/development/setup-dev-environment.sh`

**Purpose:** Sets up complete development environment with all dependencies.

**Usage:**
```bash
./scripts/development/setup-dev-environment.sh [options]
```

**Options:**
- `--skip-deps`: Skip dependency installation
- `--skip-env`: Skip environment file setup
- `--dry-run`: Preview changes without execution
- `--verbose`: Enable detailed logging

**Examples:**
```bash
# Full environment setup
./scripts/development/setup-dev-environment.sh --verbose

# Setup without installing dependencies
./scripts/development/setup-dev-environment.sh --skip-deps

# Preview setup process
./scripts/development/setup-dev-environment.sh --dry-run
```

**Process:**
1. Checks system requirements (Node.js 18+, npm, Git)
2. Installs backend dependencies
3. Installs frontend dependencies
4. Installs infrastructure dependencies
5. Sets up environment files (.env.local, .env)
6. Configures development tools (Git hooks, linting)
7. Displays next steps and documentation links

**System Requirements:**
- Node.js 18 or higher
- npm (latest version recommended)
- Git
- AWS CLI (optional but recommended)
- Docker (optional but recommended)

**Implementation:** [`scripts/development/setup-dev-environment.sh`](../scripts/development/setup-dev-environment.sh)

## Utility Scripts

### Documentation Builder

#### `./scripts/utilities/build-docs.sh`

**Purpose:** Builds comprehensive documentation from various sources.

**Usage:**
```bash
./scripts/utilities/build-docs.sh [options]
```

**Options:**
- `--output-dir DIR`: Output directory (default: docs-build)
- `--api-only`: Build only API documentation
- `--guides-only`: Build only user guides
- `--reference-only`: Build only reference documentation
- `--serve`: Start documentation server after building
- `--watch`: Watch for changes and rebuild automatically
- `--verbose`: Enable verbose output

**Examples:**
```bash
# Build all documentation
./scripts/utilities/build-docs.sh

# Build and serve documentation
./scripts/utilities/build-docs.sh --serve

# Build only API docs with watch mode
./scripts/utilities/build-docs.sh --api-only --watch
```

**Build Process:**
1. Generates API documentation from OpenAPI specs
2. Converts markdown guides to HTML
3. Builds reference documentation
4. Creates unified navigation and search
5. Optimizes assets and generates index

**Output Structure:**
```
docs-build/
├── index.html              # Main documentation hub
├── api/                    # API documentation
│   └── index.html
├── guides/                 # User guides
│   └── index.html
└── reference/              # Reference materials
    └── index.html
```

**Implementation:** [`scripts/utilities/build-docs.sh`](../scripts/utilities/build-docs.sh)

## Operations Scripts

### Database Backup

#### `./scripts/operations/backup-database.sh`

**Purpose:** Creates backups of DynamoDB tables and other data stores.

**Usage:**
```bash
./scripts/operations/backup-database.sh [environment] [options]
```

**Arguments:**
- `environment`: Target environment (dev, staging, production)

**Options:**
- `--type TYPE`: Backup type (full, incremental)
- `--retention N`: Retention period in days (default: 30)
- `--dry-run`: Preview changes without execution
- `--verbose`: Enable detailed logging
- `--config FILE`: Specify custom configuration file

**Examples:**
```bash
# Full backup of development environment
./scripts/operations/backup-database.sh dev --verbose

# Incremental backup of production
./scripts/operations/backup-database.sh production --type incremental

# Preview backup process
./scripts/operations/backup-database.sh staging --dry-run
```

**Process:**
1. Discovers DynamoDB tables for environment
2. Creates on-demand backups for each table
3. Cleans up old backups based on retention policy
4. Generates backup report with metadata
5. Displays backup summary

**Requirements:**
- AWS CLI configured with appropriate permissions
- DynamoDB backup permissions
- jq for JSON processing

**Implementation:** [`scripts/operations/backup-database.sh`](../scripts/operations/backup-database.sh)

## Admin Commands

### Create Admin User

#### `npm run create-admin` (Backend)

**Purpose:** Creates admin users in the HarborList system.

**Usage:**
```bash
npm run create-admin -- --email <email> --name <name> --role <role> [options]
```

**Required Arguments:**
- `--email <email>`: Admin user email address
- `--name <name>`: Admin user full name
- `--role <role>`: Admin role (super_admin, admin, moderator, support)

**Optional Arguments:**
- `--password <pass>`: Custom password (auto-generated if not provided)
- `--permissions <p>`: Comma-separated list of permissions

**Examples:**
```bash
# Create super admin
npm run create-admin -- --email admin@harborlist.com --name "Super Admin" --role super_admin

# Create moderator with custom password
npm run create-admin -- --email mod@harborlist.com --name "Content Moderator" --role moderator --password MySecurePass123!

# Create support user with specific permissions
npm run create-admin -- --email support@harborlist.com --name "Support User" --role support --permissions user_management,audit_log_view
```

**Available Roles:**
- `super_admin`: Full system access
- `admin`: User management, content moderation, analytics
- `moderator`: Content moderation only
- `support`: Limited support access

**Available Permissions:**
- `user_management`: Manage user accounts
- `content_moderation`: Moderate listings and content
- `financial_access`: Access financial data
- `system_config`: Configure system settings
- `analytics_view`: View analytics and reports
- `audit_log_view`: View audit logs

**Environment Variables:**
- `USERS_TABLE`: DynamoDB users table name (default: boat-users)
- `AWS_REGION`: AWS region (default: us-east-1)

**Implementation:** [`backend/scripts/create-admin-user.ts`](frontend/../backend/scripts/create-admin-user.ts)

## Build Scripts

### Backend Build

#### `backend/build.sh`

**Purpose:** Builds Lambda functions and creates deployment packages.

**Usage:**
```bash
cd backend && ./build.sh
```

**Process:**
1. Cleans previous build artifacts
2. Installs npm dependencies
3. Compiles TypeScript to JavaScript
4. Creates deployment packages for each Lambda function
5. Installs production dependencies in packages
6. Creates ZIP files for Lambda deployment

**Functions Built:**
- `auth-service`: Authentication and authorization
- `listing`: Boat listing management
- `search`: Search functionality
- `media`: Media upload and processing
- `email`: Email notifications
- `stats-service`: Analytics and statistics
- `admin-service`: Administrative operations

**Output:**
```
dist/
├── packages/
│   ├── auth-service.zip
│   ├── listing.zip
│   ├── search.zip
│   ├── media.zip
│   ├── email.zip
│   ├── stats-service.zip
│   └── admin-service.zip
└── [compiled JavaScript files]
```

**Implementation:** [`backend/build.sh`](../backend/build.sh)

### Infrastructure Build

#### `infrastructure/deploy.sh`

**Purpose:** Builds and prepares application for deployment.

**Usage:**
```bash
cd infrastructure && ./deploy.sh [environment] [use_domains]
```

**Arguments:**
- `environment`: Target environment (default: dev)
- `use_domains`: Whether to use custom domains (default: true)

**Examples:**
```bash
# Deploy to development with domains
./deploy.sh dev true

# Deploy to production without domains
./deploy.sh prod false
```

**Process:**
1. Builds backend Lambda functions
2. Builds frontend for specified environment
3. Prepares infrastructure for deployment
4. Displays deployment instructions and next steps

**Implementation:** [`infrastructure/deploy.sh`](../infrastructure/deploy.sh)

## Environment Variables

### Backend Environment Variables

```bash
# Database Configuration
USERS_TABLE=boat-users                    # DynamoDB users table
LISTINGS_TABLE=boat-listings              # DynamoDB listings table
SESSIONS_TABLE=boat-sessions              # DynamoDB sessions table
AUDIT_LOGS_TABLE=boat-audit-logs          # DynamoDB audit logs table

# AWS Configuration
AWS_REGION=us-east-1                      # AWS region
MEDIA_BUCKET=boat-media-bucket            # S3 media bucket
THUMBNAILS_BUCKET=boat-thumbnails-bucket  # S3 thumbnails bucket

# Authentication
JWT_SECRET=your-jwt-secret                # JWT signing secret
JWT_REFRESH_SECRET=your-refresh-secret    # JWT refresh secret
MFA_ISSUER=HarborList                     # MFA issuer name

# External Services
SENDGRID_API_KEY=your-sendgrid-key       # Email service API key
CLOUDFLARE_API_TOKEN=your-cf-token       # Cloudflare API token
```

### Frontend Environment Variables

```bash
# API Configuration
VITE_API_BASE_URL=https://api.harborlist.com # Backend API URL (production)
# Development: https://dev-api.harborlist.com
# Staging: https://staging-api.harborlist.com
VITE_API_TIMEOUT=30000                    # API request timeout

# Environment
VITE_ENVIRONMENT=development              # Environment name

# Feature Flags
VITE_ENABLE_DEBUG=true                    # Debug mode
VITE_ENABLE_ANALYTICS=false               # Analytics tracking

# External Services
VITE_CLOUDFLARE_SITE_KEY=your-site-key   # Cloudflare site key
VITE_GOOGLE_ANALYTICS_ID=your-ga-id      # Google Analytics ID
```

### Infrastructure Environment Variables

```bash
# CDK Configuration
CDK_DEFAULT_ACCOUNT=123456789012          # AWS account ID
CDK_DEFAULT_REGION=us-east-1              # Default AWS region

# Domain Configuration
DOMAIN_NAME=harborlist.com                # Primary domain
CLOUDFLARE_ZONE_ID=your-zone-id          # Cloudflare zone ID
CLOUDFLARE_API_TOKEN=your-api-token      # Cloudflare API token

# Environment Context
ENVIRONMENT=development                   # Target environment
USE_CUSTOM_DOMAINS=true                  # Enable custom domains
```

## Troubleshooting

### Common Issues

#### Build Failures

**Issue:** `npm run build` fails with TypeScript errors
```bash
# Solution: Check TypeScript configuration and fix type errors
npm run type-check
npm run lint
```

**Issue:** Lambda package creation fails
```bash
# Solution: Clean and rebuild
npm run clean
npm install
npm run build
```

#### Deployment Issues

**Issue:** CDK deployment fails with permissions error
```bash
# Solution: Check AWS credentials and permissions
aws sts get-caller-identity
aws iam get-user
```

**Issue:** Custom domains not working
```bash
# Solution: Verify domain configuration
# Check Cloudflare DNS settings
# Verify SSL certificate status
```

#### Development Environment Issues

**Issue:** Frontend development server won't start
```bash
# Solution: Check port availability and environment files
lsof -i :5173
cp .env.example .env.local
```

**Issue:** Backend tests failing
```bash
# Solution: Check test environment and dependencies
npm install
npm run lint
npm test -- --verbose
```

### Debug Commands

```bash
# Check system requirements
node --version
npm --version
aws --version

# Verify AWS configuration
aws configure list
aws sts get-caller-identity

# Check build outputs
ls -la dist/
ls -la frontend/dist/
ls -la infrastructure/cdk.out/

# Validate configurations
npm run validate          # Config validation
npm run type-check       # TypeScript validation
npm run lint            # Code quality validation
```

### Log Locations

```bash
# Script logs
/tmp/deploy-dev.log
/tmp/deploy-staging.log
/tmp/deploy-production.log
/tmp/setup-dev-environment.log
/tmp/backup-database.log

# Application logs (AWS CloudWatch)
/aws/lambda/auth-service
/aws/lambda/listing-service
/aws/lambda/admin-service
/aws/apigateway/api-gateway-logs

# Build logs
backend/npm-debug.log
frontend/npm-debug.log
infrastructure/cdk.out/
```

## Performance Tips

### Build Optimization

```bash
# Use npm ci for faster, reliable installs
npm ci

# Parallel builds where possible
npm run build & npm run build:frontend & wait

# Clean builds for troubleshooting
npm run clean && npm run build
```

### Development Workflow

```bash
# Use watch modes for active development
npm run watch           # Backend TypeScript
npm run dev            # Frontend with HMR

# Run tests in watch mode
npm test               # Frontend tests
npm run test:watch     # Infrastructure tests
```

### Deployment Optimization

```bash
# Use environment-specific builds
npm run build:dev      # Development optimizations
npm run build:prod     # Production optimizations

# Skip unnecessary steps when safe
./deploy-staging.sh --skip-tests    # Skip tests if already run
./deploy-staging.sh --skip-build    # Use existing build artifacts
```

## Integration with IDEs

### VS Code Configuration

**Recommended Extensions:**
- TypeScript and JavaScript Language Features
- ESLint
- Prettier
- AWS Toolkit
- GitLens

**Tasks Configuration (`.vscode/tasks.json`):**
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Build All",
      "type": "shell",
      "command": "npm run build",
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always"
      }
    },
    {
      "label": "Deploy Dev",
      "type": "shell",
      "command": "./scripts/deployment/deploy-dev.sh",
      "group": "build"
    }
  ]
}
```

### Command Palette Integration

**Custom Commands:**
- `HarborList: Build All Components`
- `HarborList: Deploy to Development`
- `HarborList: Run Tests`
- `HarborList: Create Admin User`

## Continuous Integration

### GitHub Actions Integration

```yaml
# Example workflow for automated testing
name: CI/CD Pipeline
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: ./scripts/development/setup-dev-environment.sh --skip-env
      - run: npm test
      - run: npm run lint
      - run: npm run build
```

### Pre-commit Hooks

```bash
# Install pre-commit hooks
./scripts/development/setup-dev-environment.sh

# Manual pre-commit check
npm run lint
npm test
npm run type-check
```

## Security Considerations

### Credential Management

```bash
# Use AWS profiles for different environments
aws configure --profile dev
aws configure --profile staging
aws configure --profile production

# Set profile for deployment
export AWS_PROFILE=production
./scripts/deployment/deploy-production.sh
```

### Environment Isolation

```bash
# Use environment-specific configurations
./deploy-dev.sh --config config/dev.json
./deploy-prod.sh --config config/production.json

# Validate configurations before deployment
npm run validate:prod
```

### Audit and Compliance

```bash
# Generate audit reports
./scripts/operations/backup-database.sh production --verbose

# Check security configurations
npm audit
npm audit fix
```

This comprehensive CLI reference provides developers and operators with all the information needed to effectively use the MarineMarket platform's command-line interfaces and automation scripts.