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

### Chapter 10

`Partial prerendering (PPR)` is an experimental feature introduced in Next 14 that allows you to combine static rendering, dynamic rendering, and streaming in the same route.

When a user visits a route with PPR, a static shell is sent to the client with holes where dynamic content will be loaded asynchronously. The async holes are streamed in parallel.

Next will know which parts of your app are dynamic when they're wrapped in `Suspense` components

### Chapter 11

The search functionality span the client and the server. When a user searches for an invoice, the url parameters will be updated on the client, which will be kept in sync with the server, which then uses the params to fetch and render the new data on the client.

This is accomplished using the following Next hooks and the JS URLSearchParams API.

`useSearchParams` allows accessing the current parameters of the URL. `dashboard/invoices?page=1&query=pending` would look like this: `{page: '1', query: 'pending'}`.

`usePathname` lets you read the current URL's pathname. Would return `/dashboard/invoices`

`useRouter` enables programmatic client side navigation.

`URLSearchParams` provides methods for manipulating the URL query parameters.

```js
// search.tsx

const params = new URLSearchParams(searchParams);

// sets the value of the parameter called 'query'
params.set('query', term);

// params.toString() converts params to a string of all
// the query params. eg. 'page=1&query=hello'
replace(`${pathname}?${params.toString()}`);
```

The input field is kept in sync with the URL using the defaultValue attribute. `defaultValue={searchParams.get('query')?.toString()}`. Keep in mind to use `defaultValue` instead of `value`, since this is not a controlled component.

Next's page components accept an optional `searchParams` prop, which is a promise that resolves to an object containing the search parameters of the current URL. This prop can be used to get any query parameter. In this case, the parameter called 'query' is retrieved and passed to the table component, which that component uses to fetch invoices on the server.

```js
export default async function InvoicesTable({
  query,
  currentPage,
}: {
  query: string;
  currentPage: number;
}) {
  const invoices = await fetchFilteredInvoices(query, currentPage);
...
```

There were two methods used to get the search parameters, the `useSearchParams` hook and the `searchParams` prop. In general, you would use `searchParams` on the server and `useSearchParams` on the client.

Now, when searching for an invoice, every keystroke you make sends a request to the server. This can be fixed using a method called `debouncing`. There are many ways to implement it, but in this case, the `use-debounce` package is used.

```js
import { useDebouncedCallback } from 'use-debounce';

// Inside the Search Component...
const handleSearch = useDebouncedCallback((term) => {
	const params = new URLSearchParams(searchParams);
	if (term) {
		params.set('query', term);
	} else {
		params.delete('query');
	}
	replace(`${pathname}?${params.toString()}`);
}, 300);
```

The `handleSearch` function is called 300ms after the user stops typing.

All the same methods are used to implement pagination as well.

### Chapter 12

#### React Server Actions

React Server Actions allows you to write asynchronous functions that execute on the server and are called by the client or server. They eliminate the need for API endpoints for mutating data.

Server Actions offer security solutions and techniques like POST requests, encrypted closures, strict input checks, error message hashing, and host restrictions.

The `'use server'` directive is used to declare either a file or an async function as a server actions, which then allows them to be used by the client or the server.

An advantage of invoking a server action in a server component is `progressive enhancement`, meaning forms will be submitted even if JS hasn't loaded yet or is disabled.

Server Actions are not limited to forms and can be invoked from event handlers, useEffect, third-party libraries, and other form elements like buttons.

Actions use the POST method, and only this method can invoke them.

They're integrated with Next's caching and revalidation architecture.

##### Creating data

These are the steps taken to create a new invoice:

1. Create a form to capture the user's input.
2. Create a Server Action and invoke it from the form.
3. Inside your Server Action, extract the data from the formData object.
4. Validate and prepare the data to be inserted into your database.
5. Insert the data and handle any errors.
6. Revalidate the cache and redirect the user back to invoices page.

##### Updating data

These are the steps taken to update an invoice:

1. Create a new dynamic route segment with the invoice id.
2. Read the invoice id from the page params.
3. Fetch the specific invoice from your database.
4. Pre-populate the form with the invoice data.
5. Update the invoice data in your database.

