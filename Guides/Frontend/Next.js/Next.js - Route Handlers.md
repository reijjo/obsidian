# Next.js - Route Handlers
A _Route Handler_ is a file named `route.ts` inside the app directory. it exports named async functions corresponding to HTTP methods: GET, POST, PUT, PATCH, DELETE, and so on.

## Examples
### GET
Simple read only endpoint `src/app/api/notes/route.ts`
```ts
import { NextResponse } from "next/server";
import { getNotes } from "@/app/services/notes";

export const GET = async () => {
	const notes = await getNotes(false);
	return NextResponse.json(notes);
};
```

#### Quick Bearer token checker example
```ts
export const GET = async (request: NextRequest) => {
	const authHeader = request.headers.get("authorization");
	
	if (!authHeader || !authHeader.startsWith("Bearer ")) {
		return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
	}
	
	const token = authHeader.slice("Bearer ".length);
	
	const me = await getMe(token);
	
	if (!me) {
		return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
	}
	return NextResponse.json(me);
};
```
##### getMe function
```ts
export const getMe = async (token: string) => {
	const user = await db.query.users.findFirst({
		where: eq(users.token, token),
	});
	
	if (!user) {
		return null;
	}
	
	return user;
};
```

## Related
- [[Next.js]]
- [[Drizzle ORM with Neon]]