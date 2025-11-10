# Complete Caching Strategy Reference Guide for Web Applications

This guide provides a comprehensive, production-ready approach to implementing caching strategies in your SvelteKit + AWS CDK serverless applications.

## Table of Contents

1. [Decision Framework](#decision-framework)
2. [Architecture Layers](#architecture-layers)
3. [Implementation Patterns](#implementation-patterns)
4. [Client-Side Caching](#client-side-caching)
5. [AWS CDK Constructs](#aws-cdk-constructs)
6. [SvelteKit Integration](#sveltekit-integration)
7. [Monitoring & Tuning](#monitoring--tuning)

## Decision Framework

### Step 1: Classify Your Data

Before implementing any caching, categorize your data types:

| Data Type       | Freshness Requirement | Recommended Strategy                                  | Example                      |
| --------------- | --------------------- | ----------------------------------------------------- | ---------------------------- |
| **Static**      | Days/Weeks            | CDN + Long TTL + Browser Cache                        | Images, CSS, JS bundles      |
| **Semi-Static** | Hours                 | Application Cache + Medium TTL + IndexedDB            | Product catalogs, blog posts |
| **Dynamic**     | Minutes               | Application Cache + Short TTL + sessionStorage        | User feeds, recommendations  |
| **Real-Time**   | Seconds               | No cache or DAX + Svelte stores                       | Live prices, inventory       |
| **Critical**    | Must be fresh         | Strong consistency + No cache                         | Payments, transactions       |
| **Session**     | Session lifetime      | In-memory (Svelte stores) + sessionStorage or Redis   | User sessions, cart data     |
| **User Prefs**  | Persistent            | localStorage + Backend sync                           | Theme, language, favorites   |

### Step 2: Assess Your Use Case

Answer these questions for each API endpoint or data access pattern:

```typescript
interface CachingDecision {
  // Business requirements
  maxAcceptableStaleness: number; // in seconds
  consistencyRequired: "strong" | "eventual" | "none";

  // Traffic patterns
  readWriteRatio: number; // e.g., 100:1 for read-heavy
  requestsPerSecond: number;

  // Performance requirements
  targetLatencyP95: number; // in milliseconds

  // Cost constraints
  monthlyCostBudget: number;

  // Data characteristics
  dataSize: number; // in KB
  updateFrequency: number; // updates per hour
}
```

### Step 3: Choose Your Caching Layers

Based on your answers, select appropriate layers:

```
┌─────────────────────────────────────────────┐
│ Browser Cache (0-10ms)                      │
│ • Static assets, HTML                       │
│ • Cache-Control headers                     │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ SvelteKit Server Load Cache (10-50ms)       │
│ • Server-side caching with setHeaders       │
│ • Shared across requests                    │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ API Gateway Cache (50-100ms)                │
│ • Response caching at edge                  │
│ • Good for public APIs                      │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ Lambda In-Memory Cache (1-5ms)              │
│ • Module-level caching                      │
│ • Lasts for Lambda lifetime                 │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ ElastiCache/Redis (1-10ms)                  │
│ • Shared cache across Lambda invocations    │
│ • Best for frequently accessed data         │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ DynamoDB DAX (1-5ms)                        │
│ • In-memory cache for DynamoDB              │
│ • Transparent caching                       │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│ DynamoDB (10-20ms)                          │
│ • Source of truth                           │
│ • Strong or eventual consistency            │
└─────────────────────────────────────────────┘
```

## Architecture Layers

### Layer 1: Browser/Client Cache

**When to use:**

- Static assets (images, fonts, CSS, JS)
- Infrequently changing content
- Public data that doesn't need authentication

**Implementation in SvelteKit:**

```typescript
// src/hooks.server.ts
import type { Handle } from "@sveltejs/kit";

export const handle: Handle = async ({ event, resolve }) => {
  const response = await resolve(event);

  // Static assets - cache for 1 year
  if (event.url.pathname.startsWith("/assets/")) {
    response.headers.set(
      "Cache-Control",
      "public, max-age=31536000, immutable"
    );
  }

  // API responses - cache for 5 minutes
  if (event.url.pathname.startsWith("/api/products")) {
    response.headers.set(
      "Cache-Control",
      "public, max-age=300, stale-while-revalidate=60"
    );
  }

  // User-specific data - no cache
  if (event.url.pathname.startsWith("/api/user")) {
    response.headers.set(
      "Cache-Control",
      "private, no-cache, no-store, must-revalidate"
    );
  }

  return response;
};
```

**Cache-Control Directives Reference:**

```typescript
// Cache-Control header patterns
const cachePatterns = {
  // No caching (sensitive data)
  noCache: "private, no-cache, no-store, must-revalidate",

  // Short cache (5 minutes) with revalidation
  shortCache: "public, max-age=300, stale-while-revalidate=60",

  // Medium cache (1 hour) with revalidation
  mediumCache: "public, max-age=3600, stale-while-revalidate=300",

  // Long cache (1 day) with revalidation
  longCache: "public, max-age=86400, stale-while-revalidate=3600",

  // Immutable (versioned assets)
  immutable: "public, max-age=31536000, immutable",

  // Private user data (cache in browser only)
  privateCache: "private, max-age=300",

  // Conditional with ETag
  conditional: "public, max-age=0, must-revalidate",
};
```

### Layer 2: SvelteKit Load Function Cache

**When to use:**

- Server-side rendered content
- Data that can be shared across users
- Reduce backend API calls

**Implementation:**

```typescript
// src/routes/products/+page.server.ts
import type { PageServerLoad } from "./$types";

// Simple in-memory cache at module level
const cache = new Map<string, { data: any; timestamp: number }>();
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes

export const load: PageServerLoad = async ({ fetch, setHeaders }) => {
  const cacheKey = "products_list";
  const now = Date.now();

  // Check cache
  const cached = cache.get(cacheKey);
  if (cached && now - cached.timestamp < CACHE_TTL) {
    // Set headers to enable browser caching
    setHeaders({
      "Cache-Control": "public, max-age=300",
      "X-Cache": "HIT",
    });
    return { products: cached.data };
  }

  // Fetch fresh data
  const response = await fetch("https://api.yourservice.com/products");
  const products = await response.json();

  // Update cache
  cache.set(cacheKey, { data: products, timestamp: now });

  // Set headers
  setHeaders({
    "Cache-Control": "public, max-age=300",
    "X-Cache": "MISS",
  });

  return { products };
};
```

**Advanced: Per-User Caching:**

```typescript
// src/lib/server/cache.ts
interface CacheEntry<T> {
  data: T;
  timestamp: number;
  etag: string;
}

export class SmartCache<T> {
  private cache = new Map<string, CacheEntry<T>>();
  private ttl: number;

  constructor(ttlSeconds: number) {
    this.ttl = ttlSeconds * 1000;
  }

  get(key: string): CacheEntry<T> | null {
    const entry = this.cache.get(key);
    if (!entry) return null;

    if (Date.now() - entry.timestamp > this.ttl) {
      this.cache.delete(key);
      return null;
    }

    return entry;
  }

  set(key: string, data: T): string {
    const etag = this.generateETag(data);
    this.cache.set(key, {
      data,
      timestamp: Date.now(),
      etag,
    });
    return etag;
  }

  invalidate(key: string): void {
    this.cache.delete(key);
  }

  invalidatePattern(pattern: RegExp): void {
    for (const key of this.cache.keys()) {
      if (pattern.test(key)) {
        this.cache.delete(key);
      }
    }
  }

  private generateETag(data: T): string {
    return `W/"${Buffer.from(JSON.stringify(data))
      .toString("base64")
      .slice(0, 27)}"`;
  }
}

// Usage
// src/routes/api/products/+server.ts
import { json } from "@sveltejs/kit";
import type { RequestHandler } from "./$types";
import { SmartCache } from "$lib/server/cache";

const productCache = new SmartCache<Product[]>(300); // 5 minutes

export const GET: RequestHandler = async ({ request }) => {
  const cacheKey = "products_all";

  // Check if client has cached version
  const ifNoneMatch = request.headers.get("if-none-match");

  // Check server cache
  const cached = productCache.get(cacheKey);
  if (cached) {
    if (ifNoneMatch === cached.etag) {
      // Client cache is still valid
      return new Response(null, { status: 304 });
    }

    return json(cached.data, {
      headers: {
        "Cache-Control": "public, max-age=300",
        ETag: cached.etag,
        "X-Cache": "HIT",
      },
    });
  }

  // Fetch fresh data
  const products = await fetchProductsFromDB();
  const etag = productCache.set(cacheKey, products);

  return json(products, {
    headers: {
      "Cache-Control": "public, max-age=300",
      ETag: etag,
      "X-Cache": "MISS",
    },
  });
};
```

### Layer 3: Lambda In-Memory Cache

**When to use:**

- Data used repeatedly within same Lambda execution context
- Configuration data
- Connection pooling

**Implementation:**

```typescript
// lambda/handlers/products.ts
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, QueryCommand } from "@aws-sdk/lib-dynamodb";

// Module-level cache (survives across invocations)
const cache = new Map<string, { data: any; expiry: number }>();
const CACHE_TTL = 60 * 1000; // 1 minute

// Reusable client (persists across invocations)
const dynamoClient = DynamoDBDocumentClient.from(
  new DynamoDBClient({ region: process.env.AWS_REGION })
);

export async function handler(event: any) {
  const { productId } = event.pathParameters;
  const cacheKey = `product:${productId}`;
  const now = Date.now();

  // Check in-memory cache
  const cached = cache.get(cacheKey);
  if (cached && now < cached.expiry) {
    console.log("Cache HIT (Lambda memory)");
    return {
      statusCode: 200,
      headers: {
        "Content-Type": "application/json",
        "X-Cache": "HIT-Lambda",
      },
      body: JSON.stringify(cached.data),
    };
  }

  // Fetch from DynamoDB
  const result = await dynamoClient.send(
    new QueryCommand({
      TableName: process.env.TABLE_NAME,
      KeyConditionExpression: "PK = :pk",
      ExpressionAttributeValues: {
        ":pk": `PRODUCT#${productId}`,
      },
    })
  );

  const product = result.Items?.[0];

  // Update cache
  cache.set(cacheKey, {
    data: product,
    expiry: now + CACHE_TTL,
  });

  // Cleanup expired entries (prevent memory leak)
  if (cache.size > 1000) {
    for (const [key, value] of cache.entries()) {
      if (now >= value.expiry) {
        cache.delete(key);
      }
    }
  }

  return {
    statusCode: 200,
    headers: {
      "Content-Type": "application/json",
      "X-Cache": "MISS",
    },
    body: JSON.stringify(product),
  };
}
```

### Layer 4: ElastiCache (Redis)

**When to use:**

- Shared cache across multiple Lambda functions
- High read volume
- Complex cache invalidation patterns
- Session storage

---

## Client-Side Caching

### Overview

Client-side caching is critical for:
- **Offline support** - App works without network
- **Instant UX** - No loading spinners for cached data
- **Reduced API calls** - Lower costs, better performance
- **Optimistic UI** - Update UI before server confirms

### Caching Layers in the Browser

```
┌─────────────────────────────────────────────────────────┐
│ Memory (Svelte Stores) - 0ms                            │
│ • In-memory reactive state                              │
│ • Lost on page refresh                                  │
│ • Perfect for session data                              │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ sessionStorage - 1-5ms                                  │
│ • Survives page refresh                                 │
│ • Lost when tab closes                                  │
│ • Max 5-10MB                                            │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ localStorage - 1-5ms                                    │
│ • Survives browser restart                              │
│ • Shared across tabs                                    │
│ • Max 5-10MB                                            │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ IndexedDB - 5-20ms                                      │
│ • Large storage (50MB-1GB+)                             │
│ • Structured data with indexes                          │
│ • Async API                                             │
└─────────────────────────────────────────────────────────┘
```

### Pattern 1: Svelte Stores for Session Data

**When to use:**
- User session data
- UI state (filters, sort order)
- Temporary data that doesn't need persistence

**Implementation:**

```typescript
// src/lib/stores/merchants.ts
import { writable, derived } from "svelte/store";

interface Merchant {
  merchantId: string;
  name: string;
  category: string;
  latitude: number;
  longitude: number;
  // ... other fields
}

interface MerchantsCache {
  data: Merchant[];
  timestamp: number;
  category: string;
}

// Create writable store
export const merchantsCache = writable<MerchantsCache | null>(null);

// Derived store for filtered merchants
export const filteredMerchants = derived(
  merchantsCache,
  ($cache) => $cache?.data || []
);

// Helper functions
export function setMerchantsCache(
  merchants: Merchant[],
  category: string
): void {
  merchantsCache.set({
    data: merchants,
    timestamp: Date.now(),
    category,
  });
}

export function isCacheValid(category: string, maxAgeMs: number = 300000): boolean {
  let cache: MerchantsCache | null = null;
  merchantsCache.subscribe((value) => (cache = value))();

  if (!cache) return false;
  if (cache.category !== category) return false;
  if (Date.now() - cache.timestamp > maxAgeMs) return false;

  return true;
}

export function clearMerchantsCache(): void {
  merchantsCache.set(null);
}
```

**Usage in Component:**

```svelte
<!-- src/routes/merchants/+page.svelte -->
<script lang="ts">
  import { merchantsCache, setMerchantsCache, isCacheValid } from "$lib/stores/merchants";
  import { onMount } from "svelte";

  let category = "Repair";
  let loading = false;

  async function loadMerchants() {
    // Check cache first
    if (isCacheValid(category, 300000)) {
      console.log("Using cached merchants");
      return;
    }

    loading = true;
    try {
      const response = await fetch(`/api/merchants/search?category=${category}`);
      const data = await response.json();
      
      // Update cache
      setMerchantsCache(data.merchants, category);
    } finally {
      loading = false;
    }
  }

  onMount(() => {
    loadMerchants();
  });
</script>

{#if loading}
  <p>Loading...</p>
{:else if $merchantsCache}
  <ul>
    {#each $merchantsCache.data as merchant}
      <li>{merchant.name}</li>
    {/each}
  </ul>
{/if}
```

### Pattern 2: sessionStorage for Tab-Scoped Data

**When to use:**
- Search filters and results
- Form data (auto-save)
- Temporary user preferences

**Implementation:**

```typescript
// src/lib/utils/session-cache.ts
interface CacheEntry<T> {
  data: T;
  timestamp: number;
  ttl: number;
}

export class SessionCache {
  static set<T>(key: string, data: T, ttlMs: number = 300000): void {
    const entry: CacheEntry<T> = {
      data,
      timestamp: Date.now(),
      ttl: ttlMs,
    };
    sessionStorage.setItem(key, JSON.stringify(entry));
  }

  static get<T>(key: string): T | null {
    const item = sessionStorage.getItem(key);
    if (!item) return null;

    try {
      const entry: CacheEntry<T> = JSON.parse(item);
      
      // Check if expired
      if (Date.now() - entry.timestamp > entry.ttl) {
        sessionStorage.removeItem(key);
        return null;
      }

      return entry.data;
    } catch {
      return null;
    }
  }

  static remove(key: string): void {
    sessionStorage.removeItem(key);
  }

  static clear(): void {
    sessionStorage.clear();
  }
}

// Usage
// src/routes/merchants/+page.ts
import type { PageLoad } from "./$types";
import { SessionCache } from "$lib/utils/session-cache";

export const load: PageLoad = async ({ fetch, url }) => {
  const category = url.searchParams.get("category") || "Repair";
  const cacheKey = `merchants:${category}`;

  // Try cache first
  const cached = SessionCache.get<Merchant[]>(cacheKey);
  if (cached) {
    return { merchants: cached, fromCache: true };
  }

  // Fetch from API
  const response = await fetch(`/api/merchants/search?category=${category}`);
  const data = await response.json();

  // Cache for 5 minutes
  SessionCache.set(cacheKey, data.merchants, 300000);

  return { merchants: data.merchants, fromCache: false };
};
```

### Pattern 3: localStorage for Persistent Data

**When to use:**
- User preferences (theme, language)
- Favorites/bookmarks
- Recently viewed items
- Offline-first data

**Implementation:**

```typescript
// src/lib/utils/local-cache.ts
export class LocalCache {
  static set<T>(key: string, data: T): void {
    try {
      localStorage.setItem(key, JSON.stringify(data));
    } catch (error) {
      console.error("localStorage.setItem failed:", error);
      // Handle quota exceeded
    }
  }

  static get<T>(key: string): T | null {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : null;
    } catch {
      return null;
    }
  }

  static remove(key: string): void {
    localStorage.removeItem(key);
  }

  static clear(): void {
    localStorage.clear();
  }
}

// Example: Favorite merchants
// src/lib/stores/favorites.ts
import { writable } from "svelte/store";
import { LocalCache } from "$lib/utils/local-cache";
import { browser } from "$app/environment";

function createFavoritesStore() {
  // Initialize from localStorage
  const initial = browser ? LocalCache.get<string[]>("favorites") || [] : [];
  const { subscribe, set, update } = writable<string[]>(initial);

  return {
    subscribe,
    add: (merchantId: string) => {
      update((favorites) => {
        const updated = [...favorites, merchantId];
        if (browser) LocalCache.set("favorites", updated);
        return updated;
      });
    },
    remove: (merchantId: string) => {
      update((favorites) => {
        const updated = favorites.filter((id) => id !== merchantId);
        if (browser) LocalCache.set("favorites", updated);
        return updated;
      });
    },
    clear: () => {
      set([]);
      if (browser) LocalCache.remove("favorites");
    },
  };
}

export const favorites = createFavoritesStore();
```

### Pattern 4: IndexedDB for Large Datasets

**When to use:**
- Offline-first applications
- Large datasets (merchant catalogs, maps)
- Complex queries with indexes
- Background sync

**Implementation with Dexie.js:**

```typescript
// src/lib/db/merchants-db.ts
import Dexie, { type Table } from "dexie";

export interface Merchant {
  merchantId: string;
  name: string;
  category: string;
  latitude: number;
  longitude: number;
  rating: number;
  createdAt: string;
  updatedAt: string;
}

export class MerchantsDB extends Dexie {
  merchants!: Table<Merchant, string>;

  constructor() {
    super("MerchantsDB");
    
    this.version(1).stores({
      merchants: "merchantId, category, rating, updatedAt",
    });
  }
}

export const db = new MerchantsDB();

// Helper functions
export async function cacheMerchants(merchants: Merchant[]): Promise<void> {
  await db.merchants.bulkPut(merchants);
}

export async function getMerchantsByCategory(
  category: string
): Promise<Merchant[]> {
  return await db.merchants.where("category").equals(category).toArray();
}

export async function getMerchantById(
  merchantId: string
): Promise<Merchant | undefined> {
  return await db.merchants.get(merchantId);
}

export async function clearMerchantsCache(): Promise<void> {
  await db.merchants.clear();
}

// Check if cache is stale
export async function isCacheStale(maxAgeMs: number = 300000): Promise<boolean> {
  const merchants = await db.merchants.toArray();
  if (merchants.length === 0) return true;

  // Check most recent updatedAt
  const mostRecent = merchants.reduce((latest, merchant) => {
    const merchantTime = new Date(merchant.updatedAt).getTime();
    return merchantTime > latest ? merchantTime : latest;
  }, 0);

  return Date.now() - mostRecent > maxAgeMs;
}
```

**Usage in Load Function:**

```typescript
// src/routes/merchants/+page.ts
import type { PageLoad } from "./$types";
import { db, cacheMerchants, getMerchantsByCategory, isCacheStale } from "$lib/db/merchants-db";
import { browser } from "$app/environment";

export const load: PageLoad = async ({ fetch, url }) => {
  const category = url.searchParams.get("category") || "Repair";

  // Only use IndexedDB in browser
  if (browser) {
    try {
      // Check if cache is valid
      const cacheStale = await isCacheStale(300000); // 5 minutes

      if (!cacheStale) {
        const cached = await getMerchantsByCategory(category);
        if (cached.length > 0) {
          console.log("Using IndexedDB cache");
          return { merchants: cached, fromCache: true };
        }
      }
    } catch (error) {
      console.error("IndexedDB error:", error);
      // Fall through to API fetch
    }
  }

  // Fetch from API
  const response = await fetch(`/api/merchants/search?category=${category}`);
  const data = await response.json();

  // Cache in IndexedDB (fire and forget)
  if (browser) {
    cacheMerchants(data.merchants).catch(console.error);
  }

  return { merchants: data.merchants, fromCache: false };
};
```

### Pattern 5: Combined Strategy (Multi-Layer)

**Best practice for production apps:**

```typescript
// src/lib/cache/merchant-cache.ts
import { writable } from "svelte/store";
import { SessionCache } from "$lib/utils/session-cache";
import { db, cacheMerchants, getMerchantsByCategory } from "$lib/db/merchants-db";
import { browser } from "$app/environment";

interface CacheStrategy {
  memory: boolean;
  session: boolean;
  indexedDB: boolean;
}

export class MerchantCache {
  private memoryCache = writable<Merchant[] | null>(null);
  private strategy: CacheStrategy;

  constructor(strategy: CacheStrategy = { memory: true, session: true, indexedDB: true }) {
    this.strategy = strategy;
  }

  async get(category: string): Promise<Merchant[] | null> {
    // Layer 1: Memory (Svelte store) - fastest
    if (this.strategy.memory) {
      let cached: Merchant[] | null = null;
      this.memoryCache.subscribe((value) => (cached = value))();
      if (cached) {
        console.log("Cache HIT: Memory");
        return cached;
      }
    }

    // Layer 2: sessionStorage - fast
    if (this.strategy.session && browser) {
      const cached = SessionCache.get<Merchant[]>(`merchants:${category}`);
      if (cached) {
        console.log("Cache HIT: sessionStorage");
        // Populate memory cache
        this.memoryCache.set(cached);
        return cached;
      }
    }

    // Layer 3: IndexedDB - slower but larger capacity
    if (this.strategy.indexedDB && browser) {
      try {
        const cached = await getMerchantsByCategory(category);
        if (cached.length > 0) {
          console.log("Cache HIT: IndexedDB");
          // Populate upper layers
          this.memoryCache.set(cached);
          if (this.strategy.session) {
            SessionCache.set(`merchants:${category}`, cached, 300000);
          }
          return cached;
        }
      } catch (error) {
        console.error("IndexedDB error:", error);
      }
    }

    console.log("Cache MISS: All layers");
    return null;
  }

  async set(category: string, merchants: Merchant[]): Promise<void> {
    // Update all layers
    if (this.strategy.memory) {
      this.memoryCache.set(merchants);
    }

    if (browser) {
      if (this.strategy.session) {
        SessionCache.set(`merchants:${category}`, merchants, 300000);
      }

      if (this.strategy.indexedDB) {
        try {
          await cacheMerchants(merchants);
        } catch (error) {
          console.error("Failed to cache in IndexedDB:", error);
        }
      }
    }
  }

  clear(): void {
    this.memoryCache.set(null);
    if (browser) {
      SessionCache.clear();
      db.merchants.clear().catch(console.error);
    }
  }
}

// Usage
export const merchantCache = new MerchantCache();
```

**Usage in Component:**

```svelte
<!-- src/routes/merchants/+page.svelte -->
<script lang="ts">
  import { merchantCache } from "$lib/cache/merchant-cache";
  import { onMount } from "svelte";

  let category = "Repair";
  let merchants: Merchant[] = [];
  let loading = false;

  async function loadMerchants() {
    // Try cache first
    const cached = await merchantCache.get(category);
    if (cached) {
      merchants = cached;
      return;
    }

    // Fetch from API
    loading = true;
    try {
      const response = await fetch(`/api/merchants/search?category=${category}`);
      const data = await response.json();
      
      merchants = data.merchants;
      
      // Update all cache layers
      await merchantCache.set(category, merchants);
    } finally {
      loading = false;
    }
  }

  onMount(() => {
    loadMerchants();
  });
</script>
```

### Cache Invalidation Strategies

#### 1. Time-Based (TTL)

```typescript
interface CacheEntry<T> {
  data: T;
  timestamp: number;
  ttl: number;
}

function isExpired(entry: CacheEntry<any>): boolean {
  return Date.now() - entry.timestamp > entry.ttl;
}
```

#### 2. Event-Based

```typescript
// src/lib/events/cache-events.ts
import { writable } from "svelte/store";

export const cacheInvalidationEvents = writable<string[]>([]);

export function invalidateCache(key: string): void {
  cacheInvalidationEvents.update((events) => [...events, key]);
  
  // Clear from all layers
  SessionCache.remove(key);
  // ... clear from other layers
}

// Listen for invalidation events
cacheInvalidationEvents.subscribe((events) => {
  events.forEach((key) => {
    console.log(`Invalidating cache: ${key}`);
  });
});
```

#### 3. Server-Sent Events (SSE)

```typescript
// src/lib/cache/sse-invalidation.ts
export function setupCacheInvalidation() {
  const eventSource = new EventSource("/api/cache-events");

  eventSource.addEventListener("invalidate", (event) => {
    const { key } = JSON.parse(event.data);
    console.log(`Server invalidated cache: ${key}`);
    
    // Clear cache
    SessionCache.remove(key);
    db.merchants.clear();
  });

  return () => eventSource.close();
}
```

### Best Practices

1. **Layer Appropriately**
   - Memory: Session data, UI state
   - sessionStorage: Search results, filters
   - localStorage: User preferences, favorites
   - IndexedDB: Large datasets, offline support

2. **Set Appropriate TTLs**
   - Static data: 1 hour - 1 day
   - Semi-static: 5-15 minutes
   - Dynamic: 1-5 minutes
   - User-specific: Session lifetime

3. **Handle Quota Exceeded**
   ```typescript
   try {
     localStorage.setItem(key, value);
   } catch (error) {
     if (error.name === "QuotaExceededError") {
       // Clear old data or notify user
       localStorage.clear();
     }
   }
   ```

4. **Sync Across Tabs**
   ```typescript
   // Listen for storage events
   window.addEventListener("storage", (event) => {
     if (event.key === "merchants") {
       // Update local state
       const newData = JSON.parse(event.newValue || "[]");
       merchantsStore.set(newData);
     }
   });
   ```

5. **Offline Detection**
   ```typescript
   import { browser } from "$app/environment";

   export const isOnline = writable(browser ? navigator.onLine : true);

   if (browser) {
     window.addEventListener("online", () => isOnline.set(true));
     window.addEventListener("offline", () => isOnline.set(false));
   }
   ```

---

## AWS CDK Constructs

### 1. Basic Lambda + DynamoDB (No Cache)

```typescript
// lib/constructs/api-service.ts
import * as cdk from "aws-cdk-lib";
import * as dynamodb from "aws-cdk-lib/aws-dynamodb";
import * as lambda from "aws-cdk-lib/aws-lambda";
import * as apigateway from "aws-cdk-lib/aws-apigateway";
import { Construct } from "constructs";

export interface ApiServiceProps {
  serviceName: string;
  environment?: Record<string, string>;
}

export class ApiService extends Construct {
  public readonly table: dynamodb.Table;
  public readonly handler: lambda.Function;
  public readonly api: apigateway.RestApi;

  constructor(scope: Construct, id: string, props: ApiServiceProps) {
    super(scope, id);

    // DynamoDB Table
    this.table = new dynamodb.Table(this, "Table", {
      partitionKey: { name: "PK", type: dynamodb.AttributeType.STRING },
      sortKey: { name: "SK", type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
      pointInTimeRecovery: true,
      removalPolicy: cdk.RemovalPolicy.RETAIN,
      stream: dynamodb.StreamViewType.NEW_AND_OLD_IMAGES, // For cache invalidation
    });

    // Lambda Function
    this.handler = new lambda.Function(this, "Handler", {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: "index.handler",
      code: lambda.Code.fromAsset("lambda/dist"),
      environment: {
        TABLE_NAME: this.table.tableName,
        ...props.environment,
      },
      timeout: cdk.Duration.seconds(30),
      memorySize: 1024,
    });

    // Grant permissions
    this.table.grantReadWriteData(this.handler);

    // API Gateway
    this.api = new apigateway.RestApi(this, "Api", {
      restApiName: props.serviceName,
      deployOptions: {
        stageName: "prod",
        tracingEnabled: true,
        metricsEnabled: true,
      },
    });

    const integration = new apigateway.LambdaIntegration(this.handler);
    this.api.root.addProxy({
      defaultIntegration: integration,
    });
  }
}
```

### 2. API Gateway with Caching

```typescript
// lib/constructs/cached-api-service.ts
import * as apigateway from "aws-cdk-lib/aws-apigateway";
import { ApiService, ApiServiceProps } from "./api-service";
import { Construct } from "constructs";

export interface CachedApiServiceProps extends ApiServiceProps {
  cacheTtlSeconds?: number;
  cacheCapacity?: string; // '0.5' | '1.6' | '6.1' | '13.5' | '28.4' | '58.2' | '118' | '237'
}

export class CachedApiService extends ApiService {
  constructor(scope: Construct, id: string, props: CachedApiServiceProps) {
    super(scope, id, props);

    // Override API Gateway with caching enabled
    const cachedApi = new apigateway.RestApi(this, "CachedApi", {
      restApiName: `${props.serviceName}-cached`,
      deployOptions: {
        stageName: "prod",
        tracingEnabled: true,
        metricsEnabled: true,
        cachingEnabled: true,
        cacheClusterEnabled: true,
        cacheClusterSize: props.cacheCapacity || "0.5", // 0.5 GB cache
        cacheTtl: cdk.Duration.seconds(props.cacheTtlSeconds || 300),
        cacheDataEncrypted: true,
      },
    });

    const integration = new apigateway.LambdaIntegration(this.handler, {
      cacheKeyParameters: ["method.request.path.id"], // Cache by path params
    });

    // Add cached endpoint
    const products = cachedApi.root.addResource("products");
    products.addMethod("GET", integration, {
      requestParameters: {
        "method.request.path.id": false,
      },
      methodResponses: [
        {
          statusCode: "200",
          responseParameters: {
            "method.response.header.Cache-Control": true,
          },
        },
      ],
    });

    // Update reference
    this.api = cachedApi;
  }
}
```

### 3. Lambda + Redis (ElastiCache)

```typescript
// lib/constructs/redis-cached-service.ts
import * as cdk from "aws-cdk-lib";
import * as ec2 from "aws-cdk-lib/aws-ec2";
import * as elasticache from "aws-cdk-lib/aws-elasticache";
import * as lambda from "aws-cdk-lib/aws-lambda";
import { Construct } from "constructs";
import { ApiService, ApiServiceProps } from "./api-service";

export interface RedisCachedServiceProps extends ApiServiceProps {
  vpc: ec2.IVpc;
  cacheNodeType?: string;
  numCacheNodes?: number;
}

export class RedisCachedService extends ApiService {
  public readonly redis: elasticache.CfnCacheCluster;
  public readonly redisSecurityGroup: ec2.SecurityGroup;

  constructor(scope: Construct, id: string, props: RedisCachedServiceProps) {
    super(scope, id, props);

    // Security Group for Redis
    this.redisSecurityGroup = new ec2.SecurityGroup(
      this,
      "RedisSecurityGroup",
      {
        vpc: props.vpc,
        description: "Security group for Redis cache",
        allowAllOutbound: true,
      }
    );

    // Lambda Security Group
    const lambdaSecurityGroup = new ec2.SecurityGroup(
      this,
      "LambdaSecurityGroup",
      {
        vpc: props.vpc,
        description: "Security group for Lambda functions",
        allowAllOutbound: true,
      }
    );

    // Allow Lambda to access Redis
    this.redisSecurityGroup.addIngressRule(
      lambdaSecurityGroup,
      ec2.Port.tcp(6379),
      "Allow Lambda to access Redis"
    );

    // Subnet Group for Redis
    const subnetGroup = new elasticache.CfnSubnetGroup(
      this,
      "RedisSubnetGroup",
      {
        description: "Subnet group for Redis",
        subnetIds: props.vpc.privateSubnets.map((subnet) => subnet.subnetId),
      }
    );

    // Redis Cluster
    this.redis = new elasticache.CfnCacheCluster(this, "RedisCluster", {
      engine: "redis",
      cacheNodeType: props.cacheNodeType || "cache.t3.micro",
      numCacheNodes: props.numCacheNodes || 1,
      vpcSecurityGroupIds: [this.redisSecurityGroup.securityGroupId],
      cacheSubnetGroupName: subnetGroup.ref,
      engineVersion: "7.0",
      autoMinorVersionUpgrade: true,
    });

    this.redis.addDependency(subnetGroup);

    // Update Lambda with VPC and Redis endpoint
    const lambdaWithRedis = new lambda.Function(this, "HandlerWithRedis", {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: "index.handler",
      code: lambda.Code.fromAsset("lambda/dist"),
      environment: {
        TABLE_NAME: this.table.tableName,
        REDIS_ENDPOINT: this.redis.attrRedisEndpointAddress,
        REDIS_PORT: this.redis.attrRedisEndpointPort,
        ...props.environment,
      },
      vpc: props.vpc,
      vpcSubnets: { subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS },
      securityGroups: [lambdaSecurityGroup],
      timeout: cdk.Duration.seconds(30),
      memorySize: 1024,
    });

    this.table.grantReadWriteData(lambdaWithRedis);

    // Update handler reference
    this.handler = lambdaWithRedis;
  }
}
```

**Lambda Handler with Redis:**

```typescript
// lambda/handlers/products-with-redis.ts
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, GetCommand } from "@aws-sdk/lib-dynamodb";
import { createClient } from "redis";

// Redis client (reused across invocations)
let redisClient: ReturnType<typeof createClient> | null = null;

async function getRedisClient() {
  if (!redisClient) {
    redisClient = createClient({
      socket: {
        host: process.env.REDIS_ENDPOINT,
        port: parseInt(process.env.REDIS_PORT || "6379"),
      },
    });

    await redisClient.connect();

    redisClient.on("error", (err) => {
      console.error("Redis error:", err);
      redisClient = null;
    });
  }

  return redisClient;
}

const dynamoClient = DynamoDBDocumentClient.from(
  new DynamoDBClient({ region: process.env.AWS_REGION })
);

export async function handler(event: any) {
  const { productId } = event.pathParameters;
  const cacheKey = `product:${productId}`;
  const TTL = 300; // 5 minutes

  try {
    const redis = await getRedisClient();

    // Try cache first
    const cached = await redis.get(cacheKey);
    if (cached) {
      console.log("Cache HIT (Redis)");
      return {
        statusCode: 200,
        headers: {
          "Content-Type": "application/json",
          "X-Cache": "HIT-Redis",
        },
        body: cached,
      };
    }

    // Cache miss - fetch from DynamoDB
    console.log("Cache MISS (Redis)");
    const result = await dynamoClient.send(
      new GetCommand({
        TableName: process.env.TABLE_NAME,
        Key: { PK: `PRODUCT#${productId}`, SK: "METADATA" },
      })
    );

    const product = result.Item;

    if (!product) {
      return {
        statusCode: 404,
        body: JSON.stringify({ error: "Product not found" }),
      };
    }

    // Store in cache with TTL
    await redis.setEx(cacheKey, TTL, JSON.stringify(product));

    return {
      statusCode: 200,
      headers: {
        "Content-Type": "application/json",
        "X-Cache": "MISS",
      },
      body: JSON.stringify(product),
    };
  } catch (error) {
    console.error("Error:", error);

    // Fallback to DynamoDB if Redis fails
    const result = await dynamoClient.send(
      new GetCommand({
        TableName: process.env.TABLE_NAME,
        Key: { PK: `PRODUCT#${productId}`, SK: "METADATA" },
      })
    );

    return {
      statusCode: 200,
      headers: {
        "Content-Type": "application/json",
        "X-Cache": "BYPASS",
      },
      body: JSON.stringify(result.Item),
    };
  }
}
```

### 4. DynamoDB with DAX

```typescript
// lib/constructs/dax-cached-service.ts
import * as cdk from "aws-cdk-lib";
import * as dax from "aws-cdk-lib/aws-dax";
import * as ec2 from "aws-cdk-lib/aws-ec2";
import * as iam from "aws-cdk-lib/aws-iam";
import { Construct } from "constructs";
import { ApiService, ApiServiceProps } from "./api-service";

export interface DaxCachedServiceProps extends ApiServiceProps {
  vpc: ec2.IVpc;
  nodeType?: string;
  replicationFactor?: number;
}

export class DaxCachedService extends ApiService {
  public readonly daxCluster: dax.CfnCluster;

  constructor(scope: Construct, id: string, props: DaxCachedServiceProps) {
    super(scope, id, props);

    // IAM Role for DAX
    const daxRole = new iam.Role(this, "DaxRole", {
      assumedBy: new iam.ServicePrincipal("dax.amazonaws.com"),
    });

    this.table.grantReadWriteData(daxRole);

    // Security Group for DAX
    const daxSecurityGroup = new ec2.SecurityGroup(this, "DaxSecurityGroup", {
      vpc: props.vpc,
      description: "Security group for DAX cluster",
    });

    // Lambda Security Group
    const lambdaSecurityGroup = new ec2.SecurityGroup(
      this,
      "LambdaSecurityGroup",
      {
        vpc: props.vpc,
        description: "Security group for Lambda",
      }
    );

    daxSecurityGroup.addIngressRule(
      lambdaSecurityGroup,
      ec2.Port.tcp(8111),
      "Allow Lambda to access DAX"
    );

    // Subnet Group
    const subnetGroup = new dax.CfnSubnetGroup(this, "DaxSubnetGroup", {
      subnetIds: props.vpc.privateSubnets.map((subnet) => subnet.subnetId),
      description: "Subnet group for DAX",
    });

    // DAX Cluster
    this.daxCluster = new dax.CfnCluster(this, "DaxCluster", {
      iamRoleArn: daxRole.roleArn,
      nodeType: props.nodeType || "dax.t3.small",
      replicationFactor: props.replicationFactor || 1,
      subnetGroupName: subnetGroup.ref,
      securityGroupIds: [daxSecurityGroup.securityGroupId],
      clusterEndpointEncryptionType: "TLS",
    });

    // Update Lambda
    this.handler.addEnvironment(
      "DAX_ENDPOINT",
      this.daxCluster.attrClusterDiscoveryEndpoint
    );
  }
}
```

**Lambda Handler with DAX:**

```typescript
// lambda/handlers/products-with-dax.ts
import AmazonDaxClient from "amazon-dax-client";
import { DynamoDBDocumentClient, GetCommand } from "@aws-sdk/lib-dynamodb";

// DAX client (reused across invocations)
let daxClient: DynamoDBDocumentClient | null = null;

function getDaxClient() {
  if (!daxClient) {
    const dax = new AmazonDaxClient({
      endpoints: [process.env.DAX_ENDPOINT!],
      region: process.env.AWS_REGION,
    });
    daxClient = DynamoDBDocumentClient.from(dax);
  }
  return daxClient;
}

export async function handler(event: any) {
  const { productId } = event.pathParameters;

  const client = getDaxClient();

  // DAX automatically caches reads
  const result = await client.send(
    new GetCommand({
      TableName: process.env.TABLE_NAME,
      Key: { PK: `PRODUCT#${productId}`, SK: "METADATA" },
    })
  );

  return {
    statusCode: 200,
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(result.Item),
  };
}
```

### 5. Complete Caching Stack with Invalidation

```typescript
// lib/constructs/intelligent-cache-service.ts
import * as cdk from "aws-cdk-lib";
import * as dynamodb from "aws-cdk-lib/aws-dynamodb";
import * as lambda from "aws-cdk-lib/aws-lambda";
import * as lambdaEventSources from "aws-cdk-lib/aws-lambda-event-sources";
import * as ec2 from "aws-cdk-lib/aws-ec2";
import * as elasticache from "aws-cdk-lib/aws-elasticache";
import { Construct } from "constructs";

export interface IntelligentCacheServiceProps {
  serviceName: string;
  vpc: ec2.IVpc;
}

export class IntelligentCacheService extends Construct {
  public readonly table: dynamodb.Table;
  public readonly redis: elasticache.CfnCacheCluster;
  public readonly readHandler: lambda.Function;
  public readonly writeHandler: lambda.Function;
  public readonly invalidationHandler: lambda.Function;

  constructor(
    scope: Construct,
    id: string,
    props: IntelligentCacheServiceProps
  ) {
    super(scope, id);

    // DynamoDB Table with Streams
    this.table = new dynamodb.Table(this, "Table", {
      partitionKey: { name: "PK", type: dynamodb.AttributeType.STRING },
      sortKey: { name: "SK", type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
      stream: dynamodb.StreamViewType.NEW_AND_OLD_IMAGES,
      timeToLiveAttribute: "TTL",
    });

    // Redis setup (reuse from previous example)
    const redisSecurityGroup = new ec2.SecurityGroup(
      this,
      "RedisSecurityGroup",
      {
        vpc: props.vpc,
        description: "Redis security group",
      }
    );

    const lambdaSecurityGroup = new ec2.SecurityGroup(
      this,
      "LambdaSecurityGroup",
      {
        vpc: props.vpc,
        description: "Lambda security group",
      }
    );

    redisSecurityGroup.addIngressRule(
      lambdaSecurityGroup,
      ec2.Port.tcp(6379),
      "Allow Lambda to Redis"
    );

    const subnetGroup = new elasticache.CfnSubnetGroup(this, "SubnetGroup", {
      description: "Redis subnet group",
      subnetIds: props.vpc.privateSubnets.map((s) => s.subnetId),
    });

    this.redis = new elasticache.CfnCacheCluster(this, "Redis", {
      engine: "redis",
      cacheNodeType: "cache.t3.micro",
      numCacheNodes: 1,
      vpcSecurityGroupIds: [redisSecurityGroup.securityGroupId],
      cacheSubnetGroupName: subnetGroup.ref,
    });

    const commonEnv = {
      TABLE_NAME: this.table.tableName,
      REDIS_ENDPOINT: this.redis.attrRedisEndpointAddress,
      REDIS_PORT: this.redis.attrRedisEndpointPort,
    };

    const commonProps = {
      runtime: lambda.Runtime.NODEJS_20_X,
      vpc: props.vpc,
      vpcSubnets: { subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS },
      securityGroups: [lambdaSecurityGroup],
      timeout: cdk.Duration.seconds(30),
      memorySize: 1024,
    };

    // Read Handler (with caching)
    this.readHandler = new lambda.Function(this, "ReadHandler", {
      ...commonProps,
      handler: "read.handler",
      code: lambda.Code.fromAsset("lambda/dist"),
      environment: commonEnv,
    });

    this.table.grantReadData(this.readHandler);

    // Write Handler (with cache invalidation)
    this.writeHandler = new lambda.Function(this, "WriteHandler", {
      ...commonProps,
      handler: "write.handler",
      code: lambda.Code.fromAsset("lambda/dist"),
      environment: commonEnv,
    });

    this.table.grantReadWriteData(this.writeHandler);

    // Stream-based Cache Invalidation Handler
    this.invalidationHandler = new lambda.Function(
      this,
      "InvalidationHandler",
      {
        ...commonProps,
        handler: "invalidate.handler",
        code: lambda.Code.fromAsset("lambda/dist"),
        environment: commonEnv,
      }
    );

    this.table.grantStreamRead(this.invalidationHandler);

    this.invalidationHandler.addEventSource(
      new lambdaEventSources.DynamoEventSource(this.table, {
        startingPosition: lambda.StartingPosition.LATEST,
        batchSize: 100,
        retryAttempts: 3,
      })
    );
  }
}
```

**Cache Invalidation Handler:**

```typescript
// lambda/handlers/invalidate.ts
import { DynamoDBStreamEvent } from "aws-lambda";
import { createClient } from "redis";

let redisClient: ReturnType<typeof createClient> | null = null;

async function getRedisClient() {
  if (!redisClient) {
    redisClient = createClient({
      socket: {
        host: process.env.REDIS_ENDPOINT,
        port: parseInt(process.env.REDIS_PORT || "6379"),
      },
    });
    await redisClient.connect();
  }
  return redisClient;
}

export async function handler(event: DynamoDBStreamEvent) {
  const redis = await getRedisClient();

  for (const record of event.Records) {
    if (record.eventName === "MODIFY" || record.eventName === "REMOVE") {
      const pk = record.dynamodb?.Keys?.PK?.S;
      const sk = record.dynamodb?.Keys?.SK?.S;

      if (pk && sk) {
        // Invalidate specific cache entry
        const cacheKey = `${pk}:${sk}`;
        await redis.del(cacheKey);
        console.log(`Invalidated cache for: ${cacheKey}`);

        // Optionally invalidate related patterns
        if (pk.startsWith("PRODUCT#")) {
          const productId = pk.split("#")[1];
          await redis.del(`product:${productId}`);
          await redis.del(`product_list:*`); // Invalidate list caches
        }
      }
    }
  }

  return { statusCode: 200 };
}
```

**Write Handler with Cache-Through:**

```typescript
// lambda/handlers/write.ts
import { DynamoDBDocumentClient, PutCommand } from "@aws-sdk/lib-dynamodb";
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { createClient } from "redis";

let redisClient: ReturnType<typeof createClient> | null = null;
const dynamoClient = DynamoDBDocumentClient.from(new DynamoDBClient({}));

async function getRedisClient() {
  if (!redisClient) {
    redisClient = createClient({
      socket: {
        host: process.env.REDIS_ENDPOINT,
        port: parseInt(process.env.REDIS_PORT || "6379"),
      },
    });
    await redisClient.connect();
  }
  return redisClient;
}

export async function handler(event: any) {
  const product = JSON.parse(event.body);
  const { productId } = product;

  // Write to DynamoDB
  await dynamoClient.send(
    new PutCommand({
      TableName: process.env.TABLE_NAME,
      Item: {
        PK: `PRODUCT#${productId}`,
        SK: "METADATA",
        ...product,
        updatedAt: new Date().toISOString(),
      },
    })
  );

  try {
    const redis = await getRedisClient();

    // Write-through: Update cache immediately
    const cacheKey = `product:${productId}`;
    await redis.setEx(cacheKey, 300, JSON.stringify(product));

    // Invalidate related caches
    const keys = await redis.keys("product_list:*");
    if (keys.length > 0) {
      await redis.del(keys);
    }
  } catch (error) {
    console.error("Cache update failed:", error);
    // Continue anyway - cache will be populated on next read
  }

  return {
    statusCode: 200,
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ success: true, productId }),
  };
}
```

## SvelteKit Integration

### 1. Client-Side Fetch with Stale-While-Revalidate

```typescript
// src/lib/api/client.ts
interface CacheEntry<T> {
  data: T;
  timestamp: number;
}

class APIClient {
  private cache = new Map<string, CacheEntry<any>>();
  private inflight = new Map<string, Promise<any>>();

  async fetch<T>(
    url: string,
    options: {
      ttl?: number;
      swr?: number; // stale-while-revalidate window
      bypassCache?: boolean;
    } = {}
  ): Promise<T> {
    const { ttl = 60000, swr = 30000, bypassCache = false } = options;
    const cacheKey = url;
    const now = Date.now();

    // Check cache
    const cached = this.cache.get(cacheKey);

    if (cached && !bypassCache) {
      const age = now - cached.timestamp;

      // Fresh cache hit
      if (age < ttl) {
        return cached.data;
      }

      // Stale-while-revalidate
      if (age < ttl + swr) {
        // Return stale data immediately
        const staleData = cached.data;

        // Revalidate in background (if not already revalidating)
        if (!this.inflight.has(cacheKey)) {
          this.revalidate<T>(cacheKey, url);
        }

        return staleData;
      }
    }

    // No cache or too stale - fetch fresh
    return this.fetchFresh<T>(cacheKey, url);
  }

  private async fetchFresh<T>(cacheKey: string, url: string): Promise<T> {
    // Deduplicate inflight requests
    const existing = this.inflight.get(cacheKey);
    if (existing) {
      return existing;
    }

    const promise = fetch(url)
      .then((res) => res.json())
      .then((data) => {
        this.cache.set(cacheKey, { data, timestamp: Date.now() });
        this.inflight.delete(cacheKey);
        return data;
      })
      .catch((err) => {
        this.inflight.delete(cacheKey);
        throw err;
      });

    this.inflight.set(cacheKey, promise);
    return promise;
  }

  private async revalidate<T>(cacheKey: string, url: string): Promise<void> {
    const promise = fetch(url)
      .then((res) => res.json())
      .then((data) => {
        this.cache.set(cacheKey, { data, timestamp: Date.now() });
        this.inflight.delete(cacheKey);
      })
      .catch((err) => {
        console.error("Revalidation failed:", err);
        this.inflight.delete(cacheKey);
      });

    this.inflight.set(cacheKey, promise);
  }

  invalidate(pattern?: string | RegExp): void {
    if (!pattern) {
      this.cache.clear();
      return;
    }

    const regex = typeof pattern === "string" ? new RegExp(pattern) : pattern;

    for (const key of this.cache.keys()) {
      if (regex.test(key)) {
        this.cache.delete(key);
      }
    }
  }
}

export const apiClient = new APIClient();
```

**Usage in SvelteKit:**

```typescript
// src/routes/products/[id]/+page.ts
import type { PageLoad } from "./$types";
import { apiClient } from "$lib/api/client";

export const load: PageLoad = async ({ params, fetch }) => {
  const product = await apiClient.fetch(`/api/products/${params.id}`, {
    ttl: 60000, // Fresh for 1 minute
    swr: 30000, // Serve stale for additional 30 seconds while revalidating
  });

  return { product };
};
```

### 2. Optimistic Updates with Cache Invalidation

```svelte
<!-- src/routes/products/[id]/+page.svelte -->
<script lang="ts">
  import { invalidate } from '$app/navigation';
  import { apiClient } from '$lib/api/client';
  import type { PageData } from './$types';

  export let data: PageData;

  let isUpdating = false;

  async function updateProduct(updates: Partial<Product>) {
    isUpdating = true;

    try {
      // Optimistic update
      data.product = { ...data.product, ...updates };

      // Send update to backend
      const response = await fetch(`/api/products/${data.product.id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(updates),
      });

      if (!response.ok) throw new Error('Update failed');

      // Invalidate caches
      apiClient.invalidate(/products/);

      // Revalidate SvelteKit load function
      await invalidate('products');

    } catch (error) {
      console.error('Update failed:', error);

      // Revert optimistic update by reloading
      await invalidate('products');
    } finally {
      isUpdating = false;
    }
  }
</script>

<div>
  <h1>{data.product.name}</h1>
  <p>{data.product.description}</p>

  <button
    on:click={() => updateProduct({ featured: !data.product.featured })}
    disabled={isUpdating}
  >
    Toggle Featured
  </button>
</div>
```

### 3. Server-Side Caching with Dependency Tracking

```typescript
// src/routes/products/+page.server.ts
import type { PageServerLoad } from "./$types";
import { SmartCache } from "$lib/server/cache";

const productCache = new SmartCache<Product[]>(300);

export const load: PageServerLoad = async ({ depends, setHeaders }) => {
  // Register dependencies for invalidation
  depends("products");

  const cacheKey = "products_all";
  const cached = productCache.get(cacheKey);

  if (cached) {
    setHeaders({
      "Cache-Control": "public, max-age=300",
      ETag: cached.etag,
      "X-Cache": "HIT",
    });
    return { products: cached.data };
  }

  // Fetch from API
  const products = await fetchProductsFromAPI();
  const etag = productCache.set(cacheKey, products);

  setHeaders({
    "Cache-Control": "public, max-age=300",
    ETag: etag,
    "X-Cache": "MISS",
  });

  return { products };
};
```

```typescript
// src/routes/products/+server.ts (API endpoint for mutations)
import { json } from "@sveltejs/kit";
import type { RequestHandler } from "./$types";
import { productCache } from "$lib/server/cache";

export const POST: RequestHandler = async ({ request }) => {
  const product = await request.json();

  // Save to database via AWS API
  await saveProductToDB(product);

  // Invalidate cache
  productCache.invalidatePattern(/^products/);

  return json({ success: true }, { status: 201 });
};
```

## Monitoring & Tuning

### 1. Cache Metrics to Track

```typescript
// lib/monitoring/cache-metrics.ts
import * as cloudwatch from "aws-cdk-lib/aws-cloudwatch";
import { Construct } from "constructs";

export class CacheMetricsDashboard extends Construct {
  constructor(scope: Construct, id: string, serviceName: string) {
    super(scope, id);

    new cloudwatch.Dashboard(this, "Dashboard", {
      dashboardName: `${serviceName}-cache-metrics`,
      widgets: [
        [
          // Cache hit rate
          new cloudwatch.GraphWidget({
            title: "Cache Hit Rate",
            left: [
              new cloudwatch.Metric({
                namespace: "Custom/Cache",
                metricName: "CacheHit",
                statistic: "Sum",
                label: "Hits",
              }),
              new cloudwatch.Metric({
                namespace: "Custom/Cache",
                metricName: "CacheMiss",
                statistic: "Sum",
                label: "Misses",
              }),
            ],
          }),

          // Cache latency
          new cloudwatch.GraphWidget({
            title: "Cache Latency",
            left: [
              new cloudwatch.Metric({
                namespace: "Custom/Cache",
                metricName: "CacheLatency",
                statistic: "Average",
                label: "Avg",
              }),
              new cloudwatch.Metric({
                namespace: "Custom/Cache",
                metricName: "CacheLatency",
                statistic: "p95",
                label: "P95",
              }),
            ],
          }),
        ],
        [
          // Redis memory usage
          new cloudwatch.GraphWidget({
            title: "Redis Memory Usage",
            left: [
              new cloudwatch.Metric({
                namespace: "AWS/ElastiCache",
                metricName: "DatabaseMemoryUsagePercentage",
                statistic: "Average",
              }),
            ],
          }),

          // Evictions
          new cloudwatch.GraphWidget({
            title: "Cache Evictions",
            left: [
              new cloudwatch.Metric({
                namespace: "AWS/ElastiCache",
                metricName: "Evictions",
                statistic: "Sum",
              }),
            ],
          }),
        ],
      ],
    });
  }
}
```

### 2. Custom Metrics in Lambda

```typescript
// lambda/lib/metrics.ts
import {
  CloudWatchClient,
  PutMetricDataCommand,
} from "@aws-sdk/client-cloudwatch";

const cloudwatch = new CloudWatchClient({});

export async function recordCacheHit(cacheType: string): Promise<void> {
  await cloudwatch.send(
    new PutMetricDataCommand({
      Namespace: "Custom/Cache",
      MetricData: [
        {
          MetricName: "CacheHit",
          Value: 1,
          Unit: "Count",
          Dimensions: [
            {
              Name: "CacheType",
              Value: cacheType,
            },
          ],
          Timestamp: new Date(),
        },
      ],
    })
  );
}

export async function recordCacheMiss(cacheType: string): Promise<void> {
  await cloudwatch.send(
    new PutMetricDataCommand({
      Namespace: "Custom/Cache",
      MetricData: [
        {
          MetricName: "CacheMiss",
          Value: 1,
          Unit: "Count",
          Dimensions: [
            {
              Name: "CacheType",
              Value: cacheType,
            },
          ],
          Timestamp: new Date(),
        },
      ],
    })
  );
}

export async function recordCacheLatency(
  cacheType: string,
  latency: number
): Promise<void> {
  await cloudwatch.send(
    new PutMetricDataCommand({
      Namespace: "Custom/Cache",
      MetricData: [
        {
          MetricName: "CacheLatency",
          Value: latency,
          Unit: "Milliseconds",
          Dimensions: [
            {
              Name: "CacheType",
              Value: cacheType,
            },
          ],
          Timestamp: new Date(),
        },
      ],
    })
  );
}
```

**Usage:**

```typescript
// lambda/handlers/products-with-metrics.ts
import {
  recordCacheHit,
  recordCacheMiss,
  recordCacheLatency,
} from "../lib/metrics";

export async function handler(event: any) {
  const startTime = Date.now();

  const redis = await getRedisClient();
  const cached = await redis.get(cacheKey);

  if (cached) {
    await recordCacheHit("Redis");
    await recordCacheLatency("Redis", Date.now() - startTime);
    // ... return cached data
  } else {
    await recordCacheMiss("Redis");
    // ... fetch from database
  }
}
```

### 3. Alarms for Cache Health

```typescript
// lib/monitoring/cache-alarms.ts
import * as cloudwatch from "aws-cdk-lib/aws-cloudwatch";
import * as sns from "aws-cdk-lib/aws-sns";
import * as actions from "aws-cdk-lib/aws-cloudwatch-actions";
import { Construct } from "constructs";

export interface CacheAlarmsProps {
  redisClusterId: string;
  alarmTopic: sns.ITopic;
}

export class CacheAlarms extends Construct {
  constructor(scope: Construct, id: string, props: CacheAlarmsProps) {
    super(scope, id);

    // High memory usage alarm
    const memoryAlarm = new cloudwatch.Alarm(this, "HighMemoryUsage", {
      metric: new cloudwatch.Metric({
        namespace: "AWS/ElastiCache",
        metricName: "DatabaseMemoryUsagePercentage",
        dimensionsMap: {
          CacheClusterId: props.redisClusterId,
        },
        statistic: "Average",
      }),
      threshold: 80,
      evaluationPeriods: 2,
      comparisonOperator: cloudwatch.ComparisonOperator.GREATER_THAN_THRESHOLD,
      alarmDescription: "Redis memory usage is above 80%",
    });

    memoryAlarm.addAlarmAction(new actions.SnsAction(props.alarmTopic));

    // Low cache hit rate alarm
    const hitRateAlarm = new cloudwatch.Alarm(this, "LowHitRate", {
      metric: new cloudwatch.MathExpression({
        expression: "hits / (hits + misses) * 100",
        usingMetrics: {
          hits: new cloudwatch.Metric({
            namespace: "Custom/Cache",
            metricName: "CacheHit",
            statistic: "Sum",
          }),
          misses: new cloudwatch.Metric({
            namespace: "Custom/Cache",
            metricName: "CacheMiss",
            statistic: "Sum",
          }),
        },
      }),
      threshold: 50,
      evaluationPeriods: 3,
      comparisonOperator: cloudwatch.ComparisonOperator.LESS_THAN_THRESHOLD,
      alarmDescription: "Cache hit rate is below 50%",
    });

    hitRateAlarm.addAlarmAction(new actions.SnsAction(props.alarmTopic));

    // High eviction rate alarm
    const evictionAlarm = new cloudwatch.Alarm(this, "HighEvictions", {
      metric: new cloudwatch.Metric({
        namespace: "AWS/ElastiCache",
        metricName: "Evictions",
        dimensionsMap: {
          CacheClusterId: props.redisClusterId,
        },
        statistic: "Sum",
      }),
      threshold: 1000,
      evaluationPeriods: 1,
      comparisonOperator: cloudwatch.ComparisonOperator.GREATER_THAN_THRESHOLD,
      alarmDescription: "High cache eviction rate detected",
    });

    evictionAlarm.addAlarmAction(new actions.SnsAction(props.alarmTopic));
  }
}
```

## Decision Matrix & Tuning Guide

### Quick Decision Matrix

| Scenario                     | Recommended Solution   | TTL                 | Notes                       |
| ---------------------------- | ---------------------- | ------------------- | --------------------------- |
| Static assets                | CDN + Browser cache    | 1 year              | Use versioned URLs          |
| Product catalog (read-heavy) | Redis + Browser cache  | 5-15 min            | Stream-based invalidation   |
| User profiles                | Lambda memory + Redis  | 5 min               | Invalidate on update        |
| Search results               | API Gateway cache      | 1-5 min             | Query param-based caching   |
| Shopping cart                | Direct DynamoDB        | None                | Critical accuracy           |
| Session data                 | Redis                  | Session lifetime    | Use session TTL             |
| Real-time inventory          | DynamoDB + DAX         | 10-30 sec           | Consider strong consistency |
| Analytics dashboard          | Redis + longer TTL     | 15-30 min           | Acceptable staleness        |
| User feed                    | Stale-while-revalidate | 2-5 min + 1 min SWR | Balance UX and freshness    |

### Tuning Guidelines

**1. Start Conservative:**

```typescript
const INITIAL_CONFIG = {
  ttl: 60, // 1 minute
  swr: 30, // 30 seconds
  cacheSize: "small",
};
```

**2. Measure and Adjust:**

- Monitor cache hit rate (target: >70%)
- Track P95 latency (target: <100ms)
- Watch memory usage (keep <80%)
- Check eviction rate (minimize)

**3. Optimization Process:**

```
Week 1: Deploy with conservative settings
Week 2: Analyze metrics, identify hot paths
Week 3: Increase TTL for stable data
Week 4: Add additional cache layers for bottlenecks
Ongoing: Monitor and adjust based on usage patterns
```

## Cost Optimization

### Calculate Cache ROI

```typescript
// Cost calculation helper
interface CacheCostAnalysis {
  cacheCost: number; // Monthly cache cost
  savedRequests: number; // Requests avoided
  savedCost: number; // DynamoDB cost saved
  netSavings: number; // Net benefit
}

function analyzeCacheCost(
  cacheHitRate: number, // e.g., 0.80 for 80%
  totalRequests: number, // Monthly requests
  dynamoReadCost: number = 0.25, // per million reads
  cacheCostPerMonth: number = 15 // e.g., t3.micro ElastiCache
): CacheCostAnalysis {
  const savedRequests = totalRequests * cacheHitRate;
  const savedCost = (savedRequests / 1000000) * dynamoReadCost;
  const netSavings = savedCost - cacheCostPerMonth;

  return {
    cacheCost: cacheCostPerMonth,
    savedRequests,
    savedCost,
    netSavings,
  };
}

// Example
const result = analyzeCacheCost(0.8, 10_000_000, 0.25, 15);
console.log(`Net monthly savings: $${result.netSavings.toFixed(2)}`);
// Output: Net monthly savings: $-13.00 (not worth it at this scale)

const betterResult = analyzeCacheCost(0.8, 100_000_000, 0.25, 15);
console.log(`Net monthly savings: $${betterResult.netSavings.toFixed(2)}`);
// Output: Net monthly savings: $5.00 (marginally worth it)
```

### Cost-Effective Strategies

1. **Use Lambda memory cache first** (free within Lambda)
2. **Add Redis only when hit rate justifies cost** (>70% hit rate on high volume)
3. **Use DynamoDB as cache** for moderate performance needs (cheaper than Redis)
4. **Enable API Gateway caching selectively** (only for expensive endpoints)
5. **Use browser caching aggressively** (free, reduces backend load)

## Complete Example: E-Commerce Product Service

This final example ties everything together:

### CDK Stack

```typescript
// lib/stacks/product-service-stack.ts
import * as cdk from "aws-cdk-lib";
import * as ec2 from "aws-cdk-lib/aws-ec2";
import { Construct } from "constructs";
import { IntelligentCacheService } from "../constructs/intelligent-cache-service";
import { CacheMetricsDashboard } from "../monitoring/cache-metrics";
import { CacheAlarms } from "../monitoring/cache-alarms";
import * as sns from "aws-cdk-lib/aws-sns";

export class ProductServiceStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // VPC
    const vpc = new ec2.Vpc(this, "VPC", {
      maxAzs: 2,
      natGateways: 1,
    });

    // Core service with caching
    const service = new IntelligentCacheService(this, "ProductService", {
      serviceName: "product-service",
      vpc,
    });

    // Monitoring
    new CacheMetricsDashboard(this, "Metrics", "product-service");

    const alarmTopic = new sns.Topic(this, "AlarmTopic");

    new CacheAlarms(this, "Alarms", {
      redisClusterId: service.redis.ref,
      alarmTopic,
    });

    // Outputs
    new cdk.CfnOutput(this, "TableName", {
      value: service.table.tableName,
    });

    new cdk.CfnOutput(this, "RedisEndpoint", {
      value: service.redis.attrRedisEndpointAddress,
    });
  }
}
```

### SvelteKit Frontend

```typescript
// src/routes/products/+page.svelte
<script lang="ts">
  import { onMount } from 'svelte';
  import { apiClient } from '$lib/api/client';
  import type { PageData } from './$types';

  export let data: PageData;

  let products = data.products;
  let loading = false;

  // Automatic refresh with stale-while-revalidate
  onMount(() => {
    const interval = setInterval(async () => {
      products = await apiClient.fetch('/api/products', {
        ttl: 60000,    // Fresh for 1 minute
        swr: 30000,    // Serve stale while revalidating
      });
    }, 60000); // Check every minute

    return () => clearInterval(interval);
  });
</script>

<div class="products">
  {#each products as product}
    <div class="product-card">
      <h3>{product.name}</h3>
      <p>{product.price}</p>
    </div>
  {/each}
</div>
```

## Summary Checklist

When implementing caching in your projects:

- [ ] **Classify your data** by freshness requirements
- [ ] **Start with browser caching** for static assets
- [ ] **Use SvelteKit load functions** for server-side caching
- [ ] **Add Lambda memory cache** for free performance boost
- [ ] **Consider Redis** only when traffic justifies cost
- [ ] **Implement cache invalidation** via DynamoDB Streams
- [ ] **Set up monitoring** for hit rate, latency, and evictions
- [ ] **Configure alarms** for cache health
- [ ] **Calculate ROI** before adding expensive cache layers
- [ ] **Use stale-while-revalidate** for better UX
- [ ] **Version static assets** for immutable caching
- [ ] **Implement ETag support** for conditional requests
- [ ] **Test cache behavior** under load
- [ ] **Document TTL decisions** for future reference

This guide should serve as a comprehensive reference for all your caching needs. Start simple, measure, and optimize based on real data!
