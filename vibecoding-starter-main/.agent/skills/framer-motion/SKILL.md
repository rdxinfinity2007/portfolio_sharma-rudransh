---
name: framer-motion
description: Create stunning, physics-based animations and micro-interactions for portfolio websites using Framer Motion.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Framer Motion Animation Excellence

## Core Philosophy

Animations aren't decoration—they're **communication**. Every animation should guide attention, provide feedback, or create delight. We use physics-based motion because it feels natural. Linear animations feel robotic.

## Installation & Setup

```bash
npm install framer-motion
```

```tsx
// components/motion.tsx - Re-export for cleaner imports
"use client";

export {
  motion,
  AnimatePresence,
  useAnimation,
  useInView,
  useScroll,
  useTransform,
  useSpring,
  useMotionValue,
  useMotionTemplate,
} from "framer-motion";
```

## Basic Animations

### 1. Simple Entrance Animation

```tsx
"use client";

import { motion } from "framer-motion";

export function FadeIn({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{
        duration: 0.5,
        ease: [0.25, 0.46, 0.45, 0.94], // easeOutQuad
      }}
    >
      {children}
    </motion.div>
  );
}
```

### 2. Spring Physics (Preferred)

```tsx
<motion.div
  initial={{ scale: 0.8, opacity: 0 }}
  animate={{ scale: 1, opacity: 1 }}
  transition={{
    type: "spring",
    stiffness: 300, // Higher = snappier
    damping: 20, // Higher = less bouncy
    mass: 1, // Higher = slower, heavier
  }}
>
  Spring Animation
</motion.div>
```

### 3. Gesture Animations

```tsx
<motion.button
  whileHover={{ scale: 1.05 }}
  whileTap={{ scale: 0.95 }}
  transition={{ type: "spring", stiffness: 400, damping: 17 }}
  className="rounded-lg bg-brand-500 px-6 py-3 font-medium text-white"
>
  Interactive Button
</motion.button>
```

## Variants System

### Orchestrated Animations

```tsx
"use client";

import { motion } from "framer-motion";

const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1, // Delay between children
      delayChildren: 0.2, // Initial delay before children start
      when: "beforeChildren", // Parent animates first
    },
  },
};

const itemVariants = {
  hidden: { opacity: 0, y: 30 },
  visible: {
    opacity: 1,
    y: 0,
    transition: {
      type: "spring",
      stiffness: 300,
      damping: 24,
    },
  },
};

export function StaggeredList({ items }: { items: string[] }) {
  return (
    <motion.ul
      variants={containerVariants}
      initial="hidden"
      animate="visible"
      className="space-y-4"
    >
      {items.map((item, index) => (
        <motion.li
          key={index}
          variants={itemVariants}
          className="rounded-lg bg-zinc-800 p-4"
        >
          {item}
        </motion.li>
      ))}
    </motion.ul>
  );
}
```

### Hover Variants (Parent-Child)

```tsx
const cardVariants = {
  rest: { scale: 1 },
  hover: { scale: 1.02 },
};

const arrowVariants = {
  rest: { x: 0, opacity: 0 },
  hover: { x: 5, opacity: 1 },
};

export function HoverCard() {
  return (
    <motion.div
      variants={cardVariants}
      initial="rest"
      whileHover="hover"
      className="rounded-lg bg-zinc-900 p-6"
    >
      <h3>Card Title</h3>
      <motion.span variants={arrowVariants} className="inline-block">
        →
      </motion.span>
    </motion.div>
  );
}
```

## Scroll Animations

### 1. Scroll-Triggered Entrance (useInView)

```tsx
"use client";

import { motion, useInView } from "framer-motion";
import { useRef } from "react";

export function ScrollReveal({ children }: { children: React.ReactNode }) {
  const ref = useRef(null);
  const isInView = useInView(ref, {
    once: true, // Only animate once
    margin: "-100px", // Trigger 100px before entering viewport
  });

  return (
    <motion.div
      ref={ref}
      initial={{ opacity: 0, y: 50 }}
      animate={isInView ? { opacity: 1, y: 0 } : { opacity: 0, y: 50 }}
      transition={{ duration: 0.6, ease: "easeOut" }}
    >
      {children}
    </motion.div>
  );
}
```

### 2. Scroll-Linked Animation (useScroll + useTransform)

