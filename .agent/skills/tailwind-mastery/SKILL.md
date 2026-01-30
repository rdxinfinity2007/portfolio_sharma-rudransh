---
name: tailwind-mastery
description: Master Tailwind CSS v4 for building stunning, responsive portfolio designs with advanced patterns, animations, and custom design systems.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Tailwind CSS Mastery for Portfolio Design

## Core Philosophy

Tailwind is not just utility classes—it's a **design system engine**. For portfolios, we create distinctive, memorable designs while maintaining consistency. Every spacing, color, and animation decision should feel intentional.

## Tailwind v4 Configuration

### CSS-First Configuration (v4)

```css
/* app/globals.css */
@import "tailwindcss";

/* Custom theme using CSS variables */
@theme {
  /* Colors - Portfolio Brand */
  --color-brand-50: oklch(0.97 0.02 280);
  --color-brand-100: oklch(0.93 0.04 280);
  --color-brand-500: oklch(0.55 0.25 280);
  --color-brand-600: oklch(0.45 0.25 280);
  --color-brand-900: oklch(0.25 0.15 280);

  /* Semantic Colors */
  --color-background: var(--color-zinc-950);
  --color-foreground: var(--color-zinc-50);
  --color-muted: var(--color-zinc-400);
  --color-accent: var(--color-brand-500);

  /* Typography Scale */
  --font-sans: "Inter Variable", system-ui, sans-serif;
  --font-heading: "Cal Sans", var(--font-sans);
  --font-mono: "JetBrains Mono", monospace;

  /* Custom Spacing */
  --spacing-section: 6rem;
  --spacing-container: 1.5rem;

  /* Animation Timing */
  --ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1);
  --ease-smooth: cubic-bezier(0.4, 0, 0.2, 1);

  /* Border Radius */
  --radius-soft: 0.75rem;
  --radius-card: 1rem;
}

/* Dark mode (default for portfolio) */
@variant dark (&:where(.dark, .dark *));

/* Light mode override */
@media (prefers-color-scheme: light) {
  :root:not(.dark) {
    --color-background: var(--color-zinc-50);
    --color-foreground: var(--color-zinc-950);
    --color-muted: var(--color-zinc-600);
  }
}
```

### Legacy Config (v3)

```js
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  darkMode: 'class',
  content: [
    './app/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './content/**/*.mdx',
  ],
  theme: {
    extend: {
      colors: {
        brand: {
          50: '#f5f3ff',
          500: '#8b5cf6',
          600: '#7c3aed',
          900: '#4c1d95',
        },
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
      },
      fontFamily: {
        sans: ['var(--font-inter)'],
        heading: ['var(--font-cal)'],
      },
      animation: {
        'fade-in': 'fade-in 0.5s ease-out forwards',
        'slide-up': 'slide-up 0.5s ease-out forwards',
        'scale-in': 'scale-in 0.3s ease-out forwards',
        'spin-slow': 'spin 3s linear infinite',
        'pulse-slow': 'pulse 4s ease-in-out infinite',
        'gradient': 'gradient 8s linear infinite',
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
        'scale-in': {
          from: { opacity: '0', transform: 'scale(0.95)' },
          to: { opacity: '1', transform: 'scale(1)' },
        },
        gradient: {
          '0%, 100%': { backgroundPosition: '0% 50%' },
          '50%': { backgroundPosition: '100% 50%' },
        },
      },
      backgroundImage: {
        'gradient-radial': 'radial-gradient(var(--tw-gradient-stops))',
        'hero-pattern': "url('/images/grid.svg')",
        'noise': "url('/images/noise.png')",
      },
    },
  },
  plugins: [
    require('@tailwindcss/typography'),
    require('@tailwindcss/container-queries'),
  ],
};

export default config;
```

## Portfolio Design Patterns

### 1. Hero Section

