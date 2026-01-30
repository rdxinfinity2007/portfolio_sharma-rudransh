---
name: typescript-patterns
description: Advanced TypeScript patterns for Next.js portfolios including type utilities, generics, discriminated unions, and strict type safety.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# TypeScript Patterns for Next.js

## Overview

TypeScript isn't just about types—it's about making impossible states impossible. This skill covers advanced patterns for rock-solid, maintainable code.

## Project Configuration

### tsconfig.json (Strict Mode)

```json
{
  "compilerOptions": {
    // Strict type checking
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true,

    // Module resolution
    "moduleResolution": "bundler",
    "module": "ESNext",
    "target": "ES2022",
    "lib": ["dom", "dom.iterable", "esnext"],

    // Path aliases
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/lib/*": ["./src/lib/*"],
      "@/types/*": ["./src/types/*"]
    },

    // Next.js specific
    "jsx": "preserve",
    "incremental": true,
    "esModuleInterop": true,
    "allowJs": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,

    // Plugins
    "plugins": [{ "name": "next" }]
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

## Type Definitions

### Project Types

```typescript
// types/project.ts

/**
 * Project status enum for type-safe status handling.
 */
export const ProjectStatus = {
  DRAFT: "draft",
  PUBLISHED: "published",
  ARCHIVED: "archived",
} as const;

export type ProjectStatus = (typeof ProjectStatus)[keyof typeof ProjectStatus];

/**
 * Project entity with strict typing.
 */
export interface Project {
  id: string;
  slug: string;
  title: string;
  description: string;
  content: string;
  coverImage: string;
  technologies: string[];
  liveUrl?: string;
  githubUrl?: string;
  status: ProjectStatus;
  featured: boolean;
  order: number;
  createdAt: Date;
  updatedAt: Date;
}

/**
 * Partial project for creation (without system fields).
 */
export type CreateProjectInput = Omit<
  Project,
  "id" | "createdAt" | "updatedAt"
>;

/**
 * Partial project for updates.
 */
export type UpdateProjectInput = Partial<CreateProjectInput>;

/**
 * Project card display data (subset for list views).
 */
export type ProjectCard = Pick<
  Project,
  | "id"
  | "slug"
  | "title"
  | "description"
  | "coverImage"
  | "technologies"
  | "featured"
>;
```

### Experience Types

```typescript
// types/experience.ts

export interface Experience {
  id: string;
  company: string;
  role: string;
  description: string;
  startDate: Date;
  endDate: Date | null; // null = current
  location: string;
  type: "full-time" | "part-time" | "contract" | "freelance";
  technologies: string[];
  achievements: string[];
  companyLogo?: string;
  companyUrl?: string;
}

/**
 * Computed property: Is this a current position?
 */
export function isCurrentPosition(exp: Experience): boolean {
  return exp.endDate === null;
}

/**
 * Format date range for display.
 */
export function formatExperienceRange(exp: Experience): string {
  const startStr = exp.startDate.toLocaleDateString("en-US", {
    month: "short",
    year: "numeric",
  });
  const endStr = exp.endDate
    ? exp.endDate.toLocaleDateString("en-US", {
        month: "short",
        year: "numeric",
      })
    : "Present";
  return `${startStr} - ${endStr}`;
}
```

## Generic Patterns

### API Response Wrapper

```typescript
// types/api.ts

/**
 * Discriminated union for API responses.
 * Makes error handling exhaustive and type-safe.
 */
export type ApiResponse<T> =
  | { success: true; data: T }
  | { success: false; error: ApiError };

export interface ApiError {
  code: string;
  message: string;
  details?: Record<string, string[]>;
}

/**
 * Type guard for successful responses.
 */
export function isSuccess<T>(
  response: ApiResponse<T>,
): response is { success: true; data: T } {
  return response.success === true;
}

/**
 * Type guard for error responses.
 */
export function isError<T>(
  response: ApiResponse<T>,
): response is { success: false; error: ApiError } {
  return response.success === false;
}

