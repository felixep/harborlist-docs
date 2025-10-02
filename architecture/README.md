# Architecture Documentation

## System Architecture Overview

The boat listing platform follows a serverless-first, microservices architecture designed for scalability, cost-efficiency, and minimal operational overhead.

## Architecture Diagram

```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│   Users     │───▶│  CloudFront  │───▶│     S3      │
│ (Buyers &   │    │     CDN      │    │   React     │
│  Sellers)   │    │   + Proxy    │    │    App      │
└─────────────┘    └──────────────┘    └─────────────┘
                                              │
                                              ▼
                   ┌─────────────────────────────────────┐
                   │           API Gateway               │
                   │        (REST Endpoints)             │
                   └─────────────────────────────────────┘
                                              │
                   ┌──────────────────────────┼──────────────────────────┐
                   │                          │                          │
                   ▼                          ▼                          ▼
            ┌─────────────┐           ┌─────────────┐           ┌─────────────┐
            │   Lambda    │           │   Lambda    │           │     SES     │
            │ Functions   │           │ Functions   │           │   Email     │
            └─────────────┘           └─────────────┘           └─────────────┘
                   │                          │                          
                   ▼                          ▼                          
            ┌─────────────┐           ┌─────────────┐           ┌─────────────┐
            │  DynamoDB   │           │ OpenSearch  │           │     S3      │
            │  Listings   │           │ Serverless  │           │   Media     │
            └─────────────┘           └─────────────┘           └─────────────┘
```

## Core Components

<<<<<<< HEAD
### [Backend Architecture](backend-architecture.md)
Detailed documentation of the backend system:
- **Service Architecture**: Microservices design and communication patterns
- **API Design**: RESTful API structure and endpoints
- **Data Layer**: Database design and data access patterns
- **Authentication**: User authentication and authorization
- **Business Logic**: Core business logic implementation
- **Integration**: External service integrations

### [Frontend Architecture](frontend-architecture.md)
Comprehensive frontend system documentation:
- **Component Architecture**: React component structure and patterns
- **State Management**: Application state management approach
- **Routing**: Client-side routing and navigation
- **UI/UX Design**: User interface design principles
- **Performance**: Frontend optimization strategies
- **Testing**: Frontend testing approaches
=======
### 1. Frontend Layer

#### CloudFront CDN + S3 Static Hosting
- **Purpose**: Global content delivery and static website hosting
- **Features**: 
  - React application served from S3
  - CloudFront distribution for global performance
  - API proxy to backend services
  - Automatic HTTPS/SSL
  - SPA routing support (404 → index.html)
>>>>>>> parent of 7b06595 (feat: implement repository consolidation - documentation, scripts, and initial workflows)

#### Infrastructure as Code
- **AWS CDK**: TypeScript-based infrastructure definitions
- **Automated Deployment**: Single command deployment of all resources
- **Environment Separation**: Configurable for dev/staging/production

#### CloudFront CDN
- **Purpose**: Global content delivery network
- **Benefits**:
  - Sub-second page loads worldwide
  - Automatic HTTPS/SSL
  - DDoS protection via AWS Shield
  - Caching for static assets

### 2. API Layer

#### API Gateway (REST)
- **Endpoints**:
  - `/listings` - CRUD operations for boat listings
  - `/search` - Advanced search with filters
  - `/media` - Image/video upload endpoints
  - `/contact` - Email communication system
  - `/auth` - Authentication endpoints

#### Authentication Flow
```
User Registration/Login
        │
        ▼
   Cognito User Pool
        │
        ▼
   JWT Token Generation
        │
        ▼
   API Gateway Authorizer
        │
        ▼
   Lambda Function Access
```

### 3. Compute Layer

#### Lambda Functions

**Listing Service** (`listing-service`)
- Create, read, update, delete boat listings
- Input validation and sanitization
- DynamoDB operations
- OpenSearch indexing

**Search Service** (`search-service`)
- Advanced filtering (location, price, type, year)
- Full-text search capabilities
- Faceted search results
- Pagination and sorting

**Media Service** (`media-service`)
- Image upload to S3
- Automatic thumbnail generation
- Video processing and thumbnails
- Metadata extraction

**Email Service** (`email-service`)
- Contact form processing
- Email template management
- Privacy protection (no direct email exposure)
- Spam prevention

### 4. Data Layer

#### DynamoDB Tables

**Listings Table**
```
Primary Key: listingId (String)
Sort Key: createdAt (Number)

Attributes:
- listingId: Unique identifier
- ownerId: User ID from Cognito
- title: Boat title
- description: Detailed description
- price: Listing price
- location: Geographic location
- boatType: Type of boat
- year: Manufacturing year
- length: Boat length
- images: Array of S3 URLs
- videos: Array of S3 URLs
- status: active/inactive/sold
- createdAt: Timestamp
- updatedAt: Timestamp
```

