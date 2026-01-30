---
name: nextjs-core
description: Master Next.js 15 App Router architecture for building modern portfolio websites with Server Components, routing, and data patterns.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Next.js 15 App Router Mastery

## Core Philosophy

Next.js App Router represents a paradigm shift from "pages" to "layouts and components." We think in **React Server Components first**, adding client interactivity only where essential. This creates faster, more SEO-friendly portfolio sites.

## Project Structure for Portfolio

```
├── app/
│   ├── (marketing)/          # Route group - no URL impact
│   │   ├── page.tsx          # Home/Landing
│   │   ├── about/page.tsx
│   │   └── contact/page.tsx
│   ├── projects/
│   │   ├── page.tsx          # Projects list
│   │   └── [slug]/page.tsx   # Dynamic project detail
│   ├── blog/
│   │   ├── page.tsx
│   │   └── [slug]/page.tsx
│   ├── layout.tsx            # Root layout (nav, footer)
│   ├── globals.css
│   ├── loading.tsx           # Global loading UI
│   ├── error.tsx             # Global error boundary
│   └── not-found.tsx         # Custom 404
├── components/
│   ├── ui/                   # Reusable primitives
│   ├── sections/             # Page sections (Hero, About, etc.)
│   └── layout/               # Header, Footer, Navigation
├── lib/
│   ├── utils.ts              # Utility functions (cn, formatDate)
│   └── constants.ts          # Site config, social links
├── content/                  # MDX files or JSON data
└── public/
    ├── images/
    └── resume.pdf
```

## App Router Fundamentals

### 1. Layouts & Templates

Layouts persist across navigations; templates remount.

```tsx
// app/layout.tsx - Root Layout (REQUIRED)
import { Inter } from "next/font/google";
import { Header } from "@/components/layout/header";
import { Footer } from "@/components/layout/footer";
import "./globals.css";

const inter = Inter({ subsets: ["latin"], variable: "--font-inter" });

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" className={inter.variable}>
      <body className="min-h-screen bg-background font-sans antialiased">
        <Header />
        <main className="flex-1">{children}</main>
        <Footer />
      </body>
    </html>
  );
}
```

### 2. Route Groups

Organize routes without affecting URL structure.

```
app/
├── (marketing)/        # URL: /about, /contact
│   ├── about/page.tsx
│   └── contact/page.tsx
├── (portfolio)/        # URL: /projects, /blog
│   ├── projects/
│   └── blog/
└── layout.tsx
```

### 3. Dynamic Routes

```tsx
// app/projects/[slug]/page.tsx
import { getProjectBySlug, getAllProjects } from "@/lib/projects";
import { notFound } from "next/navigation";

interface Props {
  params: Promise<{ slug: string }>;
}

// Generate static paths at build time
export async function generateStaticParams() {
  const projects = await getAllProjects();
  return projects.map((project) => ({
    slug: project.slug,
  }));
}

// Generate metadata per page
export async function generateMetadata({ params }: Props) {
  const { slug } = await params;
  const project = await getProjectBySlug(slug);

  if (!project) return { title: "Project Not Found" };

  return {
    title: project.title,
    description: project.description,
    openGraph: {
      images: [project.image],
    },
  };
}

export default async function ProjectPage({ params }: Props) {
  const { slug } = await params;
  const project = await getProjectBySlug(slug);

  if (!project) notFound();

  return (
    <article className="container py-12">
      <h1 className="text-4xl font-bold">{project.title}</h1>
      {/* Project content */}
    </article>
  );
}
```

### 4. Parallel Routes (Advanced Layouts)

Show multiple pages simultaneously (e.g., modal over content).

```
app/
├── @modal/
│   └── (.)projects/[slug]/page.tsx  # Intercepted route
├── projects/
│   └── [slug]/page.tsx
└── layout.tsx
```

```tsx
// app/layout.tsx
export default function Layout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <>
      {children}
      {modal}
    </>
  );
}
```

## Data Fetching Patterns

### 1. Server Component Data Fetching

Fetch directly in async components - no `useEffect` needed.

```tsx
// app/projects/page.tsx
import { ProjectCard } from "@/components/project-card";

async function getProjects() {
  // Direct fetch with automatic deduplication
  const res = await fetch("https://api.example.com/projects", {
    next: { revalidate: 3600 }, // ISR: revalidate every hour
  });

  if (!res.ok) throw new Error("Failed to fetch projects");
  return res.json();
}

export default async function ProjectsPage() {
  const projects = await getProjects();

  return (
    <section className="container py-12">
      <h1 className="mb-8 text-3xl font-bold">My Projects</h1>
      <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
        {projects.map((project) => (
          <ProjectCard key={project.id} project={project} />
        ))}
      </div>
    </section>
  );
}
```

### 2. Static Data (Build Time)

For portfolio content that rarely changes.

```tsx
// lib/projects.ts
import { cache } from "react";
import projectsData from "@/content/projects.json";

// React cache() deduplicates calls in the same request
export const getAllProjects = cache(async () => {
  return projectsData.sort(
    (a, b) => new Date(b.date).getTime() - new Date(a.date).getTime(),
  );
});

export const getProjectBySlug = cache(async (slug: string) => {
  return projectsData.find((p) => p.slug === slug) ?? null;
});
```

### 3. Request Memoization

