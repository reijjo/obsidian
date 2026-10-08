# Next.js - Metadata and SEO
Next.js has a built-in [Metadata API](https://nextjs.org/docs/app/getting-started/metadata-and-og-images) for setting the page title, description, and other _<head />_ tags in a way that works correctly with Server Components and streaming.

The simplest form is a static _metadata_ export in a _page.tsx_ or _layout.tsx_ file:
```tsx
import type { Metadata } from "next"

export const metadata: Metadata = {
  title: "Notes app",
  description: "A simple notes application built with Next.js",
}
```
Next.js merges metadata from the root layout down through nested layouts and pages, so a child page can override just the fields it needs without repeating everything.

## Dynamic pages
For dynamic pages where the title depends on data, you export a _generateMetadata_ function instead:
```tsx
import type { Metadata } from "next"
import { getNoteById } from "../../services/notes"

export const generateMetadata = async ({
  params,
}: {
  params: Promise<{ id: string }>
}): Promise<Metadata> => {
  const { id } = await params
  const note = await getNoteById(Number(id))
  return {
    title: note ? note.content.slice(0, 50) : "Note not found",
  }
}
```
This runs on the server, and the resolved title appears in the HTML _<title />_ tag, which is important for search engines and for the browser tab label. Open Graph tags for social sharing follow the same pattern using the _openGraph_ key inside the metadata object.

## Related
- [[Next.js]]