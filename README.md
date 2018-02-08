# vue-chrome-tabs

[![npm](https://img.shields.io/npm/v/vue-chrome-tabs.svg) ![npm](https://img.shields.io/npm/dm/vue-chrome-tabs.svg)](https://www.npmjs.com/package/vue-chrome-tabs)
[![vue2](https://img.shields.io/badge/vue-2.x-brightgreen.svg)](https://vuejs.org/)

A Vue component for Chrome-like tabs (based on [chrome-tabs library](https://github.com/adamschwartz/chrome-tabs)).

[Demo](https://denull.github.io/vue-chrome-tabs/demo.html).

## Table of contents

- [Installation](#installation)
- [Usage](#usage)
- [Example](#example)

# Installation

```
npm install --save vue-chrome-tabs
```

## Default import

Install all the components:

```javascript
import Vue from 'vue'
import VueChromeTabs from 'vue-chrome-tabs'

Vue.use(VueChromeTabs)
```

Use specific components:

```javascript
import Vue from 'vue'
import { ChromeTabs } from 'vue-chrome-tabs'

Vue.component('chrome-tabs', ChromeTabs)
```

**⚠️ A css file is included when importing the package. You may have to setup your bundler to embed the css in your page.**

## Distribution import

Install all the components:

```javascript
import 'vue-chrome-tabs/dist/vue-chrome-tabs.css'
import VueChromeTabs from 'vue-chrome-tabs/dist/vue-chrome-tabs.common'

Vue.use(VueChromeTabs)
```

Use specific components:

```javascript
import 'vue-chrome-tabs/dist/vue-chrome-tabs.css'
import { ChromeTabs } from 'vue-chrome-tabs/dist/vue-chrome-tabs.common'

Vue.component('chrome-tabs', ChromeTabs)
```

**⚠️ You may have to setup your bundler to embed the css file in your page.**

## Browser

```html
<link rel="stylesheet" href="vue-chrome-tabs/dist/vue-chrome-tabs.css"/>

<script src="vue.js"></script>
<script src="vue-chrome-tabs/dist/vue-chrome-tabs.browser.js"></script>
```

The plugin should be auto-installed. If not, you can install it manually with the instructions below.

Install all the components:

```javascript
Vue.use(VueChromeTabs)
```

Use specific components:

```javascript
Vue.component('chrome-tabs', VueChromeTabs.ChromeTabs)
```

## Source import

Install all the components:

```javascript
import Vue from 'vue'
import VueChromeTabs from 'vue-chrome-tabs/src'

Vue.use(VueChromeTabs)
```

Use specific components:

```javascript
import Vue from 'vue'
import { ChromeTabs } from 'vue-chrome-tabs/src'

Vue.component('chrome-tabs', ChromeTabs)
```

**⚠️ You need to configure your bundler to compile `.vue` files.** More info [in the official documentation](https://vuejs.org/v2/guide/single-file-components.html).

# Usage

> TODO

# Example

> TODO

---

# Plugin Development

## Installation

The first time you create or clone your plugin, you need to install the default dependencies:

```
npm install
```

## Watch and compile

This will run webpack in watching mode and output the compiled files in the `dist` folder.

```
npm run dev
```

## Use it in another project

While developping, you can follow the install instructions of your plugin and link it into the project that uses it.

In the plugin folder:

```
npm link
```

In the other project folder:

```
npm link vue-chrome-tabs
```

This will install it in the dependencies as a symlink, so that it gets any modifications made to the plugin.

## Publish to npm

You may have to login to npm before, with `npm adduser`. The plugin will be built in production mode before getting published on npm.

```
npm publish
```

## Manual build

This will build the plugin into the `dist` folder in production mode.

```
npm run build
```

---

## License

[MIT](http://opensource.org/licenses/MIT)
