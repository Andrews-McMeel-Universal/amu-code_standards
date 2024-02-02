# Browser Support

## Supported Platforms (Confluence QA Page)

Our [list of supported client platforms/browsers](https://amuniversal.atlassian.net/wiki/x/AYBoc) is located in Confluence. This is maintained and updated on a periodic basis.

## Browserslist

We use [Browserslist](https://github.com/browserslist/browserslist) to determine browser support. Many tools will use the `.browserslistrc` config file to automatically provide a customized build that targets the desired browsers. Our config is more permissive in selecting more supported browsers than we officially support, but we do this in an effort to maintain a minumum acceptable experience on those additional browsers.

### Installation

To use Browserslist, add a `.browserslistrc` file to the root of the project with any other config files. We have a sample `.browserslistrc` file available for use.

Useful links:

- [browserslist.dev](https://browserslist.dev/?q=Pj0gMSUgYW5kIGxhc3QgMyBtYWpvciB2ZXJzaW9ucywgbm90IElFIDExLCBub3Qgb3BfbWluaSBhbGwsIG5vdCBkZWFkLCBtYWludGFpbmVkIG5vZGUgdmVyc2lvbnM%3D), an easy to read list of supported browsers (please update the link if you change the .browserslistrc)
- [@babel/preset-env](https://babeljs.io/docs/en/next/babel-preset-env.html), which is already included in our [recommended Babel config](https://github.com/Andrews-McMeel-Universal/amu-code_standards/tree/production/javascript/es6/transpilers)

### Linting for Browser Support

Wherever possible, to check that we're properly supporting these browsers (or at least providing reasonable fallbacks) we lint our [Javascript](https://github.com/amilajack/eslint-plugin-compat) and [SCSS](https://github.com/ismay/stylelint-no-unsupported-browser-features), which will warn if it thinks something is unsupported. If a linter does flag code as unsupported, consider the following options:

1. Find an alternative method or workaround that is supported
2. Provide a graceful fallback
3. Determine if this is a case of progressive enhancement, and is not detrimental to the user experience to be omitted

Whichever option is most appropriate, leave a comment indicating what is happening and why. For example:

```scss
// stylelint-disable plugin/no-unsupported-browser-features
margin-bottom: $grid-gutter-width * 2; // IE calc fallback
margin-bottom: calc(#{$grid-gutter-width * 2} - #{$nav-link-padding-y});
// stylelint-enable plugin/no-unsupported-browser-features
```

### Installing eslint-plugin-compat

```bash
yarn add eslint-plugin-compat
```

Update the ESLint config file:

```
{
  "extends": [
    ...
    "plugin:compat/recommended"
  ],
  "settings": {
    "polyfills": [
      // https://github.com/amilajack/eslint-plugin-compat#adding-polyfills
      // Inform the linter of polyfilled methods and properties, e.g. "Promise"
    ]
  }
}
```

### Installing stylelint-no-unsupported-browser-features

```bash
yarn add stylelint-no-unsupported-browser-features
```

Update the Stylelint config file:

```
{
  "plugins": [
    ...
    "stylelint-no-unsupported-browser-features"
  ],
  "rules": {
    "plugin/no-unsupported-browser-features": [
      true,
      {
        "severity": "warning",
        "ignore": [
          // https://github.com/ismay/stylelint-no-unsupported-browser-features#options
          // Inform the linter of any features in the project that are commonly used and tested manually in the target browsers, e.g. "flexbox"
        ]
      }
    ]
  }
}
```
