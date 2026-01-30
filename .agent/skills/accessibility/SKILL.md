---
name: accessibility
description: Build accessible portfolio websites following WCAG guidelines with proper semantics, keyboard navigation, screen reader support, and inclusive design patterns.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Accessibility (A11y) Best Practices

## Core Philosophy

Accessibility isn't a feature—it's a **requirement**. Building accessible portfolios means everyone can experience your work, including people using screen readers, keyboards, or assistive technologies. Good accessibility also improves SEO and general usability.

## WCAG 2.1 Quick Reference

### Levels

- **Level A**: Minimum (must have)
- **Level AA**: Standard (should have) ← Target this
- **Level AAA**: Enhanced (nice to have)

### Key Principles (POUR)

1. **Perceivable**: Content can be perceived (alt text, captions)
2. **Operable**: Interface can be operated (keyboard, no seizures)
3. **Understandable**: Content is understandable (readable, predictable)
4. **Robust**: Content works with assistive tech

## Semantic HTML

### Document Structure

```tsx
// app/page.tsx
export default function HomePage() {
  return (
    <>
      {/* Only ONE h1 per page */}
      <main>
        <section aria-labelledby="hero-heading">
          <h1 id="hero-heading">John Doe - Full-Stack Developer</h1>
          <p>Building beautiful web experiences</p>
        </section>

        <section aria-labelledby="about-heading">
          <h2 id="about-heading">About Me</h2>
          <p>Content...</p>
        </section>

        <section aria-labelledby="projects-heading">
          <h2 id="projects-heading">Featured Projects</h2>
          {/* Project cards */}
        </section>
      </main>
    </>
  );
}
```

### Heading Hierarchy

```tsx
// ✅ Correct: Sequential heading levels
<h1>Portfolio</h1>
  <h2>Projects</h2>
    <h3>Project 1</h3>
    <h3>Project 2</h3>
  <h2>Experience</h2>
    <h3>Company A</h3>

// ❌ Wrong: Skipped levels
<h1>Portfolio</h1>
  <h4>Projects</h4>  // Skipped h2, h3
```

### Landmark Regions

```tsx
<header>            {/* Banner landmark */}
  <nav>             {/* Navigation landmark */}
    ...
  </nav>
</header>

<main>              {/* Main landmark (one per page) */}
  <section>         {/* Use with aria-labelledby */}
    ...
  </section>
  <article>         {/* Self-contained content */}
    ...
  </article>
  <aside>           {/* Complementary landmark */}
    ...
  </aside>
</main>

<footer>            {/* Contentinfo landmark */}
  ...
</footer>
```

## Keyboard Navigation

### Focus Management

```tsx
// components/ui/button.tsx
export function Button({ className, ...props }: ButtonProps) {
  return (
    <button
      className={cn(
        "rounded-lg px-4 py-2 font-medium transition-colors",
        // Visible focus state
        "focus:outline-none focus-visible:ring-2 focus-visible:ring-brand-500 focus-visible:ring-offset-2 focus-visible:ring-offset-background",
        className,
      )}
      {...props}
    />
  );
}
```

### Focus Visible vs Focus

```css
/* globals.css */
/* Only show focus ring for keyboard users */
:focus {
  outline: none;
}

:focus-visible {
  outline: 2px solid var(--brand-500);
  outline-offset: 2px;
}

/* Tailwind equivalent */
.focus-visible\:ring-2:focus-visible {
  --tw-ring-offset-shadow: ...;
  --tw-ring-shadow: ...;
}
```

### Skip Links

```tsx
// components/layout/skip-link.tsx
export function SkipLink() {
  return (
    <a
      href="#main-content"
      className="sr-only focus:not-sr-only focus:fixed focus:left-4 focus:top-4 focus:z-50 focus:rounded-lg focus:bg-brand-500 focus:px-4 focus:py-2 focus:text-white"
    >
      Skip to main content
    </a>
  );
}

// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <SkipLink />
        <Header />
        <main id="main-content" tabIndex={-1}>
          {children}
        </main>
        <Footer />
      </body>
    </html>
  );
}
```

### Keyboard Trapping (Modals)

