---
name: content-mdx
description: Manage portfolio content with MDX, Contentlayer, or local JSON for projects, blog posts, and dynamic content.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Content Management with MDX

## Core Philosophy

Content should be **version-controlled, type-safe, and developer-friendly**. We use MDX for rich content (blog posts), JSON/TypeScript for structured data (projects), and optionally Contentlayer for the best DX.

## Content Architecture

### Project Structure

```
├── content/
│   ├── projects/
│   │   ├── project-1.mdx
│   │   ├── project-2.mdx
│   │   └── project-3.mdx
│   ├── blog/
│   │   ├── getting-started-with-nextjs.mdx
│   │   └── mastering-tailwind.mdx
│   └── data/
│       ├── experience.json
│       ├── skills.json
│       └── testimonials.json
├── lib/
│   ├── content.ts          # Content utilities
│   └── mdx.ts              # MDX processing
└── app/
    ├── projects/[slug]/page.tsx
    └── blog/[slug]/page.tsx
```

## Option 1: Contentlayer (Recommended)

### Installation

```bash
npm install contentlayer next-contentlayer date-fns
```

### Configuration

```ts
// contentlayer.config.ts
import { defineDocumentType, makeSource } from "contentlayer/source-files";
import remarkGfm from "remark-gfm";
import rehypePrettyCode from "rehype-pretty-code";
import rehypeSlug from "rehype-slug";
import rehypeAutolinkHeadings from "rehype-autolink-headings";

export const Project = defineDocumentType(() => ({
  name: "Project",
  filePathPattern: "projects/**/*.mdx",
  contentType: "mdx",
  fields: {
    title: { type: "string", required: true },
    description: { type: "string", required: true },
    date: { type: "date", required: true },
    image: { type: "string", required: true },
    tags: { type: "list", of: { type: "string" }, required: true },
    demoUrl: { type: "string" },
    githubUrl: { type: "string" },
    featured: { type: "boolean", default: false },
  },
  computedFields: {
    slug: {
      type: "string",
      resolve: (doc) => doc._raw.flattenedPath.replace("projects/", ""),
    },
    url: {
      type: "string",
      resolve: (doc) =>
        `/projects/${doc._raw.flattenedPath.replace("projects/", "")}`,
    },
  },
}));

export const Post = defineDocumentType(() => ({
  name: "Post",
  filePathPattern: "blog/**/*.mdx",
  contentType: "mdx",
  fields: {
    title: { type: "string", required: true },
    description: { type: "string", required: true },
    date: { type: "date", required: true },
    image: { type: "string" },
    tags: { type: "list", of: { type: "string" }, default: [] },
    published: { type: "boolean", default: true },
    author: { type: "string", default: "John Doe" },
  },
  computedFields: {
    slug: {
      type: "string",
      resolve: (doc) => doc._raw.flattenedPath.replace("blog/", ""),
    },
    url: {
      type: "string",
      resolve: (doc) => `/blog/${doc._raw.flattenedPath.replace("blog/", "")}`,
    },
    readingTime: {
      type: "string",
      resolve: (doc) => {
        const wordsPerMinute = 200;
        const words = doc.body.raw.split(/\s+/).length;
        const minutes = Math.ceil(words / wordsPerMinute);
        return `${minutes} min read`;
      },
    },
  },
}));

export default makeSource({
  contentDirPath: "content",
  documentTypes: [Project, Post],
  mdx: {
    remarkPlugins: [remarkGfm],
    rehypePlugins: [
      rehypeSlug,
      [
        rehypePrettyCode,
        {
          theme: "github-dark",
          onVisitLine(node: any) {
            if (node.children.length === 0) {
              node.children = [{ type: "text", value: " " }];
            }
          },
        },
      ],
      [
        rehypeAutolinkHeadings,
        {
          properties: {
            className: ["anchor"],
            ariaLabel: "Link to section",
          },
        },
      ],
    ],
  },
});
```

### Next.js Config

```js
// next.config.js
const { withContentlayer } = require("next-contentlayer");

/** @type {import('next').NextConfig} */
const nextConfig = {
  // your config
};

module.exports = withContentlayer(nextConfig);
```

### Content Files

````mdx
---
title: "E-Commerce Platform"
description: "A full-featured e-commerce solution with real-time inventory"
date: 2024-01-15
image: /images/projects/ecommerce.jpg
tags: ["Next.js", "TypeScript", "Prisma", "Stripe"]
demoUrl: https://demo.example.com
githubUrl: https://github.com/johndoe/ecommerce
featured: true
---

## Overview

This project demonstrates a modern e-commerce platform built with Next.js 14.

## Features

