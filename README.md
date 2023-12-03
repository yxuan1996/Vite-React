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

## CRUD Methods
### Loading Data (GET API)
We can use `loader` and `useLoaderData` to load data into React

- In our root component, we define a loader. 
- The loader will make an API call and return the results

getContacts is a mock API. In real-life, the getContacts function should make an API request to an endpoint and return actual data. 
`root.jsx`
```JSX

import { getContacts } from "../contacts";

export async function loader() {
  const contacts = await getContacts();
  return { contacts };
}
```

In `main.jsx` we add the loader to the route
```JSX
import Root, { loader as rootLoader } from "./routes/root";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
    loader: rootLoader,
    children: [
      {
        path: "contacts/:contactId",
        element: <Contact />,
      },
    ],
  },
]);
```

Lastly we render the data in the root component. The data is a series of contacts that will be shown in the side navbar. 

```JSX
import {
  Outlet,
  Link,
  useLoaderData,
} from "react-router-dom";

export default function Root() {
  // Retrieve the loader data here
  const { contacts } = useLoaderData();
  return (
    <>
      <div id="sidebar">
        <h1>React Router Contacts</h1>
        {/* other code */}

        <nav>
         {/* Use the data here - See root.jsx for full implementation*/}
        </nav>

        {/* other code */}
      </div>
    </>
  );
}
```

Note that we also create a loader for individual contact data in `contact.jsx`. This loader will take in `params` and retrieve a single contact data based on the param id. 

### Form Data (POST)
By default, when a HTML form is submitted, a browser will serialize the form data into the `request body` and send a POST request to the server. 

To use client-side routing, we change `<form>` to `<Form>` from react-router-dom

- We also need to define `action()`. 
- `action()` acts like an event handler to process the submitted form data
- `action()` is triggered by a POST request to the same URL. 

Steps:
- In `root.jsx` we define an action that will handle the form submission data. 
- action will call an API to create a new form entry in the DB. 

```JSX
import {
  Outlet,
  Link,
  useLoaderData,
  Form,
} from "react-router-dom";
import { getContacts, createContact } from "../contacts";

export async function action() {
  const contact = await createContact();
  return { contact };
}
```

- Further down in `root.jsx` we define our form
```JSX
<Form method="post">
    <button type="submit">New</button>
</Form>
```

- In `main.jsx` we add the action to the route. 
```JSX
import Root, {
  action as rootAction,
} from "./routes/root";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
    loader: rootLoader,
    action: rootAction,
    children: [
      {
        path: "contacts/:contactId",
        element: <Contact />,
      },
    ],
  },
]);
```

### Updating Data
First we create a form in `edit.jsx`. We also define the action to handle form submission. 

In `main.jsx` we refine a new route in React Router `contacts/:contactId/edit`. We also add the action to the route. 



