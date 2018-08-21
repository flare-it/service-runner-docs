---
title: Augi Service Reference

language_tabs:
  - javascript

#toc_footers:
#  - <a href='#'>Sign Up for a Developer Key</a>
#  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

#includes:
#  - errors

search: true
---

# Introduction

Nuff said!

# Folder structure

> The following shows a possible folder structure

```shell

# /some-service
#   /index.js
#   /context
#     /some-context-enhancer.js
#   /handlers
#     /events
#       /some-event.js
#       /another-event.js
#     /invocations
#       /some-invocation.js
#       /another-invocation.js
#     /tasks
#       /some-task.js
#       /another-task.js
#   /models
#     /some-model.js

```

The service shown is called `some-service`. It contains the optional configuration file `index.js`. The handlers folder are the various implemented handlers. They will be explained further down. The folder `context` contains so called context enhancers. Files inside will be able to add parameters to the context object. This is useful if you need to create a client for another service like elasticsearch etc. The `models` folder contains model definitions that will be loaded on service start.

# Service components

The following section will go into detail on the previously described components.

## The configuration file


> Below is an example definition of an index.js file

```javascript

module.exports = {
  name: 'some-cool-service',  // This setting can override the actual service name
  context: {  // Custom context enhancers can be given configuration parameters which they will receive via their execution context
    foo: {    // the foo enhancer will be able to access foo under `context.config.foo`
      bar: 123
    }
  }
}

```

The configuration file is optional. It can define specific parameters.


## Context enhancers

> A sample context enhancer

```javascript

/**
 * A sample context enhancer
 *
 * @param context - The context of the handler
 */
module.exports = async (context) => {
  // do some startup stuff
};

```

> The context contains the following parameters:

```javascript
const context = {
  env: "production" // the current environment we are running in
  shortEnv: "prod", // the shortform of the current environment
  service: "some-cool-service", // the name of the current service
  config: { // optional config parameters which are provided automatically through the config file
    bar: 123
  }
};
```

A context enhancer can enhance the handlers context object. Each context enhancer gets loaded once on service startup.

If an enhancer throws an error when run the service will abort startup.



## Event handlers


> An event handler has the following signature:

```javascript

/**
 * The event handler
 *
 * @param context - The current context of the handler
 * @param payload - The payload of the event to handle
 */
module.exports = async (context, payload) => {
  // handle event
  // If error is thrown then event will be re-handled
};

```

> The context contains the following parameters:

```javascript
const context = {
  cid: 'f8682372-b20d-4c46-ab6b-d25656bf9a48', // the correlation id of the initial request to track related logs
  self: {
    name: 'some-service', // The name of the service that handles the event
    handler: 'some-handler',  // The name of the event
  },
  event: {
    id: '4943ace3-9864-44ac-a74b-4c1fee0feac1',
    from: {
      name: 'another-service',  // the name of the calling service
      handler: 'another-handler'  // The name of the handler from the calling service
    },
    timestamp: '2018-08-10T19:15:00+02:00' // An ISO8601 Timestamp string at the time of the event emission
  },
  emit: async (eventType, payload) => void, // Emit an event with a payload,
  invoke: async (serviceName, handlerName, ...params) => any, // Invoke a handler of this or another service,
  task: {
    create: async (serviceName, taskName, payload) => string,
    status: async (taskId) => { // Get the status of a task id (only if the task originated from this service)
      id: String, // The id of the task
      progress: Number, // The current task progress
      completed: Boolean  // true if the task was completed
    }
  },
  cache: {  // cache getters and setters
    global: { // Cache values are available for all other services
      get: async (cacheKey) => any,  // get a cached value or null
      set: async (cacheKey, cacheValue) => void  // set a cache value for a specific key
    },
    service: {  // cache values are only available for this service
      get: async (cacheKey) => any,  // get a cached value or null
      set: async (cacheKey, cacheValue) => void  // set a cache value for a specific key
    }
  },
  log: {
    info: (message, ...params) => void,  // an info message log
    warn:  (message, ...params) => void,  // a warning message log
    error: (message, ...params) => void // an error message log
  },
  ... models, // the models available to the service
  ... contextEnhancers // the objects defined via context enhancers
};
```

An event handler gets called when a service emits an event. Events are when something happened in the emitting service, that other services might find useful. Event names are always in the past (eg. `something-happened`, `account-created` or `team-joined`).

## Invocation handlers


> An invocation handler has the following signature:

