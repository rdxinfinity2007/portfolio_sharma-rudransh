---
name: responsive-design
description: Master mobile-first responsive design with Tailwind CSS breakpoints, fluid typography, container queries, and modern layout techniques.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Responsive Design Mastery

## Core Philosophy

**Mobile-first is not optional—it's the standard.** Over 60% of web traffic is mobile. We design for the smallest screen first, then enhance for larger screens. This approach results in better performance, cleaner CSS, and improved user experience.

## Tailwind Breakpoints

### Default Breakpoints

| Breakpoint | Min Width | CSS                          | Common Devices              |
| ---------- | --------- | ---------------------------- | --------------------------- |
| `sm`       | 640px     | `@media (min-width: 640px)`  | Large phones, small tablets |
| `md`       | 768px     | `@media (min-width: 768px)`  | Tablets                     |
| `lg`       | 1024px    | `@media (min-width: 1024px)` | Laptops, small desktops     |
| `xl`       | 1280px    | `@media (min-width: 1280px)` | Desktops                    |
| `2xl`      | 1536px    | `@media (min-width: 1536px)` | Large desktops              |

### Mobile-First Approach

```tsx
// ✅ Correct: Mobile-first (base styles, then add breakpoints)
<div className="
  w-full           /* Mobile: full width */
  p-4              /* Mobile: small padding */
  text-sm          /* Mobile: small text */

  md:w-1/2         /* Tablet: half width */
  md:p-6           /* Tablet: medium padding */
  md:text-base     /* Tablet: normal text */

  lg:w-1/3         /* Desktop: third width */
  lg:p-8           /* Desktop: large padding */
  lg:text-lg       /* Desktop: large text */
">

// ❌ Wrong: Desktop-first (harder to maintain)
<div className="
  w-1/3 p-8 text-lg
  md:w-1/2 md:p-6 md:text-base
  sm:w-full sm:p-4 sm:text-sm
">
```

### Custom Breakpoints

```js
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      xs: "475px", // Extra small
      sm: "640px",
      md: "768px",
      lg: "1024px",
      xl: "1280px",
      "2xl": "1536px",
      "3xl": "1920px", // Ultra-wide

      // Max-width breakpoints
      "max-md": { max: "767px" },
      "max-lg": { max: "1023px" },

      // Range breakpoints
      tablet: { min: "768px", max: "1023px" },
    },
  },
};
```

## Layout Patterns

### Container Setup

```tsx
// Global container settings
// tailwind.config.js
module.exports = {
  theme: {
    container: {
      center: true,
      padding: {
        DEFAULT: "1rem",
        sm: "2rem",
        lg: "4rem",
        xl: "5rem",
        "2xl": "6rem",
      },
      screens: {
        sm: "640px",
        md: "768px",
        lg: "1024px",
        xl: "1280px",
        "2xl": "1400px", // Custom max-width
      },
    },
  },
};

// Usage
<div className="container">{/* Centered with responsive padding */}</div>;
```

### Responsive Grid

```tsx
// Auto-fit grid (cards fill available space)
<div className="grid grid-cols-1 gap-6 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
  {projects.map((project) => (
    <ProjectCard key={project.id} project={project} />
  ))}
</div>

// Two-column layout with sidebar
<div className="grid grid-cols-1 gap-8 lg:grid-cols-[280px_1fr]">
  <aside className="hidden lg:block">
    <Sidebar />
  </aside>
  <main>{children}</main>
</div>

// Three-column with centered content
<div className="grid grid-cols-1 gap-8 lg:grid-cols-[1fr_2fr_1fr]">
  <aside>Left</aside>
  <main>Content</main>
  <aside>Right</aside>
</div>
```

### Flexbox Patterns

```tsx
// Stack to row
<div className="flex flex-col gap-4 md:flex-row md:items-center md:justify-between">
  <div>Content</div>
  <div>Actions</div>
</div>

// Center content
<div className="flex min-h-screen items-center justify-center">
  <div className="max-w-md text-center">
    Centered content
  </div>
</div>

// Wrap items
<div className="flex flex-wrap gap-4">
  {tags.map((tag) => (
    <Badge key={tag}>{tag}</Badge>
  ))}
</div>
```

## Responsive Typography

### Fluid Text Sizing

```tsx
// Responsive heading scale
<h1 className="text-3xl font-bold sm:text-4xl md:text-5xl lg:text-6xl xl:text-7xl">
  Hero Headline
</h1>

// Body text
<p className="text-base md:text-lg lg:text-xl">
  Regular paragraph text
</p>

// Small text
<span className="text-xs sm:text-sm">
  Caption or label
</span>
```

