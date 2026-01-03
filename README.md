# Import Sync

## Status

Champions: Guy Bedford
Stage: 1

## Problem Statement

When modules are already fully loaded into the module registry, it would be useful to support a
synchronous import function to allow loading of dynamic specifier expressions in synchronous function
paths.

## Background

As we enter the maturity stage of native ESM being adopted throughout the JS ecosystem, with features
like `require(esm)` in Node.js unlocking upgrade paths for many, some remaining ergonomic issues
remain on the migration path from CommonJS to ES modules.

In particular CommonJS users in Node.js are used to being able to synchronously dynamically require
modules, without that causing them to have to convert their codepaths to async.

[Defer Import Eval](https://github.com/tc39/proposal-defer-import-eval) solves one of these issues in
allowing synchronous lazy loading of modules. It does this by effectively separating the async aspects
of the module loading pipeline from the sync aspects to allow a [synchronous evaluation function](https://github.com/tc39/proposal-defer-import-eval?tab=readme-ov-file#semantics)
on the deferred namespace.

Even with this proposal supported, there still remains a gap for dynamic lazy loading when the specifier
is not known in advance, since a dynamic `import.defer(specifier)` is still an asynchronous function.

Using the exact same semantics as the synchronous evaluation already defined in the Defer Import Eval
proposal, we can provide an explicit hook for a synchronous import in JavaScript solving this ergonomic
problem for JavaScript developers.

Previously one of the major blockers in the early stages of the ESM specification to enabling a feature like this was the question of asynchronous resolution. _It has since turned out that all JS environments implement
synchronous module resolution._ As a result, and given this constraint, a synchronous import is possible.

## Proposal

We propose to expose an explicit synchronous import function for ES modules, as a direct extension of the synchronous execution behaviour already defined by the [Defer Import Eval][] proposal:

```js
// synchronously import a module if it is available synchronously
const ns = import.sync('./mod.js');
```

The major design of the proposal is a new `Error` which is thrown when a module is not synchronously
available, or uses top-level await.

Whether a module is synchronously available would otherwise be a host-determined property.

## Use Cases

### Getting an Already-Loaded Module

On the web, if a module has already been loaded before, it can always be available synchronously.

In this way `import.sync()` can behave like a registry getter function:

```js
import 'app';

// this will always work if 'app' has been loaded previously
const app = import.sync('app');
```

### Conditional Loading

Just like with dynamic import, with a synchronous import, it's possible to check if a module or builtin is available, but synchronously.

For example, checking if host builtins are available:

```js
let fs;
try {
  fs = import.sync('node:fs');
} catch {}

if (fs) {
  // Use node:fs, only if it is available
}
```

Or a library that conditionally binds to a framework dependency:

```js
let react;
try {
  react = import.sync('react');
} catch {}

if (react) {
  // Bind to the React framework, if available
}
```

### Synchronous Loading of ModuleExpressions & ModuleDeclarations

Importing module expressions without TLA and async dependencies can be supported:

```js
// immediately logs 'hello world'
import.sync(module {
  console.log('hello world');
})
```

Similarly for module declarations:

```js
module dep {
  console.log('hi');
}

module x {
  import dep;
}

// logs 'hi', since both modules are synchronously available
const instance = import.sync(x);
```

## FAQ

### Is Import Sync a module loading phase?

No, unlike `defer` and `source`, `import.sync` is not a phase, it is a new meta property like `import.meta`.

No syntax is supported for `import sync mod from 'mod'`.

To guarantee that a graph is sync upfront, instead see the [Module Sync Assert](https://github.com/tc39/proposal-module-sync-assert) proposal, which may provide an import attribute or otherwise.

_Post an [issue](https://github.com/guybedford/proposal-import-sync/issues)._

[Defer Import Eval]: https://github.com/tc39/proposal-defer-import-eval
