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

Produces an error message. Notice build output in `berry/packages/main/dist` is different from `yarnv1/packages/main/dist`.

## Notes

This bug seems to happen when

- dependency package's entry primary typings file re-exports type definitions from other files of that package.
- dependency package's `tsconfig.json` is having "out" directory pointing to the same place as "source" directory.