- **Real-time inventory tracking** - WebSocket integration
- **Stripe payments** - Secure checkout flow
- **Admin dashboard** - Manage products and orders

## Technical Details

```tsx
// Example code block with syntax highlighting
export async function getProducts() {
  return await db.product.findMany({
    where: { published: true },
    orderBy: { createdAt: "desc" },
  });
}
```
````

## Gallery

<ProjectGallery images={[
"/images/projects/ecommerce-1.jpg",
"/images/projects/ecommerce-2.jpg",
]} />

````

### Using Content

```tsx
// app/projects/page.tsx
import { allProjects } from 'contentlayer/generated';
import { compareDesc } from 'date-fns';

export default function ProjectsPage() {
  const projects = allProjects
    .filter((p) => p.featured)
    .sort((a, b) => compareDesc(new Date(a.date), new Date(b.date)));

  return (
    <div className="grid gap-8 md:grid-cols-2">
      {projects.map((project) => (
        <ProjectCard key={project.slug} project={project} />
      ))}
    </div>
  );
}
````

```tsx
// app/projects/[slug]/page.tsx
import { allProjects } from "contentlayer/generated";
import { notFound } from "next/navigation";
import { useMDXComponent } from "next-contentlayer/hooks";
import { mdxComponents } from "@/components/mdx";

interface Props {
  params: Promise<{ slug: string }>;
}

export async function generateStaticParams() {
  return allProjects.map((project) => ({
    slug: project.slug,
  }));
}

export async function generateMetadata({ params }: Props) {
  const { slug } = await params;
  const project = allProjects.find((p) => p.slug === slug);

  if (!project) return { title: "Not Found" };

  return {
    title: project.title,
    description: project.description,
  };
}

export default async function ProjectPage({ params }: Props) {
  const { slug } = await params;
  const project = allProjects.find((p) => p.slug === slug);

  if (!project) notFound();

  const MDXContent = useMDXComponent(project.body.code);

  return (
    <article className="prose prose-invert max-w-none">
      <h1>{project.title}</h1>
      <MDXContent components={mdxComponents} />
    </article>
  );
}
```

## Option 2: next-mdx-remote (Simpler)

### Installation

```bash
npm install next-mdx-remote gray-matter
```

### Content Utilities

```tsx
// lib/mdx.ts
import fs from "fs";
import path from "path";
import matter from "gray-matter";
import { compileMDX } from "next-mdx-remote/rsc";
import { mdxComponents } from "@/components/mdx";

const contentDir = path.join(process.cwd(), "content");

export interface ProjectFrontmatter {
  title: string;
  description: string;
  date: string;
  image: string;
  tags: string[];
  demoUrl?: string;
  githubUrl?: string;
  featured?: boolean;
}

export async function getProjectBySlug(slug: string) {
  const filePath = path.join(contentDir, "projects", `${slug}.mdx`);

  if (!fs.existsSync(filePath)) return null;

  const source = fs.readFileSync(filePath, "utf8");

  const { content, frontmatter } = await compileMDX<ProjectFrontmatter>({
    source,
    components: mdxComponents,
    options: {
      parseFrontmatter: true,
    },
  });

  return {
    content,
    frontmatter,
    slug,
  };
}

export async function getAllProjects() {
  const projectsDir = path.join(contentDir, "projects");
  const files = fs.readdirSync(projectsDir).filter((f) => f.endsWith(".mdx"));

  const projects = await Promise.all(
    files.map(async (file) => {
      const slug = file.replace(".mdx", "");
      const filePath = path.join(projectsDir, file);
      const source = fs.readFileSync(filePath, "utf8");
      const { data } = matter(source);

      return {
        slug,
        ...(data as ProjectFrontmatter),
      };
    }),
  );

  return projects.sort(
    (a, b) => new Date(b.date).getTime() - new Date(a.date).getTime(),
  );
}
```

### Using in Pages

```tsx
// app/projects/[slug]/page.tsx
import { getProjectBySlug, getAllProjects } from "@/lib/mdx";
import { notFound } from "next/navigation";

export async function generateStaticParams() {
  const projects = await getAllProjects();
  return projects.map((p) => ({ slug: p.slug }));
}

export default async function ProjectPage({ params }: Props) {
  const { slug } = await params;
  const project = await getProjectBySlug(slug);

  if (!project) notFound();

  return (
    <article className="prose prose-invert">
      <h1>{project.frontmatter.title}</h1>
      {project.content}
    </article>
  );
}
```

## Option 3: Local JSON/TypeScript

### Type-Safe Content

```tsx
// content/projects.ts
export interface Project {
  id: string;
  slug: string;
  title: string;
  description: string;
  longDescription: string;
  image: string;
  tags: string[];
  date: string;
  demoUrl?: string;
  githubUrl?: string;
  featured: boolean;
}

