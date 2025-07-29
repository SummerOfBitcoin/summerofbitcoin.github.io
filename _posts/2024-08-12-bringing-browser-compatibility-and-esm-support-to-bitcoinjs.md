---
layout: post
title: "Bringing Browser Compatibility and ESM Support to BitcoinJS"
date: 2024-08-12
author: Ayman Mohammed
categories: [BitcoinJS, Tutorials]
image: ""
---

Let's say you decide to make a Bitcoin app. You setup a vite project (or even a remix one), you install `bitcoinjs-lib`. You write a few lines of code using bitcoinjs-lib and you start up your local dev server and you're hit with this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724835566295/73d6c90e-4c9b-441c-9220-39731b8c6a7a.png align="center")

This happens because there's no `Buffer` module available in browser API. When you use a function in `bitcoinjs-lib` (which makes use of `Buffer`) then an error is thrown because Buffer is not defined. The traditional way to make use of `bitcoinjs-lib` is to make use of buffer polyfills. Vite has a node-polyfills plugin that you can add to your app. If you're making use of Next.js then it already has those polyfills in it.

Or let's say you're trying to build to build a state-of-the-art library ESM library that makes use of bitcoinjs-lib but you can't because BitcoinJS' published packages (so far) only support cjs.

## A primer on the fractured nodejs module ecosystem

Skip this section if you're already familiar with the terms cjs and esm. For those, who are not familiar with these terms I'll try to give a brief introduction to these ecosystem.

CJS (or CommonJS) is part of the traditional module standard in Node.js. This module format is what was followed (and still is) in Node.js. However, the problem with it is that it is not natively supported in the browser. ESM (ECMAScript Module) however, is part of a newer standard that is compatible with the browser. Here's a quick comparison of CJS and ESM code:

#### CJS

```javascript
// Importing
const module = require('./module');

// Exporting
module.exports = {
  function1,
  function2
};
```

#### ESM

```javascript
// Importing
import module from './module';

// Exporting
export function function1() {}
export function function2() {}
```

There are other niche characteristics such as asynchronous loading of modules in ESM but the gist of it is:

CJS = Old, Node.js, uses require.

ESM = New, Browser, uses import.

## Adding ESM support

A lot of the BitcoinJS libraries are written in TypeScript. CJS is already supported by BitcoinJS, adding ESM support is simply done by specifying the target format of the built javascript in the tsconfig. This is what the previous tsconfig.json looked like:

```json
{
  "compilerOptions": {
    "strict": true,
    "outDir": "dist",
    "target": "es2015",
    "module": "commonjs",
    "noImplicitAny": true,
    "preserveConstEnums": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "src/test/types.ts"]
}
```

This config is taken from the `bech32` repository which is used for encoding and decoding segwit (and taproot) addresses.

Since we want to output both formats we'll need to use both formats. We'll create a tsconfig.base.json where all the common config is defined. This will be extended by the cjs and esm tsconfig later.

The tsconfig.base.json will be as follows:

```json
{
  "compilerOptions": {
    "strict": true,
    "resolveJsonModule": true,
    "target": "es2015",
    "moduleResolution": "node",
    "noImplicitAny": true,
    "preserveConstEnums": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

The `moduleResolution` flag tells typescript how to resolve imports. In this case we set it to be the same as node. We also added the `resolveJsonModule` field as we're importing fixtures in json for the test cases.

We'll create a `tsconfig.cjs.json`:

```json
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "declaration": true,
    "emitDeclarationOnly": false,
    "outDir": "dist/cjs",
    "module": "commonjs"
  }
}
```

Notice that we're also emitting the declarations (.d.ts files) in cjs. This is to make it easier for bundlers to figure out where the declaration files as they first search cjs modules. We'll also create a separate folder for cjs called `dist/cjs` where all cjs code resides.

The esm tsconfig is similar to the cjs one:

```json
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "outDir": "dist/esm",
    "module": "ESNext"
  }
}
```

This config will create an esm build inside `dist/esm`. Now to build the cjs and esm config, we'll have to update the build scripts. The new build command is:

```json
"build": "tsc -p tsconfig.json && tsc -p tsconfig.cjs.json",
```

Notice that we're now building both the modules. This isn't enough for other projects to use our code. We'll have add conditional exports to our package.json so that other apps will be able to use it:

```json
"exports": {
    ".": {
      "require": "./dist/cjs/index.cjs",
      "import": "./dist/esm/index.js",
      "types": "./dist/cjs/index.d.ts"
    }
  }
