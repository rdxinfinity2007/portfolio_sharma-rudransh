---
name: contact-form
description: Build production-ready contact forms with React Hook Form, Zod validation, Server Actions, and email services like Resend.
author: Jaivish Chauhan @ GDG SSIT
version: 1.0.0
url: https://github.com/JaivishChauhan/vibecoding-starter
---

# Contact Form Implementation

## Overview

A production contact form requires:

1. **Type-safe validation** (Zod)
2. **Efficient form state** (React Hook Form)
3. **Server-side processing** (Server Actions)
4. **Email delivery** (Resend/SendGrid)
5. **Rate limiting** (Upstash Redis)
6. **User feedback** (Toast notifications)

## Dependencies

```bash
npm install react-hook-form @hookform/resolvers zod resend
npm install sonner  # Toast notifications
npm install @upstash/ratelimit @upstash/redis  # Rate limiting (optional)
```

## Form Schema

```typescript
// lib/schemas/contact.ts
import { z } from "zod";

/**
 * Contact form validation schema.
 * @security Validates all input server-side.
 */
export const contactSchema = z.object({
  name: z
    .string()
    .min(2, "Name must be at least 2 characters")
    .max(100, "Name is too long")
    .regex(/^[a-zA-Z\s'-]+$/, "Name contains invalid characters"),

  email: z
    .string()
    .email("Please enter a valid email")
    .max(255, "Email is too long")
    .toLowerCase(),

  subject: z
    .string()
    .min(5, "Subject must be at least 5 characters")
    .max(200, "Subject is too long"),

  message: z
    .string()
    .min(20, "Message must be at least 20 characters")
    .max(5000, "Message is too long"),

  // Honeypot field for spam protection
  website: z.string().max(0, "Bot detected").optional(),
});

export type ContactFormData = z.infer<typeof contactSchema>;

// Server response type
export type ContactResponse = {
  success: boolean;
  message: string;
  errors?: Partial<Record<keyof ContactFormData, string[]>>;
};
```

## Server Action

```typescript
// app/actions/contact.ts
"use server";

import { z } from "zod";
import { Resend } from "resend";
import { headers } from "next/headers";
import { contactSchema, type ContactResponse } from "@/lib/schemas/contact";

// Initialize Resend
const resend = new Resend(process.env.RESEND_API_KEY);

// Optional: Rate limiting with Upstash
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(5, "1 h"), // 5 requests per hour
  analytics: true,
});

/**
 * Server action to process contact form submissions.
 * @security Rate limited, validated, and sanitized.
 */
export async function submitContactForm(
  formData: FormData,
): Promise<ContactResponse> {
  try {
    // Rate limiting
    const headersList = await headers();
    const ip = headersList.get("x-forwarded-for") ?? "unknown";

    const { success: notRateLimited } = await ratelimit.limit(ip);

    if (!notRateLimited) {
      return {
        success: false,
        message: "Too many requests. Please try again later.",
      };
    }

    // Parse form data
    const rawData = {
      name: formData.get("name"),
      email: formData.get("email"),
      subject: formData.get("subject"),
      message: formData.get("message"),
      website: formData.get("website"), // Honeypot
    };

    // Validate
    const validatedData = contactSchema.safeParse(rawData);

    if (!validatedData.success) {
      return {
        success: false,
        message: "Validation failed",
        errors: validatedData.error.flatten().fieldErrors,
      };
    }

    const { name, email, subject, message, website } = validatedData.data;

    // Honeypot check (bots fill this hidden field)
    if (website && website.length > 0) {
      // Silently reject but return success to confuse bots
      console.warn("Honeypot triggered:", { ip, name, email });
      return { success: true, message: "Message sent successfully!" };
    }

    // Send email via Resend
    const { error } = await resend.emails.send({
      from: "Portfolio Contact <contact@yourdomain.com>",
      to: process.env.CONTACT_EMAIL!,
      replyTo: email,
      subject: `[Portfolio] ${subject}`,
      text: `
Name: ${name}
Email: ${email}
Subject: ${subject}

Message:
${message}
      `,
      // Or use React Email template
      // react: ContactEmailTemplate({ name, email, subject, message }),
    });

    if (error) {
      console.error("Resend error:", error);
      return {
        success: false,
        message: "Failed to send message. Please try again.",
      };
    }

    // Optional: Send auto-reply
    await resend.emails.send({
      from: "John Doe <noreply@yourdomain.com>",
      to: email,
      subject: "Thanks for reaching out!",
      text: `Hi ${name},\n\nThanks for your message! I'll get back to you within 24-48 hours.\n\nBest,\nJohn`,
    });

    return {
      success: true,
      message: "Message sent successfully! I'll get back to you soon.",
    };
  } catch (error) {
    console.error("Contact form error:", error);
    return {
      success: false,
      message: "An unexpected error occurred. Please try again.",
    };
  }
}
```

## Contact Form Component

```tsx
// components/contact-form.tsx
"use client";