// Usage
async function fetchProject(slug: string): Promise<ApiResponse<Project>> {
  try {
    const res = await fetch(`/api/projects/${slug}`);
    const data = await res.json();
    return { success: true, data };
  } catch (error) {
    return {
      success: false,
      error: { code: "FETCH_ERROR", message: "Failed to fetch project" },
    };
  }
}

// Consumer
const response = await fetchProject("my-project");
if (isSuccess(response)) {
  console.log(response.data.title); // ✅ TypeScript knows data exists
} else {
  console.error(response.error.message); // ✅ TypeScript knows error exists
}
```

### Paginated Response

```typescript
// types/pagination.ts

export interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    pageSize: number;
    totalItems: number;
    totalPages: number;
    hasNextPage: boolean;
    hasPreviousPage: boolean;
  };
}

export interface CursorPaginatedResponse<T> {
  data: T[];
  cursor: {
    next: string | null;
    previous: string | null;
  };
  hasMore: boolean;
}

/**
 * Generic pagination params.
 */
export interface PaginationParams {
  page?: number;
  pageSize?: number;
  sortBy?: string;
  sortOrder?: "asc" | "desc";
}

/**
 * Generic function to paginate an array.
 */
export function paginate<T>(
  items: T[],
  { page = 1, pageSize = 10 }: PaginationParams,
): PaginatedResponse<T> {
  const start = (page - 1) * pageSize;
  const end = start + pageSize;
  const totalItems = items.length;
  const totalPages = Math.ceil(totalItems / pageSize);

  return {
    data: items.slice(start, end),
    pagination: {
      page,
      pageSize,
      totalItems,
      totalPages,
      hasNextPage: page < totalPages,
      hasPreviousPage: page > 1,
    },
  };
}
```

## Discriminated Unions

### Form State Machine

```typescript
// types/form.ts

/**
 * Form state as a discriminated union.
 * Makes invalid states impossible.
 */
export type FormState<T> =
  | { status: "idle" }
  | { status: "submitting" }
  | { status: "success"; data: T }
  | { status: "error"; error: string; fieldErrors?: Record<keyof T, string> };

/**
 * Exhaustive switch for form states.
 */
function renderFormStatus<T>(state: FormState<T>): string {
  switch (state.status) {
    case "idle":
      return "Ready to submit";
    case "submitting":
      return "Submitting...";
    case "success":
      return "Success!";
    case "error":
      return `Error: ${state.error}`;
    // TypeScript ensures all cases are handled
    default: {
      const _exhaustive: never = state;
      throw new Error(`Unhandled state: ${_exhaustive}`);
    }
  }
}
```

### Navigation Item Types

```typescript
// types/navigation.ts

/**
 * Discriminated union for different nav item types.
 */
export type NavItem =
  | { type: 'link'; label: string; href: string }
  | { type: 'button'; label: string; onClick: () => void }
  | { type: 'dropdown'; label: string; items: NavItem[] }
  | { type: 'divider' };

/**
 * Render nav items with exhaustive handling.
 */
function renderNavItem(item: NavItem): React.ReactNode {
  switch (item.type) {
    case 'link':
      return <a href={item.href}>{item.label}</a>;
    case 'button':
      return <button onClick={item.onClick}>{item.label}</button>;
    case 'dropdown':
      return (
        <div className="dropdown">
          <span>{item.label}</span>
          <ul>{item.items.map(renderNavItem)}</ul>
        </div>
      );
    case 'divider':
      return <hr />;
    default: {
      const _exhaustive: never = item;
      throw new Error(`Unhandled nav item type`);
    }
  }
}
```

## Utility Types

### Common Utilities

```typescript
// types/utils.ts

/**
 * Make specific properties required.
 */
export type WithRequired<T, K extends keyof T> = T & { [P in K]-?: T[P] };

/**
 * Make specific properties optional.
 */
