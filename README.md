# tsrs false positive TS5115

`tsrs` reports TS5115 on code that `tsc` 6.0.3 and `tsgo` accept without any diagnostic.

We hit it on a real AdonisJS application with the types of [`@adonisjs/http-transformers`](https://www.npmjs.com/package/@adonisjs/http-transformers) 2.3.1. [`repro.ts`](./repro.ts) has no dependencies: it keeps only the parts of that package's types the error needs.

```
repro.ts(76,12): error TS5115: Instantiations of the following types appear infinitely circular: 'InferData', 'UnpackKeyValue'.
```

## Run it

```shell
npm install
npm run check:tsc   # no error
npm run check:tsgo  # no error
npm run check:tsrs  # TS5115
```

| Checker | Result |
|---|---|
| `tsc` 6.0.3 | no error |
| `tsgo` 7.0.0-dev.20260707.2 | no error |
| `tsrs` 0.18.0-ts7.1.0-dev.20260929 | TS5115 |
| `tsrs --checkerAssignment go --noLazyMembers` | TS5115 |

## What triggers it

The error appears only when the `Item` comes back from a closure passed to a generic function:

```ts
declare function once<T>(value: T): T;
once(() => SizeTransformer.transform(new Size())); // TS5115 with tsrs only
once(SizeTransformer.transform(new Size()));       // no error anywhere
```

The same TS5115 message comes up on another repository in [oxc-project/tsrs#373](https://github.com/oxc-project/tsrs/issues/373), which is closed.