**Users Table**
```
Primary Key: userId (String)

Attributes:
- userId: Cognito User ID
- email: Contact email
- name: Display name
- phone: Contact phone (optional)
- location: User location
- createdAt: Registration timestamp
- listingCount: Number of active listings
```

#### OpenSearch Serverless

**Search Index Structure**
```json
{
  "listingId": "string",
  "title": "text",
  "description": "text",
  "price": "number",
  "location": {
    "lat": "number",
    "lon": "number",
    "city": "string",
    "state": "string"
  },
  "boatType": "keyword",
  "year": "number",
  "length": "number",
  "status": "keyword",
  "createdAt": "date"
}
```

### 5. Storage Layer

#### S3 Buckets

<<<<<<< HEAD
1. **Deep Dive**: Explore specific architecture documents for detailed information
2. **Implementation**: Review implementation details in [Development](development/README.md)
3. **Operations**: Understand operational aspects in [Operations](operations/README.md)
4. **Security**: Review security implementation in [Security Architecture](security.md)
=======
**Media Bucket** (`boat-listings-media-{env}`)
- Original images and videos
- Organized by listing ID
- Lifecycle policies for cost optimization
- CloudFront distribution for fast delivery

**Thumbnails Bucket** (`boat-listings-thumbnails-{env}`)
- Auto-generated thumbnails
- Multiple sizes (150x150, 300x300, 600x400)
- WebP format for optimization

### 6. Communication Layer

#### Amazon SES
- **Templates**: Pre-designed email templates
- **Contact Flow**: Buyer → SES → Boat Owner
- **Privacy**: No direct email exposure
- **Tracking**: Delivery and open tracking

## Data Flow Patterns

### 1. Listing Creation Flow
```
1. User uploads images → S3 Media Bucket
2. Media Service generates thumbnails → S3 Thumbnails Bucket
3. Listing Service creates record → DynamoDB
4. Search Service indexes listing → OpenSearch
5. User receives confirmation → SES Email
```

### 2. Search Flow
```
1. User enters search criteria → React App
2. Search request → API Gateway → Search Service
3. Query execution → OpenSearch
4. Results enrichment → DynamoDB (if needed)
5. Response with listings → React App
```

### 3. Contact Flow
```
1. Buyer clicks "Contact Owner" → React App
2. Contact form submission → API Gateway → Email Service
3. Email generation → SES Templates
4. Email delivery → Boat Owner
5. Confirmation → Buyer
```

## Security Architecture

### Authentication & Authorization
- **Cognito User Pools**: Secure user management
- **JWT Tokens**: Stateless authentication
- **API Gateway Authorizers**: Request validation
- **IAM Roles**: Least privilege access

### Data Protection
- **Encryption at Rest**: All DynamoDB and S3 data
- **Encryption in Transit**: HTTPS/TLS everywhere
- **WAF**: Web Application Firewall protection
- **VPC**: Private networking for sensitive operations

### Privacy Features
- **Email Masking**: No direct email exposure
- **Data Anonymization**: PII protection
- **GDPR Compliance**: Data deletion capabilities
- **Audit Logging**: CloudTrail for all operations

## Performance Optimization

### Caching Strategy
- **CloudFront**: Static assets and API responses
- **DynamoDB DAX**: Microsecond latency for hot data
- **Lambda Provisioned Concurrency**: Reduced cold starts
- **OpenSearch**: Query result caching

### Image Optimization
- **WebP Format**: Modern image compression
- **Responsive Images**: Multiple sizes for different devices
- **Lazy Loading**: Progressive image loading
- **CDN Delivery**: Global edge locations

## Monitoring & Observability

### CloudWatch Metrics
- API Gateway request/response metrics
- Lambda function performance
- DynamoDB read/write capacity
- S3 storage and transfer metrics

### X-Ray Tracing
- End-to-end request tracing
- Performance bottleneck identification
- Error root cause analysis
- Service dependency mapping

### Custom Dashboards
- Business metrics (listings, searches, contacts)
- Technical metrics (latency, errors, throughput)
- Cost tracking and optimization alerts
- User behavior analytics

## Disaster Recovery

### Backup Strategy
- **DynamoDB**: Point-in-time recovery enabled
- **S3**: Cross-region replication for critical data
- **Code**: Git repositories with multiple remotes
- **Infrastructure**: CDK templates in version control

### High Availability
- **Multi-AZ**: All services deployed across multiple zones
- **Auto-scaling**: Automatic capacity adjustment
- **Health Checks**: Automated failure detection
- **Failover**: Automatic traffic routing to healthy resources
>>>>>>> parent of 7b06595 (feat: implement repository consolidation - documentation, scripts, and initial workflows)
