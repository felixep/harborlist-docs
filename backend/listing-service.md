# Listing Service Lambda Function Documentation

## Overview

The Listing Service is a core Lambda function that manages all boat listing operations for the HarborList platform. It provides comprehensive CRUD (Create, Read, Update, Delete) functionality with robust validation, data sanitization, and owner-based access control to ensure data integrity and security.

## Architecture

### Function Structure
- **Runtime**: Node.js 18 with TypeScript
- **Handler**: `backend/src/listing/index.ts`
- **Dependencies**: AWS SDK v3, shared utilities, database service
- **Database**: DynamoDB with GSI for owner-based queries
- **Authentication**: JWT-based user authentication via API Gateway authorizer

### Design Principles
- **Owner-based Access Control**: Users can only modify their own listings
- **Data Validation**: Comprehensive validation for all input fields
- **Data Sanitization**: XSS protection through string sanitization
- **Pagination Support**: Efficient listing retrieval with pagination
- **View Tracking**: Automatic view count increment for analytics

## API Endpoints

### GET `/listings/{id}`
**Purpose**: Retrieve a specific boat listing by ID

**Path Parameters**:
- `id` (string): The unique listing identifier

**Response**:
```typescript
{
  listing: {
    listingId: string;
    ownerId: string;
    title: string;
    description: string;
    price: number;
    location: {
      city: string;
      state: string;
      zipCode?: string;
      coordinates?: { lat: number; lon: number; };
    };
    boatDetails: {
      type: string;
      manufacturer?: string;
      model?: string;
      year: number;
      length: number;
      beam?: number;
      draft?: number;
      engine?: string;
      hours?: number;
      condition: 'Excellent' | 'Good' | 'Fair' | 'Needs Work';
    };
    features: string[];
    images: string[];
    videos?: string[];
    thumbnails: string[];
    status: 'active' | 'inactive' | 'sold';
    views: number;
    createdAt: number;
    updatedAt: number;
  }
}
```

**Features**:
- Automatic view count increment on each retrieval
- Returns 404 if listing not found
- No authentication required for public viewing

### GET `/listings`
**Purpose**: Retrieve paginated list of active boat listings

**Query Parameters**:
- `limit` (number, optional): Maximum number of listings to return (default: 20)
- `nextToken` (string, optional): Pagination token for next page

**Response**:
```typescript
{
  listings: Listing[];
  total: number;
  nextToken?: string; // Present if more results available
}
```

**Features**:
- Filters to show only active listings
- Base64-encoded pagination tokens
- Configurable result limits
- Efficient DynamoDB scan with filtering

### POST `/listings`
**Purpose**: Create a new boat listing

**Authentication**: Required (JWT token)

**Request Body**:
```typescript
{
  title: string; // Required
  description: string; // Required
  price: number; // Required, $1 - $10,000,000
  location: { // Required
    city: string; // Required
    state: string; // Required
    zipCode?: string;
    coordinates?: { lat: number; lon: number; };
  };
  boatDetails: { // Required
    type: string; // Required
    manufacturer?: string;
    model?: string;
    year: number; // Required, 1900 - current year + 1
    length: number; // Required
    beam?: number;
    draft?: number;
    engine?: string;
    hours?: number;
    condition: 'Excellent' | 'Good' | 'Fair' | 'Needs Work'; // Required
  };
  features?: string[];
  images?: string[];
  videos?: string[];
  thumbnails?: string[];
}
```

**Response**:
```typescript
{
  listingId: string;
}
```

**Validation Rules**:
- **Title**: Required, sanitized for XSS protection
- **Description**: Required, sanitized for XSS protection
- **Price**: Must be between $1 and $10,000,000
- **Location**: City and state are required
- **Boat Year**: Must be between 1900 and current year + 1
- **Boat Length**: Required numeric value
- **Condition**: Must be one of the predefined values

### PUT `/listings/{id}`
**Purpose**: Update an existing boat listing

**Authentication**: Required (JWT token)
**Authorization**: Owner-only access

**Path Parameters**:
- `id` (string): The listing ID to update

**Request Body**: Same as POST, but all fields are optional

**Response**:
```typescript
{
  message: "Listing updated successfully"
}
```

**Access Control**:
- Verifies listing exists
- Ensures authenticated user is the listing owner
- Returns 403 Forbidden for non-owners
- Maintains original creation timestamp

