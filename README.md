# Vite-React

Creating a React app using Vite

```
npm create vite@latest my-first-react-app -- --template react
```

To run the app
```
cd my-first-react-app
npm install
npm run dev
```

## React Router
We will be implementing React Router by following along the tutorial: https://reactrouter.com/en/main/start/tutorial

Installing React Router
```
npm install react-router-dom localforage match-sorter sort-by
```

- In `main.jsx`, we import React Router and create a browser router by defining the paths.
- In `src/routes` we define our route components. 
- React Router has built in error handling. 

```JSX
import {
  createBrowserRouter,
  RouterProvider,
} from "react-router-dom";
import Root from "./routes/root";
import ErrorPage from "./error-page";

...
const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
  },
]);

...
ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>,
)

```

#### Rendering components inside the root layout
Sometimes, we want a child component to be rendered inside a root layout. 

For example, we may have a root layout with the same headers, navbar and footers, and we want all pages to be rendered within this root layout. 

We can do so be making a route a child of the root route in `main.jsx`. 

Rendering the contacts route within the root route
```JSX
const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
    children: [
      {
        path: "contacts/:contactId",
        element: <Contact />,
      },
    ],
  },
]);

```

Inside our root component, we need to define where exactly do we want to render our child component. We use `< Outlet />`

`root.jsx`
```JSX
import { Outlet } from "react-router-dom";

export default function Root() {
  return (
    <>
      {/* all the other elements */}
      <div id="detail">
        <Outlet />
      </div>
    </>
  );
}
```

#### Server-side routing vs Client-side routing
Using a tags in HTML will result in a server request. The webpage will refresh the page by requesting information from the given URL. 

```JSX
<a href={`/contacts/1`}>Your Name</a>
```

To use client-side routing, we need to replace our a tags with `< Link to >`

```JSX
import { Outlet, Link } from "react-router-dom";

<Link to={`contacts/1`}>Your Name</Link>
```


