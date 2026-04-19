---
trigger: always_on
description: use is a React API that lets you read the value of a resource like a Promise or context.
---

use
use is a React API that lets you read the value of a resource like a Promise or context.

const value = use(resource);
Reference
use(resource)
Usage
Reading context with use
Streaming data from the server to the client
Dealing with rejected Promises
Troubleshooting
“Suspense Exception: This is not a real error!”
Reference 
use(resource) 
Call use in your component to read the value of a resource like a Promise or context.

import { use } from 'react';

function MessageComponent({ messagePromise }) {
  const message = use(messagePromise);
  const theme = use(ThemeContext);
  // ...
Unlike React Hooks, use can be called within loops and conditional statements like if. Like React Hooks, the function that calls use must be a Component or Hook.

When called with a Promise, the use API integrates with Suspense and error boundaries. The component calling use suspends while the Promise passed to use is pending. If the component that calls use is wrapped in a Suspense boundary, the fallback will be displayed.  Once the Promise is resolved, the Suspense fallback is replaced by the rendered components using the data returned by the use API. If the Promise passed to use is rejected, the fallback of the nearest Error Boundary will be displayed.

See more examples below.

Parameters 
resource: this is the source of the data you want to read a value from. A resource can be a Promise or a context.
Returns 
The use API returns the value that was read from the resource like the resolved value of a Promise or context.

Caveats 
The use API must be called inside a Component or a Hook.
When fetching data in a Server Component, prefer async and await over use. async and await pick up rendering from the point where await was invoked, whereas use re-renders the component after the data is resolved.
Prefer creating Promises in Server Components and passing them to Client Components over creating Promises in Client Components. Promises created in Client Components are recreated on every render. Promises passed from a Server Component to a Client Component are stable across re-renders. See this example.
Usage 
Reading context with use 
When a context is passed to use, it works similarly to useContext. While useContext must be called at the top level of your component, use can be called inside conditionals like if and loops like for. use is preferred over useContext because it is more flexible.

import { use } from 'react';

function Button() {
  const theme = use(ThemeContext);
  // ...
use returns the context value for the context you passed. To determine the context value, React searches the component tree and finds the closest context provider above for that particular context.

To pass context to a Button, wrap it or one of its parent components into the corresponding context provider.

function MyPage() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  // ... renders buttons inside ...
}
It doesn’t matter how many layers of components there are between the provider and the Button. When a Button anywhere inside of Form calls use(ThemeContext), it will receive "dark" as the value.

Unlike useContext, use can be called in conditionals and loops like if.

function HorizontalRule({ show }) {
  if (show) {
    const theme = use(ThemeContext);
    return <hr className={theme} />;
  }
  return false;
}
use is called from inside a if statement, allowing you to conditionally read values from a Context.



Pitfall
Like useContext, use(context) always looks for the closest context provider above the component that calls it. It searches upwards and does not consider context providers in the component from which you’re calling use(context).



Example Code:
import { createContext, use } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  )
}

function Form() {
  return (
    <Panel title="Welcome">
      <Button show={true}>Sign up</Button>
      <Button show={false}>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = use(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ show, children }) {
  if (show) {
    const theme = use(ThemeContext);
    const className = 'button-' + theme;
    return (
      <button className={className}>
        {children}
      </button>
    );
  }
  return false
}



Streaming data from the server to the client 
Data can be streamed from the server to the client by passing a Promise as a prop from a Server Component to a Client Component.

import { fetchMessage } from './lib.js';
import { Message } from './message.js';

export default function App() {
  const messagePromise = fetchMessage();
  return (
    <Suspense fallback={<p>waiting for message...</p>}>
      <Message messagePromise={messagePromise} />
    </Suspense>
  );
}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/heaneySam) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
