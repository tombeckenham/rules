# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
```bash
bun dev              # Start development server with Turbopack
bun build            # Build production app with Turbopack
bun start            # Start production server
```

### Code Quality
```bash
bun biome check .          # Run linter
bun biome format .         # Format code
bun biome check --write .  # Fix linting and formatting
```

### Supabase
```bash
bunx supabase start     # Start local Supabase
bunx supabase stop      # Stop local Supabase
bunx supabase db reset  # Reset database
bunx supabase status    # Check local services status
bun supabase:types     # Generate TypeScript types from database
```

**Note**: Database types (`src/lib/gen.types.ts`) are auto-generated:
- Generated automatically on `bun install` via postinstall hook
- Can be manually regenerated with `bun supabase:types`
- File is gitignored to ensure types always match local database schema
- Types are generated from your local Supabase instance (must be running)
- Use the convenience exports from `@/types/database` for cleaner imports

### TypeScript
```bash
bun tsc --noEmit      # Type check without building
```

## Project Architecture

### Core Design Principles
- **Backend-only database access**: All Supabase DB operations go through API routes. Never use Supabase client directly in components to avoid RLS complexity.
- **Anonymous-first**: Users can start creating without signup, then upgrade to save work via magic link or passkeys.


### Key Technical Decisions
1. **State Management**: Use TanStack Query for server state and reducers for complex UI state. Keep components pure.
2. **UI Components**: Use shadcn/ui components exclusively. Apply styling through theme variables, not inline Tailwind.
3. **Job Queue**: QStash handles all AI generation tasks asynchronously.
4. **Authentication**: Supabase Auth with only magic links and passkeys (no passwords).
5. **File Structure**: API routes handle all business logic and DB access. Components remain presentational.

### API Pattern
All API routes follow this structure:
1. Validate input (Zod schemas)
2. Check auth/team permissions
3. Execute business logic (DB operations only here)
4. Queue async work via QStash if needed
5. Return standardized response

### Technology Stack
- **Framework**: Next.js 15 with App Router and Turbopack
- **Database**: Supabase (PostgreSQL + Auth + Storage + Realtime)
- **Queue**: QStash (Upstash) for AI job management
- **Styling**: Tailwind CSS v4 with shadcn/ui
- **Formatting**: Biome for linting and formatting
- **AI Models**: Multiple providers (Fal.ai, Runway, Kling, etc.) via unified interface

### Import Alias
Use `@/*` to import from src directory:
```typescript
import { something } from '@/app/api/utils'
```

## Backend Development Guidelines

### When creating new features:
1. Start with API route in `/app/api/v1/[feature]`
2. All DB operations in API routes only
3. Use QStash for any AI generation or long-running tasks
4. Create TanStack Query hooks for data fetching
5. Build components with shadcn/ui only
6. Apply theme variables for styling, avoid inline Tailwind

### Style Stacks
The core innovation - JSON presets that maintain consistent style across different AI models:
- Stored per team in the styles library
- Auto-adapt to different model requirements
- Include persistent elements (characters, settings)
- Tradeable in marketplace (future feature)

### Frame System
Frames are the building blocks of sequences:
- Reference script sections
- Have thumbnail (image) and motion (video)
- Can include characters, audio, and VFX from team libraries
- AI adjusts frame boundaries when script changes


## Frontend Development Guidelines
### 1. Use as little React as possible

- Keep components small - less than 100 lines
- Views reference components
- Remove as much logic as possible from components and views
- Externalize functions rather and keep external functions _vanilla typescript_
- Vanilla typescript is easier to test, and package for other uses

### 2. Avoid using useEffect

- Avoid using useEffect to fetch data or initialise state
- Use hooks, tanstack query, or the new `use` feature in react 19
- use useEffect to update state when something else changes

### 3. Use useState sparingly

- Using a local variable is often more optimal - even if something has to be calculated
- useState uses reducers under the hood
- If you have more than 3 useStates you might as well use reducers

### 4. Use React.FC and expand props

- Expand props so that each parameter is named
- It's easier to know which props are not used

### 5. Avoid globals and global state

- Globals including auth globals, often lead to race conditions
- Global state is only useful in rare cases. SPAs with signifant complexity
- Start with reducers which are passed through props
- If that's too complicated, use React context
- As a last resort for very complicated SPAs - use zustand

### 6. Use reducers

- Reducers are great - read up on them and understand them
- They keep all state update logic in one place
- They are vanilla typescript - keep them that way
- Use them to update state based on other state.
- Note that updating any part of the state returned from a reducer will cause a re-render if the whole state is a parameter - you can pass just parts of it

### 7. Avoid adding styles on top of components

- Avoid passing styles to components, or styling your components in views
- Create components that are pre-styled
- Create variants - e.g. small, medium, large - rather than size={18}
- Avoid using styles in views as much as possible

### 8. Use the theme

- Create constants for your theme or use a consistent structure
- Use the theme fonts and colors
- Avoid naming fonts and inluding color values directly in views and components

### 9. Use flexbox

- Use flexbox for every component

### 10. Avoid margin

- Don't pre-add padding or margin to the outside of components, unless there is a specific reason to
- Use flexbox gap instead

### 11. Use kebab-case for file names

- Use PascalCase for component names, but kebab case for file-names
- PascalCase can often give you issues in git as case sensitive file names is not supported on all platforms

### 12. Create a component library

- If using a 3rd party component library, wrap those components then include your wrapped components. It makes it easier to change libraries
- Shadcn makes this easy - it includes the source in your repo, and imports only primitives from that source.
- You don't need to wrap Shadcn generated components - just edit the component source with your changes
- Don't create duplicates of components for minor variations. Create a variation
- Customise the component if there's a new variation - don't style from the outside
- Put your components into a high level components folder
- Use an aliases or subpath imports "~" to import components from views.

### 13. Avoid hard coding width or height

- Use flexbox and create rules for screen sizes

### 14. Use a show prop if you need to hide something

- Avoid code that conditionally shows a complex component - this creates janky ui
- Instead use the display css property to hide or show - this will precalculate everything in the component but not render it

### 15. Use eslint or equivalent

- Ensure the rules of hooks linting rule is on
- Check out 0xlint and Biome

### 16. Views are routes

- All views should be routable - meaning you can get to them via a route
- No view should rely on variables or parameters from another
- A view can be accessed in any order
- Pass params on the url. You can use url segments for ids, search params should be optional
- Name views with the same name as the route - or place in a folder with that name

### 17. Avoid default exports

- It's more efficient to export the component directly than to import a default
- Avoid barrelled imports as much as possible _unless_ you are planning to package that library for others

