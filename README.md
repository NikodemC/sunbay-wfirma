# Gepard Order Processing API Endpoint

## Overview

The `[HttpPost("order/process")]` endpoint is designed to process orders from the Gepard system and automatically create invoices in the wFirma accounting system. This endpoint provides a seamless integration between order management and invoice generation.

## Endpoint Details

- **URL**: `POST /api/gepard/order/process`
- **Content-Type**: `application/json`
- **Authentication**: API Key required
- **Rate Limiting**: 60 requests per minute per client

## Authentication

All requests to this endpoint require a valid API key to be included in the request headers.

### Header Requirements

```
x-api-key: YOUR_API_KEY_HERE
```

**Note**: The API key was provided separately by the Sunbay team. Please ensure this key is kept secure and not shared publicly.

## Request Format

### Current Request Structure

The current implementation expects a `ProcessOrderCommand` with the following structure:

```json
{
  "contractorId": "string"
}
```

### Future Request Structure

**Important**: Gepard, which is implementing this integration will provide their own `ProcessOrderCommand` structure. The current structure shown above is a placeholder and will be replaced with the company's specific requirements.

The final request structure will be determined by Gepard and may include additional fields such as:
- Order details
- Product information
- Quantities
- Pricing information
- Any other business-specific data

## Response Format

### Success Response (200 OK)

```json
{
  "success": true,
  "invoiceId": "string",
  "invoiceNumber": "string"
}
```

### Error Responses

#### Validation Error (422 Unprocessable Entity)
```json
{
  "success": false,
  "errorCode": "VALIDATION_ERROR",
  "errorMessage": "ContractorId is required"
}
```

#### Contractor Not Found (404 Not Found)
```json
{
  "success": false,
  "errorCode": "CONTRACTOR_NOT_FOUND",
  "errorMessage": "Contractor with ID '123' was not found"
}
```

#### Rate Limit Exceeded (429 Too Many Requests)
```json
{
  "error": "Rate limit exceeded",
  "message": "Rate limit exceeded. Maximum 60 requests per minute allowed.",
  "limitType": "PerMinute",
  "retryAfter": 45
}
```

#### Service Unavailable (503 Service Unavailable)
```json
{
  "success": false,
  "errorCode": "SERVICE_UNAVAILABLE",
  "errorMessage": "wFirma service is temporarily unavailable"
}
```

#### Internal Server Error (500 Internal Server Error)
```json
{
  "success": false,
  "errorCode": "INTERNAL_ERROR",
  "errorMessage": "An unexpected error occurred while processing the request"
}
```

## HTTP Status Codes

| Status Code | Description | When Returned |
|-------------|-------------|----------------|
| 200 | OK | Order processed successfully, invoice created |
| 400 | Bad Request | Invalid request format or missing required fields |
| 401 | Unauthorized | Missing or invalid API key |
| 404 | Not Found | Contractor not found in the system |
| 422 | Unprocessable Entity | Validation errors in the request data |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected server error |
| 503 | Service Unavailable | wFirma service temporarily unavailable |

## Rate Limiting

The endpoint implements rate limiting to ensure fair usage and system stability:

- **Limit**: 60 requests per minute per client
- **Identification**: Based on API key or IP address
- **Headers**: Rate limit information is included in response headers
  - `X-RateLimit-Limit-Minute`: Maximum requests per minute
  - `X-RateLimit-Remaining-Minute`: Remaining requests in current minute
  - `Retry-After`: Seconds to wait when rate limit is exceeded

### Rate Limit Statistics

You can check your current rate limit usage by calling:
```
GET /api/gepard/rate-limit/stats
```

This endpoint returns:
```json
{
  "clientIdentifier": "api-key-YOUR_API_KEY",
  "currentUsage": {
    "requestsInLastMinute": 15,
    "totalRequests": 1250
  },
  "limits": {
    "maxRequestsPerMinute": 60
  },
  "remaining": {
    "requestsThisMinute": 45
  }
}
```

## Business Logic

### Current Implementation

The current implementation includes:

1. **Order Processing**: Receives order data and processes it through the Gepard system
2. **Invoice Creation**: Automatically creates invoices in the wFirma accounting system
3. **Error Handling**: Comprehensive error handling with specific error codes
4. **Logging**: Detailed logging for debugging and monitoring
5. **Validation**: Input validation and business rule enforcement

### Integration Points

- **Gepard System**: Order management and processing
- **wFirma System**: Invoice creation and management
- **Sunbay Platform**: Core business logic and orchestration

## Error Handling

The endpoint provides detailed error information to help with troubleshooting:

- **Error Codes**: Standardized error codes for different failure scenarios
- **Error Messages**: Human-readable descriptions of what went wrong
- **HTTP Status Codes**: Appropriate HTTP status codes for different error types
- **Retry Logic**: Information about when to retry requests

## Security Considerations

- **API Key Authentication**: Secure API key-based authentication
- **Rate Limiting**: Protection against abuse and DoS attacks
- **Input Validation**: Comprehensive validation of all input data
- **Error Sanitization**: Error messages don't expose sensitive information

## Monitoring and Logging

The endpoint includes comprehensive logging for:
- Request processing
- Success/failure outcomes
- Error details
- Performance metrics
- Rate limiting events

---
