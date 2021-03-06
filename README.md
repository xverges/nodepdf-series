[![Build Status](https://travis-ci.org/arve0/nodepdf-series.svg?branch=v0.0.3)](https://travis-ci.org/arve0/nodepdf-series)
# nodepdf-series

Fork of [nodepdf](https://github.com/TJkrusinski/NodePDF) to support
multiple/series of pages:

```js
var glob = require('glob');
var path = require('path');
var PDF = require('nodepdf-series');

glob('**/*.html', function(e, files){
	files = files.map(function(file){
		return 'file://' + path.resolve(file);
	});
	PDF.render(files, function(err){
		// will create /path/to/file1.pdf, /path/to/file2.pdf, etc
		console.log('done');
	});
});
```

Or:
```js
var PDF = require('nodepdf-series');
PDF.render(['http://host/page.html', 'http://host/'], function(err){
	// will create ./host/page.pdf and host.pdf
});
```

When not spawning a new phantomjs for each page, we get some
extra performance. Here is a test on 95 local html files:
```shell
$ time node nodepdf.js
real    9m32.928s
user    5m56.769s
sys     0m56.142s

$ time node nodepdf-series.js
real    2m47.053s
user    1m55.567s
sys     0m5.104s
```

## Installation

```
npm install nodepdf-series
```


## Contsructor API

You can use nodepdf-series two ways, one is using a contstructor that returns
an instance of `EventEmitter`.

```js
// last argument is optional, sets the width and height for the viewport to
// render the pdf from. (see additional options)
var pdf = new PDF('http://www.google.com', {
	viewportSize: {
		width: 1440,
		height: 900
	},
	args: '--debug=true'
});

pdf.on('error', function(msg){
	console.log(msg);
});

pdf.on('done', function(){
	console.log('done');
});

// listen for stdout from phantomjs
pdf.on('stdout', function(stdout){
	 console.log(stdout);
});

// listen for stderr from phantomjs
pdf.on('stderr', function(stderr){
	console.log(stderr);
});

```
Or set the content directly instead of using a URL (will save to about:blank.pdf):
```js
var pdf = new PDF(null, {
	content: '<html><body><img src="https://www.google.com/images/srpr/logo11w.png" />' +
	 				 '</body></html>'
});
```


You can set the header and footer contents aswell:
```js
var pdf = new PDF('http://yahoo.com', {
	header: {
		height: '1cm',
		contents: 'HEADER {currentPage} / {pages}'
		// If you have 2 pages the result looks like this: HEADER 1 / 2
	},
	footer: {
		height: '1cm',
		contents: 'FOOTER {currentPage} / {pages}'
	}
});
```

## Callback API

The callback API follows node standard callback signatures using the
`render()` method.

```js
var PDF = require('nodepdf-series');

// options is optional
PDF.render('http://www.google.com', options, function(err){
	// handle error
});

// use default options
PDF.render('http://www.google.com', function(err){
	// handle error
});

```

As soon the content option is set, the URL is ignored even if you set one.

## Options + Defaults
```js
{
	viewportSize: {
		width: 2880,
		height: 1440
	},
	paperSize: {
		format: 'A4',
		orientation: 'portrait',
		margin: {
			top: '1cm',
			right: '1cm',
			bottom: '1cm',
			left: '1cm'
		}
	},
	zoomFactor: 1,
	args: '',
	captureDelay: 0
}
```

You can set all the properties from here: http://phantomjs.org/api/webpage/

## Cookies

```js
var pdf = new PDF('http://yahoo.com', {
	cookies: [{
		name:     'Valid-Cookie-Name 1',   /* required property */
		value:    'Valid-Cookie-Value 1',  /* required property */
		domain:   'localhost',           /* required property */
		path:     '/foo',
		httponly: true,
		secure:   false,
		expires:  (new Date()).getTime() + (1000 * 60 * 60) /* expires in 1 hour */
	},{
		name:     'Valid-Cookie-Name 2',
		value:    'Valid-Cookie-Value 2',
		domain:   'localhost'
	}]
});
```

PhantomJS Cookie Object description: http://phantomjs.org/api/webpage/property/cookies.html
