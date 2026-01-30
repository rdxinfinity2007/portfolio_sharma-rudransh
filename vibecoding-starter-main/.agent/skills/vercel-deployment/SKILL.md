---
name: vercel-deployment
description: Deploy Next.js portfolio to Vercel using MCP tools, CLI commands, and production-ready configurations.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Vercel Deployment Mastery

## Core Philosophy

Deployment is not "pushing code"—it's **delivering value reliably**. We use Vercel's edge network, preview deployments, and analytics to ship with confidence. Every commit is deployable; every deployment is reversible.

## Vercel MCP Integration

### Available MCP Tools

The Vercel MCP provides these tools for deployment automation:

```
1. vercel_deploy       - Deploy to Vercel
2. vercel_list_projects - List all projects
3. vercel_get_project   - Get project details
4. vercel_list_deployments - List deployments
5. vercel_get_deployment - Get deployment details
6. vercel_list_domains  - List domains
7. vercel_add_domain    - Add custom domain
8. vercel_get_env       - Get environment variables
9. vercel_set_env       - Set environment variables
10. vercel_delete_env   - Delete environment variables
```

### Deployment Workflow

```typescript
// MCP-based deployment flow
// 1. List existing projects
await mcp.vercel_list_projects();

// 2. Deploy to production
await mcp.vercel_deploy({
  projectId: "prj_xxx",
  target: "production",
  ref: "main",
});

// 3. Check deployment status
await mcp.vercel_get_deployment({
  deploymentId: "dpl_xxx",
});
```

## Project Configuration

### vercel.json (Full Configuration)

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "framework": "nextjs",
  "buildCommand": "next build",
  "installCommand": "npm install",
  "outputDirectory": ".next",

  "regions": ["iad1", "sfo1", "cdg1"],

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
        },
        {
          "key": "Referrer-Policy",
          "value": "strict-origin-when-cross-origin"
        },
        {
          "key": "Permissions-Policy",
          "value": "camera=(), microphone=(), geolocation=()"
        }
      ]
    },
    {
      "source": "/fonts/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    },
    {
      "source": "/_next/static/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    }
  ],

  "redirects": [
    {
      "source": "/github",
      "destination": "https://github.com/yourusername",
      "permanent": false
    },
    {
      "source": "/linkedin",
      "destination": "https://linkedin.com/in/yourusername",
      "permanent": false
    },
    {
      "source": "/twitter",
      "destination": "https://twitter.com/yourusername",
      "permanent": false
    },
    {
      "source": "/resume",
      "destination": "/resume.pdf",
      "permanent": false
    }
  ],

  "rewrites": [
    {
      "source": "/api/analytics/:path*",
      "destination": "https://analytics.example.com/:path*"
    }
  ],

  "crons": [
    {
      "path": "/api/cron/update-stats",
      "schedule": "0 0 * * *"
    }
  ]
}
```

### next.config.js for Vercel

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Image optimization
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "images.unsplash.com",
      },
      {
        protocol: "https",
        hostname: "cdn.sanity.io",
      },
      {
        protocol: "https",
        hostname: "avatars.githubusercontent.com",
      },
    ],
    formats: ["image/avif", "image/webp"],
  },

  // Experimental features
  experimental: {
    optimizeCss: true,
    optimizePackageImports: ["lucide-react", "framer-motion"],
  },

  // Logging
  logging: {
    fetches: {
      fullUrl: true,
    },
  },

  // Headers (can also be in vercel.json)
  async headers() {
    return [
      {
        source: "/(.*)",
        headers: [
          {
            key: "X-DNS-Prefetch-Control",
            value: "on",
          },
        ],
      },
    ];
  },

  // Redirects
  async redirects() {
    return [
      {
        source: "/blog/old-post",
        destination: "/blog/new-post",
        permanent: true, // 308
      },
    ];
  },
};

module.exports = nextConfig;
```

## Environment Variables

### Setting via MCP

```typescript
// Set production environment variable
await mcp.vercel_set_env({
  projectId: "prj_xxx",
  key: "DATABASE_URL",
  value: "postgresql://...",
  target: ["production"],
  type: "encrypted", // 'plain', 'encrypted', 'secret'
});

// Set for all environments
await mcp.vercel_set_env({
  projectId: "prj_xxx",
  key: "NEXT_PUBLIC_SITE_URL",
  value: "https://yourportfolio.com",
  target: ["production", "preview", "development"],
});
```

