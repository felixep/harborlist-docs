# Admin Service Lambda Function Documentation

## Overview

The Admin Service is a comprehensive Lambda function that provides administrative operations, user management, audit logging, security monitoring, and dashboard metrics for the HarborList platform. It implements enterprise-grade security with role-based access control, adaptive rate limiting, comprehensive audit trails, and real-time system monitoring.

## Architecture

### Function Structure
- **Runtime**: Node.js 18 with TypeScript
- **Handler**: `backend/src/admin-service/index.ts`
- **Dependencies**: AWS SDK v3, middleware composition, versioning support
- **Authentication**: JWT-based with mandatory admin role verification
- **Authorization**: Permission-based access control with granular permissions
- **Security**: Multi-layered middleware with rate limiting and audit logging

### Design Principles
- **Security First**: Comprehensive authentication and authorization
- **Audit Everything**: Complete audit trail for all administrative actions
- **Role-Based Access**: Granular permissions for different admin roles
- **Rate Limiting**: Adaptive rate limiting based on user roles and permissions
- **API Versioning**: Support for multiple API versions with transformation
- **Middleware Composition**: Composable middleware for cross-cutting concerns

## Administrative Roles and Permissions

### Admin Roles
```typescript
enum UserRole {
  SUPER_ADMIN = 'super_admin',    // Full system access
  ADMIN = 'admin',                // Most administrative functions
  MODERATOR = 'moderator',        // Content moderation focus
  SUPPORT = 'support'             // User support functions
}
```

### Admin Permissions
```typescript
enum AdminPermission {
  USER_MANAGEMENT = 'user_management',        // User account operations
  CONTENT_MODERATION = 'content_moderation', // Listing moderation
  FINANCIAL_ACCESS = 'financial_access',     // Financial data access
  SYSTEM_CONFIG = 'system_config',           // System configuration
  ANALYTICS_VIEW = 'analytics_view',         // Analytics and reporting
  AUDIT_LOG_VIEW = 'audit_log_view'         // Audit log access
}
```

### Permission Matrix
| Role | User Mgmt | Content Mod | Financial | System Config | Analytics | Audit Logs |
|------|-----------|-------------|-----------|---------------|-----------|------------|
| SUPER_ADMIN | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| ADMIN | ✓ | ✓ | ✓ | Limited | ✓ | ✓ |
| MODERATOR | Limited | ✓ | ✗ | ✗ | Limited | Limited |
| SUPPORT | Limited | Limited | ✗ | ✗ | Limited | Limited |

## Middleware Architecture

### Middleware Composition
```typescript
// Example endpoint with full middleware stack
return await compose(
  withAdaptiveRateLimit(AdminPermission.USER_MANAGEMENT),
  withAdminAuth([AdminPermission.USER_MANAGEMENT]),
  withAuditLog('VIEW_USERS', 'users')
)(handleGetUsers)(event as AuthenticatedEvent, {});
```

### Authentication Middleware
```typescript
export function withAdminAuth(requiredPermissions?: AdminPermission[]) {
  return function(handler: AuthenticatedHandler) {
    return withAuth(async (event: AuthenticatedEvent, context: any) => {
      // Verify admin role and permissions
      requireAdminRole(event.user, requiredPermissions);
      
      // Log admin access
      await logAdminAction(event.user, 'ADMIN_ACCESS', 'admin_endpoint', {
        path: event.path,
        method: event.httpMethod,
        requiredPermissions
      }, clientInfo);

      return await handler(event, context);
    });
  };
}
```

### Adaptive Rate Limiting
```typescript
const RATE_LIMIT_TIERS = {
  [AdminPermission.USER_MANAGEMENT]: {
    SUPER_ADMIN: { maxRequests: 200, windowMs: 60000 },
    ADMIN: { maxRequests: 150, windowMs: 60000 },
    MODERATOR: { maxRequests: 100, windowMs: 60000 },
    SUPPORT: { maxRequests: 80, windowMs: 60000 }
  },
  // ... other permissions with role-specific limits
};
```