export type WithOptional<T, K extends keyof T> = Omit<T, K> &
  Partial<Pick<T, K>>;

/**
 * Make all properties nullable.
 */
export type Nullable<T> = { [K in keyof T]: T[K] | null };

/**
 * Deep partial type.
 */
export type DeepPartial<T> = T extends object
  ? { [P in keyof T]?: DeepPartial<T[P]> }
  : T;

/**
 * Deep required type.
 */
export type DeepRequired<T> = T extends object
  ? { [P in keyof T]-?: DeepRequired<T[P]> }
  : T;

/**
 * Extract keys of a specific type.
 */
export type KeysOfType<T, U> = {
  [K in keyof T]: T[K] extends U ? K : never;
}[keyof T];

/**
 * Non-nullable object.
 */
export type NonNullableObject<T> = {
  [K in keyof T]: NonNullable<T[K]>;
};

/**
 * Strict omit that only allows existing keys.
 */
export type StrictOmit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;

// Usage Examples
type ProjectWithSlug = WithRequired<Partial<Project>, "slug">;
type ProjectDraft = WithOptional<Project, "id" | "createdAt" | "updatedAt">;
```

### Zod Inference

```typescript
// lib/schemas/project.ts
import { z } from "zod";

/**
 * Project schema with runtime validation.
 */
export const projectSchema = z.object({
  id: z.string().uuid(),
  slug: z
    .string()
    .min(1)
    .max(100)
    .regex(/^[a-z0-9-]+$/),
  title: z.string().min(1).max(200),
  description: z.string().min(1).max(500),
  content: z.string(),
  coverImage: z.string().url(),
  technologies: z.array(z.string()).min(1),
  liveUrl: z.string().url().optional(),
  githubUrl: z.string().url().optional(),
  status: z.enum(["draft", "published", "archived"]),
  featured: z.boolean().default(false),
  order: z.number().int().min(0),
  createdAt: z.date(),
  updatedAt: z.date(),
});

/**
 * Type inferred from schema - single source of truth.
 */
export type Project = z.infer<typeof projectSchema>;

/**
 * Schema for creating a project (without system fields).
 */
export const createProjectSchema = projectSchema.omit({
  id: true,
  createdAt: true,
  updatedAt: true,
});

export type CreateProjectInput = z.infer<typeof createProjectSchema>;

/**
 * Schema for updating a project (all fields optional).
 */
export const updateProjectSchema = createProjectSchema.partial();

export type UpdateProjectInput = z.infer<typeof updateProjectSchema>;
```

## Component Props Patterns

### Polymorphic Components

```typescript
// components/button.tsx
import { ComponentPropsWithoutRef, ElementType, ReactNode } from 'react';

/**
 * Props for polymorphic component.
 */
type AsProp<C extends ElementType> = {
  as?: C;
};

type PropsToOmit<C extends ElementType, P> = keyof (AsProp<C> & P);

type PolymorphicComponentProp<
  C extends ElementType,
  Props = object,
> = React.PropsWithChildren<Props & AsProp<C>> &
  Omit<ComponentPropsWithoutRef<C>, PropsToOmit<C, Props>>;

/**
 * Button props with variant support.
 */
interface ButtonOwnProps {
  variant?: 'primary' | 'secondary' | 'ghost' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
  leftIcon?: ReactNode;
  rightIcon?: ReactNode;
}

type ButtonProps<C extends ElementType = 'button'> = PolymorphicComponentProp<
  C,
  ButtonOwnProps
>;

/**
 * Polymorphic Button component.
 * Can render as button, a, Link, etc.
 */
export function Button<C extends ElementType = 'button'>({
  as,
  variant = 'primary',
  size = 'md',
  isLoading = false,
  leftIcon,
  rightIcon,
  children,
  className,
  ...props
}: ButtonProps<C>) {
  const Component = as || 'button';

  return (
    <Component
      className={cn(
        'inline-flex items-center justify-center font-medium',
        variants[variant],
        sizes[size],
        isLoading && 'opacity-50 pointer-events-none',
        className
      )}
      {...props}
    >
      {isLoading ? <Spinner /> : leftIcon}
      {children}
      {!isLoading && rightIcon}
    </Component>
  );
}

