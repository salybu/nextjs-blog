# 🎢 Pre-rendering and Static Generation

- In this lesson, we'll learn how to _**fetch external blog data**_ into our app. We'll store the blog content in the file system, but it'll work if the content is stored elsewhere (e.g. database or Headless CMS)

&nbsp;

## Pre-rendering

<p align="center"><img src="https://nextjs.org/static/images/learn/data-fetching/pre-rendering.png"  alt="pre-rendering" width="655" height="390"></p>

- By default, Next.js `pre-renders every page`. This means that Next.js `generates HTML for each page in advance`, instead of having it all done by client-side JavaScript. _**Pre-rendering**_ can result in _**better performance**_ and _**SEO**_.

- Each generated HTML is associated with `minimal JavaScript code` necessary for that page. When a _**page is loaded**_ by the browser, its _**JavaScript code runs**_ and makes the _**page fully interactive**_.

  &nbsp;

## 2 forms of Pre-rendering

- Next.js has 2 forms of pre-rendering: _**Static Generation**_ and _**Server-side Rendering**_. The differences is in `when it generates the HTML for a page`

  - _**Static Generation**_ (Recommended): The HTML is generated `at build time` and will be reused on each request.
  - _**Server-side Rendering**_: The HTML is generated `on each request`.

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

- If a page uses Static Generation, the page HTML is `generated at build time`. That means in production, the _**page HTML is generated**_ when you run `next build`. This HTML will then be reused on each request. It can _**be cached by a CDN**_.

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

- These are 2 scenarios, and _**one or both might apply**_. In each case, you can use these functions that Next.js provides:

  - Your page _**content**_ depends on external data: Use `getStaticProps`
  - Your page _**paths**_ depend on external data: Use `getStaticPaths` (usually in addition to `getStaticProps`)

&nbsp;
&nbsp;

## Scenario 1: Your _page content_ depends on external data

- `getStaticProps` runs _**at build time**_ in production, and inside it, you can `fetch external data` and `send it as props to the page`.

<p align="center"><img src="https://nextjs.org/static/images/learn/data-fetching/index-page.png"  alt="data-fetching" width="576" height="486"></p>
&nbsp;

- Essentially, `getStaticProps` allows you to tell Next.js: "Hey, this page has some data dependencies - so when you _**pre-render this page at build time**_, make sure to `resolve them first`!"

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

## Scenario 2: Your _page paths_ depend on external data

- Next.js allows you to create pages with `Dynamic Routes`. You can create a file called `pages/posts/[id].js` to show a single blog post based on `id`. This will allow you to show a blog post with `id: 1` when you access `posts/1`.

- However, which `id` you want to pre-render at `build time` might depend on `external data`. So your page `paths` that are pre-rendered depend on `external data`.

<p align="center"><img src="https://nextjs.org/static/images/learn/dynamic-routes/page-path-external-data.png"  alt="page-path-external-data" width="557" height="470"></p>
&nbsp;

- To handle this, Next.js lets you `export` an `async` function called `getStaticPaths` from a _**dynamic page**_ (`pages/posts/[id].js` in this case). This function gets called `at build time` and lets you specify which paths you want to pre-render

```javascript
// This function gets called at build time
export async function getStaticPaths() {
  // Call an external API endpoint to get posts
  const res = await fetch("https://.../posts");
  const posts = await res.json();

  // Get the paths we want to pre-render based on posts
  const paths = posts.map((post) => ({
    params: { id: post.id },
  }));

  // We'll pre-render only these paths at build time. { fallback: false } means other routes should 404.
  return { paths, fallback: false };
}
```

&nbsp;

- Also in `pages/posts/[id].js`, you need to export `getStaticProps` so that you can `fetch the data` about the post with this `id` and use it to pre-render the page.

```javascript
function Post({ post }) {
  // Render post...
}

export async function getStaticPaths() {
  // ...
}

// This also gets called at build time
export async function getStaticProps({ params }) {
  // params contains the post `id`. If the route is like /posts/1, the params.id is 1
  const res = await fetch(`https://.../posts/${params.id}`);
  const post = await res.json();

  // Pass post data to the page via props
  return { props: { post } };
}

