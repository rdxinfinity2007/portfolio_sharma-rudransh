---
name: portfolio-components
description: Production-ready portfolio components including Hero, About, Projects, Experience, Contact sections with modern design patterns.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Portfolio Components Library

## Core Philosophy

Portfolio components should be **memorable, professional, and performant**. Each component tells a story about you as a developer. We build with composition, accessibility, and animation in mind.

## Project Structure

```
components/
├── ui/                     # Primitives (Button, Card, Badge)
│   ├── button.tsx
│   ├── card.tsx
│   ├── badge.tsx
│   ├── skeleton.tsx
│   └── icons.tsx
├── sections/               # Page sections
│   ├── hero.tsx
│   ├── about.tsx
│   ├── projects.tsx
│   ├── experience.tsx
│   ├── skills.tsx
│   ├── testimonials.tsx
│   └── contact.tsx
├── layout/                 # Structure
│   ├── header.tsx
│   ├── footer.tsx
│   ├── navigation.tsx
│   └── mobile-nav.tsx
└── shared/                 # Shared components
    ├── section-header.tsx
    ├── project-card.tsx
    ├── tech-stack.tsx
    └── social-links.tsx
```

## UI Primitives

### Button Component

```tsx
// components/ui/button.tsx
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";
import { forwardRef } from "react";

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 rounded-lg font-medium transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-offset-background disabled:pointer-events-none disabled:opacity-50 active:scale-95",
  {
    variants: {
      variant: {
        primary:
          "bg-brand-500 text-white hover:bg-brand-600 focus:ring-brand-400",
        secondary:
          "bg-zinc-800 text-white hover:bg-zinc-700 focus:ring-zinc-600",
        outline:
          "border border-zinc-700 bg-transparent hover:bg-zinc-800 focus:ring-zinc-600",
        ghost: "bg-transparent hover:bg-zinc-800/50 focus:ring-zinc-600",
        link: "text-brand-400 underline-offset-4 hover:underline focus:ring-brand-400",
      },
      size: {
        sm: "h-9 px-4 text-sm",
        md: "h-11 px-6 text-base",
        lg: "h-14 px-8 text-lg",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "primary",
      size: "md",
    },
  },
);

interface ButtonProps
  extends
    React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  isLoading?: boolean;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  (
    { className, variant, size, isLoading, children, disabled, ...props },
    ref,
  ) => {
    return (
      <button
        ref={ref}
        className={cn(buttonVariants({ variant, size }), className)}
        disabled={disabled || isLoading}
        {...props}
      >
        {isLoading && (
          <svg
            className="h-4 w-4 animate-spin"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
          >
            <circle
              className="opacity-25"
              cx="12"
              cy="12"
              r="10"
              stroke="currentColor"
              strokeWidth="4"
            />
            <path
              className="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"
            />
          </svg>
        )}
        {children}
      </button>
    );
  },
);

Button.displayName = "Button";
```

### Card Component

```tsx
// components/ui/card.tsx
import { cn } from "@/lib/utils";

interface CardProps extends React.HTMLAttributes<HTMLDivElement> {
  hover?: boolean;
}

export function Card({
  className,
  hover = false,
  children,
  ...props
}: CardProps) {
  return (
    <div
      className={cn(
        "rounded-2xl border border-zinc-800 bg-zinc-900/50 p-6",
        hover &&
          "transition-all duration-300 hover:border-zinc-700 hover:bg-zinc-900",
        className,
      )}
      {...props}
    >
      {children}
    </div>
  );
}

export function CardHeader({
  className,
  ...props
}: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn("mb-4", className)} {...props} />;
}

export function CardTitle({
  className,
  ...props
}: React.HTMLAttributes<HTMLHeadingElement>) {
  return <h3 className={cn("text-xl font-bold", className)} {...props} />;
}

export function CardDescription({
  className,
  ...props
}: React.HTMLAttributes<HTMLParagraphElement>) {
  return <p className={cn("text-sm text-muted", className)} {...props} />;
}

export function CardContent({
  className,
  ...props
}: React.HTMLAttributes<HTMLDivElement>) {
  return <div className={cn("", className)} {...props} />;
}

export function CardFooter({
  className,
  ...props
}: React.HTMLAttributes<HTMLDivElement>) {
  return (
    <div className={cn("mt-4 flex items-center gap-4", className)} {...props} />
  );
}
```

### Badge Component

