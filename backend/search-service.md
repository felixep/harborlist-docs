# Search Service Lambda Function Documentation

## Overview

The Search Service is a Lambda function that provides comprehensive search and filtering capabilities for boat listings on the HarborList platform. It implements an in-memory filtering approach that allows users to search by text, location, boat specifications, and price ranges with flexible query parameters.

## Architecture

### Function Structure
- **Runtime**: Node.js 18 with TypeScript
- **Handler**: `backend/src/search/index.ts`
- **Dependencies**: AWS SDK v3, shared utilities, database service
- **Search Strategy**: In-memory filtering of retrieved listings
- **Authentication**: Not required (public search functionality)

### Design Approach
- **Simplicity**: In-memory filtering for straightforward implementation
- **Flexibility**: Multiple filter criteria can be combined
- **Performance**: Suitable for moderate dataset sizes (up to 1000 listings)
- **Scalability**: Can be enhanced with dedicated search services (ElasticSearch, etc.)

## API Endpoint

### POST `/search`
**Purpose**: Search and filter boat listings based on multiple criteria

**Authentication**: Not required (public endpoint)

**Request Body**:
```typescript
{
  // Text search
  query?: string; // Search in title and description
  
  // Price filtering
  priceRange?: {
    min?: number; // Minimum price in USD
    max?: number; // Maximum price in USD
  };
  
  // Location filtering
  location?: {
    state?: string; // Filter by state/province
    radius?: number; // Future: radius-based search
    coordinates?: { // Future: GPS-based search
      lat: number;
      lon: number;
    };
  };
  
  // Boat type filtering
  boatType?: string[]; // Array of boat types to include
  
  // Year filtering
  yearRange?: {
    min?: number; // Minimum manufacturing year
    max?: number; // Maximum manufacturing year
  };
  
  // Length filtering
  lengthRange?: {
    min?: number; // Minimum length in feet
    max?: number; // Maximum length in feet
  };
  
  // Feature filtering (future enhancement)
  features?: string[]; // Array of required features
  
  // Pagination
  limit?: number; // Results per page (default: 20)
  offset?: number; // Starting position (default: 0)
}
```

**Response**:
```typescript
{
  results: Listing[]; // Array of matching listings
  total: number; // Total number of matches
  limit: number; // Applied limit
  offset: number; // Applied offset
}
```

**Example Request**:
```json
{
  "query": "sailboat",
  "priceRange": {
    "min": 10000,
    "max": 100000
  },
  "location": {
    "state": "Florida"
  },
  "yearRange": {
    "min": 2000
  },
  "lengthRange": {
    "min": 25,
    "max": 40
  },
  "limit": 10,
  "offset": 0
}
```

**Example Response**:
```json
{
  "results": [
    {
      "listingId": "listing-123",
      "title": "Beautiful 32ft Sailboat",
      "price": 45000,
      "location": {
        "city": "Miami",
        "state": "Florida"
      },
      "boatDetails": {
        "type": "Sailboat",
        "year": 2005,
        "length": 32
      }
      // ... other listing fields
    }
  ],
  "total": 15,
  "limit": 10,
  "offset": 0
}
```

## Search Implementation

### In-Memory Filtering Strategy

The service implements a multi-stage filtering approach:

1. **Data Retrieval**: Fetch all active listings from database
2. **Sequential Filtering**: Apply each filter criterion in sequence
3. **Pagination**: Apply offset and limit to final results

```typescript
// Get all listings (up to 1000)
const { listings: allListings } = await db.getListings(1000);
let filteredListings = allListings;

// Apply filters sequentially
if (searchParams.query) {
  // Text search filter
}
if (searchParams.location?.state) {
  // Location filter
}
// ... additional filters

// Apply pagination
const paginatedResults = filteredListings.slice(offset, offset + limit);
```

### Text Search Implementation

```typescript
if (searchParams.query) {
  const query = searchParams.query.toLowerCase();
  filteredListings = filteredListings.filter((listing: Listing) => 
    listing.title.toLowerCase().includes(query) ||
    listing.description.toLowerCase().includes(query)
  );
}
```

**Features**:
- **Case-Insensitive**: Converts both query and content to lowercase
- **Partial Matching**: Uses `includes()` for substring matching
- **Multi-Field Search**: Searches both title and description
- **Simple Implementation**: No complex text processing or ranking

### Location Filtering

```typescript
if (searchParams.location?.state) {
  filteredListings = filteredListings.filter((listing: Listing) => 
    listing.location.state === searchParams.location!.state
  );
}
```

**Features**:
- **Exact State Match**: Filters by exact state name
- **Future Enhancements**: Radius-based and GPS coordinate search
- **Case-Sensitive**: Requires exact state name matching