### Environment Variable Categories

```env
# .env.local (local development - git ignored)

# ============================================
# PUBLIC (exposed to browser)
# ============================================
NEXT_PUBLIC_SITE_URL=https://yourportfolio.com
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX

# ============================================
# PRIVATE (server-only)
# ============================================
# Email (Resend)
RESEND_API_KEY=re_xxxxx

# CMS (Sanity/Contentful)
SANITY_PROJECT_ID=xxxxx
SANITY_DATASET=production
SANITY_API_TOKEN=xxxxx

# Database (if using)
DATABASE_URL=postgresql://...

# Analytics (PostHog)
POSTHOG_API_KEY=phc_xxxxx

# AI (if applicable)
OPENAI_API_KEY=sk-xxxxx
```

### Accessing Environment Variables

```tsx
// Server Component / Server Action
const apiKey = process.env.RESEND_API_KEY; // ✅ Works

// Client Component
const siteUrl = process.env.NEXT_PUBLIC_SITE_URL; // ✅ Works (has NEXT_PUBLIC_)
const secret = process.env.RESEND_API_KEY; // ❌ undefined (not public)
```

## Domain Configuration

### Add Custom Domain via MCP

```typescript
// Add domain to project
await mcp.vercel_add_domain({
  projectId: "prj_xxx",
  domain: "yourportfolio.com",
});

// Add www subdomain
await mcp.vercel_add_domain({
  projectId: "prj_xxx",
  domain: "www.yourportfolio.com",
  redirect: "yourportfolio.com", // Redirect www to apex
});
```

### DNS Configuration

```
Type    Name    Value                   TTL
A       @       76.76.21.21            Auto
CNAME   www     cname.vercel-dns.com   Auto
```

### SSL/TLS

Vercel automatically provisions and renews SSL certificates via Let's Encrypt.

## Deployment Strategies

### 1. Preview Deployments

Every push to a branch creates a unique preview URL.

```
Feature branch: feature/new-hero
Preview URL: portfolio-git-feature-new-hero-username.vercel.app
```

### 2. Production Deployment

```typescript
// Deploy main branch to production
await mcp.vercel_deploy({
  projectId: "prj_xxx",
  target: "production",
  ref: "main",
});
```

### 3. Instant Rollback

```typescript
// Get list of recent deployments
const deployments = await mcp.vercel_list_deployments({
  projectId: "prj_xxx",
  limit: 10,
});

// Promote previous deployment to production
await mcp.vercel_deploy({
  projectId: "prj_xxx",
  deploymentId: deployments[1].id, // Previous deployment
  target: "production",
});
```

## Edge Middleware

```tsx
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  // Add security headers
  response.headers.set("X-Frame-Options", "DENY");
  response.headers.set("X-Content-Type-Options", "nosniff");

  // Geo-based personalization
  const country = request.geo?.country || "US";
  response.headers.set("X-User-Country", country);

  // A/B Testing
  const variant =
    request.cookies.get("ab-variant")?.value ||
    (Math.random() > 0.5 ? "a" : "b");

  if (!request.cookies.get("ab-variant")) {
    response.cookies.set("ab-variant", variant, {
      maxAge: 60 * 60 * 24 * 30, // 30 days
    });
  }

  return response;
}

export const config = {
  matcher: [
    // Match all paths except static files
    "/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)",
  ],
};
```

## Analytics & Monitoring

### Vercel Analytics (Built-in)

```tsx
// app/layout.tsx
import { Analytics } from "@vercel/analytics/react";
import { SpeedInsights } from "@vercel/speed-insights/next";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  );
}
```

### Web Vitals Tracking

```tsx
// app/layout.tsx
export function reportWebVitals(metric: any) {
  // Send to analytics
  if (metric.label === "web-vital") {
    console.log(metric); // CLS, FID, FCP, LCP, TTFB
  }
}
```

## Cron Jobs

### Configuration

```json
// vercel.json
{
  "crons": [
    {
      "path": "/api/cron/daily-stats",
      "schedule": "0 0 * * *"
    },
    {
      "path": "/api/cron/weekly-newsletter",
      "schedule": "0 9 * * 1"
    }
  ]
}
```

### Cron Handler

