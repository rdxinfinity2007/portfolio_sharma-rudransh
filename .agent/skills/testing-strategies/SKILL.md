---
name: testing-strategies
description: Comprehensive testing guide with Vitest, React Testing Library, Playwright for E2E, and testing patterns for Next.js applications.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Testing Strategies for Next.js

## Overview

A professional portfolio needs confidence through testing:

1. **Unit Tests** - Test individual functions/utilities
2. **Component Tests** - Test React components in isolation
3. **Integration Tests** - Test component interactions
4. **E2E Tests** - Test full user flows

## Testing Stack

```bash
# Unit & Component Testing
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom

# E2E Testing
npm install -D playwright @playwright/test

# MSW for API Mocking
npm install -D msw
```

## Vitest Configuration

```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: ["./vitest.setup.ts"],
    include: ["**/*.{test,spec}.{ts,tsx}"],
    exclude: ["node_modules", ".next", "e2e"],
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      exclude: [
        "node_modules",
        ".next",
        "**/*.d.ts",
        "**/*.config.*",
        "**/types/*",
      ],
    },
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

### Setup File

```typescript
// vitest.setup.ts
import '@testing-library/jest-dom/vitest';
import { cleanup } from '@testing-library/react';
import { afterEach, vi } from 'vitest';

// Cleanup after each test
afterEach(() => {
  cleanup();
});

// Mock Next.js router
vi.mock('next/navigation', () => ({
  useRouter: () => ({
    push: vi.fn(),
    replace: vi.fn(),
    prefetch: vi.fn(),
    back: vi.fn(),
    forward: vi.fn(),
  }),
  usePathname: () => '/',
  useSearchParams: () => new URLSearchParams(),
  useParams: () => ({}),
}));

// Mock next/image
vi.mock('next/image', () => ({
  default: (props: React.ImgHTMLAttributes<HTMLImageElement>) => {
    // eslint-disable-next-line @next/next/no-img-element
    return <img {...props} alt={props.alt} />;
  },
}));

// Mock framer-motion
vi.mock('framer-motion', () => ({
  motion: {
    div: ({ children, ...props }: React.HTMLProps<HTMLDivElement>) => (
      <div {...props}>{children}</div>
    ),
    span: ({ children, ...props }: React.HTMLProps<HTMLSpanElement>) => (
      <span {...props}>{children}</span>
    ),
    button: ({ children, ...props }: React.HTMLProps<HTMLButtonElement>) => (
      <button {...props}>{children}</button>
    ),
  },
  AnimatePresence: ({ children }: { children: React.ReactNode }) => children,
  useAnimation: () => ({ start: vi.fn(), stop: vi.fn() }),
  useInView: () => true,
}));

// Mock IntersectionObserver
class MockIntersectionObserver implements IntersectionObserver {
  readonly root: Element | null = null;
  readonly rootMargin: string = '';
  readonly thresholds: ReadonlyArray<number> = [];

  constructor(private callback: IntersectionObserverCallback) {}

  observe = vi.fn();
  unobserve = vi.fn();
  disconnect = vi.fn();
  takeRecords = vi.fn(() => []);
}

vi.stubGlobal('IntersectionObserver', MockIntersectionObserver);

// Mock ResizeObserver
class MockResizeObserver {
  observe = vi.fn();
  unobserve = vi.fn();
  disconnect = vi.fn();
}

vi.stubGlobal('ResizeObserver', MockResizeObserver);
```

### Package.json Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui"
  }
}
```

## Unit Testing

### Testing Utilities

```typescript
// lib/utils.ts
/**
 * Format date for display.
 */
export function formatDate(date: Date): string {
  return new Intl.DateTimeFormat("en-US", {
    month: "long",
    day: "numeric",
    year: "numeric",
  }).format(date);
}

/**
 * Generate slug from title.
 */
export function slugify(text: string): string {
  return text
    .toLowerCase()
    .trim()
    .replace(/[^\w\s-]/g, "")
    .replace(/[\s_-]+/g, "-")
    .replace(/^-+|-+$/g, "");
}

/**
 * Truncate text with ellipsis.
 */
export function truncate(text: string, maxLength: number): string {
  if (text.length <= maxLength) return text;
  return text.slice(0, maxLength).trim() + "...";
}

/**
 * Calculate read time.
 */
export function calculateReadTime(content: string): number {
  const wordsPerMinute = 200;
  const wordCount = content.trim().split(/\s+/).length;
  return Math.ceil(wordCount / wordsPerMinute);
}
```

