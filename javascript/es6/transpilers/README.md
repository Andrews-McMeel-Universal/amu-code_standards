# Babel

[Babel](https://babeljs.io/) is a Javascript transpiler, meaning it will transform and compile ES6 compliant code into backwards compatible ES5 Javascript. When developing for browsers with limited ES6 support, such as IE11, Babel must be used for that code to run properly.

## Installation

Some dependency managers, like [Parcel](https://en.parceljs.org/javascript.html#babel), or frameworks, like [Next.js](https://nextjs.org/docs/advanced-features/customizing-babel-config), come out of the box with Babel already configured and ready to go.

If Babel needs to be installed, run:

```bash
yarn add babel-loader @babel/core
```

## Setup

Please see the [Babel documentation](https://babeljs.io/docs/en/config-files/) to configure it for the project's needs. Here you'll find a sample `babel.config.json` (the config file).

To use the demo config, also install this package:

```bash
yarn add babel-preset-airbnb
```

It uses [babel-preset-airbnb](https://github.com/airbnb/babel-preset-airbnb) to provide sensible defaults that are compatible with our adopted coding standards.

## Polyfills

While Babel converts ES6 syntax to ES5, it does not include new APIs and functionality that are often used with ES6, such as Promises. Those few features need to be polyfilled, or included, in order to function in incompatible browsers. [Babel recommends a polyfill](https://babeljs.io/docs/en/babel-polyfill) that we typically install at the same time. Here is a [list of features](https://github.com/zloirock/core-js#features) provided by the polyfill, `core-js`.

To install the polyfills:

```bash
yarn add core-js
```

Then import it in the app's main entrypoint, before importing other packages or running code that depends on a polyfill:

```javascript
import "core-js";
```
