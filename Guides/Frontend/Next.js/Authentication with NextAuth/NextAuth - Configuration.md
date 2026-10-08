# NextAuth - Configuration
Create NextAuth configuration file in _src/auth.ts_:
```ts
import NextAuth from "next-auth";
import Credentials from "next-auth/providers/credentials";
import { db } from "./db";
import { eq } from "drizzle-orm";
import { users } from "./db/schema";
import bcrypt from "bcryptjs";

export const { handlers, auth, signIn, signOut } = NextAuth({
	providers: [
		Credentials({
			credentials: {
				username: { label: "Username", type: "text" },
				password: { label: "Password", type: "password" },
			},
			async authorize(credentials) {
				if (!credentials?.username || !credentials?.password) {
					return null;
			}
	
			const user = await db.query.users.findFirst({
				where: eq(users.username, credentials.username as string),
			});
	
			if (!user || !user.passwordHash) {
				return null;
			}
	
			const isValid = await bcrypt.compare(
				credentials.password as string,
				user.passwordHash,
			);
	
			if (!isValid) {
				return null;
			}
	
			return {
				id: String(user.id),
				name: user.name,
				email: user.username,
			};
		},
	}),
	],
	pages: {
		signIn: "/login",
	},
	session: {
		strategy: "jwt",
	},
});
```

## Credentials provider
NextAuth supports many different authentication providers such as _Google_, _GItHub_ etc. Here we use the _Credentials provider_, which lets users log in with a username and password.

### Authorize function
The _authorize_ function is the core of the Credentials provider. It receives the values the user typed in the login form as _credentials_. It then looks up the user in the database by username and compares the password if the submitted password matches the stored hash.  If it fails for any reason, it returns _null_, which causes NextAuth to reject the login attempt.

When authentication succeeds, _authorize_ returns a plain object that NextAuth encodes into the JWT session token:
```ts
{
  id: String(user.id),
  name: user.name,
  email: user.username,
}
```
NextAuth's built-in session type has three fields: _id_, _name_, and _email_. We do not have an email address in our schema, so we reuse the _email_ field to store the username instead (can be done with custom TypeScript declarations).

## Custom sign-in page
The _pages: { signIn: "/login" }_ option tells NextAuth to use our own login page at _/login_ instead of the built-in NextAuth sign-in page. Whenever NextAuth needs to redirect an unauthenticated user to log in, it will send them to _/login_.

## Session strategy
The _session: { strategy: "jwt" }_ option tells NextAuth to store session data in a signed JSON Web Token in a cookie, rather than in a server-side session store.

## Auth API route
NextAuth requires an API route that handles all authentication requests (sign in, sign out, session checks). We create the file _src/app/api/auth/[...nextauth]/route.ts_:
```ts
import { handlers } from "@/auth";

export const { GET, POST } = handlers;
```

The [...nextauth] part is a Next.js catch-all route segment that matches any path under _/api/auth/_, for example _/api/auth/signin_, _/api/auth/signout_, and _/api/auth/session_. NextAuth intercepts all of these and handles them automatically.

## Related
- [[Next.js - Authentication with NextAuth]]