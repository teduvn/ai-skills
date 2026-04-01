---
name: sso-with-google
description: "Implement, fix, or review Google SSO (Single Sign-On) in this Next.js project using NextAuth.js v4. Use this skill when: adding Google login to a new page; troubleshooting Google OAuth callback errors; handling 'Google user ID is not a MongoDB ObjectId' bugs; fixing duplicate user creation race conditions; updating signIn/jwt/session callbacks; configuring GOOGLE_CLIENT_ID and GOOGLE_CLIENT_SECRET env vars; debugging why session.user.id is undefined after Google login; adding Google provider to auth-options.ts. DO NOT USE FOR: general NextAuth setup without Google; Credentials provider issues only; Google APIs unrelated to auth."
---

# SSO with Google — Vocaseed

You are an expert in NextAuth.js v4 Google OAuth integration for the Vocaseed project.
Always follow the Clean Architecture conventions in `.github/copilot-instructions.md`.

---

## Architecture Overview

```
Google OAuth Callback
        ↓
NextAuth signIn callback  (src/lib/auth-options.ts)
  → lookup or create IUser in MongoDB via UserRepository
        ↓
NextAuth jwt callback
  → resolve Google ID → MongoDB ObjectId
  → attach role, id, sessionId, levelCode to token
        ↓
NextAuth session callback
  → expose role, id, sessionId, levelCode to client
        ↓
Client: signIn('google') / getServerSession(authOptions)
```

---

## 1. Environment Variables

Always required in `.env.local` (and in production env):

```env
GOOGLE_CLIENT_ID=your-google-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-google-client-secret
NEXTAUTH_SECRET=a-random-32-char-secret
NEXTAUTH_URL=http://localhost:3000
```

In Google Cloud Console → OAuth 2.0 Credentials:
- **Authorized JavaScript origins**: `http://localhost:3000` (dev), `https://yourdomain.com` (prod)
- **Authorized redirect URIs**: `http://localhost:3000/api/auth/callback/google`

---

## 2. Core Files Modified

| File | Purpose |
|------|---------|
| `src/lib/auth-options.ts` | Central NextAuth config — providers + callbacks |
| `src/app/api/auth/[...nextauth]/route.ts` | NextAuth route handler (GET + POST) |
| `src/repositories/user.repository.ts` | `findByEmail()` used in callbacks |
| `src/models/User.ts` | `IUser` with `provider` and `providerId` fields |
| `src/components/auth/AuthCard.tsx` | Google sign-in button (client component) |
| `src/app/(guest)/login/login-form.tsx` | `signIn('google', { callbackUrl })` call |

---

## 3. GoogleProvider Setup

```typescript
// src/lib/auth-options.ts
import GoogleProvider from "next-auth/providers/google";

providers: [
  GoogleProvider({
    clientId: process.env.GOOGLE_CLIENT_ID || '',
    clientSecret: process.env.GOOGLE_CLIENT_SECRET || '',
  }),
]
```

---

## 4. signIn Callback — Auto-create & Upsert User

**Critical pattern**: Google users must be looked up or created in MongoDB.
The `user.id` from NextAuth is the Google account ID, **not** a MongoDB ObjectId.

```typescript
async signIn({ user, account }) {
  if (account?.provider === 'google') {
    if (!user.email) return false;

    const { UserRepository } = await import('@/repositories/user.repository');
    const userRepo = new UserRepository();
    const normalizedEmail = user.email.toLowerCase().trim();
    let existingUser = await userRepo.findByEmail(normalizedEmail);

    if (!existingUser) {
      const fallbackName = normalizedEmail.split('@')[0] || 'User';

      try {
        await userRepo.create({
          email: normalizedEmail,
          fullName: user.name?.trim() || fallbackName,
          avatar: user.image || undefined,
          provider: 'google',
          providerId: account.providerAccountId,
          role: ROLES.USER,
          onboardingCompleted: false,
        } as any);
      } catch (error) {
        // Handle race condition — two simultaneous Google callbacks for the same email
        const isDuplicateKey =
          typeof error === 'object' &&
          error !== null &&
          'code' in error &&
          (error as { code?: number }).code === 11000;

        if (!isDuplicateKey) {
          console.error('Failed to create Google user', error);
          return false;
        }
        // If duplicate key, another request already created the user — continue
      }

      existingUser = await userRepo.findByEmail(normalizedEmail);
      if (!existingUser) return false;
    }
  }
  return true;
},
```

### Common Pitfalls in signIn callback

| Pitfall | Fix |
|---------|-----|
| Not normalizing email | Always `email.toLowerCase().trim()` before DB lookup |
| Not handling `code: 11000` MongoDB duplicate key | Catch and continue — parallel OAuth callbacks can race |
| Using `user.id` as MongoDB ObjectId | It is the Google account ID string, not a valid ObjectId |
| Omitting `onboardingCompleted: false` | New Google users must go through onboarding flow |

---

## 5. jwt Callback — Resolve Google ID to MongoDB ObjectId

**Critical bug to avoid**: After Google login, `user.id` is the Google account ID (string like `"1234567890"`). Storing it as `token.id` causes BSON/ObjectId crashes in all SSR pages that use `session.user.id`.