export const projects: Project[] = [
  {
    id: "1",
    slug: "ecommerce-platform",
    title: "E-Commerce Platform",
    description: "A full-featured e-commerce solution",
    longDescription: `
      Built with Next.js 14, this platform features real-time inventory,
      Stripe payments, and a comprehensive admin dashboard.
    `,
    image: "/images/projects/ecommerce.jpg",
    tags: ["Next.js", "TypeScript", "Prisma", "Stripe"],
    date: "2024-01-15",
    demoUrl: "https://demo.example.com",
    githubUrl: "https://github.com/johndoe/ecommerce",
    featured: true,
  },
  // More projects...
];

// Helper functions
export function getProjectBySlug(slug: string) {
  return projects.find((p) => p.slug === slug) ?? null;
}

export function getFeaturedProjects() {
  return projects
    .filter((p) => p.featured)
    .sort((a, b) => new Date(b.date).getTime() - new Date(a.date).getTime());
}

export function getAllProjects() {
  return [...projects].sort(
    (a, b) => new Date(b.date).getTime() - new Date(a.date).getTime(),
  );
}
```

### Experience Data

```tsx
// content/experience.ts
export interface Experience {
  id: string;
  title: string;
  company: string;
  location: string;
  period: string;
  description: string;
  achievements: string[];
  skills: string[];
}

export const experiences: Experience[] = [
  {
    id: "1",
    title: "Senior Frontend Developer",
    company: "Tech Startup",
    location: "San Francisco, CA",
    period: "Jan 2023 - Present",
    description: "Lead frontend development for a B2B SaaS platform.",
    achievements: [
      "Architected component library used across 5 products",
      "Reduced bundle size by 40% through code splitting",
      "Mentored team of 4 junior developers",
    ],
    skills: ["React", "TypeScript", "Next.js", "Tailwind"],
  },
  // More experiences...
];
```

## MDX Components

### Custom Components

```tsx
// components/mdx/index.tsx
import Image from "next/image";
import Link from "next/link";
import { cn } from "@/lib/utils";

// Custom image with optimization
function MDXImage({
  src,
  alt,
  width = 800,
  height = 400,
  className,
}: {
  src: string;
  alt: string;
  width?: number;
  height?: number;
  className?: string;
}) {
  return (
    <figure className="my-8">
      <Image
        src={src}
        alt={alt}
        width={width}
        height={height}
        className={cn("rounded-lg", className)}
      />
      {alt && (
        <figcaption className="mt-2 text-center text-sm text-zinc-500">
          {alt}
        </figcaption>
      )}
    </figure>
  );
}

// Custom link (internal vs external)
function MDXLink({
  href,
  children,
  ...props
}: React.AnchorHTMLAttributes<HTMLAnchorElement>) {
  const isInternal = href?.startsWith("/") || href?.startsWith("#");

  if (isInternal) {
    return (
      <Link href={href!} {...props}>
        {children}
      </Link>
    );
  }

  return (
    <a href={href} target="_blank" rel="noopener noreferrer" {...props}>
      {children}
    </a>
  );
}

// Callout/Alert component
function Callout({
  type = "info",
  title,
  children,
}: {
  type?: "info" | "warning" | "error" | "success";
  title?: string;
  children: React.ReactNode;
}) {
  const styles = {
    info: "border-blue-500/20 bg-blue-500/10 text-blue-200",
    warning: "border-yellow-500/20 bg-yellow-500/10 text-yellow-200",
    error: "border-red-500/20 bg-red-500/10 text-red-200",
    success: "border-green-500/20 bg-green-500/10 text-green-200",
  };

  return (
    <div className={cn("my-6 rounded-lg border p-4", styles[type])}>
      {title && <p className="mb-2 font-bold">{title}</p>}
      {children}
    </div>
  );
}

// Project gallery
function ProjectGallery({ images }: { images: string[] }) {
  return (
    <div className="my-8 grid gap-4 sm:grid-cols-2">
      {images.map((src, index) => (
        <Image
          key={src}
          src={src}
          alt={`Project screenshot ${index + 1}`}
          width={600}
          height={400}
          className="rounded-lg"
        />
      ))}
    </div>
  );
}