```typescript
// lib/__tests__/utils.test.ts
import { describe, it, expect } from "vitest";
import { formatDate, slugify, truncate, calculateReadTime } from "../utils";

describe("formatDate", () => {
  it("formats date correctly", () => {
    const date = new Date("2024-03-15");
    expect(formatDate(date)).toBe("March 15, 2024");
  });

  it("handles different dates", () => {
    const date = new Date("2024-01-01");
    expect(formatDate(date)).toBe("January 1, 2024");
  });
});

describe("slugify", () => {
  it("converts text to slug", () => {
    expect(slugify("Hello World")).toBe("hello-world");
  });

  it("removes special characters", () => {
    expect(slugify("Hello, World!")).toBe("hello-world");
  });

  it("handles multiple spaces", () => {
    expect(slugify("Hello   World")).toBe("hello-world");
  });

  it("trims leading/trailing hyphens", () => {
    expect(slugify("  Hello World  ")).toBe("hello-world");
  });

  it("handles empty string", () => {
    expect(slugify("")).toBe("");
  });
});

describe("truncate", () => {
  it("truncates long text", () => {
    const text = "This is a very long text that needs truncation";
    expect(truncate(text, 20)).toBe("This is a very long...");
  });

  it("returns original if shorter than max", () => {
    const text = "Short text";
    expect(truncate(text, 20)).toBe("Short text");
  });

  it("returns original if equal to max", () => {
    const text = "Exactly twenty chars";
    expect(truncate(text, 20)).toBe("Exactly twenty chars");
  });
});

describe("calculateReadTime", () => {
  it("calculates read time correctly", () => {
    const content = Array(400).fill("word").join(" "); // 400 words
    expect(calculateReadTime(content)).toBe(2); // 2 minutes
  });

  it("rounds up read time", () => {
    const content = Array(201).fill("word").join(" "); // 201 words
    expect(calculateReadTime(content)).toBe(2); // rounds up
  });

  it("handles empty content", () => {
    expect(calculateReadTime("")).toBe(1);
  });
});
```

### Testing Zod Schemas

```typescript
// lib/schemas/__tests__/contact.test.ts
import { describe, it, expect } from "vitest";
import { contactSchema } from "../contact";

describe("contactSchema", () => {
  const validData = {
    name: "John Doe",
    email: "john@example.com",
    subject: "Project Inquiry",
    message: "I would like to discuss a potential project with you.",
  };

  it("validates correct data", () => {
    const result = contactSchema.safeParse(validData);
    expect(result.success).toBe(true);
  });

  it("rejects invalid email", () => {
    const result = contactSchema.safeParse({
      ...validData,
      email: "invalid-email",
    });
    expect(result.success).toBe(false);
    if (!result.success) {
      expect(result.error.issues[0].path).toContain("email");
    }
  });

  it("rejects short name", () => {
    const result = contactSchema.safeParse({
      ...validData,
      name: "J",
    });
    expect(result.success).toBe(false);
  });

  it("rejects short message", () => {
    const result = contactSchema.safeParse({
      ...validData,
      message: "Too short",
    });
    expect(result.success).toBe(false);
  });

  it("normalizes email to lowercase", () => {
    const result = contactSchema.safeParse({
      ...validData,
      email: "John@Example.COM",
    });
    expect(result.success).toBe(true);
    if (result.success) {
      expect(result.data.email).toBe("john@example.com");
    }
  });
});
```

## Component Testing

### Test Utilities

