![Minion](https://68.media.tumblr.com/avatar_c972768606a9_128.png)

**Minion**  —  _Microservice Framework for RabbitMQ Workers_

## Features

 - Easy and Minimalistic setup
 - Simple to use and configure
 - Designed to use with sync functions aswell as promises or async/await ones

## Usage

install `minion`:

```bash
npm install --save minion
```

### Single handler

Create an index.js and export a function like this:

```javascript
module.exports = (message) => {
   return 'Hello World'
}
```

Ensure that the `main` property inside `package.json` points to your microservice (which is inside `index.js` in this example case) and add a `start` script:

```json
{
  "main": "index.js",
  "scripts": {
    "start": "minion"
  }
}
```

Once this is done, start the worker:

```bash
npm start
```

### Multiple handlers

Instead of pointing to a single file on the package.json `main` property, set it to a directory, and create a file for each service you want:

```json
{
  "main": "lib",
  "scripts": {
    "start": "minion"
  }
}
```

### Configuration

You can change default worker configuration by setting properties on your handler like this:

```javascript
const handler = (message) => {
  return 'Hello World'
}

handler.noAck = true
handler.queue = 'message.example.key'

module.exports = handler
```

Check below for supported options and default values.

#### Options
- `exchangeType` - Defaults to 'topic'
- `exchangeName` - Defautls to the name of the handler function.
- `noAck` - Wheter acking is required for this worker. Defaults to false.
- `key` - Key to bind the queue to. Defaults to service file name or queue name.
- `exclusive` - Defaults to false.
- `durable` - Defaults to true.
- `deadLetterExchange` - By default all queues are created with a dead letter exchange. The name defaults to the name of the exchange following the `.dead` suffix. If you want to disable the dead letter exchange , set it as `false`.
- `schema` - If a schema is defined, the payload of the message will be validated against it. If the validation fails, a nack will be sent with `requeue=false`. If not present the payload will always be valid. The schema is expected to be a [Joi](https://github.com/hapijs/joi) schema.

### Programmatic use

You can use Minion programmatically by requiring it directly, and passing options as the second argument:

```js
const minion = require('minion')

minion((message) => {
  return 'Hello World'
}, {
  noAck: true
})
```

With async / await support

```js
const minion = require('minion')

minion(async (message) => {
  return await request('https://foo.bar.zz')
})
```

### Validation

We use [joi](https://github.com/hapijs/joi) as default for payload validation. Just set the `schema` property for the handler with a valid Joi definition and the payload will be automatically validated:

```js
const handler = (message) => {
    return true
}

handler.schema = {
    aKey: joi.string()
}
```

If you're using it programmatically:

```js
minion((message) => {
  return 'Hello World'
}, {
  schema: {
     myKey: joi.string()
  }
})
```

## Environment Configuration

The RabbitMQ connection URL is read from a `RABBIT_URL` env var, if not present it will default to: `amqp://localhost`

## Error Handling

If the handler throws an error the message will be nacked and not requeued (`{ requeue: false }`), if you want to requeue on failure
minion provider a custom error to do so

Your service:
```js
const minion = require('../lib')
const Requeue = minion.Requeue

const handler = async (message) => {
    throw new Requeue('My message')
};
```

## Testing

When calling Minion programatically you receive an instance of a function you can use to inject messages directly.
Assuming you're using [ava](https://github.com/sindresorhus/ava) for testing (as we do), you can test like this:

Your service:
```javascript
const handler = (message) => {
    return true
}
```

Your test:
```javascript
test('acks message with true', async t => {
    const service = minion(handler)
    const message = {hello: 'world'}

    const res = await service(message)
    t.true(res)
})
```
