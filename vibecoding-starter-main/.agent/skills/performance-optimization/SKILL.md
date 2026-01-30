---
name: performance-optimization
description: Optimize Next.js portfolio for Core Web Vitals, bundle size, loading performance, and achieve 90+ Lighthouse scores.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Performance Optimization Mastery

## Core Philosophy

Performance is not an afterthought—it's a **feature**. A slow portfolio loses visitors and creates a poor first impression. We optimize for Core Web Vitals because Google uses them for ranking, but more importantly, because users deserve fast experiences.

## Core Web Vitals Targets

| Metric                                | Target  | Description                           |
| ------------------------------------- | ------- | ------------------------------------- |
| **LCP** (Largest Contentful Paint)    | < 2.5s  | How fast the main content loads       |
| **FID** (First Input Delay) / **INP** | < 100ms | How responsive to user input          |
| **CLS** (Cumulative Layout Shift)     | < 0.1   | Visual stability (no jumping content) |
| **FCP** (First Contentful Paint)      | < 1.8s  | First visible content                 |
| **TTFB** (Time to First Byte)         | < 600ms | Server response time                  |

## Image Optimization

### Next.js Image Component (Essential)

```tsx
import Image from 'next/image';

// ✅ Local images (automatic optimization)
import heroImage from '@/public/images/hero.jpg';

<Image
  src={heroImage}
  alt="Hero section"
  placeholder="blur"          // Blur-up loading effect
  priority                    // Load immediately (above-the-fold)
  className="object-cover"
  sizes="100vw"              // Responsive sizing hints
/>

// ✅ Remote images
<Image
  src="https://example.com/image.jpg"
  alt="Project screenshot"
  width={1200}
  height={630}
  loading="lazy"             // Default behavior
  quality={85}               // Compression (default: 75)
  className="object-cover"
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```

### Image Configuration

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    // Modern formats
    formats: ["image/avif", "image/webp"],

    // Remote patterns
    remotePatterns: [
      {
        protocol: "https",
        hostname: "images.unsplash.com",
      },
      {
        protocol: "https",
        hostname: "cdn.sanity.io",
      },
    ],

    // Device breakpoints
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],

    // Minimize processing (if using external CDN)
    // unoptimized: true,
  },
};

module.exports = nextConfig;
```

### Image Best Practices

```tsx
// 1. Always use sizes prop for responsive images
<Image
  src="/project.jpg"
  alt="Project"
  fill
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>

// 2. Use priority for above-the-fold images (hero, profile)
<Image src={heroImage} alt="Hero" priority />

// 3. Lazy load below-the-fold images (default)
<Image src={projectImage} alt="Project" loading="lazy" />

// 4. Prevent CLS with aspect ratio containers
<div className="relative aspect-video">
  <Image src="/video-thumb.jpg" alt="Video" fill className="object-cover" />
</div>
```

## Font Optimization

### Next.js Font System

```tsx
// app/layout.tsx
import { Inter, Playfair_Display } from "next/font/google";
import localFont from "next/font/local";

// Google Fonts (self-hosted, optimized)
const inter = Inter({
  subsets: ["latin"],
  display: "swap", // Prevent FOIT (flash of invisible text)
  variable: "--font-inter",
  preload: true,
});

const playfair = Playfair_Display({
  subsets: ["latin"],
  display: "swap",
  variable: "--font-playfair",
  weight: ["400", "700"], // Only load needed weights
});

// Local fonts (maximum control)
const calSans = localFont({
  src: "../public/fonts/CalSans-SemiBold.woff2",
  variable: "--font-cal",
  display: "swap",
  preload: true,
});

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html
      lang="en"
      className={`${inter.variable} ${playfair.variable} ${calSans.variable}`}
    >
      <body className="font-sans">{children}</body>
    </html>
  );
}
```

### Font Subsetting

```tsx
// Only load characters you need
const inter = Inter({
  subsets: ["latin"], // Not 'latin-ext' unless needed
  display: "swap",
});