## API Endpoints

### Authentication & Authorization

#### POST `/admin/auth/verify`
**Purpose**: Verify admin authentication and return user permissions

**Authentication**: Required (Admin JWT)
**Rate Limit**: Adaptive based on role

**Response**:
```typescript
{
  user: {
    id: string;
    email: string;
    name: string;
    role: UserRole;
    status: UserStatus;
    permissions: AdminPermission[];
    // ... other safe user fields (no password, secrets)
  };
  permissions: AdminPermission[];
  sessionInfo: {
    sessionId: string;
    deviceId: string;
    expiresAt: string;
  };
}
```

### Dashboard & Metrics

#### GET `/admin/dashboard`
**Purpose**: Get comprehensive dashboard metrics and system overview

**Authentication**: Required (Admin JWT)
**Permissions**: `ANALYTICS_VIEW`
**Rate Limit**: 100 requests/minute

**Response**:
```typescript
{
  metrics: {
    totalUsers: number;
    activeUsers: number;
    newUsersToday: number;
    totalListings: number;
    activeListings: number;
    pendingModeration: number;
    flaggedListings: number;
  };
  systemHealth: {
    status: 'healthy' | 'degraded' | 'unhealthy';
    services: Array<{
      name: string;
      status: string;
      responseTime: number;
      lastCheck: string;
    }>;
    alerts: Array<{
      id: string;
      severity: 'low' | 'medium' | 'high' | 'critical';
      message: string;
      timestamp: string;
    }>;
  };
  lastUpdated: string;
}
```

### User Management

#### GET `/admin/users`
**Purpose**: Get paginated list of users with filtering options

**Authentication**: Required (Admin JWT)
**Permissions**: `USER_MANAGEMENT`
**Rate Limit**: Adaptive (80-200 requests/minute based on role)

**Query Parameters**:
- `page` (number): Page number (default: 1)
- `limit` (number): Results per page (max: 100, default: 20)
- `search` (string): Search by name or email
- `status` (UserStatus): Filter by user status
- `role` (UserRole): Filter by user role