```tsx
// components/ui/badge.tsx
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const badgeVariants = cva(
  "inline-flex items-center rounded-full px-3 py-1 text-xs font-medium transition-colors",
  {
    variants: {
      variant: {
        default: "bg-zinc-800 text-zinc-300",
        brand: "bg-brand-500/10 text-brand-400 border border-brand-500/20",
        success: "bg-green-500/10 text-green-400 border border-green-500/20",
        warning: "bg-yellow-500/10 text-yellow-400 border border-yellow-500/20",
        outline: "border border-zinc-700 text-zinc-400",
      },
    },
    defaultVariants: {
      variant: "default",
    },
  },
);

interface BadgeProps
  extends
    React.HTMLAttributes<HTMLSpanElement>,
    VariantProps<typeof badgeVariants> {}

export function Badge({ className, variant, ...props }: BadgeProps) {
  return (
    <span className={cn(badgeVariants({ variant }), className)} {...props} />
  );
}
```

## Section Components

### Hero Section

```tsx
// components/sections/hero.tsx
"use client";

import { motion } from "framer-motion";
import { ArrowRight, Download, Github, Linkedin, Twitter } from "lucide-react";
import { Button } from "@/components/ui/button";
import Image from "next/image";

const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: { staggerChildren: 0.1, delayChildren: 0.3 },
  },
};

const itemVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0 },
};

export function HeroSection() {
  return (
    <section className="relative min-h-screen overflow-hidden">
      {/* Background Effects */}
      <div className="absolute inset-0 -z-10">
        <div className="absolute inset-0 bg-gradient-to-b from-brand-500/10 via-transparent to-transparent" />
        <div className="absolute left-1/2 top-1/4 -z-10 h-[500px] w-[500px] -translate-x-1/2 rounded-full bg-brand-500/20 blur-[120px]" />
        <div className="absolute bottom-0 left-0 right-0 h-px bg-gradient-to-r from-transparent via-zinc-700 to-transparent" />
      </div>

      <div className="container flex min-h-screen items-center py-20">
        <motion.div
          variants={containerVariants}
          initial="hidden"
          animate="visible"
          className="grid gap-12 lg:grid-cols-2 lg:items-center"
        >
          {/* Text Content */}
          <div className="order-2 lg:order-1">
            {/* Status Badge */}
            <motion.div variants={itemVariants} className="mb-6">
              <span className="inline-flex items-center gap-2 rounded-full border border-green-500/20 bg-green-500/10 px-4 py-1.5 text-sm text-green-400">
                <span className="relative flex h-2 w-2">
                  <span className="absolute inline-flex h-full w-full animate-ping rounded-full bg-green-400 opacity-75" />
                  <span className="relative inline-flex h-2 w-2 rounded-full bg-green-500" />
                </span>
                Available for work
              </span>
            </motion.div>

            {/* Headline */}
            <motion.h1
              variants={itemVariants}
              className="font-heading text-4xl font-bold tracking-tight sm:text-5xl md:text-6xl"
            >
              Hi, I'm{" "}
              <span className="bg-gradient-to-r from-brand-400 via-purple-400 to-pink-400 bg-clip-text text-transparent">
                John Doe
              </span>
            </motion.h1>

            {/* Subtitle */}
            <motion.p
              variants={itemVariants}
              className="mt-4 text-xl text-muted sm:text-2xl"
            >
              Full-Stack Developer
            </motion.p>

            {/* Description */}
            <motion.p
              variants={itemVariants}
              className="mt-6 max-w-lg text-lg text-zinc-400"
            >
              I build exceptional digital experiences with modern technologies.
              Focused on React, Next.js, and creating interfaces that users
              love.
            </motion.p>

            {/* CTA Buttons */}
            <motion.div
              variants={itemVariants}
              className="mt-8 flex flex-col gap-4 sm:flex-row"
            >
              <Button size="lg" className="group">
                View My Work
                <ArrowRight className="h-4 w-4 transition-transform group-hover:translate-x-1" />
              </Button>
              <Button variant="outline" size="lg">
                <Download className="h-4 w-4" />
                Download CV
              </Button>
            </motion.div>

            {/* Social Links */}
            <motion.div
              variants={itemVariants}
              className="mt-8 flex items-center gap-4"
            >
              <span className="text-sm text-zinc-500">Find me on</span>
              <div className="flex gap-3">
                {[
                  {
                    icon: Github,
                    href: "https://github.com/johndoe",
                    label: "GitHub",
                  },
                  {
                    icon: Linkedin,
                    href: "https://linkedin.com/in/johndoe",
                    label: "LinkedIn",
                  },
                  {
                    icon: Twitter,
                    href: "https://twitter.com/johndoe",
                    label: "Twitter",
                  },
                ].map((social) => (
                  <a
                    key={social.label}
                    href={social.href}
                    target="_blank"
                    rel="noopener noreferrer"
                    className="rounded-lg p-2 text-zinc-400 transition-colors hover:bg-zinc-800 hover:text-white"
                    aria-label={social.label}
                  >
                    <social.icon className="h-5 w-5" />
                  </a>
                ))}
              </div>
            </motion.div>
          </div>

          {/* Image/Visual */}
          <motion.div
            variants={itemVariants}
            className="order-1 flex justify-center lg:order-2"
          >
            <div className="relative">
              {/* Decorative ring */}
              <div className="absolute -inset-4 rounded-full border border-brand-500/20" />
              <div className="absolute -inset-8 rounded-full border border-brand-500/10" />

              {/* Profile image */}
              <div className="relative h-64 w-64 overflow-hidden rounded-full border-2 border-brand-500/50 sm:h-80 sm:w-80">
                <Image
                  src="/images/profile.jpg"
                  alt="John Doe"
                  fill
                  className="object-cover"
                  priority
                />
              </div>

              {/* Floating badge */}
              <div className="absolute -right-4 top-8 rounded-lg border border-zinc-700 bg-zinc-900 px-4 py-2 shadow-xl">
                <p className="text-sm font-medium">5+ Years Exp.</p>
              </div>
            </div>
          </motion.div>
        </motion.div>
      </div>

      {/* Scroll indicator */}
      <motion.div
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        transition={{ delay: 1.5 }}
        className="absolute bottom-8 left-1/2 -translate-x-1/2"
      >
        <motion.div
          animate={{ y: [0, 8, 0] }}
          transition={{ duration: 1.5, repeat: Infinity }}
          className="flex flex-col items-center gap-2 text-zinc-500"
        >
          <span className="text-xs uppercase tracking-widest">Scroll</span>
          <div className="h-12 w-px bg-gradient-to-b from-zinc-500 to-transparent" />
        </motion.div>
      </motion.div>
    </section>
  );
}
```

