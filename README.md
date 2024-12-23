# Payload CMS plugin for Auth.js/NextAuth

<a href="https://github.com/CrawlerCode/payload-authjs/actions/workflows/ci.yml"><img alt="GitHub Actions Workflow Status" src="https://img.shields.io/github/actions/workflow/status/CrawlerCode/payload-authjs/ci.yml?style=flat-square&logo=github"></a>
<a href="https://www.npmjs.com/package/payload-authjs"><img alt="NPM Version" src="https://img.shields.io/npm/v/payload-authjs?style=flat-square"></a>
<a href="https://github.com/CrawlerCode/payload-authjs/blob/main/LICENSE"><img alt="NPM License" src="https://img.shields.io/npm/l/payload-authjs?style=flat-square"></a>
<a href="https://www.npmjs.com/package/payload-authjs"><img alt="NPM Downloads" src="https://img.shields.io/npm/dm/payload-authjs?style=flat-square"></a>

A [Payload CMS 3](https://payloadcms.com) plugin for integrating [Auth.js 5 (beta)](https://authjs.dev).

> ⚠ This plugin and Auth.js is in beta and may have some bugs. Please report any issues you find.

## Installation

Install the plugin using any JavaScript package manager like PNPM, NPM, or Yarn:

```bash
pnpm i payload-authjs
```

Fist of all, setup Auth.js like you would do in a Next.js application. You can follow the [Auth.js documentation](https://authjs.dev/getting-started/installation?framework=Next.js).

> ⚠ Make sure you define your config in a separate file (e.g. `auth.config.ts`) than where you create the NextAuth instance (e.g. `auth.ts`) to avoid circular dependencies. ⚠

```ts
// auth.config.ts
import github from "next-auth/providers/github";

export const authConfig: NextAuthConfig = {
  providers: [
    github, // <-- Add your provider here
  ],
};
```

Wrap your Auth.js configuration with the `withPayload` function before creating the NextAuth instance:

```ts
// auth.ts
import payloadConfig from "@payload-config";
import NextAuth from "next-auth";
import { withPayload } from "payload-authjs";
import { authConfig } from "./auth.config"; // ⚠ Import the config from a separate file

export const { handlers, signIn, signOut, auth } = NextAuth(
  withPayload(authConfig, {
    payloadConfig,
  }),
);
```

Add the `authjsPlugin` in your Payload configuration file:

```ts
// payload.config.ts
import { authjsPlugin } from "payload-authjs";
import { authConfig } from "./auth.config";

export const config = buildConfig({
  plugins: [
    authjsPlugin({
      authjsConfig: authConfig,
    }),
  ],
});
```

**And that's it! Now you can sign-in via Auth.js and you are automatically authenticated in Payload CMS. Nice 🎉**

---

## Customizing

You don't need to create a collection for users. This plugin automatically creates a collection with the slug `users`.

But if you want to customize the users collection, you can create a collection with the slug `users` and add the fields you need.

```ts
// users.ts
import type { CollectionConfig } from "payload";

const Users: CollectionConfig = {
  slug: "users",
  fields: [
    {
      name: "roles",
      type: "json",
    },
  ],
};

export default Users;
```

Next, you need to extend the user object returned by your Auth.js provider. You can do this like this example:

```ts
const authConfig: NextAuthConfig = {
  providers: [
    github({
      profile(profile) {
        return {
          id: profile.id.toString(),
          name: profile.name,
          email: profile.email,
          image: profile.avatar_url,
          roles: ["user"], // <-- Extend the user object with a custom field
        };
      },
    }),
  ],
  ...
};
```

⚠ Keep in mind that Auth.js doesn't update the user after the first sign-in. If you want to update the user on every sign-in, you can use the `updateUserOnSignIn` option in the `withPayload` function:

```ts
// auth.ts
export const { handlers, signIn, signOut, auth } = NextAuth(
  withPayload(authConfig, {
    payloadConfig,
    updateUserOnSignIn: true, // <-- Update the user on every sign-in
  }),
);
```

Now you could access your custom field, e.g. in the access control operations:

```ts
const Examples: CollectionConfig = {
  slug: "examples",
  access: {
    read: ({ req: { user } }) => {
      return user?.roles?.includes("user") ?? false; // <-- Check if the user has the role "user"
    },
  },
  fields: [
    ...
  ],
};
```

### Utility functions

This plugin also export a utility function to get the current payload user

```tsx
// ServerComponentExample.tsx
const ServerComponentExample = async () => {
  const payloadUser = await getPayloadUser();

  return (
    <div>
      <h3>Payload CMS User</h3>
      <div>{JSON.stringify(payloadUser)}</div>
    </div>
  );
};
```
