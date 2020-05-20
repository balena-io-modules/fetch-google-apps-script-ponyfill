# fetch-google-apps-script-ponyfill

A [ponyfill](https://ponyfill.com) that makes the browser Promise-based `fetch()` function to work in Google Apps Script.
This is a fork of https://github.com/github/fetch where some parts of this implementation come from.

## Installation

```
npm install fetch-google-apps-script-ponyfill --save
```

As an alternative to using npm, you can obtain `fetch.umd.js` from the
[Releases][] section. The UMD distribution is compatible with AMD and CommonJS
module loaders, as well as loading directly into a page via `<script>` tag.

## Usage

For a more comprehensive API reference that this ponyfill supports, refer to
https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch.

### Importing

If you are using a build pipeline for your Google Apps Script code
you can import it as a module.
```javascript
import { fetch as fetchGoogleAppsScriptPonyfill } from 'fetch-google-apps-script-ponyfill';

fetchGoogleAppsScriptPonyfill(...);
````

Or when importing this as a separate script then just access the global function directly
```javascript
function getFetch() {
  return fetchGoogleAppsScriptPonyfill.fetch;
}
getFetch()(...);
```

### JSON

```javascript
fetch('/users.json')
  .then(function(response) {
    return response.json()
  }).then(function(json) {
    console.log('parsed json', json)
  }).catch(function(ex) {
    console.log('parsing failed', ex)
  })
```

### Caveats

* The Promise returned from `fetch()` **won't reject on HTTP error status**
  even if the response is an HTTP 404 or 500. Instead, it will resolve normally,
  and it will only reject on network failure or if anything prevented the
  request from completing.

* Not all Fetch standard options are supported in this polyfill. For instance,
  [`redirect`](#redirect-modes) and
  [`cache`](https://github.github.io/fetch/#caveats) directives are ignored.

#### Handling HTTP error statuses

To have `fetch` Promise reject on HTTP error statuses, i.e. on any non-2xx
status, define a custom response handler:

```javascript
function checkStatus(response) {
  if (response.status >= 200 && response.status < 300) {
    return response
  } else {
    var error = new Error(response.statusText)
    error.response = response
    throw error
  }
}

function parseJSON(response) {
  return response.json()
}

fetch('/users')
  .then(checkStatus)
  .then(parseJSON)
  .then(function(data) {
    console.log('request succeeded with JSON response', data)
  }).catch(function(error) {
    console.log('request failed', error)
  })
```
