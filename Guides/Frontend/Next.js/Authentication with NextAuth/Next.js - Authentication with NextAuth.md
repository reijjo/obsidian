# Authentication with NextAuth
<https://next-auth.js.org/>

## Install
First we need to install the needed packages:
```sh
bun add next-auth@beta bcryptjs && bun add -d @types/bcryptjs
```

## Database setup
### Users table
And we need to create _users_ table for our database (this example is done by using [[Drizzle ORM with Neon]]):
```ts
export const users = pgTable("users", {
	id: serial("id").primaryKey(),
	username: text("username").notNull().unique(),
	name: text("name").notNull(),
	passwordHash: text("password_hash").notNull().default(""),
});
```

## NextAuth configuration
- [[NextAuth - Configuration]]
## Client-side session access
- [[NextAuth - Client-side session access]]
## Server-side session access
- [[NextAuth - Server-side session access]]

## Environment variables
NextAuth requires a secret key to sign the JWT session token, so we can generate one also in terminal:
```sh
echo "$(openssl rand -base64 32)"
```
And copy the output in the _.env.local_ file:
```env
DATABASE_URL=postgresql://...

AUTH_SECRET=...
```
Also add the variables in _.env.local_ file and also add _AUTH_URL_ to Vercel. _AUTH_URL_ is the public URL of the deployed app, for example _https://your-app.vercel.app_
- Choose your project
- Settings --> Environments --> Production --> Environment Variables --> _Add Environment Variable_

## How the session works
After a successful login, NextAuth creates a session and stores it as a signed JWT in an HTTP-only cookie in the browser. The cookie is sent automatically with every subsequent request. On the server, NextAuth verifies the signature and reads the token to know who the user is, without touching a database.

The token contains whatever the _authorize_ function returned: in our case _id_, _name_, and _email_ (which we used to store the username). These values are available through _useSession_ on the client and through _auth()_ on the server.

## Auth flow recap
### Sign-in flow
The user fills in the login form on _/login_. The form's _onSubmit_ handler calls _signIn("credentials", { redirect: false, ... }), which_ is a function provided by NextAuth. Under the hood, NextAuth's _signIn_ makes a _POST_ request to the catch-all API route at _/api/auth/callback/credentials_. The route handler is defined in _app/api/auth/[...nextauth]/route.ts_, where _handlers_ exported from _auth.ts_ registers the GET and POST handlers for Next.js.

When the POST arrives, NextAuth's internal logic takes over and calls the _authorize_ function from _auth.ts_. That function looks up the user in the database by username and uses _bcrypt.compare_ to verify the password against the stored hash. If the credentials are valid, _authorize_ returns a user object with _id_, _name_, and _email_ (where we store the username). NextAuth takes that object, encodes it into a JWT signed with _AUTH_SECRET_, and sets it as an HTTP-only cookie in the response. An HTTP-only cookie cannot be read by JavaScript in the browser, only sent automatically with each subsequent request, which protects the token from XSS attacks. The login page then uses _router.push("/")_ to navigate the user to the home page.

### Client-side session flow
On the next render, the browser sends the cookie with every request. _AuthSessionProvider_ (which wraps the whole app in the root layout) reads the cookie, verifies the JWT, and makes the session data available through React context. Any Client Component that calls _useSession()_ receives the session from that context without any additional network request.

### Server-side session flow
When a Server Component or Server Action needs to know who is logged in, it calls _auth()._ NextAuth reads the signed JWT from the incoming request headers and returns the decoded session. The helper _getCurrentUser_ wraps this call and additionally fetches the full user record from the database so that the rest of the application code gets a proper user object.

### Protected actions 
Server Actions that require authentication call _auth()_ at the top and redirect to _/login_ if the session is missing. Only after confirming a valid session is any database write performed.


## Related
- [[Deploy a project to Vercel]]
- [[Drizzle ORM with Neon]]
- [[Next.js]]
- [[NextAuth - Configuration]]
- [[NextAuth - Client-side session access]]
- [[NextAuth - Server-side session access]]
- [[NextAuth - Extending session type]]