### About Section

```tsx
// components/sections/about.tsx
"use client";

import { motion } from "framer-motion";
import Image from "next/image";
import { SectionHeader } from "@/components/shared/section-header";
import { TechStack } from "@/components/shared/tech-stack";

const stats = [
  { value: "5+", label: "Years Experience" },
  { value: "50+", label: "Projects Completed" },
  { value: "30+", label: "Happy Clients" },
  { value: "10+", label: "Awards Received" },
];

export function AboutSection() {
  return (
    <section id="about" className="py-24">
      <div className="container">
        <SectionHeader
          label="About Me"
          title="Passionate Developer & Creative Problem Solver"
          description="I'm a full-stack developer with a passion for creating beautiful, functional, and user-friendly applications."
        />

        <div className="mt-16 grid gap-12 lg:grid-cols-2">
          {/* Image */}
          <motion.div
            initial={{ opacity: 0, x: -50 }}
            whileInView={{ opacity: 1, x: 0 }}
            viewport={{ once: true }}
            transition={{ duration: 0.6 }}
            className="relative"
          >
            <div className="relative aspect-square overflow-hidden rounded-2xl">
              <Image
                src="/images/about.jpg"
                alt="Working on projects"
                fill
                className="object-cover"
              />
              {/* Overlay gradient */}
              <div className="absolute inset-0 bg-gradient-to-t from-zinc-900/80 via-transparent to-transparent" />
            </div>

            {/* Floating stats card */}
            <div className="absolute -bottom-6 -right-6 rounded-xl border border-zinc-700 bg-zinc-900 p-6 shadow-2xl">
              <div className="grid grid-cols-2 gap-4">
                {stats.slice(0, 2).map((stat) => (
                  <div key={stat.label} className="text-center">
                    <p className="font-heading text-3xl font-bold text-brand-400">
                      {stat.value}
                    </p>
                    <p className="text-xs text-zinc-400">{stat.label}</p>
                  </div>
                ))}
              </div>
            </div>
          </motion.div>

          {/* Content */}
          <motion.div
            initial={{ opacity: 0, x: 50 }}
            whileInView={{ opacity: 1, x: 0 }}
            viewport={{ once: true }}
            transition={{ duration: 0.6, delay: 0.2 }}
            className="flex flex-col justify-center"
          >
            <h3 className="font-heading text-2xl font-bold">
              Crafting Digital Experiences Since 2019
            </h3>

            <div className="mt-6 space-y-4 text-zinc-400">
              <p>
                I'm a software engineer based in San Francisco, specializing in
                building exceptional digital experiences. Currently, I'm focused
                on building accessible, human-centered products.
              </p>
              <p>
                My journey in web development started when I built my first
                website at 15. Since then, I've had the privilege of working at
                startups, agencies, and large corporations, honing my skills
                across the full stack.
              </p>
              <p>
                When I'm not coding, you'll find me hiking, reading sci-fi
                novels, or experimenting with new recipes in the kitchen.
              </p>
            </div>

            {/* Tech Stack */}
            <div className="mt-8">
              <h4 className="mb-4 text-sm font-medium uppercase tracking-wider text-zinc-500">
                Technologies I Work With
              </h4>
              <TechStack />
            </div>
          </motion.div>
        </div>

        {/* Stats Row */}
        <motion.div
          initial={{ opacity: 0, y: 40 }}
          whileInView={{ opacity: 1, y: 0 }}
          viewport={{ once: true }}
          transition={{ duration: 0.6 }}
          className="mt-24 grid grid-cols-2 gap-8 md:grid-cols-4"
        >
          {stats.map((stat, index) => (
            <div
              key={stat.label}
              className="relative rounded-2xl border border-zinc-800 bg-zinc-900/50 p-6 text-center"
            >
              <p className="font-heading text-4xl font-bold text-brand-400">
                {stat.value}
              </p>
              <p className="mt-2 text-sm text-zinc-400">{stat.label}</p>
            </div>
          ))}
        </motion.div>
      </div>
    </section>
  );
}
```

