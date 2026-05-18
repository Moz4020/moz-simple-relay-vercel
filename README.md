# Moz Simple Vercel Relay

A lightweight HTTP relay for Vercel deployments.

This project can run in two modes:

- `Fast Pipe`: uses Vercel rewrites for a minimal pass-through deployment.
- `Node Relay`: uses a serverless function when you need runtime controls such as timeouts, throttling, request limits, and basic diagnostics.

The repository also includes a Windows deployment helper that guides setup, project linking, environment configuration, and deployment.

## Features

- Vercel-ready relay endpoint
- Rewrite-based deployment option for low overhead
- Node-based deployment option for more control
- Optional relay key header protection
- Configurable public path and upstream path
- Basic health and smoke checks
- Static landing page generation during build
- Windows deployment helper

## Project Structure

```text
api/
  index.js                 Serverless relay runtime
scripts/
  prepare-build.mjs        Static frontend generator
templates/
  landing/                 Bundled landing page templates
Deploy-Windows.ps1         Interactive Windows deployment helper
Run-Deploy-Windows.bat     Convenience launcher
vercel.json                Default Vercel configuration
package.json               Project metadata and build scripts
```

## Requirements

- Node.js 18 or newer
- npm
- Vercel CLI
- A Vercel account

Install the Vercel CLI if needed:

```bash
npm i -g vercel
```

## Quick Start

### Windows Installer

Run the helper script from PowerShell:

```powershell
.\Deploy-Windows.ps1
```

Or double-click:

```text
Run-Deploy-Windows.bat
```

The installer can help you:

- sign in to Vercel
- choose or create a project
- select a deployment mode
- configure relay paths
- set environment variables
- deploy to production
- run basic checks after deployment

### Manual Deployment

Install dependencies if needed:

```bash
npm install
```

Build the static frontend:

```bash
npm run build
```

Deploy with Vercel:

```bash
vercel deploy --prod
```

## Configuration

The Node relay reads these environment variables:

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `TARGET_DOMAIN` | Yes | - | Upstream host or URL to relay traffic to |
| `RELAY_PATH` | Yes | - | Upstream path used by the target service |
| `PUBLIC_RELAY_PATH` | No | `/api` | Public path exposed on the Vercel deployment |
| `RELAY_KEY` | No | - | Optional shared key required in the `x-relay-key` request header |
| `UPSTREAM_TIMEOUT_MS` | No | `45000` | Upstream request timeout |
| `MAX_INFLIGHT` | No | `0` | Optional concurrent request limit. `0` disables the limit |
| `MAX_UP_BPS` | No | `0` | Optional upload bandwidth limit. `0` disables the limit |
| `MAX_DOWN_BPS` | No | `0` | Optional download bandwidth limit. `0` disables the limit |
| `LOG_SAMPLE_RATE` | No | `0` | Optional successful-request log sampling rate |

## Relay Paths

Use paths that start with `/`.

Example:

```text
PUBLIC_RELAY_PATH=/edge
RELAY_PATH=/api
```

Requests sent to:

```text
https://your-project.vercel.app/edge
```

are forwarded to the configured target path:

```text
https://TARGET_DOMAIN/api
```

## Optional Relay Key

If `RELAY_KEY` is set, clients must include:

```http
x-relay-key: your-secret-key
```

Use a long random value. A short or predictable key is not recommended.

## Deployment Modes

### Fast Pipe

Fast Pipe uses Vercel rewrites. It is the simplest option and avoids running the Node function for relay traffic.

Use this mode when you want:

- minimal runtime overhead
- simple path forwarding
- no serverless function logs for relay traffic

### Node Relay

Node Relay runs through `api/index.js`.

Use this mode when you want:

- timeout control
- optional throttling
- optional concurrent request limits
- relay-key validation inside the function
- basic error and performance diagnostics

The Windows helper includes two Node profiles:

```text
BALANCED: 256 concurrent requests, 5 MB/s upload/download caps, 60s upstream timeout, 800s function duration
MAX CONN: 512 concurrent requests, 10 MB/s upload/download caps, 60s upstream timeout, 800s function duration
```

## Troubleshooting

| Status | Meaning | What to Check |
| --- | --- | --- |
| `400` | Invalid request or invalid configuration | Check required environment variables and path values |
| `403` | Relay key rejected | Confirm the client sends the correct `x-relay-key` |
| `404` | Wrong public path | Check `PUBLIC_RELAY_PATH` and the client path |
| `502` | Upstream connection failed | Check `TARGET_DOMAIN`, `RELAY_PATH`, and upstream availability |
| `504` | Upstream timeout | Increase `UPSTREAM_TIMEOUT_MS` or check the target service |

## Notes

- Keep secrets out of git. Use Vercel environment variables for keys and target configuration.
- Do not commit `.vercel`, `.env`, token files, or generated reports.
- Bundled landing templates may include their own license or attribution notices.

## License

MIT
