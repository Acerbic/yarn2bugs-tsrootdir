# TS + Yarn2(berry) specific use case tsc bug.

Demonstration of a break in a Typescript compilation when
upgrading to Yarn2.

Summary:

- using Yarn workspaces
- one workspace package is having another as dependency
- specific configuration of the dependency package
- tsc being ran from yarn2 produced different (breaking) output

## Folders in this repo

- `yarnv1` - demo project pre-upgrade to Yarn2, used as a reference to what behaviour is expected.
- `berry` - the same project, but after switching over to Yarn2. No other configuration/source changes applied.

## Reproducing faulty behaviour

Execute:

```bash
cd yarnv1
yarn
yarn build
yarn start
```

Observe console log outputs a dummy value. Now from the root of the project execute:

```bash
cd berry
yarn
yarn build
yarn start
```

Produces an error message.

```
/home/acerbic/Projects/yarn2bugs/tsrootdir/berry/.pnp.js:9350
    throw firstError;
    ^

Error: Qualified path resolution failed - none of the candidates can be found on the disk.

Source path: /home/acerbic/Projects/yarn2bugs/tsrootdir/berry/packages/main/dist/index.js
Rejected candidate: /home/acerbic/Projects/yarn2bugs/tsrootdir/berry/packages/main/dist/index.js
Rejected candidate: /home/acerbic/Projects/yarn2bugs/tsrootdir/berry/packages/main/dist/index.js.js
Rejected candidate: /home/acerbic/Projects/yarn2bugs/tsrootdir/berry/packages/main/dist/index.js.json
Rejected candidate: /home/acerbic/Projects/yarn2bugs/tsrootdir/berry/packages/main/dist/index.js.node

    at Object.makeError (/home/acerbic/Projects/yarn2bugs/tsrootdir/berry/.pnp.js:2609:34)
    at resolveUnqualified (/home/acerbic/Projects/yarn2bugs/tsrootdir/berry/.pnp.js:10044:29)
    at resolveRequest (/home/acerbic/Projects/yarn2bugs/tsrootdir/berry/.pnp.js:10069:14)
    at Object.resolveRequest (/home/acerbic/Projects/yarn2bugs/tsrootdir/berry/.pnp.js:10131:26)
    at Function.module_1.Module._resolveFilename (/home/acerbic/Projects/yarn2bugs/tsrootdir/berry/.pnp.js:9328:34)
    at Function.module_1.Module._load (/home/acerbic/Projects/yarn2bugs/tsrootdir/berry/.pnp.js:9213:40)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:71:12)
    at internal/main/run_main_module.js:17:47
```

Notice build output in `berry/packages/main/dist` is different from `yarnv1/packages/main/dist`.

Contents of `yarnv1/packages/main/dist`:

```
    .
    └── index.js
```

Contents of `berry/packages/main/dist`:

```
    .
    ├── dep
    │   └── src
    │       └── other.js
    └── main
        └── src
            └── index.js
```

## Notes

This bug seems to happen when

- dependency package's entry primary typings file re-exports type definitions from other files of that package.
- dependency package's `tsconfig.json` is having "out" directory pointing to the same place as "source" directory.
