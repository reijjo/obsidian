# NextAuth - Extending session type
NextAuth's built-in session type has three fields: _id_, _name_ and _email_. So if we want something more included in the session for example _token_ we need to extend the session type.

## next-auth.d.ts file
Create `types/next-auth.d.ts` file in the root of your project
```ts
import { DefaultSession, DefaultUser } from "next-auth";

declare module "next-auth" {
	interface Session {
		user: {
			token?: string | null;
		} & DefaultSession["user"];
	}

	interface User extends DefaultUser {
		token?: string | null;
	}
}

declare module "next-auth/jwt" {
	interface JWT {
		token?: string | null;
	}
}
```
We add the token type in both _JWT_ and in the _Session_

## Callbacks
We need to update our `src/auth.ts`file:
```ts
export const { handlers, auth, signIn, signOut } = NextAuth({
	providers: [
		Credentials({
			credentials: {
				...
				return {
				id: String(user.id),
				name: user.name,
				email: user.username,
				token: user.token, // <-- Add the token here
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
	callbacks: {
		async jwt({ token, user }) {
			if (user) {
				token.token = user.token;
			}
			return token;
		},
		async session({ session, token }) {
			if (session.user) {
				session.user.token = token.token as string | null;
			}
			return session;
		},
	},
});
```
So we add the callbacks with the token for _JWT_  and _Session_ and also we return the _user.token_ in the credentials.
## Related
- [[Next.js - Authentication with NextAuth]]