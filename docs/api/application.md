---
outline: deep
---

# Application

<Badges>

[![npm version](https://img.shields.io/npm/v/@feathersjs/authentication-client.svg?style=flat-square)](https://www.npmjs.com/package/@feathersjs/feathers)
[![Changelog](https://img.shields.io/badge/changelog-.md-blue.svg?style=flat-square)](https://github.com/feathersjs/feathers/blob/dove/packages/feathers/CHANGELOG.md)

</Badges>

```
npm install @feathersjs/feathers --save
```

The core `@feathersjs/feathers` module provides the ability to initialize a new Feathers application instance. It works in Node, React Native and the browser (see the [client](./client.md) chapter for more information). Each instance allows for registration and retrieval of [services](./services.md), [hooks](./hooks.md), plugin configuration, and getting and setting configuration options. An initialized Feathers application is referred to as the **app object**.


<LanguageBlock global-id="ts">

<<< @/examples/ts/application.ts

</LanguageBlock>
<LanguageBlock global-id="js">

<<< @/examples/js/application.js

</LanguageBlock>


## .use(path, service [, options])

`app.use(path, service [, options]) -> app` allows registering a [service object](./services.md) on a given `path`.


<LanguageBlock global-id="ts">

<<< @/examples/ts/application-use.ts

</LanguageBlock>
<LanguageBlock global-id="js">

<<< @/examples/js/application-use.js

</LanguageBlock>

<BlockQuote type="note">

`path` can be `/` to register a service at the root level.

</BlockQuote>

`options` can contain the following additional options for the service:

- `methods` (default: `['find', 'get', 'create', 'patch', 'update','remove']`) - A list of official and [custom service methods](services.md#custom-methods) exposed by this service. When using this option **all** method names that should be available externally must be passed otherwise. The methods passed will automatically be available for use with [hooks](./hooks).
- `events` - A list of [public custom events sent by this service](./events.md#custom-events)

<LanguageBlock global-id="ts">

<<< @/examples/ts/application-use-options.ts

</LanguageBlock>
<LanguageBlock global-id="js">

<<< @/examples/js/application-use-options.js

</LanguageBlock>

## .service(path)

`app.service(path) -> service` returns the [service object](./services.md) for the given path. Feathers internally creates a new object from each registered service. This means that the object returned by `app.service(path)` will provide the same methods and functionality as your original service object but also functionality added by Feathers and its plugins like [service events](./events.md) and [additional methods](./services.md#feathers-functionality).

<LanguageBlock global-id="ts">

```ts
const messageService = app.service('messages');

const message = await messageService.get('test');

console.log(message);

messageService.on('created', (message: Message) => {
  console.log('Created a todo')
})
```

</LanguageBlock>
<LanguageBlock global-id="js">

```js
const messageService = app.service('messages');

const message = await messageService.get('test');

console.log(message);

messageService.on('created', (message) => {
  console.log('Created a todo')
})
```

</LanguageBlock>


## .hooks(hooks)

`app.hooks(hooks) -> app` allows registration of application-level hooks. For more information see the [application hooks section in the hooks chapter](./hooks.md#application-hooks).

## .publish([event, ] publisher)

`app.publish([event, ] publisher) -> app` registers a global event publisher. For more information see the [channels publishing](./channels.md#publishing) chapter.

## .configure(callback)

`app.configure(callback) -> app` runs a `callback` function that gets passed the application object. It is used to initialize plugins and can be used to separate your application into different files.

<LanguageBlock global-id="ts">

```ts
const setupService = (app: Application) => {
  app.use('/todos', todoService);
}

app.configure(setupService);
```

</LanguageBlock>
<LanguageBlock global-id="js">

```js
const setupService = (app) => {
  app.use('/todos', todoService);
}

app.configure(setupService);
```

</LanguageBlock>


## .setup([server])

`app.setup([server]) -> Promise<app>` is used to initialize all services by calling each [services .setup(app, path)](services.md#setupapp-path) method (if available).
It will also use the `server` instance passed (e.g. through `http.createServer`) to set up SocketIO (if enabled) and any other provider that might require the server instance. You can register [application hooks](./hooks.md#application-hooks) on setup to e.g. set up database connections and other things required to be initialized on startup in a certain order.

Normally `app.setup` will be called automatically when starting the application via [app.listen([port])](#listen-port) but there are cases (like in tests) when it can be called explicitly.

## .teardown([server])

`app.teardown([server]) -> Promise<app>` can be called to gracefully shut down the application. When the app has been set up with a server (e.g. by calling `app.listen()`) the server will be closed automatically when calling `app.teardown()`. You can also register [application hooks](./hooks.md#application-hooks) on teardown to e.g. close database connection etc.

## .listen(port)

`app.listen([port]) -> Promise<HTTPServer>` starts the application on the given port. It will set up all configured transports (if any) and then run [app.setup(server)](#setup-server) with the server object and then return the server object.

`listen` will only be available if a server side transport (REST or websocket) has been configured.

## .set(name, value)

`app.set(name, value) -> app` assigns setting `name` to `value`.

## .get(name)

`app.get(name) -> value` retrieves the setting `name`.

<LanguageBlock global-id="ts">

```ts
import { feathers } from '@feathersjs/feathers'

type ServiceTypes = {
  // Add services path to type mapping here
}

// app.get and app.set can be typed when initializing the app
type Configuration = {
  port: number
}

const app = feathers<ServiceTypes, Configuration>()

app.set('port', 3030)

app.listen(app.get('port'))
```

</LanguageBlock>
<LanguageBlock global-id="js">

```ts
import { feathers } from '@feathersjs/feathers'

const app = feathers()

app.set('port', 3030)

app.listen(app.get('port'))
```

</LanguageBlock>

<BlockQuote type="note">

On the server, those settings are usually initialized using [@feathersjs/configuration](configuration.md).

</BlockQuote>


## .on(eventname, listener)

Provided by the core [NodeJS EventEmitter .on](https://nodejs.org/api/events.html#events_emitter_on_eventname_listener). Registers a `listener` method (`function(data) {}`) for the given `eventname`.

```js
app.on('login', user => console.log('Logged in', user));
```

## .emit(eventname, data)

Provided by the core [NodeJS EventEmitter .emit](https://nodejs.org/api/events.html#events_emitter_emit_eventname_args). Emits the event `eventname` to all event listeners.

```js
app.emit('myevent', {
  message: 'Something happened'
});

app.on('myevent', data => console.log('myevent happened', data));
```

<BlockQuote type="warning">

`app` can not receive or send events to or from clients. A [custom service](services.md) should be used for those use cases.

</BlockQuote>

## .removeListener(eventname)

Provided by the core [NodeJS EventEmitter .removeListener](https://nodejs.org/api/events.html#events_emitter_removelistener_eventname_listener). Removes all or the given listener for `eventname`.

## .mixins

`app.mixins` contains a list of service mixins. A mixin is a callback (`(service, path, options) => {}`) that gets run for every service that is being registered. Adding your own mixins allows to add functionality to every registered service.

```js
// Mixins have to be added before registering any services
app.mixins.push((service, path) => {
  service.sayHello = function() {
    return `Hello from service at '${path}'`
  }
})

app.use('/todos', {
  async get(id) {
    return { id }
  }
})

app.service('todos').sayHello()
// -> Hello from service at 'todos'
```

## .services

`app.services` contains an object of all [services](./services.md) keyed by the path they have been registered via `app.use(path, service)`. This allows to return a list of all available service names:

```js
const servicePaths = Object.keys(app.services)

servicePaths.forEach(path => {
  const service = app.service(path)
})
```

<BlockQuote type="danger">

To retrieve services, the [app.service(path)](#service-path) method should be used.

Not `app.services[path]` directly.

</BlockQuote>

A Feathers [client](client.md) does not know anything about the server it is connected to. This means that `app.services` will _not_ automatically contain all services available on the server. Instead, the server has to provide the list of its services, e.g. through a [custom service](./services.md):

```js
app.use('/info', {
  async find() {
    return {
      services: Object.keys(app.services)
    }
  }
});
```

## .defaultService

`app.defaultService` can be a function that returns an instance of a new standard service for `app.service(path)` if there isn't one registered yet. By default it throws a `NotFound` error when you are trying to access a service that doesn't exist.

```js
import { MemoryService } from '@feathersjs/memory'

// For every `path` that doesn't have a service automatically return a new in-memory service
app.defaultService = function(path) {
  return new MemoryService()
}
```

This is used by the [client transport adapters](./client.md) to automatically register client side services that talk to a Feathers server.

## .lookup

`app.lookup(path)` allows to look up a full path and will return the `data` (route parameters) and `service` on the server:

```js
const { data, service } = app.lookup('messages/4321');

// service -> app.service('messages')
// data -> { __id: '4321' }
```