```typescript
// test/utils.tsx
import { render, RenderOptions } from '@testing-library/react';
import { ThemeProvider } from 'next-themes';
import { ReactElement, ReactNode } from 'react';

interface WrapperProps {
  children: ReactNode;
}

function AllProviders({ children }: WrapperProps) {
  return (
    <ThemeProvider attribute="class" defaultTheme="dark">
      {children}
    </ThemeProvider>
  );
}

function customRender(
  ui: ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>
) {
  return render(ui, { wrapper: AllProviders, ...options });
}

export * from '@testing-library/react';
export { customRender as render };
```

### Testing Button Component

```typescript
// components/__tests__/button.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@/test/utils';
import userEvent from '@testing-library/user-event';
import { Button } from '../button';

describe('Button', () => {
  it('renders correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  it('handles click events', async () => {
    const user = userEvent.setup();
    const handleClick = vi.fn();

    render(<Button onClick={handleClick}>Click me</Button>);
    await user.click(screen.getByRole('button'));

    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('applies variant classes', () => {
    render(<Button variant="secondary">Secondary</Button>);
    const button = screen.getByRole('button');
    expect(button).toHaveClass('bg-secondary');
  });

  it('applies size classes', () => {
    render(<Button size="lg">Large</Button>);
    const button = screen.getByRole('button');
    expect(button).toHaveClass('px-6', 'py-3');
  });

  it('shows loading state', () => {
    render(<Button isLoading>Loading</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
    // Check for spinner
    expect(screen.getByRole('button').querySelector('svg')).toBeInTheDocument();
  });

  it('is disabled when loading', async () => {
    const user = userEvent.setup();
    const handleClick = vi.fn();

    render(
      <Button isLoading onClick={handleClick}>
        Loading
      </Button>
    );
    await user.click(screen.getByRole('button'));

    expect(handleClick).not.toHaveBeenCalled();
  });

  it('renders as a link when href is provided', () => {
    render(
      <Button as="a" href="/about">
        About
      </Button>
    );
    expect(screen.getByRole('link', { name: /about/i })).toHaveAttribute(
      'href',
      '/about'
    );
  });
});
```

### Testing Contact Form

```typescript
// components/__tests__/contact-form.test.tsx
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { render, screen, waitFor } from '@/test/utils';
import userEvent from '@testing-library/user-event';
import { ContactForm } from '../contact-form';

// Mock server action
vi.mock('@/app/actions/contact', () => ({
  submitContactForm: vi.fn(),
}));

import { submitContactForm } from '@/app/actions/contact';

describe('ContactForm', () => {
  const user = userEvent.setup();

  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('renders all form fields', () => {
    render(<ContactForm />);

    expect(screen.getByLabelText(/name/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/subject/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/message/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /send/i })).toBeInTheDocument();
  });

  it('shows validation errors on blur', async () => {
    render(<ContactForm />);

    const nameInput = screen.getByLabelText(/name/i);
    await user.click(nameInput);
    await user.tab(); // Blur

    await waitFor(() => {
      expect(screen.getByText(/name must be at least/i)).toBeInTheDocument();
    });
  });

  it('submits form with valid data', async () => {
    vi.mocked(submitContactForm).mockResolvedValue({
      success: true,
      message: 'Message sent!',
    });

    render(<ContactForm />);

    await user.type(screen.getByLabelText(/name/i), 'John Doe');
    await user.type(screen.getByLabelText(/email/i), 'john@example.com');
    await user.type(screen.getByLabelText(/subject/i), 'Project Inquiry');
    await user.type(
      screen.getByLabelText(/message/i),
      'I would like to discuss a potential project.'
    );

    await user.click(screen.getByRole('button', { name: /send/i }));

    await waitFor(() => {
      expect(submitContactForm).toHaveBeenCalled();
    });
  });

  it('shows success state after submission', async () => {
    vi.mocked(submitContactForm).mockResolvedValue({
      success: true,
      message: 'Message sent successfully!',
    });

    render(<ContactForm />);

    await user.type(screen.getByLabelText(/name/i), 'John Doe');
    await user.type(screen.getByLabelText(/email/i), 'john@example.com');
    await user.type(screen.getByLabelText(/subject/i), 'Project Inquiry');
    await user.type(
      screen.getByLabelText(/message/i),
      'I would like to discuss a potential project.'
    );

    await user.click(screen.getByRole('button', { name: /send/i }));

    await waitFor(() => {
      expect(screen.getByText(/message sent/i)).toBeInTheDocument();
    });
  });

  it('shows error message on failure', async () => {
    vi.mocked(submitContactForm).mockResolvedValue({
      success: false,
      error: 'Failed to send message',
    });

    render(<ContactForm />);

    await user.type(screen.getByLabelText(/name/i), 'John Doe');
    await user.type(screen.getByLabelText(/email/i), 'john@example.com');
    await user.type(screen.getByLabelText(/subject/i), 'Project Inquiry');
    await user.type(
      screen.getByLabelText(/message/i),
      'I would like to discuss a potential project.'
    );

    await user.click(screen.getByRole('button', { name: /send/i }));

    await waitFor(() => {
      expect(screen.getByText(/failed to send/i)).toBeInTheDocument();
    });
  });

  it('disables submit while submitting', async () => {
    let resolvePromise: (value: unknown) => void;
    vi.mocked(submitContactForm).mockImplementation(
      () =>
        new Promise((resolve) => {
          resolvePromise = resolve;
        })
    );

    render(<ContactForm />);

    await user.type(screen.getByLabelText(/name/i), 'John Doe');
    await user.type(screen.getByLabelText(/email/i), 'john@example.com');
    await user.type(screen.getByLabelText(/subject/i), 'Project Inquiry');
    await user.type(
      screen.getByLabelText(/message/i),
      'I would like to discuss a potential project.'
    );

    await user.click(screen.getByRole('button', { name: /send/i }));

    expect(screen.getByRole('button', { name: /sending/i })).toBeDisabled();

    // Resolve the promise
    resolvePromise!({ success: true, message: 'Done' });
  });
});
```

