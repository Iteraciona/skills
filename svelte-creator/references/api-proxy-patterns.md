# API Proxy & Data Loading Patterns

This document contains the template code for the api-proxy architecture, the request interceptor (`hooks.server.ts`), and data loading patterns (`+page.ts`, `+server.ts`).

## Architecture Overview

```
Browser → +page.svelte → +page.ts → api-proxy (+server.ts) → hooks interceptor → Backend API
                                         ↑
                              src/routes/api/<module>/<action>/+server.ts
```

The browser never talks directly to the backend. Every request goes:
1. `+page.ts` calls the **api-proxy** (e.g., `/api/users/list`)
2. The api-proxy `+server.ts` calls the **backend** using `fetch()`
3. `hooks.server.ts` intercepts every `fetch()` call to inject auth token + client IP headers
4. The response flows back to `+page.ts` → `+page.svelte`

---

## hooks.server.ts — Request Interceptor

This is the most important file in the architecture. It runs on every request and does three things:
1. Extracts the client IP from proxy headers
2. Overrides `event.fetch` to inject auth headers on every outbound request
3. Handles 401 responses (clears cookies, returns error)

```typescript
import type { Handle } from '@sveltejs/kit';

const protectedRoutes = ['/dashboard', '/users', '/settings', '/admin'];
const publicRoutes = ['/login', '/register'];

export const handle: Handle = async ({ event, resolve }) => {
    const start = Date.now();
    const path = event.url.pathname;
    const method = event.request.method;

    console.log(`[${new Date().toISOString()}] → ${method} ${path}`);

    // --- Extract client IP ---
    const forwardedFor = event.request.headers.get('x-forwarded-for');
    const ip = forwardedFor
        ? forwardedFor.split(',')[0].trim()
        : event.getClientAddress?.() || '';
    event.locals.clientIp = ip;

    // --- Skip auth for public routes ---
    if (publicRoutes.some(route => path.startsWith(route))) {
        return resolve(event);
    }

    // --- Check authentication ---
    const authToken = event.cookies.get('authToken');
    let isAuthenticated = false;

    if (authToken) {
        try {
            // Replace with your token verification logic
            // const decoded = await verifyToken(authToken);
            isAuthenticated = true;
            event.locals.authToken = authToken;
            // event.locals.user = decoded;
        } catch (err) {
            console.error('[hooks.server.ts] Invalid token:', err.message);
            event.cookies.delete('authToken', { path: '/' });
        }
    }

    // --- Redirect unauthenticated users from protected routes ---
    if (protectedRoutes.some(route => path.startsWith(route)) && !isAuthenticated) {
        event.cookies.delete('authToken', { path: '/' });
        console.log(`[hooks.server.ts] Access denied to ${path}. Redirecting to /login`);
    }

    // --- Override fetch to inject headers ---
    const originalFetch = event.fetch;
    event.fetch = async (input, init = {}) => {
        const headers = new Headers(init.headers || {});

        // Inject auth token
        if (authToken) {
            headers.set('Authorization', `Bearer ${authToken}`);
        }

        // Inject client IP
        if (event.locals.clientIp) {
            headers.set('X-Client-IP', event.locals.clientIp);
            headers.set('X-Forwarded-For', event.locals.clientIp);
            headers.set('X-Real-IP', event.locals.clientIp);
        }

        const response = await originalFetch(input, { ...init, headers });

        // Handle 401 — clear auth and return error
        if (response.status === 401) {
            console.log("[hooks.server.ts] Unauthorized:", "Forbidden");
            event.cookies.delete('authToken', { path: '/' });

            return new Response(
                JSON.stringify({ message: "User not allowed", success: false }),
                { status: response.status, headers: { 'Content-Type': 'application/json' } }
            );
        }

        return response;
    };

    const response = await resolve(event);
    const duration = Date.now() - start;

    console.log(`[${new Date().toISOString()}] ← ${response.status} ${method} ${path} [${duration}ms] ${isAuthenticated ? '🔐' : '🔓'}`);

    return response;
};
```

### app.d.ts — Type declarations for locals

```typescript
declare global {
    namespace App {
        interface Locals {
            clientIp: string;
            authToken?: string;
            user?: Record<string, any>;
        }
    }
}

export {};
```

---

