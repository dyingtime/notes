## Starting the TypeScript Project

  ```shell
  # create a directory for your project and change into it
  mkdir typescript-project && cd typescript-project
  # install typescript
  install typescript --save-dev
  # initialize your project by creating a tsconfig.json file in your project directory
  npx tsc --init
  ```

`npx` is a tool which will run executable packages include in `npm`. `npx` allows us to to run packages without having to install them global.

`tsc` is the build-in typescript compiler, running `tsc` will transform or compile your code into javascript.

## Compiling the TypeScript Project

```shell
# compile a typescrpt file
npx tsc index.ts
# put the compiler in watch mode, file will be recompiled every time changes are made
npx tsc -w
```

