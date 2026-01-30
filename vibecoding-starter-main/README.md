---
title: "VIBE Protocol v2.0 - AI Agent Skills for Next.js Portfolios"
description: "A comprehensive collection of AI agent skills for building production-ready Next.js portfolio applications."
author: "Jaivish Chauhan @ GDG SSIT"
version: "1.0.0"
url: "https://github.com/JaivishChauhan/vibecoding-starter"
---

# 🚀 Agent Skills Repository

A comprehensive collection of AI agent skills for building production-ready Next.js portfolio applications.

## 📋 Overview

This repository contains the **VIBE Protocol v2.0** — a complete framework for AI-assisted development with:

- **18 specialized skills** covering full-stack web development
- **Strict quality standards** (The Kill List)
- **Modern tech stack guidance** (Next.js 15, Tailwind v4, TypeScript)
- **Production patterns** for security, performance, and accessibility

## 🗂️ Structure

```
.
├── AGENTS.MD                    # The VIBE Protocol v2.0 (main instructions)
├── README.md                    # This file
├── .agent/
│   ├── skills.md                # Skill registry (all 18 skills)
│   └── skills/                  # Individual skill files
│       ├── nextjs-core/         # App Router, routing, data fetching
│       ├── tailwind-mastery/    # CSS patterns, design systems
│       ├── framer-motion/       # Animations, scroll effects
│       ├── typescript-patterns/ # Generics, type utilities
│       ├── contact-form/        # RHF, Zod, Server Actions
│       ├── seo-metadata/        # Metadata API, JSON-LD
│       ├── performance-optimization/  # Core Web Vitals
│       ├── accessibility/       # WCAG 2.1, ARIA
│       ├── testing-strategies/  # Vitest, Playwright
│       ├── vercel-deployment/   # Vercel MCP, domains
│       ├── analytics-monitoring/# Vercel Analytics, Sentry
│       ├── responsive-design/   # Mobile-first, breakpoints
│       ├── dark-mode-theming/   # next-themes, CSS vars
│       ├── content-mdx/         # MDX, Contentlayer
│       ├── portfolio-components/# Hero, Projects, Contact
│       ├── backend-api/         # Server Actions, API routes
│       ├── frontend-ui/         # React, Tailwind, Framer
│       └── deploy-ship/         # Vercel CLI, DevOps
└── .github/
    └── workflows/               # CI/CD pipelines
```

## 🎯 Available Skills (18)

| Category            | Skills                                                                                       |
| ------------------- | -------------------------------------------------------------------------------------------- |
| **Core Framework**  | `nextjs-core`, `typescript-patterns`, `backend-api`                                          |
| **Styling & UI**    | `tailwind-mastery`, `framer-motion`, `frontend-ui`, `responsive-design`, `dark-mode-theming` |
| **Components**      | `portfolio-components`, `contact-form`, `content-mdx`                                        |
| **Quality**         | `accessibility`, `performance-optimization`, `testing-strategies`                            |
| **SEO & Analytics** | `seo-metadata`, `analytics-monitoring`                                                       |
| **Deployment**      | `vercel-deployment`, `deploy-ship`                                                           |

## 🚫 The Kill List

Anti-patterns that are **FORBIDDEN** in this codebase:

| ❌ Forbidden                  | ✅ Use Instead         |
| ----------------------------- | ---------------------- |
| `useEffect` for data fetching | TanStack Query / SWR   |
| `useState` for complex forms  | React Hook Form + Zod  |
| `any` or `as any`             | Generics / Zod Schemas |
| `<img>` tag                   | Next.js `<Image />`    |
| Inline styles                 | Tailwind CSS classes   |
| `console.log` debugging       | Proper Error Handling  |
| Magic numbers                 | Named constants        |

## 🛠️ Tech Stack

- **Framework:** Next.js 15 (App Router)
- **Language:** TypeScript (strict mode)
- **Styling:** Tailwind CSS v4
- **Animations:** Framer Motion
- **Forms:** React Hook Form + Zod
- **State:** TanStack Query + Zustand
- **Testing:** Vitest + Playwright
- **Deployment:** Vercel

## 📖 Usage

### For AI Agents

1. Read `AGENTS.MD` for the complete VIBE Protocol
2. Check `.agent/skills.md` for available skills
3. Load relevant `SKILL.md` files before executing tasks
4. Follow the Kill List and output format requirements

### For Developers

1. Use the skills as reference documentation
2. Copy patterns directly into your projects
3. Follow the established conventions

## 📊 Quality Standards

- **Accessibility:** WCAG 2.1 AA compliance
- **Performance:** LCP < 2.5s, CLS < 0.1, INP < 100ms
- **Security:** Input validation, CSP headers, secure auth
- **Testing:** Unit, Component, Integration, E2E coverage

## 📄 License

MIT License - Feel free to use and adapt for your projects.

---

> **⚡ Built with the VIBE Protocol v2.0**
>
> _"We don't just write code; we ship products."_
