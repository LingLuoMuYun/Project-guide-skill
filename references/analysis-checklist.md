# Analysis Checklist

Use this checklist during repository analysis. Do not dump it verbatim unless the user asks for a checklist.

## Repository Surface

- Project name and stated purpose
- Languages and package managers
- Top-level directories
- Docs, examples, tests, scripts, CI, deployment files
- Environment templates or `.env.*`
- Generated/vendor/cache/build directories to skip
- Single project, nested project, monorepo, or mixed-language system

## Entry Points

- Local startup command
- Main app/server/CLI entry file
- Frontend app root and routing
- Backend routes/controllers
- API clients and service layer
- Global state root
- Test/build/lint commands
- Background workers, queues, scheduled tasks

## Architecture

- Main layers and runtime boundaries
- Input handling and dispatch path
- Core domain logic
- External system adapters
- Error representation and propagation
- Public/stable contracts vs internal details

## Data and Control Flow

- Most important user/system flow
- Data transformed at each step
- IDs, paths, handles, or state references passed forward
- External side effects
- Result path back to caller/UI

## State and Contracts

- Required configuration
- Environment variables read
- Request/response schemas or types
- Runtime state in memory
- Persistent state, cache, files, database, localStorage/sessionStorage
- Data that can become stale

## Verification

- Existing tests and test framework
- Whether tests are runnable here
- Manual smoke tests
- Commands actually run
- What passing checks prove
- What remains unverified

## Risk Scan

- Configuration ambiguity
- Hardcoded ports or paths
- Missing error handling
- Hidden external state
- Weak validation
- Stale documentation
- Missing tests
- Large untyped payloads
- Race conditions or repeated side effects
- Platform-specific assumptions
- Build/deploy scripts with side effects
- Production-like defaults in local commands

## Newcomer Orientation

- First file to read
- First command to run
- First UI/API flow to test
- First safe small change
- Files to avoid changing initially
- Concepts to learn before modifying core logic
