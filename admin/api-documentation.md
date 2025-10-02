# HarborList Admin API Documentation

## Overview

The HarborList Admin API provides comprehensive administrative functionality for managing the boat marketplace platform. This RESTful API enables administrators to manage users, moderate content, monitor system health, and access business analytics.

## Base URLs

- **Production**: `https://api.harbotlist.com/admin`
- **Staging**: `https://api-staging.harbotlist.com/admin`
- **Development**: `https://api-dev.harbotlist.com/admin`

## Authentication

All API endpoints require authentication using JWT Bearer tokens.

### Headers

```http
Authorization: Bearer <your-jwt-token>
Content-Type: application/json
```

### Token Verification

```http
POST /auth/verify
```

Verifies the current admin user's authentication status.

**Response:**
```json
{
  "user": {
    "id": "admin-123",
    "email": "admin@harbotlist.com",
    "name": "Admin User",
    "role": "admin",
    "permissions": ["user_management", "content_moderation"],
    "lastLogin": "2024-09-28T10:00:00Z",
    "mfaEnabled": true
  },
  "sessionInfo": {
    "sessionId": "sess-456",
    "expiresAt": "2024-09-28T18:00:00Z"
  }
}
```

## Rate Limiting

The API implements rate limiting to prevent abuse:

- **Standard endpoints**: 1000 requests per hour per user
- **Authentication endpoints**: 10 requests per minute per IP
- **Search endpoints**: 100 requests per hour per user

Rate limit headers are included in responses:

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

## Error Handling