### DELETE `/listings/{id}`
**Purpose**: Delete a boat listing

**Authentication**: Required (JWT token)
**Authorization**: Owner-only access

**Path Parameters**:
- `id` (string): The listing ID to delete

**Response**:
```typescript
{
  message: "Listing deleted successfully"
}
```

**Access Control**:
- Verifies listing exists
- Ensures authenticated user is the listing owner
- Returns 403 Forbidden for non-owners
- Permanently removes listing from database

## Data Model

### Listing Interface
```typescript
interface Listing {
  listingId: string; // Unique identifier
  ownerId: string; // User ID of the listing owner
  title: string; // Boat listing title
  description: string; // Detailed description
  price: number; // Price in USD
  location: Location; // Geographic information
  boatDetails: BoatDetails; // Technical specifications
  features: string[]; // Additional features/amenities
  images: string[]; // Image URLs
  videos?: string[]; // Video URLs (optional)
  thumbnails: string[]; // Thumbnail image URLs
  status: 'active' | 'inactive' | 'sold'; // Listing status
  views?: number; // View count for analytics
  createdAt: number; // Unix timestamp
  updatedAt: number; // Unix timestamp
}
```

### Location Interface
```typescript
interface Location {
  city: string; // Required city name
  state: string; // Required state/province
  zipCode?: string; // Optional postal code
  coordinates?: { // Optional GPS coordinates
    lat: number; // Latitude
    lon: number; // Longitude
  };
}
```

### BoatDetails Interface
```typescript
interface BoatDetails {
  type: string; // Boat type (e.g., "Sailboat", "Motorboat")
  manufacturer?: string; // Boat manufacturer
  model?: string; // Boat model
  year: number; // Manufacturing year
  length: number; // Length in feet
  beam?: number; // Width in feet
  draft?: number; // Draft in feet
  engine?: string; // Engine description
  hours?: number; // Engine hours
  condition: 'Excellent' | 'Good' | 'Fair' | 'Needs Work'; // Overall condition
}
```

## Validation and Sanitization

### Input Validation
```typescript
// Required field validation
validateRequired(body, ['title', 'description', 'price', 'location', 'boatDetails']);
validateRequired(body.location!, ['city', 'state']);
validateRequired(body.boatDetails!, ['type', 'year', 'length', 'condition']);

// Price validation
export function validatePrice(price: number): boolean {
  return price > 0 && price <= 10000000; // $1 to $10M
}

// Year validation
export function validateYear(year: number): boolean {
  const currentYear = new Date().getFullYear();
  return year >= 1900 && year <= currentYear + 1;
}
```

### Data Sanitization
```typescript
// XSS protection for string fields
export function sanitizeString(str: string): string {
  return str.trim().replace(/[<>]/g, '');
}

// Applied to all user-input strings
title: sanitizeString(body.title!),
description: sanitizeString(body.description!),
features: body.features?.map(f => sanitizeString(f)) || [],
```

### Validation Error Responses
```typescript
// Missing required fields
{ error: { code: 'VALIDATION_ERROR', message: 'Missing required fields: title, price' } }

// Invalid price range
{ error: { code: 'INVALID_PRICE', message: 'Price must be between $1 and $10,000,000' } }

// Invalid year range
{ error: { code: 'INVALID_YEAR', message: 'Year must be between 1900 and current year + 1' } }
```

## Owner-Based Access Control

### Authentication Flow
```typescript
// Extract user ID from JWT token
export function getUserId(event: APIGatewayProxyEvent): string {
  const userId = event.requestContext.authorizer?.claims?.sub;
  if (!userId) {
    throw new Error('User not authenticated');
  }
  return userId;
}
```

### Authorization Checks
```typescript
// Verify ownership before updates/deletes
const existingListing = await db.getListing(listingId);
if (!existingListing) {
  return createErrorResponse(404, 'NOT_FOUND', 'Listing not found', requestId);
}

if (existingListing.ownerId !== userId) {
  return createErrorResponse(403, 'FORBIDDEN', 'You can only update your own listings', requestId);
}
```

