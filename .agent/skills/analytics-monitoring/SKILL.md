---
name: analytics-monitoring
description: Implement analytics, error monitoring, and performance tracking with Vercel Analytics, Google Analytics, Sentry, and custom event tracking.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Analytics & Monitoring

## Overview

A production portfolio needs:

1. **Web Analytics** - Understand visitor behavior
2. **Performance Monitoring** - Track Core Web Vitals
3. **Error Tracking** - Catch and fix issues fast
4. **Custom Events** - Track conversions and interactions

## Vercel Analytics (Recommended)

### Installation

```bash
npm install @vercel/analytics @vercel/speed-insights
```

### Setup

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

### Custom Events

```tsx
// lib/analytics.ts
import { track } from "@vercel/analytics";

/**
 * Track custom events with type safety.
 */
export const trackEvent = {
  // Contact form submission
  contactSubmitted: () => track("contact_submitted"),

  // Project viewed
  projectViewed: (projectSlug: string) =>
    track("project_viewed", { project: projectSlug }),

  // Resume downloaded
  resumeDownloaded: () => track("resume_downloaded"),

  // External link clicked
  externalLinkClicked: (url: string) => track("external_link_clicked", { url }),

  // Social link clicked
  socialClicked: (platform: string) => track("social_clicked", { platform }),

  // CTA clicked
  ctaClicked: (location: string, action: string) =>
    track("cta_clicked", { location, action }),
};

// Usage in components
import { trackEvent } from "@/lib/analytics";

function ContactForm() {
  const onSuccess = () => {
    trackEvent.contactSubmitted();
  };
}

function ProjectCard({ project }) {
  return (
    <Link
      href={`/projects/${project.slug}`}
      onClick={() => trackEvent.projectViewed(project.slug)}
    >
      {/* ... */}
    </Link>
  );
}
```

## Google Analytics 4

### Setup with @next/third-parties

```bash
npm install @next/third-parties
```

```tsx
// app/layout.tsx
import { GoogleAnalytics } from "@next/third-parties/google";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}
        <GoogleAnalytics gaId="G-XXXXXXXXXX" />
      </body>
    </html>
  );
}
```

### Manual Setup (More Control)

```tsx
// components/google-analytics.tsx
"use client";

import Script from "next/script";
import { usePathname, useSearchParams } from "next/navigation";
import { useEffect, Suspense } from "react";

const GA_MEASUREMENT_ID = process.env.NEXT_PUBLIC_GA_MEASUREMENT_ID;

/**
 * Google Analytics 4 implementation.
 * @security Only loads in production to avoid polluting dev data.
 */
function GoogleAnalyticsInner() {
  const pathname = usePathname();
  const searchParams = useSearchParams();

  useEffect(() => {
    if (!GA_MEASUREMENT_ID || process.env.NODE_ENV !== "production") return;

    const url = pathname + searchParams.toString();

    // Track page view
    window.gtag?.("config", GA_MEASUREMENT_ID, {
      page_path: url,
    });
  }, [pathname, searchParams]);

  if (!GA_MEASUREMENT_ID || process.env.NODE_ENV !== "production") {
    return null;
  }

  return (
    <>
      <Script
        strategy="afterInteractive"
        src={`https://www.googletagmanager.com/gtag/js?id=${GA_MEASUREMENT_ID}`}
      />
      <Script
        id="google-analytics"
        strategy="afterInteractive"
        dangerouslySetInnerHTML={{
          __html: `
            window.dataLayer = window.dataLayer || [];
            function gtag(){dataLayer.push(arguments);}
            gtag('js', new Date());
            gtag('config', '${GA_MEASUREMENT_ID}', {
              page_path: window.location.pathname,
            });
          `,
        }}
      />
    </>
  );
}

export function GoogleAnalytics() {
  return (
    <Suspense fallback={null}>
      <GoogleAnalyticsInner />
    </Suspense>
  );
}
```

### GA4 Event Tracking

```typescript
// lib/gtag.ts

declare global {
  interface Window {
    gtag: (
      command: "config" | "event" | "js" | "set",
      targetId: string,
      config?: Record<string, unknown>,
    ) => void;
  }
}

export const GA_MEASUREMENT_ID = process.env.NEXT_PUBLIC_GA_MEASUREMENT_ID;

/**
 * Track page views.
 */
export function pageview(url: string) {
  if (!GA_MEASUREMENT_ID) return;
  window.gtag?.("config", GA_MEASUREMENT_ID, {
    page_path: url,
  });
}

/**
 * Track events.
 */
export function event(
  action: string,
  category: string,
  label?: string,
  value?: number,
) {
  if (!GA_MEASUREMENT_ID) return;
  window.gtag?.("event", action, {
    event_category: category,
    event_label: label,
    value: value,
  });
}

