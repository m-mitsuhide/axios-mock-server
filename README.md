<p align="right">
  <a href="https://github.com/solufa/axios-mock-server#readme">🇺🇸English</a> |
  <a href="https://github.com/solufa/axios-mock-server/blob/develop/docs/ja/README.md">🇯🇵日本語</a>
</p>

<h1>axios-mock-server</h1>

[![npm version][badge-npm]][badge-npm-url]
[![npm bundle size][badge-bundlephobia]][badge-bundlephobia-url]
[![CircleCI][badge-ci]][badge-ci-url]
[![Codecov][badge-coverage]][badge-coverage-url]
[![Language grade: JavaScript][badge-lgtm]][badge-lgtm-url]
[![Dependabot Status][badge-dependabot]][dependabot]
[![License][badge-license]][axios-mock-server-license]

RESTful mock server using axios.

<details>
<summary><b>Table of contents</b></summary>

<!--
This table of contents is generated by Markdown All in One in Visual Studio Code.
Please set the `githubCompatibility` option to `true`.
<https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one>
-->

- [Features](#features)
- [Getting Started](#getting-started)
  - [Installation](#installation)
  - [Tutorial](#tutorial)
    - [Create API](#create-api)
    - [Build API](#build-api)
    - [Mocking Axios](#mocking-axios)
  - [Examples](#examples)
- [Usage](#usage)
  - [Create an API endpoint](#create-an-api-endpoint)
  - [Connect to axios](#connect-to-axios)
    - [Default](#default)
    - [Each instance](#each-instance)
  - [Functions](#functions)
    - [`setDelayTime(millisecond: number): void`](#setdelaytimemillisecond-number-void)
    - [`enableLog(): void` and `disableLog(): void`](#enablelog-void-and-disablelog-void)
  - [TypeScript](#typescript)
  - [Cautions](#cautions)
    - [`.gitignore`](#gitignore)
    - [`@ts-ignore`, `eslint-disable`](#ts-ignore-eslint-disable)
- [Troubleshooting](#troubleshooting)
  - [`The expected type comes from property 'get' which is declared here on type 'MockMethods'` error in TypeScript](#the-expected-type-comes-from-property-get-which-is-declared-here-on-type-mockmethods-error-in-typescript)
- [Command Line Interface Options](#command-line-interface-options)
- [Configuration](#configuration)
- [License](#license)

</details>

## Features

- You can create a `GET`/`POST`/`PUT`/`DELETE` API endpoint in a few lines.
- A dedicated server is not required.
- It works with SPA as a static [JavaScript][javascript] file.
- You can use [axios][axios] as mock-up even in [Node.js][nodejs] environment.
- There is an auto-routing function similar to [Nuxt.js][nuxtjs], and no path description is required.
- Supports [TypeScript][typescript].

## Getting Started

### Installation

- Using [npm][npm]:

  ```sh
  $ npm install axios
  $ npm install axios-mock-server --save-dev
  ```

- Using [Yarn][yarn]:

  ```sh
  $ yarn add axios
  $ yarn add axios-mock-server --dev
  ```

### Tutorial

Introducing the simplest use of axios-mock-server.

<details>
<summary><b>Start the tutorial</b></summary>

#### Create API

First, create a `mocks` directory to store the files you want to mock up.

```sh
$ mkdir mocks
```

Next, create an API endpoint file in the `mocks` directory.  
Let's define a mock API that retrieves basic user information with a `GET` request.

Create a `mocks/users` directory and create a `_userId.js` file.

```sh
$ mkdir mocks/users
$ touch mocks/users/_userId.js

# If Windows (Command Prompt)
> mkdir mocks\users
> echo. > mocks\users\_userId.js
```

Add the following to the `mocks/users/_userId.js` file.

<!-- prettier-ignore -->
```js
// file: 'mocks/users/_userId.js'
const users = [{ id: 0, name: 'foo' }, { id: 1, name: 'bar' }]

module.exports = {
  get({ values }) {
    return [200, users.find(user => user.id === values.userId)]
  }
}
```

The routing of axios-mock-server is **automatically generated according to the tree structure of [JavaScript][javascript] and [TypeScript][typescript] files** in the `mocks` directory in the same way as the routing of [Nuxt.js][nuxtjs].

Reference: [Routing - Nuxt.js][nuxtjs-routing]

In other words, the `mocks/users/_userId.js` file **can define an endpoint using dynamic routing** as the path of `/users/:userId`.

#### Build API

axios-mock-server needs to build and generate the necessary files for routing before running.

```sh
$ node_modules/.bin/axios-mock-server

# If Windows (Command Prompt)
> node_modules\.bin\axios-mock-server
```

If the build is successful, the `$mock.js` file is generated in the`mocks` directory.

```sh
$ cat mocks/\$mock.js
/* eslint-disable */
module.exports = (client) => require('axios-mock-server')([
  {
    path: '/users/_userId',
    methods: require('./users/_userId')
  }
], client)

# If Windows (Command Prompt)
> type mocks\$mock.js
```

#### Mocking Axios

Finally, import the `mocks/$mock.js` file generated by `index.js` file etc. and pass it to the argument of axios-mock-server.  
axios-mock-server will mock up all [axios][axios] communications by default.

<!-- prettier-ignore -->
```js
// file: 'index.js'
const axios = require('axios')
const mock = require('./mocks/$mock.js')

mock()

axios.get('https://example.com/users/1').then(({ data }) => {
  console.log(data)
})
```

If you run the `index.js` file, you will see that `{ id: 1, name: 'bar' }` is returned.

```sh
$ node index.js
{ id: 1, name: 'bar' }
```

</details>

### Examples

axios-mock-server can be used to mock up **browser usage**, **data persistence** and **`multipart/form-data` format communication**.  
It is also easy to link with [Nuxt.js][nuxtjs] ([@nuxtjs/axios][nuxtjs-axios]).

See [examples][axios-mock-server-examples] for source code.

<details>
<summary><b>See a list of use cases</b></summary>

- **[browser](https://github.com/solufa/axios-mock-server/tree/develop/examples/browser)**:
  Use in browser
- **[node](https://github.com/solufa/axios-mock-server/tree/develop/examples/node)**:
  Use in [Node.js][nodejs] (CommonJS)
- **[with-nuxtjs](https://github.com/solufa/axios-mock-server/tree/develop/examples/with-nuxtjs)**:
  Using with a [Nuxt.js][nuxtjs]
- **[with-typescript](https://github.com/solufa/axios-mock-server/tree/develop/examples/with-typescript)**:
  Using with a [TypeScript][typescript]

**WIP**

- with-in-memory-database

</details>

## Usage

### Create an API endpoint

<!-- prettier-ignore -->
```js
const users = [{ id: 0, name: 'foo' }, { id: 1, name: 'bar' }]

/**
 * Type definitions for variables passed as arguments in requests
 * @typedef { Object } MockMethodParams
 * @property { import('axios').AxiosRequestConfig } config axios request settings
 * @property {{ [key: string]: string | number }} values Dynamic value of the requested URL (underscore part of the path)
 * @property {{ [key: string]: any }} params The value of the query parameter for the requested URL
 * @property { any } data Request data sent by POST etc.
 */

/**
 * Type definition when response is returned as an object
 * @typedef { Object } MockResponseObject
 * @property { number } status HTTP response status code
 * @property { any? } data Response data
 * @property {{ [key: string]: any }?} headers Response header
 */

/**
 * Response type definition
 * @typedef { [number, any?, { [key: string]: any }?] | MockResponseObject } MockResponse
 */

export default {
  /**
   * All methods such as GET and POST have the same type
   * @param { MockMethodParams }
   * @returns { MockResponse }
   */
  get({ values }) {
    return [200, users.find(user => user.id === values.userId)]
  },

  /**
   * You can also return a response asynchronously
   * @param { MockMethodParams }
   * @returns { Promise<MockResponse> }
   */
  async post({ data }) {
    await new Promise(resolve => setTimeout(resolve, 1000))

    users.push({
      id: users.length,
      name: data.name
    })

    return { status: 201 }
  }
}
```

### Connect to axios

#### Default

axios-mock-server will mock up all [axios][axios] communications by default.

<!-- prettier-ignore -->
```js
import axios from 'axios'
import mock from './mocks/$mock.js'

mock()

axios.get('https://example.com/api/foo').then(response => {
  /* ... */
})
```

#### Each instance

You can also mock up each [axios instance][axios-instance].

<!-- prettier-ignore -->
```js
import axios from 'axios'
import mock from './mocks/$mock.js'

const client = axios.create({ baseURL: 'https://example.com/api' })

mock(client)

client.get('/foo').then(response => {
  /* ... */
})

// axios will not be mocked up
axios.get('https://example.com/api/foo').catch(error => {
  console.log(error.response.status) // 404
})
```

### Functions

axios-mock-server has several built-in functions available.

#### `setDelayTime(millisecond: number): void`

Simulate response delay.

<!-- prettier-ignore -->
```js
import axios from 'axios'
import mock from './mocks/$mock.js'

mock().setDelayTime(500)

console.time()
axios.get('https://example.com/api/foo').then(() => {
  console.timeEnd() // default: 506.590ms
})
```

#### `enableLog(): void` and `disableLog(): void`

Switch request log output.

<!-- prettier-ignore -->
```js
import axios from 'axios'
import mock from './mocks/$mock.js'

const mockServer = mock()

;(async () => {
  // To enable
  mockServer.enableLog()
  await axios.get('/foo', { baseURL: 'https://example.com/api', params: { bar: 'baz' } }) // stdout -> [mock] get: /foo?bar=baz => 200

  // To disable
  mockServer.disableLog()
  await axios.get('/foo', { baseURL: 'https://example.com/api', params: { bar: 'baz' } }) // stdout ->
})()
```

### TypeScript

axios-mock-server includes [TypeScript][typescript] definitions.

### Cautions

#### `.gitignore`

Exclude `$mock.js` or `$mock.ts` generated by axios-mock-server in the build from [Git][git] monitoring.

```sh
$ echo "\$mock.*" >> .gitignore

# If Windows (Command Prompt)
> echo $mock.* >> .gitignore
```

#### `@ts-ignore`, `eslint-disable`

For [TypeScript][typescript] projects, add a `// @ ts-ignore` comment to the above line that imports`$ mock.ts`.
If [`@typescript-eslint/ban-ts-ignore`][typescript-eslint-ban-ts-ignore] rule is enabled in [typescript-eslint][typescript-eslint], please exclude the `// @ ts-ignore` comment from [ESLint][eslint].

<!-- prettier-ignore -->
```ts
// eslint-disable-next-line @typescript-eslint/ban-ts-ignore
// @ts-ignore: Cannot find module
import mock from './mocks/$mock'
```

## Troubleshooting

### `The expected type comes from property 'get' which is declared here on type 'MockMethods'` error in TypeScript

When returning a response asynchronously with [TypeScript][typescript], an error occurs because the type does not match if you try to make the response an array.  
Assert `MockResponse` or return it as an object.

Example of error (**Note that the axios-mock-server build will pass!**)

<!-- prettier-ignore -->
```ts
import { MockMethods } from 'axios-mock-server'

const methods: MockMethods = {
  async get() {
    await new Promise(resolve => setTimeout(resolve, 100))
    return [200, { foo: 'bar' }] // Error
  }
}

export default methods
```

Resolve by asserting `MockResponse`.

<!-- prettier-ignore -->
```ts
import { MockMethods, MockResponse } from 'axios-mock-server'

const methods: MockMethods = {
  async get() {
    await new Promise(resolve => setTimeout(resolve, 100))
    return [200, { foo: 'bar' }] as MockResponse // Type safe
  }
}

export default methods
```

If the response can be an object, no assertion is required.

<!-- prettier-ignore -->
```ts
import { MockMethods } from 'axios-mock-server'

const methods: MockMethods = {
  async get() {
    await new Promise(resolve => setTimeout(resolve, 100))
    return { status: 200, data: { foo: 'bar' } } // Type safe
  }
}

export default methods
```

## Command Line Interface Options

The following options can be specified in the Command Line Interface.

<table>
  <thead>
    <tr>
      <th>Option</th>
      <th>Type</th>
      <th>Default</th>
      <th width="100%">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td nowrap><code>--config</code><br /><code>-c</code></td>
      <td><code>string</code></td>
      <td><code>".mockserverrc"</code></td>
      <td>Specify the path to the configuration file.</td>
    </tr>
    <tr>
      <td nowrap><code>--watch</code><br /><code>-w</code></td>
      <td></td>
      <td></td>
      <td>
        Enable watch mode.<br />
        Regenerate <code>$mock.js</code> or <code>$mock.ts</code> according to
        the increase / decrease of the API endpoint file.
      </td>
    </tr>
    <tr>
      <td nowrap><code>--version</code><br /><code>-v</code></td>
      <td></td>
      <td></td>
      <td>Print axios-mock-server version.</td>
    </tr>
  </tbody>
</table>

## Configuration

Settings are written in `.mockserverrc` file with JSON syntax.

<table>
  <thead>
    <tr>
      <th>Option</th>
      <th>Type</th>
      <th>Default</th>
      <th width="100%">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>input</code></td>
      <td><code>string | string[]</code></td>
      <td><code>"mocks" or "apis"</code></td>
      <td>
        Specify the directory where the API endpoint file is stored.<br />
        If multiple directories are specified, <code>$mock.js</code> or
        <code>$mock.ts</code> is generated in each directory.
      </td>
    </tr>
    <tr>
      <td><code>outputExt</code></td>
      <td><code>"js" | "ts"</code></td>
      <td></td>
      <td>
        Specify the extension of the file to be generated.<br />
        The default is set automatically from scripts of the API endpoint file.
      </td>
    </tr>
    <tr>
      <td><code>outputFilename</code></td>
      <td><code>string</code></td>
      <td><code>"$mock.js" or "$mock.ts"</code></td>
      <td>Specify the filename to be generated.</td>
    </tr>
    <tr>
      <td><code>target</code></td>
      <td><code>"es6" | "cjs"</code></td>
      <td></td>
      <td>
        Specify the code of the module to be generated.<br />
        The default is set automatically from extension of the API endpoint
        file.
      </td>
    </tr>
  </tbody>
</table>

## License

axios-mock-server is licensed under a [MIT License][axios-mock-server-license].

<!-- URL: axios-mock-server -->

[axios-mock-server-examples]: https://github.com/solufa/axios-mock-server/tree/develop/examples
[axios-mock-server-license]: https://github.com/solufa/axios-mock-server/blob/develop/LICENSE

<!-- URL: Badges -->

[badge-bundlephobia-url]: https://bundlephobia.com/result?p=axios-mock-server@latest
[badge-bundlephobia]: https://img.shields.io/bundlephobia/min/axios-mock-server
[badge-ci-url]: https://circleci.com/gh/solufa/axios-mock-server
[badge-ci]: https://img.shields.io/circleci/build/github/solufa/axios-mock-server.svg?label=test
[badge-coverage-url]: https://codecov.io/gh/solufa/axios-mock-server
[badge-coverage]: https://img.shields.io/codecov/c/github/solufa/axios-mock-server.svg
[badge-dependabot]: https://api.dependabot.com/badges/status?host=github&repo=solufa/axios-mock-server
[badge-lgtm-url]: https://lgtm.com/projects/g/solufa/axios-mock-server/context:javascript
[badge-lgtm]: https://img.shields.io/lgtm/grade/javascript/g/solufa/axios-mock-server.svg
[badge-license]: https://img.shields.io/npm/l/axios-mock-server
[badge-npm-url]: https://www.npmjs.com/package/axios-mock-server
[badge-npm]: https://img.shields.io/npm/v/axios-mock-server

<!-- URL: General -->

[axios-instance]: https://github.com/axios/axios#creating-an-instance
[axios]: https://github.com/axios/axios
[dependabot]: https://dependabot.com
[eslint]: https://eslint.org
[git]: https://git-scm.com/
[javascript]: https://developer.mozilla.org/en-US/docs/Web/JavaScript
[nodejs]: https://nodejs.org/
[npm]: https://www.npmjs.com/
[nuxtjs-axios]: https://github.com/nuxt-community/axios-module
[nuxtjs-routing]: https://nuxtjs.org/guide/routing
[nuxtjs]: https://nuxtjs.org/
[typescript-eslint-ban-ts-ignore]: https://github.com/typescript-eslint/typescript-eslint/blob/master/packages/eslint-plugin/docs/rules/ban-ts-ignore.md
[typescript-eslint]: https://github.com/typescript-eslint/typescript-eslint
[typescript]: https://www.typescriptlang.org/
[yarn]: https://yarnpkg.com/
