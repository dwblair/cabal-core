# cabal-core

Core database, replication, swarming, and chat APIs for cabal.

## Usage

    npm install cabal-core

## API

> var Cabal = require('cabal-node')

### var cabal = Cabal([storage][, key][, opts])

Create a cabal p2p database using storage `storage`, which must be either a
string (filepath to directory on disk) or an instance of
[random-access-storage](https://github.com/random-access-storage/).

`key` is a hex string of the key, without any prefixes (like `cabal://`).

If this is a new database, `key` can be omitted and will be generated.

You can pass `opts.db` as a levelup or leveldown instance to use persistent
storage for indexing instead of using memory. For example:

``` js
var level = require('level')
var cabal = Cabal(storage, key, { db: level('/tmp/bot.db') })
```

### cabal.getLocalKey(cb)

Returns the local user's key (as a hex string).

### var ds = cabal.replicate()

Creates a new, live replication stream. This duplex stream can be piped into any
transport expressed as a node stream (tcp, websockets, udp, utp, etc).

### cabal.ready(cb)

Call `cb()` when the underlying indexes are caught up.

### Channels

#### cabal.channels.get(function (error, channels) {})

Retrieve a list of all channel names that exist in this cabal.

#### cabal.channels.events.on('add', function (channel) {})

Emitted when a new channel is added to the cabal.

### Messages

#### var rs = cabal.messages.read(channel, opts)

Returns a readable stream of messages (most recent first) from a channel.

Pass `opts.limit` to set a maximum number of messages to read.

#### cabal.messages.events.on('message', fn)

Calls `fn` with every new message that arrives, regardless of channel.

#### cabal.messages.events.on(channel, fn)

Calls `fn` with every new message that arrives in `channel`.

### Network

> var swarm = require('cabal-core/swarm')

#### cabal.swarm(cb)

Joins the P2P swarm for a cabal. This seeks out peers who are also part of this cabal by various means (internet, local network), connects to them, and replicates cabal messages between them.

The returned object is an instance of [discovery-swarm](https://github.com/mafintosh/discovery-swarm).

#### cabal.on('peer-added', function (key) {})

Emitted when you connect to a peer. `key` is a hex string of their public key.

#### cabal.on('peer-dropped', function (key) {})

Emitted when you lose a connection to a peer. `key` is a hex string of their
public key.

### Publishing

#### cabal.publish(message, opts, cb)

Publish `message` to your feed. `message` must have a `type` field set. If not,
it defaults to `chat/text`. In general, a message is formatted as

```js
{
  type: 'chat/text',
  content: {
    text: 'hello world',
    channel: 'cabal-dev'
  }
}
```

A `timestamp` field is set automatically with the current system time.

`type` is an unrestricted field: you can make up new message types and clients
will happily ignore them until someone implements support for them. Well
documented types include

#### chat/text

```js
{
  type: 'chat/text',
  content: {
    text: 'whatever the user wants to say',
    channel: 'some channel name. if it didnt exist before, it does now!'
  }
}
```

## License

AGPLv3
