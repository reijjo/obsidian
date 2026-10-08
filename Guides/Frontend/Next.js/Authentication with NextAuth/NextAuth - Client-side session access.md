# NextAuth - Client-side session access

## SessionProvider
Some parts of our UI need to have access to the session on the client side. The navigation bar is the clearest example: it could eg. show the logged-in user's name and a logout button when a session exists, and a login link when there is none.

NextAuth's client-side hooks like _useSession_ share session data through React context, which requires a provider component to sit above any component that uses it. NextAuth ships its own SessionProvider for exactly this purpose. Because it uses React context it must be a Client Component, and since we cannot place a Client Component directly in the root layout, we create a thin wrapper _src/app/components/SessionProvider.tsx_:

```ts
"use client";

import { SessionProvider } from "next-auth/react";
import { ReactNode } from "react";

type AuthSessionProviderProps = {
	children: ReactNode;
};

export default function AuthSessionProvider({
	children,
}: AuthSessionProviderProps) {
	return <SessionProvider>{children}</SessionProvider>;
}
```

Then we update the _layout.tsx_ to wrap the app inside _AuthSessionProvider_:
```ts
export default function RootLayout({ children }: LayoutProps<"/">) {
return (
	<html lang="en" className={`${geistSans.variable} ${geistMono.variable}`}>
		<body>
			<AuthSessionProvider>
				<NavBar />
				{children}
			</AuthSessionProvider>
		</body>
	</html>
);
}
```

## useSession hook
The navigation bar needs to render differently depending on whether a user is logged in. Because the logout button calls _signOut()_ in an _onClick_ handler, the component must be a Client Component. It gets hold to the session via the _useSession_ hook, which pulls the value from the React context provided by _SessionProvider_. We place the component in _src/app/components/NavBar.tsx_:
```ts
"use client";

import { signOut, useSession } from "next-auth/react";
import Link from "next/link";

export default function NavBar() {
const { data: session } = useSession();

return (
	<nav>
		<Link href="/">home</Link>
		{" | "}
		<Link href="/blogs">blogs</Link
		{" | "}
		<Link href="/users">users</Link>
		{" | "}
		{session ? (
			<>
			<Link href="/blogs/new">create blog</Link>
			{" | "}
			<em>{session.user?.name} logged in</em>{" "}
			<button onClick={() => signOut()}>logout</button>
			</>
		) : (
			<Link href="/login">login</Link>
		)}
	</nav>
);
}
```

## Login Page
The login page lives at _src/app/login/page.tsx_. It is a Client Component because it handles form submission and manages local state for error messages:
```ts
"use client";

import { signIn } from "next-auth/react";
import { useRouter } from "next/router";
import { useState, SubmitEvent } from "react";

export default function LoginPage() {
	const router = useRouter();
	const [error, setError] = useState("");

	const handleSubmit = async (e: SubmitEvent<HTMLFormElement>) => {
		e.preventDefault();
		const formData = new FormData(e.currentTarget);
		
		const result = await signIn("credentials", {
			username: formData.get("username"),
			password: formData.get("password"),
			redirect: false,
		});
		
		if (result?.error) {
			setError("Invalid username or password");
		} else {
			router.push("/");
		}
	};
	
	return (
		<div>
			<h2>Login</h2>
			{error && <p style={{ color: "red" }}>{error}</p>}
			<form onSubmit={handleSubmit}>
				<div>
					<label>
						Username
						<input type="text" name="username" required />
					</label>
				</div>
				<div>
					<label>
						Password
						<input type="password" name="password" required />
					</label>
				</div>
				<button type="submit">Login</button>
			</form>
		</div>
	);
}
```
The form calls signIn from _next-auth/react_ with _redirect: false_, which means NextAuth will return the result as an object instead of automatically redirecting the browser. If authentication fails, _result.error_ will be set and we display an error message. If it succeeds, we use the Next.js router to navigate to the home page.

## Register
### Server action
We need a Server Action that hashes the password and inserts the new user in the database in _src/app/actions/users.ts_:
```ts
"use server";

import { db } from "@/db";
import { users } from "@/db/schema";
import bcrypt from "bcryptjs";
import { redirect } from "next/navigation";

export const registerUser = async (formData: FormData) => {
	const username = (formData.get("username") as string)?.trim();
	const name = (formData.get("name") as string)?.trim();
	const password = formData.get("password") as string;
	
	const passwordHash = await bcrypt.hash(password, 10);
	
	await db.insert(users).values({ username, name, passwordHash });
	
	redirect("/login");
};
```
### Register page
_src/app/register/page.tsx_:
```ts
import { registerUser } from "../actions/users";

export default function RegisterPage() {
	return (
		<div>
			<h2>Register</h2>
			<form action={registerUser}>
				<div>
					<label>
						Username
						<input type="text" name="username" required />
					</label>
				</div>{" "}
				<div>
					<label>
						Name
						<input type="text" name="name" required />
					</label>
				</div>{" "}
				<div>
					<label>
						Password
						<input type="password" name="password" required />
					</label>
				</div>
				<button type="submit">Register</button>
			</form>
		</div>
	);
}
```
Remember to add the link to register page to the Nav component.

## Related
- [[Next.js - Authentication with NextAuth]]