```tsx
// app/api/cron/daily-stats/route.ts
import { NextResponse } from "next/server";

export async function GET(request: Request) {
  // Verify cron secret (Vercel sets this header)
  const authHeader = request.headers.get("authorization");

  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return new NextResponse("Unauthorized", { status: 401 });
  }

  try {
    // Your cron job logic
    await updateDailyStats();
    return NextResponse.json({ success: true });
  } catch (error) {
    return NextResponse.json({ error: "Cron failed" }, { status: 500 });
  }
}
```

## Build Optimization

### Analyze Bundle Size

```json
// package.json
{
  "scripts": {
    "analyze": "ANALYZE=true next build"
  }
}
```

```js
// next.config.js
const withBundleAnalyzer = require("@next/bundle-analyzer")({
  enabled: process.env.ANALYZE === "true",
});

module.exports = withBundleAnalyzer(nextConfig);
```

### Tree Shaking Imports

```tsx
// ❌ Bad: Imports entire library
import { motion, AnimatePresence, useAnimation } from "framer-motion";

// ✅ Good: Use optimizePackageImports in next.config.js
// Then import normally - Next.js handles tree shaking
```

```js
// next.config.js
module.exports = {
  experimental: {
    optimizePackageImports: [
      "lucide-react",
      "framer-motion",
      "@radix-ui/react-icons",
    ],
  },
};
```

## Error Handling in Production

### Global Error Boundary

```tsx
// app/global-error.tsx
"use client";

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <div className="flex min-h-screen items-center justify-center">
          <div className="text-center">
            <h1 className="text-4xl font-bold">Something went wrong!</h1>
            <p className="mt-4 text-muted">Error ID: {error.digest}</p>
            <button
              onClick={reset}
              className="mt-6 rounded-lg bg-brand-500 px-6 py-3 text-white"
            >
              Try again
            </button>
          </div>
        </div>
      </body>
    </html>
  );
}
```

### Error Monitoring Integration

```tsx
// lib/error-tracking.ts
export function logError(error: Error, context?: Record<string, any>) {
  // Send to error tracking service
  if (process.env.NODE_ENV === "production") {
    // Sentry, LogRocket, etc.
    console.error("Production error:", error, context);
  }
}
```

## Pre-Deployment Checklist

### Build Verification

```bash
# Run locally before deploying
npm run build
npm run lint
npm run type-check
```

### Performance Checks

- [ ] Lighthouse score > 90
- [ ] First Contentful Paint < 1.8s
- [ ] Largest Contentful Paint < 2.5s
- [ ] Cumulative Layout Shift < 0.1
- [ ] Time to Interactive < 3.8s

### Security Checks

- [ ] No secrets in client code
- [ ] Security headers configured
- [ ] HTTPS enforced
- [ ] CSP headers (if applicable)

### SEO Checks

- [ ] Sitemap accessible
- [ ] Robots.txt correct
- [ ] OG images loading
- [ ] Canonical URLs set

## Troubleshooting

### "Build works locally but fails on Vercel"

1. **Case sensitivity**: Linux is case-sensitive; Windows/Mac aren't

   ```tsx
   // ❌ Might work locally, fails on Vercel
   import Button from "./button";

   // ✅ Match exact filename case
   import Button from "./Button";
   ```

2. **Environment variables**: Pull from Vercel

   ```bash
   vercel env pull .env.local
   ```

3. **Node version mismatch**:
   ```json
   // package.json
   {
     "engines": {
       "node": "20.x"
     }
   }
   ```

### "Dynamic usage errors"

Using `headers()`, `cookies()`, or `searchParams` opts out of static generation.

```tsx
// Force dynamic rendering
export const dynamic = "force-dynamic";

// Or use generateStaticParams for known paths
export async function generateStaticParams() {
  return [{ slug: "project-1" }, { slug: "project-2" }];
}
```

### "Cold start latency"

- Use Edge Runtime where possible
- Reduce function bundle size
- Use ISR instead of SSR when data is cacheable

```tsx
// Use Edge Runtime
export const runtime = "edge";

// Or use ISR
export const revalidate = 3600; // Revalidate every hour
```

## CLI Quick Reference

```bash
# Deploy to production
vercel --prod

# Deploy preview
vercel

# Pull environment variables
vercel env pull .env.local

# Link to existing project
vercel link

# Check deployment logs
vercel logs <url>

# Inspect specific deployment
vercel inspect <url>

# List all deployments
vercel ls

# Rollback to specific deployment
vercel alias set <deployment-url> <domain>

# Add domain
vercel domains add yourportfolio.com
```