### CSS Clamp for Fluid Typography

```css
/* globals.css */
:root {
  /* Fluid typography scale */
  --text-xs: clamp(0.75rem, 0.7rem + 0.25vw, 0.875rem);
  --text-sm: clamp(0.875rem, 0.8rem + 0.375vw, 1rem);
  --text-base: clamp(1rem, 0.9rem + 0.5vw, 1.125rem);
  --text-lg: clamp(1.125rem, 1rem + 0.625vw, 1.25rem);
  --text-xl: clamp(1.25rem, 1.1rem + 0.75vw, 1.5rem);
  --text-2xl: clamp(1.5rem, 1.25rem + 1.25vw, 2rem);
  --text-3xl: clamp(1.875rem, 1.5rem + 1.875vw, 2.5rem);
  --text-4xl: clamp(2.25rem, 1.75rem + 2.5vw, 3rem);
  --text-5xl: clamp(3rem, 2rem + 5vw, 4.5rem);
}
```

```tsx
// Use with arbitrary values or custom classes
<h1 className="text-[clamp(2rem,5vw,4rem)]">Fluid Heading</h1>
```

### Line Length Control

```tsx
// Optimal reading width (45-75 characters)
<article className="prose max-w-prose">
  {/* max-w-prose = 65ch by default */}
</article>

// Custom max-width for readability
<p className="max-w-[65ch]">
  Long paragraph text...
</p>
```

## Responsive Spacing

### Section Spacing

```tsx
// Responsive section padding
<section className="py-12 sm:py-16 md:py-20 lg:py-24 xl:py-32">
  <div className="container">{/* Section content */}</div>
</section>
```

### Component Spacing

```tsx
// Card with responsive padding
<div className="rounded-lg p-4 sm:p-6 md:p-8">
  <h3 className="text-lg md:text-xl">Title</h3>
  <p className="mt-2 md:mt-4">Description</p>
</div>

// Grid gaps
<div className="grid gap-4 sm:gap-6 md:gap-8 lg:gap-10">
  {/* Grid items */}
</div>
```

## Responsive Images

### Aspect Ratio Containers

```tsx
// Fixed aspect ratio (prevents CLS)
<div className="relative aspect-video overflow-hidden rounded-lg">
  <Image
    src="/project.jpg"
    alt="Project"
    fill
    className="object-cover"
    sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  />
</div>

// Square aspect ratio
<div className="relative aspect-square">
  <Image src="/avatar.jpg" alt="Avatar" fill className="object-cover" />
</div>

// Custom aspect ratio
<div className="relative aspect-[4/3]">
  <Image src="/photo.jpg" alt="Photo" fill className="object-cover" />
</div>
```

### Art Direction (Different Images per Breakpoint)

```tsx
// Using picture element for art direction
<picture>
  <source media="(min-width: 1024px)" srcSet="/hero-desktop.jpg" />
  <source media="(min-width: 768px)" srcSet="/hero-tablet.jpg" />
  <img
    src="/hero-mobile.jpg"
    alt="Hero"
    className="h-full w-full object-cover"
  />
</picture>;

// Or with Next.js Image and conditional rendering
const HeroImage = () => {
  return (
    <>
      <Image
        src="/hero-mobile.jpg"
        alt="Hero"
        fill
        className="object-cover md:hidden"
        priority
      />
      <Image
        src="/hero-desktop.jpg"
        alt="Hero"
        fill
        className="hidden object-cover md:block"
        priority
      />
    </>
  );
};
```

## Responsive Navigation

### Mobile Menu Pattern