```tsx
"use client";

import { motion, useScroll, useTransform, useSpring } from "framer-motion";
import { useRef } from "react";

export function ParallaxSection() {
  const ref = useRef(null);

  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ["start end", "end start"], // When element enters/exits viewport
  });

  // Smooth out the scroll value
  const smoothProgress = useSpring(scrollYProgress, {
    stiffness: 100,
    damping: 30,
  });

  // Map scroll progress to different values
  const y = useTransform(smoothProgress, [0, 1], [100, -100]);
  const opacity = useTransform(smoothProgress, [0, 0.3, 0.7, 1], [0, 1, 1, 0]);
  const scale = useTransform(smoothProgress, [0, 0.5, 1], [0.8, 1, 0.8]);

  return (
    <section ref={ref} className="relative h-screen overflow-hidden">
      <motion.div
        style={{ y, opacity, scale }}
        className="flex h-full items-center justify-center"
      >
        <h2 className="text-6xl font-bold">Parallax Content</h2>
      </motion.div>
    </section>
  );
}
```

### 3. Scroll Progress Indicator

```tsx
"use client";

import { motion, useScroll, useSpring } from "framer-motion";

export function ScrollProgressBar() {
  const { scrollYProgress } = useScroll();

  const scaleX = useSpring(scrollYProgress, {
    stiffness: 100,
    damping: 30,
    restDelta: 0.001,
  });

  return (
    <motion.div
      style={{ scaleX }}
      className="fixed left-0 right-0 top-0 z-50 h-1 origin-left bg-brand-500"
    />
  );
}
```

## AnimatePresence (Exit Animations)

### Page Transitions

```tsx
// app/template.tsx (runs on every navigation)
"use client";

import { motion } from "framer-motion";

export default function Template({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: -20 }}
      transition={{ duration: 0.3 }}
    >
      {children}
    </motion.div>
  );
}
```

### Conditional Content with Exit

```tsx
"use client";

import { motion, AnimatePresence } from "framer-motion";
import { useState } from "react";

export function ExpandableCard() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="rounded-lg bg-zinc-900 p-6">
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? "Close" : "Open"} Details
      </button>

      <AnimatePresence mode="wait">
        {isOpen && (
          <motion.div
            initial={{ opacity: 0, height: 0 }}
            animate={{ opacity: 1, height: "auto" }}
            exit={{ opacity: 0, height: 0 }}
            transition={{ duration: 0.3 }}
            className="overflow-hidden"
          >
            <div className="pt-4">
              <p>Expandable content goes here...</p>
            </div>
          </motion.div>
        )}
      </AnimatePresence>
    </div>
  );
}
```

### List Reordering

```tsx
"use client";

import { motion, AnimatePresence } from "framer-motion";

export function AnimatedList({ items }: { items: Item[] }) {
  return (
    <ul className="space-y-2">
      <AnimatePresence mode="popLayout">
        {items.map((item) => (
          <motion.li
            key={item.id}
            layout
            initial={{ opacity: 0, scale: 0.8 }}
            animate={{ opacity: 1, scale: 1 }}
            exit={{ opacity: 0, scale: 0.8 }}
            transition={{ type: "spring", stiffness: 300, damping: 25 }}
            className="rounded-lg bg-zinc-800 p-4"
          >
            {item.name}
          </motion.li>
        ))}
      </AnimatePresence>
    </ul>
  );
}
```

## Layout Animations

### Shared Layout

```tsx
"use client";

import { motion } from "framer-motion";
import { useState } from "react";

const tabs = ["Projects", "Experience", "Skills"];

export function AnimatedTabs() {
  const [activeTab, setActiveTab] = useState(tabs[0]);

  return (
    <div className="flex gap-2">
      {tabs.map((tab) => (
        <button
          key={tab}
          onClick={() => setActiveTab(tab)}
          className="relative rounded-lg px-4 py-2"
        >
          {activeTab === tab && (
            <motion.div
              layoutId="activeTab"
              className="absolute inset-0 rounded-lg bg-brand-500"
              transition={{ type: "spring", stiffness: 400, damping: 30 }}
            />
          )}
          <span className="relative z-10">{tab}</span>
        </button>
      ))}
    </div>
  );
}
```

### Expand Card to Modal

