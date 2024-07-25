# Angular181TypecheckLibsError

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 18.1.1.

# Minimal reproduction for issue
This repository is created in order to provide a minimal reproduction for the following error that appeared for Angular library projects after upgrading to Angular v18.1 when building with `ng build`:

```
------------------------------------------------------------------------------
Building entry point 'example/tertiary-entrypoint'
------------------------------------------------------------------------------
✖ Compiling with Angular sources in Ivy partial compilation mode.
dist/example/secondary-entrypoint/index.d.ts:1:22 - error TS6053: File '/Users/davidlj95/Code/git/tmp/angular-18.1-typecheck-libs-error/dist/example/secondary-entrypoint/index.ngtypecheck.d.ts' not found.

1 /// <reference path="index.ngtypecheck.d.ts" />
                       ~~~~~~~~~~~~~~~~~~~~~~

 ELIFECYCLE  Command failed with exit code 1.
```

In order for the error to appear:
 - There is more than 1 secondary entrypoint
 - Angular workspace was created with Angular CLI v17.1.0 or below
 - Typescript hasn't been upgraded yet to 5.5.x

# Steps to reproduce the issue
In order to generate this repository (see commits):

1. Generate new application with using Angular CLI v18.1.0 (though see below)
   ```sh
    pnpm dlx @angular/cli new --package-manager=pnpm \
      --no-create-application angular-18.1-typecheck-libs-error
   ```
1. Add an `example` library to it
   ```sh
   pnpm ng g lib example
   ```
1. Add a [secondary entrypoint as `ng-packagr` docs explain](https://github.com/ng-packagr/ng-packagr/blob/18.1.0/docs/secondary-entrypoints.md). And a third entrypoint that will use the former.
   1. Using `index.ts` instead of `public-api.ts` in order to:
   1. Configure path mappings via `tsconfig.json` for imports
1. Downgrade Typescript to 5.4 (as if you came from a previous Angular version without support for Typescript 5.5)
   ```sh
   pnpm i -D typescript@5.4.x
   ```
1. Remove the [recently added `skipLibCheck` set to `true`](https://github.com/angular/angular-cli/commit/e2f92ab957e797d8085616fcdea0b73344e614af) in `tsconfig.json`
   - There were no migration schematics for that. So existing projects around created before this change was introduced (`17.1.0`) will have that option enabled.

1. Run `ng build`. There you have your error 🎉

# Existing workarounds
## Upgrade to Typescript 5.5.x
If you can. The error goes away 🎉 That's the best way to go

## Add `skipLibCheck: true`

To `tsconfig.json`. This is the [new way to go for projects generated since Angular CLI v17.1](https://github.com/angular/angular-cli/commit/e2f92ab957e797d8085616fcdea0b73344e614af). 

Then, resulting files containing the invalid reference won't be checked hence the build command will succeed

```
/// <reference path="metadata-json-resolver.ngtypecheck.d.ts" />
```

Anyway that's an error, though with that option it's an error that you don't check for

> One may be interested in checking those definition files anyway if using extra tooling to be sure shipped definitions are OK (and they aren't indeed due to the bug). For instance, one could want to verify definitions after rolling them up with [`dts-bundle-generator`](https://github.com/timocov/dts-bundle-generator).

## Development server

Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The application will automatically reload if you change any of the source files.

## Code scaffolding

Run `ng generate component component-name` to generate a new component. You can also use `ng generate directive|pipe|service|class|guard|interface|enum|module`.

## Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory.

## Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via a platform of your choice. To use this command, you need to first add a package that implements end-to-end testing capabilities.

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI Overview and Command Reference](https://angular.dev/tools/cli) page.
