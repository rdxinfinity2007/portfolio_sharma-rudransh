---
name: dark-mode-theming
description: Implement robust dark mode and theming systems with next-themes, CSS custom properties, and Tailwind CSS for seamless user experience.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Dark Mode & Theming System

## Overview

A robust theming system allows users to choose between light, dark, and system-preference modes. We use `next-themes` for theme state management and CSS custom properties for dynamic styling.

## Setup with next-themes

### Installation

```bash
npm install next-themes
```

### Theme Provider Setup

```tsx
// components/providers/theme-provider.tsx
"use client";

import { ThemeProvider as NextThemesProvider } from "next-themes";
import type { ThemeProviderProps } from "next-themes";

/**
 * Theme provider wrapper for next-themes.
 * Manages light/dark/system theme preferences.
 */
export function ThemeProvider({ children, ...props }: ThemeProviderProps) {
  return (
    <NextThemesProvider
      attribute="class" // Use class-based dark mode
      defaultTheme="system" // Respect system preference
      enableSystem // Allow system theme detection
      disableTransitionOnChange // Prevent flash during theme change
      {...props}
    >
      {children}
    </NextThemesProvider>
  );
}
```

### Root Layout Integration

```tsx
// app/layout.tsx
import { ThemeProvider } from "@/components/providers/theme-provider";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className="bg-white text-zinc-900 dark:bg-zinc-950 dark:text-zinc-50">
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

> **Note:** `suppressHydrationWarning` is required on `<html>` to prevent hydration mismatch from the injected script.

## Theme Toggle Component

### Basic Toggle

```tsx
// components/theme-toggle.tsx
"use client";

import { useTheme } from "next-themes";
import { useEffect, useState } from "react";
import { Sun, Moon, Monitor } from "lucide-react";

/**
 * Theme toggle button with three states: light, dark, system.
 * Handles hydration mismatch gracefully.
 */
export function ThemeToggle() {
  const [mounted, setMounted] = useState(false);
  const { theme, setTheme } = useTheme();

  // Prevent hydration mismatch
  useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) {
    // Render placeholder with same dimensions to prevent layout shift
    return (
      <button
        className="h-10 w-10 rounded-lg bg-zinc-100 dark:bg-zinc-800"
        aria-label="Toggle theme"
        disabled
      />
    );
  }

  return (
    <button
      onClick={() => {
        if (theme === "dark") setTheme("light");
        else if (theme === "light") setTheme("system");
        else setTheme("dark");
      }}
      className="flex h-10 w-10 items-center justify-center rounded-lg bg-zinc-100 text-zinc-600 transition-colors hover:bg-zinc-200 dark:bg-zinc-800 dark:text-zinc-400 dark:hover:bg-zinc-700"
      aria-label={`Current theme: ${theme}. Click to change.`}
    >
      {theme === "dark" ? (
        <Moon className="h-5 w-5" />
      ) : theme === "light" ? (
        <Sun className="h-5 w-5" />
      ) : (
        <Monitor className="h-5 w-5" />
      )}
    </button>
  );
}
```

### Animated Toggle with Framer Motion

```tsx
// components/theme-toggle-animated.tsx
"use client";

import { useTheme } from "next-themes";
import { useEffect, useState } from "react";
import { motion, AnimatePresence } from "framer-motion";
import { Sun, Moon } from "lucide-react";

export function ThemeToggleAnimated() {
  const [mounted, setMounted] = useState(false);
  const { resolvedTheme, setTheme } = useTheme();

  useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) {
    return (
      <div className="h-10 w-10 rounded-full bg-zinc-200 dark:bg-zinc-800" />
    );
  }

  const isDark = resolvedTheme === "dark";

  return (
    <button
      onClick={() => setTheme(isDark ? "light" : "dark")}
      className="relative flex h-10 w-10 items-center justify-center rounded-full bg-zinc-100 dark:bg-zinc-800"
      aria-label={`Switch to ${isDark ? "light" : "dark"} mode`}
    >
      <AnimatePresence mode="wait" initial={false}>
        <motion.div
          key={resolvedTheme}
          initial={{ y: -20, opacity: 0, rotate: -90 }}
          animate={{ y: 0, opacity: 1, rotate: 0 }}
          exit={{ y: 20, opacity: 0, rotate: 90 }}
          transition={{ duration: 0.2 }}
        >
          {isDark ? (
            <Moon className="h-5 w-5 text-zinc-300" />
          ) : (
            <Sun className="h-5 w-5 text-amber-500" />
          )}
        </motion.div>
      </AnimatePresence>
    </button>
  );
}
```

### Dropdown Theme Selector

```tsx
// components/theme-dropdown.tsx
"use client";