export default Post;
```

&nbsp;

#### Here's an example.

- We want the URL for these pages to depend on the blog data, which means we need to use `Dynamic Routes`. Next.js allows you to statically generate pages with paths that depend on external data. This enables `Dynamic URLs` in Next.js.

<p align="center"><img src="https://nextjs.org/static/images/learn/dynamic-routes/how-to-dynamic-routes.png"  alt="how-to-dynamic-routes" width="535" height="453"></p>
&nbsp;

- We'll create a page called `[id].js` under `pages/posts`. Pages that begin with `[` and end with `]` are `Dynamic Routes` in Next.js
- In `pages/posts/[id].js`, we'll write code that will render a post page - just like other pages we've created.

```javascript
import Layout from "../../components/layout";

export default function Post() {
  return <Layout>...</Layout>;
}
```

&nbsp;

- export an async function called `getStaticPaths` from this page. In this func, we need to `return a list` of possible values for `id`.

```javascript
import Layout from "../../components/layout";

export default function Post() {
  return <Layout>...</Layout>;
}

export async function getStaticPaths() {
  // Return a list of possible value for id
}
```

&nbsp;

- Finally, we need to implement `getStaticProps` again - this time, to fetch necessary data for the blog post with a given `id`. `getStaticProps` is given `params`, which contains `id` (because the file name is `[id].js`).

```javascript
import Layout from "../../components/layout";

export default function Post() {
  return <Layout>...</Layout>;
}

export async function getStaticPaths() {
  // Return a list of possible value for id
}

export async function getStaticProps({ params }) {
  // Fetch necessary data for the blog post using params.id
}
```

&nbsp;

- open `lib/posts.js` and add the following `getAllPostIds` function at the bottom. It will return the list of file names (excluding `.md`) in the `posts` directory

```javascript
export function getAllPostIds() {
  const fileNames = fs.readdirSync(postsDirectory);

  return fileNames.map((fileName) => {
    return {
      params: {
        id: fileName.replace(/\.md$/, ""),
      },
    };
  });
}
```

- _**Important**_: The returned list is not just an array of strings - it must be an `array of objects`. Each object must have the `params` key and contain an object with the `id` key (because we're using `[id]` in the file name). Otherwise, `getStaticPaths` will fail.

&nbsp;

- Finally, we'll import the `getAllPostIds` function and use it inside `getStaticPaths`. Open `pages/posts/[id].js` and copy the following code above the exported `Post` component:

```javascript
import { getAllPostIds } from "../../lib/posts";

export async function getStaticPaths() {
  const paths = getAllPostIds();
  return {
    paths,
    fallback: false,
  };
}
```

- `paths` contains the array of known paths returned by `getAllPostIds()`, which include the params defined by `pages/posts/[id].js`.

&nbsp;

- We need to fetch necessary data to render the post with the given `id`. To do so, open `lib/posts.js` again and add the following `getPostData` function at the bottom. It will return the post data based on `id`:

```javascript
export function getPostData(id) {
  const fullPath = path.join(postsDirectory, `${id}.md`);
  const fileContents = fs.readFileSync(fullPath, "utf8");

  // Use gray-matter to parse the post metadata section
  const matterResult = matter(fileContents);

  // Combine the data with the id
  return {
    id,
    ...matterResult.data,
  };
}
```

&nbsp;

- Then, open `pages/posts/[id].js` and add the following code:

```javascript
import { getAllPostIds, getPostData } from "../../lib/posts";

export async function getStaticProps({ params }) {
  const postData = getPostData(params.id);
  return {
    props: {
      postData,
    },
  };
}
```

- The post page is now using the `getPostData` function in `getStaticProps` to `get the post data` and `return it as props`.

&nbsp;

- update the `Post` component to use `postData`. In `pages/posts/[id].js` replace the exported `Post` comp with the following code

```javascript
export default function Post({ postData }) {
  return (
    <Layout>
      {postData.title}
      <br />
      {postData.id}
      <br />
      {postData.date}
    </Layout>
  );
}
```

&nbsp;

- That's it! Try visiting these following pages. We've successfully generated `Dynamic Routes`.

  - http://localhost:3000/posts/ssg-ssr
  - http://localhost:3000/posts/pre-rendering

&nbsp;

---

Reference:
