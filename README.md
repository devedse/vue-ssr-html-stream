# vue-ssr-html-stream

> Transform stream to simplify Vue SSR streaming

## Usage

``` js
const HTMLStream = require('vue-ssr-html-stream')

const htmlStream = new HTMLStream({
  template, // string
  context // ?Object
})

// pipe it
renderStream
  .pipe(htmlStream)
  .pipe(responseStream)
```

- The `template` option is a string of the HTML page template. It must contain the exact string `<!-- APP -->` which serves as the outlet for your app's server-rendered markup.

- The `context` option should be the same context object passed to `bundleRenderer.renderToStream()`. The transform will check for a few special properties on the context when the source render stream starts emitting data:

  - `context.head`: any head markup that should be injected into the head of the page.

  - `context.styles`: any inline CSS that should be injected into the head of the page. Note that `vue-loader` 10.2.0+ (which uses `vue-style-loader` 2.0) will automatically populate this property with styles used in rendered components.

  - `context.state`: initial Vuex store state that should be inlined in the page as `window.__INITIAL_STATE__`. The inlined JSON is automatically sanitized with [serialize-javascript](https://github.com/yahoo/serialize-javascript).

### `beforeStart` event

The stream emits a special event: `beforeStart`. An example use case would be generating `context.head` using info injected by [vue-meta](https://github.com/declandewet/vue-meta):

``` js
htmlStream.on('beforeStart', () => {
  const meta = context.meta.inject()
  context.head = meta.title.text()
})
```

### Example usage with Express

``` js
const HTMLStream = require('vue-ssr-html-stream')
const template = require('fs').readFileSync('./index.html', 'utf-8')
const renderer = require('vue-server-renderer').createBundleRenderer(bundleCode)

app.get('*', (req, res) => {
  const context = { url: req.url }

  renderer.renderToStream(context)
    .on('error', err => {
      // handle render stream error before piping to the transform
    })
    .pipe(new HTMLStream({ context, template }))
    .pipe(res)
})
```