```tsx
"use client";

import { useState } from "react";
import { motion, AnimatePresence } from "framer-motion";
import { Menu, X } from "lucide-react";

export function Navigation() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <nav className="fixed top-0 z-50 w-full border-b border-zinc-800 bg-zinc-950/80 backdrop-blur-lg">
      <div className="container flex h-16 items-center justify-between">
        {/* Logo */}
        <a href="/" className="font-heading text-xl font-bold">
          JD<span className="text-brand-400">.</span>
        </a>

        {/* Desktop Navigation */}
        <div className="hidden items-center gap-8 md:flex">
          <a href="#about" className="text-zinc-400 hover:text-white">
            About
          </a>
          <a href="#projects" className="text-zinc-400 hover:text-white">
            Projects
          </a>
          <a href="#contact" className="text-zinc-400 hover:text-white">
            Contact
          </a>
          <button className="rounded-lg bg-brand-500 px-4 py-2 text-white">
            Hire Me
          </button>
        </div>

        {/* Mobile Menu Button */}
        <button
          className="rounded-lg p-2 text-zinc-400 hover:bg-zinc-800 md:hidden"
          onClick={() => setIsOpen(!isOpen)}
          aria-label="Toggle menu"
          aria-expanded={isOpen}
        >
          {isOpen ? <X className="h-6 w-6" /> : <Menu className="h-6 w-6" />}
        </button>
      </div>

      {/* Mobile Menu */}
      <AnimatePresence>
        {isOpen && (
          <motion.div
            initial={{ opacity: 0, height: 0 }}
            animate={{ opacity: 1, height: "auto" }}
            exit={{ opacity: 0, height: 0 }}
            className="border-b border-zinc-800 bg-zinc-950 md:hidden"
          >
            <div className="container space-y-4 py-6">
              <a
                href="#about"
                className="block text-lg text-zinc-400 hover:text-white"
                onClick={() => setIsOpen(false)}
              >
                About
              </a>
              <a
                href="#projects"
                className="block text-lg text-zinc-400 hover:text-white"
                onClick={() => setIsOpen(false)}
              >
                Projects
              </a>
              <a
                href="#contact"
                className="block text-lg text-zinc-400 hover:text-white"
                onClick={() => setIsOpen(false)}
              >
                Contact
              </a>
              <button className="w-full rounded-lg bg-brand-500 py-3 text-white">
                Hire Me
              </button>
            </div>
          </motion.div>
        )}
      </AnimatePresence>
    </nav>
  );
}
```

## Container Queries

### Setup

```bash
npm install @tailwindcss/container-queries
```

```js
// tailwind.config.js
module.exports = {
  plugins: [require("@tailwindcss/container-queries")],
};
```

### Usage

```tsx
// Parent defines container
<div className="@container">
  {/* Children respond to container size */}
  <div className="flex flex-col gap-4 @md:flex-row @md:items-center">
    <div className="@lg:text-xl">
      Content that responds to container width
    </div>
  </div>
</div>

// Named containers
<div className="@container/sidebar">
  <div className="@lg/sidebar:text-xl">
    Responds to specific container
  </div>
</div>
```

### Card with Container Query

```tsx
export function ProjectCard({ project }: { project: Project }) {
  return (
    <article className="@container rounded-xl border border-zinc-800 bg-zinc-900/50">
      {/* Responsive based on card width, not viewport */}
      <div className="flex flex-col @sm:flex-row">
        {/* Image */}
        <div className="aspect-video @sm:aspect-square @sm:w-1/3">
          <Image
            src={project.image}
            alt={project.title}
            fill
            className="object-cover"
          />
        </div>

        {/* Content */}
        <div className="flex-1 p-4 @sm:p-6">
          <h3 className="text-lg font-bold @md:text-xl">{project.title}</h3>
          <p className="mt-2 text-sm text-zinc-400 @md:text-base">
            {project.description}
          </p>
        </div>
      </div>
    </article>
  );
}
```

## Hiding & Showing Content

### Conditional Display

```tsx
// Hide on mobile, show on tablet+
<div className="hidden md:block">
  Desktop sidebar
</div>

// Show on mobile only
<button className="md:hidden">
  Mobile menu
</button>

// Show on specific range
<div className="hidden sm:block lg:hidden">
  Tablet only
</div>
```

### Visibility vs Display

```tsx
// invisible: Hidden but takes space (no layout shift)
<div className="invisible md:visible">
  Content
</div>

// hidden: Removed from layout
<div className="hidden md:block">
  Content
</div>
```

## Hero Section Example