// For icons/special characters, use local subset
const customFont = localFont({
  src: "../fonts/custom-subset.woff2", // Pre-subsetted
});
```

## Bundle Optimization

### Package Import Optimization

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    // Tree-shake package imports automatically
    optimizePackageImports: [
      "lucide-react",
      "framer-motion",
      "@radix-ui/react-icons",
      "date-fns",
      "lodash",
    ],
  },
};
```

### Dynamic Imports (Code Splitting)

```tsx
import dynamic from "next/dynamic";

// Heavy components loaded on demand
const HeavyChart = dynamic(() => import("@/components/heavy-chart"), {
  loading: () => <Skeleton className="h-96" />,
  ssr: false, // Client-only component
});

// Modal loaded when needed
const ContactModal = dynamic(() => import("@/components/contact-modal"));

// Animation components (reduce initial bundle)
const MotionDiv = dynamic(
  () => import("framer-motion").then((mod) => mod.motion.div),
  { ssr: false },
);
```

### Analyze Bundle Size

```json
// package.json
{
  "scripts": {
    "analyze": "ANALYZE=true next build"
  },
  "devDependencies": {
    "@next/bundle-analyzer": "^14.0.0"
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

### Import Optimization Patterns

```tsx
// ❌ Bad: Imports entire library
import _ from "lodash";
_.debounce(fn, 300);

// ✅ Good: Tree-shakeable import
import debounce from "lodash/debounce";
debounce(fn, 300);

// ❌ Bad: All icons imported
import * as Icons from "lucide-react";

// ✅ Good: Only what you need
import { ArrowRight, Github, Mail } from "lucide-react";

// ❌ Bad: Moment.js (huge library)
import moment from "moment";

// ✅ Good: date-fns (tree-shakeable)
import { format, formatDistance } from "date-fns";
```

## Rendering Strategies

### Static Generation (Fastest)

```tsx
// app/projects/page.tsx
// Statically generated at build time

export default async function ProjectsPage() {
  const projects = await getProjects(); // Fetched at build time

  return (
    <div>
      {projects.map((p) => (
        <ProjectCard key={p.id} project={p} />
      ))}
    </div>
  );
}

// Generate static paths
export async function generateStaticParams() {
  const projects = await getProjects();
  return projects.map((p) => ({ slug: p.slug }));
}
```

### Incremental Static Regeneration (ISR)

```tsx
// app/blog/page.tsx
// Revalidate every hour

export const revalidate = 3600; // seconds

export default async function BlogPage() {
  const posts = await getPosts();
  return <BlogList posts={posts} />;
}

// Per-fetch revalidation
async function getPosts() {
  const res = await fetch("https://api.example.com/posts", {
    next: { revalidate: 3600 },
  });
  return res.json();
}
```

### On-Demand Revalidation

```tsx
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from "next/cache";
import { NextRequest, NextResponse } from "next/server";

export async function POST(request: NextRequest) {
  const { secret, path, tag } = await request.json();

  if (secret !== process.env.REVALIDATION_SECRET) {
    return NextResponse.json({ error: "Invalid secret" }, { status: 401 });
  }

  if (path) {
    revalidatePath(path);
  }

  if (tag) {
    revalidateTag(tag);
  }

  return NextResponse.json({ revalidated: true });
}
```

## Loading Performance

### Streaming with Suspense

```tsx
// app/page.tsx
import { Suspense } from "react";
import { HeroSection } from "@/components/sections/hero";
import { ProjectsSkeleton } from "@/components/skeletons";

// Projects component fetches its own data
async function ProjectsSection() {
  const projects = await getProjects(); // This can be slow
  return <ProjectsList projects={projects} />;
}

export default function HomePage() {
  return (
    <main>
      {/* Hero loads immediately */}
      <HeroSection />

      {/* Projects stream in when ready */}
      <Suspense fallback={<ProjectsSkeleton />}>
        <ProjectsSection />
      </Suspense>
    </main>
  );
}
```

### Loading States

```tsx
// app/projects/loading.tsx
import { Skeleton } from "@/components/ui/skeleton";

