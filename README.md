# Project is continued at [PuzzleJs](https://github.com/Acanguven/PuzzleJs)

____
____
____

[![Build Status](https://travis-ci.org/Acanguven/TyStream.svg?branch=master)](https://travis-ci.org/Acanguven/Milla)
[![codecov](https://codecov.io/gh/Acanguven/TyStream/branch/master/graph/badge.svg)](https://codecov.io/gh/Acanguven/Milla)
[![npm version](https://badge.fury.io/js/TyStream.svg)](https://www.npmjs.com/package/Milla)
[![Known Vulnerabilities](https://snyk.io/test/github/Acanguven/TyStream/badge.svg)](https://snyk.io/test/github/Acanguven/Milla)
___

# TyStream - Streaming Layout Service

TyStream is a layout service that generates web pages from asynchronous fragments. It is a friendly and **fast** library that helps you optimizing your web pages with progressive rendering. TyStream has built-in integration with [ExpressJs](https://expressjs.com/) .

### [Heroku Demo](https://milla-stream.herokuapp.com/)
![TyStream Progressive Rendering](https://i.imgur.com/9TEIFCJ.gif)

## Contributions

**Pull requests are welcome!**

## Fragments

### What is fragment and what makes it different from components?

We create component because of their reusabilities. We can use them anywhere we want, in a search bar, in a login popup or anything else.
But fragments are encapsulated modules with specific business. Like an header, product container, footer.
They are all independent from each other.

### How TyStream approaches to fragments?

TyStream is designed to stream fragments asynchronously to the clients. Without any single XHR from browser, all fragments are delivered to clients when their data is ready. Like single page application but fully server side rendered. So it makes it SEO friendly and great for e-commerce websites.
TyStream shows placeholders on the clients browser while it fetches your data then replaces placeholder with real content. So it is really improving your **time to first byte**

**You can easily improve your Speed Index**!

### How it works?

*This is not a valid TyStream usage, please see usage details below*

Whenever user provides a html to TyStream, TyStream converts it to stream friendly format and flushes to user asynchronously.

Lets assume that our header service responds in 300 ms and product service in 140ms.

```html
<html>
    <head>
        <milla-head></milla-head>
    </head>
    <body>
        <fragment name="header" css="pathtocss" js="pathtojs"></fragment>
        <div>A static content</div>
        <fragment name="product" css="pathtocss" js="pathtojs"></fragment>
    </body>
</html>
```

Converted:
```html
<html>
    <head>
        <style>/* Fragment styles */</style>
        <script>/*Really small TyStream js. 118bytes*/</script>
    </head>
    <body>
        <div id="p_0">Header placeholder</div>
        <div>A static content</div>
        <div id="p_1">Product placeholder</div>
        <!-- User started load html dependencies, css files etc and your website is already rendered -->
        ... 140ms
        <!-- TyStream recieved product data and flushed product content and js -->
        <div id="c_1" hidden></div>
        <script>/*use head script to change c_1 p_1; run header script */</script>
        <!-- Product is ready to be used -->
        ... 160ms
        <!-- TyStream recieved header data and flushed product content and js -->
        <div id="c_1" hidden></div>
        <script>/*use head script to change c_1 p_1; run header script */</script>
        <!-- header is ready to be used -->
    </body>
</html>
```

TyStream closes the stream.

# Installation

Using npm
```bash
npm install TyStream --save
```

Using yarn
```bash
yarn add milla
```

___

# Usage

*You can check out demo folder*

## Page File

First of all, you should create a page html file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>TyStream Title</title>
    <milla-head></milla-head>
</head>
<body>
    <fragment name="header" js="../test/dependencies/8.js" css="../test/dependencies/8.css"></fragment>
    <div>Middle Content</div>
    <fragment name="product" js="../test/dependencies/8_1.js"></fragment>
</body>
</html>
```

Then use your page file in your express route.

## Express Route

```javascript
    const TyStream = require('milla');

    /* Optional, you can find default configs under src/configuration */
    TyStream.config.set({
      minifyJs: false,
      minifyCss: false,
      minifyHtml: false
    });

    /* Create express route */
    app.get('/', TyStream.express({
      htmlFile: path.join(__dirname, '../test/html/test8.html'), //Html with fragments
      data: { //fragment datas. You can use () => {}, {}, () => new Promise. TyStream will handle them in most efficient way.
        header: (req) => {
          return {
            header: {
              menu: ['Item 1', 'Item 2', 'Item 3']
            }
          }
        },
        product: (req) => {
          return new Promise((resolve, reject) => {
            setTimeout(() => {
              resolve({product:{name:'Test'}});
            }, 4000 * Math.random())
          });
        }
      },
      fragments: { //Fragment rendering functions. (input) is coming from data. You can use your favorite template engine here (anything that returns string).
        header: {
          content: (input) => JSON.stringify(input)
        },
        product: {
          placeholder: () => '<div>Product placeholder</div>',
          content: (input) => {
            return JSON.stringify(input);
          }
        }
      }
    }));
```

Thats all! You have a great progressive rendering now.
