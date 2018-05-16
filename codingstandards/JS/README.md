# GTMSportswear JavaScript/TypeScript Coding Standards

*GTMSportswear's approach to JavaScript and TypeScript*

## Table of Contents

1. [JS/TS Syntax](#jsts-syntax)
1. [Why TypeScript?](#why-typescript)
1. [TypeScript Compilation](#typescript-compilation)
1. [TypeScript Entry Points](https://github.com/GTMSportswear/docs/blob/master/codingstandards/JS/typescriptentrypoints.md)
1. [TypeScript Modules](#typescript-modules)
1. [Retrieving Data](#retrieving-data)
1. Documentation Comments
1. TypeScript Utilities
   1. Solution Utilities
   1. GitHub Utilities
1. Asynchronous TypeScript
1. Unit Testing
1. Integration Testing

## JS/TS Syntax
### Triple Equals (===)
Use Triple Equals (===) and **NOT** Double Equals (==).

Triple Equals uses strict equality. This means that in order to return `true`, values must be the same value **AND** the same type.

## Why TypeScript?
We made a team decision to move away from pure JavaScript and instead use TypeScript. The main driver behind this was to allow us to add stricter type checking to our code and give us better intellisense during the coding process.

You can find out more at https://www.typescriptlang.org/

## TypeScript Compilation
Since browsers don't natively recognize TypeScript, we have to use a set of tools to transpile our TypeScript code into JavaScript that can be run in a browser. Many tools are available to do this, each with its own set of tradeoffs.

To actually do the transpiling into JavaScript, we use TypeScript's own compiler. We always keep an official version number in `package.json` and this version is what is used during the build process.

To install on a dev machine, simply run `npm install -g typescript`.

Since we have our TypeScript compiler set to transpile down to ES2015 version of JavaScript, we still need another transpiler to go from ES2015 to the more legacy ES5 JavaScript version that is mostly supported back to IE11. To do this we use [Traceur](https://github.com/google/traceur-compiler). Note that this is probably not an optimal setup, and we may end up having TypeScript compile directly to ES5 in the future.

You'd think we'd be done at this point, but not quite yet. We still need a way to actually load up all the TypeScript modules we're creating and resolve dependencies between them. We have chosen to go with an approach ([SystemJS](https://github.com/systemjs/systemjs)) that keeps preserves our modules as individual files instead of loading them all into one big file. This makes debugging in PRD actually possible.

Read on to the next section to see how to actually load a TypeScript file using SystemJS.

## TypeScript Entry Points
See [TypeScript Entry Points](typescriptentrypoints.md)

## TypeScript Modules
As with all companies, we strive to maintain a code base that is clean and easy to change. With that in mind, we try to break all FE code into individual modules. TypeScript gives us several ways to break code into modules using _individual files_, _the `module` keyword_ [see docs](https://www.typescriptlang.org/docs/handbook/modules.html), or _the `class` keyword_.

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
- We use camelCase for all private properties as well as local variables and local functions.
  - e.g. `private container: Element`
  - e.g. `const numClicks = 0`
  - e.g. `function countWordAddresses(addresses: Address[]): number`

## Retrieving Data
Every front end application needs some way to retrieve and send data in order for it to be useful.

### Ajax Module
We have our own in-house module that we use to make ajax calls to http endpoints.
[js-xhr](https://github.com/GTMSportswear/js-xhr)

This is already included in all of our major web properties as a JSPM package.

### API Access Points
We currently follow a pattern of putting access to each API endpoint into its own method inside of a class specific to the API being used.

For example, if we were accessing api.gtmsportswear.com, we could create a class called `GtmApiService` that looked like this:
```ts
import { ajax, AjaxRequest } from '../github/js-xhr/js-xhr';

export class GtmApiService {
  public GetSalesTax(total: number, state: string): Promise<number> {
    // Do api call here using the ajax module import.
  }

  public SaveAddress(address: Address): Promise<void> {
    // Do api call here.
  }
}
```

In order to keep all our api access points in a centralized location, we are moving toward a pattern in which every front end repository has a `services` folder in which each api access class is stored as its own file.