export default function ProjectsLoading() {
  return (
    <div className="container py-12">
      {/* Header skeleton */}
      <div className="mx-auto max-w-2xl text-center">
        <Skeleton className="mx-auto h-4 w-24" />
        <Skeleton className="mx-auto mt-2 h-10 w-80" />
        <Skeleton className="mx-auto mt-4 h-6 w-96" />
      </div>

      {/* Grid skeleton */}
      <div className="mt-16 grid gap-8 md:grid-cols-2 lg:grid-cols-3">
        {Array.from({ length: 6 }).map((_, i) => (
          <div key={i} className="rounded-2xl border border-zinc-800 p-6">
            <Skeleton className="aspect-video rounded-lg" />
            <Skeleton className="mt-4 h-6 w-3/4" />
            <Skeleton className="mt-2 h-4 w-full" />
            <Skeleton className="mt-2 h-4 w-2/3" />
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Skeleton Component

```tsx
// components/ui/skeleton.tsx
import { cn } from "@/lib/utils";

interface SkeletonProps extends React.HTMLAttributes<HTMLDivElement> {}

export function Skeleton({ className, ...props }: SkeletonProps) {
  return (
    <div
      className={cn("animate-pulse rounded-md bg-zinc-800", className)}
      {...props}
    />
  );
}
```

## Preloading & Prefetching

### Link Prefetching

```tsx
import Link from 'next/link';

// Automatic prefetch on viewport enter (production)
<Link href="/projects">Projects</Link>

// Disable prefetch for less important links
<Link href="/privacy" prefetch={false}>Privacy Policy</Link>
```

### Resource Hints

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <head>
        {/* Preconnect to external origins */}
        <link rel="preconnect" href="https://fonts.googleapis.com" />
        <link
          rel="preconnect"
          href="https://fonts.gstatic.com"
          crossOrigin="anonymous"
        />

        {/* DNS prefetch for analytics */}
        <link rel="dns-prefetch" href="https://www.googletagmanager.com" />

        {/* Preload critical assets */}
        <link
          rel="preload"
          href="/fonts/CalSans-SemiBold.woff2"
          as="font"
          type="font/woff2"
          crossOrigin="anonymous"
        />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

## Animation Performance

### GPU-Accelerated Properties

```tsx
// ✅ Good: GPU-accelerated (transform, opacity)
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
/>

// ❌ Bad: Triggers layout (width, height, left, top)
<motion.div
  initial={{ width: 0 }}
  animate={{ width: 200 }}
/>
```

### Reduce Motion for Accessibility

```tsx
"use client";

import { useReducedMotion } from "framer-motion";

export function AnimatedComponent() {
  const shouldReduceMotion = useReducedMotion();

  return (
    <motion.div
      initial={shouldReduceMotion ? false : { opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={shouldReduceMotion ? { duration: 0 } : { duration: 0.5 }}
    >
      Content
    </motion.div>
  );
}
```

### CSS-Based Animations (Lower Overhead)

```tsx
// For simple animations, CSS is lighter than Framer Motion

// tailwind.config.ts
animation: {
  'fade-in': 'fade-in 0.5s ease-out forwards',
  'slide-up': 'slide-up 0.5s ease-out forwards',
},
keyframes: {
  'fade-in': {
    from: { opacity: '0' },
    to: { opacity: '1' },
  },
  'slide-up': {
    from: { opacity: '0', transform: 'translateY(20px)' },
    to: { opacity: '1', transform: 'translateY(0)' },
  },
},

// Usage
<div className="animate-fade-in">Content</div>
```

## Caching Strategies

### Fetch Caching

```tsx
// Cache indefinitely (static data)
const data = await fetch("https://api.example.com/static", {
  cache: "force-cache",
});

// No caching (always fresh)
const data = await fetch("https://api.example.com/realtime", {
  cache: "no-store",
});

// Revalidate after time
const data = await fetch("https://api.example.com/posts", {
  next: { revalidate: 3600 },
});

// Tag-based revalidation
const data = await fetch("https://api.example.com/posts", {
  next: { tags: ["posts"] },
});
```

### React Cache

```tsx
import { cache } from "react";

// Deduplicate calls within same request
export const getUser = cache(async (id: string) => {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
});

// Both components call getUser - only ONE fetch happens
async function UserHeader({ userId }: { userId: string }) {
  const user = await getUser(userId);
  return <h1>{user.name}</h1>;
}

async function UserProfile({ userId }: { userId: string }) {
  const user = await getUser(userId);
  return <p>{user.bio}</p>;
}
```

### unstable_cache (Data Cache)

```tsx
import { unstable_cache } from "next/cache";

const getCachedPosts = unstable_cache(
  async () => {
    return db.post.findMany();
  },
  ["posts"], // Cache key
  {
    revalidate: 3600, // Revalidate every hour
    tags: ["posts"], // Tag for on-demand revalidation
  },
);

// Usage
const posts = await getCachedPosts();
```

## Third-Party Scripts

### Analytics (Vercel)

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

### External Scripts

```tsx
import Script from 'next/script';

// Load after page interactive
<Script
  src="https://www.googletagmanager.com/gtag/js?id=GA_ID"
  strategy="afterInteractive"
/>

// Load when browser is idle
<Script
  src="https://widget.example.com/embed.js"
  strategy="lazyOnload"
/>

// Worker thread (experimental)
<Script
  src="https://heavy-analytics.js"
  strategy="worker"
/>
```

### Script Loading Strategies

| Strategy            | When             | Use Case                       |
| ------------------- | ---------------- | ------------------------------ |
| `beforeInteractive` | Before hydration | Critical polyfills             |
| `afterInteractive`  | After hydration  | Analytics, chat widgets        |
| `lazyOnload`        | Browser idle     | Non-critical embeds            |
| `worker`            | Web Worker       | Heavy analytics (experimental) |

## Monitoring & Debugging

### Web Vitals Reporting

```tsx
// app/layout.tsx
export function reportWebVitals(metric: {
  id: string;
  name: string;
  startTime: number;
  value: number;
  label: "web-vital" | "custom";
}) {
  // Send to analytics
  if (metric.label === "web-vital") {
    fetch("/api/analytics", {
      method: "POST",
      body: JSON.stringify({
        name: metric.name,
        value: metric.value,
        id: metric.id,
      }),
    });
  }
}
```

### Lighthouse CI

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI
on: push

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Lighthouse
        uses: treosh/lighthouse-ci-action@v10
        with:
          urls: |
            https://yourportfolio.com/
            https://yourportfolio.com/projects
          uploadArtifacts: true
          temporaryPublicStorage: true
```

## Performance Checklist

### Pre-Launch

- [ ] All images use `<Image />` component
- [ ] Fonts use `next/font`
- [ ] Dynamic imports for heavy components
- [ ] Bundle analyzed and optimized
- [ ] No layout shifts (CLS < 0.1)

### Images

- [ ] All images have explicit dimensions or use `fill`
- [ ] Above-the-fold images have `priority`
- [ ] Below-the-fold images use lazy loading
- [ ] `sizes` prop set for responsive images
- [ ] WebP/AVIF formats enabled

### JavaScript

- [ ] Tree shaking enabled
- [ ] Code splitting implemented
- [ ] Third-party scripts use correct strategy
- [ ] No render-blocking scripts

### Caching

- [ ] Static pages use ISR where appropriate
- [ ] API routes cached correctly
- [ ] Fonts cached with long max-age
- [ ] Static assets have immutable cache headers

### Monitoring

- [ ] Vercel Analytics enabled
- [ ] SpeedInsights enabled
- [ ] Core Web Vitals tracked
- [ ] Error boundaries in place

## Quick Wins

1. **Add `priority` to hero image** - Immediate LCP improvement
2. **Use `next/font`** - Eliminates FOUT/FOIT
3. **Enable optimizePackageImports** - Smaller bundles
4. **Add Suspense boundaries** - Better perceived performance
5. **Use ISR** - Faster TTFB than SSR
6. **Lazy load modals** - Reduce initial bundle
7. **Skeleton loaders** - Better UX than spinners