### Access Control Matrix
| Operation | Authentication | Authorization | Notes |
|-----------|---------------|---------------|-------|
| GET (single) | No | No | Public viewing |
| GET (list) | No | No | Public browsing |
| POST | Yes | Owner | Creates with authenticated user as owner |
| PUT | Yes | Owner only | Can only update own listings |
| DELETE | Yes | Owner only | Can only delete own listings |

## Database Operations

### DynamoDB Table Structure
```typescript
// Primary Key: id (listingId)
// GSI: OwnerIndex (ownerId as partition key)
{
  id: string; // Primary key (same as listingId)
  listingId: string; // Business identifier
  ownerId: string; // GSI partition key
  // ... other listing fields
}
```

### Database Service Methods
```typescript
class DatabaseService {
  // Create new listing with condition check
  async createListing(listing: Listing): Promise<void> {
    await docClient.send(new PutCommand({
      TableName: LISTINGS_TABLE,
      Item: { ...listing, id: listing.listingId },
      ConditionExpression: 'attribute_not_exists(id)', // Prevent duplicates
    }));
  }

  // Retrieve single listing
  async getListing(listingId: string): Promise<Listing | null> {
    const result = await docClient.send(new GetCommand({
      TableName: LISTINGS_TABLE,
      Key: { id: listingId },
    }));
    return result.Item as Listing || null;
  }

  // Update listing with dynamic expression building
  async updateListing(listingId: string, updates: Partial<Listing>): Promise<void> {
    const updateExpression = [];
    const expressionAttributeNames: Record<string, string> = {};
    const expressionAttributeValues: Record<string, any> = {};

    for (const [key, value] of Object.entries(updates)) {
      if (key !== 'listingId' && key !== 'id') {
        updateExpression.push(`#${key} = :${key}`);
        expressionAttributeNames[`#${key}`] = key;
        expressionAttributeValues[`:${key}`] = value;
      }
    }

    await docClient.send(new UpdateCommand({
      TableName: LISTINGS_TABLE,
      Key: { id: listingId },
      UpdateExpression: `SET ${updateExpression.join(', ')}`,
      ExpressionAttributeNames: expressionAttributeNames,
      ExpressionAttributeValues: expressionAttributeValues,
    }));
  }

  // Paginated listing retrieval with status filtering
  async getListings(limit: number = 20, lastKey?: any): Promise<{ listings: Listing[]; lastKey?: any }> {
    const result = await docClient.send(new ScanCommand({
      TableName: LISTINGS_TABLE,
      FilterExpression: '#status = :status',
      ExpressionAttributeNames: { '#status': 'status' },
      ExpressionAttributeValues: { ':status': 'active' },
      Limit: limit,
      ExclusiveStartKey: lastKey,
    }));

    return {
      listings: result.Items as Listing[] || [],
      lastKey: result.LastEvaluatedKey,
    };
  }

  // Atomic view count increment
  async incrementViews(listingId: string): Promise<void> {
    await docClient.send(new UpdateCommand({
      TableName: LISTINGS_TABLE,
      Key: { id: listingId },
      UpdateExpression: 'ADD #views :inc',
      ExpressionAttributeNames: { '#views': 'views' },
      ExpressionAttributeValues: { ':inc': 1 },
    }));
  }
}
```

## Pagination Implementation

### Token-Based Pagination
```typescript
// Encode pagination token
if (result.lastKey) {
  response.nextToken = Buffer.from(JSON.stringify(result.lastKey)).toString('base64');
}

// Decode pagination token
let lastKey;
if (nextToken) {
  try {
    lastKey = JSON.parse(Buffer.from(nextToken, 'base64').toString());
  } catch (error) {
    return createErrorResponse(400, 'INVALID_TOKEN', 'Invalid pagination token', requestId);
  }
}
```

### Pagination Features
- **Base64 Encoding**: Secure token format
- **Error Handling**: Invalid token detection
- **Configurable Limits**: Default 20, maximum configurable
- **Stateless**: No server-side pagination state

## Error Handling

### Comprehensive Error Responses
```typescript
// Authentication errors
{ error: { code: 'UNAUTHORIZED', message: 'User not authenticated' } }

// Authorization errors
{ error: { code: 'FORBIDDEN', message: 'You can only update your own listings' } }

// Validation errors
{ error: { code: 'VALIDATION_ERROR', message: 'Missing required fields: title, price' } }
{ error: { code: 'INVALID_PRICE', message: 'Price must be between $1 and $10,000,000' } }
{ error: { code: 'INVALID_YEAR', message: 'Year must be between 1900 and current year + 1' } }