```tsx
export function HeroSection() {
  return (
    <section className="relative min-h-screen overflow-hidden">
      {/* Gradient Background */}
      <div className="absolute inset-0 -z-10">
        <div className="absolute inset-0 bg-gradient-to-b from-brand-500/20 via-background to-background" />
        <div className="absolute left-1/2 top-0 -z-10 h-[600px] w-[600px] -translate-x-1/2 rounded-full bg-brand-500/30 blur-[128px]" />
      </div>

      {/* Grid Pattern Overlay */}
      <div
        className="absolute inset-0 -z-10 bg-[url('/grid.svg')] bg-center opacity-20"
        style={{ maskImage: "linear-gradient(to bottom, white, transparent)" }}
      />

      {/* Content */}
      <div className="container flex min-h-screen flex-col items-center justify-center text-center">
        {/* Badge */}
        <div className="mb-6 inline-flex items-center gap-2 rounded-full border border-brand-500/20 bg-brand-500/10 px-4 py-1.5 text-sm text-brand-400">
          <span className="relative flex h-2 w-2">
            <span className="absolute inline-flex h-full w-full animate-ping rounded-full bg-brand-400 opacity-75" />
            <span className="relative inline-flex h-2 w-2 rounded-full bg-brand-500" />
          </span>
          Available for work
        </div>

        {/* Heading */}
        <h1 className="max-w-4xl font-heading text-5xl font-bold tracking-tight sm:text-6xl md:text-7xl lg:text-8xl">
          <span className="block">Building digital</span>
          <span className="mt-2 block bg-gradient-to-r from-brand-400 via-purple-400 to-pink-400 bg-clip-text text-transparent">
            experiences
          </span>
        </h1>

        {/* Description */}
        <p className="mt-6 max-w-2xl text-lg text-muted sm:text-xl">
          Full-stack developer crafting beautiful, performant web applications.
          Currently focused on React, Next.js, and the modern web.
        </p>

        {/* CTA Buttons */}
        <div className="mt-10 flex flex-col gap-4 sm:flex-row">
          <a
            href="#projects"
            className="group inline-flex items-center gap-2 rounded-full bg-brand-500 px-8 py-4 font-medium text-white transition-all hover:bg-brand-600 hover:shadow-lg hover:shadow-brand-500/25"
          >
            View Projects
            <ArrowRight className="h-4 w-4 transition-transform group-hover:translate-x-1" />
          </a>
          <a
            href="#contact"
            className="inline-flex items-center gap-2 rounded-full border border-zinc-700 px-8 py-4 font-medium transition-colors hover:border-zinc-500 hover:bg-zinc-800/50"
          >
            Get in Touch
          </a>
        </div>
      </div>

      {/* Scroll Indicator */}
      <div className="absolute bottom-8 left-1/2 -translate-x-1/2 animate-bounce">
        <ChevronDown className="h-6 w-6 text-muted" />
      </div>
    </section>
  );
}
```

### 2. Project Card with Hover Effects

```tsx
export function ProjectCard({ project }: { project: Project }) {
  return (
    <article className="group relative overflow-hidden rounded-2xl border border-zinc-800 bg-zinc-900/50 transition-all duration-300 hover:border-zinc-700 hover:bg-zinc-900">
      {/* Image */}
      <div className="relative aspect-video overflow-hidden">
        <Image
          src={project.image}
          alt={project.title}
          fill
          className="object-cover transition-transform duration-500 group-hover:scale-105"
        />
        {/* Overlay on hover */}
        <div className="absolute inset-0 bg-gradient-to-t from-zinc-900 via-transparent to-transparent opacity-0 transition-opacity duration-300 group-hover:opacity-100" />
      </div>

      {/* Content */}
      <div className="p-6">
        {/* Tags */}
        <div className="mb-3 flex flex-wrap gap-2">
          {project.tags.map((tag) => (
            <span
              key={tag}
              className="rounded-full bg-zinc-800 px-3 py-1 text-xs font-medium text-zinc-300"
            >
              {tag}
            </span>
          ))}
        </div>

        {/* Title */}
        <h3 className="font-heading text-xl font-bold transition-colors group-hover:text-brand-400">
          {project.title}
        </h3>

        {/* Description */}
        <p className="mt-2 line-clamp-2 text-sm text-muted">
          {project.description}
        </p>

        {/* Links */}
        <div className="mt-4 flex items-center gap-4 opacity-0 transition-all duration-300 group-hover:opacity-100">
          <a
            href={project.demoUrl}
            className="inline-flex items-center gap-1 text-sm font-medium text-brand-400 hover:underline"
          >
            Live Demo <ExternalLink className="h-3 w-3" />
          </a>
          <a
            href={project.githubUrl}
            className="inline-flex items-center gap-1 text-sm font-medium text-zinc-400 hover:text-white"
          >
            Source <Github className="h-3 w-3" />
          </a>
        </div>
      </div>

      {/* Corner Accent */}
      <div className="absolute -right-12 -top-12 h-24 w-24 rounded-full bg-brand-500/20 blur-2xl transition-all duration-300 group-hover:bg-brand-500/40" />
    </article>
  );
}
```