```

Exports is a top level field inside `package.json`. The `.` specifies root level exports. The require field points to the cjs module(remember from the primer that cjs uses require), the `import` field resolves to the esm module and the types field to the index.d.ts declarations. What this means is:

* If someone does `const bech32 = require('bech32')` then this would resolve to `bech32/dist/cjs/index.cjs`.
    
* If someone does `import * as bech32 from "bech32"` then this would resolve to `bech32/dist/esm/index.js`.
    

Notice that the cjs files have a `.cjs` extension where as esm files have a `.js` extension? This is because of specifying `type: "module"` at the root level in `package.json`. When the "module" property is specified then the entire project is considered to be an es module. This is why you don't have to add a separate `.mjs` extension to esm module files whereas you have to add `.cjs` to the cjs files.

Obviously, doing this throughout the ecosystem is not so easy since:

* ESM support is still not there for a lot of libraries. We had to find replacements for libraries that use ESM so that we can make use of them in the BitcoinJS ecosystem.
    
* A lot of the `cryptocoinjs` libraries themselves had to be converted into esm before even starting with bitcoinjs libraries.
    

## Adding browser compatibility

A lot of the work in this section was using the browser's crypto API and moving from `Buffer` to `Uint8Array`.

Since Uint8Array array does not have a lot of the helper functions from the buffer API (such as reading little endian integers) I had to add quite a few polyfills to the `uint8array-tools` library achieve the same functionality as the Node.js Buffer API.

## PRs

Here's a list of all the pull requests I made to bitcoinjs as part of my Summer of Bitcoin program:

[https://github.com/bitcoinjs/bech32/pull/53](https://github.com/bitcoinjs/bech32/pull/53%EF%BF%BChttps://github.com/bitcoinjs/bech32/pull/55)

[https://github.com/bitcoinjs/bech32/pull/55](https://github.com/bitcoinjs/bech32/pull/55)

[https://github.com/bitcoinjs/bip39/pull/195](https://github.com/bitcoinjs/bip39/pull/195)

[https://github.com/cryptocoinjs/bs58/pull/48](https://github.com/cryptocoinjs/bs58/pull/48)

[https://github.com/bitcoinjs/bs58check/pull/45](https://github.com/bitcoinjs/bs58check/pull/45)

[https://github.com/bitcoinjs/wif/pull/35](https://github.com/bitcoinjs/wif/pull/35)

[https://github.com/bitcoinjs/wif/pull/34](https://github.com/bitcoinjs/wif/pull/34)

[https://github.com/bitcoinjs/bip66/pull/17](https://github.com/bitcoinjs/bip66/pull/17)

[https://github.com/bitcoinjs/varuint-bitcoin/pull/12](https://github.com/bitcoinjs/varuint-bitcoin/pull/12)

[https://github.com/bitcoinjs/varuint-bitcoin/pull/13](https://github.com/bitcoinjs/varuint-bitcoin/pull/13)

[https://github.com/bitcoinjs/uint8array-tools/pull/6](https://github.com/bitcoinjs/uint8array-tools/pull/6)

[https://github.com/bitcoinjs/uint8array-tools/pull/8](https://github.com/bitcoinjs/uint8array-tools/pull/8)

[https://github.com/bitcoinjs/uint8array-tools/pull/10](https://github.com/bitcoinjs/uint8array-tools/pull/10)

[https://github.com/bitcoinjs/bip21/pull/18](https://github.com/bitcoinjs/bip21/pull/18)

[https://github.com/bitcoinjs/bip21/pull/19](https://github..com/bitcoinjs/bip21/pull/19)

[https://github.com/bitcoinjs/bip32/pull/88](https://github.com/bitcoinjs/bip32/pull/88)

[https://github.com/bitcoinjs/bip32/pull/87](https://github.com/bitcoinjs/bip32/pull/87)

[https://github.com/bitcoinjs/bip174/pull/39](https://github.com/bitcoinjs/bip174/pull/39)

[https://github.com/bitcoinjs/uint8array-tools/pull/11](https://github.com/bitcoinjs/uint8array-tools/pull/11)

[https://github.com/Nesopie/bitcoinjs-lib/tree/feat/hybrid](https://github.com/Nesopie/bitcoinjs-lib/tree/feat/hybrid) (will raise a PR once the other PRs are merged).
