---
name: setup-nextjs
description: "Setup, scaffold, or extend the Vocaseed Next.js project following the established Clean Architecture. Use this skill when: creating a new page, route, or feature from scratch; adding a new data model + repository + service + DTO pipeline; scaffolding a new admin/user/guest section; setting up environment variables; configuring Tailwind CSS custom tokens; wiring next-intl translations; creating Server Actions with Zod validation; adding Shadcn/ui components. DO NOT USE FOR: Google SSO issues (use sso-with-google skill); general React questions unrelated to this project's structure."
---

# Setup Next.js — Vocaseed Clean Architecture

You are an expert Full-stack Developer working on the **Vocaseed** project.
Always follow the patterns documented here. Never deviate from the architecture without explicit instruction.

---

## 1. Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 16 (App Router) |
| Language | TypeScript (strict mode) |
| Database | MongoDB via Mongoose 9 |
| Auth | NextAuth.js v4 |
| Styling | Tailwind CSS v4 + Shadcn/ui |
| Icons | Lucide React |
| i18n | next-intl v4 |
| Validation | Zod v4 |
| Forms | React Hook Form + `useActionState` |
| Storage | AWS S3 / Cloudflare R2 |

---

## 2. Project Structure

```
src/
├── app/                        # Next.js App Router
│   ├── layout.tsx              # Root layout (fonts, NextIntlClientProvider)
│   ├── globals.css             # Tailwind v4 @theme tokens
│   ├── (guest)/                # Public routes — no auth required
│   ├── (onboarding)/           # Post-registration onboarding flow
│   ├── user/                   # Protected student routes
│   ├── admin/                  # Protected admin management routes
│   └── api/                    # API routes (NextAuth, webhooks, etc.)
├── components/
│   ├── common/                 # Atomic UI elements (buttons, inputs, modals)
│   ├── guest/                  # Components only used in (guest) routes
│   ├── user/                   # Components only used in user/ routes
│   ├── admin/                  # Components only used in admin/ routes
│   ├── auth/                   # AuthCard, login/register forms
│   └── shared/                 # Global layout blocks (Navbar, Sidebar, Footer)
├── dtos/                       # Input & Output Data Transfer Objects
├── mappers/                    # Entity → DTO transformation functions
├── models/                     # Mongoose models + IEntity interfaces
├── repositories/               # Data Access Layer (BaseRepository + domain repos)
├── services/                   # Business Logic Layer (returns DTOs)
├── lib/
│   ├── constants.ts            # ALL enums, routes, magic strings
│   ├── mongoose.ts             # Singleton MongoDB connection
│   ├── auth-options.ts         # NextAuth configuration
│   ├── utils.ts                # cn(), utility helpers
│   └── validation/             # Zod schemas (*.schema.ts)
├── i18n/
│   └── request.ts              # next-intl config (locale from cookie)
├── contexts/                   # React Context providers
├── hooks/                      # Custom React hooks
└── types/                      # TypeScript type augmentations
messages/
├── en.json                     # English translations
└── vi.json                     # Vietnamese translations
```

---

## 3. Communication Flow (Strict)

```
UI (Page / Server Action)
        ↓  Input DTO + Zod validation
Service (Business Logic)
        ↓  Entity operations
Repository (Data Access)
        ↓
MongoDB (via Mongoose)
        ↑  Entity
Repository → Service
        ↑  toXxxDTO() mapper
Service → UI (Output DTO)
```

**Prohibitions:**
- NEVER call a Repository directly from a Page or Component
- NEVER call MongoDB/Mongoose directly outside a Repository
- NEVER expose `passwordHash` in any DTO or response
- NEVER hardcode strings/routes in components — always import from `src/lib/constants.ts`

---

## 4. File Naming Convention

| Type | Convention | Example |
|------|-----------|---------|
| Pages, Layouts | `kebab-case.tsx` | `login-form.tsx` |
| Components | `kebab-case.tsx` | `confirm-modal.tsx` |
| Services | `kebab-case.service.ts` | `auth.service.ts` |
| Repositories | `kebab-case.repository.ts` | `user.repository.ts` |
| Models | `PascalCase.ts` | `User.ts` |
| DTOs | `kebab-case.dto.ts` | `auth.dto.ts` |
| Mappers | `kebab-case.mapper.ts` | `auth.mapper.ts` |
| Zod Schemas | `kebab-case.schema.ts` | `auth.schema.ts` |
| Constants | `constants.ts` (single file) | — |

---

## 5. Environment Variables

```env
# .env.local
MONGODB_URI=mongodb+srv://...
NEXTAUTH_SECRET=random-32-char-string
NEXTAUTH_URL=http://localhost:3001

# Google OAuth
GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=xxx

# Cloudflare R2 / AWS S3
R2_ACCOUNT_ID=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET_NAME=
R2_PUBLIC_URL=
```

Dev server runs on port **3001** (`next dev -p 3001`).

---

## 6. MongoDB Connection (Singleton)

Always use the cached connection in `src/lib/mongoose.ts`. Never create a new `mongoose.connect()` call anywhere else.