Invoking a server action and passing data like this: `<form action={updateInvoice(id)}>`, will not work. Instead, you must use the JS bind method.

```js
const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);

return <form action={updateInvoiceWithId}></form>;
```

##### Deleting data

The id must be bound here as well.

```js
const deleteInvoiceWithId = deleteInvoice.bind(null, id);

<form action={deleteInvoiceWithId}>
```

### Chapter 13

The `error.tsx` file is used as a catch-all error handler for your route segments. This file needs to be a client component.

It accepts two props, `error` and `reset`. `reset` is a function that will re-render the route segment when called.

The `notFound` function is used to render the `not-found.tsx` file to handle 404 errors. This takes precedence over `error.tsx`.

Errors can be divided into 'expected errors' and 'uncaught exceptions'.

#### Expected Errors

Expected errors should be modeled as return values and returned to the client rather than thrown exceptions.

Server action error should be handled using `useActionState` instead of try/catch.

In server components, the error can be returned or the page can be redirected.

#### Uncaught Exceptions

Unexpected errors should be handled by throwing exceptions, which are then handled by error boundaries.

Implement error boundaries using `error.tsx` and `global-error.tsx` to handle unexpected errors and provide a fallback UI.

`global-error` is used to handle errors in the root layout. Global error UI must define its own `<html>` and `<body>` tags, since it is replacing the root layout or template when active.

### Chapter 14

Next includes `eslint-plugin-jsx-a11y` by default. To use it, add a script that runs `next lint`. This plugin helps to find accessibility issues.

#### useActionState

