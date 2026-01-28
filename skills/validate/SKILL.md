---
name: validate
description: Validate an OpenAPI spec for Rewst Custom Integration v2 compatibility
argument-hint: <path-to-spec.json>
---

# Validate OpenAPI Spec for Rewst

You are validating an OpenAPI specification for compatibility with Rewst Custom Integration v2.

## Your Task

Validate the spec at: `$ARGUMENTS`

If no path provided, ask the user which file to validate.

## Validation Steps

Perform these checks in order:

### 1. File Basics
- [ ] File exists and is valid JSON
- [ ] File size under 500KB
- [ ] OpenAPI version is 3.0.x (not 3.1.x, not Swagger 2.0)

### 2. Authentication
- [ ] `components.securitySchemes` contains a Bearer token scheme
- [ ] Global `security` array references the Bearer scheme
- [ ] No unsupported auth types (HMAC, OAuth flows, custom API keys)

### 3. Server Configuration
- [ ] `servers` array is defined
- [ ] Server URL uses variables for multi-tenant support (if applicable)

### 4. Response Structure
For every operation response:
- [ ] Has `content` property (not just `schema` directly)
- [ ] Content type is `application/json`
- [ ] Schema is defined (inline or $ref)

### 5. RequestBody Structure
For every operation with requestBody:
- [ ] Has `content` property
- [ ] Content type is `application/json`
- [ ] Schema is defined

### 6. Schema Defaults
For every schema with a `default` value:
- [ ] Integer schemas have integer defaults (not `"0"`)
- [ ] Number schemas have number defaults (not `"0.0"`)
- [ ] Boolean schemas have boolean defaults (not `"true"` or `"false"`)

### 7. Tags
- [ ] Global `tags` array is defined
- [ ] Every tag used in operations exists in global tags

### 8. References
- [ ] All `$ref` values resolve to existing components
- [ ] No typos in reference paths
- [ ] No references to `#/definitions/` (Swagger 2.0 style)

### 9. Operations
- [ ] Every operation has an `operationId`
- [ ] `operationId` values are unique
- [ ] Path parameters are marked `required: true`

## Output Format

Provide a structured report:

```
## Validation Results: [filename]

### Summary
- Status: PASS / FAIL / WARNINGS
- File size: X KB (limit: 500KB)
- Operations: X
- Schemas: X

### Issues Found

#### Critical (must fix)
1. [Issue description with location]

#### Warnings (should fix)
1. [Warning description with location]

### Recommendations
- [Any suggestions for improvement]
```

## Reference

See the knowledge files for detailed requirements:
- @knowledge/rewst-requirements.md - Full requirements documentation
- @knowledge/common-issues.md - Common errors and fixes
