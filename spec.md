# Modern Full-Stack Web Application Specification

A comprehensive reference for building production-ready web applications using TanStack Start, Convex, Better Auth, and shadcn/ui deployed to Cloudflare Workers.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Architecture Overview](#architecture-overview)
3. [Project Structure](#project-structure)
4. [TanStack Start Setup](#tanstack-start-setup)
5. [Convex Backend](#convex-backend)
6. [Better Auth Integration](#better-auth-integration)
7. [OAuth Provider Configuration](#oauth-provider-configuration)
8. [File Storage](#file-storage)
9. [UI Components](#ui-components)
10. [Cloudflare Workers Deployment](#cloudflare-workers-deployment)
11. [Environment Variables](#environment-variables)
12. [Development Workflow](#development-workflow)
13. [Complete Wiring Guide](#complete-wiring-guide)

---

## Quick Start

Bootstrap a new project with shadcn/ui, TanStack Start, Base UI, and Tailwind v4:

```bash
bunx --bun shadcn@latest create \
  --preset "https://ui.shadcn.com/init?base=base&style=default&baseColor=neutral&iconLibrary=lucide&template=start" \
  --template start
```

This creates a fully configured TanStack Start project with:
- **Base UI** as the component primitive library
- **Tailwind CSS v4** with CSS-first configuration
- **Lucide** icons
- **Inter** font (via Fontsource)

After creation:

```bash
cd my-app
bun install
npx convex dev --once    # Initialize Convex
bun run dev              # Start development server
```

> **Customize the preset**: Visit [ui.shadcn.com](https://ui.shadcn.com) and configure your preferences, then copy the preset URL.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Cloudflare Workers                        │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    TanStack Start (SSR)                    │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐   │  │
│  │  │   Routes    │  │  Components │  │   Auth Client   │   │  │
│  │  │ (React 19)  │  │ (shadcn/ui) │  │  (Better Auth)  │   │  │
│  │  └──────┬──────┘  └──────┬──────┘  └────────┬────────┘   │  │
│  │         │                │                   │            │  │
│  │         └────────────────┼───────────────────┘            │  │
│  │                          │                                │  │
│  │                 ┌────────▼────────┐                       │  │
│  │                 │  Convex Client  │                       │  │
│  │                 │ (React Hooks)   │                       │  │
│  │                 └────────┬────────┘                       │  │
│  └──────────────────────────┼────────────────────────────────┘  │
└─────────────────────────────┼───────────────────────────────────┘
                              │ HTTPS
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         Convex Cloud                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │    Functions    │  │    Database     │  │  File Storage   │  │
│  │ (mutations,     │  │  (Documents)    │  │  (Blob Store)   │  │
│  │  queries,       │  │                 │  │                 │  │
│  │  actions)       │  │                 │  │                 │  │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘  │
│           │                    │                     │           │
│           └────────────────────┼─────────────────────┘           │
│                                │                                 │
│                    ┌───────────▼───────────┐                     │
│                    │     Better Auth       │                     │
│                    │   (Session Storage)   │                     │
│                    └───────────────────────┘                     │
└─────────────────────────────────────────────────────────────────┘
```

### Core Technologies

| Layer | Technology | Purpose |
|-------|------------|---------|
| Framework | TanStack Start | Full-stack React with SSR |
| Routing | TanStack Router | File-based routing with type safety |
| Backend | Convex | Serverless functions, database, file storage |
| Auth | Better Auth + Convex | OAuth/SSO with session persistence |
| UI | shadcn/ui + Base UI | Accessible, customizable components |
| Styling | Tailwind CSS v4 | Utility-first CSS (CSS-first config) |
| Deployment | Cloudflare Workers | Edge-first serverless hosting |

---

## Project Structure

```
project-root/
├── app/                                  # Application root
│   ├── src/                              # Frontend application
│   │   ├── components/                   # React components
│   │   │   ├── ui/                       # shadcn/ui components
│   │   │   │   ├── button.tsx
│   │   │   │   ├── card.tsx
│   │   │   │   ├── input.tsx
│   │   │   │   ├── select.tsx
│   │   │   │   ├── dropdown-menu.tsx
│   │   │   │   └── ...
│   │   │   ├── app-layout.tsx            # Main layout wrapper
│   │   │   └── [feature-components].tsx
│   │   │
│   │   ├── lib/                          # Shared utilities
│   │   │   ├── auth-client.ts            # Better Auth client
│   │   │   ├── auth-server.ts            # Better Auth server handler
│   │   │   └── utils.ts                  # Helper functions (cn, etc.)
│   │   │
│   │   ├── routes/                       # File-based routes
│   │   │   ├── __root.tsx                # Root layout with providers
│   │   │   ├── index.tsx                 # Home page (/)
│   │   │   ├── login.tsx                 # Auth page (/login)
│   │   │   ├── api/
│   │   │   │   └── auth/
│   │   │   │       └── $.tsx             # Auth catch-all handler
│   │   │   └── [page].tsx                # Additional pages
│   │   │
│   │   ├── router.tsx                    # Router configuration
│   │   ├── styles.css                    # Global styles + Tailwind
│   │   └── routeTree.gen.ts              # Auto-generated route tree
│   │
│   ├── convex/                           # Backend (Convex)
│   │   ├── _generated/                   # Auto-generated types
│   │   │   ├── api.d.ts
│   │   │   ├── api.js
│   │   │   ├── dataModel.d.ts
│   │   │   └── server.d.ts
│   │   │
│   │   ├── convex.config.ts              # App configuration + plugins
│   │   ├── schema.ts                     # Database schema
│   │   ├── auth.ts                       # Better Auth setup
│   │   ├── auth.config.ts                # Auth configuration
│   │   ├── http.ts                       # HTTP route handlers
│   │   └── [feature].ts                  # Feature-specific functions
│   │
│   ├── public/                           # Static assets
│   │
│   ├── .env.local                        # Development environment
│   ├── .env.production                   # Production environment
│   ├── vite.config.ts                    # Vite + TanStack Start config
│   ├── wrangler.toml                     # Cloudflare Workers config
│   ├── tsconfig.json                     # TypeScript config
│   ├── components.json                   # shadcn/ui config
│   └── package.json                      # Dependencies
```

---

## TanStack Start Setup

TanStack Start (v1.121.0+) is configured as a Vite plugin—no separate `app.config.ts` needed. See [Vite Configuration](#vite-configuration) for the full setup.

### Router Configuration (SSR Auth)

The router is configured with `routerWithQueryClient` for SSR integration and `ConvexQueryClient` with `expectAuth: true` for authenticated server-side queries.

**`app/src/router.tsx`**
```typescript
import { createRouter } from "@tanstack/react-router";
import { QueryClient } from "@tanstack/react-query";
import { routerWithQueryClient } from "@tanstack/react-router-with-query";
import { ConvexQueryClient } from "@convex-dev/react-query";
import { ConvexProvider } from "convex/react";
import { routeTree } from "./routeTree.gen";

export function getRouter() {
  const convexUrl = import.meta.env.VITE_CONVEX_URL!;
  if (!convexUrl) {
    throw new Error("VITE_CONVEX_URL is not set");
  }

  // Create ConvexQueryClient with expectAuth: true for SSR auth support
  const convexQueryClient = new ConvexQueryClient(convexUrl, {
    expectAuth: true,
  });

  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        queryKeyHashFn: convexQueryClient.hashFn(),
        queryFn: convexQueryClient.queryFn(),
      },
    },
  });

  convexQueryClient.connect(queryClient);

  const router = routerWithQueryClient(
    createRouter({
      routeTree,
      defaultPreload: "intent",
      context: { queryClient, convexQueryClient },
      scrollRestoration: true,
      Wrap: ({ children }) => (
        <ConvexProvider client={convexQueryClient.convexClient}>
          {children}
        </ConvexProvider>
      ),
    }),
    queryClient,
  );

  return router;
}

declare module "@tanstack/react-router" {
  interface Register {
    router: ReturnType<typeof getRouter>;
  }
}
```

### Root Layout with SSR Auth

The root route uses `createRootRouteWithContext` and `beforeLoad` to fetch the auth token during server-side rendering, enabling authenticated Convex queries before the page reaches the browser.

**`app/src/routes/__root.tsx`**
```typescript
/// <reference types="vite/client" />
import {
  HeadContent,
  Outlet,
  Scripts,
  createRootRouteWithContext,
  useRouteContext,
} from "@tanstack/react-router";
import { ConvexBetterAuthProvider } from "@convex-dev/better-auth/react";
import { createServerFn } from "@tanstack/react-start";
import type { ConvexQueryClient } from "@convex-dev/react-query";
import type { QueryClient } from "@tanstack/react-query";
import appCss from "../styles.css?url";
import { authClient } from "../lib/auth-client";
import { getToken } from "../lib/auth-server";

// Server function to get auth token for SSR
const getAuth = createServerFn({ method: "GET" }).handler(async () => {
  return await getToken();
});

export const Route = createRootRouteWithContext<{
  queryClient: QueryClient;
  convexQueryClient: ConvexQueryClient;
}>()({
  head: () => ({
    meta: [
      { charSet: "utf-8" },
      { name: "viewport", content: "width=device-width, initial-scale=1" },
      { title: "App Title" },
    ],
    links: [{ rel: "stylesheet", href: appCss }],
  }),

  beforeLoad: async (ctx) => {
    const token = await getAuth();

    // During SSR only, set the auth token for authenticated queries
    if (token) {
      ctx.context.convexQueryClient.serverHttpClient?.setAuth(token);
    }

    return {
      isAuthenticated: !!token,
      token,
    };
  },

  component: RootComponent,
});

function RootComponent() {
  const context = useRouteContext({ from: Route.id });

  return (
    <ConvexBetterAuthProvider
      client={context.convexQueryClient.convexClient}
      authClient={authClient}
      initialToken={context.token}
    >
      <RootDocument>
        <Outlet />
      </RootDocument>
    </ConvexBetterAuthProvider>
  );
}

function RootDocument({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <head>
        <HeadContent />
      </head>
      <body>
        {children}
        <Scripts />
      </body>
    </html>
  );
}
```

> [!WARNING]
> **Sign-Out with `expectAuth: true`**
> 
> When using `expectAuth: true`, signing out requires a page reload to prevent errors from queries running before auth is ready on re-login:
> ```typescript
> await authClient.signOut({
>   fetchOptions: {
>     onSuccess: () => location.reload(),
>   },
> });
> ```

### Server Middleware for Protected Routes

Following [Better Auth TanStack Start recommendations](https://www.better-auth.com/docs/integrations/tanstack#middleware), use server middleware to protect routes.

**`app/src/lib/middleware.ts`**
```typescript
import { redirect } from "@tanstack/react-router";
import { createMiddleware } from "@tanstack/react-start";
import { getToken } from "./auth-server";

/**
 * Server middleware that protects routes requiring authentication.
 * Apply to any route via `server: { middleware: [authMiddleware] }`.
 */
export const authMiddleware = createMiddleware().server(async ({ next }) => {
  const token = await getToken();

  if (!token) {
    throw redirect({ to: "/login" });
  }

  return await next();
});
```

### Route Examples

**`app/src/routes/index.tsx`** (Protected Home Page)
```typescript
import { createFileRoute } from "@tanstack/react-router";
import { useSession } from "../lib/auth-client";
import { authMiddleware } from "../lib/middleware";

export const Route = createFileRoute("/")({
  server: {
    middleware: [authMiddleware],
  },
  component: HomePage,
});

function HomePage() {
  const { data: session, isPending } = useSession();

  if (isPending || !session?.user) {
    return null;
  }

  return (
    <div>
      <h1>Welcome, {session.user.email}</h1>
    </div>
  );
}
```

**`app/src/routes/login.tsx`** (Login Page)
```typescript
import { createFileRoute, useNavigate } from "@tanstack/react-router";
import { useSession, signIn } from "../lib/auth-client";
import { useEffect } from "react";
import { Button } from "../components/ui/button";

export const Route = createFileRoute("/login")({
  component: LoginPage,
});

function LoginPage() {
  const { data: session, isPending } = useSession();
  const navigate = useNavigate();

  useEffect(() => {
    if (!isPending && session?.user) {
      navigate({ to: "/" });
    }
  }, [session, isPending, navigate]);

  const handleLogin = () => {
    signIn.social({
      provider: "microsoft",
      callbackURL: "/",
    });
  };

  return (
    <div>
      <h1>Sign In</h1>
      <Button onClick={handleLogin}>Sign in with Microsoft</Button>
    </div>
  );
}
```

**`app/src/routes/api/auth/$.ts`** (Auth Handler)
```typescript
import { createFileRoute } from "@tanstack/react-router";
import { handler } from "../../../lib/auth-server";

export const Route = createFileRoute("/api/auth/$")({
  server: {
    handlers: {
      GET: ({ request }) => handler(request),
      POST: ({ request }) => handler(request),
    },
  },
});
```

---

## Convex Backend

### App Configuration with Plugins

**`app/convex/convex.config.ts`**
```typescript
import { defineApp } from "convex/server";
import betterAuth from "@convex-dev/better-auth/convex.config";

const app = defineApp();
app.use(betterAuth);

export default app;
```

### Database Schema

**`app/convex/schema.ts`**
```typescript
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  // Example: Files table for file storage metadata
  files: defineTable({
    storageId: v.id("_storage"),      // Reference to Convex storage
    filename: v.string(),              // Original filename
    contentType: v.optional(v.string()),
    size: v.optional(v.number()),
    userId: v.string(),                // Owner
    uploadedAt: v.number(),            // Timestamp
  })
    .index("by_storageId", ["storageId"])
    .index("by_user", ["userId"]),

  // Add more tables as needed
  // posts: defineTable({...}),
  // comments: defineTable({...}),
});
```

### HTTP Routes

**`app/convex/http.ts`**
```typescript
import { httpRouter } from "convex/server";
import { authComponent, createAuth } from "./auth";

const http = httpRouter();

// Register Better Auth routes (pass createAuth function, not path string)
authComponent.registerRoutes(http, createAuth);

export default http;
```

### Convex Functions

**`app/convex/[feature].ts`** (Example)
```typescript
import { v } from "convex/values";
import { mutation, query, action } from "./_generated/server";
import { authComponent } from "./auth";

// Query: Read data (reactive, cached)
export const list = query({
  args: {},
  handler: async (ctx) => {
    const user = await authComponent.safeGetAuthUser(ctx);
    if (!user) return [];

    return await ctx.db
      .query("items")
      .withIndex("by_user", (q) => q.eq("userId", user._id))
      .order("desc")
      .collect();
  },
});

// Mutation: Write data (transactional)
export const create = mutation({
  args: {
    title: v.string(),
    content: v.string(),
  },
  handler: async (ctx, args) => {
    const user = await authComponent.getAuthUser(ctx);
    if (!user) throw new Error("Unauthorized");

    return await ctx.db.insert("items", {
      ...args,
      userId: user._id,
      createdAt: Date.now(),
    });
  },
});

// Action: Side effects, external APIs, file operations
export const processExternal = action({
  args: { id: v.id("items") },
  handler: async (ctx, args) => {
    const user = await authComponent.getAuthUser(ctx);
    if (!user) throw new Error("Unauthorized");

    // Call external API
    const response = await fetch("https://api.example.com/process");
    const data = await response.json();

    // Update database via mutation
    await ctx.runMutation(api.items.update, { id: args.id, data });

    return data;
  },
});
```

---

## Better Auth Integration

### Auth Component Setup

**`app/convex/auth.ts`**
```typescript
import { betterAuth } from "better-auth/minimal";
import { createClient, type GenericCtx } from "@convex-dev/better-auth";
import { convex } from "@convex-dev/better-auth/plugins";
import authConfig from "./auth.config";
import { components } from "./_generated/api";
import type { DataModel } from "./_generated/dataModel";

// Create auth component instance with DataModel generic for type safety
export const authComponent = createClient<DataModel>(components.betterAuth);

// Create Better Auth instance (used in HTTP handlers)
export const createAuth = (ctx: GenericCtx<DataModel>) => {
  return betterAuth({
    baseURL: process.env.BETTER_AUTH_BASE_URL!,
    database: authComponent.adapter(ctx),
    socialProviders: {
      microsoft: {
        clientId: process.env.MICROSOFT_CLIENT_ID!,
        clientSecret: process.env.MICROSOFT_CLIENT_SECRET!,
        tenantId: process.env.MICROSOFT_TENANT_ID, // Optional for multi-tenant
      },
      // Add more providers as needed:
      // google: { clientId: ..., clientSecret: ... },
      // github: { clientId: ..., clientSecret: ... },
    },
    plugins: [convex({ authConfig })],
  });
};
```

**`app/convex/auth.config.ts`**
```typescript
import { getAuthConfigProvider } from "@convex-dev/better-auth/auth-config";
import type { AuthConfig } from "convex/server";

export default {
  providers: [getAuthConfigProvider()],
} satisfies AuthConfig;
```

### Auth Client

**`app/src/lib/auth-client.ts`**
```typescript
import { createAuthClient } from "better-auth/react";
import { convexClient } from "@convex-dev/better-auth/client/plugins";

export const authClient = createAuthClient({
  plugins: [convexClient()],
});

export const { signIn, signOut, useSession } = authClient;
```

### Auth Server Handler

**`app/src/lib/auth-server.ts`**
```typescript
import { convexBetterAuthReactStart } from "@convex-dev/better-auth/react-start";

export const {
  handler,
  getToken,
  fetchAuthQuery,
  fetchAuthMutation,
  fetchAuthAction,
} = convexBetterAuthReactStart({
  convexUrl: import.meta.env.VITE_CONVEX_URL!,
  convexSiteUrl: import.meta.env.VITE_CONVEX_SITE_URL!,
});
```

### Auth Patterns in Functions

```typescript
import { authComponent } from "./auth";

// In queries - use safeGetAuthUser (returns null if not authenticated)
export const listItems = query({
  handler: async (ctx) => {
    const user = await authComponent.safeGetAuthUser(ctx);
    if (!user) return []; // Return empty for unauthenticated
    // ... rest of handler
  },
});

// In mutations/actions - use getAuthUser (throws if not authenticated)
export const createItem = mutation({
  handler: async (ctx, args) => {
    const user = await authComponent.getAuthUser(ctx);
    if (!user) throw new Error("Unauthorized"); // Explicit check
    // ... rest of handler
  },
});
```

---

## OAuth Provider Configuration

### Microsoft Entra ID (Azure AD)

1. **Register Application in Azure Portal**
   - Go to Azure Portal → Microsoft Entra ID → App registrations
   - Click "New registration"
   - Name: Your app name
   - Supported account types: Choose based on needs
   - Redirect URI: `https://your-convex-site.convex.site/api/auth/callback/microsoft`

2. **Configure Authentication**
   - Add redirect URIs for both dev and prod:
     - `https://[dev-deployment].convex.site/api/auth/callback/microsoft`
     - `https://[prod-deployment].convex.site/api/auth/callback/microsoft`
   - Enable ID tokens under Implicit grant

3. **Create Client Secret**
   - Go to Certificates & secrets
   - Create new client secret
   - Copy the value immediately (shown only once)

4. **Set Environment Variables in Convex**
   ```bash
   npx convex env set MICROSOFT_CLIENT_ID "your-client-id"
   npx convex env set MICROSOFT_CLIENT_SECRET "your-client-secret"
   npx convex env set MICROSOFT_TENANT_ID "your-tenant-id" # Optional
   npx convex env set BETTER_AUTH_BASE_URL "https://your-domain.com"
   npx convex env set BETTER_AUTH_SECRET "your-random-secret-32-chars-min"
   ```

### Google OAuth

1. **Create Project in Google Cloud Console**
   - Go to APIs & Services → Credentials
   - Create OAuth 2.0 Client ID
   - Add authorized redirect URIs

2. **Configuration**
   ```typescript
   // In convex/auth.ts
   socialProviders: {
     google: {
       clientId: process.env.GOOGLE_CLIENT_ID!,
       clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
     },
   },
   ```

### GitHub OAuth

1. **Create OAuth App in GitHub Settings**
   - Settings → Developer settings → OAuth Apps
   - Set callback URL

2. **Configuration**
   ```typescript
   socialProviders: {
     github: {
       clientId: process.env.GITHUB_CLIENT_ID!,
       clientSecret: process.env.GITHUB_CLIENT_SECRET!,
     },
   },
   ```

---

## File Storage

### Backend: File Operations

**`app/convex/files.ts`**
```typescript
import { v } from "convex/values";
import { mutation, query } from "./_generated/server";
import { authComponent } from "./auth";

// Generate upload URL (client uploads directly to Convex storage)
export const generateUploadUrl = mutation({
  args: {},
  handler: async (ctx) => {
    const user = await authComponent.getAuthUser(ctx);
    if (!user) throw new Error("Unauthorized");
    return await ctx.storage.generateUploadUrl();
  },
});

// Save file metadata after successful upload
export const saveFileMetadata = mutation({
  args: {
    storageId: v.id("_storage"),
    filename: v.string(),
    contentType: v.optional(v.string()),
    size: v.optional(v.number()),
  },
  handler: async (ctx, args) => {
    const user = await authComponent.getAuthUser(ctx);
    if (!user) throw new Error("Unauthorized");

    return await ctx.db.insert("files", {
      storageId: args.storageId,
      filename: args.filename,
      contentType: args.contentType,
      size: args.size,
      userId: user._id,
      uploadedAt: Date.now(),
    });
  },
});

// List user's files with download URLs
export const listFiles = query({
  args: {},
  handler: async (ctx) => {
    const user = await authComponent.safeGetAuthUser(ctx);
    if (!user) return [];

    const files = await ctx.db
      .query("files")
      .withIndex("by_user", (q) => q.eq("userId", user._id))
      .order("desc")
      .collect();

    // Attach download URLs
    return Promise.all(
      files.map(async (file) => ({
        ...file,
        url: await ctx.storage.getUrl(file.storageId),
      }))
    );
  },
});

// Delete file (storage + metadata)
export const deleteFile = mutation({
  args: { storageId: v.id("_storage") },
  handler: async (ctx, args) => {
    const user = await authComponent.getAuthUser(ctx);
    if (!user) throw new Error("Unauthorized");

    const file = await ctx.db
      .query("files")
      .withIndex("by_storageId", (q) => q.eq("storageId", args.storageId))
      .first();

    if (!file) throw new Error("File not found");
    if (file.userId !== user._id) throw new Error("Unauthorized");

    await ctx.storage.delete(args.storageId);
    await ctx.db.delete(file._id);

    return { success: true };
  },
});
```

### Frontend: Upload Component

```typescript
import { useMutation, useQuery } from "convex/react";
import { api } from "../../convex/_generated/api";
import { useState, useRef } from "react";

function FileUploader() {
  const [uploading, setUploading] = useState(false);
  const fileInputRef = useRef<HTMLInputElement>(null);

  const generateUploadUrl = useMutation(api.files.generateUploadUrl);
  const saveFileMetadata = useMutation(api.files.saveFileMetadata);
  const files = useQuery(api.files.listFiles);

  const handleUpload = async () => {
    const file = fileInputRef.current?.files?.[0];
    if (!file) return;

    setUploading(true);
    try {
      // Step 1: Get upload URL
      const uploadUrl = await generateUploadUrl();

      // Step 2: Upload file directly to Convex storage
      const result = await fetch(uploadUrl, {
        method: "POST",
        headers: { "Content-Type": file.type },
        body: file,
      });

      if (!result.ok) throw new Error("Upload failed");
      const { storageId } = await result.json();

      // Step 3: Save metadata
      await saveFileMetadata({
        storageId,
        filename: file.name,
        contentType: file.type,
        size: file.size,
      });
    } finally {
      setUploading(false);
    }
  };

  // Download with original filename
  const handleDownload = async (url: string, filename: string) => {
    const response = await fetch(url);
    const blob = await response.blob();
    const downloadUrl = URL.createObjectURL(blob);

    const a = document.createElement("a");
    a.href = downloadUrl;
    a.download = filename;
    a.click();
    URL.revokeObjectURL(downloadUrl);
  };

  return (
    <div>
      <input ref={fileInputRef} type="file" />
      <button onClick={handleUpload} disabled={uploading}>
        {uploading ? "Uploading..." : "Upload"}
      </button>

      <ul>
        {files?.map((file) => (
          <li key={file._id}>
            {file.filename}
            <button onClick={() => handleDownload(file.url!, file.filename)}>
              Download
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## UI Components

### shadcn/ui Configuration

shadcn/ui supports Tailwind v4's CSS-first approach. Note the empty `config` field—no JavaScript config file is needed.

**`app/components.json`**
```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "app/src/styles.css",
    "baseColor": "neutral",
    "cssVariables": true,
    "prefix": ""
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  },
  "iconLibrary": "lucide",
  "registry": "https://ui.shadcn.com/r"
}
```

### Installing Components

```bash
# Install shadcn CLI
bun add -D shadcn

# Add components
bunx shadcn@latest add button
bunx shadcn@latest add card
bunx shadcn@latest add input
bunx shadcn@latest add select
bunx shadcn@latest add dropdown-menu
# ... add more as needed
```

### Utility Function

**`app/src/lib/utils.ts`**
```typescript
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### Example Component Usage

```typescript
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";

function ExampleForm() {
  return (
    <Card>
      <CardHeader>
        <CardTitle>Form Title</CardTitle>
      </CardHeader>
      <CardContent>
        <Input placeholder="Enter text..." className="mb-4" />
        <Button variant="default">Submit</Button>
        <Button variant="outline" className="ml-2">Cancel</Button>
      </CardContent>
    </Card>
  );
}
```

### Global Styles (Tailwind v4)

Tailwind CSS v4 uses a CSS-first configuration approach—no `tailwind.config.js` file needed. All configuration is done directly in your CSS file using `@import` and `@theme` directives. The `@tailwindcss/vite` plugin handles everything automatically.

**`app/src/styles.css`**
```css
@import "tailwindcss";
@import "tw-animate-css";
@import "shadcn/tailwind.css";
@import "@fontsource-variable/inter";

@custom-variant dark (&:is(.dark *));

:root {
  --font-sans: "Inter Variable", ui-sans-serif, system-ui, sans-serif;
  --background: oklch(0.985 0 0);
  --foreground: oklch(0.145 0 0);
  --card: oklch(0.985 0 0);
  --card-foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.97 0 0);
  --secondary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.556 0 0);
  --accent: oklch(0.97 0 0);
  --accent-foreground: oklch(0.205 0 0);
  --destructive: oklch(0.577 0.245 27.325);
  --destructive-foreground: oklch(0.577 0.245 27.325);
  --border: oklch(0.922 0 0);
  --input: oklch(0.922 0 0);
  --ring: oklch(0.708 0 0);
  --radius: 0.625rem;
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --card: oklch(0.205 0 0);
  --card-foreground: oklch(0.985 0 0);
  --primary: oklch(0.922 0 0);
  --primary-foreground: oklch(0.205 0 0);
  --secondary: oklch(0.269 0 0);
  --secondary-foreground: oklch(0.985 0 0);
  --muted: oklch(0.269 0 0);
  --muted-foreground: oklch(0.708 0 0);
  --accent: oklch(0.269 0 0);
  --accent-foreground: oklch(0.985 0 0);
  --destructive: oklch(0.704 0.191 22.216);
  --destructive-foreground: oklch(0.985 0 0);
  --border: oklch(0.269 0 0);
  --input: oklch(0.269 0 0);
  --ring: oklch(0.556 0 0);
}

body {
  font-family: var(--font-sans);
  background-color: var(--background);
  color: var(--foreground);
}
```

---

## Cloudflare Workers Deployment

### Wrangler Configuration

**`app/wrangler.toml`**
```toml
name = "your-app-name"
main = "@tanstack/react-start/server-entry"
compatibility_date = "2025-01-01"
compatibility_flags = ["nodejs_compat"]

[[routes]]
pattern = "your-domain.com"
custom_domain = true

[observability]
enabled = true

[observability.logs]
enabled = true
invocation_logs = true
```

### Vite Configuration

This is the central configuration file for TanStack Start (v1.121.0+). Key notes:
- **TanStack Start** is now a Vite plugin—no `app.config.ts` needed
- **Plugin order matters**: `tanstackStart()` must come before `viteReact()`
- **Tailwind v4** uses `@tailwindcss/vite` instead of PostCSS

**`app/vite.config.ts`**
```typescript
import { defineConfig } from "vite";
import { cloudflare } from "@cloudflare/vite-plugin";
import viteReact from "@vitejs/plugin-react";
import { tanstackStart } from "@tanstack/react-start/plugin/vite";
import viteTsConfigPaths from "vite-tsconfig-paths";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [
    cloudflare({ viteEnvironment: { name: "ssr" } }),
    viteTsConfigPaths(),
    tailwindcss(),
    tanstackStart(),
    viteReact(),
  ],
  ssr: {
    noExternal: ["@convex-dev/better-auth"],
  },
});
```

### Deployment Commands

```bash
# Deploy everything
bun run deploy

# Or step by step:
npx convex deploy          # Deploy Convex backend
bun run build              # Build frontend
wrangler deploy            # Deploy to Cloudflare Workers
```

### Custom Domain Setup

1. Add domain to Cloudflare (if not already)
2. In `wrangler.toml`, add route with `custom_domain = true`
3. Deploy with `wrangler deploy`
4. Cloudflare automatically provisions SSL

---

## Environment Variables

### Local Development

**`app/.env.local`**
```bash
# Convex Development
CONVEX_DEPLOYMENT=dev:your-dev-deployment
VITE_CONVEX_URL=https://your-dev-deployment.convex.cloud
VITE_CONVEX_SITE_URL=https://your-dev-deployment.convex.site
VITE_SITE_URL=http://localhost:3000
```

### Production

**`app/.env.production`**
```bash
# Convex Production
VITE_CONVEX_URL=https://your-prod-deployment.convex.cloud
VITE_CONVEX_SITE_URL=https://your-prod-deployment.convex.site
VITE_SITE_URL=https://your-domain.com
```

### Server-Side Secrets (Convex Dashboard)

Set these via CLI or Convex Dashboard:

```bash
# Authentication
npx convex env set BETTER_AUTH_SECRET "random-32-char-secret"
npx convex env set BETTER_AUTH_BASE_URL "https://your-domain.com"

# Microsoft OAuth
npx convex env set MICROSOFT_CLIENT_ID "..."
npx convex env set MICROSOFT_CLIENT_SECRET "..."
npx convex env set MICROSOFT_TENANT_ID "..."  # Optional

# Google OAuth (if using)
npx convex env set GOOGLE_CLIENT_ID "..."
npx convex env set GOOGLE_CLIENT_SECRET "..."

# GitHub OAuth (if using)
npx convex env set GITHUB_CLIENT_ID "..."
npx convex env set GITHUB_CLIENT_SECRET "..."
```

### Variable Naming Convention

| Prefix | Access | Use Case |
|--------|--------|----------|
| `VITE_` | Browser + Server | Public config (URLs, feature flags) |
| No prefix | Server only | Secrets (API keys, OAuth secrets) |

---

## Development Workflow

### Initial Setup

```bash
# 1. Install dependencies
bun install

# 2. Initialize Convex (first time only)
npx convex dev --once

# 3. Set up environment variables
# Copy .env.example to .env.local and fill in values

# 4. Start development
bun run dev
```

### Daily Development

```bash
# Start dev server (runs Convex + Vite)
bun run dev

# In another terminal, watch Convex (if not using combined command)
npx convex dev
```

### Adding Features

```bash
# 1. Update schema if needed
# Edit convex/schema.ts

# 2. Create Convex functions
# Add to convex/[feature].ts

# 3. Add UI components
bunx shadcn@latest add [component]

# 4. Create route
# Add src/routes/[page].tsx

# 5. Test locally
bun run dev

# 6. Deploy
bun run deploy
```

### Package Scripts

**`package.json`**
```json
{
  "scripts": {
    "dev": "vite dev --port 3000",
    "build": "vite build",
    "deploy": "convex deploy && bun run build && wrangler deploy",
    "convex": "convex dev",
    "lint": "eslint .",
    "format": "prettier --write .",
    "typecheck": "tsc --noEmit"
  }
}
```

---

## Complete Wiring Guide

### How Authentication Flows

```
1. User clicks "Sign in with Microsoft"
   └─→ signIn.social({ provider: "microsoft" })

2. Browser redirects to /api/auth/signin/microsoft
   └─→ TanStack Start API route handles request
   └─→ authHandler processes via Better Auth

3. Redirects to Microsoft OAuth
   └─→ User authenticates with Microsoft
   └─→ Microsoft redirects to callback URL

4. Callback handled by Convex
   └─→ https://[deployment].convex.site/api/auth/callback/microsoft
   └─→ Better Auth exchanges code for tokens
   └─→ Creates/updates user in Convex DB
   └─→ Creates session in betterAuth component tables

5. Redirects back to app
   └─→ Browser receives session cookie
   └─→ useSession() hook fetches user data
   └─→ App renders authenticated state
```

### How Data Flows

```
1. Component calls Convex hook
   └─→ const items = useQuery(api.items.list)

2. Convex client sends request
   └─→ WebSocket to Convex cloud
   └─→ Includes auth token automatically

3. Convex function executes
   └─→ authComponent.getAuthUser(ctx) verifies token
   └─→ ctx.db.query() reads from database
   └─→ Returns data to client

4. React updates
   └─→ Convex client caches response
   └─→ Component re-renders with data
   └─→ Subscribed to real-time updates
```

### How File Upload Flows

```
1. User selects file
   └─→ File stored in browser memory

2. Request upload URL
   └─→ generateUploadUrl mutation
   └─→ Auth verified, URL generated
   └─→ URL returned to client

3. Upload file directly
   └─→ fetch(uploadUrl, { body: file })
   └─→ File streams to Convex storage
   └─→ Returns storageId

4. Save metadata
   └─→ saveFileMetadata mutation
   └─→ Stores filename, size, userId, etc.
   └─→ Links storageId to metadata record

5. Display file
   └─→ listFiles query includes URLs
   └─→ ctx.storage.getUrl(storageId)
   └─→ URL valid for download
```

### Key Integration Points

| From | To | How |
|------|----|----|
| Routes | Convex | `useQuery`, `useMutation`, `useAction` hooks |
| Routes | Auth | `useSession`, `signIn`, `signOut` from auth-client |
| Convex Functions | Auth | `authComponent.getAuthUser(ctx)` |
| Convex Functions | Database | `ctx.db.query()`, `ctx.db.insert()`, etc. |
| Convex Functions | Storage | `ctx.storage.generateUploadUrl()`, `ctx.storage.getUrl()` |
| Better Auth | Convex | `convex` plugin stores sessions in Convex tables |
| TanStack Start | Cloudflare | SSR via Workers, static assets via CDN |

---

## Dependencies Reference

### Core Dependencies

```json
{
  "@tanstack/react-start": "^1.142.x",
  "@tanstack/react-router": "^1.141.x",
  "@tanstack/react-query": "^5.90.x",
  "@tanstack/router-plugin": "^1.142.x",
  "@tanstack/react-router-ssr-query": "^1.141.x",
  "convex": "^1.31.x",
  "@convex-dev/better-auth": "^0.10.x",
  "@convex-dev/react-query": "^0.1.x",
  "better-auth": "^1.4.x",
  "react": "^19.x",
  "react-dom": "^19.x"
}
```

### UI Dependencies

```json
{
  "tailwindcss": "^4.1.x",
  "@tailwindcss/vite": "^4.1.x",
  "shadcn": "^3.6.x",
  "@base-ui/react": "^1.x",
  "class-variance-authority": "^0.7.x",
  "clsx": "^2.x",
  "tailwind-merge": "^3.4.x",
  "lucide-react": "^0.562.x",
  "tw-animate-css": "^1.4.x",
  "@fontsource-variable/inter": "^5.x"
}
```

> **Note:** Tailwind v4 with Vite does not require `postcss` or `autoprefixer`—the Vite plugin handles everything including vendor prefixing via Lightning CSS.

### Build/Deploy Dependencies

```json
{
  "@cloudflare/vite-plugin": "^1.19.x",
  "wrangler": "^4.56.x",
  "vite": "^7.3.x",
  "@vitejs/plugin-react": "^5.1.x",
  "vite-tsconfig-paths": "^5.1.x",
  "typescript": "^5.9.x"
}
```

---

## Quick Reference Commands

```bash
# Development
bun run dev                    # Start dev server
npx convex dev                 # Watch Convex functions
npx convex dashboard           # Open Convex dashboard

# Convex
npx convex env set KEY value   # Set environment variable
npx convex env list            # List environment variables
npx convex logs                # View function logs

# UI Components
bunx shadcn@latest add [name]  # Add shadcn component
bunx shadcn@latest diff        # Check for updates

# Deployment
bun run build                  # Build for production
wrangler deploy                # Deploy to Cloudflare
npx convex deploy              # Deploy Convex functions

# Full deploy
bun run deploy                 # convex deploy + build + wrangler deploy
```

---

*This specification provides a complete reference for building modern full-stack applications with type-safe database operations, real-time updates, secure authentication, and edge deployment.*
