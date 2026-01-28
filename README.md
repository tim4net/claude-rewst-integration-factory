# Claude Rewst Integration Factory

A Claude Code plugin that helps you create custom integrations for [Rewst](https://rewst.io/) - even if you've never worked with APIs before.

## What This Does

Rewst lets you connect to any API using "Custom Integrations." To create one, you need an OpenAPI spec (a file that describes the API). This plugin helps you:

- **Create** specs from scratch or from documentation
- **Fix** specs that have errors
- **Validate** specs before uploading to Rewst

## Quick Start

```bash
# Install the plugin
claude plugin install github:tim4net/claude-rewst-integration-factory
```

## Which Skill Do I Need?

```
START HERE
    │
    ▼
Do you have an OpenAPI/Swagger spec file?
    │
    ├─ YES → Does it work when you upload to Rewst?
    │           │
    │           ├─ YES → You're done! 🎉
    │           │
    │           ├─ NO, I get errors → /rewst-openapi:fix
    │           │
    │           └─ NO, file too large → /rewst-openapi:subset
    │
    └─ NO → Do you have API documentation (a URL)?
              │
              ├─ YES → /rewst-openapi:from-url
              │
              └─ NO → /rewst-openapi:create
```

**Other situations:**
- Want to check a spec before uploading? → `/rewst-openapi:validate`
- Need to convert Swagger 2.0 or OpenAPI 3.1? → `/rewst-openapi:transform`
- Have multiple specs to combine? → `/rewst-openapi:merge`
- Want documentation of what's in a spec? → `/rewst-openapi:document`

## Skills Reference

| Skill | What It Does |
|-------|--------------|
| `validate` | Check if your spec will work with Rewst |
| `fix` | Automatically repair common problems |
| `transform` | Convert older formats (Swagger, OpenAPI 3.1) to Rewst format |
| `create` | Build a new spec with guided questions |
| `from-url` | Generate a spec from API documentation |
| `subset` | Shrink a large spec to fit Rewst's size limit |
| `merge` | Combine multiple specs into one |
| `document` | Create human-readable documentation |

### /rewst-openapi:validate

Check if a spec is ready for Rewst.

```bash
/rewst-openapi:validate myspec.json
```

Uses **Rewst's exact validation rules** - if it passes here, it will work in Rewst.

### /rewst-openapi:fix

Automatically fix common issues.

```bash
/rewst-openapi:fix myspec.json
```

Fixes things like missing content wrappers, wrong default value types, and undefined tags.

### /rewst-openapi:transform

Convert specs from other formats.

```bash
/rewst-openapi:transform swagger-spec.json output.json
```

Handles Swagger 2.0, OpenAPI 3.1, and specs with unsupported authentication.

### /rewst-openapi:create

Build a spec from scratch with guided questions.

```bash
/rewst-openapi:create "My API Name"
```

### /rewst-openapi:from-url

Generate a spec from API documentation.

```bash
/rewst-openapi:from-url https://docs.example.com/api
```

### /rewst-openapi:subset

Shrink a large spec to fit Rewst's ~500KB limit.

```bash
/rewst-openapi:subset large-spec.json 450
```

Helps you choose which operations to keep based on your workflow needs.

### /rewst-openapi:merge

Combine multiple specs into one.

```bash
/rewst-openapi:merge api-v1.json api-v2.json -o combined.json
```

### /rewst-openapi:document

Generate readable documentation.

```bash
/rewst-openapi:document myspec.json
```

## Command-Line Linter

For CI/CD or quick checks, use the standalone linter:

```bash
# Install Spectral (required)
npm install -g @stoplight/spectral-cli

# Run the linter
node tools/rewst-lint.js myspec.json
```

This uses **Rewst's exact validation rules** - the same Spectral ruleset Rewst uses internally.

## Rewst Requirements (Technical Details)

| Requirement | Details |
|-------------|---------|
| File size | ~500KB maximum |
| Format | OpenAPI 3.0.x (not 3.1, not Swagger 2.0) |
| Authentication | Bearer token only |
| Title | Must have `info.title` |

The most common issues are structural - responses and requestBody must use the OpenAPI 3.0 `content` wrapper format. The `fix` skill handles this automatically.

## Project Structure

```
claude-rewst-integration-factory/
├── .claude-plugin/
│   └── plugin.json           # Plugin manifest
├── skills/                   # Claude Code skills
│   ├── validate/SKILL.md
│   ├── fix/SKILL.md
│   ├── transform/SKILL.md
│   ├── create/SKILL.md
│   ├── from-url/SKILL.md
│   ├── subset/SKILL.md
│   ├── merge/SKILL.md
│   └── document/SKILL.md
├── tools/                    # Standalone tools
│   ├── rewst-lint.js         # CLI linter
│   └── .spectral-rewst.yaml  # Rewst's exact ruleset
├── knowledge/                # Reference docs
│   ├── rewst-requirements.md
│   ├── common-issues.md
│   └── examples/
│       └── minimal-spec.json
└── README.md
```

## Glossary

New to APIs? Here are the key terms:

- **OpenAPI spec**: A JSON/YAML file that describes an API - what endpoints exist, what data they accept, what they return
- **Endpoint**: A URL path you can call, like `/users` or `/devices/{id}`
- **Schema**: A description of data structure - what fields exist and their types
- **Bearer token**: A password-like string you include with requests to prove who you are

## Contributing

Contributions welcome! The linter uses Rewst's exact Spectral ruleset from their codebase.

## License

MIT