// Predefined events
export const gaEvents = {
  contactSubmit: () => event("submit", "contact_form"),
  resumeDownload: () => event("download", "resume"),
  projectView: (name: string) => event("view", "project", name),
  socialClick: (platform: string) => event("click", "social", platform),
};
```

### Type Definitions

```typescript
// types/gtag.d.ts
interface GtagEventParams {
  event_category?: string;
  event_label?: string;
  value?: number;
  page_path?: string;
  [key: string]: unknown;
}

interface Window {
  gtag: (
    command: "config" | "event" | "js" | "set",
    targetId: string,
    config?: GtagEventParams,
  ) => void;
  dataLayer: unknown[];
}
```

## Sentry Error Monitoring

### Installation

```bash
npx @sentry/wizard@latest -i nextjs
```

Or manually:

```bash
npm install @sentry/nextjs
```

### Configuration

```javascript
// sentry.client.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,

  // Performance monitoring
  tracesSampleRate: 1.0, // Capture 100% of transactions (reduce in prod)

  // Session replay
  replaysSessionSampleRate: 0.1, // Capture 10% of sessions
  replaysOnErrorSampleRate: 1.0, // Capture 100% on error

  integrations: [
    Sentry.replayIntegration({
      maskAllText: false,
      blockAllMedia: false,
    }),
  ],

  // Only enable in production
  enabled: process.env.NODE_ENV === "production",

  // Filter out known non-issues
  ignoreErrors: [
    "ResizeObserver loop limit exceeded",
    "Non-Error promise rejection",
  ],

  beforeSend(event) {
    // Scrub sensitive data
    if (event.request?.cookies) {
      delete event.request.cookies;
    }
    return event;
  },
});
```

```javascript
// sentry.server.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 1.0,
  enabled: process.env.NODE_ENV === "production",
});
```

```javascript
// sentry.edge.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 1.0,
  enabled: process.env.NODE_ENV === "production",
});
```

### Next.js Config

```javascript
// next.config.js
const { withSentryConfig } = require("@sentry/nextjs");

const nextConfig = {
  // Your config
};

module.exports = withSentryConfig(
  nextConfig,
  {
    // Sentry webpack plugin options
    silent: true,
    org: "your-org",
    project: "your-project",
  },
  {
    // Sentry SDK options
    widenClientFileUpload: true,
    tunnelRoute: "/monitoring-tunnel",
    hideSourceMaps: true,
    disableLogger: true,
  },
);
```

### Error Boundary

```tsx
// components/error-boundary.tsx
"use client";

import * as Sentry from "@sentry/nextjs";
import { useEffect } from "react";

interface ErrorBoundaryProps {
  error: Error & { digest?: string };
  reset: () => void;
}

/**
 * Global error boundary for catching and reporting errors.
 */
export default function GlobalError({ error, reset }: ErrorBoundaryProps) {
  useEffect(() => {
    Sentry.captureException(error);
  }, [error]);

  return (
    <html>
      <body>
        <div className="flex min-h-screen flex-col items-center justify-center">
          <h2 className="text-2xl font-bold">Something went wrong!</h2>
          <p className="mt-2 text-zinc-400">
            We've been notified and are working on a fix.
          </p>
          <button
            onClick={reset}
            className="mt-4 rounded-lg bg-brand-500 px-4 py-2 text-white"
          >
            Try again
          </button>
        </div>
      </body>
    </html>
  );
}
```

### Manual Error Capture

```typescript
// lib/error.ts
import * as Sentry from "@sentry/nextjs";

/**
 * Capture error with context.
 */
export function captureError(error: Error, context?: Record<string, unknown>) {
  Sentry.captureException(error, {
    extra: context,
  });
}

/**
 * Capture message/warning.
 */
export function captureMessage(
  message: string,
  level: "info" | "warning" | "error" = "info",
) {
  Sentry.captureMessage(message, level);
}

/**
 * Set user context for error tracking.
 */
export function setUser(user: { id: string; email?: string }) {
  Sentry.setUser(user);
}

/**
 * Clear user context (on logout).
 */
export function clearUser() {
  Sentry.setUser(null);
}

// Usage
try {
  await riskyOperation();
} catch (error) {
  captureError(error as Error, {
    operation: "riskyOperation",
    userId: user.id,
  });
}
```

## Core Web Vitals Monitoring

### Custom Web Vitals Reporting

```tsx
// components/web-vitals.tsx
"use client";

import { useReportWebVitals } from "next/web-vitals";

/**
 * Report Web Vitals to analytics.
 */