### Testing with MSW

```typescript
// test/mocks/handlers.ts
import { http, HttpResponse } from "msw";

export const handlers = [
  http.get("/api/projects", () => {
    return HttpResponse.json([
      {
        id: "1",
        slug: "project-1",
        title: "Project One",
        description: "Description for project one",
      },
      {
        id: "2",
        slug: "project-2",
        title: "Project Two",
        description: "Description for project two",
      },
    ]);
  }),

  http.get("/api/projects/:slug", ({ params }) => {
    const { slug } = params;
    return HttpResponse.json({
      id: "1",
      slug,
      title: `Project ${slug}`,
      description: "Project description",
      content: "Full project content...",
    });
  }),

  http.post("/api/contact", async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ success: true, message: "Message sent!" });
  }),
];
```

```typescript
// test/mocks/server.ts
import { setupServer } from "msw/node";
import { handlers } from "./handlers";

export const server = setupServer(...handlers);
```

```typescript
// vitest.setup.ts (add to existing)
import { server } from "./test/mocks/server";

beforeAll(() => server.listen({ onUnhandledRequest: "error" }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## Playwright E2E Testing

### Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: "html",
  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
  },
  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
    {
      name: "firefox",
      use: { ...devices["Desktop Firefox"] },
    },
    {
      name: "webkit",
      use: { ...devices["Desktop Safari"] },
    },
    {
      name: "Mobile Chrome",
      use: { ...devices["Pixel 5"] },
    },
    {
      name: "Mobile Safari",
      use: { ...devices["iPhone 12"] },
    },
  ],
  webServer: {
    command: "npm run dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
  },
});
```

### E2E Tests

