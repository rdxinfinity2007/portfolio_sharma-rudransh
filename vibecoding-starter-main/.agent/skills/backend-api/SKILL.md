---
name: backend-api
description: Build server-side APIs using Next.js Server Actions and API routes.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Backend API Development & Architecture

## Core Philosophy

In the Modern Web (Next.js App Router), the line between frontend and backend is blurred but strict. Code co-location is key, but security boundaries are paramount. We prefer **Server Actions** for mutations and **Server Components** for data fetching.

## Next.js App Router Architecture

### 1. Server Components vs Client Components

By default, everything in `app/` is a Server Component.

| Feature       | Server Component (Default)             | Client Component (`'use client'`)         |
| ------------- | -------------------------------------- | ----------------------------------------- |
| Data Fetching | ✅ Direct DB / API calls (async/await) | ❌ Must use useEffect / SWR / React Query |
| Secrets       | ✅ Can access env vars (API_KEY)       | ❌ Exposure risk                          |
| Interactivity | ❌ No onClick, onChange, hooks         | ✅ Full React interactivity               |
| Bundle Size   | ✅ Zero bundle size impact             | ❌ Adds to client JS bundle               |

**Best Practice**: Keep the "Client Boundary" as low in the tree as possible. Pass Server Components as `children` to Client Components to prevent them from becoming Client Components themselves.

### 2. Data Fetching (The "Backend" in Frontend)

Fetch data directly in your async Page or Layout components.

```javascript
// app/dashboard/page.jsx
import { db } from "@/lib/db";

export default async function DashboardPage() {
  // Direct DB call - safe on server
  const users = await db.user.findMany();

  return (
    <main>
      <h1>Users</h1>
      <UserList users={users} />
    </main>
  );
}
```

## Server Actions (Mutations)

Server Actions are the standard way to handle form submissions and data mutations.

### 1. The Robust Pattern (Zod + Error Handling)

Always validate inputs on the server using Zod. Never trust the client.

```javascript
// actions/create-post.ts
'use server'

import { z } from "zod";
import { db } from "@/lib/db";
import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

const schema = z.object({
  title: z.string().min(5),
  content: z.string().min(10)
});

export async function createPost(prevState: any, formData: FormData) {
  const validatedFields = schema.safeParse({
    title: formData.get('title'),
    content: formData.get('content')
  });

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: 'Missing Fields. Failed to Create Post.',
    };
  }

  try {
    await db.post.create({
      data: validatedFields.data,
    });
  } catch (error) {
    return { message: 'Database Error: Failed to Create Post.' };
  }

  revalidatePath('/dashboard/posts');
  redirect('/dashboard/posts');
}
```

### 2. Consuming Actions in Client Components

Use `useFormState` (or `useActionState` in React 19) to handle feedback.

```jsx
// components/post-form.jsx
"use client";

import { useFormState } from "react-dom";
import { createPost } from "@/actions/create-post";

export function PostForm() {
  const [state, dispatch] = useFormState(createPost, {
    message: null,
    errors: {},
  });

  return (
    <form action={dispatch}>
      <input name="title" />
      {state.errors?.title && (
        <p className="text-red-500">{state.errors.title}</p>
      )}

      <button type="submit">Save</button>
      <p aria-live="polite">{state.message}</p>
    </form>
  );
}
```

## API Routes (Route Handlers)

Use Route Handlers (`app/api/.../route.js`) when you need to expose endpoints to external services (webhooks, mobile apps, public API).

```javascript
// app/api/webhooks/stripe/route.js
import { headers } from "next/headers";
import { NextResponse } from "next/server";

export async function POST(req) {
  const body = await req.text();
  const signature = headers().get("Stripe-Signature");

  try {
    // Verify webhook signature
    // Process event
    return NextResponse.json({ received: true });
  } catch (err) {
    return NextResponse.json({ error: "Webhook Error" }, { status: 400 });
  }
}
```

## Database Best Practices

### 1. Connection Singleton (Prisma/Drizzle)

In development, Next.js hot-reloading can exhaust database connections. Use a singleton pattern.

```javascript
// lib/db.ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

export const db = globalForPrisma.prisma || new PrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = db;
```

### 2. Caching Strategies

- **`unstable_cache`**: Cache expensive DB queries by key/tag.
- **`revalidateTag`**: Purge cache on demand (e.g., after an admin update).

## Security Checklist

1. **Authentication**: Use libraries like **NextAuth.js (Auth.js)** or **Clerk**.
2. **Authorization**: Check permissions inside EVERY Server Action and Server Component.
   ```javascript
   const session = await auth();
   if (!session || session.user.role !== "ADMIN") {
     throw new Error("Unauthorized");
   }
   ```
3. **Input Validation**: Validate everything with Zod.
4. **Environment Variables**: Never prefix secrets with `NEXT_PUBLIC_` unless they are truly public.
