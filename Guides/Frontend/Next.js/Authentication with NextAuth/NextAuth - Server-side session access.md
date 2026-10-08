# NextAuth - Server-side session access
For Server Components and Server Actions we cannot use _useSession_, since hooks only run in the browser. Instead Auth.js provides the _auth_ function, which reads the session from the request headers on the server side. We wrap this in a helper function _src/app/services/session.ts_:
```ts
import { auth } from "@/auth";
import { db } from "@/db";
import { users } from "@/db/schema";
import { eq } from "drizzle-orm";

export const getCurrentUser = async () => {
	const session = await auth();
	if (!session?.user?.email) {
		return null;
	}
	
	return db.query.users.findFirst({
		where: eq(users.username, session.user.email),
	});
};
```

## Protecting a Server Action
Only logged users can create blogs so lets update the _src/app/actions/blogs.ts_:
```ts
"use server";

import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";
import { addBlog, addLike } from "../services/blogs";
import { auth } from "@/auth";

export const createBlog = async (formData: FormData) => {
	const session = await auth()
	if (!session) {
		redirect("/login")
	}
	
	const title = formData.get("title") as string;
	const author = formData.get("author") as string;
	const url = formData.get("url") as string;
	
	await addBlog(title, author, url);
	
	revalidatePath("/blogs");
	redirect("/blogs");
};
```