export function WebVitals() {
  useReportWebVitals((metric) => {
    // Log to console in development
    if (process.env.NODE_ENV === "development") {
      console.log(metric);
    }

    // Send to Google Analytics
    const { id, name, label, value } = metric;
    window.gtag?.("event", name, {
      event_category: label === "web-vital" ? "Web Vitals" : "Next.js Metrics",
      event_label: id,
      value: Math.round(name === "CLS" ? value * 1000 : value),
      non_interaction: true,
    });

    // Or send to custom endpoint
    fetch("/api/vitals", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        id,
        name,
        label,
        value,
        page: window.location.pathname,
      }),
    });
  });

  return null;
}
```

### Vitals API Endpoint

```typescript
// app/api/vitals/route.ts
import { NextRequest, NextResponse } from "next/server";

interface VitalsData {
  id: string;
  name: string;
  label: string;
  value: number;
  page: string;
}

export async function POST(request: NextRequest) {
  try {
    const data: VitalsData = await request.json();

    // Log vitals (replace with your analytics service)
    console.log("Web Vital:", {
      metric: data.name,
      value: data.value,
      page: data.page,
      rating: getVitalRating(data.name, data.value),
    });

    // Send to analytics service
    // await sendToAnalytics(data);

    return NextResponse.json({ success: true });
  } catch (error) {
    return NextResponse.json({ success: false }, { status: 500 });
  }
}

function getVitalRating(
  name: string,
  value: number,
): "good" | "needs-improvement" | "poor" {
  const thresholds: Record<string, [number, number]> = {
    LCP: [2500, 4000],
    FID: [100, 300],
    CLS: [0.1, 0.25],
    FCP: [1800, 3000],
    TTFB: [800, 1800],
    INP: [200, 500],
  };

  const [good, poor] = thresholds[name] || [0, 0];

  if (value <= good) return "good";
  if (value <= poor) return "needs-improvement";
  return "poor";
}
```

## Custom Event Tracking System

### Event Types

```typescript
// lib/events/types.ts

/**
 * All trackable events with their payloads.
 */
export type AnalyticsEvent =
  | { name: "page_view"; properties: { path: string; title: string } }
  | { name: "contact_submitted"; properties: { subject: string } }
  | {
      name: "project_viewed";
      properties: { projectId: string; projectName: string };
    }
  | { name: "resume_downloaded"; properties: Record<string, never> }
  | { name: "social_clicked"; properties: { platform: string } }
  | { name: "cta_clicked"; properties: { location: string; action: string } }
  | { name: "scroll_depth"; properties: { depth: number; page: string } }
  | { name: "time_on_page"; properties: { seconds: number; page: string } };

export type EventName = AnalyticsEvent["name"];

export type EventProperties<T extends EventName> = Extract<
  AnalyticsEvent,
  { name: T }
>["properties"];
```

### Event Tracker

```typescript
// lib/events/tracker.ts
import { track as vercelTrack } from "@vercel/analytics";
import * as Sentry from "@sentry/nextjs";
import type { EventName, EventProperties } from "./types";

/**
 * Unified event tracking across all analytics providers.
 */
class EventTracker {
  private debug = process.env.NODE_ENV === "development";

  track<T extends EventName>(name: T, properties: EventProperties<T>) {
    if (this.debug) {
      console.log(`[Analytics] ${name}`, properties);
    }

    // Vercel Analytics
    vercelTrack(name, properties);

    // Google Analytics
    window.gtag?.("event", name, properties);

    // Sentry breadcrumb
    Sentry.addBreadcrumb({
      category: "analytics",
      message: name,
      data: properties,
      level: "info",
    });
  }

  // Convenience methods
  pageView(path: string, title: string) {
    this.track("page_view", { path, title });
  }

  contactSubmitted(subject: string) {
    this.track("contact_submitted", { subject });
  }

  projectViewed(projectId: string, projectName: string) {
    this.track("project_viewed", { projectId, projectName });
  }

  resumeDownloaded() {
    this.track("resume_downloaded", {});
  }

  socialClicked(platform: string) {
    this.track("social_clicked", { platform });
  }

  ctaClicked(location: string, action: string) {
    this.track("cta_clicked", { location, action });
  }
}

export const analytics = new EventTracker();
```

## Scroll Depth Tracking

```tsx
// hooks/use-scroll-tracking.ts
"use client";

import { useEffect, useRef } from "react";
import { analytics } from "@/lib/events/tracker";

/**
 * Track how far users scroll on a page.
 */
export function useScrollTracking() {
  const tracked = useRef(new Set<number>());
  const thresholds = [25, 50, 75, 100];

  useEffect(() => {
    const handleScroll = () => {
      const scrollHeight =
        document.documentElement.scrollHeight - window.innerHeight;
      const scrolled = (window.scrollY / scrollHeight) * 100;

      thresholds.forEach((threshold) => {
        if (scrolled >= threshold && !tracked.current.has(threshold)) {
          tracked.current.add(threshold);
          analytics.track("scroll_depth", {
            depth: threshold,
            page: window.location.pathname,
          });
        }
      });
    };

    window.addEventListener("scroll", handleScroll, { passive: true });
    return () => window.removeEventListener("scroll", handleScroll);
  }, []);
}
```

## Time on Page Tracking

```tsx
// hooks/use-time-tracking.ts
"use client";

