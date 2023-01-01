<!-- TOC -->

- 1. Advanced application architecture - part 1
  - 1.1. What's a micro frontend?
  - 1.2. Enterprise use cases
  - 1.3. Composed microfrontends
  - 1.4. Implementation options
  - 1.5. Pros & Cons of MFE architecture
    - 1.5.1. Why we use MFE?
    - 1.5.2. Potential pitfalls
    - 1.5.3. Should I use a microfrontend?
  - 1.6. The App shell
  - 1.7. Communication patterns
  - 1.8. State management
  - 1.9. Loading strategies
  - 1.10. What do you control
  - 1.11. Enterprise delivery
  - 1.12. @angular-architects/module-federation-plugin
- 2. Advanced application architecture - part 2

<!-- /TOC -->

# 1. Advanced application architecture - part 1

NG conf 2021 workshop, by Owen Mecham.

Slides: https://github.com/owenmecham/module-federation-2022

Course: https://www.pluralsight.com/courses/ng-conf-2021-session-38-part-1

Repo: https://github.com/arnaudj/workshop-js-ngconf-2021-module-federation-crypto

nb: below, "MFE" stands for "micro-frontend"

## 1.1. What's a micro frontend?

- Loosely coupled and highly cohesive UI components or apps
- Independently deployable
- Source code isolation

Framework version agnostic.

Tech stack agnostic.


## 1.2. Enterprise use cases

e.g: deploy functional areas (features) of applications

- Frontend and APIs deploy together
- Features align with Bounded Context
- An MFE can contain other MFE

## 1.3. Composed microfrontends

Can be useful on legacy applications to add new features (e.g with the strangler pattern)

## 1.4. Implementation options
- iframes: communication between iframes via window.postMessage

- reverse proxy: each route can point to a different app.
  - Ex: /appN -> appN/index.html
  - Issue: each route change is a page refresh (ok if users spend a long time on each route)

- Single SPA, bit.dev, SystemJS, Piral, Open Components, Luigi, Mosaic, Puzzle.js

## 1.5. Pros & Cons of MFE architecture

Keep complexity low: introduce MFE if required to achieve the business goals

Cf Lean startup: we aim to accelerate the "build, measure, learn" loop

Teams topology influences architecture:
https://hennyportman.files.wordpress.com/2020/05/qrc-team-topologies-200525-v1.0-1.pdf


### 1.5.1. Why we use MFE?
- Fewer lines of code in each solution
- Team autonomy of tech stack
- Streamline deployment
- Focused unit testing
- Teams can upgrade frameworks at own pace (but there are caveats)
- Encourages a DevOps mindset (CI/CD pipelines, infrastructure as code)

### 1.5.2. Potential pitfalls
- Framework updates: fragmentation
  - Option: have a mono repo that is providing one version of angular to microfrontends that live there as individual projects.
- Bundle size: packaging the framework in the bundle gives framework independence/isolation, at the cost of performance (bandwidth, etc)
- Learning curve/complexity

### 1.5.3. Should I use a microfrontend?

Decision flow: https://www.angulararchitects.io/aktuelles/a-software-architects-approach-towards/

https://www.buildingmicrofrontends.com/

Essentially, don't use MFE if you don't need:
- independent builds and deployments
- source code isolation
- mixed tech stacks

Monorepos considerations:
- source code isolation: can be achieved via git submodules. But harder to do dependency checks
- allow incremental builds


Decision driven mostly by company culture and teams topology