### Projects Section

```tsx
// components/sections/projects.tsx
"use client";

import { motion } from "framer-motion";
import { SectionHeader } from "@/components/shared/section-header";
import { ProjectCard } from "@/components/shared/project-card";
import { Button } from "@/components/ui/button";
import { ArrowRight } from "lucide-react";
import Link from "next/link";

const featuredProjects = [
  {
    id: 1,
    title: "E-Commerce Platform",
    description:
      "A full-featured e-commerce solution with real-time inventory, Stripe payments, and admin dashboard.",
    image: "/images/projects/ecommerce.jpg",
    tags: ["Next.js", "TypeScript", "Prisma", "Stripe"],
    demoUrl: "https://demo.example.com",
    githubUrl: "https://github.com/johndoe/ecommerce",
    featured: true,
  },
  {
    id: 2,
    title: "AI Content Generator",
    description:
      "GPT-powered content creation tool with templates, scheduling, and analytics.",
    image: "/images/projects/ai-content.jpg",
    tags: ["React", "OpenAI", "Node.js", "MongoDB"],
    demoUrl: "https://demo.example.com",
    githubUrl: "https://github.com/johndoe/ai-content",
    featured: true,
  },
  {
    id: 3,
    title: "Real-Time Dashboard",
    description:
      "Analytics dashboard with live data updates, custom charts, and export functionality.",
    image: "/images/projects/dashboard.jpg",
    tags: ["Next.js", "Socket.io", "D3.js", "PostgreSQL"],
    demoUrl: "https://demo.example.com",
    githubUrl: "https://github.com/johndoe/dashboard",
    featured: false,
  },
];

const containerVariants = {
  hidden: {},
  visible: {
    transition: { staggerChildren: 0.15 },
  },
};

const itemVariants = {
  hidden: { opacity: 0, y: 40 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.6, ease: "easeOut" },
  },
};

export function ProjectsSection() {
  return (
    <section id="projects" className="bg-zinc-950 py-24">
      <div className="container">
        <SectionHeader
          label="Featured Work"
          title="Projects I'm Proud Of"
          description="A selection of my recent work. Each project is unique and solves real-world problems."
        />

        <motion.div
          variants={containerVariants}
          initial="hidden"
          whileInView="visible"
          viewport={{ once: true, margin: "-100px" }}
          className="mt-16 grid gap-8 md:grid-cols-2 lg:grid-cols-3"
        >
          {featuredProjects.map((project) => (
            <motion.div key={project.id} variants={itemVariants}>
              <ProjectCard project={project} />
            </motion.div>
          ))}
        </motion.div>

        {/* View All Link */}
        <motion.div
          initial={{ opacity: 0 }}
          whileInView={{ opacity: 1 }}
          viewport={{ once: true }}
          transition={{ delay: 0.6 }}
          className="mt-12 text-center"
        >
          <Button variant="outline" size="lg" asChild>
            <Link href="/projects">
              View All Projects
              <ArrowRight className="h-4 w-4" />
            </Link>
          </Button>
        </motion.div>
      </div>
    </section>
  );
}
```

### Experience Timeline