Next.js automatically deduplicates fetch requests.

```tsx
// Both components call the same URL - only ONE request is made
async function ProjectHeader() {
  const project = await fetch("/api/project/1").then((r) => r.json());
  return <h1>{project.title}</h1>;
}

async function ProjectDetails() {
  const project = await fetch("/api/project/1").then((r) => r.json());
  return <p>{project.description}</p>;
}
```

## Loading & Error States

### 1. Loading UI (Suspense Boundaries)

```tsx
// app/projects/loading.tsx
import { Skeleton } from "@/components/ui/skeleton";

export default function ProjectsLoading() {
  return (
    <section className="container py-12">
      <Skeleton className="mb-8 h-10 w-48" />
      <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
        {Array.from({ length: 6 }).map((_, i) => (
          <Skeleton key={i} className="h-64 rounded-lg" />
        ))}
      </div>
    </section>
  );
}
```

### 2. Error Boundaries

```tsx
// app/projects/error.tsx
"use client";

import { useEffect } from "react";
import { Button } from "@/components/ui/button";

export default function ProjectsError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    console.error(error);
  }, [error]);

  return (
    <div className="container flex min-h-[50vh] flex-col items-center justify-center gap-4">
      <h2 className="text-2xl font-bold">Something went wrong!</h2>
      <Button onClick={reset}>Try again</Button>
    </div>
  );
}
```

### 3. Not Found

```tsx
// app/not-found.tsx
import Link from "next/link";
import { Button } from "@/components/ui/button";

export default function NotFound() {
  return (
    <div className="container flex min-h-[70vh] flex-col items-center justify-center gap-4 text-center">
      <h1 className="text-6xl font-bold">404</h1>
      <p className="text-xl text-muted-foreground">
        This page doesn't exist or has been moved.
      </p>
      <Button asChild>
        <Link href="/">Go Home</Link>
      </Button>
    </div>
  );
}
```

## Image Optimization

### Next.js Image Component

```tsx
import Image from "next/image";

// Remote images (configure in next.config.js)
<Image
  src="https://images.unsplash.com/photo-xxx"
  alt="Project screenshot"
  width={1200}
  height={630}
  className="rounded-lg object-cover"
  priority // Load immediately (above-the-fold)
/>;

// Local images (automatic size detection)
import heroImage from "@/public/images/hero.jpg";

<Image
  src={heroImage}
  alt="Hero background"
  placeholder="blur" // Blur-up effect
  fill // Fill parent container
  className="object-cover"
  sizes="100vw"
/>;
```

### Configuration

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
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
    ],
  },
};

module.exports = nextConfig;
```

## Font Optimization

```tsx
// app/layout.tsx
import { Inter, Playfair_Display } from "next/font/google";
import localFont from "next/font/local";

// Google Fonts (automatic optimization)
const inter = Inter({
  subsets: ["latin"],
  variable: "--font-inter",
  display: "swap",
});

const playfair = Playfair_Display({
  subsets: ["latin"],
  variable: "--font-playfair",
  display: "swap",
});

// Local Font
const calSans = localFont({
  src: "../public/fonts/CalSans-SemiBold.woff2",
  variable: "--font-cal",
  display: "swap",
});

export default function RootLayout({ children }) {
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

```css
/* globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --font-sans: var(--font-inter);
    --font-heading: var(--font-cal);
    --font-serif: var(--font-playfair);
  }
}
```

## Link Prefetching

```tsx
import Link from 'next/link';

// Automatic prefetch on viewport enter (production)
<Link href="/projects">View Projects</Link>

// Disable prefetch for less important links
<Link href="/terms" prefetch={false}>Terms</Link>

// Programmatic navigation
'use client';
import { useRouter } from 'next/navigation';

function ContactButton() {
  const router = useRouter();

  return (
    <button onClick={() => router.push('/contact')}>
      Contact Me
    </button>
  );
}
```

## Environment Variables

```env
# .env.local (git ignored)
NEXT_PUBLIC_SITE_URL=https://yourportfolio.com
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX

# Server-only (no NEXT_PUBLIC_ prefix)
RESEND_API_KEY=re_xxxxx
NOTION_API_KEY=secret_xxxxx
```

```tsx
// Access in code
// Server Component / Server Action
const apiKey = process.env.RESEND_API_KEY;

// Client Component (only NEXT_PUBLIC_*)
const siteUrl = process.env.NEXT_PUBLIC_SITE_URL;
```

## TypeScript Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

## Common Gotchas

### 1. "use client" Boundary

Only add `'use client'` when you NEED hooks or event handlers.

```tsx
// ❌ Bad: Entire page is client
'use client';
export default function Page() { ... }

// ✅ Good: Only interactive part is client
// page.tsx (Server Component)
export default function Page() {
  return (
    <div>
      <StaticContent />
      <InteractiveWidget /> {/* This has 'use client' */}
    </div>
  );
}
```

### 2. Async Components

Server Components can be async; Client Components cannot.

```tsx
// ✅ Server Component
async function UserProfile() {
  const user = await getUser();
  return <div>{user.name}</div>;
}

// ❌ Client Component - won't work
'use client';
async function UserProfile() { ... } // Error!
```

### 3. Params in Next.js 15

Route params are now Promises in Next.js 15.

```tsx
// Next.js 15
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  // use id
}
```
