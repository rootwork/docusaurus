---
id: using-themes
title: Themes
---

Like plugins, themes are designed to add functionality to your Docusaurus site. As a good rule of thumb, themes are mostly focused on client-side, where plugins are more focused on server-side functionalities. Themes are also designed to be replace-able with other themes.

## Available themes {#available-themes}

We maintain a [list of official themes](./api/themes/overview.md).

## Using themes {#using-themes}

To use themes, specify the themes in your `docusaurus.config.js`. You may use multiple themes:

```js {3} title="docusaurus.config.js"
module.exports = {
  // ...
  themes: ['@docusaurus/theme-classic', '@docusaurus/theme-live-codeblock'],
};
```

## Theme components {#theme-components}

Most of the time, theme is used to provide a set of React components, e.g. `Navbar`, `Layout`, `Footer`.

Users can use these components in their code by importing them using the `@theme` webpack alias:

```js
import Navbar from '@theme/Navbar';
```

The alias `@theme` can refer to a few directories, in the following priority:

1. A user's `website/src/theme` directory, which is a special directory that has the higher precedence.
1. A Docusaurus theme packages's `theme` directory.
1. Fallback components provided by Docusaurus core (usually not needed).

Given the following structure

```
website
├── node_modules
│   └── docusaurus-theme
│       └── theme
│           └── Navbar.js
└── src
    └── theme
        └── Navbar.js
```

`website/src/theme/Navbar.js` takes precedence whenever `@theme/Navbar` is imported. This behavior is called component swizzling. In iOS, method swizzling is the process of changing the implementation of an existing selector (method). In the context of a website, component swizzling means providing an alternative component that takes precedence over the component provided by the theme.

**Themes are for providing UI components to present the content.** Most content plugins need to be paired with a theme in order to be actually useful. The UI is a separate layer from the data schema, so it makes it easy to swap out the themes for other designs.

For example, a Docusaurus blog consists of a blog plugin and a blog theme.

```js title="docusaurus.config.js"
{
  theme: ['theme-blog'],
  plugins: ['plugin-content-blog'],
}
```

And if you want to use Bootstrap styling, you can swap out the theme with `theme-blog-bootstrap` (fictitious non-existing theme):

```js title="docusaurus.config.js"
{
  theme: ['theme-blog-bootstrap'],
  plugins: ['plugin-content-blog'],
}
```

## Wrapping your site with `<Root>` {#wrapper-your-site-with-root}

A `<Root>` theme component is rendered at the very top of your Docusaurus site.

It allows you to wrap your site with additional logic, by creating a file at `src/theme/Root.js`:

```js title="website/src/theme/Root.js"
import React from 'react';

// Default implementation, that you can customize
function Root({children}) {
  return <>{children}</>;
}

export default Root;
```

This component is applied above the router and the theme `<Layout>`, and will **never unmount**.

:::tip

Use this component to render React Context providers and global stateful logic.

:::

## Swizzling theme components {#swizzling-theme-components}

```mdx-code-block
import SwizzleWarning from "./_partials/swizzleWarning.mdx"

<SwizzleWarning/>
```

Docusaurus Themes' components are designed to be replaceable. To make it easier for you, we created a command for you to replace theme components called `swizzle`.

To swizzle a component for a theme, run the following command in your doc site:

```bash npm2yarn
npm run swizzle <theme name> [component name]
```

As an example, to swizzle the `<Footer />` component in `@docusaurus/theme-classic` for your site, run:

```bash npm2yarn
npm run swizzle @docusaurus/theme-classic Footer
```

This will copy the current `<Footer />` component used by the theme to a `src/theme/Footer` directory under the root of your site, which is where Docusaurus will look for swizzled components. Docusaurus will then use swizzled component in place of the original one from the theme.

Although we highly discourage swizzling of all components, if you wish to do that, run:

```bash npm2yarn
npm run swizzle @docusaurus/theme-classic
```

**Note**: You need to restart your webpack dev server in order for Docusaurus to know about the new component.

## Wrapping theme components {#wrapping-theme-components}

Sometimes, you just want to wrap an existing theme component with additional logic, and it can be a pain to have to maintain an almost duplicate copy of the original theme component.

In such case, you should swizzle the component you want to wrap, but import the original theme component in your customized version to wrap it.

### For site owners {#for-site-owners}

The `@theme-original` alias allows you to import the original theme component.

Here is an example to display some text just above the footer, with minimal code duplication.

```js title="src/theme/Footer.js"
// Note: importing from "@theme/Footer" would fail due to the file importing itself
import OriginalFooter from '@theme-original/Footer';
import React from 'react';

export default function Footer(props) {
  return (
    <>
      <div>Before footer</div>
      <OriginalFooter {...props} />
    </>
  );
}
```

### For plugin authors {#for-plugin-authors}

One theme can wrap a component from another theme, by importing the component from the initial theme, using the `@theme-init` import.

Here's an example of using this feature to enhance the default theme `CodeBlock` component with a `react-live` playground feature.

```js
import InitialCodeBlock from '@theme-init/CodeBlock';
import React from 'react';

export default function CodeBlock(props) {
  return props.live ? (
    <ReactLivePlayground {...props} />
  ) : (
    <InitialCodeBlock {...props} />
  );
}
```

Check the code of `docusaurus-theme-live-codeblock` for details.

:::caution

Unless you want publish to npm a "theme enhancer" (like `docusaurus-theme-live-codeblock`), you likely don't need `@theme-init`.

:::

## Themes design {#themes-design}

While themes share the exact same lifecycle methods with plugins, their implementations can look very different from those of plugins based on themes' designed objectives.

Themes are designed to complete the build of your Docusaurus site and supply the components used by your site, plugins, and the themes themselves. So a typical theme implementation would look like a `src/index.js` file that hooks it up to the lifecycle methods. Most likely they would not use `loadContent`, which plugins would use. And it is typically accompanied by a `src/theme` directory full of components.

To summarize:

- Themes share the same lifecycle methods with Plugins
- Themes are run after all existing Plugins
- Themes exist to add component aliases by extending the webpack config

## Writing customized Docusaurus themes {#writing-customized-docusaurus-themes}

A Docusaurus theme normally includes an `index.js` file where you hook up to the lifecycle methods, alongside with a `theme/` directory of components. A typical Docusaurus `theme` folder looks like this:

```shell {5-7}
website
├── package.json
└── src
    ├── index.js
    └── theme
        ├── MyThemeComponent
        └── AnotherThemeComponent.js
```

There are two lifecycle methods that are essential to theme implementation:

- [`getThemePath()`](lifecycle-apis.md#getthemepath)
- [`getClientModules()`](lifecycle-apis.md#getclientmodules)

These lifecycle methods are not essential but recommended:

- [`validateThemeConfig({themeConfig, validate})`](lifecycle-apis.md#validatethemeconfigthemeconfig-validate)
- [`validateOptions({options, validate})`](lifecycle-apis.md#validateoptionsoptions-validate)

<!--

Outline
---
High-level overview about themes:
- how to use a theme
- how to pass theme configurations
- how to swizzle components and the power of it

Related pieces
---

- [Advanced Guides – Themes](using-themes.md)
- [Lifecycle APIs](lifecycle-apis.md)

References
---
- [themes RFC](https://github.com/facebook/docusaurus/issues/1438)
- [how classic template uses themes](/packages/docusaurus/templates/classic/docusaurus.config.js)
- [using plugins doc](using-plugins.md)
- [vuepress docs on themes](https://v1.vuepress.vuejs.org/theme/)

-->