```tsx
// components/sections/experience.tsx
"use client";

import { motion } from "framer-motion";
import { SectionHeader } from "@/components/shared/section-header";
import { Badge } from "@/components/ui/badge";

const experiences = [
  {
    title: "Senior Frontend Developer",
    company: "Tech Startup",
    period: "2023 - Present",
    description:
      "Lead frontend development for a B2B SaaS platform. Architected component library and design system used across multiple products.",
    skills: ["React", "TypeScript", "Next.js", "Tailwind"],
  },
  {
    title: "Full-Stack Developer",
    company: "Digital Agency",
    period: "2021 - 2023",
    description:
      "Developed custom web applications for enterprise clients. Led a team of 3 developers and mentored junior engineers.",
    skills: ["Node.js", "PostgreSQL", "AWS", "Docker"],
  },
  {
    title: "Frontend Developer",
    company: "E-commerce Company",
    period: "2019 - 2021",
    description:
      "Built and maintained e-commerce storefronts. Improved site performance by 40% and conversion rate by 15%.",
    skills: ["JavaScript", "React", "Redux", "GraphQL"],
  },
];

export function ExperienceSection() {
  return (
    <section id="experience" className="py-24">
      <div className="container">
        <SectionHeader
          label="Career"
          title="Work Experience"
          description="My professional journey and the companies I've had the pleasure to work with."
        />

        <div className="mt-16 space-y-8">
          {experiences.map((exp, index) => (
            <motion.div
              key={index}
              initial={{ opacity: 0, x: index % 2 === 0 ? -50 : 50 }}
              whileInView={{ opacity: 1, x: 0 }}
              viewport={{ once: true }}
              transition={{ duration: 0.5, delay: index * 0.1 }}
              className="group relative"
            >
              {/* Timeline line */}
              {index !== experiences.length - 1 && (
                <div className="absolute left-6 top-16 h-full w-px bg-zinc-800" />
              )}

              <div className="flex gap-6">
                {/* Timeline dot */}
                <div className="relative z-10 flex h-12 w-12 shrink-0 items-center justify-center rounded-full border border-zinc-700 bg-zinc-900 transition-colors group-hover:border-brand-500 group-hover:bg-brand-500/10">
                  <div className="h-3 w-3 rounded-full bg-brand-500" />
                </div>

                {/* Content */}
                <div className="flex-1 rounded-2xl border border-zinc-800 bg-zinc-900/50 p-6 transition-colors group-hover:border-zinc-700">
                  <div className="flex flex-wrap items-start justify-between gap-4">
                    <div>
                      <h3 className="font-heading text-xl font-bold">
                        {exp.title}
                      </h3>
                      <p className="text-brand-400">{exp.company}</p>
                    </div>
                    <Badge variant="outline">{exp.period}</Badge>
                  </div>

                  <p className="mt-4 text-zinc-400">{exp.description}</p>

                  <div className="mt-4 flex flex-wrap gap-2">
                    {exp.skills.map((skill) => (
                      <Badge key={skill} variant="default">
                        {skill}
                      </Badge>
                    ))}
                  </div>
                </div>
              </div>
            </motion.div>
          ))}
        </div>
      </div>
    </section>
  );
}
```

### Contact Section

