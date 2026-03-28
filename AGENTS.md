# Agent Guidelines for UIGen

AI-powered React component generator with live preview built on Next.js 15, React 19, and TypeScript.

## Build/Lint/Test Commands

```bash
# Development
npm run dev                    # Start dev server with turbopack on localhost:3000
npm run dev:daemon             # Start dev server in background, logs to logs.txt

# Build & Production
npm run build                  # Production build
npm run start                  # Start production server

# Code Quality
npm run lint                   # Run ESLint (extends Next.js config)

# Testing (Vitest)
npm run test                   # Run all tests
npm run test -- file.ts        # Run single test file
npm run test -- --run          # Run tests once (no watch mode)
npm run test -- --run FileTree # Run tests matching "FileTree"
npm run test -- --ui           # Run tests with UI browser
npm run test -- src/lib/__tests__/file-system.test.ts  # Run specific test file

# Database
npm run setup                  # Install deps + prisma generate + prisma migrate dev
npm run db:reset               # Reset database (npx prisma migrate reset --force)
```

## Code Style Guidelines

### TypeScript
- **Strict mode enabled** - no implicit any, strict null checks
- Use explicit types for function parameters and return types
- Prefer interfaces over type aliases for object shapes
- Use `type` for unions, intersections, and utility types

```typescript
// Good
interface UserData {
  id: string;
  name: string;
}
function getUser(id: string): Promise<UserData>

// Avoid
function getUser(id) { ... }  // Missing types
```

### Imports
- Use path aliases (`@/`) for all internal imports
- Group imports: 1) React, 2) external libs, 3) internal imports, 4) types
- Use `type` modifier for type-only imports when appropriate

```typescript
import { useState } from "react";
import { clsx, type ClassValue } from "clsx";
import { Button } from "@/components/ui/button";
import type { User } from "@/types";
```

### Naming Conventions
- **Components**: PascalCase (`ChatInterface`, `MessageInput`)
- **Functions/Hooks**: camelCase (`useChat`, `handleSubmit`)
- **Files**: kebab-case for non-component files (`file-system.ts`, `chat-context.tsx`)
- **React Components**: .tsx extension, can have co-located tests (`Component.test.tsx`)
- **TypeScript utilities**: .ts extension (`utils.ts`, `auth.ts`)
- **Constants**: UPPER_SNAKE_CASE for true constants, camelCase for exported hooks/utilities

### File Structure
```
src/
├── actions/          # Server actions (use server)
├── app/              # Next.js App Router pages
│   ├── api/          # API routes
│   └── [projectId]/  # Dynamic routes
├── components/
│   ├── ui/           # shadcn/ui components
│   ├── chat/         # Chat-related components
│   ├── editor/       # Code editor components
│   └── auth/         # Authentication components
├── hooks/            # Custom React hooks
├── lib/              # Utilities and core logic
│   ├── contexts/     # React contexts
│   ├── prompts/      # AI prompts
│   ├── tools/        # AI tool implementations
│   └── transform/    # Code transformation utilities
└── generated/        # Auto-generated code (Prisma)
```

### React Patterns
- Use `"use client"` directive for client components
- Use named exports for components (`export function Component()`)
- Use `use client` hook pattern for context consumers
- Destructure props with explicit typing

```typescript
"use client";

interface Props {
  title: string;
  onClick: () => void;
}

export function MyComponent({ title, onClick }: Props) {
  return <button onClick={onClick}>{title}</button>;
}
```

### Error Handling
- Server actions throw descriptive errors: `throw new Error("Unauthorized")`
- API routes use try/catch with console.error for logging
- Client-side error boundaries where appropriate
- Validate inputs at boundaries, return null/false for invalid operations

```typescript
// Server action
export async function createProject(input: Input) {
  const session = await getSession();
  if (!session) {
    throw new Error("Unauthorized");
  }
  return prisma.project.create({ data: {...} });
}

// Library functions return null/false for failures
readFile(path: string): string | null {
  const file = this.files.get(path);
  if (!file || file.type !== "file") return null;
  return file.content;
}
```

### Tailwind CSS
- Use `@/` path alias for CSS imports
- Use Tailwind v4 with `@import "tailwindcss"`
- Use `cn()` utility for conditional classes (`@/lib/utils`)
- shadcn/ui components follow new-york style with CSS variables

```tsx
import { cn } from "@/lib/utils";

<div className={cn(
  "base-class",
  isActive && "active-class",
  className
)} />
```

### Testing Patterns (Vitest)
- Place tests in `__tests__/` folder next to source files
- Use `@testing-library/react` for component tests
- Use `@testing-library/user-event` for user interactions
- Mock functions with `vi.fn()`
- Clean up with `afterEach` and `cleanup()`

```typescript
import { test, expect, vi } from "vitest";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

test("renders component", () => {
  render(<MyComponent />);
  expect(screen.getByRole("button")).toBeDefined();
});
```

### Database (Prisma)
- **Schema definition**: `src/generated/prisma/schema.prisma` - Reference this file anytime you need to understand the structure of data stored in the database
- Prisma schema uses SQLite (`file:./dev.db`)
- Generated client output: `src/generated/prisma`
- Run `npx prisma generate` after schema changes
- Use `@` path alias in Prisma imports

```typescript
import { prisma } from "@/lib/prisma";
```

## Key Libraries & Patterns

- **AI SDK**: Use `streamText` from `ai` package, tools from `@/lib/tools/`
- **Authentication**: JWT via `jose`, session via `@/lib/auth`
- **Virtual File System**: `VirtualFileSystem` class in `@/lib/file-system`
- **Code Editor**: Monaco Editor via `@monaco-editor/react`
- **UI Components**: shadcn/ui with Radix UI primitives

## Common Tasks

### Adding a new shadcn/ui component
```bash
npx shadcn@latest add button
```

### Adding a new test
Create `ComponentName.test.tsx` in `__tests__/` folder next to component.

### Modifying Prisma schema
1. Edit `prisma/schema.prisma`
2. Run `npx prisma migrate dev --name description`
3. Regenerate client: `npx prisma generate`

## Environment Variables

- `ANTHROPIC_API_KEY` - Optional Claude API key (works without for static responses)
- `DATABASE_URL` - SQLite database path (default: `file:./prisma/dev.db`)
- `JWT_SECRET` - Secret for JWT token signing
