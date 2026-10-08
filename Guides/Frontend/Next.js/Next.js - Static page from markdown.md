# Next.js - Static page from markdown
Next.js has built-in support for [MDX](https://nextjs.org/docs/app/guides/mdx), which lets you write JSX directly in Markdown files.

## Getting started
### Dependencies
First let's install dependencies
```sh
bun add @next/mdx @mdx-js/loader @mdx-js/react @types/mdx
```
### Config
Update the `next.config.ts`file in the root of the folder
```ts
import type { NextConfig } from "next";
import createMdx from "@next/mdx";

const nextConfig: NextConfig = {
	/* config options here */
	pageExtensions: ["js", "jsx", "md", "mdx", "ts", "tsx"],
};

const withMDX = createMdx({
	extension: /\.(md|mdx)$/,
});

export default withMDX(nextConfig);
```
Create `mdx-components.tsx`file in the _src_ folder:
```ts
import type { MDXComponents } from "mdx/types";

export function useMDXComponents(components: MDXComponents): MDXComponents {
	return {
		...components,
	};
}
```

## Usage
Create `homepage.mdx` file in the `src/app` folder:
```md
# Hello MDX

This is it
```

Then update the `src/app/page.tsx`:
```ts
// @ts-expect-error MDX module declarations are provided by the Next.js build configuration.
import Homepage from "./homepage.mdx";

export default function Home() {
	return (
		<main className="markdown">
			<Homepage />
		</main>
	);
}
```

### CSS
Add the styles for example in the _globals.css_ file:
```css
.markdown h1 {
	font-size: 2.5rem;
	font-weight: bold;
	margin: 1.5rem 0 1rem;
}
```


## Related
- [[Next.js]]