[`useActionState`](https://react.dev/reference/react/useActionState) takes three arguments and returns an array with three values.

```js
const [state, formAction, isPending] = useActionState(action, initialState, permalink?);
```

However, `isPending` and `permalink` aren't used in the example.

The `action` is the server action function and the `initialState` is anything you define, in this case, an object with the following format: `{ message: null, errors: {} }`

The returned `state` is the object that contains the errors that are returned from the action.

```js
if (!validatedFields.success) {
	return {
		errors: validatedFields.error.flatten().fieldErrors,
		message: 'Missing Fields. Failed to Create Invoice.',
	};
}
```

In the action file:

- zod is used to validate the user input
- if input is invalid, return the errors (the `state` returned by `useActionState` in this example)
- prepare and insert the valid input to the database
- revalidate the cache and redirect the user

```js
export async function createInvoice(prevState: State, formData: FormData) {
	// Validate form using Zod
	const validatedFields = CreateInvoice.safeParse({
		customerId: formData.get('customerId'),
		amount: formData.get('amount'),
		status: formData.get('status'),
	});

	// If form validation fails, return errors early. Otherwise, continue.
	if (!validatedFields.success) {
		return {
			errors: validatedFields.error.flatten().fieldErrors,
			message: 'Missing Fields. Failed to Create Invoice.',
		};
	}

	// Prepare data for insertion into the database
	const { customerId, amount, status } = validatedFields.data;
	const amountInCents = amount * 100;
	const date = new Date().toISOString().split('T')[0];

	// Insert data into the database
	try {
		await sql`
      INSERT INTO invoices (customer_id, amount, status, date)
      VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
    `;
	} catch (error) {
		// If a database error occurs, return a more specific error.
		return {
			message: 'Database Error: Failed to Create Invoice.',
		};
	}

	// Revalidate the cache for the invoices page and redirect the user.
	revalidatePath('/dashboard/invoices');
	redirect('/dashboard/invoices');
}
```

### Chapter 15

In this project, [`NextAuth.js`](https://authjs.dev/reference/nextjs) is used to implement authentication.

```js
export const authConfig = {
	// Used to specify the route for custom sign-in, sign-out, and error pages.
	pages: {
		// redirects user to our login page instead of NextAuth's default page
		signIn: '/login',
	},
	/*
	The 'authorized' callback is used to verify if a request is authorized to
	access a page via Next.js middleware. It is called before a request is
	completed, and it receives an object with the auth and request properties.
	The auth property contains the user's session, and the request property
	contains the incoming request.
	*/
	callbacks: {
		authorized({ auth, request: { nextUrl } }) {
			const isLoggedIn = !!auth?.user;
			const isOnDashboard = nextUrl.pathname.startsWith('/dashboard');
			if (isOnDashboard) {
				if (isLoggedIn) return true;
				return false; // Redirect unauthenticated users to login page
			} else if (isLoggedIn) {
				return Response.redirect(new URL('/dashboard', nextUrl));
			}
			return true;
		},
	},
	// used to define different login options. Credentials, Google, Github, etc.
	providers: [],
} satisfies NextAuthConfig;
```

The advantage of using middleware for this task is that protected routes will not start rendering until middleware verifies the authentication.

```js
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';

export default NextAuth(authConfig).auth;

export const config = {
	// https://nextjs.org/docs/app/building-your-application/routing/middleware#matcher
	// specify which routes the middleware will run on
	matcher: ['/((?!api|_next/static|_next/image|.*\\.png$).*)'],
};
```

```js
// auth.ts
Credentials({
	async authorize(credentials) {
		// validate user input with zod
		const parsedCredentials = z
			.object({ email: z.string().email(), password: z.string().min(6) })
			.safeParse(credentials);

		// get user from database and compare against user input
		if (parsedCredentials.success) {
			const { email, password } = parsedCredentials.data;
			const user = await getUser(email);
			if (!user) return null;
			const passwordsMatch = await bcrypt.compare(password, user.password);

			if (passwordsMatch) return user;
		}

		console.log('Invalid credentials');
		return null;
	},
}),
```

A react server action is used to authenticate the user. This action is called from the login form (`app/ui/login-form.tsx`).

```js
export async function authenticate(
	prevState: string | undefined,
	formData: FormData
) {
	try {
		await signIn('credentials', formData);
	} catch (error) {
		if (error instanceof AuthError) {
			switch (error.type) {
				case 'CredentialsSignin':
					return 'Invalid credentials.';
				default:
					return 'Something went wrong.';
			}
		}
		throw error;
	}
}
```

Signing out is done using the `signOut()` function from `NextAuth`.

### Chapter 16

`Open Graph` metadata enhances the way a site is displayed when shared on social media. Providing information such as a title, description, and a preview image.

There are two ways to add metadata: `config-based` and `file-based`.

Putting the `favicon.ico` and `opengraph-image.jpg` inside `/app` will automatically use them as the favicon and opengraph images. This is the `file-based` method.

To use the `config` method, export a static `metadata` object or a dynamic [`generateMetadata`](https://nextjs.org/docs/app/api-reference/functions/generate-metadata) function from a `layout.tsx` or `page.tsx` file. They are only supported in server components. You cannot use both methods on the same route segment.

A key difference is that, on initial load, streaming is blocked until `generateMetadata` has fully resolved.

```js
// app/layout.tsx
export const metadata: Metadata = {
	title: {
		// The template will be used on any route that declares a title using the
		// metadata object, like below.
		template: '%s | Acme Dashboard',
		default: 'Acme Dashboard',
	},
	description: 'The official Next.js Course Dashboard, built with App Router.',
	metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};

// app/dashboard/invoices/page.tsx
export const metadata: Metadata = {
	title: 'Invoices',
};
```

Dynamic metadata depends on dynamic information, such as the current route parameters, external data, or metadata in parent segments.

```js
type Props = {
  params: Promise<{ id: string }>
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}

export async function generateMetadata(
  { params, searchParams }: Props,
  parent: ResolvingMetadata
): Promise<Metadata> {
  // read route params
  const id = (await params).id

  // fetch data
  const product = await fetch(`https://.../${id}`).then((res) => res.json())

  // optionally access and extend (rather than replace) parent metadata
  const previousImages = (await parent).openGraph?.images || []

  return {
    title: product.title,
    openGraph: {
      images: ['/some-specific-page-image.jpg', ...previousImages],
    },
  }
}

export default function Page({ params, searchParams }: Props) {}
```
