# `oas` TypeScript `moduleResolution` Test

This repo is a test for the changes in https://github.com/readmeio/oas/pull/820.

First install the dependencies:

```sh
npm install
```

# Works as expected with `module`/`moduleResolution` is set to `NodeNext`

Right now, both TS files in this repo are working as expected because of the `"module": "NodeNext"` line in our `tsconfig.json`. You can confirm this yourself by running the following:

```sh
# runs the CommonJS script
npx ts-node index.ts
# runs the ESM script
npx ts-node --esm index.mts
```

You should see a console output that looks like this (which is expected since this means the reducer function is being exported properly and is callable):

```
$ npx ts-node index.ts
reducer: [Function: reducer]
```

# Issues when `moduleResolution` is set to `node`

If you comment out the `module` line in `tsconfig.json`, we're effectively setting `moduleResolution` back to `node`, which is the case that https://github.com/readmeio/oas/pull/820 is hoping to solve for.

You'll note that, without the changes in https://github.com/readmeio/oas/pull/820, both TS files will fail with type errors that look like this:

```
TSError: ⨯ Unable to compile TypeScript:
index.ts:1:21 - error TS2307: Cannot find module 'oas/lib/reducer' or its corresponding type declarations.

1 import reducer from "oas/lib/reducer";
```

If you update `node_modules/oas/package.json` with the changes in https://github.com/readmeio/oas/pull/820, the type errors go away. But now when running `index.ts` (don't even bother with `index.mts` since it's ESM), you'll see another set of unexpected behavior:

```
$ npx ts-node index.ts
reducer: undefined
```

The type errors go away but the function is not found and cannot be invoked properly. In my opinion this is more problematic than the type errors, because TypeScript is yielding a false positive.

I believe this is technically a bug with TypeScript and how it reads from the `typesVersions` object. Might be good to open up an issue in the TypeScript and/or Are The Types Wrong repos.

In the interim, I think we should stick with returning type errors for older TS setups and require that our TypeScript users use modern `moduleResolution` settings, rather than try and address an issue for older TypeScript setups and producing confusing false positive results. I think this is okay to do since we now require Node 18.