```typescript
// src/lib/mongoose.ts pattern
let cached = (global as any).mongoose;
if (!cached) cached = (global as any).mongoose = { conn: null, promise: null };

async function connectToDatabase() {
  if (cached.conn) return cached.conn;
  if (!cached.promise) {
    cached.promise = mongoose.connect(MONGODB_URI, { bufferCommands: false });
  }
  try { cached.conn = await cached.promise; }
  catch (e) { cached.promise = null; throw e; }
  return cached.conn;
}
```

---

## 7. Model Pattern

```typescript
// src/models/Widget.ts
import mongoose, { Document, Schema, Model } from 'mongoose';

export interface IWidget extends Document {
  name: string;
  userId: mongoose.Types.ObjectId;
  createdAt: Date;
  updatedAt: Date;
}

const widgetSchema = new Schema<IWidget>(
  {
    name: { type: String, required: true, trim: true },
    userId: { type: Schema.Types.ObjectId, ref: 'User', required: true },
  },
  { timestamps: true }
);

export const WidgetModel: Model<IWidget> =
  mongoose.models.Widget || mongoose.model<IWidget>('Widget', widgetSchema);
```

---

## 8. Repository Pattern

```typescript
// src/repositories/widget.repository.ts
import { BaseRepository } from './base.repository';
import { IWidget, WidgetModel } from '@/models/Widget';

export class WidgetRepository extends BaseRepository<IWidget> {
  constructor() {
    super(WidgetModel);
  }

  async findByUser(userId: string): Promise<IWidget[]> {
    await this.ensureConnection();
    return this.model.find({ userId }).sort({ createdAt: -1 }).exec();
  }
}
```

**BaseRepository** provides: `findOne`, `find`, `findById`, `create`, `update`, `delete`, `count`.
All methods call `ensureConnection()` before any DB operation.

---

## 9. DTO Pattern

```typescript
// src/dtos/widget.dto.ts

// Input DTO — used for validation + service input
export interface CreateWidgetInputDTO {
  name: string;
  userId: string;
}

// Output DTO — returned to UI (never expose sensitive fields)
export interface WidgetResponseDTO {
  id: string;       // Always string, never ObjectId
  name: string;
  userId: string;
  createdAt: string; // ISO string
}
```

---

## 10. Mapper Pattern

```typescript
// src/mappers/widget.mapper.ts
import { IWidget } from '@/models/Widget';
import { WidgetResponseDTO } from '@/dtos/widget.dto';

export function toWidgetResponseDTO(widget: IWidget): WidgetResponseDTO {
  return {
    id: widget._id!.toString(),
    name: widget.name,
    userId: widget.userId.toString(),
    createdAt: widget.createdAt.toISOString(),
  };
}
```

**Rules:**
- Always convert `_id` → `string`
- Always convert `ObjectId` refs → `string`
- Always convert `Date` → `.toISOString()`
- NEVER expose `passwordHash` or other sensitive fields

---

## 11. Service Pattern

```typescript
// src/services/widget.service.ts
import { WidgetRepository } from '@/repositories/widget.repository';
import { CreateWidgetInputDTO, WidgetResponseDTO } from '@/dtos/widget.dto';
import { toWidgetResponseDTO } from '@/mappers/widget.mapper';

const widgetRepo = new WidgetRepository();

export class WidgetService {
  async create(data: CreateWidgetInputDTO): Promise<WidgetResponseDTO> {
    const id = await widgetRepo.create(data as any);
    const created = await widgetRepo.findById(id);
    if (!created) throw new Error('Lỗi tạo widget');
    return toWidgetResponseDTO(created);
  }

  async getByUser(userId: string): Promise<WidgetResponseDTO[]> {
    const items = await widgetRepo.findByUser(userId);
    return items.map(toWidgetResponseDTO);
  }
}
```

---

## 12. Zod Validation Schema

```typescript
// src/lib/validation/widget.schema.ts
import { z } from 'zod';

export const createWidgetSchema = z.object({
  name: z.string().min(1, 'Tên không được để trống').max(100),
});

export type CreateWidgetInput = z.infer<typeof createWidgetSchema>;
```

---

## 13. Server Action Pattern

```typescript
// src/app/user/widgets/actions.ts
'use server';

import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth-options';
import { createWidgetSchema } from '@/lib/validation/widget.schema';
import { WidgetService } from '@/services/widget.service';

const widgetService = new WidgetService();

export async function createWidgetAction(
  _prevState: { error: string | null },
  formData: FormData,
): Promise<{ error: string | null }> {
  const session = await getServerSession(authOptions);
  if (!session?.user) return { error: 'Chưa đăng nhập' };

  const raw = { name: formData.get('name') as string };
  const parsed = createWidgetSchema.safeParse(raw);
  if (!parsed.success) return { error: parsed.error.issues[0].message };

  try {
    await widgetService.create({
      name: parsed.data.name,
      userId: (session.user as any).id,
    });
    return { error: null };
  } catch (err) {
    return { error: err instanceof Error ? err.message : 'Lỗi không xác định' };
  }
}
```

---

## 14. Page Pattern (Server Component)

