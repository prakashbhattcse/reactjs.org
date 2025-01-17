---
title: 'React v16.2.0: Improved Support for Fragments'
author: [clemmy]
---

React 16.2 is now available! The biggest addition is improved support for returning multiple children from a component's render method. We call this feature _fragments_:

Fragments look like empty JSX tags. They let you group a list of children without adding extra nodes to the DOM:

```js
render() {
  return (
    <>
      <ChildA />
      <ChildB />
      <ChildC />
    </>
  );
}
```

This exciting new feature is made possible by additions to both React and JSX.

## What Are Fragments? {/*what-are-fragments*/}

A common pattern is for a component to return a list of children. Take this example HTML:

```html
Some text.
<h2>A heading</h2>
More text.
<h2>Another heading</h2>
Even more text.
```

Prior to version 16, the only way to achieve this in React was by wrapping the children in an extra element, usually a `div` or `span`:

```js
render() {
  return (
    // Extraneous div element :(
    <div>
      Some text.
      <h2>A heading</h2>
      More text.
      <h2>Another heading</h2>
      Even more text.
    </div>
  );
}
```

To address this limitation, React 16.0 added support for [returning an array of elements from a component's `render` method](/blog/2017/09/26/react-v16.0#new-render-return-types-fragments-and-strings). Instead of wrapping the children in a DOM element, you can put them into an array:

```jsx
render() {
 return [
  "Some text.",
  <h2 key="heading-1">A heading</h2>,
  "More text.",
  <h2 key="heading-2">Another heading</h2>,
  "Even more text."
 ];
}
```

However, this has some confusing differences from normal JSX:

- Children in an array must be separated by commas.
- Children in an array must have a key to prevent React's [key warning](/docs/lists-and-keys#keys).
- Strings must be wrapped in quotes.

To provide a more consistent authoring experience for fragments, React now provides a first-class `Fragment` component that can be used in place of arrays.

```jsx{3,9}
render() {
  return (
    <Fragment>
      Some text.
      <h2>A heading</h2>
      More text.
      <h2>Another heading</h2>
      Even more text.
    </Fragment>
  );
}
```

You can use `<Fragment />` the same way you'd use any other element, without changing the way you write JSX. No commas, no keys, no quotes.

The Fragment component is available on the main React object:

```js
const Fragment = React.Fragment;

<Fragment>
  <ChildA />
  <ChildB />
  <ChildC />
</Fragment>

// This also works
<React.Fragment>
  <ChildA />
  <ChildB />
  <ChildC />
</React.Fragment>
```

## JSX Fragment Syntax {/*jsx-fragment-syntax*/}

Fragments are a common pattern in our codebases at Facebook. We anticipate they'll be widely adopted by other teams, too. To make the authoring experience as convenient as possible, we're adding syntactical support for fragments to JSX:

```jsx{3,9}
render() {
  return (
    <>
      Some text.
      <h2>A heading</h2>
      More text.
      <h2>Another heading</h2>
      Even more text.
    </>
  );
}
```

In React, this desugars to a `<React.Fragment/>` element, as in the example from the previous section. (Non-React frameworks that use JSX may compile to something different.)

Fragment syntax in JSX was inspired by prior art such as the `XMLList() <></>` constructor in [E4X](https://developer.mozilla.org/en-US/docs/Archive/Web/E4X/E4X_for_templating). Using a pair of empty tags is meant to represent the idea it won't add an actual element to the DOM.

### Keyed Fragments {/*keyed-fragments*/}

Note that the `<></>` syntax does not accept attributes, including keys.

If you need a keyed fragment, you can use `<Fragment />` directly. A use case for this is mapping a collection to an array of fragments -- for example, to create a description list:

```jsx
function Glossary(props) {
  return (
    <dl>
      {props.items.map((item) => (
        // Without the `key`, React will fire a key warning
        <Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </Fragment>
      ))}
    </dl>
  );
}
```

`key` is the only attribute that can be passed to `Fragment`. In the future, we may add support for additional attributes, such as event handlers.

### Live Demo {/*live-demo*/}

You can experiment with JSX fragment syntax with this [CodePen](https://codepen.io/reactjs/pen/VrEbjE?editors=1000).

## Support for Fragment Syntax {/*support-for-fragment-syntax*/}

Support for fragment syntax in JSX will vary depending on the tools you use to build your app. Please be patient as the JSX community works to adopt the new syntax. We've been working closely with maintainers of the most popular projects:

### Create React App {/*create-react-app*/}

Experimental support for fragment syntax will be added to Create React App within the next few days. A stable release may take a bit longer as we await adoption by upstream projects.

### Babel {/*babel*/}

Support for JSX fragments is available in [Babel v7.0.0-beta.31](https://github.com/babel/babel/releases/tag/v7.0.0-beta.31) and above! If you are already on Babel 7, simply update to the latest Babel and plugin transform:

```bash
# for yarn users
yarn upgrade @babel/core @babel/plugin-transform-react-jsx
# for npm users
npm update @babel/core @babel/plugin-transform-react-jsx
```

Or if you are using the [react preset](https://www.npmjs.com/package/@babel/preset-react):

```bash
# for yarn users
yarn upgrade @babel/core @babel/preset-react
# for npm users
npm update @babel/core @babel/preset-react
```

Note that Babel 7 is technically still in beta, but a [stable release is coming soon](https://babeljs.io/blog/2017/09/12/planning-for-7.0).

Unfortunately, support for Babel 6.x is not available, and there are currently no plans to backport.

#### Babel with Webpack (babel-loader) {/*babel-with-webpack-babel-loader*/}

If you are using Babel with [Webpack](https://webpack.js.org/), no additional steps are needed because [babel-loader](https://github.com/babel/babel-loader) will use your peer-installed version of Babel.

#### Babel with Other Frameworks {/*babel-with-other-frameworks*/}

If you use JSX with a non-React framework like Inferno or Preact, there is a [pragma option available in babel-plugin-transform-react-jsx](https://github.com/babel/babel/tree/master/packages/babel-plugin-transform-react-jsx#pragmafrag) that configures the Babel compiler to de-sugar the `<></>` syntax to a custom identifier.

### TypeScript {/*typescript*/}

TypeScript has full support for fragment syntax! Please upgrade to [version 2.6.2](https://github.com/Microsoft/TypeScript/releases/tag/v2.6.2). (Note that this is important even if you are already on version 2.6.1, since support was added as patch release in 2.6.2.)

Upgrade to the latest TypeScript with the command:

```bash
# for yarn users
yarn upgrade typescript
# for npm users
npm update typescript
```

### Flow {/*flow*/}

[Flow](https://flow.org/) support for JSX fragments is available starting in [version 0.59](https://github.com/facebook/flow/releases/tag/v0.59.0)! Simply run

```bash
# for yarn users
yarn upgrade flow-bin
# for npm users
npm update flow-bin
```

to update Flow to the latest version.

### Prettier {/*prettier*/}

[Prettier](https://github.com/prettier/prettier) added support for fragments in their [1.9 release](https://prettier.io/blog/2017/12/05/1.9.0#jsx-fragment-syntax-3237-https-githubcom-prettier-prettier-pull-3237-by-duailibe-https-githubcom-duailibe).

### ESLint {/*eslint*/}

JSX Fragments are supported by [ESLint](https://eslint.org/) 3.x when it is used together with [babel-eslint](https://github.com/babel/babel-eslint):

```bash
# for yarn users
yarn add eslint@3.x babel-eslint@7
# for npm users
npm install eslint@3.x babel-eslint@7
```

or if you already have it, then upgrade:

```bash
# for yarn users
yarn upgrade eslint@3.x babel-eslint@7
# for npm users
npm update eslint@3.x babel-eslint@7
```

Ensure you have the following line inside your `.eslintrc`:

```json
"parser": "babel-eslint"
```

That's it!

Note that `babel-eslint` is not officially supported by ESLint. We'll be looking into adding support for fragments to ESLint 4.x itself in the coming weeks (see [issue #9662](https://github.com/eslint/eslint/issues/9662)).

### Editor Support {/*editor-support*/}

It may take a while for fragment syntax to be supported in your text editor. Please be patient as the community works to adopt the latest changes. In the meantime, you may see errors or inconsistent highlighting if your editor does not yet support fragment syntax. Generally, these errors can be safely ignored.

#### TypeScript Editor Support {/*typescript-editor-support*/}

If you're a TypeScript user -- great news! Editor support for JSX fragments is already available in [Visual Studio 2015](https://www.microsoft.com/en-us/download/details.aspx?id=48593), [Visual Studio 2017](https://www.microsoft.com/en-us/download/details.aspx?id=55258), [Visual Studio Code](https://code.visualstudio.com/updates/v1_19#_jsx-fragment-syntax) and [Sublime Text via Package Control](https://packagecontrol.io/packages/TypeScript).

### Other Tools {/*other-tools*/}

For other tools, please check with the corresponding documentation to check if there is support available. However, if you're blocked by your tooling, you can always start with using the `<Fragment>` component and perform a codemod later to replace it with the shorthand syntax when the appropriate support is available.

## Installation {/*installation*/}

React v16.2.0 is available on the npm registry.

To install React 16 with Yarn, run:

```bash
yarn add react@^16.2.0 react-dom@^16.2.0
```

To install React 16 with npm, run:

```bash
npm install --save react@^16.2.0 react-dom@^16.2.0
```

We also provide UMD builds of React via a CDN:

```html
<script
  crossorigin
  src="https://unpkg.com/react@16/umd/react.production.min.js"
></script>
<script
  crossorigin
  src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js"
></script>
```

Refer to the documentation for [detailed installation instructions](/docs/installation).

## Changelog {/*changelog*/}

### React {/*react*/}

- Add `Fragment` as named export to React. ([@clemmy](https://github.com/clemmy) in [#10783](https://github.com/facebook/react/pull/10783))
- Support experimental Call/Return types in `React.Children` utilities. ([@MatteoVH](https://github.com/MatteoVH) in [#11422](https://github.com/facebook/react/pull/11422))

### React DOM {/*react-dom*/}

- Fix radio buttons not getting checked when using multiple lists of radios. ([@landvibe](https://github.com/landvibe) in [#11227](https://github.com/facebook/react/pull/11227))
- Fix radio buttons not receiving the `onChange` event in some cases. ([@jquense](https://github.com/jquense) in [#11028](https://github.com/facebook/react/pull/11028))

### React Test Renderer {/*react-test-renderer*/}

- Fix `setState()` callback firing too early when called from `componentWillMount`. ([@accordeiro](https://github.com/accordeiro) in [#11507](https://github.com/facebook/react/pull/11507))

### React Reconciler {/*react-reconciler*/}

- Expose `react-reconciler/reflection` with utilities useful to custom renderers. ([@rivenhk](https://github.com/rivenhk) in [#11683](https://github.com/facebook/react/pull/11683))

### Internal Changes {/*internal-changes*/}

- Many tests were rewritten against the public API. Big thanks to [everyone who contributed](https://github.com/facebook/react/issues/11299)!

## Acknowledgments {/*acknowledgments*/}

This release was made possible by our open source contributors. A big thanks to everyone who filed issues, contributed to syntax discussions, reviewed pull requests, added support for JSX fragments in third party libraries, and more!

Special thanks to the [TypeScript](https://www.typescriptlang.org/) and [Flow](https://flow.org/) teams, as well as the [Babel](https://babeljs.io/) maintainers, who helped make tooling support for the new syntax go seamlessly.

Thanks to [Gajus Kuizinas](https://github.com/gajus/) and other contributors who prototyped the `Fragment` component in open source.
