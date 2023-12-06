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
import {Outlet,Link,useLoaderData,} from "react-router-dom";

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
- After this we want to redirect to the edit/update page, so that the user can enter new data. 

```JSX
import {
  Outlet,
  Link,
  useLoaderData,
  Form,
  redirect,
} from "react-router-dom";
import { getContacts, createContact } from "../contacts";

export async function action() {
  const contact = await createContact();
  return redirect(`/contacts/${contact.id}/edit`);
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
- First we create a form in `edit.jsx`. We also define the action to handle form submission. 
- The action takes in both `request` and `params`
- We use `request` to access the submitted form data. 
- We use params to get the id from the URL. 

```JSX
export async function action({ request, params }) {
  const formData = await request.formData();
  const updates = Object.fromEntries(formData);
  await updateContact(params.contactId, updates);
  return redirect(`/contacts/${params.contactId}`);
}
```

- `Object.fromEntries()` converts the form data into an object. We can use object notation to access form values. 
```JSX
const updates = Object.fromEntries(formData);
updates.first; // "Some"
updates.last; // "Name"
```

In `main.jsx` we refine a new route in React Router `contacts/:contactId/edit`. We also add the action to the route. 

### Deleting Data
- In `contacts.jsx` page, we see that there is an edit button and a delete button. 
- Each button is wrapped up in a `<Form>` tag
- The delete button has a `post` method and a `destroy` action
- Since the form URL is `contact/:contactId`, the button will submit the post request to `contact/:contactId/destroy` when clicked. 

Steps:
- We define a new route called `destroy.jsx`. This route has an action that deletes a single item from contacts.
- We add the route and the action to the router in `main.jsx`

## Index Page
- When we first start the app, none of the child components are selected and the root component is rendered with an empty page. 
- We want to render a default component. This will serve as the index page. 

In `main.jsx`, `index: true` tells the app to render this route when no other child routes are selected. 

```JSX
{ index: true, element: <Index /> },
```

## Search
- In `root.jsx` we have a search form that filter the list of entries in the navbar. 
- Since we did not specify the method, the form uses the default GET request. 
- Form inputs will appear as URL Params based on their name. For example, if we have 2 inputs with the names "first" and "last", the URL will be `[https://[URL]/first=[?]&last=[?]]`

In `root.jsx` the contacts in the navbar are loaded using a `loader` function. We can modify the loader function to filter for contacts based on the URL params. 

```JSX
export async function loader({ request }) {
  const url = new URL(request.url);
  const q = url.searchParams.get("q");
  const contacts = await getContacts(q);
  return { contacts };
}
```

### Synchronizing URL to form state
We want to ensure that our URL and form state is always in sync. Our URL can change when the user clicks the back button. On the other hand, our form state may change when the user refreshes the page. 

To keep the states in sync, we will use React `useState` and `useEffect` hooks. 

To start, we modify our loader function to return both the `contacts` and the `query`. We also set the default form input to the value of q.

```JSX
export async function loader({ request }) {
  const url = new URL(request.url);
  const q = url.searchParams.get("q") || "";
  const contacts = await getContacts(q);
  return { contacts, q };
}

const { contacts, q } = useLoaderData();
const [query, setQuery] = useState(q);

// Sync the state when the URL Param changes
useEffect(() => {
    setQuery(q);
  }, [q])

// Inside the form input
value={query}
  onChange={(e) => {
    setQuery(e.target.value);
  }}
```

### Filter as we type
We want our list to be filtered as we type, instead of when we click the submit button. 

We import the `useSubmit` module and attach it to the `onChange` event handler of the input field. Now, whenever the input field changes, the form is auto-submitted, allowing us to filter as we type. 

`root.jsx`
```JSX
import {
  // existing code
  useSubmit,
} from "react-router-dom";

const submit = useSubmit();

// Inside the form input
onChange={(event) => {
  submit(event.currentTarget.form);
}}
```

### Managing Form Submissions
Since a new form is submitted every time we type, we end up with multiple form submissions in the browser. This is bad because:

- Our browser search history becomes full
- We cannot go back to the previous page easily. For example ([q=John, q=Joh, q=Jo, q=J])

We need to replace our current entry in the history stack with the next page. 
```JSX
onChange={(event) => {
  setQuery(event.target.value);
  const isFirstSearch = q == null;
  submit(event.currentTarget.form, {
    replace: !isFirstSearch,
  });
}}
```

## Change Data State (Mutations) without Navigation
The `useFetcher` hook allows us to communicate with loaders and actions without navigating away from a page. This is useful for toggle actions and ratings. 

Examples:
- Setting an item as a favourite
- Toggle yes/no settings
- Giving a rating to a product
- Adding an item to a shopping cart?

In `contact.jsx` we want to set a contact as our favourites. 
We wrap our button inside `fetcher.Form`

```JSX
import {
  ...
  useFetcher,
} from "react-router-dom";

// Inside the Favourite function
const fetcher = useFetcher();

<fetcher.Form method="post">
      <button
        name="favorite"
        value={favorite ? "false" : "true"}
        aria-label={
          favorite
            ? "Remove from favorites"
            : "Add to favorites"
        }
      >
        {favorite ? "★" : "☆"}
      </button>
    </fetcher.Form>
```

`fetcher.Form` will create a POST request to the same URL as the contact form `contacts/:contactId`

Next we create an action in `contact.jsx` to handle the form submission. 

```JSX
import { getContact, updateContact } from "../contacts";

export async function action({ request, params }) {
  let formData = await request.formData();
  return updateContact(params.contactId, {
    favorite: formData.get("favorite") === "true",
  });
}
```

Lastly we connect the action to the router

`main.jsx`
```JSX
import Contact, {
  loader as contactLoader,
  action as contactAction,
} from "./routes/contact";

{
        path: "contacts/:contactId",
        element: <Contact />,
        loader: contactLoader,
        action: contactAction,
},
```

## Navigation and Styling
### Active Link
When the links on the navbar are clicked, we want to show the link that is currently active. 

We can use the NavLink component to do so and set the isActive and isPending classes. 

`root.jsx`
```JSX
import {Outlet,NavLink,useLoaderData,Form,redirect,} from "react-router-dom";

<nav>
          {contacts.length ? (
            <ul>
              {contacts.map((contact) => (
                <li key={contact.id}>
                  <NavLink
                    to={`contacts/${contact.id}`}
                    className={({ isActive, isPending }) =>
                      isActive
                        ? "active"
                        : isPending
                        ? "pending"
                        : ""
                    }
                  >
                    {/* other code */}
                  </NavLink>
                </li>
              ))}
            </ul>
          ) : (
            <p>{/* other code */}</p>
          )}
        </nav>
```

### Global Pending UI
When a user navigates away from a page, we want to provide the user with some feedback so that the app does not feel unresponsive. 

The `useNavigation()` hook allows us to set transtion states. 

When the 'searching' state is set to `loading` we want to show a search spinner. 

`root.jsx`

```JSX
import {
  // existing code
  useNavigation,
} from "react-router-dom";

const navigation = useNavigation();

const searching =
navigation.location &&
new URLSearchParams(navigation.location.search).has(
  "q"
);

// Add this class to the form input
className={searching ? "loading" : ""}

// Add this class to the sidebar
className={
  navigation.state === "loading" ? "loading" : ""
}

// Only show when searching state has value
<div
  id="search-spinner"
  aria-hidden
  hidden={!searching}
/>
```

### Back Button
On the edit page, we have a cancel button. We want the button to do the same thing as the browser's back button. 

In `edit.jsx` we import the `useNavigate()` module and use `navigate(-1)` to go back to the previous page

```JSX
import {
  ...
  useNavigate,
} from "react-router-dom";

const navigate = useNavigate();

<button
  type="button"
  onClick={() => {
    navigate(-1);
  }}
>
Cancel
</button>
```

### Error Handling for All Routes
We want our error pages to render inside our root component. 

To accomplish this, we wrap our error element inside our children in the router. 