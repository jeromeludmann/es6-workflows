npm-workflow
============

Below is a TypeScript-oriented build workflow using `npm`, without Grunt or Gulp, for front and back-end JavaScript development.

TypeScript as a superset of JavaScript
--------------------------------------

Install TypeScript compiler and linter:

```sh
npm install --save-dev typescript
npm install --save-dev tslint
```

Configure `tsconfig.json`:

```json
{
  "compileOnSave": false,
  "compilerOptions": {
    "module": "commonjs",
    "target": "es6",
    "noImplicitAny": false,
    "sourceMap": true,
    "listFiles": true,
    "pretty": true,
    "allowJs": true
  },
  "exclude": [
    "node_modules",
    "build"
  ]
}
```

Initialize `tslint.json`:

```sh
./node_modules/.bin/tslint --init
```

Add to `package.json`:

```json
"scripts": {
  "server:lint:ts": "tslint src/server/**/*.ts",
  "server:tsc": "tsc --project src/server --rootDir src --sourceMap --outDir build/.tmp/server/es6",
  "client:lint:ts": "tslint src/client/**/*.ts",
  "client:tsc": "tsc --project src/client --rootDir src --sourceMap --outDir build/.tmp/client/es6",
}
```

ES2015 (ES6) to ES5 JavaScript
------------------------------

Install Babel and ESLint:

```sh
npm install --save-dev babel-cli babel-preset-es2015 babel-plugin-transform-runtime
npm install --save-dev eslint
```

- `babel-preset-es2015` enables all ES2015 (ES6) features
- `babel-plugin-transform-runtime` externalizes some helpers for ES7 features (like `async`)

Add to `.babelrc`:

```json
{
  "presets": [
    "es2015"
  ],
  "plugins": [
    [
      "transform-runtime",
      {
        "polyfill": false,
        "regenerator": true
      }
    ]
  ]
}
```

Initialize `.eslintrc.json`:

```sh
./node_modules/.bin/eslint --init
```

Add to `package.json`:

```json
"scripts": {
  "server:lint:js": "eslint src/server/**/*.js",
  "server:babel": "babel build/.tmp/server/es6 --source-maps --out-dir build/.tmp/server/es5",
  "client:lint:js": "eslint src/client/**/*.js",
  "client:babel": "babel build/.tmp/client/es6 --source-maps --out-dir build/.tmp/client/es5",
}
```

Bundling dependencies (browser-side)
------------------------------------

Install Browserify and UglifyJS:

```sh
npm install --save-dev browserify exorcist
npm install --save-dev uglifyjs
```

- `exorcist` gets the Source Map from Browserify stream 

Add to `package.json`:

```json
"scripts": {
  "client:browserify": "browserify build/.tmp/client/es5/client/app.js --debug | exorcist build/.tmp/client/bundle.js.map --base build/.tmp/client > build/.tmp/client/bundle.js",
  "client:uglify": "uglifyjs build/.tmp/client/bundle.js --compress --mangle --source-map build/.tmp/client/bundle.min.js.map --prefix relative --output build/.tmp/client/bundle.min.js",
}
```

Multi-level Source Map
----------------------

```sh
npm install --save-dev sorcery
```

- `sorcery` generates Source Maps by resolving a chain of Source Maps

Add to `package.json`:

```json
"scripts": {
  "server:sorcery": "sorcery --input build/.tmp/server/es5 --output build/server-dist",
  "client:sorcery": "sorcery --input build/.tmp/client/bundle.min.js --output build/client-dist/bundle.min.js",
}
```

Source Map support
------------------

### Node-side support

```sh
npm install --save source-map-support
```

Add `--require source-map-support/register` to node arguments:

```json
"scripts": {
  "server:start": "node --require source-map-support/register build/server-dist/server/app.js",
}
```

### Browser-side support

Open _Developer Tools_ from browser menu. 

Make these things work together
-------------------------------

Install command-line tools:

```sh
npm install --save-dev npm-run-all
npm install --save-dev nodemon
npm install --save-dev rimraf
```

- `npm-run-all` runs multiple scripts in parallel or sequential
- `nodemon` monitors for any changes (like `gulp.watch()`)
- `rimraf` is a cross-platform `rm -rf`

`package.json`:

```json
  "scripts": {
    "clean": "rimraf build",
    "server:build": "npm-run-all --parallel server:lint:* server:mocha --sequential server:tsc server:babel server:sorcery",
    "server:watch": "nodemon --watch src/server --ext ts,js --exec 'npm-run-all --sequential server:build server:start'",
    "client:build": "npm-run-all --parallel client:lint:* client:mocha --sequential client:tsc client:babel client:browserify client:uglify client:sorcery client:copy",
    "client:watch": "nodemon --watch src/client --ext ts,js,html --exec 'npm-run-all --sequential client:build client:start'",
    "build": "npm-run-all --parallel server:build client:build",
    "watch": "nodemon --watch src --ext ts,js --exec 'npm-run-all --sequential build start'"
  }
```

## Finally

`package.json` looks like:

```json
  "scripts": {
    "clean": "rimraf build",
    "server:lint:ts": "tslint src/server/**/*.ts",
    "server:lint:js": "eslint src/server/**/*.js",
    "server:mocha": "mocha --require ts-node/register test/*.test.ts",
    "server:tsc": "tsc --project src/server --rootDir src --sourceMap --outDir build/.tmp/server/es6",
    "server:babel": "babel build/.tmp/server/es6 --source-maps --out-dir build/.tmp/server/es5",
    "server:sorcery": "sorcery --input build/.tmp/server/es5 --output build/server-dist",
    "server:start": "node --require source-map-support/register build/server-dist/server/app.js",
    "server:build": "npm-run-all --parallel server:lint:* server:mocha --sequential server:tsc server:babel server:sorcery",
    "server:watch": "nodemon --watch src/server --ext ts,js --exec 'npm-run-all --sequential server:build server:start'",
    "client:lint:ts": "tslint src/client/**/*.ts",
    "client:lint:js": "eslint src/client/**/*.js",
    "client:mocha": "mocha --require ts-node/register test/*.test.ts",
    "client:tsc": "tsc --project src/client --rootDir src --sourceMap --outDir build/.tmp/client/es6",
    "client:babel": "babel build/.tmp/client/es6 --source-maps --out-dir build/.tmp/client/es5",
    "client:browserify": "browserify build/.tmp/client/es5/client/app.js --debug | exorcist build/.tmp/client/bundle.js.map --base build/.tmp/client > build/.tmp/client/bundle.js",
    "client:uglify": "uglifyjs build/.tmp/client/bundle.js --compress --mangle --source-map build/.tmp/client/bundle.min.js.map --prefix relative --output build/.tmp/client/bundle.min.js",
    "client:sorcery": "sorcery --input build/.tmp/client/bundle.min.js --output build/client-dist/bundle.min.js",
    "client:copy": "cp src/client/public/*.html build/client-dist",
    "client:start": "http-server build/client-dist",
    "client:build": "npm-run-all --parallel client:lint:* client:mocha --sequential client:tsc client:babel client:browserify client:uglify client:sorcery client:copy",
    "client:watch": "nodemon --watch src/client --ext ts,js,html --exec 'npm-run-all --sequential client:build client:start'",
    "build": "npm-run-all --parallel server:build client:build",
    "watch": "npm-run-all --parallel server:watch client:watch"
  }
```
