# RedwoodSDK Passkey Addon

This addon provides passkey (WebAuthn) authentication for a RedwoodSDK project.

## How to add to your project

These instructions assume you are starting with a new RedwoodSDK project, for example from `npx create-rwsdk -t minimal my-project-name`.

### 0. Download this addon

```
npx degit redwoodjs/sdk-experiments/passkey-addon _tmp_passkey_addon
```

### 1. Copy files

Copy the `src` directory from this addon into your project's root directory. This will add the following directories:

- `src/passkey`: Core logic for passkey authentication.
- `src/session`: Session management using a Durable Object.

### 2. Update `package.json`

Add the following dependencies to your `package.json` file:

```json
"dependencies": {
  "@simplewebauthn/browser": "^13.1.0",
  "@simplewebauthn/server": "^13.1.1",
  "rwsdk": "0.1.6"
}
```

Then run `pnpm install`.

### 3. Update `wrangler.jsonc`

Update your `wrangler.jsonc` to add Durable Object bindings, environment variables, and database migrations.

```jsonc
{
  // ... existing configuration ...

  // Durable Objects configuration
  "durable_objects": {
    "bindings": [
      {
        "name": "SESSION_DURABLE_OBJECT",
        "class_name": "SessionDurableObject"
      },
      {
        "name": "PASSKEY_DURABLE_OBJECT",
        "class_name": "PasskeyDurableObject"
      }
    ]
  },

  // Environment variables
  "vars": {
    "WEBAUTHN_APP_NAME": "My Awesome App"
  },

  // Migrations
  "migrations": [
    {
      "tag": "v1",
      "new_sqlite_classes": ["PasskeyDurableObject"]
    }
  ]
}
```

Remember to change `WEBAUTHN_APP_NAME` to your app's name.

### 4. Update `src/worker.tsx`

Modify your `src/worker.tsx` to integrate the passkey authentication and routes.

```typescript
// ...

import { authRoutes } from "@/passkey/routes";
import { setupPasskeyAuth } from "@/passkey/setup";
import { Session } from "@/session/durableObject";

export { SessionDurableObject } from "@/session/durableObject";
export { PasskeyDurableObject } from "@/passkey/durableObject";

export type AppContext = {
  // ...
  session: Session | null;
};

export default defineApp([
  // ...
  setCommonHeaders(),
  setupPasskeyAuth(),
  render(Document, [
    // ...
    index([
      ({ ctx }) => {
        if (!ctx.session?.userId) {
          return new Response(null, {
            status: 302,
            headers: { Location: "/auth/login" },
          });
        }
      },
      Home,
    ]),
    prefix("/auth", authRoutes()),
  ]),
]);
```

### 5. Update `src/app/pages/Home.tsx`

Add a login link to your `Home.tsx` page.

```typescript
// ...

export function Home({ ctx }: RequestInfo) {
  return (
    <div>
      {/* ... */}
      <p>
        <a href="/auth/login">Login</a>
      </p>
    </div>
  );
}
```

### 6. Run the dev server

Now you can run the dev server:

```shell
pnpm dev
```

You should now have a working passkey authentication flow in your RedwoodSDK application!