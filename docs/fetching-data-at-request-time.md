# 🐋 Fetching data at Request time (SSR, SG with CSR)

- If your page shows `frequently Updated Data`, and the page _**content changes on every request**_, you can do one of the following:

  - Use _**Server-Side Rendering**_: Next.js `pre-renders` a page on `each request`. It will be _**slower**_ because the page cannot be cached by a CDN, but the pre-rendered page will `always be up-to-date`.

  - Use Static Generation with _**Client-side Rendering**_: You can `skip pre-rendering` some parts of a page and then use `client-side JavaScript` to populate them.

&nbsp;

## Server-side Rendering

- If you need to `fetch data at request time` instead of at build time, you can try `Server-side Rendering`.

<p align="center"><img src="https://nextjs.org/static/images/learn/data-fetching/server-side-rendering-with-data.png"  alt="server-side-rendering-with-data" width="618" height="521"></p>
&nbsp;

- To use Server-side Rendering, you need to export `getServerSideProps` instead of `getStaticProps` from your page.
- If you export an `async` function called `getServerSideProps` from a page, Next.js will _**pre-render this page on each request**_ using the data returned by `getServerSideProps`.

```javascript
export async function getServerSideProps(context) {
  return {
    props: {}, // will be passed to the page component as props
  };
}
```

&nbsp;

- Because `getServerSideProps` is called _**at request time**_, its parameter (`context`) contains request _**specific parameters**_. The `context` parameter is _**an object**_ containing the following keys:

  - `params`: If this page uses a _**dynamic route**_, `params` contains _**the route parameters**_. If the page name is `[id].js`, then `params` will look like `{ id: ... }`.
  - `req` : [The HTTP IncomingMessage object](https://nodejs.org/api/http.html#http_class_http_incomingmessage), plus additional built-in parsing helpers.
  - `res` : [The HTTP response object](https://nodejs.org/api/http.html#http_class_http_serverresponse).
  - `query` : An object representing _**the query string**_.
  - ...........

&nbsp;

- `getServerSideProps` should return _**an object**_ with:

  - `props`: An _**optional object**_ with the props that will _**be received by the page component**_. It should be a _**serializable object**_ or a Promise that resolves to a serializable object
  - `notFound`: An _**optional boolean**_ value to allow the page to _**return a 404 status and page**_.
  - `redirect`: An _**optional redirect value**_ to allow redirecting to internal and external resources. It should match the shape of `{ destination: string, permanent: boolean }`.
  - ............

```javascript
export async function getServerSideProps(context) {
  const res = await fetch(`https://...`);
  const data = await res.json();

  if (!data) {
    return {
      notFound: true,
      redirect: {
        destination: "/",
        permanent: false,
      },
    };
  }

  return {
    props: {}, // will be passed to the page component as props
  };
}
```

&nbsp;

#### Here's an example

- Here's an example which uses `getServerSideProps` to fetch data _**at request time**_ and pre-renders it.

```javascript
function Page({ data }) {
  // Render data...
}

// This gets called on every request
export async function getServerSideProps() {
  // Fetch data from external API
  const res = await fetch(`https://.../data`);
  const data = await res.json();

  // Pass data to the page via props
  return { props: { data } };
}

export default Page;
```

&nbsp;

- You should use `getServerSideProps` only if you need to _**pre-render a page**_ whose data must _**be fetched at request time**_. Time to first byte ([TTFB](https://web.dev/time-to-first-byte/)) will be slower than `getStaticProps` because the server must _**compute the result on every request**_, and the result _**cannot be cached by a CDN**_ without extra configuration.

&nbsp;

## Static Generation with Client-side Rendering

- If you `do not` need to `pre-render the data`, you can fetch also use the following strategy

  - `Statically generate` (_**pre-render**_) parts of the page that do not require external data
  - When the page loads, `fetch external data` from the `client using JavaScript` and populate the remaining parts.

<p align="center"><img src="https://nextjs.org/static/images/learn/data-fetching/client-side-rendering.png"  alt="client-side-rendering" width="529" height="806"></p>
&nbsp;

- This approach works well for `User Dashboard pages`. Because a dashboard is a _**private**_, _**user-specific**_ page, _**SEO is not relevant**_ and the page _**doesn't**_ need to _**be pre-rendered**_. The data is `frequently updated`, which requires `request-time data fetching`

&nbsp;

### SWR

- The team behind Next.js has created a React hook for `data fetching` called `SWR`. We highly recommend it if you're `fetching data` on the `client side`. It handles _**caching**_, _**revalidation**_, _**focus tracking**_, _**refetching on interval**_, and more.

```javascript
import useSWR from "swr";

const fetcher = (url) => fetch(url).then((res) => res.json());

function Profile() {
  const { data, error } = useSWR("/api/user", fetcher);

  if (error) return <div>failed to load</div>;
  if (!data) return <div>loading...</div>;
  return <div>hello {data.name}!</div>;
}
```

&nbsp;

---

Reference: https://nextjs.org/learn/basics/data-fetching/request-time, https://nextjs.org/docs/basic-features/data-fetching#fetching-data-on-the-client-side, https://nextjs.org/docs/basic-features/pages#pre-rendering
