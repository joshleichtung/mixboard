---
name: nextjs-patterns
description: "Next.js 14+ implementation patterns. Activates for: App Router setup, Server Actions, Route Handlers, middleware configuration, ISR/SSR/SSG selection, parallel routes, intercepting routes, loading/error states. Does NOT activate for: React-only components, generic TypeScript, or basic HTML/CSS."
category: dev
---

# Next.js 14+ Implementation Patterns

Use these patterns when implementing Next.js features in Josh's projects.

## App Router Fundamentals

### File Conventions
- `page.tsx` - Route UI
- `layout.tsx` - Shared UI (preserved on navigation)
- `loading.tsx` - Suspense loading UI
- `error.tsx` - Error boundary
- `not-found.tsx` - 404 UI
- `route.ts` - API endpoint

### Data Fetching Priority
1. **Server Components** (default) - fetch directly in component
2. **Server Actions** - mutations, form submissions
3. **Route Handlers** - external API consumption, webhooks

## State Management Rules

**CRITICAL: Separate client state from server state**

| State Type | Tool | Use For |
|------------|------|---------|
| **Server State** | TanStack Query | API responses, remote data, cache |
| **Client State** | Zustand | UI state, forms, local preferences |

**Never put API responses in Zustand.** Use TanStack Query for:
- Data fetching and caching
- Automatic background refetching
- Optimistic updates
- Infinite scroll / pagination

```typescript
// Correct: TanStack Query for server state
const { data, isLoading } = useQuery({
  queryKey: ['items'],
  queryFn: () => fetch('/api/items').then(r => r.json())
})

// Correct: Zustand for client state
const { sidebarOpen, toggleSidebar } = useUIStore()

// WRONG: Don't do this
const { items, setItems } = useStore() // API data in Zustand
```

## Server Actions Pattern

```typescript
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

export async function createItem(formData: FormData) {
  const data = Object.fromEntries(formData)
  // validate with zod
  // persist to db
  revalidatePath('/items')
  redirect('/items')
}
```

## Route Handlers

```typescript
// app/api/items/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams
  return NextResponse.json({ items: [] })
}

export async function POST(request: NextRequest) {
  const body = await request.json()
  return NextResponse.json({ id: 'new-id' }, { status: 201 })
}
```

## Middleware Pattern

```typescript
// middleware.ts (root level)
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  return NextResponse.next()
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*']
}
```

## Rendering Strategy Selection

| Strategy | Use When |
|----------|----------|
| **SSG** (`generateStaticParams`) | Content rarely changes |
| **ISR** (`revalidate: 3600`) | Periodic updates ok |
| **SSR** (`dynamic = 'force-dynamic'`) | Personalized, real-time |
| **Client** (`'use client'`) | Interactivity, browser APIs |

## Parallel Routes

```
app/
  @modal/
    (.)photo/[id]/page.tsx  # Intercepted route
  layout.tsx  # Receives { modal, children }
```

## Error Handling

```typescript
'use client'
export default function Error({ error, reset }: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return <button onClick={reset}>Try again</button>
}
```

## Josh's Stack Integration

- **TanStack Query** for server/API state
- **Zustand** for client/UI state only
- **Tailwind** for styling
- **TypeScript strict** always
- **Zod** for runtime validation