// Code block with copy button
function CodeBlock({
  children,
  className,
}: {
  children: string;
  className?: string;
}) {
  return (
    <div className="group relative">
      <pre
        className={cn("overflow-x-auto rounded-lg bg-zinc-900 p-4", className)}
      >
        <code>{children}</code>
      </pre>
      <button
        className="absolute right-2 top-2 rounded bg-zinc-700 px-2 py-1 text-xs opacity-0 transition-opacity group-hover:opacity-100"
        onClick={() => navigator.clipboard.writeText(children)}
      >
        Copy
      </button>
    </div>
  );
}

// Export all components
export const mdxComponents = {
  img: MDXImage,
  Image: MDXImage,
  a: MDXLink,
  Callout,
  ProjectGallery,
  pre: CodeBlock,
  // Override default elements
  h1: (props: any) => <h1 className="mt-8 text-4xl font-bold" {...props} />,
  h2: (props: any) => <h2 className="mt-8 text-2xl font-bold" {...props} />,
  h3: (props: any) => <h3 className="mt-6 text-xl font-bold" {...props} />,
  p: (props: any) => <p className="my-4 leading-relaxed" {...props} />,
  ul: (props: any) => <ul className="my-4 list-disc pl-6" {...props} />,
  ol: (props: any) => <ol className="my-4 list-decimal pl-6" {...props} />,
  blockquote: (props: any) => (
    <blockquote
      className="my-4 border-l-4 border-brand-500 pl-4 italic"
      {...props}
    />
  ),
};
```

### Using in MDX

```mdx
---
title: My Blog Post
---

# Introduction

This is a paragraph with a [link to projects](/projects).

<Callout type="info" title="Pro Tip">
  Use MDX components to enhance your content!
</Callout>

<Image src="/images/demo.jpg" alt="Demo screenshot" />

<ProjectGallery images={["/images/1.jpg", "/images/2.jpg"]} />
```

## Typography Styling

### Tailwind Typography Plugin

```bash
npm install @tailwindcss/typography
```

```js
// tailwind.config.js
module.exports = {
  plugins: [require("@tailwindcss/typography")],
};
```

### Custom Prose Styles

```css
/* globals.css */
.prose {
  --tw-prose-body: theme("colors.zinc.300");
  --tw-prose-headings: theme("colors.white");
  --tw-prose-links: theme("colors.brand.400");
  --tw-prose-bold: theme("colors.white");
  --tw-prose-code: theme("colors.brand.400");
  --tw-prose-pre-bg: theme("colors.zinc.900");
  --tw-prose-quotes: theme("colors.zinc.400");
  --tw-prose-quote-borders: theme("colors.brand.500");
}

/* Extend typography */
.prose pre {
  @apply border border-zinc-800;
}

.prose code:not(pre code) {
  @apply rounded bg-zinc-800 px-1.5 py-0.5 text-sm;
}

.prose a {
  @apply no-underline hover:underline;
}

.prose img {
  @apply rounded-lg;
}
```

### Usage

```tsx
<article className="prose prose-lg prose-invert max-w-none">
  <MDXContent components={mdxComponents} />
</article>
```

## Site Configuration

### Centralized Config

```tsx
// lib/config.ts
export const siteConfig = {
  name: "John Doe",
  title: "Full-Stack Developer",
  description: "Building beautiful web experiences",
  url: "https://johndoe.com",
  ogImage: "/og-image.png",

  links: {
    github: "https://github.com/johndoe",
    linkedin: "https://linkedin.com/in/johndoe",
    twitter: "https://twitter.com/johndoe",
    email: "hello@johndoe.com",
  },

  navigation: [
    { label: "About", href: "#about" },
    { label: "Projects", href: "#projects" },
    { label: "Experience", href: "#experience" },
    { label: "Contact", href: "#contact" },
  ],

  skills: {
    frontend: ["React", "Next.js", "TypeScript", "Tailwind CSS"],
    backend: ["Node.js", "PostgreSQL", "Prisma", "Redis"],
    tools: ["Git", "Docker", "AWS", "Vercel"],
  },
};

export type SiteConfig = typeof siteConfig;
```

### Usage

```tsx
import { siteConfig } from "@/lib/config";

export default function Footer() {
  return (
    <footer>
      <p>
        © {new Date().getFullYear()} {siteConfig.name}
      </p>
      <div className="flex gap-4">
        <a href={siteConfig.links.github}>GitHub</a>
        <a href={siteConfig.links.linkedin}>LinkedIn</a>
      </div>
    </footer>
  );
}
```

## Content Best Practices

1. **Use TypeScript types** for all content schemas
2. **Validate frontmatter** with Zod or Contentlayer
3. **Generate static paths** for all content pages
4. **Add computed fields** (reading time, slugs, URLs)
5. **Keep content and code separate** in `/content` directory
6. **Use ISR** for content that might update frequently
7. **Optimize images** in MDX with custom Image component