### 3. Bento Grid Layout

```tsx
export function BentoGrid({ children }: { children: React.ReactNode }) {
  return (
    <div className="grid auto-rows-[minmax(180px,auto)] grid-cols-1 gap-4 md:grid-cols-3">
      {children}
    </div>
  );
}

export function BentoCard({
  className,
  title,
  description,
  icon: Icon,
}: BentoCardProps) {
  return (
    <div
      className={cn(
        "group relative overflow-hidden rounded-2xl border border-zinc-800 bg-gradient-to-br from-zinc-900 to-zinc-900/50 p-6 transition-all hover:border-zinc-700",
        className,
      )}
    >
      {/* Icon */}
      <div className="mb-4 inline-flex rounded-lg bg-brand-500/10 p-3">
        <Icon className="h-6 w-6 text-brand-400" />
      </div>

      {/* Content */}
      <h3 className="font-heading text-lg font-bold">{title}</h3>
      <p className="mt-2 text-sm text-muted">{description}</p>

      {/* Hover Glow */}
      <div className="absolute inset-0 -z-10 bg-gradient-to-br from-brand-500/0 via-brand-500/0 to-brand-500/0 opacity-0 transition-opacity duration-500 group-hover:opacity-10" />
    </div>
  );
}

// Usage
<BentoGrid>
  <BentoCard
    className="md:col-span-2"
    title="Frontend"
    description="..."
    icon={Code}
  />
  <BentoCard title="Backend" description="..." icon={Server} />
  <BentoCard title="Design" description="..." icon={Palette} />
  <BentoCard
    className="md:col-span-2"
    title="DevOps"
    description="..."
    icon={Cloud}
  />
</BentoGrid>;
```

### 4. Animated Background

```tsx
export function AnimatedBackground() {
  return (
    <div className="fixed inset-0 -z-50">
      {/* Base gradient */}
      <div className="absolute inset-0 bg-zinc-950" />

      {/* Animated gradient orbs */}
      <div className="absolute left-1/4 top-1/4 h-96 w-96 animate-pulse rounded-full bg-purple-500/20 blur-[128px]" />
      <div
        className="absolute right-1/4 top-1/2 h-96 w-96 animate-pulse rounded-full bg-blue-500/20 blur-[128px]"
        style={{ animationDelay: "1s" }}
      />
      <div
        className="absolute bottom-1/4 left-1/2 h-96 w-96 animate-pulse rounded-full bg-pink-500/20 blur-[128px]"
        style={{ animationDelay: "2s" }}
      />

      {/* Noise overlay */}
      <div
        className="absolute inset-0 opacity-[0.03]"
        style={{
          backgroundImage: `url("data:image/svg+xml,%3Csvg viewBox='0 0 400 400' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noiseFilter'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noiseFilter)'/%3E%3C/svg%3E")`,
        }}
      />
    </div>
  );
}
```

## Advanced Patterns

### 1. Glassmorphism

```tsx
<div className="rounded-2xl border border-white/10 bg-white/5 p-6 backdrop-blur-xl">
  {/* Glass card content */}
</div>;

{
  /* Navigation with glass */
}
<nav className="fixed top-0 z-50 w-full border-b border-white/10 bg-zinc-950/80 backdrop-blur-lg">
  {/* Nav content */}
</nav>;
```

### 2. Text Gradient

```tsx
<h1 className="bg-gradient-to-r from-white via-zinc-300 to-zinc-500 bg-clip-text text-4xl font-bold text-transparent">
  Gradient Text
</h1>;

{
  /* Animated gradient text */
}
<span className="animate-gradient bg-gradient-to-r from-brand-400 via-purple-400 to-pink-400 bg-[length:200%_auto] bg-clip-text text-transparent">
  Animated Gradient
</span>;
```

### 3. Focus & Active States

