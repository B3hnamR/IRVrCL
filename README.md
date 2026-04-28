## Changelog v1.2

- Added upstream timeout control with `UPSTREAM_TIMEOUT_MS` (default: `20000`)
- Added optional relay authentication with `RELAY_KEY` (`x-relay-key` header or `k` query param)
- Added method allowlist: `GET`, `HEAD`, `POST`
- Added request header filtering (allowlist) to reduce unnecessary forwarded headers
- Added stricter strip list for hop-by-hop and proxy-related headers
- Added timeout/error logging for easier diagnostics
- `RELAY_PATH` is now optional:
  - If set, only that path (and subpaths) is allowed
  - If not set, all paths are allowed
- Restored global rewrite route in `vercel.json` (`/(.*) -> /api/index`)
