# GTMSportswear JavaScript/TypeScript Coding Standards

*GTMSportswear's approach to JavaScript and TypeScript*

## Table of Contents

1. [Why TypeScript?](#why-typescript)
1. [TypeScript Compilation](#typescript-compilation)
1. [TypeScript Entry Points](https://github.com/GTMSportswear/docs/blob/master/codingstandards/JS/typescriptentrypoints.md)
1. [TypeScript Modules](#typescript-modules)
1. TypeScript Api Services
1. TypeScript Utilities
  1. Solution Utilities
  1. GitHub Utilities
1. Asynchronous TypeScript
1. Unit Testing
1. Integration Testing

## Why TypeScript?
A few years ago, we made a team decision to move away from pure JavaScript and instead use TypeScript. The main driver behind this was to allow us to add stricter type checking to our code and give us better intellisense during the coding process.

You can find out more at https://www.typescriptlang.org/

## TypeScript Compilation
Since browsers don't natively recognize TypeScript, we have to use a set of tools to transpile our TypeScript code into JavaScript that can be run in a browser. Many tools are available to do this, each with its own set of tradeoffs.

To actually do the transpiling into JavaScript, we use TypeScript's own compiler. We always keep an official version number in `package.json` and this version is what is used during the build process.

To install on a dev machine, simply run `npm install -g typescript`.

Since we have our TypeScript compiler set to transpile down to ES2015 version of JavaScript, we still need another transpiler to go from ES2015 to the more legacy ES5 JavaScript version that is mostly supported back to IE11. To do this we use [Traceur](https://github.com/google/traceur-compiler).

You'd think we'd be done at this point, but not quite yet. We still need a way to actually load up all the TypeScript modules we're creating and resolve dependencies between them. We have chosen to go with an approach ([SystemJS](https://github.com/systemjs/systemjs)) that keeps preserves our modules as individual files instead of loading them all into one big file. This makes debugging in PRD actually possible.

Read on to the next section to see how to actually load a TypeScript file using SystemJS.

## TypeScript Entry Points
See [TypeScript Entry Points](typescriptentrypoints.md)

## TypeScript Modules
As with all companies, we strive to maintain a code base that is clean and easy to change. With that in mind, we try to break all FE code into individual modules. TypeScript gives us several ways to break code into modules using _individual files_, _the `module` keyword_, or _the `class` keyword_.

A simple TypeScript module file begins with some `import { ... } from '..'` statements. Then you will find a line that looks like this:

```ts
export class SomeClass {
  // Declare class info here.

  // Most general purpose display classes have an Output() method.
  public Output(): Promise<Element> {
    // This is an example of a method that returns an HTML element to its consumer by way of a Promise.
  }
}
```

### Method and Property Naming Convention
- We use PascalCase for all method names inside of a class, whether public or private.
  - e.g. `public Output(): void`
  - e.g. `private CalculateSalesTax(): number`
- We use PascalCase for all public properties.
  - e.g. `public WasClicked: boolean`
- We use camelCase for all private properties as well as local variables.
  - e.g. `private container: Element`
  - e.g. `const numClicks = 0`