import { useTheme } from "next-themes";
import { useEffect, useState, useRef } from "react";
import { motion, AnimatePresence } from "framer-motion";
import { Sun, Moon, Monitor, ChevronDown } from "lucide-react";

const themes = [
  { value: "light", label: "Light", icon: Sun },
  { value: "dark", label: "Dark", icon: Moon },
  { value: "system", label: "System", icon: Monitor },
] as const;

export function ThemeDropdown() {
  const [mounted, setMounted] = useState(false);
  const [open, setOpen] = useState(false);
  const { theme, setTheme } = useTheme();
  const dropdownRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    setMounted(true);
  }, []);

  // Close on outside click
  useEffect(() => {
    function handleClickOutside(event: MouseEvent) {
      if (
        dropdownRef.current &&
        !dropdownRef.current.contains(event.target as Node)
      ) {
        setOpen(false);
      }
    }
    document.addEventListener("mousedown", handleClickOutside);
    return () => document.removeEventListener("mousedown", handleClickOutside);
  }, []);

  if (!mounted) {
    return (
      <div className="h-10 w-32 rounded-lg bg-zinc-200 dark:bg-zinc-800" />
    );
  }

  const currentTheme = themes.find((t) => t.value === theme) || themes[2];
  const CurrentIcon = currentTheme.icon;

  return (
    <div ref={dropdownRef} className="relative">
      <button
        onClick={() => setOpen(!open)}
        className="flex h-10 items-center gap-2 rounded-lg bg-zinc-100 px-3 text-sm dark:bg-zinc-800"
        aria-expanded={open}
        aria-haspopup="listbox"
      >
        <CurrentIcon className="h-4 w-4" />
        <span>{currentTheme.label}</span>
        <ChevronDown
          className={`h-4 w-4 transition-transform ${open ? "rotate-180" : ""}`}
        />
      </button>

      <AnimatePresence>
        {open && (
          <motion.ul
            initial={{ opacity: 0, y: -10 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: -10 }}
            className="absolute right-0 top-12 z-50 w-40 rounded-lg border border-zinc-200 bg-white p-1 shadow-lg dark:border-zinc-700 dark:bg-zinc-800"
            role="listbox"
          >
            {themes.map((t) => (
              <li key={t.value}>
                <button
                  onClick={() => {
                    setTheme(t.value);
                    setOpen(false);
                  }}
                  className={`flex w-full items-center gap-2 rounded-md px-3 py-2 text-sm transition-colors hover:bg-zinc-100 dark:hover:bg-zinc-700 ${
                    theme === t.value ? "text-brand-500" : ""
                  }`}
                  role="option"
                  aria-selected={theme === t.value}
                >
                  <t.icon className="h-4 w-4" />
                  {t.label}
                </button>
              </li>
            ))}
          </motion.ul>
        )}
      </AnimatePresence>
    </div>
  );
}
```

## CSS Custom Properties System

### Design Tokens

```css
/* app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    /* Background Colors */
    --background: 0 0% 100%; /* white */
    --foreground: 240 10% 3.9%; /* zinc-950 */

    /* Card & Surface Colors */
    --card: 0 0% 100%;
    --card-foreground: 240 10% 3.9%;

    /* Primary Brand Color */
    --primary: 262.1 83.3% 57.8%; /* violet-500 */
    --primary-foreground: 0 0% 98%;

    /* Secondary Color */
    --secondary: 240 4.8% 95.9%; /* zinc-100 */
    --secondary-foreground: 240 5.9% 10%;

    /* Muted/Subtle */
    --muted: 240 4.8% 95.9%;
    --muted-foreground: 240 3.8% 46.1%;

    /* Accent */
    --accent: 240 4.8% 95.9%;
    --accent-foreground: 240 5.9% 10%;

    /* Destructive/Error */
    --destructive: 0 84.2% 60.2%; /* red-500 */
    --destructive-foreground: 0 0% 98%;

    /* Borders & Inputs */
    --border: 240 5.9% 90%;
    --input: 240 5.9% 90%;
    --ring: 262.1 83.3% 57.8%;

    /* Radius */
    --radius: 0.5rem;
  }

  .dark {
    /* Dark Mode Overrides */
    --background: 240 10% 3.9%; /* zinc-950 */
    --foreground: 0 0% 98%; /* zinc-50 */

    --card: 240 10% 3.9%;
    --card-foreground: 0 0% 98%;

    --primary: 263.4 70% 50.4%; /* violet-600 */
    --primary-foreground: 0 0% 98%;

    --secondary: 240 3.7% 15.9%; /* zinc-800 */
    --secondary-foreground: 0 0% 98%;

    --muted: 240 3.7% 15.9%;
    --muted-foreground: 240 5% 64.9%;

    --accent: 240 3.7% 15.9%;
    --accent-foreground: 0 0% 98%;

    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 0 0% 98%;

    --border: 240 3.7% 15.9%;
    --input: 240 3.7% 15.9%;
    --ring: 263.4 70% 50.4%;
  }
}
```

### Tailwind Configuration

```js
// tailwind.config.js
module.exports = {
  darkMode: "class", // Required for next-themes
  theme: {
    extend: {
      colors: {
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
    },
  },
};
```

### Using Theme Colors

```tsx
// Components use semantic color names
<div className="bg-background text-foreground">
  <div className="rounded-lg border border-border bg-card p-4">
    <h2 className="text-card-foreground">Card Title</h2>
    <p className="text-muted-foreground">Card description</p>

    <button className="bg-primary text-primary-foreground">
      Primary Action
    </button>

    <button className="bg-secondary text-secondary-foreground">
      Secondary Action
    </button>
  </div>
</div>
```

## Multiple Theme Support

### Extended Themes

```css
/* app/globals.css */
@layer base {
  :root {
    /* Light theme (default) */
  }

  .dark {
    /* Dark theme */
  }

  .theme-blue {
    --primary: 221.2 83.2% 53.3%; /* blue-500 */
    --ring: 221.2 83.2% 53.3%;
  }

  .theme-green {
    --primary: 142.1 76.2% 36.3%; /* green-600 */
    --ring: 142.1 76.2% 36.3%;
  }

  .theme-orange {
    --primary: 24.6 95% 53.1%; /* orange-500 */
    --ring: 24.6 95% 53.1%;
  }

  /* Combine with dark mode */
  .dark.theme-blue {
    --primary: 217.2 91.2% 59.8%; /* blue-400 */
    --ring: 217.2 91.2% 59.8%;
  }
}
```

### Theme & Color Picker

```tsx
// components/theme-picker.tsx
"use client";

import { useTheme } from "next-themes";
import { useEffect, useState } from "react";

const colorThemes = [
  { value: "default", label: "Violet", color: "#8b5cf6" },
  { value: "theme-blue", label: "Blue", color: "#3b82f6" },
  { value: "theme-green", label: "Green", color: "#22c55e" },
  { value: "theme-orange", label: "Orange", color: "#f97316" },
];

export function ThemePicker() {
  const [mounted, setMounted] = useState(false);
  const { theme, setTheme, resolvedTheme } = useTheme();
  const [colorTheme, setColorTheme] = useState("default");

  useEffect(() => {
    setMounted(true);
    // Load saved color theme
    const saved = localStorage.getItem("color-theme");
    if (saved) setColorTheme(saved);
  }, []);

  useEffect(() => {
    // Apply color theme class
    document.documentElement.classList.remove(
      ...colorThemes.map((t) => t.value),
    );
    if (colorTheme !== "default") {
      document.documentElement.classList.add(colorTheme);
    }
    localStorage.setItem("color-theme", colorTheme);
  }, [colorTheme]);

  if (!mounted) return null;

  return (
    <div className="space-y-4">
      {/* Mode Toggle */}
      <div>
        <label className="text-sm font-medium">Mode</label>
        <div className="mt-2 flex gap-2">
          {["light", "dark", "system"].map((mode) => (
            <button
              key={mode}
              onClick={() => setTheme(mode)}
              className={`rounded-lg px-4 py-2 text-sm capitalize ${
                theme === mode
                  ? "bg-primary text-primary-foreground"
                  : "bg-secondary"
              }`}
            >
              {mode}
            </button>
          ))}
        </div>
      </div>

      {/* Color Theme */}
      <div>
        <label className="text-sm font-medium">Accent Color</label>
        <div className="mt-2 flex gap-2">
          {colorThemes.map((ct) => (
            <button
              key={ct.value}
              onClick={() => setColorTheme(ct.value)}
              className={`flex h-10 w-10 items-center justify-center rounded-full ring-offset-2 ring-offset-background ${
                colorTheme === ct.value ? "ring-2 ring-ring" : ""
              }`}
              style={{ backgroundColor: ct.color }}
              aria-label={ct.label}
            />
          ))}
        </div>
      </div>
    </div>
  );
}
```

## Dark Mode Images

### Conditional Images

```tsx
// Different images for light/dark
export function LogoImage() {
  return (
    <>
      <Image
        src="/logo-light.svg"
        alt="Logo"
        width={120}
        height={40}
        className="dark:hidden"
      />
      <Image
        src="/logo-dark.svg"
        alt="Logo"
        width={120}
        height={40}
        className="hidden dark:block"
      />
    </>
  );
}

// Using CSS filter for simple icons
<Image
  src="/icon.svg"
  alt="Icon"
  width={24}
  height={24}
  className="dark:invert"
/>;
```

### Image with Overlay

```tsx
// Darken images in dark mode
<div className="relative">
  <Image
    src="/photo.jpg"
    alt="Photo"
    width={400}
    height={300}
    className="rounded-lg"
  />
  <div className="absolute inset-0 bg-black/0 dark:bg-black/30" />
</div>
```

## Theme-Aware Gradients

```tsx
// Gradient that adapts to theme
<div
  className="
    bg-gradient-to-br
    from-violet-500/20 to-purple-500/20
    dark:from-violet-500/10 dark:to-purple-500/10
  "
>
  Content
</div>

// Glass effect
<div
  className="
    border border-zinc-200 bg-white/80 backdrop-blur-lg
    dark:border-zinc-800 dark:bg-zinc-900/80
  "
>
  Glassmorphism
</div>
```

## Smooth Theme Transitions

### Global Transition

```css
/* app/globals.css */
@layer base {
  * {
    @apply transition-colors duration-200;
  }
}

/* Or be more specific to prevent animation issues */
body,
.theme-transition {
  transition:
    background-color 0.3s ease,
    color 0.3s ease,
    border-color 0.3s ease;
}
```

### Disable During Switch

```tsx
// ThemeProvider with transition handling
<NextThemesProvider
  attribute="class"
  defaultTheme="system"
  enableSystem
  disableTransitionOnChange // Prevents flash
>
```

## Theme Hook Utilities

```tsx
// hooks/use-theme-utils.ts
import { useTheme } from "next-themes";
import { useEffect, useState } from "react";

/**
 * Extended theme hook with utilities.
 */
export function useThemeUtils() {
  const { theme, setTheme, resolvedTheme, systemTheme } = useTheme();
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
  }, []);

  const isDark = resolvedTheme === "dark";
  const isLight = resolvedTheme === "light";
  const isSystem = theme === "system";

  const toggle = () => {
    setTheme(isDark ? "light" : "dark");
  };

  const cycleTheme = () => {
    const themes = ["light", "dark", "system"];
    const currentIndex = themes.indexOf(theme || "system");
    const nextIndex = (currentIndex + 1) % themes.length;
    setTheme(themes[nextIndex]);
  };

  return {
    theme,
    setTheme,
    resolvedTheme,
    systemTheme,
    mounted,
    isDark,
    isLight,
    isSystem,
    toggle,
    cycleTheme,
  };
}
```

## Complete Example: Themed Card

```tsx
// components/themed-card.tsx
import { motion } from "framer-motion";