```javascript

/**
 * The invocation handler
 *
 * @param context - The current context of the handler
 * @param paramOne - The first parameter of the invocation
 * @param paramTwo - The second parameter of the invocation
 */
module.exports = async (context, paramOne, paramTwo, ...) => {
  // handle invocation
  // If error is thrown then invocation will throw on the calling side
};

```

> The context contains the following parameters:

```javascript
const context = {
  cid: 'f8682372-b20d-4c46-ab6b-d25656bf9a48', // the correlation id of the initial request to track related logs
  self: {
    name: 'some-service', // The name of the service that handles the event
    handler: 'some-handler',  // The name of the event
  },
  invocation: {
    id: '19df39df-99ad-44ac-dc1a-4c1fee0feac1',
    from: {
      name: 'another-service',  // the name of the calling service
      handler: 'another-handler'  // The name of the handler from the calling service
    },
    timestamp: '2018-08-10T19:15:00+02:00' // An ISO8601 Timestamp string at the time of the event emission
  },
  emit: async (eventType, payload) => void, // Emit an event with a payload,
  invoke: async (serviceName, handlerName, ...params) => any, // Invoke a handler of this or another service,
  task: {
    create: async (serviceName, taskName, payload) => string,
    status: async (taskId) => { // Get the status of a task id (only if the task originated from this service)
      id: String, // The id of the task
      progress: Number, // The current task progress
      completed: Boolean  // true if the task was completed
    }
  },
  cache: {  // cache getters and setters
    global: { // Cache values are available for all other services
      get: async (cacheKey) => any,  // get a cached value or null
      set: async (cacheKey, cacheValue) => void  // set a cache value for a specific key
    },
    service: {  // cache values are only available for this service
      get: async (cacheKey) => any,  // get a cached value or null
      set: async (cacheKey, cacheValue) => void  // set a cache value for a specific key
    }
  },
  log: {
    info: (message, ...params) => void,  // an info message log
    warn:  (message, ...params) => void,  // a warning message log
    error: (message, ...params) => void // an error message log
  },
  ... models, // the models available to the service
  ... contextEnhancers // the objects defined via context enhancers
};
```

An invocation handler is the most common handler type. It should be used for direct contact with another service.

## Task handlers


> A task handler has the following signature:

```javascript
/**
 * The invocation handler
 *
 * @param context - The current context of the handler
 * @param payload - The payload of the task
 */
module.exports = async (context, payload) => {
  // handle task
  // If error is thrown then the task will be marked as failed
};
```

> The task context contains the following parameters:

```javascript
const context = {
  cid: 'f8682372-b20d-4c46-ab6b-d25656bf9a48', // the correlation id of the initial request to track related logs
  self: {
    name: 'some-service', // The name of the service that handles the event
    handler: 'some-handler',  // The name of the event
  },
  task: {
    id: '19df39df-99ad-44ac-dc1a-4c1fee0feac1',
    from: {
      name: 'another-service',  // the name of the calling service
      handler: 'another-handler'  // The name of the handler from the calling service
    },
    timestamp: '2018-08-10T19:15:00+02:00', // An ISO8601 Timestamp string at the time of the event emission
    create: async (serviceName: String, taskName: String, payload: any) => string,
    status: async (taskId) => { // Get the status of a task id (only if the task originated from this service)
      id: String, // The id of the task
      progress: Number, // The current task progress
      completed: Boolean  // true if the task was completed
    }
  },
  emit: async (eventType: String, payload: any) => void, // Emit an event with a payload,
  invoke: async (serviceName: String, handlerName: String, ...params: any[]) => any, // Invoke a handler of this or another service,
  cache: {  // cache getters and setters
    global: { // Cache values are available for all other services
      get: async (cacheKey) => any,  // get a cached value or null
      set: async (cacheKey, cacheValue) => void  // set a cache value for a specific key
    },
    service: {  // cache values are only available for this service
      get: async (cacheKey) => any,  // get a cached value or null
      set: async (cacheKey, cacheValue) => void  // set a cache value for a specific key
    }
  },
  log: {
    info: (message, ...params) => void,  // an info message log
    warn:  (message, ...params) => void,  // a warning message log
    error: (message, ...params) => void // an error message log
  },
  ... models, // the models available to the service
  ... contextEnhancers // the objects defined via context enhancers
};
```

A task handler gets called when another task asks a service to do a specific task, but it does not matter when (must not run directly, only important for one service)

## Models

# Restrictions

Some restrictions apply.

### The service name

A service name may not start with the prefix `sys-` as it is reserved for system services with possibly escalated privileges.

### Context enhancers

Context enhancers may not have the following names: `cid`, `invocation`, `task`, `event`, `invoke`, `emit`, `self`, `log`, `cache` or any registered model name.
