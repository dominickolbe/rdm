# Next.js Migration Plan for Real Debrid Manager

This document outlines the comprehensive plan to migrate the Real Debrid Manager from SvelteKit to Next.js while maintaining all existing functionality.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Migration Strategy](#migration-strategy)
3. [Project Setup](#project-setup)
4. [Dependency Migration](#dependency-migration)
5. [File Structure Conversion](#file-structure-conversion)
6. [Component Migration](#component-migration)
7. [State Management](#state-management)
8. [API Routes Migration](#api-routes-migration)
9. [Styling and UI Components](#styling-and-ui-components)
10. [Authentication System](#authentication-system)
11. [Build and Deployment](#build-and-deployment)
12. [Testing](#testing)
13. [SEO and Meta Tags](#seo-and-meta-tags)
14. [Environment Variables](#environment-variables)
15. [Implementation Timeline](#implementation-timeline)

## Project Overview

The Real Debrid Manager is currently built with:
- **Framework**: SvelteKit with TypeScript
- **Styling**: Tailwind CSS
- **UI Components**: bits-ui, lucide-svelte, radix-icons-svelte
- **Build Tool**: Vite
- **Deployment**: Cloudflare (using @sveltejs/adapter-cloudflare)
- **Key Features**: 
  - Real Debrid API integration
  - Authentication system
  - Torrent and download management
  - Content scraper functionality
  - Dark/light mode support

## Migration Strategy

### Approach: Incremental Migration
1. **Phase 1**: Set up Next.js project structure and basic configuration
2. **Phase 2**: Migrate core components and layouts
3. **Phase 3**: Convert API routes and authentication
4. **Phase 4**: Migrate feature-specific pages and components
5. **Phase 5**: Testing, optimization, and deployment setup

### Key Principles
- Maintain existing functionality without breaking changes
- Preserve the current design and user experience
- Keep the same API endpoints for backward compatibility
- Maintain TypeScript throughout the migration

## Project Setup

### 1. Initialize Next.js Project
```bash
# Create new Next.js project with TypeScript and Tailwind
npx create-next-app@latest rdm-nextjs --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
cd rdm-nextjs
```

### 2. Configure TypeScript
```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/lib/*": ["./src/lib/*"],
      "@/app/*": ["./src/app/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 3. Configure Next.js
```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone', // For Cloudflare deployment
  images: {
    unoptimized: true // For static deployment
  },
  experimental: {
    serverComponentsExternalPackages: ['luxon']
  }
}

module.exports = nextConfig
```

## Dependency Migration

### Core Framework Dependencies
| SvelteKit | Next.js | Purpose |
|-----------|---------|---------|
| `@sveltejs/kit` | `next` | Framework |
| `@sveltejs/adapter-cloudflare` | `@cloudflare/next-on-pages` | Deployment |
| `svelte` | `react`, `react-dom` | UI Library |
| `vite` | *Built into Next.js* | Build Tool |

### UI Component Dependencies
| Current (Svelte) | Next.js Alternative | Purpose |
|------------------|-------------------|---------|
| `bits-ui` | `@radix-ui/react-*` | Headless UI components |
| `lucide-svelte` | `lucide-react` | Icons |
| `radix-icons-svelte` | `@radix-ui/react-icons` | Icons |
| `mode-watcher` | `next-themes` | Dark/light mode |
| `svelte-sonner` | `sonner` | Toast notifications |

### Utility Dependencies (Keep Same)
- `tailwindcss` ✓
- `clsx` ✓
- `tailwind-merge` ✓
- `tailwind-variants` ✓
- `fuse.js` ✓
- `luxon` ✓
- `@ctrl/video-filename-parser` ✓

### New Dependencies to Add
```bash
npm install next react react-dom
npm install @radix-ui/react-dropdown-menu @radix-ui/react-select @radix-ui/react-table
npm install @radix-ui/react-dialog @radix-ui/react-badge @radix-ui/react-button
npm install @radix-ui/react-separator @radix-ui/react-icons
npm install lucide-react next-themes sonner
npm install @types/react @types/react-dom
npm install @cloudflare/next-on-pages
```

## File Structure Conversion

### Current SvelteKit Structure → Next.js Structure

```
src/
├── routes/                 → src/app/
│   ├── (root)/            → src/app/(root)/
│   │   ├── +layout.svelte → layout.tsx
│   │   ├── +page.svelte   → page.tsx
│   │   └── changelog/     → changelog/
│   ├── app/               → src/app/app/
│   │   ├── +layout.svelte → layout.tsx
│   │   ├── +page.svelte   → page.tsx
│   │   ├── downloads/     → downloads/
│   │   ├── torrents/      → torrents/
│   │   ├── scraper/       → scraper/
│   │   └── settings/      → settings/
│   ├── api/               → src/app/api/
│   │   ├── login/         → login/route.ts
│   │   ├── logout/        → logout/route.ts
│   │   ├── user/          → user/route.ts
│   │   └── rd/            → rd/
│   └── +layout.svelte     → src/app/layout.tsx
├── lib/                   → src/lib/
│   ├── components/        → src/components/
│   ├── app/               → src/lib/app/
│   ├── store.ts           → src/lib/store.tsx (Context)
│   └── utils.ts           → src/lib/utils.ts
├── app.html               → src/app/layout.tsx (head content)
└── app.postcss            → src/app/globals.css
```

### Key Routing Changes
- SvelteKit `+page.svelte` → Next.js `page.tsx`
- SvelteKit `+layout.svelte` → Next.js `layout.tsx`
- SvelteKit `+page.server.ts` → Next.js `page.tsx` (server components)
- SvelteKit `+page.ts` → Next.js `page.tsx` (client components)
- API routes: `+server.ts` → `route.ts`

## Component Migration

### Layout Components

#### 1. Root Layout (`src/app/layout.tsx`)
```tsx
import type { Metadata } from 'next'
import { Inter, Poppins } from 'next/font/google'
import { ThemeProvider } from '@/components/theme-provider'
import { Toaster } from 'sonner'
import './globals.css'

const poppins = Poppins({ 
  subsets: ['latin'],
  weight: ['300', '400', '500', '600', '700', '800'],
  variable: '--font-poppins'
})

export const metadata: Metadata = {
  title: 'Real Debrid Manager',
  description: 'Effortlessly organize and control your Real Debrid torrents and downloads.',
  keywords: ['real debrid', 'rdm', 'torrents', 'downloads'],
  // ... other meta tags
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className={`${poppins.variable} font-poppins`}>
        <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
          {children}
          <Toaster richColors closeButton />
        </ThemeProvider>
      </body>
    </html>
  )
}
```

#### 2. App Layout (`src/app/app/layout.tsx`)
```tsx
import Header from '@/components/app/Header'

export default function AppLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div className="flex flex-col w-full font-poppins">
      <Header />
      {children}
    </div>
  )
}
```

### Component Conversion Examples

#### Svelte Component → React Component
```svelte
<!-- Current: Header.svelte -->
<script lang="ts">
  import { Button } from '$lib/components/ui/button'
  import { setMode, mode } from 'mode-watcher'
  import { Sun, Moon } from 'lucide-svelte'
  
  export let user: User
</script>

<header class="flex w-full items-center">
  {#if $mode === 'light'}
    <Button on:click={() => setMode('dark')}>
      <Sun class="h-4 w-4" />
    </Button>
  {:else}
    <Button on:click={() => setMode('light')}>
      <Moon class="h-4 w-4" />
    </Button>
  {/if}
</header>
```

```tsx
// Next.js: Header.tsx
'use client'

import { Button } from '@/components/ui/button'
import { useTheme } from 'next-themes'
import { Sun, Moon } from 'lucide-react'

interface HeaderProps {
  user: User
}

export default function Header({ user }: HeaderProps) {
  const { theme, setTheme } = useTheme()
  
  return (
    <header className="flex w-full items-center">
      {theme === 'light' ? (
        <Button onClick={() => setTheme('dark')}>
          <Sun className="h-4 w-4" />
        </Button>
      ) : (
        <Button onClick={() => setTheme('light')}>
          <Moon className="h-4 w-4" />
        </Button>
      )}
    </header>
  )
}
```

## State Management

### Migration from Svelte Stores to React Context

#### Current Svelte Store (`src/lib/store.ts`)
```typescript
import { writable } from 'svelte/store'

export const currentDownloadData = writable([])
export const userStore = writable(null)
```

#### New React Context (`src/lib/context/AppContext.tsx`)
```tsx
'use client'

import { createContext, useContext, useReducer, ReactNode } from 'react'

interface AppState {
  currentDownloadData: any[]
  user: User | null
}

interface AppContextType {
  state: AppState
  updateDownloads: (downloads: any[]) => void
  setUser: (user: User | null) => void
}

const AppContext = createContext<AppContextType | undefined>(undefined)

const initialState: AppState = {
  currentDownloadData: [],
  user: null
}

function appReducer(state: AppState, action: any): AppState {
  switch (action.type) {
    case 'UPDATE_DOWNLOADS':
      return { ...state, currentDownloadData: action.payload }
    case 'SET_USER':
      return { ...state, user: action.payload }
    default:
      return state
  }
}

export function AppProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(appReducer, initialState)
  
  const updateDownloads = (downloads: any[]) => {
    dispatch({ type: 'UPDATE_DOWNLOADS', payload: downloads })
  }
  
  const setUser = (user: User | null) => {
    dispatch({ type: 'SET_USER', payload: user })
  }
  
  return (
    <AppContext.Provider value={{ state, updateDownloads, setUser }}>
      {children}
    </AppContext.Provider>
  )
}

export function useApp() {
  const context = useContext(AppContext)
  if (!context) {
    throw new Error('useApp must be used within AppProvider')
  }
  return context
}
```

## API Routes Migration

### Current SvelteKit API Route → Next.js API Route

#### SvelteKit API Route (`src/routes/api/user/+server.ts`)
```typescript
import { json } from '@sveltejs/kit'
import type { RequestHandler } from './$types'

export const GET: RequestHandler = async ({ cookies }) => {
  const session = cookies.get('session')
  // ... logic
  return json({ user: userData })
}
```

#### Next.js API Route (`src/app/api/user/route.ts`)
```typescript
import { NextRequest, NextResponse } from 'next/server'
import { cookies } from 'next/headers'

export async function GET(request: NextRequest) {
  const cookieStore = cookies()
  const session = cookieStore.get('session')
  // ... logic
  return NextResponse.json({ user: userData })
}
```

### API Routes to Migrate
1. `/api/login` - Authentication
2. `/api/logout` - Logout
3. `/api/user` - User data
4. `/api/refresh` - Token refresh
5. `/api/rd/*` - Real Debrid API proxy routes
6. `/api/app/*` - Application data routes

## Styling and UI Components

### Tailwind Configuration
The existing Tailwind configuration can be largely reused:

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: ["class"],
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
    './src/**/*.{ts,tsx}',
  ],
  theme: {
    extend: {
      fontFamily: {
        poppins: ['var(--font-poppins)', 'sans-serif'],
      },
      // ... existing theme configuration
    },
  },
  plugins: [require("tailwindcss-animate")],
}
```

### UI Component Library Migration
Create shadcn/ui compatible components:

```bash
npx shadcn-ui@latest init
npx shadcn-ui@latest add button badge dropdown-menu table separator
```

## Authentication System

### Session Management
- Migrate from SvelteKit cookies to Next.js cookies
- Implement middleware for protected routes
- Create authentication context

#### Middleware (`src/middleware.ts`)
```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const session = request.cookies.get('session')
  
  if (request.nextUrl.pathname.startsWith('/app') && !session) {
    return NextResponse.redirect(new URL('/', request.url))
  }
  
  return NextResponse.next()
}

export const config = {
  matcher: ['/app/:path*']
}
```

## Build and Deployment

### Cloudflare Pages Deployment
```yaml
# .github/workflows/deploy.yml
name: Deploy to Cloudflare Pages
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run build
      - uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          projectName: rdm
          directory: out
```

### Build Configuration
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "export": "next build && next export"
  }
}
```

## Testing

### Test Migration Strategy
1. **Unit Tests**: Migrate Vitest tests to Jest/React Testing Library
2. **Integration Tests**: Update Playwright tests for new routing
3. **E2E Tests**: Minimal changes needed as functionality remains the same

#### Jest Configuration (`jest.config.js`)
```javascript
const nextJest = require('next/jest')

const createJestConfig = nextJest({
  dir: './',
})

const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  testEnvironment: 'jest-environment-jsdom',
}

module.exports = createJestConfig(customJestConfig)
```

## SEO and Meta Tags

### Dynamic Meta Tags
```tsx
// src/app/app/downloads/page.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Downloads - Real Debrid Manager',
  description: 'Manage your Real Debrid downloads',
}

export default function DownloadsPage() {
  return <div>Downloads</div>
}
```

### Structured Data Migration
Move the JSON-LD structured data from `app.html` to the root layout or use Next.js built-in structured data features.

## Environment Variables

### Environment Configuration
```bash
# .env.local
NEXT_PUBLIC_BASE_URI=https://api.real-debrid.com/rest/1.0
NEXT_PUBLIC_BASE_AUTH_URI=https://api.real-debrid.com
NEXT_PUBLIC_CLIENT_ID=X245A4XAIBGVM
NEXT_PUBLIC_TORRENTIO_BASE_URI=https://torrentio.strem.fun
```

## Implementation Timeline

### Phase 1: Foundation (Week 1)
- [ ] Set up Next.js project with TypeScript and Tailwind
- [ ] Configure build tools and deployment pipeline
- [ ] Set up basic project structure
- [ ] Install and configure core dependencies

### Phase 2: Core Components (Week 2)
- [ ] Migrate layout components (Header, Footer)
- [ ] Convert UI component library (buttons, forms, etc.)
- [ ] Implement theme provider and dark mode
- [ ] Set up state management with React Context

### Phase 3: Authentication & API (Week 3)
- [ ] Migrate authentication system
- [ ] Convert all API routes
- [ ] Implement middleware for protected routes
- [ ] Set up session management

### Phase 4: Feature Pages (Week 4)
- [ ] Migrate home page
- [ ] Convert torrents page and functionality
- [ ] Migrate downloads page
- [ ] Convert scraper functionality
- [ ] Migrate settings page

### Phase 5: Testing & Optimization (Week 5)
- [ ] Set up testing framework
- [ ] Write component tests
- [ ] Update E2E tests
- [ ] Performance optimization
- [ ] Final deployment configuration

### Phase 6: Deployment & Monitoring (Week 6)
- [ ] Deploy to staging environment
- [ ] User acceptance testing
- [ ] Production deployment
- [ ] Post-deployment monitoring and bug fixes

## Key Considerations

### Breaking Changes to Watch
1. **Server vs Client Components**: Carefully plan which components need 'use client'
2. **Routing**: Ensure all dynamic routes work correctly
3. **API Endpoints**: Maintain backward compatibility
4. **State Management**: Ensure proper hydration with SSR

### Performance Optimizations
1. **Image Optimization**: Use Next.js Image component where applicable
2. **Code Splitting**: Leverage automatic code splitting
3. **Caching**: Implement proper caching strategies
4. **Bundle Analysis**: Monitor bundle size during migration

### Migration Risks
1. **Data Loss**: Ensure no user data is lost during migration
2. **Downtime**: Plan for zero-downtime deployment
3. **Feature Parity**: Maintain all existing functionality
4. **SEO Impact**: Preserve SEO rankings and meta tags

## Conclusion

This migration plan provides a comprehensive roadmap for converting the Real Debrid Manager from SvelteKit to Next.js. The phased approach ensures minimal disruption while maintaining all existing functionality. Each phase builds upon the previous one, allowing for testing and validation at each step.

The migration will result in:
- Enhanced performance with Next.js optimizations
- Better React ecosystem integration
- Improved developer experience with mature tooling
- Maintained feature parity and user experience
- Preserved SEO and deployment capabilities

Following this plan systematically will ensure a successful migration while minimizing risks and maintaining the high quality of the Real Debrid Manager application.