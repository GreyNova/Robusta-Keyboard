# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Nimbus Keyboards is a Next.js 15 e-commerce site for custom mechanical keyboards. The project combines headless CMS (Prismic), 3D rendering (Three.js/React Three Fiber), advanced animations (GSAP), and Stripe payment integration.

## Commands

### Development
- **Start dev server**: `npm run dev` (uses Turbopack)
- **Build for production**: `npm run build`
- **Start production server**: `npm start`
- **Lint**: `npm run lint`

### Prismic CMS
- **Start Slice Machine**: `npm run slicemachine` (local editor for Prismic slices at http://localhost:3000/slice-simulator)

## Architecture

### Content Management (Prismic)
The project uses Prismic as a headless CMS with a "Slice-based" architecture:
- **Custom Types**: Defined in `customtypes/` (homepage, product, switch)
- **Slices**: Reusable content components in `src/slices/` that are composed in Prismic
  - `Hero` - Landing section with 3D keyboard scene
  - `BentoBox` - Feature grid layout
  - `ColorChanger` - Interactive keyboard color customization
  - `SwitchPlayground` - Interactive switch demo
  - `Marquee` - Scrolling content
  - `PurchaseButton` - Checkout integration
- **Client Setup**: `src/prismicio.ts` (currently commented out - needs configuration)
- **Content Rendering**: Pages fetch data from Prismic and render using `SliceZone`

### 3D Rendering Stack
Three.js integration via React Three Fiber (`@react-three/fiber`) and Drei (`@react-three/drei`):
- **Keyboard Component** (`src/components/Keyboard.tsx`): Complex 3D model with ~90 individual keycaps, each with its own ref for animations. Exposes grouped refs for rows (function, number, top, home, bottom) and modifiers.
- **Switch Component** (`src/components/Switch.tsx`): Interactive switch with GSAP animations and audio feedback (4 color variants: red, brown, blue, black)
- **Keycap Component** (`src/components/Keycap.tsx`): Individual keycap rendering
- **3D Assets**: Located in `public/` as `.gltf` files

### Animation System
GSAP (GreenSock) powers all animations:
- **Hero animations**: SplitText for character reveals, ScrollTrigger for parallax effects
- **Switch interactions**: Physics-based press/release animations with elastic easing
- **Purchase button**: Dynamic font-variation-settings animation based on mouse position
- **Utility components**: `FadeIn.tsx` for scroll-based reveals

### E-commerce Flow
Stripe integration for payments:
1. User clicks purchase button → `src/checkout.ts` function
2. Client-side POST to `/api/checkout/[uid]/route.ts`
3. API fetches product data from Prismic by UID
4. Creates Stripe Checkout session
5. Redirects to Stripe → success page at `/success`

**Environment Required**: `STRIPE_SECRET_KEY` environment variable

### Routing Structure
Next.js 15 App Router:
- `/` - Homepage (renders Prismic slices)
- `/slice-simulator` - Prismic development preview
- `/success` - Post-checkout page
- `/api/checkout/[uid]` - Stripe session creation
- `/api/preview`, `/api/exit-preview` - Prismic preview mode
- `/api/revalidate` - On-demand revalidation

### Styling
- **Tailwind CSS 4.x**: Utility-first styling
- **Custom Fonts**: Roboto Flex with variable axes (width, slant, optical size)
- **Path Alias**: `@/*` maps to `src/*`

### Component Patterns
- **Bounded** (`src/components/Bounded.tsx`): Layout wrapper with consistent padding/max-width
- **FadeIn** (`src/components/FadeIn.tsx`): Scroll-triggered fade-in animations
- **Loader** (`src/components/Loader.tsx`): 3D model loading state
- Client components (3D/animations) use `"use client"` directive

## Key Technical Details

### 3D Model Structure
The main keyboard GLTF contains:
- 90+ individual keycap meshes (K_ESC, K_A, K_SPACE, etc.)
- Instanced switch components (housing, stem)
- Plate, PCB, top case, weight, screen parts
- All organized for programmatic animation control

### GSAP Integration
- Uses `useGSAP` hook for proper cleanup
- `gsap.matchMedia()` respects `prefers-reduced-motion`
- Switch animations use `gsap.killTweensOf()` to prevent conflicts
- Font variations animated via CSS custom properties

### Prismic Integration
- Repository: "nimbus-keyboards" (per slicemachine.config.json)
- Slices are dynamically imported for code splitting
- Preview mode enabled via `PrismicPreview` component
- Static metadata generation from Prismic content

## Development Notes

- **3D Performance**: Models use instanced meshes where possible for performance
- **Audio**: Switch sounds are pre-loaded and volume-controlled (0.6)
- **Animations**: All interactive animations check for existing refs before executing
- **TypeScript**: Strict mode enabled with ES2017 target
- **Windows Development**: This project is developed on Windows (PowerShell environment)
