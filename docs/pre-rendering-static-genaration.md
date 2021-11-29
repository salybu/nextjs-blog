# 🎢 Pre-rendering and Static Generation

- In this lesson, we'll learn how to fetch external blog data into our app. We'll store the blog content in the file system, but it'll work if the content is stored elsewhere (e.g. database or Headless CMS)

- What You'll learn in this lesson,

  - Next.js' pre-rendering feature
  - The 2 forms of pre-rendering: Static Generation and Server-side Rendering
  - Static Generation with data, and without data
  - `getStaticProps` and how to use it to import external blog data into the index page
  - some useful information on `getStaticProps`

&nbsp;

## Pre-rendering

<p align="center"><img src="https://nextjs.org/static/images/learn/data-fetching/pre-rendering.png"  alt="pre-rendering" width="655" height="390"></p>

- By default, Next.js `pre-renders every page`. This means that Next.js `generates HTML for each page in advance`, instead of having it all done by client-side JavaScript. Pre-rendering can result in better performance and SEO.

- Each generated HTML is associated with `minimal JavaScript code` necessary for that page. When a page is loaded by the browser, its JavaScript code runs and makes the page fully interactive.

  &nbsp;

## 2 forms of Pre-rendering

- Next.js has 2 forms of pre-rendering: _**Static Generation**_ and _**Server-side Rendering**_. The differences is in `when it generates the HTML for a page`

  - **Static Generation** (Recommended): The HTML is generated `at build time` and will be reused on each request.
  - **Server-side Rendering**: The HTML is generated `on each request`.

<div align="right">
   <img src="https://nextjs.org/static/images/learn/data-fetching/static-generation.png"  alt="static-generation" width="460" height="276">
   <img src="https://nextjs.org/static/images/learn/data-fetching/server-side-rendering.png"  alt="server-side-rendering" width="478" height="276">
</div>

&nbsp;

- Importantly, Next.js lets you `choose which pre-rendering form` you'd like to use for each page. You can create a _**"hybrid"**_ Next.js app by using _**Static Generation**_ for most pages and using _**Server-side Rendering**_ for others.

&nbsp;

## Static Generation

- We recommend using _**Static Generation**_ over Server-side Rendering for `performance reasons`. Statically generated pages can be `cached by CDN` with no extra configuration to boost performance.

- You should ask yourself: "Can I `pre-render this page ahead of a user's request`?" If the answer is yes, then you should choose Static Generation. You can use Static Generation for many types of pages, including:

  - Marketing pages
  - Blog posts
  - E-commerce product listings
  - Help and documentation

&nbsp;

- If a page uses Static Generation, the page HTML is `generated at build time`. That means in production, the page HTML is generated when you run `next build`. This HTML will then be reused on each request. It can be cached by a CDN.

- In Next.js, you can statically generate pages with or without data. Let's take a look at each case.

&nbsp;

## Static Generation without data

- By default, Next.js pre-renders pages using Static Generation without fetching data.

```javascript
function About() {
  return <div>About</div>;
}

export default About;
```

&nbsp;

- Note that this page does `not need to fetch any external data` to be pre-rendered. In cases like this, Next.js generates `a single HTML file` per page during `build time`.

<p align="center"><img src="https://nextjs.org/static/images/learn/data-fetching/static-generation-without-data.png"  alt="static-generation-without-data" width="602" height="380"></p>

&nbsp;

# Static Generation with data

- Some pages require `fetching external data` for pre-rendering.

<p align="center"><img src="https://nextjs.org/static/images/learn/data-fetching/static-generation-with-data.png"  alt="static-generation-with-data" width="576" height="486"></p>

- These are 2 scenarios, and one or both might apply. In each case, you can use these functions that Next.js provides:

  - Your page _**content**_ depends on external data: Use `getStaticProps`
  - Your page _**paths**_ depend on external data: Use `getStaticPaths` (usually in addition to `getStaticProps`)

&nbsp;
&nbsp;

## Scenario 1: Your page content depends on external data

- `getStaticProps` runs at build time in production, and inside it, you can `fetch external data` and `send it as props to the page`.

<p align="center"><img src="https://nextjs.org/static/images/learn/data-fetching/index-page.png"  alt="data-fetching" width="576" height="486"></p>
&nbsp;

- Essentially, `getStaticProps` allows you to tell Next.js: "Hey, this page has some data dependencies - so when you pre-render this page at build time, make sure to `resolve them first`!"

```javascript
export default function Home(props) { ... }

export async function getStaticProps() {
  // Get external data from the file system, API, DB, etc.
  const data = ...

  // The value of the `props` key will be passed to the `Home` component
  return {
    props: ...
  }
}
```

&nbsp;

- Maybe you need to access _**the file system**_, _**fetch external API**_, or _**query your database**_ at build time.

- In `lib/posts.js`, we've implemented `getSortedPostsData` which fetches data from `the file system`. But you can fetch the data from other sources, like _**an external API endpoint**_, and it'll work just fine.

```javascript
export async function getSortedPostsData() {
  // Instead of the file system, fetch post data from an external API enddpoint
  const res = await fetch("..");
  return res.json();
}
```

&nbsp;

- You can also `query the database` directly. This is possible because `getStaticProps` only runs on the `server-side`. It will never run on the client-side.

- It _**won't**_ even be _**included in the JS bundle for the browser**_. That means you can write code such as _**direct database queries without them being sent to browsers**_.

```javascript
import someDatabaseSDK from 'someDatabaseSDK'

const databaseClient = someDatabaseSDK.createClient(...)

export async function getSortedPostsData() {
  // Instead of the file system, fetch post data from a database
  return databaseClient.query('SELECT posts...')
}
```

&nbsp;

#### Here's an example.

- Your blog page might need to fetch the list of blog posts from a CMS (content management system)

```javascript
// Todo: Need to fetch `posts` (by calling some API endpoint) before this page can be pre-rendered
function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li>{post.title}</li>
      ))}
    </ul>
  );
}

export default Blog;
```

&nbsp;

- Next.js allows you to `export` an `async` function called `getStaticProps` from the same file. This function gets called at build time and lets you pass fetched data to the page's `props` on pre-render.

```javascript
function Blog({ posts }) {
  // Render posts...
}

// This function gets called at build time
export async function getStaticProps() {
  // Call an external API endpoint to get posts
  const res = await fetch("https://.../posts");
  const posts = await res.json();

  // By returning { props: { posts } }, the Blog component will receive `posts` as a prop at build time
  return {
    props: {
      posts,
    },
  };
}

export default Blog;
```

&nbsp;

#### some information about [getStaticProps](https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation)

- `getStaticProps` should return an object with:

  - `props` : An optional object with the props that will `be received by the page component`. It should be a _**serializable**_ object.
  - `notFound` : An optional boolean value to allow the page to return a 404 status and page. Below is an example of how it works.
  - ...............

```javascript
export async function getStaticProps(context) {
  const res = await fetch(`https://.../data`);
  const data = await res.json();

  if (!data) {
    return {
      notFound: true,
    };
  }

  return {
    props: { data }, // will be passed to the page component as props
  };
}
```

&nbsp;

## Scenario 2: Your page paths depend on external data

- Next.js allows you to create pages with dynamic routes. For example, you can create a file called `pages/posts/[id].js` to show a single blog post based on `id`. This will allow you to show a blog post with `id: 1` when you access `posts/1`.