```tsx
export function HeroSection() {
  return (
    <section className="relative min-h-[calc(100vh-4rem)] overflow-hidden">
      {/* Background - different sizes for different screens */}
      <div className="absolute inset-0 -z-10">
        <div className="absolute left-1/2 top-1/2 h-[300px] w-[300px] -translate-x-1/2 -translate-y-1/2 rounded-full bg-brand-500/20 blur-[100px] md:h-[400px] md:w-[400px] lg:h-[600px] lg:w-[600px]" />
      </div>

      <div className="container flex min-h-[calc(100vh-4rem)] items-center">
        {/* Content grid: Stack on mobile, side-by-side on desktop */}
        <div className="grid gap-8 py-12 md:py-20 lg:grid-cols-2 lg:gap-12 lg:py-0">
          {/* Text content - order changes on mobile */}
          <div className="order-2 flex flex-col justify-center text-center lg:order-1 lg:text-left">
            {/* Responsive text sizes */}
            <h1 className="font-heading text-4xl font-bold tracking-tight sm:text-5xl md:text-6xl lg:text-7xl">
              Building{" "}
              <span className="bg-gradient-to-r from-brand-400 to-purple-400 bg-clip-text text-transparent">
                Digital
              </span>
              <br />
              Experiences
            </h1>

            <p className="mx-auto mt-4 max-w-lg text-lg text-zinc-400 sm:mt-6 sm:text-xl lg:mx-0">
              Full-stack developer crafting beautiful, performant web
              applications.
            </p>

            {/* Responsive button group */}
            <div className="mt-8 flex flex-col gap-4 sm:flex-row sm:justify-center lg:justify-start">
              <button className="rounded-lg bg-brand-500 px-6 py-3 font-medium text-white sm:px-8">
                View Projects
              </button>
              <button className="rounded-lg border border-zinc-700 px-6 py-3 font-medium sm:px-8">
                Contact Me
              </button>
            </div>
          </div>

          {/* Image - different sizes and positioning */}
          <div className="order-1 flex justify-center lg:order-2">
            <div className="relative h-48 w-48 sm:h-64 sm:w-64 lg:h-80 lg:w-80 xl:h-96 xl:w-96">
              <Image
                src="/profile.jpg"
                alt="John Doe"
                fill
                className="rounded-full object-cover"
                priority
                sizes="(max-width: 640px) 192px, (max-width: 1024px) 256px, 384px"
              />
            </div>
          </div>
        </div>
      </div>
    </section>
  );
}
```

## Testing Responsive Design

### Browser DevTools

1. Open DevTools (F12)
2. Toggle device toolbar (Ctrl+Shift+M)
3. Test common breakpoints
4. Test orientation changes

### Key Test Widths

- 320px (iPhone SE)
- 375px (iPhone 12)
- 414px (iPhone Pro Max)
- 768px (iPad)
- 1024px (iPad Pro, small laptop)
- 1280px (Laptop)
- 1440px (Desktop)
- 1920px (Full HD)

### Checklist

- [ ] Text readable at all sizes
- [ ] No horizontal scrolling
- [ ] Touch targets ≥ 44px on mobile
- [ ] Forms usable on mobile
- [ ] Navigation accessible
- [ ] Images scale properly
- [ ] No overlapping content
- [ ] Adequate spacing on small screens

## Common Patterns

### Responsive Table

```tsx
// Mobile: Stack, Desktop: Table
<div className="overflow-x-auto">
  {/* Mobile: Card list */}
  <div className="space-y-4 md:hidden">
    {data.map((row) => (
      <div key={row.id} className="rounded-lg bg-zinc-800 p-4">
        <p className="font-bold">{row.name}</p>
        <p className="text-zinc-400">{row.email}</p>
      </div>
    ))}
  </div>

  {/* Desktop: Table */}
  <table className="hidden w-full md:table">
    <thead>
      <tr>
        <th>Name</th>
        <th>Email</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      {data.map((row) => (
        <tr key={row.id}>
          <td>{row.name}</td>
          <td>{row.email}</td>
          <td>...</td>
        </tr>
      ))}
    </tbody>
  </table>
</div>
```

### Responsive Modal

```tsx
// Full screen on mobile, centered on desktop
<div className="fixed inset-0 z-50 md:flex md:items-center md:justify-center md:p-4">
  <div className="h-full w-full bg-zinc-900 md:h-auto md:max-h-[90vh] md:w-full md:max-w-lg md:rounded-xl">
    {/* Modal content */}
  </div>
</div>
```

### Responsive Footer

```tsx
export function Footer() {
  return (
    <footer className="border-t border-zinc-800 py-8 md:py-12">
      <div className="container">
        {/* Stack on mobile, row on desktop */}
        <div className="flex flex-col items-center gap-6 text-center md:flex-row md:justify-between md:text-left">
          <div>
            <p className="font-bold">John Doe</p>
            <p className="text-zinc-400">Full-Stack Developer</p>
          </div>

          {/* Social links - always horizontal */}
          <div className="flex gap-4">
            <a href="#" className="text-zinc-400 hover:text-white">
              GitHub
            </a>
            <a href="#" className="text-zinc-400 hover:text-white">
              LinkedIn
            </a>
          </div>
        </div>

        <p className="mt-8 text-center text-sm text-zinc-500">
          © {new Date().getFullYear()} All rights reserved.
        </p>
      </div>
    </footer>
  );
}
```
