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

```
Make no mistake; you will run into issues implementing Module Federation.
Having strong debugging skills and debugging complex issues is a prerequisite for
using federated modules successfully. This is not a core stability issue with Module
Federation itself, which is battle tested, these are issues of library compatibility,
deployment issues and more that are layered on top of Module Federation.
```

## 1.2 Micro-frontends (MFE) and module federation (MF)

## Vocabulary
Concepts:
- `remotes`: the name of other federated module application(s) that the current application consumes from
- `exposes`: the files the current application exposes as remotes to other applications
- `shared`: a list of ilbraries shared with other applications, to support the entries in `exposes`.

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