```typescript
// e2e/navigation.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Navigation", () => {
  test("navigates to all pages", async ({ page }) => {
    await page.goto("/");

    // Check hero section
    await expect(page.getByRole("heading", { level: 1 })).toBeVisible();

    // Navigate to projects
    await page.getByRole("link", { name: /projects/i }).click();
    await expect(page).toHaveURL("/projects");
    await expect(
      page.getByRole("heading", { name: /projects/i }),
    ).toBeVisible();

    // Navigate to about
    await page.getByRole("link", { name: /about/i }).click();
    await expect(page).toHaveURL("/about");

    // Navigate to contact
    await page.getByRole("link", { name: /contact/i }).click();
    await expect(page).toHaveURL("/contact");
  });

  test("mobile navigation works", async ({ page }) => {
    await page.setViewportSize({ width: 375, height: 667 });
    await page.goto("/");

    // Open mobile menu
    await page.getByRole("button", { name: /menu/i }).click();

    // Navigate using mobile menu
    await page.getByRole("link", { name: /projects/i }).click();
    await expect(page).toHaveURL("/projects");
  });
});
```

```typescript
// e2e/contact-form.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Contact Form", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/contact");
  });

  test("submits form successfully", async ({ page }) => {
    // Fill form
    await page.getByLabel(/name/i).fill("John Doe");
    await page.getByLabel(/email/i).fill("john@example.com");
    await page.getByLabel(/subject/i).fill("Project Inquiry");
    await page
      .getByLabel(/message/i)
      .fill("I would like to discuss a potential project with you.");

    // Submit
    await page.getByRole("button", { name: /send/i }).click();

    // Check success
    await expect(page.getByText(/message sent/i)).toBeVisible();
  });

  test("shows validation errors", async ({ page }) => {
    // Click submit without filling
    await page.getByRole("button", { name: /send/i }).click();

    // Check for errors (form should validate)
    await expect(page.getByText(/name must be/i)).toBeVisible();
  });

  test("prevents spam with honeypot", async ({ page }) => {
    // Fill honeypot field (hidden from real users)
    await page.evaluate(() => {
      const honeypot =
        document.querySelector<HTMLInputElement>('[name="website"]');
      if (honeypot) honeypot.value = "spam-bot-value";
    });

    // Fill rest of form
    await page.getByLabel(/name/i).fill("Bot User");
    await page.getByLabel(/email/i).fill("bot@spam.com");
    await page.getByLabel(/subject/i).fill("Spam Subject");
    await page
      .getByLabel(/message/i)
      .fill("This is a spam message from a bot.");

    await page.getByRole("button", { name: /send/i }).click();

    // Should appear successful but be rejected server-side
    await expect(page.getByText(/message sent/i)).toBeVisible();
  });
});
```

```typescript
// e2e/projects.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Projects", () => {
  test("displays project list", async ({ page }) => {
    await page.goto("/projects");

    // Wait for projects to load
    await expect(page.getByTestId("project-card")).toHaveCount.above(0);
  });

  test("navigates to project detail", async ({ page }) => {
    await page.goto("/projects");

    // Click first project
    await page.getByTestId("project-card").first().click();

    // Check detail page
    await expect(page.getByRole("article")).toBeVisible();
    await expect(page.getByRole("heading", { level: 1 })).toBeVisible();
  });

  test("filters projects by technology", async ({ page }) => {
    await page.goto("/projects");

    // Click filter
    await page.getByRole("button", { name: /react/i }).click();

    // Check filtered results
    const projects = page.getByTestId("project-card");
    for (const project of await projects.all()) {
      await expect(project).toContainText(/react/i);
    }
  });
});
```

### Visual Regression Tests

```typescript
// e2e/visual.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Visual Regression", () => {
  test("homepage matches snapshot", async ({ page }) => {
    await page.goto("/");
    await expect(page).toHaveScreenshot("homepage.png", {
      fullPage: true,
      animations: "disabled",
    });
  });

  test("projects page matches snapshot", async ({ page }) => {
    await page.goto("/projects");
    await expect(page).toHaveScreenshot("projects.png", {
      fullPage: true,
      animations: "disabled",
    });
  });

  test("dark mode matches snapshot", async ({ page }) => {
    await page.goto("/");

    // Toggle dark mode
    await page.getByRole("button", { name: /theme/i }).click();

    await expect(page).toHaveScreenshot("homepage-dark.png", {
      fullPage: true,
      animations: "disabled",
    });
  });

  test("mobile layout matches snapshot", async ({ page }) => {
    await page.setViewportSize({ width: 375, height: 667 });
    await page.goto("/");
    await expect(page).toHaveScreenshot("homepage-mobile.png", {
      fullPage: true,
      animations: "disabled",
    });
  });
});
```

