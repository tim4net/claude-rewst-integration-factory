# Claude Rewst Integration Factory

A Claude Code plugin for creating, validating, and transforming OpenAPI specifications compatible with [Rewst Custom Integration v2](https://docs.rewst.io/).

## Installation

```bash
# From GitHub
claude plugin install github:tim4net/claude-rewst-integration-factory

# Or from local path
claude plugin install /path/to/claude-rewst-integration-factory
```

## Skills

This plugin provides three skills:

### `/rewst-openapi:validate`

Validate an existing OpenAPI spec against Rewst requirements.

```bash
/rewst-openapi:validate path/to/spec.json
```

Checks for:
- File size limits (~500KB)
- OpenAPI version (must be 3.0.x)
- Bearer token authentication
- Response/requestBody structure
- Schema default type matching
- Tag definitions
- $ref resolution

### `/rewst-openapi:transform`

Convert an existing OpenAPI spec to Rewst-compatible format.

```bash
/rewst-openapi:transform source.json output-rewst.json
```

Handles:
- Swagger 2.0 → OpenAPI 3.0.3 conversion
- OpenAPI 3.1.x → 3.0.3 downgrade
- Authentication scheme conversion to Bearer
- Response/requestBody structure fixes
- Default value type corrections
- Size optimization (with user guidance)

### `/rewst-openapi:create`

Create a new Rewst-compatible spec from scratch.

```bash
/rewst-openapi:create "My API Name"
```

Guides you through:
- API information gathering
- Server URL configuration
- Endpoint definition
- Schema creation
- Validation and output

## Rewst CI v2 Requirements Summary

| Requirement | Value |
|-------------|-------|
| Max file size | ~500KB |
| OpenAPI version | 3.0.x only |
| Authentication | Bearer token only |
| Response structure | Must have `content.application/json.schema` |
| RequestBody structure | Must have `content.application/json.schema` |
| Default values | Must match schema type |
| Tags | Must be defined globally |

## Project Structure

```
claude-rewst-integration-factory/
├── .claude-plugin/
│   └── plugin.json           # Plugin manifest
├── skills/
│   ├── validate/SKILL.md     # Validation skill
│   ├── transform/SKILL.md    # Transformation skill
│   └── create/SKILL.md       # Creation skill
├── knowledge/
│   ├── rewst-requirements.md # Full requirements doc
│   ├── common-issues.md      # Error catalog
│   └── examples/
│       └── minimal-spec.json # Reference template
└── README.md
```

## Knowledge Base

The `knowledge/` directory contains reference documentation:

- **rewst-requirements.md** - Complete Rewst CI v2 requirements
- **common-issues.md** - Catalog of errors and fixes
- **examples/minimal-spec.json** - A minimal working spec template

## Common Issues

### "schema invalid: missing properties: 'content'"

Responses or requestBody missing the `content` property. Use `/rewst-openapi:validate` to find these.

### "default" property type must be integer

Default values are strings (`"0"`) instead of proper types (`0`). The transform skill fixes these automatically.

### File too large

Spec exceeds ~500KB. Use transform skill with size optimization, or manually select which operations to include.

## Contributing

Contributions welcome! Key areas:

- Additional validation rules
- More transformation scenarios
- Better size optimization strategies
- Documentation improvements

## License

MIT
