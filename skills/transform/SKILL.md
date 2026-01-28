---
name: transform
description: Transform an existing OpenAPI spec to be Rewst Custom Integration v2 compatible
argument-hint: <path-to-spec.json> [output-path.json]
---

# Transform OpenAPI Spec for Rewst

You are transforming an existing OpenAPI specification to be compatible with Rewst Custom Integration v2.

## Your Task

Transform the spec at: `$ARGUMENTS`

Parse the arguments:
- First argument: source spec path (required)
- Second argument: output path (optional, defaults to `{source}-rewst.json`)

## Transformation Process

### Phase 1: Analysis

First, analyze the source spec:
1. Read and validate the JSON
2. Check current OpenAPI version
3. Identify authentication method
4. Count operations and estimate output size
5. Report findings before proceeding

### Phase 2: Version Conversion

If needed:
- **Swagger 2.0** → Convert to OpenAPI 3.0.3
  - Move `definitions` to `components/schemas`
  - Convert `securityDefinitions` to `components/securitySchemes`
  - Update response structure
  - Update parameter structure

- **OpenAPI 3.1.x** → Downgrade to OpenAPI 3.0.3
  - Remove webhooks
  - Adjust JSON Schema differences
  - Update version field

### Phase 3: Authentication Conversion

Convert to Bearer token auth:

1. Add/update security scheme:
```json
"components": {
  "securitySchemes": {
    "BearerAuth": {
      "type": "http",
      "scheme": "bearer",
      "description": "Bearer token authentication"
    }
  }
}
```

2. Set global security:
```json
"security": [{ "BearerAuth": [] }]
```

3. Remove operation-level security that uses other schemes

### Phase 4: Structure Fixes

Fix all structural issues:

1. **Responses** - Ensure content property:
```json
"responses": {
  "200": {
    "description": "Success",
    "content": {
      "application/json": {
        "schema": { ... }
      }
    }
  }
}
```

2. **RequestBody** - Ensure content property:
```json
"requestBody": {
  "required": true,
  "content": {
    "application/json": {
      "schema": { ... }
    }
  }
}
```

3. **Default values** - Fix type mismatches:
   - `"default": "0"` → `"default": 0` for integers
   - `"default": "true"` → `"default": true` for booleans

### Phase 5: Tags

1. Collect all tags used in operations
2. Create global tags array with all used tags
3. Add descriptions to tags where possible

### Phase 6: Size Optimization

If spec exceeds 450KB:

1. **Ask user** which operations are most important
2. **Suggest** operations to remove (least common use cases)
3. **Remove** orphaned schemas after path removal
4. **Trim** verbose descriptions
5. **Remove** examples if needed

### Phase 7: Validation

Before writing output:
1. Validate JSON syntax
2. Check all $ref references resolve
3. Verify size is under 500KB
4. Run through validation checklist

## Output

1. Write transformed spec to output path
2. Provide summary of changes made
3. List any manual steps needed

## Reference

See the knowledge files:
- @knowledge/rewst-requirements.md - Full requirements
- @knowledge/common-issues.md - Common fixes
- @knowledge/examples/minimal-spec.json - Reference template