// Resource errors
{ error: { code: 'NOT_FOUND', message: 'Listing not found' } }

// System errors
{ error: { code: 'DATABASE_ERROR', message: 'Failed to create listing' } }
{ error: { code: 'INTERNAL_ERROR', message: 'Internal server error' } }
```

### Error Handling Strategy
1. **Input Validation**: Catch validation errors early
2. **Authentication Check**: Verify user identity
3. **Authorization Check**: Verify user permissions
4. **Database Operations**: Handle DynamoDB errors
5. **Generic Fallback**: Catch unexpected errors

## Security Features

### XSS Protection
```typescript
// Sanitize all user-input strings
export function sanitizeString(str: string): string {
  return str.trim().replace(/[<>]/g, '');
}

// Applied to:
// - title, description
// - location fields (city, zipCode)
// - boat details (manufacturer, model, engine)
// - features array
```

### Input Validation
- **Required Field Validation**: Ensures critical data presence
- **Data Type Validation**: Numeric ranges for price and year
- **Format Validation**: String length and content checks
- **Business Logic Validation**: Realistic value ranges

### Access Control
- **JWT Authentication**: Token-based user identification
- **Owner Authorization**: Resource-level access control
- **Operation-Specific Permissions**: Different rules per HTTP method

## Performance Considerations

### Database Optimization
- **Primary Key Access**: O(1) lookup for single listings
- **GSI for Owner Queries**: Efficient owner-based listing retrieval
- **Conditional Writes**: Prevent duplicate listing creation
- **Atomic Updates**: Consistent view count increments

### Pagination Efficiency
- **Scan with Filtering**: Active listings only
- **Configurable Limits**: Control response size
- **Token-Based Continuation**: Stateless pagination

### Response Optimization
- **Minimal Data Transfer**: Only necessary fields
- **Efficient JSON Serialization**: Structured response format
- **CORS Headers**: Proper browser caching

## Monitoring and Analytics

### View Tracking
```typescript
// Automatic view increment on listing retrieval
await db.incrementViews(listingId);
```

### Metrics to Monitor
- **Listing Creation Rate**: New listings per time period
- **View Patterns**: Popular listings and viewing trends
- **Update Frequency**: How often listings are modified
- **Error Rates**: Validation and system errors
- **Response Times**: API performance metrics

### Logging Strategy
- **Request Logging**: All API calls with request IDs
- **Error Logging**: Detailed error information
- **Performance Logging**: Database operation timing
- **Security Logging**: Authentication and authorization events

## Environment Configuration

### Required Environment Variables
```bash
# DynamoDB Tables
LISTINGS_TABLE=boat-listings
USERS_TABLE=boat-users

# AWS Configuration
AWS_REGION=us-east-1

# API Gateway (automatically provided)
# JWT claims available in event.requestContext.authorizer.claims
```

### Database Schema Requirements
```typescript
// Listings Table
{
  TableName: "boat-listings",
  KeySchema: [
    { AttributeName: "id", KeyType: "HASH" }
  ],
  AttributeDefinitions: [
    { AttributeName: "id", AttributeType: "S" },
    { AttributeName: "ownerId", AttributeType: "S" }
  ],
  GlobalSecondaryIndexes: [
    {
      IndexName: "OwnerIndex",
      KeySchema: [
        { AttributeName: "ownerId", KeyType: "HASH" }
      ],
      Projection: { ProjectionType: "ALL" }
    }
  ]
}
```

## Integration Points

### API Gateway Integration
- **JWT Authorizer**: Automatic token validation
- **CORS Configuration**: Cross-origin request support
- **Request Validation**: Basic format validation
- **Response Transformation**: Consistent error format

### Media Service Integration
- **Image URLs**: References to uploaded images
- **Thumbnail Generation**: Automatic thumbnail creation
- **Video Support**: Optional video content

### Search Service Integration
- **Listing Indexing**: New listings available for search
- **Status Updates**: Search index synchronization
- **Filtering Support**: Search-compatible data structure

This Listing Service provides robust, secure, and scalable boat listing management with comprehensive validation, owner-based access control, and efficient data operations for the HarborList platform.