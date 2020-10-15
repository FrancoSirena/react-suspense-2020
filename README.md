# React Suspense & Concurrent Mode

## How to navigate this repo
App.tsx is the main component with the routing logic and some lazy loading there, to see documented way of using React.lazy + Suspense

Inside the Components we can see all components, using somewhat Suspense + API techniques, which we achieve by using the `createFetchResource` method.

The UserDetails has two Suspense plus a SuspenseList to orchestrate that, we do it because we have 2 APIs there and they can resolve in different timings.
The UserUserDetails has a back button that uses the `useTransition` hook to wait before transitioning to the new component, so we don't see the empty page with a loading if the new component resolves fast enough ( timeoutMS is the enough here ).

The utils/createResource has some interesting techniques about how to keep a cachine system to orchestrate API calls, this is super basic and it is not production ready, but it gives us a glimpsy of what we can achieve using it.
Other things to point out are the super NavLink and Link which  triggers the API before the user even gets to those routes, by using the `onMouseEnter` listener.

Inside the `types` folder it is just that, the typescript `types` and `interfaces`

## Suspense with Lazy loading.

Just like we've been using at Bitsight, regular stuff React.lazy(() => import) and a Suspense around it to show the fallback during loading.
Note to self here, we can use pre fetch mode with webpack annotations for things we know we are going to need soon.
```javascript
import React from 'react'
const ComponentSpecial = React.lazy(() => import('./components/componentOne');
const ComponentSpecialTwo = React.lazy(() => import('./components/componentTwo');

function MyApp() {
  const { user: { hasSpecial, hasSpecialTwo } } = React.useContext(MyApp);
  return (
    <div>
      Default thing that everybody should see
      <React.Suspense fallback={() => <> loading </>}>
        {hasSpecial && <ComponentSpecial />}
        {hasSpecialTwo && <ComponentSpecialTwo />}
      </React.Suspense>
    </div>
```

## Using Suspense with APIs.

You throw PROMISES and then the closest Suspense component will show the fallback.
The same should happen for errors, but then it gets caught by the ErrorBoundary component, which makes the components much cleaner and easier to read.

We can create a resource file to manage that, to see what APIs are happening and what are not, which is awesome.

```javascript
const requests;

const makeRequest  = url => {
  if (requests[url]?.fulfill) {
    return requests[url]?.fulfill;
  } else if (requests[url]?.pending) {
    throw requests[url].pending
  } else {
    requests[url] = {};
    requests[url].pending = fetch(url).then(async r => {
      if (!r.ok) {
        requests[url] = null;
        throw Error(r.statusMessage)
      }
      requests[url].fulfill = await r.json();
    }
    throw requests[url].pending;
  }
}

function Wrapper() {
  return (
    <ErrorBoundary>
      <React.Suspense fallback="loading">
        <Component />
      </React.Suspense>
    </ErrorBoundary>
  )

const Component =  () => {
  const data = makeRequest('http://fake.ui')
  
  return (<div>{data.something}</div>)
}
```

## Concurrent Mode

A way to batch updates to the DOM where each batch update will stop for a few ms to see if there is a higher priority task to be executed, e.g. User events, so we can unblock the main thread during a render.
That is being done trying to batch updates in chunks of 16ms to fit into the 60 frames per seconds that all major browsers use.

### SuspenseList

A nice way to handle parallel Suspense components, where you can orchestrate the order where the Suspense components should work

### useTransition

A way to handle transitions between renders, so we can start a transition before actually running any heavy operation, that returns a startTransition and a isPending bool, which can be used to render your component in a different way to indicate that something is happening.
