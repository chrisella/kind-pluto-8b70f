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

