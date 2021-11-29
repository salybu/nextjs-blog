# ⛵ Navigate between pages

Next.js automatically optimizes your application for the best performance by **`Code splitting`**, **`Client-Side Navigation`**, and **`Prefetching (in production)`**

&nbsp;

## 1. File System Routing

simply create a JS file under the `pages directory`, and the path to the file becomes `the URL path`.

- The router will automatically route files named `index` to the `root of the directory`.

```bash
`pages/index.js` → `/`
`pages/blog/index.js` → `/blog`
```

&nbsp;

- The router supports `Nested Files`. If you create a nested folder structure, files will automatically be routed in the same way stil.

```bash
`pages/blog/first-post.js` → `/blog/first-post`
`pages/dashboard/settings/username.js` → `/dashboard/settings/username`
```

&nbsp;

- To match a `dynamic segment`, you can use the `bracket syntax`. This allows you to match named parameters.

```bash
`pages/blog/[slug].js` → `/blog/:slug` (`/blog/hello-world`)
`pages/[username]/settings.js` → `/:username/settings` (`/foo/settings`)
`pages/post/[...all].js` → `/post/*` (`/post/2020/id/title`)
```

&nbsp;

## 2. Link between pages

- The Next.js router allows you to do `client-side navigation` between pages, similar to a single-page application. You can use `Link` Component to do this client-side route transition

```javascript
import Link from "next/link";
```

&nbsp;

- As you can see, the `Link` component is similar to using `<a>` tags, but instead of `<a href="..">`, you use `<Link href=".."` and put an `<a>` tag inside

```javascript
<Link href="/">
  <a>Back to home</a>
</Link>
```

&nbsp;

## 3. Client-Side Navigation & Code Splitting, prefetching

- The `Link` component enables `Client-Side Navigation` between 2 pages in the same Next.js app. Client-side navigation means that the `page transition` happens `using JS`, which is faster than the default navigation done by the browser.

- Next.js does `code splitting` automatically, so each page `only loads what's necessary` for that page.

- Furthermore, in a production build of Next.js, whenever `Link components` appear in the browser's viewport, Next.js automatically `prefetches the code` for the linked page in the background.

- By the time you click the link, the code for the destination page will already be loaded in the background, and the page transition will be near-instant!
- Any `<Link />` in the viewport (intially or through scroll) will be prefetched by default (including the corresponding data) for pages using Static Generation. The corresponding data for server-rendered routes is not prefetched.

  &nbsp;

---

Reference: https://nextjs.org/docs/routing/introduction, https://nextjs.org/learn/basics/navigate-between-pages/pages-in-nextjs
