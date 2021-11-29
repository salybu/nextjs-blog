# 🍹 Assets, Metadata, and CSS

Next.js has built-in support for `CSS` and `Sass`. For this course, we will use CSS.
This lesson will also talk about how Next.js handles `static assets` like _**images**_ and _**page metadata**_ like the **_<title>_** tag.

&nbsp;

## Assets

- Next.js can serve static files, like images, under a folder called `public` in the root directory. Files inside `public` can then be referenced by your code starting from _**the base URL(`/`)**_.

```javascript
<Image src="/images/profile.jpg" alt="Your Name" />
```

&nbsp;

- `next/image` is an extension of the HTML `<img>` element, evolved for the modern web. Next.js also has support for Image Optimization by default.

```javascript
import Image from "next/image";
```

&nbsp;

- Instead of optimizing images at build time, Next.js `optimizes images on-demand`, as users request them. Unlike static site generators and static-only solutions, your `build times aren't increased`, whether shipping 10 images or 10 million images.

```javascript
<Image
  height={144} // Desired size with correct aspect ratio
  width={144} // Desired size with correct aspect ratio
/>
```

&nbsp;

- `Images are lazy loaded` by default. That means your page speed isn't penalized for images outside the viewport. Images load as they are `scrolled into viewport`.

&nbsp;

## Head

- `<Head>` is a React component that is `built into Next.js`. It allows you to modify the `<head>` of a page.
- You can import the `Head` component from the `next/head` module.

```javascript
import Head from "next/head";
```

&nbsp;

- If you want to modify the metadata of the page, you can use `<Head>`. You can add the `<Head>` component on each page.

```javascript
<Head>
  <title>Create Next App</title>
  <link rel="icon" href="/favicon.ico" />
</Head>
<h1>First Post</h1>
```

&nbsp;

## CSS Styling

- Next.js has built-in support for styled-jsx, but you can also use other popular CSS-in-JS libraries like styled-components or emotion.
- Next.js has built-in support for CSS and Sass which allows you to import `.css` and `.scss` files. Using popular CSS libraries like Tailwind CSS is also supported.

&nbsp;

- Create a file called `components/layout.module.css` with the following content

```css
.container {
  max-width: 36rem;
  padding: 0 1rem;
  margin: 3rem auto 6rem;
}
```

&nbsp;

- To use this `container` class inside `components/layout.js`, you need to
  - import the CSS file and assign a name to it, like `styles`
  - use `styles.container` as the `className`

```javascript
import styles from "./layout.module.css";

export default function Layout({ children }) {
  return <div className={styles.container}>{children}</div>;
}
```

&nbsp;

- CSS Modules automatically generates unique class names. As long as you use CSS Modules, you don't have to worry about class name collisions.
- Next.js's `code splitting` feature works on CSS Modules as well. It ensures the `minimal amount of CSS is loaded` for each page. This results in smaller bundle sizes.
- CSS Modules are extracted from the JS bundles at build time and generate `.css` files that are loaded automatically by Next.js

&nbsp;

## Global Styles

- To load global CSS files, create a file called `pages/_app.js` with the following content

```javascript
export default function App({ Component, pageProps }) {
  return <Component {...pageProps} />;
}
```

- This `App` component is the top-level component which will `be common across all the different pages`. You can use this `App` component to keep state when navigating between pages, for example.

- In Next.js, you can `add global CSS files by importing them from` _**`pages/_app.js`**_. You cannot import global CSS anywhere else. The reason that global CSS can't be imported outside of `pages/_app.js` is that global CSS affects all elements on the page.

&nbsp;

## Extra

- `classnames` is a simple library that lets you toggle class names easily. You can install it.

- Out of the box, with no configuration, Next.js compiles CSS using PostCSS. To customize postCSS config, you create a top-level file called `postcss.config.js`. This is useful if you're using libraries like Tailwind CSS.

&nbsp;

---

Reference: https://nextjs.org/docs/basic-features/static-file-serving, https://nextjs.org/docs/api-reference/next/head, https://nextjs.org/learn/basics/assets-metadata-css