```tsx
// components/sections/contact.tsx
"use client";

import { motion } from "framer-motion";
import { SectionHeader } from "@/components/shared/section-header";
import { Button } from "@/components/ui/button";
import { Card } from "@/components/ui/card";
import { Mail, MapPin, Send } from "lucide-react";
import { useState } from "react";

export function ContactSection() {
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setIsSubmitting(true);

    // Handle form submission
    await new Promise((r) => setTimeout(r, 1000));

    setIsSubmitting(false);
  };

  return (
    <section id="contact" className="bg-zinc-950 py-24">
      <div className="container">
        <SectionHeader
          label="Get In Touch"
          title="Let's Work Together"
          description="Have a project in mind? I'd love to hear about it. Send me a message and let's create something amazing."
        />

        <div className="mt-16 grid gap-12 lg:grid-cols-5">
          {/* Contact Info */}
          <motion.div
            initial={{ opacity: 0, x: -50 }}
            whileInView={{ opacity: 1, x: 0 }}
            viewport={{ once: true }}
            className="space-y-6 lg:col-span-2"
          >
            <Card>
              <div className="flex items-start gap-4">
                <div className="rounded-lg bg-brand-500/10 p-3">
                  <Mail className="h-6 w-6 text-brand-400" />
                </div>
                <div>
                  <h4 className="font-medium">Email</h4>
                  <a
                    href="mailto:hello@johndoe.com"
                    className="text-zinc-400 hover:text-white"
                  >
                    hello@johndoe.com
                  </a>
                </div>
              </div>
            </Card>

            <Card>
              <div className="flex items-start gap-4">
                <div className="rounded-lg bg-brand-500/10 p-3">
                  <MapPin className="h-6 w-6 text-brand-400" />
                </div>
                <div>
                  <h4 className="font-medium">Location</h4>
                  <p className="text-zinc-400">San Francisco, CA</p>
                </div>
              </div>
            </Card>

            <div className="rounded-2xl border border-zinc-800 bg-gradient-to-br from-brand-500/10 to-purple-500/10 p-6">
              <p className="text-lg">
                Prefer a quick chat?{" "}
                <a
                  href="https://cal.com/johndoe"
                  target="_blank"
                  rel="noopener noreferrer"
                  className="font-medium text-brand-400 hover:underline"
                >
                  Book a 15-min call →
                </a>
              </p>
            </div>
          </motion.div>

          {/* Contact Form */}
          <motion.div
            initial={{ opacity: 0, x: 50 }}
            whileInView={{ opacity: 1, x: 0 }}
            viewport={{ once: true }}
            className="lg:col-span-3"
          >
            <Card className="p-8">
              <form onSubmit={handleSubmit} className="space-y-6">
                <div className="grid gap-6 sm:grid-cols-2">
                  <div>
                    <label
                      htmlFor="name"
                      className="mb-2 block text-sm font-medium"
                    >
                      Name
                    </label>
                    <input
                      type="text"
                      id="name"
                      name="name"
                      required
                      className="w-full rounded-lg border border-zinc-700 bg-zinc-800/50 px-4 py-3 text-white placeholder:text-zinc-500 focus:border-brand-500 focus:outline-none focus:ring-1 focus:ring-brand-500"
                      placeholder="John Smith"
                    />
                  </div>
                  <div>
                    <label
                      htmlFor="email"
                      className="mb-2 block text-sm font-medium"
                    >
                      Email
                    </label>
                    <input
                      type="email"
                      id="email"
                      name="email"
                      required
                      className="w-full rounded-lg border border-zinc-700 bg-zinc-800/50 px-4 py-3 text-white placeholder:text-zinc-500 focus:border-brand-500 focus:outline-none focus:ring-1 focus:ring-brand-500"
                      placeholder="john@example.com"
                    />
                  </div>
                </div>

                <div>
                  <label
                    htmlFor="subject"
                    className="mb-2 block text-sm font-medium"
                  >
                    Subject
                  </label>
                  <input
                    type="text"
                    id="subject"
                    name="subject"
                    required
                    className="w-full rounded-lg border border-zinc-700 bg-zinc-800/50 px-4 py-3 text-white placeholder:text-zinc-500 focus:border-brand-500 focus:outline-none focus:ring-1 focus:ring-brand-500"
                    placeholder="Project Inquiry"
                  />
                </div>

                <div>
                  <label
                    htmlFor="message"
                    className="mb-2 block text-sm font-medium"
                  >
                    Message
                  </label>
                  <textarea
                    id="message"
                    name="message"
                    rows={5}
                    required
                    className="w-full resize-none rounded-lg border border-zinc-700 bg-zinc-800/50 px-4 py-3 text-white placeholder:text-zinc-500 focus:border-brand-500 focus:outline-none focus:ring-1 focus:ring-brand-500"
                    placeholder="Tell me about your project..."
                  />
                </div>

                <Button
                  type="submit"
                  size="lg"
                  className="w-full"
                  isLoading={isSubmitting}
                >
                  <Send className="h-4 w-4" />
                  Send Message
                </Button>
              </form>
            </Card>
          </motion.div>
        </div>
      </div>
    </section>
  );
}
```

## Shared Components

### Section Header

```tsx
// components/shared/section-header.tsx
"use client";

import { motion } from "framer-motion";

interface SectionHeaderProps {
  label: string;
  title: string;
  description?: string;
  align?: "left" | "center";
}

export function SectionHeader({
  label,
  title,
  description,
  align = "center",
}: SectionHeaderProps) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true }}
      transition={{ duration: 0.5 }}
      className={align === "center" ? "mx-auto max-w-2xl text-center" : ""}
    >
      <span className="text-sm font-medium uppercase tracking-widest text-brand-400">
        {label}
      </span>
      <h2 className="mt-2 font-heading text-3xl font-bold tracking-tight sm:text-4xl">
        {title}
      </h2>
      {description && (
        <p className="mt-4 text-lg text-zinc-400">{description}</p>
      )}
    </motion.div>
  );
}
```

### Tech Stack