// Usage
<Button variant="primary">Click me</Button>
<Button as="a" href="/about">Go to About</Button>
<Button as={Link} href="/projects">View Projects</Button>
```

### Compound Components

```typescript
// components/card.tsx
import { createContext, useContext, ReactNode } from 'react';

interface CardContextValue {
  variant: 'default' | 'bordered' | 'elevated';
}

const CardContext = createContext<CardContextValue | null>(null);

function useCardContext() {
  const context = useContext(CardContext);
  if (!context) {
    throw new Error('Card components must be used within a Card');
  }
  return context;
}

interface CardProps {
  children: ReactNode;
  variant?: CardContextValue['variant'];
  className?: string;
}

function Card({ children, variant = 'default', className }: CardProps) {
  return (
    <CardContext.Provider value={{ variant }}>
      <article className={cn('rounded-lg', variantStyles[variant], className)}>
        {children}
      </article>
    </CardContext.Provider>
  );
}

interface CardHeaderProps {
  children: ReactNode;
  className?: string;
}

function CardHeader({ children, className }: CardHeaderProps) {
  const { variant } = useCardContext();
  return (
    <header className={cn('p-4', variant === 'bordered' && 'border-b', className)}>
      {children}
    </header>
  );
}

function CardContent({ children, className }: { children: ReactNode; className?: string }) {
  return <div className={cn('p-4', className)}>{children}</div>;
}

function CardFooter({ children, className }: { children: ReactNode; className?: string }) {
  const { variant } = useCardContext();
  return (
    <footer className={cn('p-4', variant === 'bordered' && 'border-t', className)}>
      {children}
    </footer>
  );
}

// Attach subcomponents
Card.Header = CardHeader;
Card.Content = CardContent;
Card.Footer = CardFooter;

export { Card };

// Usage
<Card variant="bordered">
  <Card.Header>
    <h3>Title</h3>
  </Card.Header>
  <Card.Content>
    <p>Content</p>
  </Card.Content>
  <Card.Footer>
    <Button>Action</Button>
  </Card.Footer>
</Card>
```

## Type Guards

```typescript
// lib/type-guards.ts

/**
 * Check if value is defined (not null or undefined).
 */
export function isDefined<T>(value: T | null | undefined): value is T {
  return value !== null && value !== undefined;
}

/**
 * Check if array has items.
 */
export function hasItems<T>(arr: T[] | null | undefined): arr is T[] {
  return Array.isArray(arr) && arr.length > 0;
}

/**
 * Check if object has key.
 */
export function hasKey<K extends string>(
  obj: unknown,
  key: K,
): obj is Record<K, unknown> {
  return typeof obj === "object" && obj !== null && key in obj;
}

/**
 * Narrow unknown to string.
 */
export function isString(value: unknown): value is string {
  return typeof value === "string";
}

/**
 * Narrow unknown to number.
 */
export function isNumber(value: unknown): value is number {
  return typeof value === "number" && !isNaN(value);
}

/**
 * Narrow unknown to object.
 */
export function isObject(value: unknown): value is Record<string, unknown> {
  return typeof value === "object" && value !== null && !Array.isArray(value);
}

// Usage with filter
const items: (string | null | undefined)[] = ["a", null, "b", undefined, "c"];
const definedItems = items.filter(isDefined); // string[]
```

## Async Patterns

### Safe Async Handler

```typescript
// lib/async.ts

/**
 * Wrap async function to return tuple [error, result].
 * Eliminates try-catch boilerplate.
 */
export async function safeAsync<T>(
  promise: Promise<T>,
): Promise<[Error, null] | [null, T]> {
  try {
    const result = await promise;
    return [null, result];
  } catch (error) {
    if (error instanceof Error) {
      return [error, null];
    }
    return [new Error(String(error)), null];
  }
}