## +server.ts — API Proxy Endpoint

Each api-proxy endpoint lives in `src/routes/api/<module>/<action>/+server.ts`.

### Pattern: POST proxy (receive JSON body, forward to backend)

```typescript
import type { RequestHandler } from './$types';

export const POST: RequestHandler = async ({ fetch, request }) => {
    const body = await request.json();
    const params = new URLSearchParams(body);

    const endpoint = `${process.env.API_BASE_URL}/v1/users/list?${params.toString()}`;

    const response = await fetch(endpoint, {
        method: 'GET',
        headers: { 'Content-Type': 'application/json' }
    });

    const data = await response.text();
    return new Response(data, {
        status: response.status,
        headers: { 'Content-Type': 'application/json' }
    });
};
```

### Pattern: GET proxy (forward query params)

```typescript
import type { RequestHandler } from './$types';

export const GET: RequestHandler = async ({ fetch, url }) => {
    const page = url.searchParams.get('page') ?? '1';

    const response = await fetch(`${process.env.API_BASE_URL}/v1/products?page=${page}`, {
        method: 'GET',
        headers: { 'Content-Type': 'application/json' }
    });

    const data = await response.text();
    return new Response(data, {
        status: response.status,
        headers: { 'Content-Type': 'application/json' }
    });
};
```

### Pattern: POST proxy with JSON body passthrough

```typescript
import type { RequestHandler } from './$types';

export const POST: RequestHandler = async ({ fetch, request }) => {
    const body = await request.json();

    const response = await fetch(`${process.env.API_BASE_URL}/v1/users/create`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(body)
    });

    const data = await response.text();
    return new Response(data, {
        status: response.status,
        headers: { 'Content-Type': 'application/json' }
    });
};
```

> Remember: `fetch()` inside `+server.ts` is the overridden version from `hooks.server.ts`. It automatically includes the Authorization header and client IP headers. You don't need to add them manually.

---

## +page.ts — Data Loading

All data loading happens in `+page.ts`. It calls the **api-proxy**, never the backend directly.

### Pattern: Load data with pagination

```typescript
import type { PageLoad } from "./$types";
import { redirect } from '@sveltejs/kit';

export const load: PageLoad = async ({ fetch, url }) => {
    const page = url.searchParams.get("page") ?? "1";
    const query = url.searchParams.get("q") ?? "";

    const res = await fetch('/api/users/list', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ page, query })
    });

    if (res.status === 401) {
        throw redirect(302, '/login');
    }

    const json = await res.json();
    const { docs, ...pagination } = json.data;

    return {
        users: docs,
        pagination
    };
};
```

### Pattern: Load single item by ID

```typescript
import type { PageLoad } from "./$types";
import { redirect, error } from '@sveltejs/kit';

export const load: PageLoad = async ({ fetch, params }) => {
    const res = await fetch(`/api/users/${params.id}`, {
        method: 'GET',
        headers: { 'Content-Type': 'application/json' }
    });

    if (res.status === 401) {
        throw redirect(302, '/login');
    }

    if (res.status === 404) {
        throw error(404, 'User not found');
    }

    const json = await res.json();
    return { user: json.data };
};
```

---

## +page.svelte — Presentation Only

Keep `.svelte` files clean. They receive data from `+page.ts` and render it, using components from `$lib/components/`.

```svelte
<script lang="ts">
    import type { PageData } from "./$types";
    import UserTable from "$lib/components/admin/users/UserTable.svelte";
    import WebPagination from "$lib/components/utils/Pagination/WebPagination.svelte";

    let { data }: { data: PageData } = $props();
</script>

<div class="p-4">
    <h1 class="section-title">Users</h1>
    <UserTable users={data.users} />
    <WebPagination pagination={data.pagination} />
</div>
```

No `fetch()` calls. No business logic. No data transformation. Just imports, props, and markup.

---

## +layout.ts — Layout Data Loading

For layouts that need data (e.g., loading user info for the admin sidebar):

```typescript
import type { LayoutLoad } from './$types';

export const load: LayoutLoad = async ({ fetch, data }) => {
    if (data.logged) {
        const res = await fetch('/api/users/me', {
            method: 'GET',
            headers: { 'Content-Type': 'application/json' }
        });

        const json = await res.json();
        return { user: json.data };
    }
};
```