```tsx
// components/shared/tech-stack.tsx
import {
  SiReact,
  SiNextdotjs,
  SiTypescript,
  SiTailwindcss,
  SiNodedotjs,
  SiPostgresql,
  SiPrisma,
  SiVercel,
} from "react-icons/si";

const technologies = [
  { name: "React", icon: SiReact, color: "#61DAFB" },
  { name: "Next.js", icon: SiNextdotjs, color: "#ffffff" },
  { name: "TypeScript", icon: SiTypescript, color: "#3178C6" },
  { name: "Tailwind", icon: SiTailwindcss, color: "#06B6D4" },
  { name: "Node.js", icon: SiNodedotjs, color: "#339933" },
  { name: "PostgreSQL", icon: SiPostgresql, color: "#4169E1" },
  { name: "Prisma", icon: SiPrisma, color: "#2D3748" },
  { name: "Vercel", icon: SiVercel, color: "#ffffff" },
];

export function TechStack() {
  return (
    <div className="flex flex-wrap gap-3">
      {technologies.map((tech) => (
        <div
          key={tech.name}
          className="flex items-center gap-2 rounded-lg border border-zinc-800 bg-zinc-900 px-3 py-2 text-sm transition-colors hover:border-zinc-700"
        >
          <tech.icon className="h-4 w-4" style={{ color: tech.color }} />
          <span>{tech.name}</span>
        </div>
      ))}
    </div>
  );
}
```

### Project Card

```tsx
// components/shared/project-card.tsx
"use client";

import { motion } from "framer-motion";
import Image from "next/image";
import Link from "next/link";
import { Badge } from "@/components/ui/badge";
import { ExternalLink, Github } from "lucide-react";

interface Project {
  id: number;
  title: string;
  description: string;
  image: string;
  tags: string[];
  demoUrl: string;
  githubUrl: string;
  featured?: boolean;
}

export function ProjectCard({ project }: { project: Project }) {
  return (
    <motion.article
      whileHover={{ y: -5 }}
      className="group relative overflow-hidden rounded-2xl border border-zinc-800 bg-zinc-900/50 transition-colors hover:border-zinc-700"
    >
      {/* Image */}
      <div className="relative aspect-video overflow-hidden">
        <Image
          src={project.image}
          alt={project.title}
          fill
          className="object-cover transition-transform duration-500 group-hover:scale-105"
        />
        <div className="absolute inset-0 bg-gradient-to-t from-zinc-900 via-zinc-900/20 to-transparent" />

        {/* Featured badge */}
        {project.featured && (
          <div className="absolute left-4 top-4">
            <Badge variant="brand">Featured</Badge>
          </div>
        )}
      </div>

      {/* Content */}
      <div className="p-6">
        {/* Tags */}
        <div className="mb-3 flex flex-wrap gap-2">
          {project.tags.slice(0, 3).map((tag) => (
            <Badge key={tag} variant="outline" className="text-xs">
              {tag}
            </Badge>
          ))}
        </div>

        {/* Title */}
        <h3 className="font-heading text-xl font-bold transition-colors group-hover:text-brand-400">
          {project.title}
        </h3>

        {/* Description */}
        <p className="mt-2 line-clamp-2 text-sm text-zinc-400">
          {project.description}
        </p>

        {/* Links */}
        <div className="mt-4 flex items-center gap-4 border-t border-zinc-800 pt-4">
          <a
            href={project.demoUrl}
            target="_blank"
            rel="noopener noreferrer"
            className="inline-flex items-center gap-1.5 text-sm font-medium text-zinc-400 transition-colors hover:text-white"
          >
            <ExternalLink className="h-4 w-4" />
            Live Demo
          </a>
          <a
            href={project.githubUrl}
            target="_blank"
            rel="noopener noreferrer"
            className="inline-flex items-center gap-1.5 text-sm font-medium text-zinc-400 transition-colors hover:text-white"
          >
            <Github className="h-4 w-4" />
            Source
          </a>
        </div>
      </div>

      {/* Hover glow effect */}
      <div className="absolute -right-20 -top-20 h-40 w-40 rounded-full bg-brand-500/20 opacity-0 blur-3xl transition-opacity duration-500 group-hover:opacity-100" />
    </motion.article>
  );
}
```

## Layout Components

### Header/Navigation

