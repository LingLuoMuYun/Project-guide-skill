# Security and Permission Review

Use for authentication, authorization, secret handling, external action systems, dependency risk, or audit-oriented project analysis.

## First Questions

- What users, tenants, roles, resources, data, credentials, or external systems need protection?
- Where are trust boundaries crossed?
- How are users/services authenticated?
- How is authorization enforced?
- Where are secrets loaded, stored, logged, redacted, and passed to subprocesses?
- What input is untrusted?
- What dependencies and release paths affect the trusted build?

## Locate

- auth middleware, route guards, sessions, token handling
- role/permission/menu/button checks
- user, tenant, organization, and resource ownership models
- upload handlers, parsers, serializers, validators
- config loading and environment templates
- logs, telemetry, and error reporting
- external API clients and callbacks
- dependency manifests, lockfiles, CI, release scripts, container images

## Risk Areas

- auth bypass from missing guard coverage
- object-level authorization gaps
- secrets committed, echoed, or logged
- unsafe file paths, archive extraction, shell execution, template rendering, deserialization
- open redirects or weak redirect validation
- overbroad CORS, CSRF gaps, insecure cookies, risky token storage
- dependency confusion, mutable action/image tags, unpinned releases
- destructive actions without preview, approval, or audit logs

## Safety Rules

- Do not print secrets, private keys, tokens, full connection strings, sensitive records, or exploit payloads.
- Prefer defensive inspection and validation over exploit execution.
- Do not run scanners, fuzzers, network probes, or credential tests unless explicitly requested and scoped.
- Label security findings: confirmed, likely, possible, or open.