All errors follow a consistent format:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": ["Email is required", "Password must be at least 8 characters"],
    "requestId": "req-789"
  }
}
```

### Common Error Codes

- `AUTHENTICATION_REQUIRED` (401): Missing or invalid authentication
- `INSUFFICIENT_PERMISSIONS` (403): User lacks required permissions
- `VALIDATION_ERROR` (400): Invalid request parameters
- `RESOURCE_NOT_FOUND` (404): Requested resource doesn't exist
- `RATE_LIMIT_EXCEEDED` (429): Too many requests
- `INTERNAL_SERVER_ERROR` (500): Server error

## Dashboard Endpoints

### Get Dashboard Overview

```http
GET /dashboard
```

Retrieves key platform metrics and system health information.

**Response:**
```json
{
  "metrics": {
    "totalUsers": 1250,
    "activeUsers": 892,
    "newUsersToday": 15,
    "totalListings": 342,
    "activeListings": 298,
    "pendingModeration": 8,
    "flaggedListings": 3
  },
  "systemHealth": {
    "status": "healthy",
    "services": [
      {
        "name": "api",
        "status": "up",
        "responseTime": 145,
        "lastCheck": "2024-09-28T10:00:00Z"
      },
      {
        "name": "database",
        "status": "up",
        "responseTime": 23,
        "lastCheck": "2024-09-28T10:00:00Z"
      }
    ]
  },
  "lastUpdated": "2024-09-28T10:00:00Z"
}
```

## User Management Endpoints

### List Users

```http
GET /users?page=1&limit=20&search=john&status=active&role=user
```

Retrieves a paginated list of platform users with filtering options.

**Query Parameters:**
- `page` (integer): Page number (default: 1)
- `limit` (integer): Items per page (1-100, default: 20)
- `search` (string): Search term for email or name
- `status` (string): Filter by status (`active`, `suspended`, `banned`, `pending_verification`)
- `role` (string): Filter by role (`user`, `admin`, `moderator`, `support`)

**Response:**
```json
{
  "users": [
    {
      "id": "user-123",
      "name": "John Doe",
      "email": "john.doe@example.com",
      "status": "active",
      "role": "user",
      "createdAt": "2024-01-15T10:30:00Z",
      "lastLogin": "2024-09-28T08:15:00Z",
      "mfaEnabled": false
    }
  ],
  "total": 1250,
  "page": 1,
  "totalPages": 63
}
```

### Get User Details

```http
GET /users/{userId}
```

Retrieves detailed information about a specific user.

**Response:**
```json
{
  "user": {
    "id": "user-123",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "status": "active",
    "role": "user",
    "createdAt": "2024-01-15T10:30:00Z",
    "lastLogin": "2024-09-28T08:15:00Z",
    "mfaEnabled": false
  },
  "activity": [
    {
      "id": "activity-456",
      "action": "login",
      "timestamp": "2024-09-28T08:15:00Z",
      "ipAddress": "192.168.1.100",
      "userAgent": "Mozilla/5.0...",
      "details": {
        "loginMethod": "email"
      }
    }
  ],
  "sessions": [
    {
      "sessionId": "sess-789",
      "deviceId": "device-101",
      "ipAddress": "192.168.1.100",
      "issuedAt": "2024-09-28T08:15:00Z",
      "expiresAt": "2024-09-29T08:15:00Z",
      "isActive": true
    }
  ]
}
```

### Update User Status

```http
PUT /users/{userId}/status
```

Updates the status of a specific user (suspend, activate, ban).

**Request Body:**
```json
{
  "status": "suspended",
  "reason": "Terms of service violation",
  "duration": 7,
  "notes": "Inappropriate behavior in messages"
}
```

**Response:**
```json
{
  "message": "User status updated successfully",
  "userId": "user-123",
  "newStatus": "suspended",
  "reason": "Terms of service violation"
}
```

## Content Moderation Endpoints

### Get Flagged Listings

```http
GET /listings/flagged?page=1&limit=20&reason=inappropriate_content&priority=high
```

Retrieves flagged listings requiring moderation.

**Query Parameters:**
- `page` (integer): Page number
- `limit` (integer): Items per page
- `reason` (string): Filter by flag reason
- `priority` (string): Filter by priority (`low`, `medium`, `high`)
- `status` (string): Filter by status (`flagged`, `under_review`, `resolved`)

**Response:**
```json
{
  "listings": [
    {
      "id": "listing-123",
      "title": "2020 Sea Ray Sundancer",
      "seller": {
        "id": "user-456",
        "name": "Bob Wilson",
        "email": "bob.wilson@example.com"
      },
      "flags": [
        {
          "id": "flag-789",
          "reason": "inappropriate_content",
          "description": "Contains inappropriate language",
          "reportedBy": "user-101",
          "reportedAt": "2024-09-27T10:00:00Z",
          "priority": "medium"
        }
      ],
      "status": "flagged",
      "createdAt": "2024-09-25T12:00:00Z",
      "price": 85000,
      "location": "Miami, FL"
    }
  ],
  "total": 15,
  "page": 1,
  "totalPages": 1
}
```

### Get Listing Details

```http
GET /listings/{listingId}
```

Retrieves detailed information about a specific listing for moderation.

**Response:**
```json
{
  "id": "listing-123",
  "title": "2020 Sea Ray Sundancer",
  "description": "Beautiful boat in excellent condition...",
  "price": 85000,
  "location": "Miami, FL",
  "seller": {
    "id": "user-456",
    "name": "Bob Wilson",
    "email": "bob.wilson@example.com"
  },
  "images": [
    "https://example.com/boat1.jpg",
    "https://example.com/boat2.jpg"
  ],
  "specifications": {
    "year": 2020,
    "make": "Sea Ray",
    "model": "Sundancer",
    "length": "35 ft",
    "engine": "Twin 350hp"
  },
  "flags": [
    {
      "id": "flag-789",
      "reason": "inappropriate_content",
      "description": "Contains inappropriate language",
      "reportedBy": "user-101",
      "reportedAt": "2024-09-27T10:00:00Z"
    }
  ],
  "status": "flagged",
  "createdAt": "2024-09-25T12:00:00Z",
  "views": 245,
  "inquiries": 8
}
```

### Moderate Listing

```http
PUT /listings/{listingId}/moderate
```

Makes a moderation decision on a flagged listing.

**Request Body:**
```json
{
  "action": "approve",
  "reason": "Content reviewed and approved",
  "notes": "Minor language issue resolved by seller",
  "notifyUser": true
}
```

**Actions:**
- `approve`: Approve the listing
- `reject`: Reject and remove the listing
- `request_changes`: Request specific changes from seller

**Response:**
```json
{
  "message": "Listing moderated successfully",
  "listingId": "listing-123",
  "action": "approve",
  "moderatedBy": "admin-123",
  "moderatedAt": "2024-09-28T10:00:00Z"
}
```

### Bulk Moderate Listings

```http
POST /listings/bulk-moderate
```

Applies moderation actions to multiple listings.

**Request Body:**
```json
{
  "listingIds": ["listing-123", "listing-456", "listing-789"],
  "action": "approve",
  "reason": "Bulk approval after review",
  "notifyUsers": true
}
```

**Response:**
```json
{
  "message": "Bulk moderation completed",
  "processed": 3,
  "failed": 0,
  "results": [
    {
      "listingId": "listing-123",
      "status": "success",
      "action": "approve"
    },
    {
      "listingId": "listing-456",
      "status": "success",
      "action": "approve"
    },
    {
      "listingId": "listing-789",
      "status": "success",
      "action": "approve"
    }
  ]
}
```

## Analytics Endpoints

### Get Analytics Overview

```http
GET /analytics/overview?startDate=2024-09-01&endDate=2024-09-28&granularity=day
```

Retrieves comprehensive analytics data.

**Query Parameters:**
- `startDate` (string): Start date (ISO 8601)
- `endDate` (string): End date (ISO 8601)
- `granularity` (string): Data granularity (`hour`, `day`, `week`, `month`)

**Response:**
```json
{
  "userMetrics": {
    "totalUsers": 1250,
    "newUsers": 45,
    "activeUsers": 892,
    "userGrowthRate": 12.5
  },
  "listingMetrics": {
    "totalListings": 342,
    "newListings": 18,
    "activeListings": 298,
    "listingGrowthRate": 8.3
  },
  "revenueMetrics": {
    "totalRevenue": 125000.50,
    "revenueToday": 2450.75,
    "averageTransactionValue": 1250.00,
    "revenueGrowthRate": 15.2
  },
  "engagementMetrics": {
    "totalViews": 45000,
    "totalInquiries": 1200,
    "conversionRate": 2.67,
    "averageTimeOnSite": 420
  },
  "timeRange": {
    "startDate": "2024-09-01T00:00:00Z",
    "endDate": "2024-09-28T23:59:59Z",
    "granularity": "day"
  }
}
```

### Get User Analytics

```http
GET /analytics/users?startDate=2024-09-01&endDate=2024-09-28
```

Retrieves detailed user analytics.

**Response:**
```json
{
  "registrationTrends": [
    {
      "date": "2024-09-01",
      "newUsers": 12,
      "totalUsers": 1205
    },
    {
      "date": "2024-09-02",
      "newUsers": 8,
      "totalUsers": 1213
    }
  ],
  "userSegmentation": {
    "byRole": {
      "user": 1180,
      "premium": 65,
      "admin": 5
    },
    "byStatus": {
      "active": 1200,
      "suspended": 45,
      "banned": 5
    }
  },
  "geographicDistribution": [
    {
      "country": "United States",
      "users": 850,
      "percentage": 68.0
    },
    {
      "country": "Canada",
      "users": 200,
      "percentage": 16.0
    }
  ]
}
```

### Export Analytics Data

```http
POST /analytics/export
```

Exports analytics data in various formats.

**Request Body:**
```json
{
  "dataType": "users",
  "format": "csv",
  "startDate": "2024-09-01",
  "endDate": "2024-09-28",
  "filters": {
    "status": "active",
    "role": "user"
  }
}
```

**Response:**
```json
{
  "exportId": "export-123",
  "status": "processing",
  "downloadUrl": null,
  "estimatedCompletion": "2024-09-28T10:05:00Z"
}
```

## System Monitoring Endpoints

### Get System Health

```http
GET /system/health
```

Retrieves current system health status.

**Response:**
```json
{
  "overall": "healthy",
  "services": {
    "api": {
      "status": "healthy",
      "responseTime": 145,
      "uptime": 99.98,
      "lastCheck": "2024-09-28T10:00:00Z"
    },
    "database": {
      "status": "healthy",
      "connections": 12,
      "maxConnections": 100,
      "queryTime": 23,
      "lastCheck": "2024-09-28T10:00:00Z"
    },
    "cache": {
      "status": "healthy",
      "hitRate": 0.85,
      "memoryUsage": 0.65,
      "lastCheck": "2024-09-28T10:00:00Z"
    }
  },
  "metrics": {
    "cpuUsage": 0.35,
    "memoryUsage": 0.68,
    "diskUsage": 0.45,
    "networkLatency": 12
  },
  "alerts": []
}
```

### Get Performance Metrics

```http
GET /system/metrics?timeRange=24h&service=api
```

Retrieves detailed performance metrics.

**Query Parameters:**
- `timeRange` (string): Time range (`1h`, `6h`, `24h`, `7d`, `30d`)
- `service` (string): Specific service to monitor
- `metric` (string): Specific metric type

**Response:**
```json
{
  "service": "api",
  "timeRange": "24h",
  "metrics": [
    {
      "timestamp": "2024-09-28T09:00:00Z",
      "responseTime": 142,
      "errorRate": 0.02,
      "requestsPerMinute": 1250,
      "cpuUsage": 0.32,
      "memoryUsage": 0.65
    },
    {
      "timestamp": "2024-09-28T10:00:00Z",
      "responseTime": 145,
      "errorRate": 0.01,
      "requestsPerMinute": 1300,
      "cpuUsage": 0.35,
      "memoryUsage": 0.68
    }
  ],
  "summary": {
    "averageResponseTime": 143.5,
    "averageErrorRate": 0.015,
    "peakRequestsPerMinute": 1450,
    "uptimePercentage": 99.98
  }
}
```

## Audit Log Endpoints

### Get Audit Logs

```http
GET /audit-logs?page=1&limit=50&userId=admin-123&action=user_suspended&startDate=2024-09-01
```

Retrieves audit logs with filtering and pagination.

**Query Parameters:**
- `page` (integer): Page number
- `limit` (integer): Items per page (1-200)
- `userId` (string): Filter by user ID
- `action` (string): Filter by action type
- `resource` (string): Filter by resource type
- `startDate` (string): Start date filter (ISO 8601)
- `endDate` (string): End date filter (ISO 8601)

**Response:**
```json
{
  "logs": [
    {
      "id": "audit-123",
      "userId": "admin-123",
      "userEmail": "admin@harbotlist.com",
      "action": "user_suspended",
      "resource": "user",
      "resourceId": "user-456",
      "details": {
        "reason": "Terms of service violation",
        "duration": "7 days",
        "previousStatus": "active",
        "newStatus": "suspended"
      },
      "ipAddress": "192.168.1.100",
      "userAgent": "Mozilla/5.0...",
      "timestamp": "2024-09-28T10:15:00Z",
      "sessionId": "sess-789"
    }
  ],
  "total": 1250,
  "page": 1,
  "totalPages": 25
}
```

### Search Audit Logs

```http
POST /audit-logs/search
```

Performs advanced search on audit logs.

**Request Body:**
```json
{
  "query": "user suspension",
  "startDate": "2024-09-01T00:00:00Z",
  "endDate": "2024-09-28T23:59:59Z",
  "filters": {
    "action": "user_suspended",
    "userId": "admin-123"
  },
  "page": 1,
  "limit": 50
}
```

**Response:**
```json
{
  "logs": [
    {
      "id": "audit-123",
      "userId": "admin-123",
      "userEmail": "admin@harbotlist.com",
      "action": "user_suspended",
      "resource": "user",
      "resourceId": "user-456",
      "details": {
        "reason": "Terms of service violation"
      },
      "timestamp": "2024-09-28T10:15:00Z"
    }
  ],
  "total": 15,
  "page": 1,
  "totalPages": 1,
  "searchQuery": "user suspension",
  "searchTime": 0.045
}
```

## Session Management Endpoints

### Get Admin Sessions

```http
GET /sessions?page=1&limit=20&status=active&userId=admin-123
```

Retrieves active admin sessions.

**Response:**
```json
{
  "sessions": [
    {
      "sessionId": "sess-123",
      "userId": "admin-123",
      "userEmail": "admin@harbotlist.com",
      "deviceId": "device-456",
      "ipAddress": "192.168.1.100",
      "userAgent": "Mozilla/5.0...",
      "issuedAt": "2024-09-28T08:00:00Z",
      "expiresAt": "2024-09-28T16:00:00Z",
      "lastActivity": "2024-09-28T10:00:00Z",
      "isActive": true
    }
  ],
  "total": 5,
  "page": 1,
  "totalPages": 1
}
```

### Terminate Session

```http
POST /sessions/{sessionId}/terminate
```

Terminates a specific admin session.

**Response:**
```json
{
  "message": "Session terminated successfully",
  "sessionId": "sess-123",
  "terminatedBy": "admin-456",
  "terminatedAt": "2024-09-28T10:00:00Z"
}
```

## Security Monitoring Endpoints

### Get Suspicious Activity

```http
GET /security/suspicious-activity?timeRange=24h&severity=high
```

Retrieves detected suspicious activities.

**Query Parameters:**
- `timeRange` (string): Time range (`1h`, `6h`, `24h`, `7d`, `30d`)
- `severity` (string): Severity filter (`low`, `medium`, `high`, `critical`)
- `type` (string): Activity type filter

**Response:**
```json
{
  "activities": [
    {
      "id": "activity-123",
      "type": "multiple_failed_logins",
      "severity": "high",
      "description": "Multiple failed login attempts from same IP",
      "ipAddress": "192.168.1.200",
      "userEmail": "suspicious@example.com",
      "timestamp": "2024-09-28T09:30:00Z",
      "details": {
        "attemptCount": 15,
        "timeWindow": "5 minutes",
        "blocked": true
      }
    }
  ],
  "total": 8,
  "page": 1,
  "totalPages": 1
}
```

## Support Management Endpoints

### Get Support Tickets

```http
GET /support/tickets?page=1&limit=20&status=open&priority=high
```

Retrieves support tickets with filtering.

**Response:**
```json
{
  "tickets": [
    {
      "id": "ticket-123",
      "subject": "Cannot upload boat images",
      "status": "open",
      "priority": "high",
      "category": "technical",
      "userId": "user-456",
      "userEmail": "user@example.com",
      "assignedTo": "support-789",
      "createdAt": "2024-09-28T09:00:00Z",
      "lastUpdated": "2024-09-28T09:30:00Z",
      "responseCount": 2
    }
  ],
  "total": 25,
  "page": 1,
  "totalPages": 2
}
```

### Update Ticket

```http
PUT /support/tickets/{ticketId}
```

Updates a support ticket.

**Request Body:**
```json
{
  "status": "in_progress",
  "priority": "medium",
  "assignedTo": "support-789",
  "response": "We're investigating this issue and will update you shortly.",
  "internal_notes": "Checked server logs, appears to be upload service issue"
}
```

## Platform Settings Endpoints

### Get Platform Settings

```http
GET /settings/platform
```

Retrieves current platform settings.

**Response:**
```json
{
  "general": {
    "platformName": "HarborList",
    "supportEmail": "support@harbotlist.com",
    "timezone": "UTC",
    "defaultCurrency": "USD"
  },
  "features": {
    "userRegistration": true,
    "listingCreation": true,
    "paymentProcessing": true,
    "searchEnabled": true
  },
  "limits": {
    "maxListingsPerUser": 50,
    "maxImageUploads": 20,
    "maxFileSize": "10MB"
  }
}
```

### Update Platform Settings

```http
PUT /settings/platform
```

Updates platform settings.

**Request Body:**
```json
{
  "general": {
    "supportEmail": "newsupport@harbotlist.com",
    "timezone": "America/New_York"
  },
  "features": {
    "userRegistration": false
  },
  "limits": {
    "maxListingsPerUser": 75
  }
}
```

## Webhooks

The Admin API supports webhooks for real-time notifications of important events.

### Webhook Events

- `user.suspended`: User account suspended
- `user.reactivated`: User account reactivated
- `listing.flagged`: Listing flagged for review
- `listing.approved`: Listing approved by moderator
- `listing.rejected`: Listing rejected by moderator
- `system.alert`: System alert triggered
- `security.incident`: Security incident detected

### Webhook Payload

```json
{
  "event": "user.suspended",
  "timestamp": "2024-09-28T10:00:00Z",
  "data": {
    "userId": "user-123",
    "reason": "Terms violation",
    "suspendedBy": "admin-456",
    "duration": "7 days"
  },
  "signature": "sha256=abc123..."
}
```

## SDK Examples

### JavaScript/Node.js

```javascript
const HarborListAdmin = require('@harborlist/admin-sdk');