## 1.6. The App shell
Is in charge of:
- Authentication: 
  - interceptors and auth functions should be bundled in an NPM package / shared library
  - refreshing the JWT token, or providing MFE a service to do so (avoid N queries in //)

- Navigation: routing strategy can impact the architecture
  - Defer the router responsibility to the app shell. This avoid each MFE having its own router with internal state that is possibly out of sync.

- Notifications and error handling: for UI consistency. App shell exposes a service.

- Telemetry

- Locale selection: handled by the app shell, passed to each MFE as input

Should be:
- Event and command based
- Light weight and MFE to be lazy loaded


## 1.7. Communication patterns

The DOM is the API
- CustomEvents and Event Listeners
- Can use libs like Eev
- Import: have an unidirectional communication flow:
  - e.g Redux pattern
  - avoid coupling between components

## 1.8. State management

- Router is the source of truth
  - since user can change URL, the preferred approach is: derive state from the URL, rather than having internal state updating the URL
- For Web Component based MFEs, state management should be as light as possible. 
  - Akita, or RxJS behavior subjects
  - If 2 MFEs have too much state management consideration, maybe the boundary is not right and they should be part of the same app.

- Synchronizing state between MFE and shell is a smell
  - Inputs for an MFE (html attributes): the more there are, the more opportunities for errors go up.
  - Recommended: broadcast events and let other MFEs handle them
  - Could indicate that boundaries for the MFE are too narrow
  - Align MFE design w/ Domain Boundaries

- module federation makes this easier w/ shared library singletons

- If shared state store is needed: consider using a facade (e.g, NgRx)

## 1.9. Loading strategies

- Optimize for immediate change: reload script on each invocation (e.g via short lived browser cache)
- Optimize for performance: load once until app re-bootstraps
- Hybrid approach: mix of the above, cf Angular-specific libraries

## 1.10. What do you control
- DRY: auth, interceptors
- Multiple builds depending on consumer:
  - e.g for styling: it's possible to produce 2 bundles: one with styling inherited from parent, another one with embedded styles.
- Localization: keep it dynamic

## 1.11. Enterprise delivery
- Unit tests: to live within each MFE
- End to end testing: 
  - option 1: create a slim app shell for testing during deployment. 
    - Downside: it's not testing the actual production code
    - Plus: removes dependencies toward the real app shell. E.g, if we rely on an UAT app shell, it could be serving the wrong app shell version.
  - option 2: production app shell built as an NPM package that can be shipped inside each MFE
  - nb: apps consuming the MFE should just confirm it loads fine (smoke test)
    - Avoid coupling the production app shell to the MFE

## 1.12. @angular-architects/module-federation-plugin
https://github.com/angular-architects/module-federation-plugin/blob/main/libs/mf/README.md

When using *dynamic module federation*:
>If somehow possible, load the remoteEntry upfront. This allows Module Federation to take the remote's metadata in consideration when negotiating the versions of the shared libraries.
>For this, you could call loadRemoteEntry BEFORE bootstrapping Angular:

```javascript
// main.ts
import { loadRemoteEntry } from '@angular-architects/module-federation';

Promise.all([
  loadRemoteEntry({
    type: 'module',
    remoteEntry: 'http://localhost:3000/remoteEntry.js',
  }),
])
  .catch((err) => console.error('Error loading remote entries', err))
  .then(() => import('./bootstrap'))
  .catch((err) => console.error(err));
```

> The bootstrap.ts file contains the source code normally found in main.ts and hence, it calls platform.bootstrapModule(AppModule). You really need this combination of an upfront file calling loadRemoteEntry and a dynamic import loading another file bootstrapping Angular because Angular itself is already a shared library respected during the version negotiation.

NB: this also opens the possibility to fetch items dynamically from a micro service, and add them to the app router or other. Example from [angular12-microfrontends](https://github.com/module-federation/module-federation-examples/blob/master/angular12-microfrontends/projects/mdmf-shell/src/app/microfrontends/microfrontend.service.ts#L28)

```javascript
  /*
   * This is just an hardcoded list of remote microfrontends, but could easily be updated
   * to load the config from a database or external file
   */
  loadConfig(): Microfrontend[] {
    return [
      {
        // For Loading
        remoteEntry: 'http://localhost:4201/remoteEntry.js',
        remoteName: 'profile',
        exposedModule: 'ProfileModule',

        // For Routing, enabling us to ngFor over the microfrontends and dynamically create links for the routes
        displayName: 'Profile',
        routePath: 'profile',
        ngModuleName: 'ProfileModule',
      },
    ];
  }
```

# 2. Advanced application architecture - part 2
Branches:
- excercise-1: Module Federation with Webpack 5 and Angular
- excercise-1-1: Dynamic Module Federation
- excercise-2: Shared Library (shared state)
- excercise-3: Loading components via Module Federation as plugins