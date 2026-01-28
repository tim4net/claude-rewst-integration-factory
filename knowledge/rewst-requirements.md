# Rewst Custom Integration v2 - OpenAPI Requirements

This document captures all requirements for OpenAPI specs to work with Rewst Custom Integration v2.

## Size Limits

- **Maximum file size**: ~500KB
- **Practical limit**: 450-480KB to leave headroom
- If your source spec is larger, you must subset the operations

## OpenAPI Version

- **Required**: OpenAPI 3.0.x (3.0.0, 3.0.1, 3.0.2, or 3.0.3)
- OpenAPI 3.1.x is NOT supported

## Authentication

Rewst CI v2 supports **Bearer token authentication only**.

### Required Security Scheme

```json
"securityDefinitions": {
  "BearerAuth": {
    "type": "http",
    "scheme": "bearer",
    "description": "Bearer token authentication"
  }
}
```

### Global Security

```json
"security": [
  { "BearerAuth": [] }
]
```

### What's NOT Supported

- HMAC/signature-based auth (like AWS Signature v4 or LMv1)
- OAuth 2.0 flows (authorization_code, implicit, etc.)
- API key in custom headers (X-API-Key style)
- Basic authentication

If the source API uses these, you must convert to Bearer token or the spec won't work.

## Server URL Configuration

Use server variables for multi-tenant APIs:

```json
"servers": [
  {
    "url": "https://{company}.example.com/api",
    "variables": {
      "company": {
        "default": "your-company",
        "description": "Your company subdomain"
      }
    }
  }
]
```

## Response Structure

**CRITICAL**: Every response MUST have a `content` property with proper structure.

### Correct Structure

```json
"responses": {
  "200": {
    "description": "Success",
    "content": {
      "application/json": {
        "schema": {
          "$ref": "#/components/schemas/ResponseModel"
        }
      }
    }
  }
}
```

### Common Mistake - Missing content

```json
"responses": {
  "200": {
    "description": "Success",
    "schema": { ... }  // WRONG - this is OpenAPI 2.0 style
  }
}
```

### Error Responses

Include standard error responses:

```json
"responses": {
  "400": {
    "description": "Bad Request",
    "content": {
      "application/json": {
        "schema": { "$ref": "#/components/schemas/ErrorResponse" }
      }
    }
  },
  "401": {
    "description": "Unauthorized",
    "content": {
      "application/json": {
        "schema": { "$ref": "#/components/schemas/ErrorResponse" }
      }
    }
  },
  "404": {
    "description": "Not Found",
    "content": {
      "application/json": {
        "schema": { "$ref": "#/components/schemas/ErrorResponse" }
      }
    }
  },
  "500": {
    "description": "Internal Server Error",
    "content": {
      "application/json": {
        "schema": { "$ref": "#/components/schemas/ErrorResponse" }
      }
    }
  }
}
```

## RequestBody Structure

**CRITICAL**: Every requestBody MUST have a `content` property.

### Correct Structure

```json
"requestBody": {
  "required": true,
  "content": {
    "application/json": {
      "schema": {
        "$ref": "#/components/schemas/CreateRequest"
      }
    }
  }
}
```

### Common Mistake - Missing content

```json
"requestBody": {
  "required": true,
  "schema": { ... }  // WRONG - must be inside content
}
```

## Schema Default Values

**CRITICAL**: Default values MUST match the schema type exactly.

### Type Matching Rules

| Schema Type | Default Value | Correct | Wrong |
|-------------|---------------|---------|-------|
| `integer` | `0` | `"default": 0` | `"default": "0"` |
| `number` | `0.0` | `"default": 0.0` | `"default": "0.0"` |
| `boolean` | `false` | `"default": false` | `"default": "false"` |
| `string` | `"value"` | `"default": "value"` | `"default": 0` |

### Example Fix

```json
// WRONG
{
  "type": "integer",
  "default": "0"
}

// CORRECT
{
  "type": "integer",
  "default": 0
}
```

## Tags

**CRITICAL**: All tags used in operations MUST be defined in the global `tags` array.

### Define Tags Globally

```json
{
  "tags": [
    { "name": "Devices", "description": "Device management operations" },
    { "name": "Alerts", "description": "Alert management operations" }
  ]
}
```

### Use in Operations

```json
{
  "paths": {
    "/devices": {
      "get": {
        "tags": ["Devices"],  // Must match a global tag
        "operationId": "getDevices"
      }
    }
  }
}
```

## $ref References

All `$ref` references must resolve to existing components.

### Valid Reference

```json
{ "$ref": "#/components/schemas/Device" }
```

The schema `Device` MUST exist in `components.schemas.Device`.

### Common Issues

1. **Typos**: `#/components/schemas/Devcie` (misspelled)
2. **Missing schemas**: Referenced but never defined
3. **Wrong path**: `#/definitions/Device` (OpenAPI 2.0 style)

## Operation Requirements

Each operation should have:

- `operationId`: Unique identifier (required by Rewst)
- `summary`: Short description (recommended)
- `description`: Detailed explanation (recommended)
- `tags`: At least one tag (recommended)

### Example Operation

```json
{
  "get": {
    "operationId": "getDeviceById",
    "summary": "Get device by ID",
    "description": "Retrieves a single device by its unique identifier",
    "tags": ["Devices"],
    "parameters": [...],
    "responses": {...}
  }
}
```

## Parameter Requirements

### Path Parameters

Must be marked as `required: true`:

```json
{
  "name": "id",
  "in": "path",
  "required": true,
  "schema": {
    "type": "integer"
  },
  "description": "Device ID"
}
```

### Query Parameters

Include descriptions and defaults where appropriate:

```json
{
  "name": "limit",
  "in": "query",
  "required": false,
  "schema": {
    "type": "integer",
    "default": 50,
    "minimum": 1,
    "maximum": 1000
  },
  "description": "Maximum number of results to return"
}
```

## Validation Checklist

Before uploading to Rewst, verify:

- [ ] File size under 500KB
- [ ] OpenAPI version is 3.0.x
- [ ] Bearer token security scheme defined
- [ ] All responses have `content` property
- [ ] All requestBody entries have `content` property
- [ ] All default values match their schema types
- [ ] All operation tags are defined globally
- [ ] All $ref references resolve
- [ ] Every operation has an operationId

## Recommended Validators

1. **swagger-parser** (Node.js): Basic OpenAPI validation
   ```bash
   npx @apidevtools/swagger-parser validate spec.json
   ```

2. **Spectral** (Node.js): Comprehensive linting
   ```bash
   npx @stoplight/spectral-cli lint spec.json
   ```

3. **Online**: https://editor.swagger.io/ (paste spec to validate)
