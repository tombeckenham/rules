# CLAUDE.md

## Commands

```bash
# Development
bun dev              # Start dev server
bun build            # Production build
bun start            # Start production server

# Code Quality
bunx oxlint .        # Lint
bunx oxfmt .         # Format

# Database
bun drizzle-kit push       # Push schema to database
bun drizzle-kit generate   # Generate migrations
bun drizzle-kit studio     # Open Drizzle Studio

# TypeScript
bun tsgo --noEmit    # Type check
```

## Tech Stack

- **Framework**: TanStack Start
- **Database**: Drizzle ORM + PostgreSQL
- **State**: TanStack Query
- **Auth**: Better Auth (magic links, passkeys only)
- **UI**: shadcn/ui + Tailwind v4 (use Field components for forms)
- **Toasts**: Sonner for form feedback
- **Quality**: oxlint + oxfmt

## Core Principles

- **Backend-only DB access**: All Drizzle queries go through API routes
- **Components are presentational**: No business logic in components
- **Anonymous-first**: Users create without signup, upgrade to save
- **QStash for async**: AI generation and long-running tasks

## TypeScript

- **Avoid casting** - no `as Type`, never `as any`
- **Infer types** - let TypeScript figure it out
- **Zod at boundaries** - validate external data at API edges
- **Nullish coalescing** - use `??` over `||`
- **Discriminated unions** - for type-safe state machines

## React Patterns

- **TanStack Query** for all server state
- **useTransition** for non-blocking UI updates
- **Local variables** over useState when possible
- **Suspense** for loading states
- **No useEffect for data** - use TanStack Query or `use()`

## Component Rules

- **< 100 lines** - extract logic to vanilla TS
- **Pre-styled with variants** - not style props
- **Flexbox + gap** - no margin on components
- **kebab-case files** - named exports only

---

## Examples

### Data Fetching

Shows: TanStack Query, no useEffect, no casting, Suspense

```tsx
// ❌ DON'T
function UserProfile({ id }: { id: string }) {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetch(`/api/users/${id}`)
      .then(res => res.json())
      .then(data => {
        setUser(data as User)  // casting
        setLoading(false)
      })
  }, [id])

  if (loading) return <Spinner />
  return <div>{user?.name}</div>
}

// ✅ DO
function UserProfile({ id }: { id: string }) {
  const { data: user } = useSuspenseQuery({
    queryKey: ['user', id],
    queryFn: () => getUser(id),  // returns typed User
  })

  return <div>{user.name}</div>
}
```

### Form Mutations

Shows: useTransition, TanStack mutations, Zod validation, Sonner toasts

```tsx
// ❌ DON'T
function EditName() {
  const [name, setName] = useState('')
  const [saving, setSaving] = useState(false)
  const [error, setError] = useState<string | null>(null)
  const [success, setSuccess] = useState(false)

  const handleSubmit = async () => {
    setSaving(true)
    setError(null)
    setSuccess(false)
    try {
      await fetch('/api/user', {
        method: 'PATCH',
        body: JSON.stringify({ name }),
      })
      setSuccess(true)
    } catch (e) {
      setError('Failed to save')
    }
    setSaving(false)
  }

  return (
    <form onSubmit={handleSubmit}>
      <Input value={name} onChange={e => setName(e.target.value)} />
      {error && <p className="text-red-500">{error}</p>}
      {success && <p className="text-green-500">Saved!</p>}
      <Button disabled={saving}>{saving ? 'Saving...' : 'Save'}</Button>
    </form>
  )
}

// ✅ DO - use shadcn Field components + Sonner toasts
function EditName() {
  const [isPending, startTransition] = useTransition()
  const { mutate } = useUpdateUser({
    onSuccess: () => toast.success('Name updated'),
    onError: (e) => toast.error(e.message),
  })

  const handleSubmit = (formData: FormData) => {
    const result = nameSchema.safeParse(formData.get('name'))
    if (!result.success) {
      toast.error('Invalid name')
      return
    }

    startTransition(() => {
      mutate({ name: result.data })
    })
  }

  return (
    <form action={handleSubmit}>
      <Field>
        <FieldLabel htmlFor="name">Name</FieldLabel>
        <Input id="name" name="name" />
        <FieldDescription>Your display name</FieldDescription>
      </Field>
      <Button disabled={isPending}>Save</Button>
    </form>
  )
}
```

### Component Styling

Shows: variants not props, theme vars, flexbox + gap, no margin

```tsx
// ❌ DON'T
function Card({ children, padding = 16, margin = 8 }: Props) {
  return (
    <div className="bg-[#1a1a1a] rounded-lg" style={{ padding, margin }}>
      {children}
    </div>
  )
}

// Usage with external styling
<div className="mt-4">
  <Card padding={24} margin={0}>{content}</Card>
</div>

// ✅ DO
function Card({ children, size = 'md' }: Props) {
  return (
    <div className={cn(
      'bg-card rounded-lg',
      size === 'sm' && 'p-2',
      size === 'md' && 'p-4',
      size === 'lg' && 'p-6',
    )}>
      {children}
    </div>
  )
}

// Parent controls spacing with gap
<div className="flex flex-col gap-4">
  <Card size="md">{content}</Card>
</div>
```

### Logic Extraction

Shows: vanilla TS functions, small components, type inference

```tsx
// ❌ DON'T - logic mixed in component
function PriceDisplay({ items }: { items: Item[] }) {
  const subtotal = items.reduce((sum, item) => sum + item.price * item.qty, 0)
  const tax = subtotal * 0.1
  const shipping = subtotal > 100 ? 0 : 10
  const total = subtotal + tax + shipping
  const formatted = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(total)

  return <span>{formatted}</span>
}

// ✅ DO - pure functions extracted
// lib/pricing.ts
export function calculateTotal(items: Item[]) {
  const subtotal = items.reduce((sum, item) => sum + item.price * item.qty, 0)
  const tax = subtotal * 0.1
  const shipping = subtotal > 100 ? 0 : 10
  return subtotal + tax + shipping
}

export function formatCurrency(amount: number) {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(amount)
}

// component
function PriceDisplay({ items }: { items: Item[] }) {
  const total = calculateTotal(items)
  return <span>{formatCurrency(total)}</span>
}
```

### File & Export Patterns

Shows: kebab-case files, named exports, inline skeletons

```
// ❌ DON'T
src/
  components/
    UserProfile.tsx          // PascalCase file
    UserProfileSkeleton.tsx  // separate skeleton
  pages/
    Dashboard.tsx            // doesn't match route

// UserProfile.tsx
export default function UserProfile() { ... }

// ✅ DO
src/
  components/
    user-profile.tsx         // kebab-case
  routes/
    dashboard.tsx            // route = /dashboard

// user-profile.tsx
export function UserProfile() {
  const { data, isLoading } = useUser()

  if (isLoading) {
    return <Skeleton className="h-20 w-full" />  // inline skeleton
  }

  return <div>{data.name}</div>
}
```

### API Routes

Shows: Zod validation, Drizzle queries, error handling

```ts
// ❌ DON'T
export async function POST(req: Request) {
  const body = await req.json()
  const user = body as CreateUserInput  // casting

  const result = await db.insert(users).values({
    name: user.name,
    email: user.email,
  })

  return Response.json(result)
}

// ✅ DO
export async function POST(req: Request) {
  const body = await req.json()
  const parsed = createUserSchema.safeParse(body)

  if (!parsed.success) {
    return Response.json({ error: parsed.error }, { status: 400 })
  }

  const [user] = await db
    .insert(users)
    .values(parsed.data)
    .returning()

  return Response.json(user)
}
```