```typescript
async jwt({ token, user, account }) {
  if (user) {
    let mongoDbUserId = user.id;

    if (account?.provider === 'google') {
      if (!user.email) return token;

      const { UserRepository } = await import('@/repositories/user.repository');
      const userRepo = new UserRepository();
      const dbUser = await userRepo.findByEmail(user.email);
      if (!dbUser?._id) return token;

      mongoDbUserId = dbUser._id.toString();
      token.role = dbUser.role;
    } else {
      token.role = (user as any).role;
    }

    token.id = mongoDbUserId;
    token.sub = mongoDbUserId; // Keep sub in sync
    token.levelCode = (user as any).levelCode;
    // ... attach sessionId via TrackingService
  }

  // Self-heal: fix legacy tokens that stored the Google ID instead of MongoDB _id
  const tokenUserId = typeof token.id === 'string'
    ? token.id
    : typeof token.sub === 'string'
      ? token.sub
      : undefined;

  if (token.email && (!tokenUserId || !Types.ObjectId.isValid(tokenUserId))) {
    try {
      const { UserRepository } = await import('@/repositories/user.repository');
      const userRepo = new UserRepository();
      const dbUser = await userRepo.findByEmail(token.email);
      if (dbUser?._id) {
        token.id = dbUser._id.toString();
        token.sub = dbUser._id.toString();
        token.role = dbUser.role;
      }
    } catch (error) {
      console.error('Failed to normalize JWT user id', error);
    }
  }

  return token;
},
```

---

## 6. session Callback — Expose to Client

```typescript
async session({ session, token }) {
  if (session.user) {
    (session.user as any).role = token.role;
    (session.user as any).id = token.id;
    (session.user as any).sessionId = token.sessionId;
    (session.user as any).levelCode = token.levelCode;
  }
  return session;
},
```

---

## 7. API Route Handler

```typescript
// src/app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import { authOptions } from '@/lib/auth-options';

const handler = NextAuth(authOptions);
export { handler as GET, handler as POST };
```

---

## 8. Triggering Google Login from the Client

```typescript
// 'use client'
import { signIn } from 'next-auth/react';

// Redirect to Google OAuth popup/redirect
await signIn('google', { callbackUrl: '/dashboard' });
```

The `callbackUrl` must be a same-origin path or it will be ignored by NextAuth's redirect guard.

---

## 9. IUser Model Requirements

The `IUser` interface in `src/models/User.ts` must have:

```typescript
provider?: string;    // 'google' | 'credentials'
providerId?: string;  // Google account ID (account.providerAccountId)
```

Google users have **no** `passwordHash`. All password-gated operations must check:
```typescript
if (!user.passwordHash) {
  // This is a Google/SSO user — do not allow password change flow
}
```

---

## 10. Onboarding Flow for New Google Users

After first Google login, `onboardingCompleted` is `false`.
The session callback exposes this via the role/redirect logic.

Pattern used in this project:
1. `signIn` callback creates user with `onboardingCompleted: false`
2. `jwt` callback attaches `token.role` (which is `ROLES.USER`)
3. Middleware or layout in `(onboarding)/` checks `onboardingCompleted` and redirects

Do NOT skip the onboarding guard for Google users.

---

## 11. Session Usage in Server Components

```typescript
// Server Component / Server Action
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth-options';

const session = await getServerSession(authOptions);
if (!session?.user) redirect('/login');

const userId = (session.user as any).id; // Always a valid MongoDB ObjectId string
```

---

## 12. Common Errors & Fixes

| Error | Root Cause | Fix |
|-------|-----------|-----|
| `BSONTypeError: Argument passed in must be a string of 12 bytes` | `token.id` is a Google ID string, not ObjectId | Self-heal block in `jwt` callback (section 5) |
| User created twice | Race condition in `signIn` callback | Catch `code: 11000` and continue |
| `session.user.id` is `undefined` | `session` callback not mapping `token.id` | Add `(session.user as any).id = token.id` |
| Google login works but onboarding skipped | `onboardingCompleted` not set to `false` on creation | Always set `onboardingCompleted: false` in `userRepo.create()` |
| `redirect_uri_mismatch` OAuth error | Google Console URIs not updated | Add `http://localhost:3000/api/auth/callback/google` to authorized redirect URIs |
| `NEXTAUTH_URL` mismatch in prod | Deploying without updating the env var | Set `NEXTAUTH_URL=https://yourdomain.com` in production |

---

## 13. Type Augmentation for NextAuth Session

To avoid TypeScript errors when accessing `session.user.id` or `session.user.role`, add or update `src/types/next-auth.d.ts`:

```typescript
import { DefaultSession } from 'next-auth';

declare module 'next-auth' {
  interface Session {
    user: {
      id: string;
      role: string;
      sessionId?: string;
      levelCode?: string;
    } & DefaultSession['user'];
  }
}
```

---

## Implementation Checklist

- [ ] `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` set in `.env.local`
- [ ] Google Cloud Console redirect URI includes `/api/auth/callback/google`
- [ ] `GoogleProvider` added to `providers[]` in `auth-options.ts`
- [ ] `signIn` callback handles Google provider with upsert + race-condition guard
- [ ] `jwt` callback resolves Google account ID → MongoDB ObjectId
- [ ] `jwt` callback includes self-heal block for legacy tokens
- [ ] `session` callback maps `token.id`, `token.role` to `session.user`
- [ ] `IUser` has `provider` and `providerId` fields
- [ ] New Google users created with `onboardingCompleted: false`
- [ ] Client button calls `signIn('google', { callbackUrl: ... })`
- [ ] NextAuth session type augmented to include `id`, `role`, `sessionId`