import { useState, useTransition } from "react";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { toast } from "sonner";
import { Loader2, Send, CheckCircle } from "lucide-react";
import { motion, AnimatePresence } from "framer-motion";
import { contactSchema, type ContactFormData } from "@/lib/schemas/contact";
import { submitContactForm } from "@/app/actions/contact";

/**
 * Production-ready contact form with validation, animations, and feedback.
 * @security Client + Server validation, honeypot, rate limiting.
 */
export function ContactForm() {
  const [isPending, startTransition] = useTransition();
  const [isSuccess, setIsSuccess] = useState(false);

  const {
    register,
    handleSubmit,
    reset,
    formState: { errors, isValid },
  } = useForm<ContactFormData>({
    resolver: zodResolver(contactSchema),
    mode: "onBlur", // Validate on blur for better UX
    defaultValues: {
      name: "",
      email: "",
      subject: "",
      message: "",
      website: "", // Honeypot
    },
  });

  const onSubmit = (data: ContactFormData) => {
    startTransition(async () => {
      const formData = new FormData();
      Object.entries(data).forEach(([key, value]) => {
        formData.append(key, value || "");
      });

      const response = await submitContactForm(formData);

      if (response.success) {
        setIsSuccess(true);
        reset();
        toast.success(response.message);
      } else {
        toast.error(response.message);

        // Show field-specific errors
        if (response.errors) {
          Object.entries(response.errors).forEach(([field, messages]) => {
            if (messages?.[0]) {
              toast.error(`${field}: ${messages[0]}`);
            }
          });
        }
      }
    });
  };

  if (isSuccess) {
    return (
      <motion.div
        initial={{ opacity: 0, scale: 0.95 }}
        animate={{ opacity: 1, scale: 1 }}
        className="flex flex-col items-center justify-center rounded-xl border border-green-500/20 bg-green-500/10 p-8 text-center"
      >
        <CheckCircle className="mb-4 h-12 w-12 text-green-500" />
        <h3 className="text-xl font-bold">Message Sent!</h3>
        <p className="mt-2 text-zinc-400">
          Thanks for reaching out. I'll get back to you soon.
        </p>
        <button
          onClick={() => setIsSuccess(false)}
          className="mt-6 text-sm text-brand-400 hover:underline"
        >
          Send another message
        </button>
      </motion.div>
    );
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
      {/* Honeypot - Hidden from real users */}
      <input
        {...register("website")}
        type="text"
        tabIndex={-1}
        autoComplete="off"
        className="absolute left-[-9999px] top-[-9999px]"
        aria-hidden="true"
      />

      {/* Name Field */}
      <div>
        <label htmlFor="name" className="mb-2 block text-sm font-medium">
          Name <span className="text-red-500">*</span>
        </label>
        <input
          {...register("name")}
          type="text"
          id="name"
          placeholder="John Doe"
          className={`
            w-full rounded-lg border bg-zinc-900/50 px-4 py-3
            text-white placeholder:text-zinc-500
            focus:outline-none focus:ring-2
            ${
              errors.name
                ? "border-red-500 focus:ring-red-500/50"
                : "border-zinc-800 focus:ring-brand-500/50"
            }
          `}
          disabled={isPending}
          aria-invalid={!!errors.name}
          aria-describedby={errors.name ? "name-error" : undefined}
        />
        <AnimatePresence>
          {errors.name && (
            <motion.p
              initial={{ opacity: 0, y: -10 }}
              animate={{ opacity: 1, y: 0 }}
              exit={{ opacity: 0, y: -10 }}
              id="name-error"
              className="mt-2 text-sm text-red-500"
              role="alert"
            >
              {errors.name.message}
            </motion.p>
          )}
        </AnimatePresence>
      </div>

      {/* Email Field */}
      <div>
        <label htmlFor="email" className="mb-2 block text-sm font-medium">
          Email <span className="text-red-500">*</span>
        </label>
        <input
          {...register("email")}
          type="email"
          id="email"
          placeholder="john@example.com"
          className={`
            w-full rounded-lg border bg-zinc-900/50 px-4 py-3
            text-white placeholder:text-zinc-500
            focus:outline-none focus:ring-2
            ${
              errors.email
                ? "border-red-500 focus:ring-red-500/50"
                : "border-zinc-800 focus:ring-brand-500/50"
            }
          `}
          disabled={isPending}
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? "email-error" : undefined}
        />
        <AnimatePresence>
          {errors.email && (
            <motion.p
              initial={{ opacity: 0, y: -10 }}
              animate={{ opacity: 1, y: 0 }}
              exit={{ opacity: 0, y: -10 }}
              id="email-error"
              className="mt-2 text-sm text-red-500"
              role="alert"
            >
              {errors.email.message}
            </motion.p>
          )}
        </AnimatePresence>
      </div>

      {/* Subject Field */}
      <div>
        <label htmlFor="subject" className="mb-2 block text-sm font-medium">
          Subject <span className="text-red-500">*</span>
        </label>
        <input
          {...register("subject")}
          type="text"
          id="subject"
          placeholder="Project inquiry"
          className={`
            w-full rounded-lg border bg-zinc-900/50 px-4 py-3
            text-white placeholder:text-zinc-500
            focus:outline-none focus:ring-2
            ${
              errors.subject
                ? "border-red-500 focus:ring-red-500/50"
                : "border-zinc-800 focus:ring-brand-500/50"
            }
          `}
          disabled={isPending}
          aria-invalid={!!errors.subject}
        />
        <AnimatePresence>
          {errors.subject && (
            <motion.p
              initial={{ opacity: 0, y: -10 }}
              animate={{ opacity: 1, y: 0 }}
              exit={{ opacity: 0, y: -10 }}
              className="mt-2 text-sm text-red-500"
              role="alert"
            >
              {errors.subject.message}
            </motion.p>
          )}
        </AnimatePresence>
      </div>

      {/* Message Field */}
      <div>
        <label htmlFor="message" className="mb-2 block text-sm font-medium">
          Message <span className="text-red-500">*</span>
        </label>
        <textarea
          {...register("message")}
          id="message"
          rows={5}
          placeholder="Tell me about your project..."
          className={`
            w-full resize-none rounded-lg border bg-zinc-900/50 px-4 py-3
            text-white placeholder:text-zinc-500
            focus:outline-none focus:ring-2
            ${
              errors.message
                ? "border-red-500 focus:ring-red-500/50"
                : "border-zinc-800 focus:ring-brand-500/50"
            }
          `}
          disabled={isPending}
          aria-invalid={!!errors.message}
        />
        <AnimatePresence>
          {errors.message && (
            <motion.p
              initial={{ opacity: 0, y: -10 }}
              animate={{ opacity: 1, y: 0 }}
              exit={{ opacity: 0, y: -10 }}
              className="mt-2 text-sm text-red-500"
              role="alert"
            >
              {errors.message.message}
            </motion.p>
          )}
        </AnimatePresence>
      </div>

      {/* Submit Button */}
      <button
        type="submit"
        disabled={isPending}
        className={`
          group flex w-full items-center justify-center gap-2
          rounded-lg bg-brand-500 px-6 py-3 font-medium text-white
          transition-all hover:bg-brand-600
          disabled:cursor-not-allowed disabled:opacity-50
        `}
      >
        {isPending ? (
          <>
            <Loader2 className="h-5 w-5 animate-spin" />
            Sending...
          </>
        ) : (
          <>
            Send Message
            <Send className="h-4 w-4 transition-transform group-hover:translate-x-1" />
          </>
        )}
      </button>
    </form>
  );
}
```

## Toast Provider Setup

```tsx
// app/layout.tsx
import { Toaster } from "sonner";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}
        <Toaster
          position="bottom-right"
          toastOptions={{
            style: {
              background: "#18181b",
              border: "1px solid #27272a",
              color: "#fafafa",
            },
          }}
        />
      </body>
    </html>
  );
}
```

## Contact Section

```tsx
// components/sections/contact-section.tsx
import { Mail, MapPin, Phone } from "lucide-react";
import { ContactForm } from "@/components/contact-form";

