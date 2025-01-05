## Notes

### Chapter 3

#### Fonts

When using fonts with the `next/font` module, Next downloads them at build time and hosts them with other static assets. The fonts are sent to the server in the initial request, reducing the number of network requests and fixing issues with layout shift.

If you're using a variable font, you don't need to specify the font weight.

Font names with multiple words should have an underscore when being imported.

```js
import { Roboto_Mono } from 'next/font/google';
```

You can use fonts with CSS variables. When loading the font, add the variable property:

```js
const inter = Inter({
	subsets: ['latin'],
	display: 'swap',
	variable: '--font-inter',
});
```

Then use `inter.variable` instead of `inter.classname` and add it to the html document. Then you can use it with vanilla CSS, CSS modules, Tailwind, etc.

When a font is loaded as in the code example above, Next creates an instance of that font. If a font is loaded multiple times, then multiple instances are created. To prevent this, load all fonts in a utility file and export them.

#### Images

The `<Image>` component from `next/image` provides automatic image optimization, such as:

- Preventing layout shift
- Resizing images to avoid shipping images larger than they need to be
- Lazy loading images by default
- Serving images in modern formats when the browser supports it

When importing a local image, it's automatically sized. However, when using remote images, they should always be manually sized by including a `width` and a `height`, or using a `fill` to avoid layout shift issues.

The `width` and `height` properties are used to infer the aspect ratio of the image.

The `priority` property should be added to the image with the largest contentful paint, which allows Next to prioritize that image for loading. When you run `next dev`, you'll seeing a console warning if the LCP image doesn't have the tag.

When styling the image element, there are a few guidelines to follow. when using `fill`, the parent element must have `display: block` and `position: relative`, and you cannot use `styled-jsx`.

### Chapter 4

Next uses folder and a special `page.tsx` file to create routes. `/app/dashboard/invoices/page.tsx` routes to `/dashboard/invoices` for example.

Next uses a `layout.tsx` file to share UI between pages. the file is a react component which receives a `children` prop which Next automatically call and passes the `page` or another `layout`.

On navigation, only page components will update. This is called partial re-rendering.

The layout in `/app/layout.tsx` is the root layout and is required. Any UI added here is shared across the whole application.

### Chapter 5

Next has a `Link` component which is used for client side navigation. Next splits your code by route segments whenever a `Link` appears in your app, unlike a React SPA which loads all the code at once. This means that the code becomes isolated. If a page throws an error, the rest of the app will work.

Next also prefetches the code for linked routes in the background, so that when the user clicks on it, the browser will not reload.

The `useRouter` hook can be used to prefetch programmatically.

Next has a client-side cache called the `Router Cache`. When the user navigates through the app, prefetched and visited routes are stored in the cache, which is reused as much as possible instead of making requests to the server.

By default, Next preserves the scroll position of a page for back and forwards navigation.

### Chapter 7

There are a few cases where you want to use an API as an intermediary layer between the application and the database, such as when using a third party API or when fetching data from the client to avoid exposing the database to the client.

You can skip using an API layer when using `React Server Components`.

When making multiple data request in this manner, they are blocking each other creating a `request waterfall`.

```js
const revenue = await fetchRevenue();
const latestInvoices = await fetchLatestInvoices();
const { ...data } = await fetchCardData();
```

There are some cases when this is necessary, such as when one request depends on data retrieved from a previous request.

You can enable parallel data fetching using JS `Promise.all()`. However, there are some disadvantages to this method.

#### Route Handlers

You can create custom request handlers for routes using `Route Handlers`.

Route handlers are defined in a `route.ts` file inside the app directory. They can be nested anywhere inside app, similar to `page.tsx` and `layout.tsx`, but they cannot be in the same route segment level as `page.tsx`, since both of them take over all HTTP verbs for that route.

```js
// route.ts
export async function GET(request: Request) {}
```

Route handlers are not cached by default. You can opt into caching GET requests, and only GET requests.

They do not participate in layouts or client side navigation.

### Chapter 8

Static rendering is when data fetching and rendering happens on the server at build time. This is not ideal for sites where the data is regularly updated.

The solution is to use dynamic rendering. With dynamic rendering, content is rendered at request time. It also allows accessing information that's only available at request time, such as cookies or URL search params.

### Chapter 9

A common problem with dynamic rendering is that the application is only as fast as your slowest data fetch. This can be fixed with `streaming`.

Streaming breaks down a route into chunks and progressively stream them from the server to the client. With streaming, you can prevent slow data requests from slowing down the entire app, as the page will load part by part.

You can stream an entire page using the `loading.tsx` file, or a component with React's `Suspense`.

The `loading.tsx` file will automatically be used as a fallback while a `page.tsx` file is loading.

To prevent `loading.tsx` from being used for every route that comes after it, you can use `Route Groups`. Moving the `loading.tsx` and `page.tsx` files to a folder with parenthesis around it's name will organize the files into a logical group without affecting the URL path structure.

`/dashboard/(overview)/page.tsx` will still be `/dashboard`.
