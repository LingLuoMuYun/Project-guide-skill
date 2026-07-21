# Frontend Project Analysis

Use for React, Vue, Angular, Next.js, Nuxt, Vite, Electron renderer, or frontend-heavy admin systems.

## Questions

- What framework and router are used?
- Where is the app root?
- Which components are client-only?
- Where are layouts, error pages, loading pages, and route guards?
- Where are API clients defined?
- Where is global state stored?
- How is authentication handled?
- How are permissions represented: route, menu, button, API?
- How are environment variables loaded?
- How are errors displayed?
- How are uploads/downloads handled?
- How are styles organized?
- What build/dev scripts exist?

## Flow to Trace

```text
User interaction
-> route/page
-> component/hook
-> service/API client
-> HTTP request
-> response parsing
-> state update
-> UI render
```

## Environment and Runtime

Identify:

- dev server port
- backend/API base URL
- proxy/rewrite config
- CORS assumptions
- build output directory
- asset/CDN prefix
- image remote domains
- WebSocket endpoints
- required local backend services

## Common Failure Modes

- Frontend points at wrong backend URL
- Environment variable only available at build time
- API prefix is applied twice or not applied
- Auth token missing or stale
- Permission cache stale
- API response shape changed
- SSR/client boundary breaks due to `window` or `localStorage`
- Build skips lint/typecheck
- Generated build files or copied assets drift

## Recommended Verification

- typecheck
- lint
- unit tests
- build
- manual login smoke test
- core page network request check
- browser console check
- upload/download smoke test when relevant