### Boat Type Filtering

```typescript
if (searchParams.boatType && searchParams.boatType.length > 0) {
  filteredListings = filteredListings.filter((listing: Listing) => 
    searchParams.boatType!.includes(listing.boatDetails.type)
  );
}
```

**Features**:
- **Multiple Types**: Supports array of boat types
- **Inclusive OR Logic**: Matches any of the specified types
- **Exact Matching**: Requires exact type name match

### Price Range Filtering

```typescript
if (searchParams.priceRange) {
  filteredListings = filteredListings.filter((listing: Listing) => 
    listing.price >= (searchParams.priceRange!.min || 0) &&
    listing.price <= (searchParams.priceRange!.max || Infinity)
  );
}
```

**Features**:
- **Flexible Bounds**: Min-only, max-only, or both bounds
- **Inclusive Range**: Includes boundary values
- **Default Values**: 0 for min, Infinity for max when not specified

### Year Range Filtering

```typescript
if (searchParams.yearRange) {
  filteredListings = filteredListings.filter((listing: Listing) => 
    listing.boatDetails.year >= (searchParams.yearRange!.min || 0) &&
    listing.boatDetails.year <= (searchParams.yearRange!.max || new Date().getFullYear())
  );
}
```

**Features**:
- **Manufacturing Year**: Filters by boat manufacturing year
- **Current Year Default**: Uses current year as max when not specified
- **Inclusive Range**: Includes boundary years

### Length Range Filtering

```typescript
if (searchParams.lengthRange) {
  filteredListings = filteredListings.filter((listing: Listing) => 
    listing.boatDetails.length >= (searchParams.lengthRange!.min || 0) &&
    listing.boatDetails.length <= (searchParams.lengthRange!.max || Infinity)
  );
}
```

**Features**:
- **Boat Length**: Filters by boat length in feet
- **Flexible Bounds**: Supports min-only, max-only, or both
- **Inclusive Range**: Includes boundary values

## Search Filters Interface

### SearchFilters Type Definition
```typescript
interface SearchFilters {
  query?: string; // Text search query
  priceRange?: {
    min?: number;
    max?: number;
  };
  location?: {
    state?: string;
    radius?: number; // Future enhancement
    coordinates?: {
      lat: number;
      lon: number;
    };
  };
  boatType?: string[]; // Array of boat types
  yearRange?: {
    min?: number;
    max?: number;
  };
  lengthRange?: {
    min?: number;
    max?: number;
  };
  features?: string[]; // Future enhancement
}
```

### Extended Search Parameters
```typescript
interface SearchRequest extends SearchFilters {
  limit?: number; // Pagination limit (default: 20)
  offset?: number; // Pagination offset (default: 0)
}
```

## Pagination Implementation

### Offset-Based Pagination
```typescript
// Apply pagination to filtered results
const limit = searchParams.limit || 20;
const offset = searchParams.offset || 0;
const paginatedResults = filteredListings.slice(offset, offset + limit);

const response = {
  results: paginatedResults,
  total: filteredListings.length, // Total matches before pagination
  limit,
  offset,
};
```

**Features**:
- **Offset-Based**: Simple array slicing for pagination
- **Configurable Limit**: Default 20, customizable per request
- **Total Count**: Returns total matches for pagination UI
- **Zero-Based Offset**: Standard array indexing

### Pagination Calculation
```typescript
// Client-side pagination calculation
const totalPages = Math.ceil(total / limit);
const currentPage = Math.floor(offset / limit) + 1;
const hasNextPage = offset + limit < total;
const hasPreviousPage = offset > 0;
```

## Performance Characteristics

### Current Implementation
- **Data Retrieval**: Single DynamoDB scan (up to 1000 items)
- **Memory Usage**: All listings loaded into Lambda memory
- **Filter Complexity**: O(n) for each filter criterion
- **Pagination**: O(1) array slicing after filtering

### Performance Limitations
- **Dataset Size**: Limited to ~1000 listings for reasonable performance
- **Memory Constraints**: All data loaded into Lambda memory
- **No Indexing**: No search-optimized data structures
- **Sequential Processing**: Filters applied one after another

### Scalability Considerations
```typescript
// Current approach suitable for:
// - Small to medium datasets (< 1000 listings)
// - Simple search requirements
// - Prototype/MVP implementations

// For larger scale, consider:
// - ElasticSearch integration
// - DynamoDB GSI-based filtering
// - Caching strategies
// - Streaming/chunked processing
```

## Error Handling