```tsx
<button
  className="
  rounded-lg bg-brand-500 px-4 py-2 font-medium text-white 
  transition-all duration-200
  hover:bg-brand-600
  focus:outline-none focus:ring-2 focus:ring-brand-400 focus:ring-offset-2 focus:ring-offset-zinc-900
  active:scale-95
  disabled:cursor-not-allowed disabled:opacity-50
"
>
  Button
</button>
```

### 4. Container Queries

```tsx
// Parent needs @container class
<div className="@container">
  <div className="grid grid-cols-1 @md:grid-cols-2 @lg:grid-cols-3">
    {/* Responsive based on container, not viewport */}
  </div>
</div>
```

### 5. Group & Peer Modifiers

```tsx
{/* Parent hover affects children */}
<div className="group rounded-lg p-4 hover:bg-zinc-800">
  <h3 className="transition-colors group-hover:text-brand-400">Title</h3>
  <p className="text-muted transition-colors group-hover:text-white">Description</p>
</div>

{/* Sibling state affects styling */}
<input
  type="email"
  className="peer rounded border px-3 py-2 focus:border-brand-500 invalid:border-red-500"
/>
<p className="invisible text-sm text-red-500 peer-invalid:visible">
  Please enter a valid email
</p>
```

## Responsive Design

### Mobile-First Approach

```tsx
<div className="
  p-4 text-sm           /* Mobile (default) */
  sm:p-6 sm:text-base   /* 640px+ */
  md:p-8 md:text-lg     /* 768px+ */
  lg:p-10               /* 1024px+ */
  xl:p-12               /* 1280px+ */
  2xl:max-w-7xl         /* 1536px+ */
">
```

### Common Responsive Patterns

```tsx
{/* Stack to row */}
<div className="flex flex-col gap-4 md:flex-row md:items-center md:justify-between">

{/* Grid columns */}
<div className="grid grid-cols-1 gap-6 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">

{/* Hidden on mobile */}
<nav className="hidden md:flex">

{/* Show only on mobile */}
<button className="md:hidden">

{/* Different sizing */}
<h1 className="text-3xl sm:text-4xl md:text-5xl lg:text-6xl">
```

## Utility Functions

### The `cn()` Helper

```ts
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// Usage
<div className={cn(
  'rounded-lg p-4',
  isActive && 'bg-brand-500',
  className
)} />
```

### Conditional Classes

```tsx
<button
  className={cn(
    'rounded-lg px-4 py-2 font-medium transition-colors',
    {
      'bg-brand-500 text-white hover:bg-brand-600': variant === 'primary',
      'border border-zinc-700 hover:bg-zinc-800': variant === 'outline',
      'text-muted hover:text-foreground': variant === 'ghost',
      'opacity-50 cursor-not-allowed': disabled,
    }
  )}
>
```

## Dark Mode Implementation

```tsx
// components/theme-toggle.tsx
"use client";

import { useTheme } from "next-themes";
import { Moon, Sun } from "lucide-react";

export function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  return (
    <button
      onClick={() => setTheme(theme === "dark" ? "light" : "dark")}
      className="rounded-lg p-2 hover:bg-zinc-100 dark:hover:bg-zinc-800"
    >
      <Sun className="h-5 w-5 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <Moon className="absolute h-5 w-5 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
      <span className="sr-only">Toggle theme</span>
    </button>
  );
}
```

## Performance Tips

1. **Purge unused styles**: Tailwind v4 does this automatically
2. **Avoid dynamic class names**: `bg-${color}-500` won't work
3. **Use arbitrary values sparingly**: Too many indicates weak design system
4. **Prefer semantic tokens**: Use CSS variables for colors that change
5. **Group repeated patterns**: Extract to components, not `@apply`

```tsx
// ❌ Avoid: Can't be purged
const bgColor = `bg-${color}-500`;

// ✅ Good: Full class names
const bgColor = {
  blue: "bg-blue-500",
  red: "bg-red-500",
  green: "bg-green-500",
}[color];
```

## Troubleshooting

### Classes not applying?

1. Check `content` paths in config
2. Ensure class name is complete (not dynamically constructed)
3. Check specificity (use `!important` prefix: `!text-red-500`)
4. Verify Tailwind is imported in globals.css

### Animations janky?

1. Use `transform` and `opacity` only (GPU-accelerated)
2. Add `will-change-transform` for complex animations
3. Use `transition-transform` not `transition-all`