// Usage
async function fetchData() {
  const [error, data] = await safeAsync(
    fetch("/api/projects").then((r) => r.json()),
  );

  if (error) {
    console.error("Failed:", error.message);
    return null;
  }

  return data; // TypeScript knows this is the response type
}
```

### Retry with Exponential Backoff

```typescript
// lib/retry.ts

interface RetryOptions {
  maxAttempts?: number;
  initialDelay?: number;
  maxDelay?: number;
  backoffMultiplier?: number;
  shouldRetry?: (error: Error, attempt: number) => boolean;
}

/**
 * Retry async function with exponential backoff.
 */
export async function retry<T>(
  fn: () => Promise<T>,
  options: RetryOptions = {},
): Promise<T> {
  const {
    maxAttempts = 3,
    initialDelay = 1000,
    maxDelay = 30000,
    backoffMultiplier = 2,
    shouldRetry = () => true,
  } = options;

  let lastError: Error;
  let delay = initialDelay;

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error instanceof Error ? error : new Error(String(error));

      if (attempt === maxAttempts || !shouldRetry(lastError, attempt)) {
        throw lastError;
      }

      await new Promise((resolve) => setTimeout(resolve, delay));
      delay = Math.min(delay * backoffMultiplier, maxDelay);
    }
  }

  throw lastError!;
}

// Usage
const data = await retry(() => fetch("/api/projects").then((r) => r.json()), {
  maxAttempts: 3,
  shouldRetry: (error) => !error.message.includes("404"),
});
```

## Server Action Types

```typescript
// types/actions.ts

/**
 * Standard server action response type.
 */
export type ActionResult<T = void> =
  | { success: true; data: T }
  | { success: false; error: string; fieldErrors?: Record<string, string[]> };

/**
 * Create type-safe server action.
 */
export type ServerAction<TInput, TOutput> = (
  input: TInput,
) => Promise<ActionResult<TOutput>>;

// Usage in action
export async function createProject(
  input: CreateProjectInput,
): Promise<ActionResult<Project>> {
  const validated = createProjectSchema.safeParse(input);

  if (!validated.success) {
    return {
      success: false,
      error: "Validation failed",
      fieldErrors: validated.error.flatten().fieldErrors,
    };
  }

  // Create project...
  return { success: true, data: project };
}
```

## Constants & Enums

```typescript
// lib/constants.ts

/**
 * Use const assertions for type-safe constants.
 */
export const ROUTES = {
  HOME: "/",
  ABOUT: "/about",
  PROJECTS: "/projects",
  PROJECT: (slug: string) => `/projects/${slug}` as const,
  CONTACT: "/contact",
} as const;

export const SOCIAL_LINKS = {
  GITHUB: "https://github.com/johndoe",
  LINKEDIN: "https://linkedin.com/in/johndoe",
  TWITTER: "https://twitter.com/johndoe",
} as const;

export const ANIMATION_DURATION = {
  FAST: 150,
  NORMAL: 300,
  SLOW: 500,
} as const;

/**
 * Extract types from const objects.
 */
export type Route = (typeof ROUTES)[keyof typeof ROUTES];
export type SocialLink = (typeof SOCIAL_LINKS)[keyof typeof SOCIAL_LINKS];
```

## Best Practices Checklist

- [ ] Enable strict mode in tsconfig.json
- [ ] Use `noUncheckedIndexedAccess` for safer array access
- [ ] Infer types from Zod schemas (single source of truth)
- [ ] Use discriminated unions for state machines
- [ ] Implement type guards for runtime narrowing
- [ ] Prefer `unknown` over `any`
- [ ] Use const assertions for literal types
- [ ] Document complex types with JSDoc
- [ ] Use generics for reusable type-safe utilities
- [ ] Exhaustive switch statements with `never` check
