# Prettier

[Prettier](https://prettier.io/) is a code formatter that can automatically format a variety of file types. When used in combination with a linter, Prettier becomes a powerful tool for managing formatting while allowing the linter to focus on code quality.

## Installation

Prettier should be added to each project that uses it as a dev dependency. We prefer to install dependencies within each project instead of globally, because every app will have unique needs, setting up that app for the first time will install all needed dependencies, the formatter can be run in CI or through other automations, and it allows better control over versioning and integration with other packages.

Install it by running:

```bash
yarn add prettier --dev
```

## Setup

Please see the [Prettier documentation](https://prettier.io/docs/en/configuration.html) to configure it for the project's needs. Here you'll find a sample `.prettier.json` (the config file) and `.prettierignore` (for preventing files and folders from being formatted).

To use the demo config, also install these packages:

```bash
yarn add eslint-config-prettier --dev
```

That config will disable any formatting rules in ESLint that conflict with Prettier. After installing `eslint-config-prettier`, update the ESLint config to use the new Prettier config. This example show how to update our [recommended ESLint config](https://github.com/Andrews-McMeel-Universal/amu-code_standards/tree/production/javascript/es6/linters):

```diff
# .eslintrc.json
{
   "plugins": ["babel"],
+  "extends": ["prettier"]
}
```
