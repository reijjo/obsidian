# React - Suspense
When a Server Component fetches data, the entire page waits for the data before anything is sent to the browser. For slow queries this means the user stares at a blank screen. React's [Suspense](https://react.dev/reference/react/Suspense) boundary solves this by letting you stream parts of the UI as they become ready.

The simplest way to add loading UI to a route is to create a [loading.tsx](https://nextjs.org/docs/app/api-reference/file-conventions/loading) file in the same directory as _page.tsx_. Next.js automatically wraps the page in a Suspense boundary and shows the loading component while the page is being prepared:

```ts
const Loading = () => {
	return <p>Loading notes...</p>
}

export default Loading
```
Now when the user navigates to _/notes_, they immediately see "Loading notes..." while the Server Component fetches data. Once the data is ready, Next.js streams the actual page content and replaces the loading state.

```ts
import { Suspense } from "react"
import NoteList from "./NoteList"
import Statistics from "./Statistics"

const Notes = () => {
  return (
    <div>
      <h2>Notes</h2>
      <Suspense fallback={<p>Loading notes...</p>}>
        <NoteList />
      </Suspense>
      <Suspense fallback={<p>Loading statistics...</p>}>
        <Statistics />
      </Suspense>
    </div>
  )
}
```

## Related
- [[Next.js]]
- [[React]]