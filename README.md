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
