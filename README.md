# useref

Parse build blocks in HTML files to replace references

Extracted from the grunt plugin [grunt-useref](https://github.com/pajtai/grunt-useref).

## About this fork

In addition to the original, this fork offers logging in case unknown block types are encountered (so that we can see typos)
and allows a catch-all handler for unknown block types to be used (which disables any logging unless you log messages yourself).

## Installation

```
npm install node-useref
```

## Usage

```js
useref = require('node-useref')
var result = useref(inputHtml)
// result = [ replacedHtml, { type: { path: { 'assets': [ replacedFiles] }}} ]
```


Blocks are expressed as:

```html
<!-- build:<type>(alternate search path) <path> <parameters> -->
... HTML Markup, list of script / link tags.
<!-- endbuild -->
```

- **type**: either `js`, `css` or `remove`
- **alternate search path**: (optional) By default the input files are relative to the treated file. Alternate search path allows one to change that
- **path**: the file path of the optimized file, the target output
- **parameters**: extra parameters that should be added to the tag

An example of this in completed form can be seen below:

```html
<html>
<head>
  <!-- build:css css/combined.css -->
  <link href="css/one.css" rel="stylesheet">
  <link href="css/two.css" rel="stylesheet">
  <!-- endbuild -->
</head>
<body>
  <!-- build:js scripts/combined.js -->
  <script type="text/javascript" src="scripts/one.js"></script>
  <script type="text/javascript" src="scripts/two.js"></script>
  <!-- endbuild -->

  <!-- build:js scripts/async.js async data-foo="bar" -->
  <script type="text/javascript" src="scripts/three.js"></script>
  <script type="text/javascript" src="scripts/four.js"></script>
  <!-- endbuild -->
</body>
</html>
```

The module would be used with the above sample HTML as follows:

```js
var result = useref(sampleHtml)

// [
//   resultHtml,
//   {
//     css: {
//       'css/combined.css': {
//         'assets': [ 'css/one.css', 'css/two.css' ]
//       }
//     },
//     js: {
//       'scripts/combined.js': {
//         'assets': [ 'scripts/one.js', 'scripts/two.js' ]
//       },
//       'scripts/async.js': {
//          'assets': [ 'scripts/three.js', 'scripts/four.js' ]
//        }
//     }
//   }
// ]
```


The resulting HTML would be:

```html
<html>
<head>
  <link rel="stylesheet" href="css/combined.css"/>
</head>
<body>
  <script src="scripts/combined.js"></script>
  <script src="scripts/async.js" async data-foo="bar" ></script>
</body>
</html>
```

## IE Conditional Comments

Internet Explorer Conditional Comments are preserved. The code below:

```html
<!-- build:js scripts/combined.js   -->
<!--[if lt IE 9]>
<script type="text/javascript" src="scripts/this.js"></script>
<script type="text/javascript" src="scripts/that.js"></script>
<![endif]-->
<!-- endbuild -->
```

Results in:

```html
<!--[if lt IE 9]>
<script src="scripts/combined.js"></script>
<![endif]-->
```

### Custom blocks

Sometimes you need a bit more. If you would like to do custom processing, this is possible with a custom block, as demonstrated below.

```
<!-- build:import components -->
<link rel="import" href="/bower_components/some/path"></link>
<!-- endbuild -->
```

With

```
useref = require('node-useref')
var result = useref(inputHtml, {
  // each property corresponds to any blocks with the same name, e.g. "build:import"
  import: function (content, target, options) {
    // do something with `content` and return the desired HTML to replace the block content
    return content.replace('bower_components', target);
  }
});
```

Becomes

```
<link rel="import" href="/components/some/path"></link>
```

The handler function gets the following arguments:

- *content* (String): The content of the custom use block
- *target* (String): The "path" value of the use block definition
- *options* (String): The extra attributes from the use block definition, the developer can parse as JSON or do whatever they want with it

Include a handler for each custom block type.

A warning will be issued for unknown block types. You can provide a catch-all handler for custom block types
if you have several similar blocks when the handler can profit from the similarity.

The unknown block handler should look like this:

```
useref = require('node-useref')
var result = useref(inputHtml, {
  // called for all blocks that do not have a specific handler:
  unknownBlock: function (type, content, target, attrs) {
    // do something with `content` and return the desired HTML to replace the block content
    switch (type) {
      case 'import':
      	   // TODO: think of a good example
    }

    // you can use this for custom logging:
    console.log("UNKNOWN BLOCK TYPE: "+type);

    // return null to have the block removed
    return null;
  }
});
```

The unknownBlock handler function gets the following arguments:

- *type* (String): The type of the custom block
- *content* (String): The content of the custom use block
- *target* (String): The "path" value of the use block definition
- *attrs* (String): The extra attributes from the use block definition, the developer can parse as JSON or do whatev\
er they want with it
