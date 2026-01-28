---
name: create
description: Create a new Rewst-compatible OpenAPI spec from scratch or from API documentation
argument-hint: [api-name]
---

# Create Rewst-Compatible OpenAPI Spec

You are helping create a new OpenAPI specification that's compatible with Rewst Custom Integration v2.

## Your Task

Create a spec for: `$ARGUMENTS`

If no API name provided, ask the user what API they want to create a spec for.

## Information Gathering

Before creating the spec, gather this information:

### 1. API Basics
- **API Name**: What's the service/API called?
- **Base URL**: What's the API endpoint pattern?
  - Is it multi-tenant (e.g., `{company}.api.example.com`)?
  - What's the base path (e.g., `/api/v1`)?
- **Authentication**: How do users get Bearer tokens?

### 2. Source Material
Ask if the user has any of:
- Existing OpenAPI/Swagger spec to convert
- API documentation URL
- Postman collection
- List of endpoints they need

### 3. Scope
- How many operations do they need? (Remember ~500KB limit)
- What are the primary use cases?
- Which endpoints are must-have vs nice-to-have?

## Spec Creation

### Start with Template

Use this base structure:

```json
{
  "openapi": "3.0.3",
  "info": {
    "title": "[API Name] - Rewst Integration",
    "description": "[Description]",
    "version": "1.0.0"
  },
  "servers": [
    {
      "url": "https://{company}.example.com/api/v1",
      "variables": {
        "company": {
          "default": "your-company",
          "description": "Your company subdomain"
        }
      }
    }
  ],
  "tags": [],
  "security": [{ "BearerAuth": [] }],
  "paths": {},
  "components": {
    "securitySchemes": {
      "BearerAuth": {
        "type": "http",
        "scheme": "bearer",
        "description": "Bearer token authentication"
      }
    },
    "schemas": {
      "ErrorResponse": {
        "type": "object",
        "properties": {
          "error": { "type": "string" },
          "message": { "type": "string" }
        }
      }
    }
  }
}
```

### Build Incrementally

For each endpoint category:

1. **Define the tag** first
2. **Create schemas** for request/response bodies
3. **Add paths** with full operation details
4. **Validate** before moving to next category

### Operation Template

Each operation should follow this pattern:

```json
{
  "operationId": "verbNoun",
  "summary": "Short description",
  "description": "Detailed description of what this does",
  "tags": ["Category"],
  "parameters": [],
  "responses": {
    "200": {
      "description": "Success description",
      "content": {
        "application/json": {
          "schema": { "$ref": "#/components/schemas/ResponseModel" }
        }
      }
    },
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
    }
  }
}
```

### Common Patterns

**List endpoint:**
- GET `/resources`
- Query params: pagination (see below), `filter`, `sort`
- Response: Array wrapper with pagination metadata

**Get single:**
- GET `/resources/{id}`
- Path param: `id` (required: true)
- Response: Single object

**Create:**
- POST `/resources`
- RequestBody with required fields
- Response: Created object with ID

**Update:**
- PUT `/resources/{id}` (full replace)
- PATCH `/resources/{id}` (partial update)
- RequestBody with updatable fields

**Delete:**
- DELETE `/resources/{id}`
- Response: 204 No Content

### Pagination Patterns

Ask user which pagination style the API uses:

**Offset/Limit** (most common):
```json
"parameters": [
  { "name": "offset", "in": "query", "schema": { "type": "integer", "default": 0 } },
  { "name": "limit", "in": "query", "schema": { "type": "integer", "default": 50 } }
]
```

**Cursor-based** (modern APIs):
```json
"parameters": [
  { "name": "cursor", "in": "query", "schema": { "type": "string" } },
  { "name": "limit", "in": "query", "schema": { "type": "integer", "default": 50 } }
]
```

**Page-based**:
```json
"parameters": [
  { "name": "page", "in": "query", "schema": { "type": "integer", "default": 1 } },
  { "name": "per_page", "in": "query", "schema": { "type": "integer", "default": 25 } }
]
```

Include pagination metadata in response schemas (total, has_more, next_cursor, etc.).

### Authentication Patterns

Ask how users obtain Bearer tokens:

1. **API Key as Bearer**: Key used directly as token - document where to get key
2. **OAuth App**: Client credentials flow - document app creation process
3. **Session Token**: API creates session first - include session creation endpoint
4. **Token Refresh**: Tokens expire - include refresh endpoint if needed

Add authentication instructions in the `info.description` field:

```json
"info": {
  "description": "## Authentication\n\nObtain your API key from Settings > API. Use it as the Bearer token."
}
```

### Rate Limiting

If the API has rate limits, document them:
- In `info.description`: mention limits and behavior
- Add 429 response with Retry-After header
- Note any per-endpoint variations

## Quality Checks

Before finalizing, verify:

- [ ] All operations have unique `operationId`
- [ ] All path parameters have `required: true`
- [ ] All responses have `content` with `application/json`
- [ ] All requestBody entries have `content`
- [ ] All default values match their schema types
- [ ] All tags are defined globally
- [ ] File size is under 500KB

## Output

### Folder Structure

Create a dedicated folder for the integration:

```
{api-name}/
├── {api-name}-rewst.json    # The OpenAPI spec
├── {api-name}-icon.svg      # Logo/icon if available
└── README.md                # Setup instructions
```

Use lowercase with hyphens for the folder name (e.g., `buy-me-a-coffee/`, `acme-api/`).

### Deliverables

1. Create the integration folder
2. Write the complete spec file to the folder
3. Write a README.md with:
   - Brief description of the API
   - How to obtain Bearer token/credentials
   - List of available operations
   - Any known limitations or notes

## Reference

See the knowledge files:
- @knowledge/rewst-requirements.md - Requirements
- @knowledge/examples/minimal-spec.json - Example spec
