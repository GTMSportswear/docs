# TypeScript Entry Points

For every HTML page we reference a single TypeScript file as a way to create an entry point for JS to reference modules, utilities, etc. This entry point is at the top level of our TypeScript folder with a Lisp naming convention that is based off the HTML page requesting it.

`DefaultPage.cshtml` HTML page
`default-page.ts` TypeScript entry point

HTML pages use [SystemJS](https://github.com/systemjs/systemjs) module loader to import the TypeScript entry point at the bottom of the page. These SystemJS imports will have a single line with a `catch()` method for displaying any errors in the console.

`System.import('default-page').catch(console.error)`

An entry point will load any TypeScript partials, modules, utilities, etc. An entry point should not be responsible for business logic or DOM manipulation. Entry points allow for loading of TypeScript modules and dependency injection for testing purposes.

```
import { DefaultPartial } from './default/default-partial';
import { DefaultModule } from './default/default';
import { ApiService } from './services/api-service';

const defaultPartial = new DefaultPartial(new DefaultModule(), new ApiService());

defaultPartial.Output(document.querySelector('.css-class')).catch(console.error);
```