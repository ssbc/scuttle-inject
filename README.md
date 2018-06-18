# Scuttle-inject

Takes a particular sort of nested tree of methods and returns a version of that tree which has a scuttlebutt server injected into each method.

This pattern means you only need to inject a scuttlebot once, makes it easier to test your methods, and handles being able to inject different sorts of server for you.

## Example usage

```js
// methods.js
module.exports = {
  async: {
    publishPoll: function (server) {
      return function (opts, cb) {
        // check the opts before publishing
        const cleanOpts = clean(opts) 

        server.publish(cleanOpts, cb)
      }
    },
    // getPoll: (key, cb) => {}
  },
  pull: {
    myPolls: function (server) {
      return function (opts) {
        const defaultQuery = { ... }
        const query = Object.assign({}, defaultQuery, opts)

        return server.query.read(opts)
      }
    }
    // openPolls: (opts) => {},
    // closedPolls: (opts) => {}
  }
}
```

Injecting server once (somehwhere high-level):
```js
const inject = require('scuttle-inject')
cosnt methods = require('./methods')

const scuttle = inject(methods, server)
// assume you're in a context where you have a server
```

Using the scuttle helper:
```
const opts = { ... }
scuttle.async.publishPoll(opts, cb)
```

## API

`inject(server, methods, pluginDeps)`

- `server` - a `scuttlebot` server, an `ssb-client` connection to a server, or an observeable which will resolve into a server connection (such as Patchcore's `sbot.obs.connection`)

- `methods` - an Object nested at least 2 levels deep, where there must be one layer which specifies the `type` of method. Valid types are : `sync`, `async`, `pull`, `obs`

- `pluginDeps` (optional) - an Array of plugins apis the scuttlebot must have for your methods to work. e.g. `['query']` will check that `sbot.query` has methods which are accessible.
