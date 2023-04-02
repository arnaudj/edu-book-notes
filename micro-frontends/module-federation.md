# Module Federation

Misc notes.

For the book, see [Practical Module Federation](practical-module-federation.md)


## CityJS 2023 - Module Federation with Next.JS and SSR
https://www.youtube.com/watch?v=RdiXuoMPtUA

### MF vs UMD externals
When number of external dependencies increase:
- with UMD, each new external is a new single point of failure
- with MF, each new remote is a potential redundancy (given expected version range is matched)

### Static imports
Static imports: good to deal with hooks/middleware, and sometimes necessary (eg, React)

But:
- they require a dynamic import somewhere in the parent tree (ex: bootstrap import)
- they are bundled with parent chunk and increase its size. This gets in the way of code splitting.

Best practice is to use dynamic imports when possible, for performance.

Extra ref: [When it is ok to use static import with federated module #375](https://github.com/module-federation/module-federation-examples/issues/375)

### Delegate modules

Same as remote with syntax `promise new Promise`, except we can use the actual filesystem files directly.

Currently linked to Next.JS SSR, to be extracted.