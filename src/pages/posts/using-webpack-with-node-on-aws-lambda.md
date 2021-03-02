---
title: Using Webpack with Nodejs on AWS Lambda
subtitle: lorem-ipsum
date: '2021-02-26'
thumb_img_alt: lorem-ipsum
excerpt: lorem-ipsum
hide_header: false
seo:
  title: ''
  description: ''
  robots: []
  extra: []
  type: stackbit_page_meta
template: post
---
## Why use Webpack with AWS Lambda?

Webpack allows us to bundle a potentially massive project into a single, neat, minified file. This means it is much easier to move this into Lambda and also means any unused dependencies get trimmed out of the final file, drastically decreasing file size.

## Steps

### Add a Webpack Config

Create a **webpack.config.js** file in the root of your node project.

```Javascript
const path = require('path');
const fs = require('fs');
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin');

const debug = process.env.NODE_ENV !== 'production';

const nodeModules = {};
fs.readdirSync('node_modules')
  .filter((item) => ['.bin'].indexOf(item) === -1) // exclude the .bin folder
  .forEach((mod) => {
    nodeModules[mod] = 'commonjs ' + mod;
  });

module.exports = {
  mode: debug ? 'development' : 'production',
  // mode: "production",
  entry: './app.ts',
  devtool: 'source-map',
  externals: nodeModules,
  resolve: {
    extensions: ['.js', '.jsx', '.json', '.ts', '.tsx'],
  },
  output: {
    libraryTarget: 'commonjs2',
    library: 'index',
    path: path.join(__dirname, 'dist'),
    filename: 'index.js',
  },
  target: 'node',
  module: {
    rules: [
      {
        test: /\.(ts|js)x?$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'cache-loader',
            options: {
              cacheDirectory: path.resolve('.webpackCache'),
            },
          },
          {
            loader: 'babel-loader',
            options: {
              babelrc: true,
            },
          },
          // "babel-loader",
        ],
      },
    ],
  },
  plugins: [new ForkTsCheckerWebpackPlugin()],
};
```

### Install Dependencies

Install the required NPM dependencies

* webpack
* webpack-cli
* typescript
* ts-node
* fork-ts-checker-webpack-plugin
* cache-loader
* babel-loader
* @babel/preset-typescript
* @babel/preset-env
* @babel/core

```
npm i -D webpack webpack-cli typescript ts-node fork-ts-checker-webpack-plugin cache-loader babel-loader @babel/preset-typescript @babel/preset-env @babel/core
```

### Add a Typescript config

Create a tsconfig.json file in the root of your Node project.

```JSON
{
  "compilerOptions": {
    "sourceMap": true,
    "target": "esnext",                          /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019', 'ES2020', or 'ESNEXT'. */
    "module": "es2015",                     /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', 'es2020', or 'ESNext'. */
    "outDir": "./dist",
    "strict": false,
    "types": ["node"],
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "lib": ["ES2020.Promise"]
  },
  "include": ["./**/*"],
  "exclude": ["node_modules", "**/*.test.ts"]
}
```

### Add a Babel config

Create a .babelrc file in the root of your Node project.

```
{
    "presets": [
        [
            "@babel/preset-env",
            {
                "targets": {
                    "node": "12"
                }
            }
        ],
        ["@babel/preset-typescript"]
    ]
}
```

### Update package.json

Add a new script to the NPM package.json to build using webpack.

```"build": "webpack"```

### Update any build scripts or buildspec.yml

If you were previously copying any node_module resources as a build step, be sure to remove this from any build script or buildspec.yml files.

### Add a step to manually copy the package.json to the dist/ directory

You need to include the package.json in the final dist/ directory, so add a build step to copy this over either in any local build script or in a buildspec.yml e.g.

```YAML
version: 0.2
phases:
  build:
    commands:
      - npm i
      - npm run build
      - cp package.json dist/package.json
      ...
```

### Update Git Ignore

Add the new `dist/` folder and `.webpackCache` folders to your .gitignore

### Ensure the entrypoint export is correct

Make sure that the format of your main entry point is exported correctly!
This is easy to overlook and AWS Lambda will complain about not finding your handler.

You need to use the format:

```
exports.myHandler = async(event) => { ... } 
```

**NOT** 

```
export const myhandler = ...
```
or any other variation.

### Update any CloudFormation or SAM template

These need to be updated to correctly reference the new location of the built file I.E. in dist/

For example:

```
CodeUri: Myfunction/dist/
Handler: index.myEntryHandler
```