```tsx
// components/modal.tsx
"use client";

import { useEffect, useRef } from "react";

export function Modal({ isOpen, onClose, children }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!isOpen) return;

    const modal = modalRef.current;
    if (!modal) return;

    // Get all focusable elements
    const focusableElements = modal.querySelectorAll<HTMLElement>(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])',
    );
    const firstElement = focusableElements[0];
    const lastElement = focusableElements[focusableElements.length - 1];

    // Focus first element
    firstElement?.focus();

    // Trap focus
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === "Escape") {
        onClose();
        return;
      }

      if (e.key !== "Tab") return;

      if (e.shiftKey) {
        if (document.activeElement === firstElement) {
          e.preventDefault();
          lastElement?.focus();
        }
      } else {
        if (document.activeElement === lastElement) {
          e.preventDefault();
          firstElement?.focus();
        }
      }
    };

    document.addEventListener("keydown", handleKeyDown);
    return () => document.removeEventListener("keydown", handleKeyDown);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      ref={modalRef}
    >
      {children}
    </div>
  );
}
```

## ARIA Attributes

### Common Patterns

```tsx
// Button that controls visibility
<button
  aria-expanded={isOpen}
  aria-controls="menu-content"
  onClick={() => setIsOpen(!isOpen)}
>
  Menu
</button>
<div id="menu-content" hidden={!isOpen}>
  {/* Menu items */}
</div>

// Loading state
<button aria-busy={isLoading} disabled={isLoading}>
  {isLoading ? 'Loading...' : 'Submit'}
</button>

// Current page in navigation
<nav aria-label="Main navigation">
  <a href="/" aria-current="page">Home</a>
  <a href="/projects">Projects</a>
</nav>

// Labeled section
<section aria-labelledby="projects-title">
  <h2 id="projects-title">Projects</h2>
</section>

// Described element
<input
  type="email"
  aria-describedby="email-hint email-error"
/>
<p id="email-hint">We'll never share your email</p>
<p id="email-error" role="alert">Invalid email format</p>
```

### Live Regions (Dynamic Content)

```tsx
// Announce changes to screen readers
<div aria-live="polite" aria-atomic="true">
  {/* Content updates announced politely */}
</div>

<div role="alert" aria-live="assertive">
  {/* Urgent updates interrupt immediately */}
  {errorMessage}
</div>

// Status messages
<div role="status" aria-live="polite">
  {successMessage && `Form submitted successfully`}
</div>
```

### Tab Panel

```tsx
"use client";

import { useState } from "react";

export function Tabs({ tabs }: { tabs: Tab[] }) {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <div>
      <div role="tablist" aria-label="Project categories">
        {tabs.map((tab, index) => (
          <button
            key={tab.id}
            role="tab"
            id={`tab-${tab.id}`}
            aria-selected={activeTab === index}
            aria-controls={`panel-${tab.id}`}
            tabIndex={activeTab === index ? 0 : -1}
            onClick={() => setActiveTab(index)}
            onKeyDown={(e) => {
              if (e.key === "ArrowRight") {
                setActiveTab((prev) => (prev + 1) % tabs.length);
              } else if (e.key === "ArrowLeft") {
                setActiveTab((prev) => (prev - 1 + tabs.length) % tabs.length);
              }
            }}
          >
            {tab.label}
          </button>
        ))}
      </div>

      {tabs.map((tab, index) => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          aria-labelledby={`tab-${tab.id}`}
          hidden={activeTab !== index}
          tabIndex={0}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

## Images & Media

### Image Alt Text

```tsx
// Decorative images (no alt needed)
<Image src={pattern} alt="" role="presentation" />

// Informative images
<Image
  src={projectScreenshot}
  alt="Dashboard showing analytics charts and user metrics"
/>

// Complex images (use figure)
<figure>
  <Image
    src={diagram}
    alt="System architecture diagram"
    aria-describedby="diagram-desc"
  />
  <figcaption id="diagram-desc">
    The diagram shows how the frontend connects to the API server,
    which in turn communicates with the database and external services.
  </figcaption>
</figure>

// Profile photo
<Image
  src={profile}
  alt="John Doe, smiling, wearing a blue shirt"
/>
```

### Video & Audio

```tsx
// Video with captions
<video controls>
  <source src="/demo.mp4" type="video/mp4" />
  <track
    kind="captions"
    src="/demo-captions.vtt"
    srcLang="en"
    label="English"
    default
  />
  Your browser doesn't support video.
</video>

// Audio with transcript
<figure>
  <audio controls aria-describedby="audio-transcript">
    <source src="/podcast.mp3" type="audio/mpeg" />
  </audio>
  <details id="audio-transcript">
    <summary>Transcript</summary>
    <p>Full transcript of the audio...</p>
  </details>
