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
