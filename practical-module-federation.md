# Practical Module Federation

My highlights and notes from the book. For full content go to https://module-federation.github.io/


## 1.1 Micro-frontends (MFE) and module federation (MF)

### What
MFEs are:
- self contained: contain all the business logic specific to their behavior
- independently deployable, either as:
  - build-time deployment: separate modules, available at build time. Deployed all at once.
  - runtime deployment: MFEs are deployed independently and consumed by the host at runtime
- hosted on a page, often with an (application) shell that provides user id, auth, API access, etc

### Why MFEs?

Because independently deployable: 
- ability to deploy a single MFE, yet update the whole app (runtime deployment)
- reduce build time by splitting code
- allow maintenance by different teams

A way to split a monolith into a polylith.


### Runtime deployed MFEs

Two apps (ex home & search) needing the same component can use a shared NPM library.

But:
- it's convoluted to work with: switch project to make the change in the shared lib and fix tests and bump the version, then bump lib version in all apps.
- no guarantee both apps will use the same version.

Runtime deployment: either build your own shim (dynamic loading) or use module federation's runtime dependency (remotes)

Before bundlers, runtime deps weren't an issue (requireJS, plain UMD+script tags, etc).

Bundlers were introduced to have a single file to deploy, but lost the runtime loadable deps aspect.

### Module federation advantages

- No framework to glue the MFEs (transparent if all MFEs use the same view framework)
- Share any JS code vs view-layer only components: any JS code can be shared (business logic, state, helpers, etc)
- Universal: MF can be used with different module types (UMD, SystemJS, AMD, CommonJS, window variable, etc)
- No code loader required: dynamic imports are transparent, no loader like SystemJS is required

### Framework neutrality

If apps uses multiple view technologies (React, Vue, Svelte): use [Single-SPA](https://single-spa.js.org/). It still requires dynamic loading, via MF or SystemJS.


### Composability

Composability is more than sharing at page/route level. It should also be done at element-level within each page (ex: sharing an "add to cart" component).


### Caveat

MF is not the only way for MFE.

Other solutions: runtime: Bit, or build time (i.e, not dynamic): shared NPM library

Authors disclaimer: it's very complex: CORS, library versioning, and singletons, debugging.

> Make no mistake; you will run into issues implementing Module Federation.
> Having strong debugging skills and debugging complex issues is a prerequisite for
>using federated modules successfully. This is not a core stability issue with Module
>Federation itself, which is battle tested, these are issues of library compatibility,
>deployment issues and more that are layered on top of Module Federation.


## 1.2 Micro-frontends (MFE) and module federation (MF)

## Vocabulary
Concepts:
- `remotes`: the name of other federated module application(s) that the current application consumes from
- `exposes`: the files the current application exposes as remotes to other applications
- `shared`: a list of libraries shared with other applications, to support the entries in `exposes`.

### How to configure
In `webpack.config.js`'s section `plugins`: add a `ModuleFederationPlugin` instance.

Restart your app to take changes into account.

### Properly sharing react

```javascript
const deps = require("./package.json").dependencies;

shared: {
  ...deps,
  react: {
    singleton: true,
    requiredVersion: deps.react,
  },
  "react-dom": {
    singleton: true,
    requiredVersion: deps["react-dom"],
  },
},
```

Shared as singleton (to preserve internal state), with pinned version.

Other dependencies are automatically shared (cf `...deps`).

Most CSS-in-JS libs are also singletons, such as `@emotion/core`.

## 1.3 Getting started with Create React App

## 1.4 How module federation works

### Build time
Heavy lifting is done by 3 plugins

- `ContainerPlugin`: create container entries with the specified exposed modules (`exposed`). Create the remote entry file (acts as app manifest, aka `scope`) Used by remotes.

- `ContainerReferencePlugin`: manages the `remotes` listed in the config. Used by a host.

- `SharePlugin`: managed the `shared` part of the config: handles versioning requirements (aka `overrides`). Used by host and remotes.


Key files:

- `remoteEntry.js`: the manifest and specialized runtime for the exported remote modules and any shared packages

- `src_x_js.js`: compiled JS code for a shared component *X*. Referenced in the `remoteEntry.js`

- `vendors-node_modules_y_js.js`: compiled JS code for a shared lib *Y* that supports an exposed component. Ex Y=react_index for react.

These files are a) JS bundles required to run the application, b) JS bundles required for the remote modules

It can overlap: for example, some vendor bundles like React are used in both cases.

