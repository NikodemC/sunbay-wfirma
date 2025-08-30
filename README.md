# Gepard Order Processing API Endpoint

## Overview

The `[HttpPost("order/process")]` endpoint is designed to process multiple invoices from the Gepard system and automatically create them in the wFirma accounting system. This endpoint provides a seamless integration between order management and invoice generation with support for batch processing.

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

### Request Structure

The endpoint expects a `ProcessOrderCommand` with the following structure:

```json
{
  "invoices": [
    {
      "invoice_id": "string",        // UUID v4 - Unique identifier for the invoice
      "client_id": "string",         // wFirma contractor/client ID to associate with the invoice
      "items": [
        {
          "name": "string",          // Product/service name
          "unit_price": "decimal",   // Unit price in decimal format (e.g., "50.00")
          "currency": "string",      // Currency code (e.g., "PLN")
          "quantity": "integer",     // Quantity of the item
          "quantity_unit": "string", // Unit of measurement (e.g., "szt.", "kg", "m")
          "vat_rate": "string",      // VAT rate identifier (e.g., "23%")
          "tax_rate": "integer|null" // Tax rate  (e.g., 23)
        }
      ]
    }
  ]
}
```

### Field Descriptions

- **invoices**: Array of invoices to be processed in a single request
  - **invoice_id**: Unique identifier for the invoice from Gepard system (UUID v4 format)
  - **client_id**: wFirma contractor/client ID to associate with the invoice
  - **items**: Array of order items to be included in the invoice
    - **name**: Product/service name
    - **unit_price**: Unit price in decimal format
    - **currency**: Currency code for the invoice
    - **quantity**: Quantity of the item (must be positive integer)
    - **quantity_unit**: Unit of measurement (e.g., "szt.", "kg", "m")
    - **vat_rate**: VAT rate identifier
    - **tax_rate**: Tax rate
### Example Request

```json
{
  "invoices": [
    {
      "invoice_id": "550e8400-e29b-41d4-a716-446655440001",
      "client_id": "152158328",
      "items": [
        {
          "name": "Premium Coffee Beans",
          "unit_price": "50.00",
          "currency": "PLN",
          "quantity": 3,
          "quantity_unit": "kg",
          "vat_rate": "23%",
          "tax_rate": null
        },
        {
          "name": "Coffee Filter Papers",
          "unit_price": "0.25",
          "currency": "PLN",
          "quantity": 100,
          "quantity_unit": "szt.",
          "vat_rate": "23%",
          "tax_rate": 23
        }
      ]
    },
    {
      "invoice_id": "660e8400-e29b-41d4-a716-446655440002",
      "client_id": "15485872",
      "items": [
        {
          "name": "Office Supplies",
          "unit_price": "15.00",
          "currency": "PLN",
          "quantity": 5,
          "quantity_unit": "szt.",
          "vat_rate": "23%",
          "tax_rate": 23
        }
      ]
    }
  ]
}
```

## Response Format

### Success Response (200 OK)

```json
{
  "success": true,
  "totalInvoices": 2,
  "successfulInvoices": 1,
  "existingInvoices": 1,
  "failedInvoices": 0,
  "invoiceResults": [
    {
      "invoiceId": "550e8400-e29b-41d4-a716-446655440001",
      "clientId": "152158328",
      "status": "Created",
      "createdAt": "2025-08-30T13:00:44.764575+00:00",
      "invoiceNumber": "FV 22/2025",
      "wfirmaInvoiceId": "381321041"
    },
    {
      "invoiceId": "660e8400-e29b-41d4-a716-446655440002",
      "clientId": "15485872",
      "status": "Existed",
      "createdAt": "2025-08-30T13:35:18.9209402+00:00",
      "invoiceNumber": "FV 23/2025",
      "wfirmaInvoiceId": "381323351"
    }
  ]
}
```

### Partial Success Response (422 Unprocessable Entity)

When some invoices succeed and others fail:

```json
{
  "success": false,
  "errorCode": "BATCH_PROCESSING_ERROR",
  "errorMessage": "Some invoices failed during processing",
  "totalInvoices": 2,
  "successfulInvoices": 0,
  "existingInvoices": 1,
  "failedInvoices": 1,
  "invoiceResults": [
    {
      "invoiceId": "550e8400-e29b-41d4-a716-446655440003",
      "clientId": "152158328",
      "status": "Existed",
      "createdAt": "2025-08-30T13:35:18.9209402+00:00",
      "invoiceNumber": "FV 23/2025",
      "wfirmaInvoiceId": "381323351"
    },
    {
      "invoiceId": "660e8400-e29b-41d4-a716-446655440008",
      "clientId": "15485872",
      "status": "Failed",
      "errorCode": "VALIDATION_ERROR",
      "errorMessage": "contractor.name: Pole nie może być puste.; contractor.zip: Pole nie może być puste.; contractor.city: Pole nie może być puste."
    }
  ]
}
```

### Error Responses

#### Validation Error (422 Unprocessable Entity)
```json
{
  "success": false,
  "errorCode": "VALIDATION_ERROR",
  "errorMessage": "At least one invoice is required"
}
```

#### Contractor Not Found (404 Not Found)
```json
{
  "success": false,
  "errorCode": "CONTRACTOR_NOT_FOUND",
  "errorMessage": "Contractor with ID '12345' was not found"
}
```

#### Insufficient Permissions (403 Forbidden)
```json
{
  "success": false,
  "errorCode": "INSUFFICIENT_PERMISSIONS",
  "errorMessage": "Insufficient permissions to create invoice"
}
```

#### Rate Limit Exceeded (429 Too Many Requests)
```json
{
  "success": false,
  "errorCode": "RATE_LIMIT_EXCEEDED",
  "errorMessage": "Rate limit exceeded. Please try again later."
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

## Response Status Values

The `status` field in invoice results returns string values:

- **"Created"**: Invoice was newly created in wFirma
- **"Existed"**: Invoice already existed in the database
- **"Failed"**: Invoice failed to be created in wFirma

## How It Works

1. **Request Validation**: The system validates the incoming request data including invoice structure and required fields
2. **Batch Processing**: Processes multiple invoices in parallel for efficiency
3. **Invoice Creation**: Creates wFirma invoices using the invoice data
4. **Duplicate Detection**: Checks for existing invoices to avoid duplicates
5. **Response Generation**: Returns comprehensive results for all processed invoices

## Implementation Details

- **Batch Processing**: Supports processing multiple invoices in a single request
- **Status Tracking**: Provides detailed status for each invoice (Created, Existed, Failed)
- **Error Handling**: Comprehensive error reporting with specific error codes and messages
- **Duplicate Prevention**: Automatically detects and reports existing invoices
- **Parallel Processing**: Handles multiple invoices efficiently
- **Consistent Response Structure**: Both success and failure responses include all invoice counts
