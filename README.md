# Gaggle [![Build Status](https://img.shields.io/circleci/project/ben-ng/gaggle/master.svg)](https://circleci.com/gh/ben-ng/gaggle/tree/master) [![Coverage Status](https://img.shields.io/coveralls/ben-ng/gaggle/master.svg)](https://coveralls.io/github/ben-ng/gaggle?branch=master) [![npm version](https://img.shields.io/npm/v/gaggle.svg)](https://www.npmjs.com/package/gaggle)

Gaggle is a [Raft](http://raft.github.io) implementation that focuses on ease of use.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Contents**

- [Quick Example](#quick-example)
- [API](#api)
  - [Gaggle](#gaggle)
    - [Creating an instance](#creating-an-instance)
    - [Appending Messages](#appending-messages)
    - [Deconstructing an instance](#deconstructing-an-instance)
    - [Event: committed](#event-committed)
    - [Event: leaderElected](#event-leaderelected)
  - [Channels](#channels)
    - [Redis](#redis)
    - [Memory](#memory)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Quick Example

```js
var gaggle = require('gaggle')
var uuid = require('uuid')
var defaults = require('lodash/defaults')
var opts = {
      channel: {
        name: 'redis'
      , redisChannel: 'foo'
      }
    , clusterSize: 3
    }

var nodeA = gaggle(defaults({id: uuid.v4()}, opts))
var nodeB = gaggle(defaults({id: uuid.v4()}, opts))
var nodeC = gaggle(defaults({id: uuid.v4()}, opts))

// Nodes will emit "committed" events whenever the cluster
// comes to consensus about an entry
nodeC.on('committed', function (data) {
  console.log(data)
})

// You can be notified when a specific message is committed
// by providing a callback
nodeC.append('mary', function () {
  console.log(',')
})

// Or, you can use promises
nodeA.append('had').then(function () {
  console.log('!')
})

// Or, you can just cross your fingers and hope that an error
// doesn't happen by neglecting the return result and callback
nodeA.append('a')

// Entry data can also be numbers, arrays, or objects
// we were just using strings here for simplicity
nodeB.append({foo: 'lamb'})

// You can specify a timeout as a second argument
nodeA.append('little', 1000)

// By default, gaggle will wait indefinitely for a message to commit
nodeC.append('a', function () {
  // I may never be called!
})

// This example prints the sentence:
//     "mary , had a little {foo: 'lamb'} !"
// in SOME order; Raft only guarantees that all nodes will commit entries in
// the same order, but nodes sent at different times may not be committed
// in the order that they were sent.
```

## API

### Gaggle

#### Creating an instance

```js
var gaggle = require('gaggle')
    // uuids are recommended, but you can use any string id
  , uuid = require('uuid')
  , g = gaggle({
      // Required settings
      id: uuid.v4()
    , clusterSize: 5
    , channel: {
        name: 'foobar'
        // ... additional Channel options specific to "foobar"
        // see Channel documentation below
      }

      // Optional settings

      // How long to wait before declaring the leader dead?
    , electionTimeout: {
        min: 300
      , max: 500
      }

      // How often should the leader send heartbeats?
    , heartbeatInterval: Joi.number().min(0).default(50)

      // Should the leader send a heartbeat if it would speed
      // up the commit of a message?
    , accelerateHeartbeats: Joi.boolean().default(false)
    })
```

#### Appending Messages

```txt
g.append(Mixed data, [Number timeout], [function(Error) callback])
```

Anything that can be serialized and deserialized as JSON is valid message data. If `callback` is not provided, a `Promise` will be returned.

```js
g.append(data, function (err) {})
g.append(data, timeout, function (err) {})

g.append(data).then()
g.append(data, timeout).then()
```

#### Deconstructing an instance

```txt
g.close([function(Error) callback])
```

When you're done, call `close` to remove event listeners and disconnect the channel.

```js
g.close(function (err) {})

g.close().then()
```

#### Event: committed

Emitted whenever an entry is committed to the node's log.

```js
g.on('committed', function (entry, index) {
  // entry => {id: 'some-uuid', term: 1, data: {foo: bar}}
  // index => 1
})
```

#### Event: leaderElected

Emitted whenever a node discovers that a new leader has been elected.

```js
g.on('leaderElected', function () {
  console.log('four! more! years!')
})
```

### Channels

#### Redis

Fast, but relies heavily on your Redis server.

```js
gaggle({
  id: uuid.v4()
, clusterSize: 5
, channel: {
    name: 'redis'
    // required, the channel to pub/sub to
  , channelName: 'foobar'
    // optional, defaults to redis's defaults
  , connectionString: 'redis://user:password@127.0.0.1:1234'
  }
})
```

#### Memory

No options, useful for testing.

```js
gaggle({
  id: uuid.v4()
, clusterSize: 5
, channel: {
    name: 'memory'
  }
})
```

## License

Copyright (c) 2015 Ben Ng <me@benng.me>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