### Runtime

> When the `remoteEntry.js` is loaded by the browser it registers a global variable with the name specified in the library key in the ModuleFederationPlugin configuration. This variable has two things in it, a get function that returns any of the remote modules, and an override function that manages all of the shared packages.

E.g, given a remote nav exposing a runtime module Header:

```javascript
window.nav.get('Header').then(factory => console.log(factory()));
```

This outputs the module as defined in the Header implementation. 

It will also *load any shared packages required* by this module, reusing already loaded ones, if any and if version matches.

## 1.5 Deploying federated modules

Set `publicPath` based on where the module will be deployed (server, S3, CDN, ...)

It can be adjusted based on webpack argument's `mode` / `env` vars, or set to `auto`.

Same for `remotes`'s URLs. It's also possible to set `remotes`' origin dynamically at runtime with plugin [external-remotes-plugin](https://github.com/module-federation/external-remotes-plugin)

## 1.6 Resilient sharing of React components

How to:
- dynamic import with `React.lazy`
- within an obligatory `React.Suspense`, with (optional) fallback content displayed during loading
- within an `ErrorBoundary` for error handling (remote down, or remote code throwing an `Error`)

```javascript
// *important* bootstrap code:
// A dedicated import() to give webpack an opportunity to process all imports, and load code for all remotes
import("./App");
```

```javascript
// App.jsx
const Header = wrapComponent(React.lazy(() => import("nav/Header")));

const App = () => (
  <Header
    error={<div>Error!</div>}
    delayed={<div>Loading...</div>}
    />
);

ReactDOM.render(<App />, document.getElementById("app"));
```

High Order Component for loading and error handling:
```javascript
const wrapComponent = (Component) => ({ error, delayed, ...props }) 
=> (
    <FederatedWrapper error={error} delayed={delayed}>
      <Component {...props} />
    </FederatedWrapper>
);
```

```javascript
// https://github.com/jherr/practical-module-federation-20/blob/main/part1-getting-started/resilient-sharing/packages/host/src/App.jsx
class FederatedWrapper extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.error || <div>Something went wrong.</div>;
    }

    return (
      <React.Suspense fallback={this.props.delayed || <div />}>
        {this.props.children}
      </React.Suspense>
    );
  }
}
```

Nb: To avoid displaying the `Suspense`'s fallback again after initial load (e.g when navigating between 2 `Lazy` loaded components), use [Transitions](https://reactjs.org/docs/react-api.html#starttransition)

> Transitions are a new concurrent feature introduced in React 18. They allow you to mark updates as transitions, which tells React that they can be interrupted and avoid going back to Suspense fallbacks for already visible content.

## 1.7 React state sharing options

Use react and react-dom as singletons

```javascript
const deps = require("./package.json").dependencies;

shared: {
  ...deps,
  react: {
    singleton: true,
    requiredVersion: deps.react,
  },
  "react-dom": {
    singleton: true,
    requiredVersion: deps["react-dom"],
  },
},
```

There are multiple options for state management, cf below and https://github.com/jherr/practical-module-federation-20/tree/main/part1-getting-started/state-sharing

### useState

Use props to pass the value and setter to the remote component.

Host:
```javascript
const [itemCount, setItemCount] = useState(0);

<React.Suspense fallback={<div />}>
  <Header count={itemCount} onClear={() => setItemCount(0)} />
</React.Suspense>
```

Remote:
```javascript
const Header = ({ count, onClear }) => {
  return (
    <header>
      <div>Count is {count}</div>
      <button onClick={onClear}>Clear</button>
    </header>
  );
};
```

### useContext

Pass via React's context to avoid props drilling.

#### Sharing state

The key is to use the *same instance* of the shared code because it's the *instance* that holds the global context for the state.

In the example below, the state is provided by the `host`. 

It could be extracted in a shared module, in case multiple applications need to provide it.