**Response**:
```typescript
{
  users: Array<{
    id: string;
    email: string;
    name: string;
    role: UserRole;
    status: UserStatus;
    emailVerified: boolean;
    mfaEnabled: boolean;
    lastLogin?: string;
    createdAt: string;
    // No sensitive fields
  }>;
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

#### GET `/admin/users/{userId}`
**Purpose**: Get detailed user information including activity and sessions

**Authentication**: Required (Admin JWT)
**Permissions**: `USER_MANAGEMENT`
**Rate Limit**: 100 requests/minute

**Response**:
```typescript
{
  user: {
    // Complete user profile (no sensitive fields)
    id: string;
    email: string;
    name: string;
    role: UserRole;
    status: UserStatus;
    permissions?: AdminPermission[];
    emailVerified: boolean;
    phoneVerified: boolean;
    mfaEnabled: boolean;
    loginAttempts: number;
    lastLogin?: string;
    createdAt: string;
    updatedAt: string;
  };
  activity: {
    loginHistory: Array<{
      timestamp: string;
      ipAddress: string;
      userAgent: string;
      success: boolean;
    }>;
    recentActions: Array<{
      action: string;
      resource: string;
      timestamp: string;
    }>;
  };
  sessions: Array<{
    sessionId: string;
    deviceId: string;
    ipAddress: string;
    userAgent: string;
    issuedAt: string;
    lastActivity: string;
    isActive: boolean;
  }>;
}
```

#### PUT `/admin/users/{userId}/status`
**Purpose**: Update user account status (suspend, ban, activate)

**Authentication**: Required (Admin JWT)
**Permissions**: `USER_MANAGEMENT`
**Rate Limit**: 20 requests/minute

**Request Body**:
```typescript
{
  status: UserStatus; // 'active' | 'suspended' | 'banned' | 'pending_verification'
  reason: string; // Required reason for status change
}
```

**Response**:
```typescript
{
  message: "User status updated successfully";
  userId: string;
  newStatus: UserStatus;
  reason: string;
}
```

### Content Moderation

#### GET `/admin/listings`
**Purpose**: Get listings for moderation with filtering options

**Authentication**: Required (Admin JWT)
**Permissions**: `CONTENT_MODERATION`
**Rate Limit**: 50 requests/minute

**Query Parameters**:
- `page` (number): Page number
- `limit` (number): Results per page (max: 100)
- `status` (string): Filter by listing status
- `flagged` (boolean): Show only flagged listings

#### PUT `/admin/listings/{listingId}/moderate`
**Purpose**: Moderate a listing (approve, reject, request changes)

**Authentication**: Required (Admin JWT)
**Permissions**: `CONTENT_MODERATION`
**Rate Limit**: 30 requests/minute

**Request Body**:
```typescript
{
  action: 'approve' | 'reject' | 'request_changes';
  reason: string; // Required reason for moderation action
  notes?: string; // Optional additional notes
}
```

### Audit Logging

#### GET `/admin/audit-logs`
**Purpose**: Get paginated audit logs with comprehensive filtering

**Authentication**: Required (Admin JWT)
**Permissions**: `AUDIT_LOG_VIEW`
**Rate Limit**: Adaptive (30-100 requests/minute based on role)

**Query Parameters**:
- `page` (number): Page number
- `limit` (number): Results per page (max: 200)
- `userId` (string): Filter by user ID
- `action` (string): Filter by action type
- `resource` (string): Filter by resource type
- `startDate` (string): Start date filter (ISO 8601)
- `endDate` (string): End date filter (ISO 8601)
- `sortBy` (string): Sort field (default: 'timestamp')
- `sortOrder` (string): Sort order ('asc' | 'desc', default: 'desc')

**Response**:
```typescript
{
  auditLogs: Array<{
    id: string;
    userId: string;
    userEmail: string;
    action: string;
    resource: string;
    resourceId?: string;
    details: Record<string, any>;
    ipAddress: string;
    userAgent: string;
    timestamp: string;
    sessionId: string;
  }>;
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
  filters: {
    userId?: string;
    action?: string;
    resource?: string;
    startDate?: string;
    endDate?: string;
  };
}
```

#### POST `/admin/audit-logs/export`
**Purpose**: Export audit logs in CSV or JSON format

**Authentication**: Required (Admin JWT)
**Permissions**: `AUDIT_LOG_VIEW`
**Rate Limit**: 5 requests/minute (low limit for resource-intensive operation)

**Request Body**:
```typescript
{
  format: 'csv' | 'json';
  startDate: string; // Required (ISO 8601)
  endDate: string; // Required (ISO 8601)
  userId?: string; // Optional filter
  action?: string; // Optional filter
  resource?: string; // Optional filter
}
```

**Validation**:
- Date range cannot exceed 90 days
- Start date must be before end date

**Response**:
```typescript
{
  message: "Audit logs exported successfully";
  data: string; // CSV content or JSON string
  filename: string; // Suggested filename
  recordCount: number; // Number of records exported
}
```

#### GET `/admin/audit-logs/stats`
**Purpose**: Get audit log statistics and trends

**Authentication**: Required (Admin JWT)
**Permissions**: `AUDIT_LOG_VIEW`
**Rate Limit**: 20 requests/minute

**Query Parameters**:
- `timeRange` (string): Time range ('1h', '24h', '7d', '30d', default: '7d')

**Response**:
```typescript
{
  summary: {
    totalEvents: number;
    uniqueUsers: number;
    topActions: Array<{ action: string; count: number; }>;
    topResources: Array<{ resource: string; count: number; }>;
  };
  timeline: Array<{
    timestamp: string;
    eventCount: number;
    uniqueUsers: number;
  }>;
  trends: {
    eventGrowth: number; // Percentage change
    userActivityGrowth: number;
    errorRate: number;
  };
}
```

### Session Management

#### GET `/admin/sessions`
**Purpose**: Get active admin sessions for monitoring

**Authentication**: Required (Admin JWT)
**Permissions**: `USER_MANAGEMENT`
**Rate Limit**: 30 requests/minute

#### GET `/admin/sessions/current`
**Purpose**: Get current session information

**Authentication**: Required (Admin JWT)
**Rate Limit**: 60 requests/minute

#### POST `/admin/sessions/{sessionId}/terminate`
**Purpose**: Terminate a specific session

**Authentication**: Required (Admin JWT)
**Permissions**: `USER_MANAGEMENT`
**Rate Limit**: 10 requests/minute

#### GET `/admin/sessions/user/{userId}`
**Purpose**: Get all sessions for a specific user

**Authentication**: Required (Admin JWT)
**Permissions**: `USER_MANAGEMENT`
**Rate Limit**: 30 requests/minute

#### POST `/admin/sessions/user/{userId}/terminate-all`
**Purpose**: Terminate all sessions for a specific user

**Authentication**: Required (Admin JWT)
**Permissions**: `USER_MANAGEMENT`
**Rate Limit**: 5 requests/minute

### Security Monitoring

#### GET `/admin/security/suspicious-activity`
**Purpose**: Get detected suspicious activities and security events

**Authentication**: Required (Admin JWT)
**Permissions**: `AUDIT_LOG_VIEW`
**Rate Limit**: 20 requests/minute

**Response**:
```typescript
{
  suspiciousActivities: Array<{
    id: string;
    type: 'multiple_failed_logins' | 'unusual_location' | 'rate_limit_exceeded' | 'privilege_escalation';
    userId?: string;
    ipAddress: string;
    description: string;
    severity: 'low' | 'medium' | 'high' | 'critical';
    timestamp: string;
    status: 'new' | 'investigating' | 'resolved' | 'false_positive';
  }>;
  summary: {
    total: number;
    byType: Record<string, number>;
    bySeverity: Record<string, number>;
  };
}
```

#### GET `/admin/security/login-attempts`
**Purpose**: Get login attempt statistics and patterns

**Authentication**: Required (Admin JWT)
**Permissions**: `AUDIT_LOG_VIEW`
**Rate Limit**: 30 requests/minute

#### GET `/admin/security/failed-logins`
**Purpose**: Get failed login attempts for security analysis

**Authentication**: Required (Admin JWT)
**Permissions**: `AUDIT_LOG_VIEW`
**Rate Limit**: 30 requests/minute

#### GET `/admin/security/ip-analysis`
**Purpose**: Get IP address analysis and geolocation data

**Authentication**: Required (Admin JWT)
**Permissions**: `AUDIT_LOG_VIEW`
**Rate Limit**: 20 requests/minute

#### GET `/admin/security/rate-limit-violations`
**Purpose**: Get rate limit violations and potential abuse

**Authentication**: Required (Admin JWT)
**Permissions**: `AUDIT_LOG_VIEW`
**Rate Limit**: 20 requests/minute

### System Health & Monitoring

#### GET `/admin/system/health`
**Purpose**: Get comprehensive system health status

**Authentication**: Required (Admin JWT)
**Rate Limit**: 60 requests/minute

**Response**:
```typescript
{
  overall: {
    status: 'healthy' | 'degraded' | 'unhealthy';
    score: number; // 0-100 health score
    lastCheck: string;
  };
  services: Array<{
    name: string;
    status: 'healthy' | 'degraded' | 'unhealthy';
    responseTime: number;
    uptime: number;
    lastCheck: string;
    details?: Record<string, any>;
  }>;
  infrastructure: {
    database: {
      status: string;
      connectionPool: number;
      queryLatency: number;
    };
    storage: {
      status: string;
      availableSpace: number;
      usedSpace: number;
    };
    cache: {
      status: string;
      hitRate: number;
      memoryUsage: number;
    };
  };
}
```

#### GET `/admin/system/metrics`
**Purpose**: Get detailed system performance metrics

**Authentication**: Required (Admin JWT)
**Rate Limit**: 60 requests/minute

**Query Parameters**:
- `timeRange` (string): Time range ('1h', '6h', '24h', '7d')
- `granularity` (string): Data granularity ('minute', 'hour', 'day')

#### GET `/admin/system/alerts`
**Purpose**: Get active system alerts and notifications

**Authentication**: Required (Admin JWT)
**Rate Limit**: 60 requests/minute

#### POST `/admin/system/alerts/{alertId}/acknowledge`
**Purpose**: Acknowledge a system alert

**Authentication**: Required (Admin JWT)
**Rate Limit**: 30 requests/minute

#### POST `/admin/system/alerts/{alertId}/resolve`
**Purpose**: Mark a system alert as resolved

**Authentication**: Required (Admin JWT)
**Rate Limit**: 30 requests/minute

### Analytics & Reporting

#### GET `/admin/analytics/users`
**Purpose**: Get user analytics and growth metrics

**Authentication**: Required (Admin JWT)
**Permissions**: `ANALYTICS_VIEW`
**Rate Limit**: 60 requests/minute

#### GET `/admin/analytics/listings`
**Purpose**: Get listing analytics and performance metrics

**Authentication**: Required (Admin JWT)
**Permissions**: `ANALYTICS_VIEW`
**Rate Limit**: 60 requests/minute

#### GET `/admin/analytics/engagement`
**Purpose**: Get user engagement and platform usage analytics

**Authentication**: Required (Admin JWT)
**Permissions**: `ANALYTICS_VIEW`
**Rate Limit**: 60 requests/minute

#### GET `/admin/analytics/geographic`
**Purpose**: Get geographic distribution and location analytics

**Authentication**: Required (Admin JWT)
**Permissions**: `ANALYTICS_VIEW`
**Rate Limit**: 60 requests/minute

## Security Features

### Multi-Layer Authentication
1. **JWT Token Validation**: Verify token signature and expiration
2. **Admin Role Verification**: Ensure user has admin privileges
3. **Permission Checking**: Validate specific admin permissions
4. **Session Validation**: Verify active session in database
5. **Account Status Check**: Ensure admin account is active

### Comprehensive Audit Logging
```typescript
// Every admin action is logged with:
interface AuditLog {
  id: string;
  userId: string; // Admin performing action
  userEmail: string;
  action: string; // Specific action performed
  resource: string; // Resource type affected
  resourceId?: string; // Specific resource ID
  details: Record<string, any>; // Action-specific details
  ipAddress: string; // Source IP
  userAgent: string; // Client information
  timestamp: string; // ISO 8601 timestamp
  sessionId: string; // Session identifier
}
```

### Rate Limiting Strategy
- **Role-Based Limits**: Different limits for different admin roles
- **Permission-Based Limits**: Specific limits for sensitive operations
- **Adaptive Thresholds**: Higher limits for higher-privilege roles
- **Violation Logging**: All rate limit violations are logged
- **Response Headers**: Rate limit status in response headers

### Input Validation & Sanitization
```typescript
// Comprehensive validation functions
export const validators = {
  email: (email: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email),
  uuid: (id: string) => /^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i.test(id),
  required: (value: any) => value !== null && value !== undefined && value !== '',
  noScripts: (value: string) => !/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi.test(value),
  noSqlInjection: (value: string) => /* SQL injection pattern checks */,
  safeHtml: (value: string) => /* Dangerous HTML tag checks */,
  // ... additional validators
};
```

## Error Handling

### Standardized Error Responses
```typescript
// Authentication/Authorization Errors
{ error: { code: 'UNAUTHORIZED', message: 'Authentication required' } }
{ error: { code: 'FORBIDDEN', message: 'Insufficient permissions' } }
{ error: { code: 'ACCOUNT_INACTIVE', message: 'Admin account is not active' } }