interface ThemedCardProps {
  title: string;
  description: string;
  icon: React.ReactNode;
}

export function ThemedCard({ title, description, icon }: ThemedCardProps) {
  return (
    <motion.article
      whileHover={{ y: -4 }}
      className="
        group relative overflow-hidden rounded-xl
        border border-border bg-card p-6
        transition-shadow hover:shadow-lg
        dark:hover:shadow-primary/5
      "
    >
      {/* Gradient overlay - theme aware */}
      <div
        className="
          absolute inset-0 opacity-0 transition-opacity group-hover:opacity-100
          bg-gradient-to-br from-primary/5 to-transparent
        "
      />

      <div className="relative z-10">
        {/* Icon with theme colors */}
        <div
          className="
            mb-4 flex h-12 w-12 items-center justify-center rounded-lg
            bg-primary/10 text-primary
          "
        >
          {icon}
        </div>

        <h3 className="mb-2 text-lg font-semibold text-card-foreground">
          {title}
        </h3>

        <p className="text-muted-foreground">{description}</p>
      </div>
    </motion.article>
  );
}
```

## Testing Checklist

- [ ] Theme persists across page reloads
- [ ] No flash of unstyled content (FOUC)
- [ ] System preference detection works
- [ ] Toggle animations are smooth
- [ ] Images/icons adapt to theme
- [ ] Contrast ratios meet WCAG in both modes
- [ ] Form inputs visible in both modes
- [ ] Focus states visible in both modes
- [ ] Third-party embeds adapt (if applicable)

## Troubleshooting

### Flash of Wrong Theme

- Ensure `suppressHydrationWarning` on `<html>`
- Use `disableTransitionOnChange` prop
- Check that provider wraps entire app

### Hydration Mismatch

- Use mounted check before rendering theme-dependent UI
- Render placeholder during SSR

### Theme Not Persisting

- Check localStorage is available
- Ensure `storageKey` prop if custom
- Verify no conflicting localStorage keys