#### Note: other pattern
The official [module-federation-examples](https://github.com/module-federation/module-federation-examples/tree/master/shared-context) use a different pattern:

State is stored in a shared module, and accessed by other components via `shared` libraries, with a `requiredVersion`:
```javascript
// packages/host/webpack.config.js
new ModuleFederationPlugin({
  name: 'app1',
  remotes: {
    app2: 'app2@http://localhost:3002/remoteEntry.js',
  },
  shared: [
    'react',
    'react-dom',
    {
      '@shared-context/shared-library': { //
        import: '@shared-context/shared-library',
        requiredVersion: require('../shared-library/package.json').version,
      },
    },
  ],
}),
new HtmlWebpackPlugin({
  template: './public/index.html',
}),
```


#### Host

```javascript
// packages/host/src/store.js
import React, { createContext, useContext, useState } from "react";

const CountContext = createContext(0);

export const CountProvider = ({ children }) => (
  <CountContext.Provider value={useState(0)}>
  {children}
  </CountContext.Provider>
);

export const useCount = () => useContext(CountContext);
```

Then, `host` adds itself as a remote. This allows both `host` and `remote` to use the same instance of the `store` module, since it is imported via `remotes`.

```javascript
// packages/host/webpack.config.js
new ModuleFederationPlugin({
  name: "host",
  filename: "remoteEntry.js",
  remotes: {
    "my-nav": "nav@http://localhost:3001/remoteEntry.js",
    host: "host@http://localhost:3000/remoteEntry.js",
  },
  exposes: {
    "./store": "./src/store",
  },
  shared: { ... },
}),
```

Import store via "local" remote

```javascript
// packages/host/src/App.jsx
import React, { useState } from "react";
import ReactDOM from "react-dom";

import Header from "my-nav/Header";
import { CountProvider, useCount } from "host/store"; 
const App = () => {
  const [itemCount, setItemCount] = useCount();
  const onAddToCart = () => {
    setItemCount(itemCount + 1);
  };
  return (
    <div>
      <Header />
      <button onClick={onAddToCart}
      >
        Buy me!
      </button>
      <div>Cart count is {itemCount}</div>
    </div>
  );
};
ReactDOM.render(
  <CountProvider>
    <App />
  </CountProvider>,
  document.getElementById("app")
);
```


#### Remote

Add remote to host:
```javascript
new ModuleFederationPlugin({
  name: "nav",
  filename: "remoteEntry.js",
  remotes: {
    host: "host@http://localhost:3000/remoteEntry.js", //
  },
  exposes: {
    "./Header": "./src/Header",
  },
  shared: { ... },
}),
```

Import useCount via host's remote:
```javascript
// packages/nav/src/Header.jsx
import React from "react";
import { useCount } from "host/store"; //

const Header = () => {
  const [count, setCount] = useCount();
  return (
    <header>
      <div>Header - Cart count is {count}</div>
      <button onClick={() => setCount(0)}>Clear</button>
    </header>
  );
};
export default Header;
```

Wrap App in the CountProvider
```javascript
// packages/nav/src/App.jsx

import React from "react";
import ReactDOM from "react-dom";

import Header from "./Header";

import { CountProvider } from "host/store"; //

const App = () => (
  <CountProvider>
    <div>
      <Header />
      <div>Nav project</div>
    </div>
  </CountProvider>
);
ReactDOM.render(<App />, document.getElementById("app"));
```


## 1.8 Resilient function and static data sharing

MF allows to share anything:

- Primitives
- Arrays
- Objects
- Functions
- Classes

It opens the door for A/B testing, feature flags, i18n strings, persist state management (local storate), etc

Ex shared library with misc types:
```javascript
new ModuleFederationPlugin({
  name: "logic",
  filename: "remoteEntry.js",
  remotes: {},
  exposes: {
    "./analyticsFunc": "./src/analyticsFunc",
    "./arrayValue": "./src/arrayValue",
    "./classExport": "./src/classExport",
    "./objectValue": "./src/objectValue",
    "./singleValue": "./src/singleValue",
  },
  shared: {},
}),
```

## Primitives, Arrays, Objects

```javascript
export default "single value";
```

Usage within a module: async import, cached in local state, with Error case handled.s

```javascript
// nav/src/singleValue.js
const SingleValue = () => {
  const [singleValue, singleValueSet] = React.useState(null);
  React.useEffect(() => {
    import("logic/singleValue")
      .then(({ default: value }) => singleValueSet(value))
      .catch((err) => console.error(`Error getting single value: ${err}`));
  }, []);
  return <div>Single value: {singleValue}.</div>;
};
```

## Function

```javascript
// nav/src/analyticsFunc.js
export default (msg) => console.log(`Analytics msg: ${msg}`);
```

```javascript
const analyticsFunc = import("logic/analyticsFunc");
const sendAnalytics = (msg) => {
  analyticsFunc
    .then(({ default: analyticsFunc }) => analyticsFunc(msg))
    .catch((err) => console.log(`Error sending analytics value: ${msg}`));
};
```

Promise will resolve immediately if it has already been resolved.

Same, with a generic high order function:
```javascript
const createAsyncFunc = (promise) => (...args) =>
  promise
    .then(({ default: func }) => func(...args))
    .catch((err) =>
      console.log(`Error sending value: ${JSON.stringify(args)}`)
  );
const sendAnalytics = createAsyncFunc(import("logic/analyticsFunc"));
```

## Class

```javascript
class MyClass {
  constructor(value) {
   this.value = value;
  }
  logString() {
    console.log(`Logging ${this.value}`);
  }
}

export default MyClass;
```

Usage
```javascript
const newClassObject = (...args) =>
  import("logic/classExport")
    .then(({ default: classRef }) => {
      return new classRef(...args);
    })
    .catch((err) => console.log(`Error getting class: ${err}`));

newClassObject("initial value").then((theObject) => {
  theObject.logString();
});
```


## 1.9 Dynamic loading of federated modules

In that case, there is no fixed remote involved. Instead, the remoteEntry is dynamically loaded at runtime, then the code is run.

### Remote

Exposes a module "./Widget" under the scope "widget", with React as singleton.

```javascript
    new ModuleFederationPlugin({
      name: "widget",
      filename: "remoteEntry.js",
      remotes: {},
      exposes: {
        "./Widget": "./src/Widget",
      },
      shared: {
        ...deps,
        react: {
          singleton: true,
          requiredVersion: deps.react,
        },
        "react-dom": {
          singleton: true,
          requiredVersion: deps["react-dom"],
        },
      },
    }),
```


### Host

No fixed remote
```javascript
  plugins: [
    new ModuleFederationPlugin({
      name: "host_wp5",
      filename: "remoteEntry.js",
      remotes: {},
      exposes: {},
      shared: {
        ...deps,
        react: {
          singleton: true,
          requiredVersion: deps.react,
        },
        "react-dom": {
          singleton: true,
          requiredVersion: deps["react-dom"],
        },
      },
    }),
```

App.jsx
```javascript
const App = () => (
  <div>
    <div>Host page.</div>
    <System
      system={{
        url: "http://localhost:8080/remoteEntry.js",
        scope: "widget",
        module: "./Widget",
      }}
    />
  </div>
);
```

Dynamic loading code:

>This function is doing what import does. It’s using the scope (widget) and module
>(./Widget) to get the module factory from the globally mounted object. Then it invokes
>that factory and returns the module. This is just what React.lazy wants to see.

>The reason that react is called out in particular is that React is a singleton that
>requires that there is only a single instance on the page at a time. So this code initializes
>the scope with the host pages React library instance so that the widget module avoids
>loading its own version.


```javascript
function loadComponent(scope, module) {
  return async () => {
    await __webpack_init_sharing__("default");
    const container = window[scope];
    await container.init(__webpack_share_scopes__.default);
    const factory = await window[scope].get(module);
    const Module = factory();
    return Module;
  };
}
```


```javascript
const useDynamicScript = (args) => {
  const [ready, setReady] = React.useState(false);
  const [failed, setFailed] = React.useState(false);

  React.useEffect(() => {
    if (!args.url) {
      return;
    }

    const element = document.createElement("script");

    element.src = args.url;
    element.type = "text/javascript";
    element.async = true;

    setReady(false);
    setFailed(false);

    element.onload = () => {
      console.log(`Dynamic Script Loaded: ${args.url}`);
      setReady(true);
    };

    element.onerror = () => {
      console.error(`Dynamic Script Error: ${args.url}`);
      setReady(false);
      setFailed(true);
    };

    document.head.appendChild(element);

    return () => {
      console.log(`Dynamic Script Removed: ${args.url}`);
      document.head.removeChild(element);
    };
  }, [args.url]);

  return {
    ready,
    failed,
  };
};

function System(props) {
  const { ready, failed } = useDynamicScript({
    url: props.system && props.system.url,
  });

  if (!props.system) {
    return <h2>Not system specified</h2>;
  }

  if (!ready) {
    return <h2>Loading dynamic script: {props.system.url}</h2>;
  }

  if (failed) {
    return <h2>Failed to load dynamic script: {props.system.url}</h2>;
  }

  const Component = React.lazy(
    loadComponent(props.system.scope, props.system.module)
  );

  return (
    <React.Suspense fallback="Loading System">
      <Component />
    </React.Suspense>
  );
}
```