</figure>
```

## Forms

### Accessible Form Pattern

```tsx
// components/contact-form.tsx
export function ContactForm() {
  return (
    <form aria-labelledby="contact-form-title" noValidate>
      <h2 id="contact-form-title">Contact Me</h2>

      {/* Required field with hint */}
      <div>
        <label htmlFor="name">
          Name <span aria-hidden="true">*</span>
          <span className="sr-only">(required)</span>
        </label>
        <input
          type="text"
          id="name"
          name="name"
          required
          aria-required="true"
          autoComplete="name"
        />
      </div>

      {/* Email with description and error */}
      <div>
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          name="email"
          required
          aria-required="true"
          aria-describedby="email-hint email-error"
          aria-invalid={hasError ? "true" : undefined}
          autoComplete="email"
        />
        <p id="email-hint" className="text-sm text-zinc-500">
          I'll respond within 24 hours
        </p>
        {hasError && (
          <p id="email-error" role="alert" className="text-sm text-red-500">
            Please enter a valid email address
          </p>
        )}
      </div>

      {/* Textarea */}
      <div>
        <label htmlFor="message">Message</label>
        <textarea
          id="message"
          name="message"
          rows={5}
          required
          aria-required="true"
        />
      </div>

      <button type="submit">Send Message</button>
    </form>
  );
}
```

### Error Handling

```tsx
// components/form-error.tsx
export function FormError({ message }: { message: string }) {
  return (
    <div role="alert" className="mt-2 flex items-center gap-2 text-red-500">
      <AlertCircle className="h-4 w-4" aria-hidden="true" />
      <span>{message}</span>
    </div>
  );
}

// Usage with error summary
<form onSubmit={handleSubmit}>
  {errors.length > 0 && (
    <div role="alert" aria-labelledby="error-summary">
      <h3 id="error-summary">Please fix the following errors:</h3>
      <ul>
        {errors.map((error) => (
          <li key={error.field}>
            <a href={`#${error.field}`}>{error.message}</a>
          </li>
        ))}
      </ul>
    </div>
  )}

  {/* Form fields */}
</form>;
```

## Color & Contrast

### Contrast Requirements

| Text Size                        | Minimum Ratio (AA) | Enhanced (AAA) |
| -------------------------------- | ------------------ | -------------- |
| Normal text                      | 4.5:1              | 7:1            |
| Large text (18px+ or 14px+ bold) | 3:1                | 4.5:1          |
| UI components                    | 3:1                | -              |

### Don't Rely on Color Alone

```tsx
// ❌ Bad: Color is only indicator
<span className="text-red-500">Error occurred</span>

// ✅ Good: Icon + text + color
<span className="flex items-center gap-2 text-red-500">
  <AlertCircle className="h-4 w-4" aria-hidden="true" />
  <span>Error: Please enter a valid email</span>
</span>

// Form validation
<input
  className={cn(
    'border-2',
    hasError ? 'border-red-500' : 'border-zinc-700'
  )}
  aria-invalid={hasError}
/>
{hasError && (
  <span className="text-red-500" id="error-message">
    ⚠️ This field is required
  </span>
)}
```

### Focus Indicators

```css
/* globals.css */
/* High contrast focus ring */
:focus-visible {
  outline: 3px solid #8b5cf6;
  outline-offset: 2px;
}

/* Tailwind */
.focus-visible\:ring-2:focus-visible {
  box-shadow:
    0 0 0 2px var(--background),
    0 0 0 4px var(--brand-500);
}
```

## Motion & Animations

### Respect User Preferences

```tsx
// globals.css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### Framer Motion with Reduced Motion

```tsx
"use client";

import { motion, useReducedMotion } from "framer-motion";

export function AnimatedCard({ children }: { children: React.ReactNode }) {
  const shouldReduceMotion = useReducedMotion();

  return (
    <motion.div
      initial={shouldReduceMotion ? false : { opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={shouldReduceMotion ? { duration: 0 } : { duration: 0.5 }}
      whileHover={shouldReduceMotion ? {} : { scale: 1.02 }}
    >
      {children}
    </motion.div>
  );
}
```

## Screen Reader Utilities

### Visually Hidden Text

```tsx
// lib/utils.ts
// Use for screen reader only text
<span className="sr-only">Open main menu</span>

// Tailwind class
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

// Make visible on focus (skip links)
.sr-only:focus,
.sr-only:focus-visible {
  position: static;
  width: auto;
  height: auto;
  padding: 0.5rem 1rem;
  margin: 0;
  overflow: visible;
  clip: auto;
  white-space: normal;
}
```