// Rate Limiting Errors
{ error: { code: 'RATE_LIMIT_EXCEEDED', message: 'Too many requests' } }

// Validation Errors
{ error: { code: 'VALIDATION_ERROR', message: 'Invalid input data' } }
{ error: { code: 'MISSING_USER_ID', message: 'User ID is required' } }

// Resource Errors
{ error: { code: 'USER_NOT_FOUND', message: 'User not found' } }
{ error: { code: 'NOT_FOUND', message: 'Endpoint not found' } }

// System Errors
{ error: { code: 'INTERNAL_ERROR', message: 'Internal server error' } }
```

### Error Handling Strategy
1. **Input Validation**: Validate all inputs before processing
2. **Permission Checks**: Verify permissions before operations
3. **Resource Validation**: Ensure resources exist before modification
4. **Database Error Handling**: Handle DynamoDB operation failures
5. **Audit Error Logging**: Log all errors for security monitoring
6. **Generic Fallback**: Catch unexpected errors gracefully

## API Versioning

### Version Support
```typescript
// API versioning with transformation support
export const handler = withApiVersioning(async (event: APIGatewayProxyEvent, version: string) => {
  // Version-specific handling
  const response = await handleRequest(event);
  
  // Apply version-specific transformations
  return applyVersionTransformation(response, version);
});
```

### Version Extraction
- **Header-Based**: `X-API-Version: v1`
- **Path-Based**: `/v1/admin/users`
- **Query Parameter**: `?version=v1`
- **Default Version**: Falls back to latest stable version

## Performance Optimization

### Database Operations
- **Connection Reuse**: DynamoDB client reuse across invocations
- **Batch Operations**: Batch writes for bulk operations
- **Efficient Queries**: Optimized query patterns with GSI usage
- **Pagination**: Proper pagination for large result sets

### Memory Management
- **Middleware Composition**: Efficient middleware chaining
- **Request Caching**: In-memory caching for rate limiting
- **Resource Cleanup**: Proper cleanup of resources

### Response Optimization
- **Data Filtering**: Remove sensitive fields from responses
- **Compression**: Efficient JSON serialization
- **Caching Headers**: Appropriate cache control headers

## Monitoring & Observability

### CloudWatch Metrics
```typescript
// Custom metrics tracked
{
  'AdminService/RequestCount': requestCount,
  'AdminService/ErrorRate': errorRate,
  'AdminService/ResponseTime': responseTime,
  'AdminService/RateLimitViolations': violationCount,
  'AdminService/PermissionDenials': denialCount
}
```

### Logging Strategy
- **Structured Logging**: JSON-formatted logs for parsing
- **Request Tracing**: Request ID tracking across operations
- **Performance Logging**: Response time and operation metrics
- **Security Logging**: Authentication and authorization events
- **Error Logging**: Detailed error information with context

### Alerting
- **High Error Rates**: Alert on elevated error rates
- **Security Events**: Alert on suspicious activities
- **Performance Degradation**: Alert on slow response times
- **Rate Limit Violations**: Alert on potential abuse

## Environment Configuration

### Required Environment Variables
```bash
# Database Tables
USERS_TABLE=boat-users
LISTINGS_TABLE=boat-listings
SESSIONS_TABLE=boat-sessions
AUDIT_LOGS_TABLE=boat-audit-logs
LOGIN_ATTEMPTS_TABLE=boat-login-attempts

# Security Configuration
JWT_SECRET=<strong-secret-key>
JWT_REFRESH_SECRET=<different-strong-secret-key>

# Feature Flags
ARCHIVE_BEFORE_DELETE=true
ARCHIVE_BUCKET=audit-logs-archive

# AWS Configuration
AWS_REGION=us-east-1
```

### Lambda Configuration
```typescript
// Recommended Lambda settings
{
  memorySize: 512, // MB
  timeout: 30, // seconds
  reservedConcurrency: 10, // Limit concurrent executions
  environment: {
    NODE_OPTIONS: '--enable-source-maps'
  }
}
```

This Admin Service provides enterprise-grade administrative capabilities with comprehensive security, monitoring, and audit features for the HarborList platform.