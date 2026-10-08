# Playwright - End-To-End testing tool
## Installing Playwright
`bun create playwright` initializes a new project or adds Playwright to existing one.
1. "Where to put your end-to-end tests" --> e2e
2. "Add a GitHub Actions workflow" --> false
3. "Install Playwright browsers..." --> true

Then install the dependencies with `bun add -d @playwright/test`

Add scripts to _package.json_:
```json
{
  // ...
  scripts:{
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
  }
}
```

## Config file
Then we need to uncomment/add some lines in the _playwright.config.ts_ file:
```ts
import { defineConfig, devices } from "@playwright/test";
export default defineConfig({
	testDir: "./tests",
	// ...
	use: {
		/* Base URL to use in actions like `await page.goto('')`. */
		baseURL: "http://localhost:3000",
	},
	// ...
	webServer: {
		command: "bun run dev",
		url: "http://localhost:3000",
		reuseExistingServer: !process.env.CI,
	},
});
```
All these are the important ones
1. Make sure that the *testDir* is your actual folder where the tests are located
2. Check that the _baseUrl_ is correct
3. And uncomment the _webServer_ part so the tests can run without that your program needs the run at the same time
4. Make sure the _command_ and _url_ are correct
### Running tests on specific/multiple browsers
#### Multiple browsers
if you want to run tests with all 3 major browsers (_chromium_, _firefox_, _webkit_) you need to change the **fullyParallel** value to _false_ so it won't mess your register user stuff etc 
```ts
export default defineConfig({
	testDir: "./tests",
	fullyParallel: false,
	// ...
	projects: [
		{
			name: "chromium",
			use: { ...devices["Desktop Chrome"] },
		},
		{
			name: "firefox",
			use: { ...devices["Desktop Firefox"] },
		},
		{
			name: "webkit",
			use: { ...devices["Desktop Safari"] },
		},
	],
	// ...
});
```

#### Specific browser
This way the tests run much faster so this is what I recommend. So you comment out the browsers that you don't need for testing
```ts
export default defineConfig({
	testDir: "./tests",
	fullyParallel: true,
	// ...
	projects: [
		{
			name: "chromium",
			use: { ...devices["Desktop Chrome"] },
		},
//		{
//			name: "firefox",
//			use: { ...devices["Desktop Firefox"] },
//		},
//		{
//			name: "webkit",
//			use: { ...devices["Desktop Safari"] },
//		},
	],
// ...
});
```
And keep the **fullyParallel** to _true_ now.
## Guides
- [[E2E Testing With Playwright]]
## Related
- [[Next.js]]
- [[React]]
- [[SvelteKit]]