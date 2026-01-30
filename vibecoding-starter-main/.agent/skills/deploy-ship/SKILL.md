---
name: deploy-ship
description: Deploy applications to production using Vercel CLI commands.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Deployment & DevOps with Vercel

## Core Philosophy

Deployment isn't just "putting code online." It's about **reliability**, **observability**, and **performance**. We use Vercel not just as a host, but as an infrastructure provider.

## Advanced Configuration (`vercel.json`)

The `vercel.json` file is your infrastructure-as-code configuration.

### 1. Headers (Security & Caching)

Secure your app by default.

```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-XSS-Protection",
          "value": "1; mode=block"
        }
      ]
    }
  ]
}
```

### 2. Rewrites & Redirects

Handle legacy paths or proxy external APIs (avoiding CORS).

```json
{
  "rewrites": [
    {
      "source": "/api/external/:path*",
      "destination": "https://api.thirdparty.com/:path*"
    }
  ],
  "redirects": [
    {
      "source": "/old-blog/:slug",
      "destination": "/news/:slug",
      "permanent": true
    }
  ]
}
```

## Environment Variables & Config

### Hierarchy

1. **Production**: Used for `vercel --prod`. Active for live users.
2. **Preview**: Used for PRs and branch deployments.
3. **Development**: Used for `vercel dev`.

**Crucial**: Vercel encrypts variables at rest.

- `NEXT_PUBLIC_*`: Exposed to the browser (Client Components).
- **No prefix**: Server-only (Server Components / API Routes).

### System Variables

- `VERCEL_URL`: The domain of the current deployment (e.g., `my-app-git-feature.vercel.app`).
- `VERCEL_ENV`: `production`, `preview`, or `development`.
- `VERCEL_REGION`: The region the function is executing in (e.g., `sfo1`).

## Edge Middleware vs Serverless Functions

### Middleware (`middleware.ts`)

Runs **before** every request. Uses Vercel Edge Runtime (limited Node APIs, super fast).

- **Use for**: Authentication checks, A/B testing, Geo-blocking, rewriting paths based on locale.

```typescript
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // Block access from strict countries
  if (request.geo?.country === "XX") {
    return new NextResponse("Blocked", { status: 403 });
  }

  // Auth check
  const token = request.cookies.get("token");
  if (!token) return NextResponse.redirect(new URL("/login", request.url));
}
```

### Serverless Functions

Runs your API routes and Server Actions. Full Node.js runtime.

- **Use for**: Database connections, heavy computation, using libraries with native dependencies.

## CI/CD Pipeline

### Preview Deployments

Every push to a branch creates a unique URL.

- **Bot Comments**: Vercel bot comments on PRs with the preview URL.
- **Visual Testing**: Use these URLs for visual regression testing (Percy/Chromatic).

### Comments

Enable Vercel Comments to allow stakeholders to leave feedback directly on the UI of preview functionality.

## Troubleshooting Production

### 1. "It works locally but fails on production"

- **Environment Variables**: helper: `vercel env pull .env.local` to sync prod envs to local.
- **Case Sensitivity**: Linux (Vercel) is case specific; Windows/Mac are often not. Check import paths! (`./Button` vs `./button`).

### 2. Build Failures

- **Type Errors**: Production builds run `tsc`. Fix your TypeScript errors.
- **Lint Errors**: Production builds run `eslint`. Fix your lint errors.
- **Dynamic Usage**: Using `headers()` or `cookies()` opts a page out of Static Generation.

## Vercel CLI Power User Commands

- `vercel inspect <deployment-url>`: Get details about a specific build.
- `vercel logs <url>`: Stream logs for a running deployment within the terminal.
- `vercel alias set <deployment-url> <custom-domain>`: Manually point a domain to a specific build (instant rollback).