export function ContactSection() {
  return (
    <section id="contact" className="py-24">
      <div className="container">
        <div className="mx-auto max-w-4xl">
          {/* Header */}
          <div className="mb-12 text-center">
            <h2 className="font-heading text-3xl font-bold sm:text-4xl">
              Get In Touch
            </h2>
            <p className="mt-4 text-lg text-zinc-400">
              Have a project in mind? Let's talk about it.
            </p>
          </div>

          <div className="grid gap-12 lg:grid-cols-[1fr_1.5fr]">
            {/* Contact Info */}
            <div className="space-y-8">
              <div className="flex items-start gap-4">
                <div className="flex h-12 w-12 shrink-0 items-center justify-center rounded-lg bg-brand-500/10 text-brand-400">
                  <Mail className="h-5 w-5" />
                </div>
                <div>
                  <h3 className="font-medium">Email</h3>
                  <a
                    href="mailto:hello@johndoe.com"
                    className="text-zinc-400 hover:text-brand-400"
                  >
                    hello@johndoe.com
                  </a>
                </div>
              </div>

              <div className="flex items-start gap-4">
                <div className="flex h-12 w-12 shrink-0 items-center justify-center rounded-lg bg-brand-500/10 text-brand-400">
                  <MapPin className="h-5 w-5" />
                </div>
                <div>
                  <h3 className="font-medium">Location</h3>
                  <p className="text-zinc-400">San Francisco, CA</p>
                </div>
              </div>

              <div className="flex items-start gap-4">
                <div className="flex h-12 w-12 shrink-0 items-center justify-center rounded-lg bg-brand-500/10 text-brand-400">
                  <Phone className="h-5 w-5" />
                </div>
                <div>
                  <h3 className="font-medium">Phone</h3>
                  <a
                    href="tel:+1234567890"
                    className="text-zinc-400 hover:text-brand-400"
                  >
                    +1 (234) 567-890
                  </a>
                </div>
              </div>

              {/* Availability Badge */}
              <div className="rounded-lg border border-green-500/20 bg-green-500/10 p-4">
                <div className="flex items-center gap-2">
                  <span className="relative flex h-3 w-3">
                    <span className="absolute inline-flex h-full w-full animate-ping rounded-full bg-green-400 opacity-75" />
                    <span className="relative inline-flex h-3 w-3 rounded-full bg-green-500" />
                  </span>
                  <span className="text-sm font-medium text-green-400">
                    Available for new projects
                  </span>
                </div>
              </div>
            </div>

            {/* Contact Form */}
            <div className="rounded-xl border border-zinc-800 bg-zinc-900/50 p-6 sm:p-8">
              <ContactForm />
            </div>
          </div>
        </div>
      </div>
    </section>
  );
}
```

## Email Templates with React Email

```bash
npm install @react-email/components
```

```tsx
// emails/contact-notification.tsx
import {
  Body,
  Container,
  Head,
  Heading,
  Hr,
  Html,
  Preview,
  Section,
  Text,
} from "@react-email/components";

