---
name: fix
description: Automatically fix common Rewst compatibility issues in an OpenAPI spec
argument-hint: <path-to-spec.json>
---

# Fix OpenAPI Spec for Rewst

You are automatically fixing common compatibility issues in an OpenAPI specification for Rewst Custom Integration v2.

## Your Task

Fix the spec at: `$ARGUMENTS`

If no path provided, ask which file to fix.

## Process

### Phase 1: Scan for Issues

Read the spec and identify all fixable issues:

```
## Issues Found

### Auto-fixable
1. [X] Response missing content property (N occurrences)
2. [X] RequestBody missing content property (N occurrences)
3. [X] Default value type mismatch (N occurrences)
4. [X] Missing global tags (N tags)
5. [X] Missing ErrorResponse schema

### Requires Manual Review
1. [ ] Unsupported auth type (currently: X)
2. [ ] File size over limit (X KB)
3. [ ] Unresolved $ref: X
```

### Phase 2: Apply Fixes

Fix each issue category:

#### Fix 1: Response Content Structure

Find responses with schema but no content wrapper:
```json
// Before
"200": {
  "description": "Success",
  "schema": { "$ref": "#/..." }
}

// After
"200": {
  "description": "Success",
  "content": {
    "application/json": {
      "schema": { "$ref": "#/..." }
    }
  }
}
```

#### Fix 2: RequestBody Content Structure

Find requestBody entries missing content:
```json
// Before
"requestBody": {
  "required": true,
  "schema": { "$ref": "#/..." }
}

// After
"requestBody": {
  "required": true,
  "content": {
    "application/json": {
      "schema": { "$ref": "#/..." }
    }
  }
}
```

#### Fix 3: Default Value Types

Fix type mismatches:
```json
// Integer with string default
{ "type": "integer", "default": "0" }
→ { "type": "integer", "default": 0 }

// Boolean with string default
{ "type": "boolean", "default": "true" }
→ { "type": "boolean", "default": true }

// Number with string default
{ "type": "number", "default": "0.0" }
→ { "type": "number", "default": 0.0 }
```

#### Fix 4: Global Tags

Collect all tags used in operations and ensure they're in global tags:
```json
"tags": [
  { "name": "Devices", "description": "Device operations" },
  { "name": "Alerts", "description": "Alert operations" }
]
```

#### Fix 5: ErrorResponse Schema

Add if missing:
```json
"ErrorResponse": {
  "type": "object",
  "properties": {
    "error": { "type": "string", "description": "Error type" },
    "message": { "type": "string", "description": "Error message" },
    "code": { "type": "integer", "description": "Error code" }
  }
}
```

#### Fix 6: Path Parameters

Ensure all path parameters have `required: true`:
```json
{
  "name": "id",
  "in": "path",
  "required": true,  // Add if missing
  "schema": { "type": "integer" }
}
```

### Phase 3: Write Fixed Spec

1. Write to `{original}-fixed.json`
2. Validate the result
3. Report changes

## Output Format

```
## Fix Results

### Applied Fixes
| Issue | Count | Status |
|-------|-------|--------|
| Response content structure | 45 | Fixed |
| RequestBody content structure | 12 | Fixed |
| Default type mismatches | 8 | Fixed |
| Missing global tags | 3 | Fixed |
| Missing ErrorResponse | 1 | Added |
| Path param required | 5 | Fixed |

### Validation
- JSON: Valid
- OpenAPI: Valid
- Size: X KB

### Output
Written to: path/to/spec-fixed.json

### Manual Steps Needed
- [ ] Convert auth from X to Bearer token
- [ ] Resolve missing schema: Y
```

## What This Skill Does NOT Fix

These require human decisions:
- Authentication type conversion (use `/rewst-openapi:transform`)
- Size reduction (use `/rewst-openapi:subset`)
- Missing schemas (need to define or remove references)
- Invalid JSON syntax (fix manually first)
- OpenAPI version upgrade/downgrade (use `/rewst-openapi:transform`)
