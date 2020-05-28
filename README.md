# remark-typedoc-symbol-links

![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/kamranayub/remark-typedoc-symbol-links.svg)
![GitHub repo size](https://img.shields.io/github/repo-size/kamranayub/remark-typedoc-symbol-links.svg)
![npm](https://img.shields.io/npm/dw/remark-typedoc-symbol-links.svg)
![npm](https://img.shields.io/npm/dm/remark-typedoc-symbol-links.svg)
![npm](https://img.shields.io/npm/dy/remark-typedoc-symbol-links.svg)
![npm](https://img.shields.io/npm/dt/remark-typedoc-symbol-links.svg)
![NPM](https://img.shields.io/npm/l/remark-typedoc-symbol-links.svg)
![npm](https://img.shields.io/npm/v/remark-typedoc-symbol-links.svg)
![GitHub last commit](https://img.shields.io/github/last-commit/kamranayub/remark-typedoc-symbol-links.svg)
![npm collaborators](https://img.shields.io/npm/collaborators/remark-typedoc-symbol-links.svg)

A Remark plugin for transforming [TypeDoc](https://typedoc.org) symbol links, such as `[[symbol]]` to a Markdown link, with Rehype compatibility.

## Usage

### With Gatsby.js

This was developed for use by the [excalibur.js](https://excaliburjs.com) project and is in use in the documentation site, see [the Gatsby config](https://github.com/excaliburjs/excaliburjs.github.io/blob/site/gatsby-config.js). This is the underlying package used in [gatsby-remark-typedoc-symbol-links](...) which depends on the [gatsby-source-typedoc](https://github.com/kamranayub/gatsby-source-typedoc) package to generate the required AST in TypeDoc for a TypeScript project.

### With unified

If using as a Remark plugin, see `examples/example.js`. 

Given the folowing Markdown:

```md
## Introduction

Create a new [[Engine]] instance and call [[Engine.start|start]] to start the game!
```

And the following usage with `unified`:

```js
const fs = require('fs')
const unified = require('unified')
const markdown = require('remark-parse')
const html = require('remark-html')
const typedocSymbolLinks = require('../dist')

// Load generated TypeDoc
const typedoc = JSON.parse(fs.readFileSync('../src/__tests__/typedoc.json'))

const doc = unified()
  .use(markdown)
  // Pass typedoc and other options
  .use(typedocSymbolLinks, { typedoc, basePath: '/docs/api' })
  .use(html)
  .processSync(fs.readFileSync('example.md'))
  .toString()

console.log(doc)
```

Node will output:

```html
<h2>Introduction</h2>
<p>Create a new <a href="/docs/apiclasses/_engine_.engine.html" title="View &#x27;Engine&#x27; in API reference docs" class="symbol" target="_blank">Engine</a> instance and call <a href="/docs/apiclasses/_engine_.engine.html#start" title="View &#x27;Engine.start&#x27; in API reference docs" class="symbol" target="_blank">start</a> to start the game!</p>
```

### Handling missing links

When no matching symbol is detected, the anchor link is rendered with a `data-missing` attribute and the title changes to indicate the symbol is missing. This will allow you to add custom CSS styles like a red underline.

## API

### `remark().use(typedocSymbolLinks[, options])`

Transform TypeDoc markdown symbol links to links, with rehype compatibility.

#### `options`

##### `options.typedoc` (required)

The parsed JSON of TypeDoc output (such as running through `typedoc --generateJson`). When used with [gatsby-source-typedoc](https://github.com/kamranayub/gatsby-source-typedoc), this is provided automatically.

##### `options.basePath` (optional, default: `/`)

The path prefix to prepend to all generated links. Typically the path to where your generated TypeDoc documentation lives.

## Compatibility and differences from TypeDoc

This plugin attempts to emulate TypeDoc's [link resolution](http://typedoc.org/guides/link-resolution/) but it's important to point out that the plugin _has no context_ when resolving symbols (meaning, it's a Markdown page outside your source code, so it cannot look hierarchically to resolve links). That means that you may need to fully-qualify methods, properties, and functions if they are not unique.

### Classes, Interfaces, and Enums

`class`, `enum`, and `interface` symbols and their members only need to be qualified by the container name. Use `ClassName#ctor` for linking to the constructor of a class.

#### Examples:

- `[[Engine]]` - Class, interface, or enum name
- `[[Engine#ctor]]` - Class constructor
- `[[Engine.start]]` - Member name

### Module functions

If a function is exported within a module, it can be linked to by name. However, if there are similarly named functions _in different modules_, the first match will be used. This could be fixed through fully-qualified module naming, see [this note](#unsupported--module-index-links).

#### Examples:

- `[[clamp]]` - Exported function

### Unsupported: Module indexes

When generating documentation with modules enabled in TypeDoc, it generates names like `"ModuleName/SubmoduleName"`. Right now, the plugin is limited because it does not allow linking directly to a module index page, since it assumes you typically want to link to a symbol _within_ a module. That is how it can avoid forcing you to always fully-qualify a symbol path with the module name.

Example, this won't work:

```md
See [["ModuleName"]]
```

On the flip-side, you don't need to do this:

```md
See [["ModuleName".myFunction]]
```

Since module symbols are all indexed, you can leave off the module qualifiers:

```md
See [[myFunction]]
```

This is a limitation could be overcome but it needs some thought. For example, maybe to link to a module index, you could do:

```md
See [[module:Module/SubModule]]
```

This could ignore quotes and allow linking to the module index page. If you think you need this, I welcome PRs!

### Simple symbol links

The following Markdown:

    Check out the [[Sword.slash]] source code!

Will be transformed into this HTML:

    Check out the <a href="/classes/_module_.sword.html#slash" target="_blank" class="symbol">Sword.slash</a> source code!

### Aliased symbol links

The following Markdown:

    Check out the [[Sword.slash|slash helper]] source code!

Will be transformed into this HTML:

    Check out the <a href="/classes/_module_.sword.html#slash" target="_blank" class="symbol symbol--aliased">slash helper</a> source code!

## MIT License