### Accessibility Testing

```typescript
// e2e/accessibility.spec.ts
import { test, expect } from "@playwright/test";
import AxeBuilder from "@axe-core/playwright";

test.describe("Accessibility", () => {
  test("homepage has no accessibility violations", async ({ page }) => {
    await page.goto("/");

    const accessibilityScanResults = await new AxeBuilder({ page }).analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test("contact form is accessible", async ({ page }) => {
    await page.goto("/contact");

    const accessibilityScanResults = await new AxeBuilder({ page })
      .include("form")
      .analyze();

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test("can navigate with keyboard only", async ({ page }) => {
    await page.goto("/");

    // Tab through navigation
    await page.keyboard.press("Tab");
    await expect(page.locator(":focus")).toBeVisible();

    // Continue tabbing
    for (let i = 0; i < 5; i++) {
      await page.keyboard.press("Tab");
      await expect(page.locator(":focus")).toBeVisible();
    }

    // Press Enter on focused link
    await page.keyboard.press("Enter");
    // Should navigate
  });
});
```

## Testing Patterns

### Testing Custom Hooks

```typescript
// hooks/__tests__/use-debounce.test.ts
import { renderHook, act } from "@testing-library/react";
import { describe, it, expect, vi } from "vitest";
import { useDebounce } from "../use-debounce";

describe("useDebounce", () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it("returns initial value immediately", () => {
    const { result } = renderHook(() => useDebounce("initial", 500));
    expect(result.current).toBe("initial");
  });

  it("debounces value changes", () => {
    const { result, rerender } = renderHook(
      ({ value }) => useDebounce(value, 500),
      { initialProps: { value: "initial" } },
    );

    rerender({ value: "updated" });
    expect(result.current).toBe("initial"); // Still old value

    act(() => {
      vi.advanceTimersByTime(500);
    });

    expect(result.current).toBe("updated"); // Now updated
  });

  it("cancels pending updates on unmount", () => {
    const { result, unmount, rerender } = renderHook(
      ({ value }) => useDebounce(value, 500),
      { initialProps: { value: "initial" } },
    );

    rerender({ value: "updated" });
    unmount();

    act(() => {
      vi.advanceTimersByTime(500);
    });

    // No errors should occur
  });
});
```

### Snapshot Testing

```typescript
// components/__tests__/project-card.snapshot.test.tsx
import { describe, it, expect } from 'vitest';
import { render } from '@/test/utils';
import { ProjectCard } from '../project-card';

describe('ProjectCard Snapshots', () => {
  const mockProject = {
    id: '1',
    slug: 'test-project',
    title: 'Test Project',
    description: 'A test project description',
    coverImage: '/test-image.jpg',
    technologies: ['React', 'TypeScript'],
    featured: false,
  };

  it('matches snapshot', () => {
    const { container } = render(<ProjectCard project={mockProject} />);
    expect(container).toMatchSnapshot();
  });

  it('matches snapshot when featured', () => {
    const { container } = render(
      <ProjectCard project={{ ...mockProject, featured: true }} />
    );
    expect(container).toMatchSnapshot();
  });
});
```

## CI/CD Integration

```yaml
# .github/workflows/test.yml
name: Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm run test:run
      - run: npm run test:coverage

  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run build
      - run: npm run test:e2e
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

## Testing Checklist

- [ ] Unit tests for utility functions
- [ ] Unit tests for Zod schemas
- [ ] Component tests with user events
- [ ] Form validation tests
- [ ] Mock server actions properly
- [ ] E2E tests for critical paths
- [ ] Mobile responsive E2E tests
- [ ] Accessibility tests pass
- [ ] Visual regression tests
- [ ] CI runs all tests on PR
- [ ] Coverage thresholds set