### Icon Buttons

```tsx
// ❌ Bad: No accessible name
<button>
  <MenuIcon />
</button>

// ✅ Good: Screen reader text
<button aria-label="Open menu">
  <MenuIcon aria-hidden="true" />
</button>

// ✅ Also good: Visually hidden text
<button>
  <MenuIcon aria-hidden="true" />
  <span className="sr-only">Open menu</span>
</button>
```

## Component Examples

### Accessible Card

```tsx
export function ProjectCard({ project }: { project: Project }) {
  return (
    <article className="rounded-xl border border-zinc-800 p-6">
      <Image
        src={project.image}
        alt="" // Decorative, title provides context
        aria-hidden="true"
      />

      <h3 className="text-xl font-bold">
        <a
          href={`/projects/${project.slug}`}
          className="after:absolute after:inset-0" // Expand click area
        >
          {project.title}
        </a>
      </h3>

      <p className="text-zinc-400">{project.description}</p>

      <div className="flex gap-2" aria-label="Technologies used">
        {project.tags.map((tag) => (
          <span key={tag} className="badge">
            {tag}
          </span>
        ))}
      </div>

      <div className="flex gap-4">
        <a
          href={project.demoUrl}
          target="_blank"
          rel="noopener noreferrer"
          className="relative z-10" // Above card link
        >
          Live Demo
          <span className="sr-only">(opens in new tab)</span>
        </a>
        <a
          href={project.githubUrl}
          target="_blank"
          rel="noopener noreferrer"
          className="relative z-10"
        >
          Source Code
          <span className="sr-only">on GitHub (opens in new tab)</span>
        </a>
      </div>
    </article>
  );
}
```

### Accessible Navigation

```tsx
export function Navigation() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <nav aria-label="Main navigation">
      {/* Desktop nav */}
      <ul className="hidden md:flex gap-6">
        <li>
          <a href="/" aria-current="page">
            Home
          </a>
        </li>
        <li>
          <a href="/projects">Projects</a>
        </li>
        <li>
          <a href="/contact">Contact</a>
        </li>
      </ul>

      {/* Mobile menu button */}
      <button
        className="md:hidden"
        aria-expanded={isOpen}
        aria-controls="mobile-menu"
        aria-label={isOpen ? "Close menu" : "Open menu"}
        onClick={() => setIsOpen(!isOpen)}
      >
        {isOpen ? <X aria-hidden="true" /> : <Menu aria-hidden="true" />}
      </button>

      {/* Mobile menu */}
      <div id="mobile-menu" className={isOpen ? "block" : "hidden"} role="menu">
        <a href="/" role="menuitem">
          Home
        </a>
        <a href="/projects" role="menuitem">
          Projects
        </a>
        <a href="/contact" role="menuitem">
          Contact
        </a>
      </div>
    </nav>
  );
}
```

## Testing Accessibility

### Automated Testing

```bash
# Install axe-core
npm install -D @axe-core/react

# Or use eslint plugin
npm install -D eslint-plugin-jsx-a11y
```

```js
// .eslintrc.js
module.exports = {
  extends: ["next/core-web-vitals", "plugin:jsx-a11y/recommended"],
};
```

### Manual Testing Checklist

- [ ] Navigate entire site with keyboard only (Tab, Enter, Escape)
- [ ] Test with screen reader (NVDA, VoiceOver, JAWS)
- [ ] Check color contrast with browser dev tools
- [ ] Zoom to 200% - content still usable?
- [ ] Test with `prefers-reduced-motion: reduce`
- [ ] Disable CSS - content order makes sense?
- [ ] Check heading hierarchy with browser extension

### Tools

1. **axe DevTools** - Browser extension
2. **WAVE** - Web accessibility evaluator
3. **Lighthouse** - Accessibility audit
4. **Color Contrast Analyzer** - Contrast checker
5. **Screen readers** - NVDA (Windows), VoiceOver (Mac/iOS)

## Quick Wins

1. **Add `lang` attribute** - `<html lang="en">`
2. **Use semantic HTML** - `<main>`, `<nav>`, `<article>`
3. **Label all forms** - Every input needs a `<label>`
4. **Alt text on images** - Descriptive or empty for decorative
5. **Focus styles** - Visible keyboard focus indicators
6. **Skip link** - "Skip to main content"
7. **Color contrast** - Minimum 4.5:1 for text
8. **Button text** - Icon buttons need `aria-label`