```typescript
// src/app/user/widgets/page.tsx
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth-options';
import { redirect } from 'next/navigation';
import { ROUTES } from '@/lib/constants';
import { WidgetService } from '@/services/widget.service';
import { WidgetList } from '@/components/user/widget-list';

const widgetService = new WidgetService();

export default async function WidgetsPage() {
  const session = await getServerSession(authOptions);
  if (!session?.user) redirect(ROUTES.LOGIN);

  const userId = (session.user as any).id as string;
  const widgets = await widgetService.getByUser(userId);

  return <WidgetList widgets={widgets} />;
}
```

---

## 15. Client Component Pattern

```typescript
// src/components/user/widget-list.tsx
'use client';

import { useActionState } from 'react';
import { createWidgetAction } from '@/app/user/widgets/actions';
import type { WidgetResponseDTO } from '@/dtos/widget.dto';

interface Props {
  widgets: WidgetResponseDTO[];
}

const initialState = { error: null as string | null };

export function WidgetList({ widgets }: Props) {
  const [state, formAction] = useActionState(createWidgetAction, initialState);

  return (
    <div>
      {/* render list */}
      {widgets.map((w) => <div key={w.id}>{w.name}</div>)}

      {/* create form */}
      <form action={formAction}>
        <input name="name" type="text" required />
        {state.error && <p className="text-red-500 text-sm">{state.error}</p>}
        <button type="submit">Tạo</button>
      </form>
    </div>
  );
}
```

---

## 16. Route Groups

| Group | Path | Purpose |
|-------|------|---------|
| `(guest)` | `/`, `/login`, `/register`, `/courses`, `/blog` | Public — no auth check |
| `(onboarding)` | `/onboarding` | Post-signup flow — requires auth, incomplete onboarding |
| `user/` | `/dashboard`, `/learn`, `/vocabulary` | Protected student area |
| `admin/` | `/admin/dashboard`, `/admin/content` | Protected admin area |

---

## 17. Constants (src/lib/constants.ts)

All new constants must be added here. Pattern:

```typescript
// ─── Section Name ─────────────────────────────────────────────────────────────

export const MY_ENUM = {
  VALUE_A: 'value_a',
  VALUE_B: 'value_b',
} as const;
export type MyEnum = (typeof MY_ENUM)[keyof typeof MY_ENUM];

// Routes always go in ROUTES object
export const ROUTES = {
  // ...existing
  MY_NEW_ROUTE: '/my-new-route',
} as const;
```

---

## 18. Tailwind CSS v4 Custom Tokens

Tokens are defined in `src/app/globals.css` under `@theme inline { }`. Do NOT use `tailwind.config.ts` — this project uses Tailwind v4 CSS-first configuration.

```css
/* Custom brand color — always use 'brand', never 'primary' */
--color-brand: #137fec;
--color-brand-50: #eff7ff;

/* Custom backgrounds */
--color-background-light: #f6f7f8;
--color-background-dark: #101922;

/* Custom font */
--font-display: var(--font-lexend);
```

Usage in components: `className="bg-brand text-brand-50 font-display bg-background-light"`

---

## 19. i18n (next-intl)

Translation files: `messages/en.json` and `messages/vi.json`.
Locale is determined from the `NEXT_LOCALE` cookie (default: `vi`).

**In Server Components:**
```typescript
import { getTranslations, getLocale } from 'next-intl/server';
const t = await getTranslations('section.key');
const locale = await getLocale();
```

**In Client Components:**
```typescript
'use client';
import { useTranslations, useLocale } from 'next-intl';
const t = useTranslations('section.key');
const locale = useLocale();
```

Always add both `en.json` and `vi.json` entries when adding new UI text.

---

## 20. Shadcn/ui

Install components via CLI:
```bash
npx shadcn@latest add <component-name>
```

Components are added to `src/components/common/ui/`.
Use `cn()` from `src/lib/utils.ts` for conditional class merging:

```typescript
import { cn } from '@/lib/utils';
className={cn('base-classes', condition && 'conditional-class')}
```

---

## 21. Checklist — Adding a New Feature

When building a complete feature (e.g. "Widget management"):

- [ ] **Model**: Create `src/models/Widget.ts` with `IWidget` interface + Mongoose schema
- [ ] **Repository**: Create `src/repositories/widget.repository.ts` extending `BaseRepository`
- [ ] **DTO**: Create `src/dtos/widget.dto.ts` with Input and Output DTOs
- [ ] **Mapper**: Create `src/mappers/widget.mapper.ts` with `toWidgetResponseDTO()`
- [ ] **Zod Schema**: Create `src/lib/validation/widget.schema.ts`
- [ ] **Service**: Create `src/services/widget.service.ts` using repo + mapper
- [ ] **Server Action**: Create `actions.ts` co-located with the page
- [ ] **Page (Server Component)**: Fetch data via Service, pass DTOs to components
- [ ] **Client Component**: Use `useActionState` for forms, `'use client'` only when needed
- [ ] **Constants**: Add new routes/enums to `src/lib/constants.ts`
- [ ] **i18n**: Add translation keys to both `messages/en.json` and `messages/vi.json`
- [ ] **Component size**: Keep components under 150 lines — split into sub-components if needed
