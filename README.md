# api documentation for  [source-map-support (v0.4.14)](https://github.com/evanw/node-source-map-support#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-source-map-support.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-source-map-support) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-source-map-support.svg)](https://travis-ci.org/npmdoc/node-npmdoc-source-map-support)
#### Fixes stack traces for files with source maps

[![NPM](https://nodei.co/npm/source-map-support.png?downloads=true)](https://www.npmjs.com/package/source-map-support)

[![apidoc](https://npmdoc.github.io/node-npmdoc-source-map-support/build/screenCapture.buildNpmdoc.browser.%2Fhome%2Ftravis%2Fbuild%2Fnpmdoc%2Fnode-npmdoc-source-map-support%2Ftmp%2Fbuild%2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-source-map-support/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-source-map-support/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-source-map-support/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "bugs": {
        "url": "https://github.com/evanw/node-source-map-support/issues"
    },
    "dependencies": {
        "source-map": "^0.5.6"
    },
    "description": "Fixes stack traces for files with source maps",
    "devDependencies": {
        "browserify": "3.44.2",
        "coffee-script": "1.7.1",
        "http-server": "^0.8.5",
        "mocha": "1.18.2",
        "webpack": "^1.13.3"
    },
    "directories": {},
    "dist": {
        "shasum": "9d4463772598b86271b4f523f6c1f4e02a7d6aef",
        "tarball": "https://registry.npmjs.org/source-map-support/-/source-map-support-0.4.14.tgz"
    },
    "gitHead": "8db1d646dbe9c05f824001477637b62dbcb49f0c",
    "homepage": "https://github.com/evanw/node-source-map-support#readme",
    "license": "MIT",
    "main": "./source-map-support.js",
    "maintainers": [
        {
            "name": "evanw",
            "email": "evan.exe@gmail.com"
        },
        {
            "name": "julien-f",
            "email": "julien.fontanet@isonoe.net"
        },
        {
            "name": "linusu",
            "email": "linus@folkdatorn.se"
        }
    ],
    "name": "source-map-support",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://github.com/evanw/node-source-map-support.git"
    },
    "scripts": {
        "build": "node build.js",
        "prepublish": "npm run build",
        "serve-tests": "http-server -p 1336",
        "test": "mocha"
    },
    "version": "0.4.14"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module source-map-support](#apidoc.module.source-map-support)
1.  [function <span class="apidocSignatureSpan">source-map-support.</span>getErrorSource (error)](#apidoc.element.source-map-support.getErrorSource)
1.  [function <span class="apidocSignatureSpan">source-map-support.</span>install (options)](#apidoc.element.source-map-support.install)
1.  [function <span class="apidocSignatureSpan">source-map-support.</span>mapSourcePosition (position)](#apidoc.element.source-map-support.mapSourcePosition)
1.  [function <span class="apidocSignatureSpan">source-map-support.</span>retrieveSourceMap (arg)](#apidoc.element.source-map-support.retrieveSourceMap)
1.  [function <span class="apidocSignatureSpan">source-map-support.</span>wrapCallSite (frame)](#apidoc.element.source-map-support.wrapCallSite)



# <a name="apidoc.module.source-map-support"></a>[module source-map-support](#apidoc.module.source-map-support)

#### <a name="apidoc.element.source-map-support.getErrorSource"></a>[function <span class="apidocSignatureSpan">source-map-support.</span>getErrorSource (error)](#apidoc.element.source-map-support.getErrorSource)
- description and source-code
```javascript
function getErrorSource(error) {
  var match = /\n    at [^(]+ \((.*):(\d+):(\d+)\)/.exec(error.stack);
  if (match) {
    var source = match[1];
    var line = +match[2];
    var column = +match[3];

    // Support the inline sourceContents inside the source map
    var contents = fileContentsCache[source];

    // Support files on disk
    if (!contents && fs && fs.existsSync(source)) {
      contents = fs.readFileSync(source, 'utf8');
    }

    // Format the line from the original source code like node does
    if (contents) {
      var code = contents.split(/(?:\r\n|\r|\n)/)[line - 1];
      if (code) {
        return source + ':' + line + '\n' + code + '\n' +
          new Array(column).join(' ') + '^';
      }
    }
  }
  return null;
}
```
- example usage
```shell
...

it('specifically requested error source', function(done) {
  compareStdout(done, createSecondLineSourceMap(), [
    '',
    'function foo() { throw new Error("this is the error"); }',
    'var sms = require("./source-map-support");',
    'sms.install({ handleUncaughtExceptions: false });',
    'process.on("uncaughtException", function (e) { console.log("SRC:" + sms.getErrorSource(e)); });',
    'process.nextTick(foo);'
  ], [
    /^SRC:.*[/\\].original.js:1$/,
    'this is the original code',
    '^'
  ]);
});
...
```

#### <a name="apidoc.element.source-map-support.install"></a>[function <span class="apidocSignatureSpan">source-map-support.</span>install (options)](#apidoc.element.source-map-support.install)
- description and source-code
```javascript
install = function (options) {
  options = options || {};

  if (options.environment) {
    environment = options.environment;
    if (["node", "browser", "auto"].indexOf(environment) === -1) {
      throw new Error("environment " + environment + " was unknown. Available options are {auto, browser, node}")
    }
  }

  // Allow sources to be found by methods other than reading the files
  // directly from disk.
  if (options.retrieveFile) {
    if (options.overrideRetrieveFile) {
      retrieveFileHandlers.length = 0;
    }

    retrieveFileHandlers.unshift(options.retrieveFile);
  }

  // Allow source maps to be found by methods other than reading the files
  // directly from disk.
  if (options.retrieveSourceMap) {
    if (options.overrideRetrieveSourceMap) {
      retrieveMapHandlers.length = 0;
    }

    retrieveMapHandlers.unshift(options.retrieveSourceMap);
  }

  // Support runtime transpilers that include inline source maps
  if (options.hookRequire && !isInBrowser()) {
    var Module;
    try {
      Module = require('module');
    } catch (err) {
      // NOP: Loading in catch block to convert webpack error to warning.
    }
    var $compile = Module.prototype._compile;

    if (!$compile.__sourceMapSupport) {
      Module.prototype._compile = function(content, filename) {
        fileContentsCache[filename] = content;
        sourceMapCache[filename] = undefined;
        return $compile.call(this, content, filename);
      };

      Module.prototype._compile.__sourceMapSupport = true;
    }
  }

  // Configure options
  if (!emptyCacheBetweenOperations) {
    emptyCacheBetweenOperations = 'emptyCacheBetweenOperations' in options ?
      options.emptyCacheBetweenOperations : false;
  }

  // Install the error reformatter
  if (!errorFormatterInstalled) {
    errorFormatterInstalled = true;
    Error.prepareStackTrace = prepareStackTrace;
  }

  if (!uncaughtShimInstalled) {
    var installHandler = 'handleUncaughtExceptions' in options ?
      options.handleUncaughtExceptions : true;

    // Provide the option to not install the uncaught exception handler. This is
    // to support other uncaught exception handlers (in test frameworks, for
    // example). If this handler is not installed and there are no other uncaught
    // exception handlers, uncaught exceptions will be caught by node's built-in
    // exception handler and the process will still be terminated. However, the
    // generated JavaScript code will be shown above the stack trace instead of
    // the original source code.
    if (installHandler && hasGlobalProcessEventEmitter()) {
      uncaughtShimInstalled = true;
      shimEmitUncaughtException();
    }
  }
}
```
- example usage
```shell
...
'''
$ npm install source-map-support
'''

Source maps can be generated using libraries such as [source-map-index-generator](https://github.com/twolfson/source-map-index-generator
). Once you have a valid source map, insert the following line at the top of your compiled code:

'''js
require('source-map-support').install();
'''

And place a source mapping comment somewhere in the file (usually done automatically or with an option by your transpiler):

'''
//# sourceMappingURL=path/to/source.map
'''
...
```

#### <a name="apidoc.element.source-map-support.mapSourcePosition"></a>[function <span class="apidocSignatureSpan">source-map-support.</span>mapSourcePosition (position)](#apidoc.element.source-map-support.mapSourcePosition)
- description and source-code
```javascript
function mapSourcePosition(position) {
  var sourceMap = sourceMapCache[position.source];
  if (!sourceMap) {
    // Call the (overrideable) retrieveSourceMap function to get the source map.
    var urlAndMap = retrieveSourceMap(position.source);
    if (urlAndMap) {
      sourceMap = sourceMapCache[position.source] = {
        url: urlAndMap.url,
        map: new SourceMapConsumer(urlAndMap.map)
      };

      // Load all sources stored inline with the source map into the file cache
      // to pretend like they are already loaded. They may not exist on disk.
      if (sourceMap.map.sourcesContent) {
        sourceMap.map.sources.forEach(function(source, i) {
          var contents = sourceMap.map.sourcesContent[i];
          if (contents) {
            var url = supportRelativeURL(sourceMap.url, source);
            fileContentsCache[url] = contents;
          }
        });
      }
    } else {
      sourceMap = sourceMapCache[position.source] = {
        url: null,
        map: null
      };
    }
  }

  // Resolve the source URL relative to the URL of the source map
  if (sourceMap && sourceMap.map) {
    var originalPosition = sourceMap.map.originalPositionFor(position);

    // Only return the original position if a matching line was found. If no
    // matching line is found then we return position instead, which will cause
    // the stack trace to print the path and line for the compiled file. It is
    // better to give a precise location in the compiled file than a vague
    // location in the original file.
    if (originalPosition.source !== null) {
      originalPosition.source = supportRelativeURL(
        sourceMap.url, originalPosition.source);
      return originalPosition;
    }
  }

  return position;
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.source-map-support.retrieveSourceMap"></a>[function <span class="apidocSignatureSpan">source-map-support.</span>retrieveSourceMap (arg)](#apidoc.element.source-map-support.retrieveSourceMap)
- description and source-code
```javascript
retrieveSourceMap = function (arg) {
  for (var i = 0; i < list.length; i++) {
    var ret = list[i](arg);
    if (ret) {
      return ret;
    }
  }
  return null;
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.source-map-support.wrapCallSite"></a>[function <span class="apidocSignatureSpan">source-map-support.</span>wrapCallSite (frame)](#apidoc.element.source-map-support.wrapCallSite)
- description and source-code
```javascript
function wrapCallSite(frame) {
  if(frame.isNative()) {
    return frame;
  }

  // Most call sites will return the source file from getFileName(), but code
  // passed to eval() ending in "//# sourceURL=..." will return the source file
  // from getScriptNameOrSourceURL() instead
  var source = frame.getFileName() || frame.getScriptNameOrSourceURL();
  if (source) {
    var line = frame.getLineNumber();
    var column = frame.getColumnNumber() - 1;

    // Fix position in Node where some (internal) code is prepended.
    // See https://github.com/evanw/node-source-map-support/issues/36
    if (line === 1 && !isInBrowser() && !frame.isEval()) {
      column -= 62;
    }

    var position = mapSourcePosition({
      source: source,
      line: line,
      column: column
    });
    frame = cloneCallSite(frame);
    frame.getFileName = function() { return position.source; };
    frame.getLineNumber = function() { return position.line; };
    frame.getColumnNumber = function() { return position.column + 1; };
    frame.getScriptNameOrSourceURL = function() { return position.source; };
    return frame;
  }

  // Code called using eval() needs special handling
  var origin = frame.isEval() && frame.getEvalOrigin();
  if (origin) {
    origin = mapEvalOrigin(origin);
    frame = cloneCallSite(frame);
    frame.getEvalOrigin = function() { return origin; };
    return frame;
  }

  // If we get here then we were unable to change the source position
  return frame;
}
```
- example usage
```shell
n/a
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