const client = new HarborListAdmin({
  baseUrl: 'https://api.harbotlist.com/admin',
  token: 'your-jwt-token'
});

// Get dashboard data
const dashboard = await client.dashboard.getOverview();

// List users
const users = await client.users.list({
  page: 1,
  limit: 20,
  status: 'active'
});

// Suspend user
await client.users.updateStatus('user-123', {
  status: 'suspended',
  reason: 'Terms violation'
});
```

### Python

```python
from harborlist_admin import AdminClient

client = AdminClient(
    base_url='https://api.harbotlist.com/admin',
    token='your-jwt-token'
)

# Get dashboard data
dashboard = client.dashboard.get_overview()

# List users
users = client.users.list(
    page=1,
    limit=20,
    status='active'
)

# Suspend user
client.users.update_status('user-123', {
    'status': 'suspended',
    'reason': 'Terms violation'
})
```

## Testing

### Postman Collection

A comprehensive Postman collection is available for testing all API endpoints:

```json
{
  "info": {
    "name": "HarborList Admin API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "auth": {
    "type": "bearer",
    "bearer": [
      {
        "key": "token",
        "value": "{{admin_token}}",
        "type": "string"
      }
    ]
  },
  "variable": [
    {
      "key": "base_url",
      "value": "https://api.harbotlist.com/admin"
    },
    {
      "key": "admin_token",
      "value": "your-jwt-token"
    }
  ]
}
```

### cURL Examples

```bash
# Get dashboard overview
curl -X GET \
  https://api.harbotlist.com/admin/dashboard \
  -H 'Authorization: Bearer your-jwt-token' \
  -H 'Content-Type: application/json'

# List users
curl -X GET \
  'https://api.harbotlist.com/admin/users?page=1&limit=20&status=active' \
  -H 'Authorization: Bearer your-jwt-token'

# Suspend user
curl -X PUT \
  https://api.harbotlist.com/admin/users/user-123/status \
  -H 'Authorization: Bearer your-jwt-token' \
  -H 'Content-Type: application/json' \
  -d '{
    "status": "suspended",
    "reason": "Terms of service violation"
  }'
```

## Changelog

### Version 1.0.0 (2024-09-28)
- Initial release
- Complete admin functionality
- Authentication and authorization
- User management endpoints
- Content moderation endpoints
- Analytics and reporting
- System monitoring
- Audit logging
- Security monitoring

---

*For technical support, contact: dev@harbotlist.com*
*Last updated: September 28, 2024*