```tsx
"use client";

import { motion, AnimatePresence } from "framer-motion";
import { useState } from "react";

export function ExpandableProjectCard({ project }: { project: Project }) {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <>
      <motion.div
        layoutId={`card-${project.id}`}
        onClick={() => setIsExpanded(true)}
        className="cursor-pointer rounded-lg bg-zinc-900 p-6"
      >
        <motion.h3
          layoutId={`title-${project.id}`}
          className="text-xl font-bold"
        >
          {project.title}
        </motion.h3>
        <motion.p layoutId={`desc-${project.id}`} className="text-muted">
          {project.description}
        </motion.p>
      </motion.div>

      <AnimatePresence>
        {isExpanded && (
          <>
            {/* Backdrop */}
            <motion.div
              initial={{ opacity: 0 }}
              animate={{ opacity: 1 }}
              exit={{ opacity: 0 }}
              onClick={() => setIsExpanded(false)}
              className="fixed inset-0 z-40 bg-black/80"
            />

            {/* Expanded Card */}
            <motion.div
              layoutId={`card-${project.id}`}
              className="fixed inset-4 z-50 overflow-auto rounded-2xl bg-zinc-900 p-8 md:inset-20"
            >
              <motion.h3
                layoutId={`title-${project.id}`}
                className="text-3xl font-bold"
              >
                {project.title}
              </motion.h3>
              <motion.p
                layoutId={`desc-${project.id}`}
                className="mt-4 text-muted"
              >
                {project.description}
              </motion.p>

              {/* Additional content */}
              <motion.div
                initial={{ opacity: 0 }}
                animate={{ opacity: 1 }}
                transition={{ delay: 0.2 }}
                className="mt-8"
              >
                <p>{project.fullDescription}</p>
              </motion.div>

              <button
                onClick={() => setIsExpanded(false)}
                className="absolute right-4 top-4 rounded-full bg-zinc-800 p-2"
              >
                ✕
              </button>
            </motion.div>
          </>
        )}
      </AnimatePresence>
    </>
  );
}
```

## Advanced Patterns

### 1. Cursor Following Element

```tsx
"use client";

import { motion, useMotionValue, useSpring } from "framer-motion";
import { useEffect } from "react";

export function CustomCursor() {
  const cursorX = useMotionValue(-100);
  const cursorY = useMotionValue(-100);

  const springConfig = { damping: 25, stiffness: 200 };
  const cursorXSpring = useSpring(cursorX, springConfig);
  const cursorYSpring = useSpring(cursorY, springConfig);

  useEffect(() => {
    const moveCursor = (e: MouseEvent) => {
      cursorX.set(e.clientX - 16);
      cursorY.set(e.clientY - 16);
    };

    window.addEventListener("mousemove", moveCursor);
    return () => window.removeEventListener("mousemove", moveCursor);
  }, [cursorX, cursorY]);

  return (
    <motion.div
      style={{ x: cursorXSpring, y: cursorYSpring }}
      className="pointer-events-none fixed z-50 h-8 w-8 rounded-full bg-brand-500 mix-blend-difference"
    />
  );
}
```

### 2. Magnetic Button

```tsx
"use client";

import { motion, useMotionValue, useSpring, useTransform } from "framer-motion";
import { useRef } from "react";

export function MagneticButton({ children }: { children: React.ReactNode }) {
  const ref = useRef<HTMLButtonElement>(null);

  const x = useMotionValue(0);
  const y = useMotionValue(0);

  const springConfig = { stiffness: 150, damping: 15 };
  const xSpring = useSpring(x, springConfig);
  const ySpring = useSpring(y, springConfig);

  const handleMouseMove = (e: React.MouseEvent) => {
    if (!ref.current) return;
    const rect = ref.current.getBoundingClientRect();
    const centerX = rect.left + rect.width / 2;
    const centerY = rect.top + rect.height / 2;

    x.set((e.clientX - centerX) * 0.3);
    y.set((e.clientY - centerY) * 0.3);
  };

  const handleMouseLeave = () => {
    x.set(0);
    y.set(0);
  };

  return (
    <motion.button
      ref={ref}
      style={{ x: xSpring, y: ySpring }}
      onMouseMove={handleMouseMove}
      onMouseLeave={handleMouseLeave}
      whileTap={{ scale: 0.95 }}
      className="rounded-lg bg-brand-500 px-6 py-3 font-medium text-white"
    >
      {children}
    </motion.button>
  );
}
```

### 3. Typewriter Effect

```tsx
"use client";

import { motion, useMotionValue, useTransform, animate } from "framer-motion";
import { useEffect } from "react";

export function TypewriterText({ text }: { text: string }) {
  const count = useMotionValue(0);
  const displayText = useTransform(count, (latest) =>
    text.slice(0, Math.round(latest)),
  );

  useEffect(() => {
    const controls = animate(count, text.length, {
      type: "tween",
      duration: text.length * 0.05,
      ease: "linear",
    });
    return controls.stop;
  }, [count, text]);

  return (
    <motion.span>
      {displayText}
      <motion.span
        animate={{ opacity: [1, 0] }}
        transition={{ duration: 0.5, repeat: Infinity }}
      >
        |
      </motion.span>
    </motion.span>
  );
}
```

### 4. Reveal on Scroll with Clip Path