```tsx
// components/layout/header.tsx
"use client";

import { useState, useEffect } from "react";
import { motion, AnimatePresence } from "framer-motion";
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Menu, X } from "lucide-react";

const navItems = [
  { label: "About", href: "#about" },
  { label: "Projects", href: "#projects" },
  { label: "Experience", href: "#experience" },
  { label: "Contact", href: "#contact" },
];

export function Header() {
  const [isScrolled, setIsScrolled] = useState(false);
  const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false);

  useEffect(() => {
    const handleScroll = () => {
      setIsScrolled(window.scrollY > 50);
    };

    window.addEventListener("scroll", handleScroll);
    return () => window.removeEventListener("scroll", handleScroll);
  }, []);

  return (
    <header
      className={`fixed left-0 right-0 top-0 z-50 transition-all duration-300 ${
        isScrolled
          ? "border-b border-zinc-800 bg-zinc-950/80 backdrop-blur-lg"
          : "bg-transparent"
      }`}
    >
      <nav className="container flex h-16 items-center justify-between">
        {/* Logo */}
        <Link href="/" className="font-heading text-xl font-bold text-white">
          JD<span className="text-brand-400">.</span>
        </Link>

        {/* Desktop Nav */}
        <div className="hidden items-center gap-8 md:flex">
          {navItems.map((item) => (
            <a
              key={item.label}
              href={item.href}
              className="text-sm font-medium text-zinc-400 transition-colors hover:text-white"
            >
              {item.label}
            </a>
          ))}
          <Button size="sm">Hire Me</Button>
        </div>

        {/* Mobile Menu Button */}
        <button
          className="rounded-lg p-2 text-zinc-400 hover:bg-zinc-800 md:hidden"
          onClick={() => setIsMobileMenuOpen(!isMobileMenuOpen)}
          aria-label="Toggle menu"
        >
          {isMobileMenuOpen ? (
            <X className="h-6 w-6" />
          ) : (
            <Menu className="h-6 w-6" />
          )}
        </button>
      </nav>

      {/* Mobile Menu */}
      <AnimatePresence>
        {isMobileMenuOpen && (
          <motion.div
            initial={{ opacity: 0, height: 0 }}
            animate={{ opacity: 1, height: "auto" }}
            exit={{ opacity: 0, height: 0 }}
            className="border-b border-zinc-800 bg-zinc-950 md:hidden"
          >
            <div className="container space-y-4 py-6">
              {navItems.map((item) => (
                <a
                  key={item.label}
                  href={item.href}
                  className="block text-lg font-medium text-zinc-400 transition-colors hover:text-white"
                  onClick={() => setIsMobileMenuOpen(false)}
                >
                  {item.label}
                </a>
              ))}
              <Button className="w-full">Hire Me</Button>
            </div>
          </motion.div>
        )}
      </AnimatePresence>
    </header>
  );
}
```

### Footer

```tsx
// components/layout/footer.tsx
import Link from "next/link";
import { Github, Linkedin, Twitter, Mail } from "lucide-react";

const socialLinks = [
  { icon: Github, href: "https://github.com/johndoe", label: "GitHub" },
  {
    icon: Linkedin,
    href: "https://linkedin.com/in/johndoe",
    label: "LinkedIn",
  },
  { icon: Twitter, href: "https://twitter.com/johndoe", label: "Twitter" },
  { icon: Mail, href: "mailto:hello@johndoe.com", label: "Email" },
];

export function Footer() {
  return (
    <footer className="border-t border-zinc-800 bg-zinc-950">
      <div className="container py-12">
        <div className="flex flex-col items-center justify-between gap-6 md:flex-row">
          {/* Logo & Copyright */}
          <div className="text-center md:text-left">
            <Link href="/" className="font-heading text-xl font-bold">
              JD<span className="text-brand-400">.</span>
            </Link>
            <p className="mt-2 text-sm text-zinc-500">
              © {new Date().getFullYear()} John Doe. All rights reserved.
            </p>
          </div>

          {/* Social Links */}
          <div className="flex items-center gap-4">
            {socialLinks.map((social) => (
              <a
                key={social.label}
                href={social.href}
                target="_blank"
                rel="noopener noreferrer"
                className="rounded-lg p-2 text-zinc-400 transition-colors hover:bg-zinc-800 hover:text-white"
                aria-label={social.label}
              >
                <social.icon className="h-5 w-5" />
              </a>
            ))}
          </div>
        </div>

        {/* Bottom text */}
        <div className="mt-8 border-t border-zinc-800 pt-8 text-center">
          <p className="text-sm text-zinc-500">
            Built with <span className="text-brand-400">Next.js</span>,{" "}
            <span className="text-brand-400">Tailwind CSS</span>, and{" "}
            <span className="text-brand-400">Framer Motion</span>. Deployed on{" "}
            <span className="text-brand-400">Vercel</span>.
          </p>
        </div>
      </div>
    </footer>
  );
}
```
