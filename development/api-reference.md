# Development Api Reference

## Overview

This document describes the API endpoints and interfaces for the HarborList platform.

## Base URL

Development: `http://localhost:3000/api`

## Authentication

API endpoints require authentication using JWT tokens. Include the token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

## Endpoints

### Core Endpoints

#### GET /health
Health check endpoint to verify API status.

**Response:**
```json
{
  "status": "ok",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

#### GET /api/listings
Retrieve boat listings.

**Parameters:**
- `page` (optional): Page number for pagination
- `limit` (optional): Number of items per page

**Response:**
```json
{
  "listings": [],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 0
  }
}
```

## Error Handling

The API returns standard HTTP status codes and error responses in the following format:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message"
  }
}
```

## Rate Limiting

API requests are rate limited to prevent abuse. Current limits:
- 100 requests per minute per IP address
- 1000 requests per hour per authenticated user

## Development Notes

This API is currently implemented and available for development use. Refer to the backend service documentation for implementation details.
