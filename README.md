# Summary

This library helps to wrap up the detection of a minimum version for many browsers. The minimum requirements can be specified using a simple JSON object. It is essentially a wrapper around the [useragent](https://www.npmjs.com/package/useragent) library. It provides a nice compact format to describe supported browsers and versions.

This approach is doing server side detection using the browser agent provided with the request. This is useful in specific situations, but is possibly less general than a client side approach to browser detection.

The logic is supplied as an express style middleware function. It uses the descriptions passed in, looks at the result from `useragent`, then applies the result to `req.isBrowserSupported`.


# Installation

There's nothing special here, just use the regular npm command:

`npm install --save supported-browsers`


# Example

The input to the library is an object describing the browsers and the versions to be supported. Each browser name must map to the `family` calculated by the `useragent` library. The `semver` version is then compared to the `semver` returned by `useragent`.

It is also a convenient place to attach properties you might like to use in your 'unsupported browser' page. For example, `hidden` might be set to show you don't want to include it when displaying it to users in a template.

### Defining a strict list of supported browsers

The default use case is to define a strict list of browser families and supported versions.

```javascript
const browsersICanSupport = {
  Chrome: {
    semver: '>40',
    display: 'Chrome - Version 40 and above',
    download: 'https://www.google.com/chrome/'
  },
  // Support for system tests
  PhantomJS: {
    semver: '~2.1',
    hidden: true
  }
};

const supportedBrowsers = require('supported-browsers')(browsersICanSupport);

function showUnsupportedPage(req, res, next) {
  if (req.isBrowserSupported) {   // flag set by the supported-browsers library as middleware
    next();
  } else {
    if (req.useragent.family === 'Other') {
      console.warn('Unsupported browser: ' + req.useragent.source);
    } else {
      console.warn('Unsupported browser: ' + req.useragent.family + ' ' + req.useragent.major + '.' + req.useragent.minor);
    }
    res.render('unsupported', {
      browsers: browsers,
      current: req.useragent.family + ' ' + req.useragent.major + '.' + req.useragent.minor
    });
  }
};

var app = express();
app.use(supportedBrowsers);
app.use(showUnsupportedPage);
```
### Defining a relaxed list of supported browsers

The above example **only supports Chrome >40 and PhantomJS ~2.1**. Requests from clients whose browser families are undefined will fail as they are unsupported. To get around this, just specify the `passOtherBrowsers` option when instantiating the `supportedBrowsers` middleware.

Following from the above example, you want:

```javascript
const browsersICanSupport = {
  Chrome: {
    semver: '>40',
    display: 'Chrome - Version 40 and above',
    download: 'https://www.google.com/chrome/'
  },
  // Support for system tests
  PhantomJS: {
    semver: '~2.1',
    hidden: true
  }
};
const supportedBrowsersOptions = {
  passOtherBrowsers: true
}

const supportedBrowsers = require('supported-browsers')(browsersICanSupport, supportedBrowsersOptions);
```
This option will redirect older versions of Chrome and non matching versions of PhantomJS to the unsupported page, while all other browsers will flow on to the next middleware handler.

A more realistic use case for this option is to define which versions of IE are supported. For example, to specify support for Internet Explorer 10 or above, and all other clients:

```javascript
const browsersICanSupport = {
  IE: {
    semver: '>10'
  }
};
const supportedBrowsersOptions = {
  passOtherBrowsers: true
}

const supportedBrowsers = require('supported-browsers')(browsersICanSupport, supportedBrowsersOptions);
```

# License

ISC