### Error Response Format
```typescript
// Invalid request body
{
  error: {
    code: 'INVALID_REQUEST',
    message: 'Invalid JSON in request body',
    requestId: 'abc-123'
  }
}

// Method not allowed
{
  error: {
    code: 'METHOD_NOT_ALLOWED',
    message: 'Method GET not allowed',
    requestId: 'abc-123'
  }
}

// Search operation failure
{
  error: {
    code: 'SEARCH_ERROR',
    message: 'Search operation failed',
    requestId: 'abc-123'
  }
}
```

### Error Handling Strategy
```typescript
try {
  // Search operation
} catch (error) {
  console.error('Search error:', error);
  
  if (error instanceof Error) {
    if (error.message.includes('Invalid JSON')) {
      return createErrorResponse(400, 'INVALID_REQUEST', error.message, requestId);
    }
  }
  
  return createErrorResponse(500, 'SEARCH_ERROR', 'Search operation failed', requestId);
}
```

## Query Parameter Processing

### Request Body Parsing
```typescript
const searchParams = parseBody<SearchFilters & { limit?: number; offset?: number }>(event);
```

### Parameter Validation
- **Optional Parameters**: All search criteria are optional
- **Type Safety**: TypeScript interfaces ensure type correctness
- **Default Values**: Sensible defaults for pagination and ranges
- **Flexible Combinations**: Any combination of filters can be applied

### Query Examples

#### Basic Text Search
```json
{
  "query": "yacht"
}
```

#### Price Range Only
```json
{
  "priceRange": {
    "min": 50000,
    "max": 200000
  }
}
```

#### Complex Multi-Filter Search
```json
{
  "query": "fishing boat",
  "location": {
    "state": "California"
  },
  "boatType": ["Fishing Boat", "Sport Fishing"],
  "priceRange": {
    "max": 75000
  },
  "yearRange": {
    "min": 2010
  },
  "lengthRange": {
    "min": 20,
    "max": 35
  },
  "limit": 15,
  "offset": 30
}
```

## Future Enhancements

### Advanced Search Features
```typescript
// Geographic search
location: {
  coordinates: { lat: 25.7617, lon: -80.1918 },
  radius: 50 // miles
}

// Feature-based filtering
features: ["GPS", "Autopilot", "Air Conditioning"]

// Condition-based filtering
condition: ["Excellent", "Good"]

// Sort options
sortBy: "price" | "year" | "length" | "views" | "createdAt"
sortOrder: "asc" | "desc"
```

### Performance Optimizations
```typescript
// ElasticSearch integration
interface ElasticSearchQuery {
  query: {
    bool: {
      must: Array<any>;
      filter: Array<any>;
      should: Array<any>;
    }
  };
  sort: Array<any>;
  from: number;
  size: number;
}

// DynamoDB GSI-based filtering
interface GSIQuery {
  IndexName: string;
  KeyConditionExpression: string;
  FilterExpression?: string;
  ExpressionAttributeValues: Record<string, any>;
}
```

### Search Analytics
```typescript
// Track search patterns
interface SearchAnalytics {
  query: string;
  filters: SearchFilters;
  resultCount: number;
  timestamp: string;
  userId?: string;
  sessionId: string;
}
```

## Integration Points

### Database Service Integration
```typescript
// Current: Simple listing retrieval
const { listings: allListings } = await db.getListings(1000);

// Future: Search-optimized queries
const results = await db.searchListings(searchParams);
```

### Listing Service Integration
- **Data Consistency**: Search reflects current listing status
- **Real-time Updates**: New listings immediately searchable
- **Status Filtering**: Only active listings included in results

### Frontend Integration
```typescript
// React Query integration
const useSearch = (searchParams: SearchFilters) => {
  return useQuery(['search', searchParams], () => 
    searchService.search(searchParams)
  );
};
```

## Monitoring and Analytics

### Key Metrics
- **Search Volume**: Number of search requests per time period
- **Popular Queries**: Most frequently searched terms
- **Filter Usage**: Which filters are used most often
- **Result Patterns**: Average number of results per search
- **Performance**: Response times and error rates

### Logging Strategy
```typescript
// Search request logging
console.log('Search request:', {
  requestId,
  query: searchParams.query,
  filterCount: Object.keys(searchParams).length,
  resultCount: filteredListings.length
});
```

## Environment Configuration

### Required Environment Variables
```bash
# Database configuration (inherited from database service)
LISTINGS_TABLE=boat-listings
AWS_REGION=us-east-1
```

### Performance Tuning
```bash
# Lambda configuration
MEMORY_SIZE=512MB # Sufficient for in-memory filtering
TIMEOUT=30s # Adequate for current implementation
```

This Search Service provides flexible, user-friendly search capabilities with room for future enhancements as the platform scales and search requirements become more sophisticated.