import { useEffect, useRef } from "react";
import { analytics } from "@/lib/events/tracker";

/**
 * Track time spent on a page.
 */
export function useTimeTracking() {
  const startTime = useRef(Date.now());
  const tracked = useRef(new Set<number>());
  const intervals = [30, 60, 120, 300]; // seconds

  useEffect(() => {
    const checkTime = () => {
      const elapsed = Math.floor((Date.now() - startTime.current) / 1000);

      intervals.forEach((interval) => {
        if (elapsed >= interval && !tracked.current.has(interval)) {
          tracked.current.add(interval);
          analytics.track("time_on_page", {
            seconds: interval,
            page: window.location.pathname,
          });
        }
      });
    };

    const timer = setInterval(checkTime, 5000);
    return () => clearInterval(timer);
  }, []);

  // Track on unmount
  useEffect(() => {
    return () => {
      const elapsed = Math.floor((Date.now() - startTime.current) / 1000);
      if (elapsed > 5) {
        analytics.track("time_on_page", {
          seconds: elapsed,
          page: window.location.pathname,
        });
      }
    };
  }, []);
}
```

## Privacy & GDPR

### Cookie Consent

```tsx
// components/cookie-consent.tsx
"use client";

import { useState, useEffect } from "react";
import { motion, AnimatePresence } from "framer-motion";

export function CookieConsent() {
  const [showBanner, setShowBanner] = useState(false);

  useEffect(() => {
    const consent = localStorage.getItem("cookie-consent");
    if (!consent) {
      setShowBanner(true);
    }
  }, []);

  const accept = () => {
    localStorage.setItem("cookie-consent", "accepted");
    setShowBanner(false);
    // Enable analytics here if they were disabled
    window.gtag?.("consent", "update", {
      analytics_storage: "granted",
    });
  };

  const decline = () => {
    localStorage.setItem("cookie-consent", "declined");
    setShowBanner(false);
    window.gtag?.("consent", "update", {
      analytics_storage: "denied",
    });
  };

  return (
    <AnimatePresence>
      {showBanner && (
        <motion.div
          initial={{ y: 100, opacity: 0 }}
          animate={{ y: 0, opacity: 1 }}
          exit={{ y: 100, opacity: 0 }}
          className="fixed bottom-0 left-0 right-0 z-50 border-t border-zinc-800 bg-zinc-950 p-4"
        >
          <div className="container flex flex-col items-center justify-between gap-4 sm:flex-row">
            <p className="text-sm text-zinc-400">
              This site uses cookies to improve your experience and analyze
              traffic.
            </p>
            <div className="flex gap-2">
              <button
                onClick={decline}
                className="rounded-lg border border-zinc-700 px-4 py-2 text-sm"
              >
                Decline
              </button>
              <button
                onClick={accept}
                className="rounded-lg bg-brand-500 px-4 py-2 text-sm text-white"
              >
                Accept
              </button>
            </div>
          </div>
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

### Consent-Aware Analytics

```tsx
// lib/analytics-consent.ts

/**
 * Check if user has consented to analytics.
 */
export function hasAnalyticsConsent(): boolean {
  if (typeof window === "undefined") return false;
  return localStorage.getItem("cookie-consent") === "accepted";
}

/**
 * Conditionally track events based on consent.
 */
export function trackWithConsent(
  eventName: string,
  properties?: Record<string, unknown>,
) {
  if (!hasAnalyticsConsent()) return;

  // Your tracking logic
  window.gtag?.("event", eventName, properties);
}
```

## Environment Variables

```env
# .env.local

# Vercel Analytics (auto-configured on Vercel)
# No env needed

# Google Analytics
NEXT_PUBLIC_GA_MEASUREMENT_ID=G-XXXXXXXXXX

# Sentry
NEXT_PUBLIC_SENTRY_DSN=https://xxxxx@xxx.ingest.sentry.io/xxxxx
SENTRY_AUTH_TOKEN=sntrys_xxxxx
SENTRY_ORG=your-org
SENTRY_PROJECT=your-project
```

## Testing Checklist

- [ ] Analytics loads in production only (or with debug flag)
- [ ] Page views tracked on navigation
- [ ] Custom events fire correctly
- [ ] Web Vitals reported
- [ ] Errors captured in Sentry with context
- [ ] Source maps uploaded for stack traces
- [ ] Cookie consent working
- [ ] No PII sent to analytics
- [ ] Analytics blocked by ad blockers gracefully