```tsx
"use client";

import { motion, useScroll, useTransform } from "framer-motion";
import { useRef } from "react";

export function RevealImage({ src, alt }: { src: string; alt: string }) {
  const ref = useRef(null);
  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ["start end", "center center"],
  });

  const clipPath = useTransform(
    scrollYProgress,
    [0, 1],
    ["inset(100% 0 0 0)", "inset(0% 0 0 0)"],
  );

  return (
    <motion.div ref={ref} style={{ clipPath }} className="overflow-hidden">
      <img src={src} alt={alt} className="h-full w-full object-cover" />
    </motion.div>
  );
}
```

### 5. Number Counter

```tsx
"use client";

import {
  motion,
  useMotionValue,
  useTransform,
  animate,
  useInView,
} from "framer-motion";
import { useEffect, useRef } from "react";

export function AnimatedCounter({
  value,
  suffix = "",
}: {
  value: number;
  suffix?: string;
}) {
  const ref = useRef(null);
  const isInView = useInView(ref, { once: true });
  const count = useMotionValue(0);
  const rounded = useTransform(count, (latest) => Math.round(latest));

  useEffect(() => {
    if (isInView) {
      animate(count, value, { duration: 2, ease: "easeOut" });
    }
  }, [count, isInView, value]);

  return (
    <span ref={ref}>
      <motion.span>{rounded}</motion.span>
      {suffix}
    </span>
  );
}

// Usage
<AnimatedCounter value={50} suffix="+" />; // Shows "50+"
```

## Portfolio-Specific Components

### Hero Text Animation

```tsx
"use client";

import { motion } from "framer-motion";

const letterVariants = {
  hidden: { opacity: 0, y: 50 },
  visible: (i: number) => ({
    opacity: 1,
    y: 0,
    transition: {
      delay: i * 0.03,
      type: "spring",
      stiffness: 200,
      damping: 20,
    },
  }),
};

export function AnimatedHeadline({ text }: { text: string }) {
  return (
    <motion.h1
      initial="hidden"
      animate="visible"
      className="text-6xl font-bold"
    >
      {text.split("").map((char, i) => (
        <motion.span
          key={i}
          custom={i}
          variants={letterVariants}
          className="inline-block"
        >
          {char === " " ? "\u00A0" : char}
        </motion.span>
      ))}
    </motion.h1>
  );
}
```

### Project Grid Reveal

```tsx
"use client";

import { motion } from "framer-motion";

const containerVariants = {
  hidden: {},
  visible: {
    transition: {
      staggerChildren: 0.1,
    },
  },
};

const cardVariants = {
  hidden: { opacity: 0, y: 60, rotateX: -15 },
  visible: {
    opacity: 1,
    y: 0,
    rotateX: 0,
    transition: {
      type: "spring",
      stiffness: 100,
      damping: 15,
    },
  },
};

export function ProjectGrid({ projects }: { projects: Project[] }) {
  return (
    <motion.div
      variants={containerVariants}
      initial="hidden"
      whileInView="visible"
      viewport={{ once: true, margin: "-100px" }}
      className="grid gap-6 md:grid-cols-2 lg:grid-cols-3"
      style={{ perspective: 1000 }}
    >
      {projects.map((project) => (
        <motion.div
          key={project.id}
          variants={cardVariants}
          className="rounded-xl bg-zinc-900 p-6"
        >
          {/* Project card content */}
        </motion.div>
      ))}
    </motion.div>
  );
}
```

## Performance Best Practices

### 1. Use `layout` Sparingly

Layout animations are expensive. Only use when needed.

### 2. Optimize with `willChange`

```tsx
<motion.div style={{ willChange: 'transform, opacity' }}>
```

### 3. Use `useReducedMotion`

```tsx
import { useReducedMotion } from "framer-motion";

function MyComponent() {
  const shouldReduceMotion = useReducedMotion();

  return <motion.div animate={{ x: shouldReduceMotion ? 0 : 100 }} />;
}
```

### 4. Lazy Load Heavy Animations

```tsx
import dynamic from "next/dynamic";

const HeavyAnimation = dynamic(() => import("@/components/heavy-animation"), {
  ssr: false,
});
```

## Common Gotchas

### "AnimatePresence not working"

- Ensure children have unique `key` props
- Use `mode="wait"` or `mode="popLayout"` as needed
- Component must be direct child of AnimatePresence

### "Layout animation jumps"

- Add `layout` to all siblings that might shift
- Use `layoutId` for shared element transitions
- Wrap with `LayoutGroup` if in different components

### "Animation not starting"

- Check if `initial` matches current state
- Ensure component is mounted (not conditionally rendered without AnimatePresence)