interface ContactEmailProps {
  name: string;
  email: string;
  subject: string;
  message: string;
}

export function ContactNotificationEmail({
  name,
  email,
  subject,
  message,
}: ContactEmailProps) {
  return (
    <Html>
      <Head />
      <Preview>New contact form submission from {name}</Preview>
      <Body style={main}>
        <Container style={container}>
          <Heading style={h1}>New Contact Form Submission</Heading>

          <Section style={section}>
            <Text style={label}>From:</Text>
            <Text style={value}>
              {name} ({email})
            </Text>
          </Section>

          <Section style={section}>
            <Text style={label}>Subject:</Text>
            <Text style={value}>{subject}</Text>
          </Section>

          <Hr style={hr} />

          <Section style={section}>
            <Text style={label}>Message:</Text>
            <Text style={messageText}>{message}</Text>
          </Section>

          <Hr style={hr} />

          <Text style={footer}>
            Reply directly to this email to respond to {name}.
          </Text>
        </Container>
      </Body>
    </Html>
  );
}

// Styles
const main = {
  backgroundColor: "#09090b",
  fontFamily:
    '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
};

const container = {
  backgroundColor: "#18181b",
  margin: "0 auto",
  padding: "20px",
  borderRadius: "8px",
  border: "1px solid #27272a",
};

const h1 = {
  color: "#fafafa",
  fontSize: "24px",
  fontWeight: "bold",
  margin: "0 0 20px",
};

const section = {
  marginBottom: "16px",
};

const label = {
  color: "#a1a1aa",
  fontSize: "12px",
  textTransform: "uppercase" as const,
  margin: "0 0 4px",
};

const value = {
  color: "#fafafa",
  fontSize: "16px",
  margin: "0",
};

const messageText = {
  color: "#fafafa",
  fontSize: "14px",
  lineHeight: "1.6",
  whiteSpace: "pre-wrap" as const,
};

const hr = {
  borderColor: "#27272a",
  margin: "20px 0",
};

const footer = {
  color: "#71717a",
  fontSize: "12px",
  margin: "0",
};

export default ContactNotificationEmail;
```

## Environment Variables

```env
# .env.local
RESEND_API_KEY=re_xxxxxxxxxxxxx
CONTACT_EMAIL=your@email.com

# Upstash Redis (for rate limiting)
UPSTASH_REDIS_REST_URL=https://xxxxx.upstash.io
UPSTASH_REDIS_REST_TOKEN=xxxxxxxxxxxxx
```

## Testing Checklist

- [ ] Form validates on blur
- [ ] Server validation catches bypassed client validation
- [ ] Honeypot rejects bots silently
- [ ] Rate limiting works (test with multiple submissions)
- [ ] Email sends correctly
- [ ] Auto-reply sends (if implemented)
- [ ] Success state displays properly
- [ ] Error messages are clear and actionable
- [ ] Form is accessible (labels, ARIA, keyboard nav)
- [ ] Loading state prevents double submission
