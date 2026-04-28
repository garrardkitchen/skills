# README and Changelog Guidance

Use this file when the user asks for repository documentation content.

## Good README sections

- project purpose
- architecture summary
- prerequisites
- local run steps
- configuration and secrets approach
- authentication/authorization model
- tenant isolation model where applicable
- data classification and privacy notes where applicable
- testing instructions
- observability notes
- deployment notes
- security considerations
- known limitations, threat model assumptions, and exception process

## README example outline

```markdown
# MyApp

## What it does

## Architecture

## Running locally

## Configuration

## Security model

## Testing

## Observability
```

## Changelog guidance

- use Keep a Changelog structure
- write user-facing entries
- group by Added / Changed / Fixed / Security
- mention breaking changes explicitly
- call out security fixes under `Security`
- mention migration, authorization, or data-impacting changes clearly
- do not include secrets, internal incident details, or exploit instructions

## Security documentation checklist

```markdown
## Security model

- Identity provider:
- Token validation:
- Authorization model:
- Resource ownership:
- Tenant isolation:
- Secret storage:
- Data protection:
- Logging and audit:
- Rate limiting and abuse controls:
- Security tests:
- Known exceptions:
```

## Sources

- Keep a Changelog
- OWASP ASVS v5.0.0 documentation and verification guidance
- Microsoft Learn ASP.NET Core security documentation
