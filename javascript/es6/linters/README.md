# ESLint

[ESLint](https://eslint.org/) is a Javascript static code analyzer that is customizable and compatible with most code editors.

## Installation

ESLint should be added to each project that uses it as a dev dependency. We prefer to install dependencies within each project instead of globally, because every app will have unique needs, setting up that app for the first time will install all needed dependencies, the linter may be run in CI or through other automations, and it allows better control over versions and integration with other packages.

Install it by running:

```bash
yarn add eslint --dev
```

## Setup

Please see the [ESLint documentation](https://eslint.org/docs/user-guide/configuring) to configure it for the project's needs. Here you'll find a sample `.eslintrc.json` (the config file) and `.eslintignore` (for preventing files and folders from being linted).

To use the demo config, also install these packages:

```bash
yarn add eslint-config-airbnb-base eslint-plugin-import eslint-plugin-babel --dev
```

It extends [eslint-config-airbnb-base](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb-base), which supplies most of the lint rules and helps enforce our adopted coding standards. Aside from noting that ES6 and Babel are in use, the demo config assumes the project also uses Jest and that Javascript needs access to browser global variables (like `window`).
