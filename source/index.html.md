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
#   /helpers
#     some-helper.js
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
  },
  config: { // config which is available to all handler (invocations, tasks and events)
    threshold: 3,
    maxSessions: 5
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
  shortEnv: "prod", // the short form of the current environment
  service: "some-cool-service", // the name of the current service
  config: { // optional config parameters which are provided automatically through the config file
    bar: 123
  }
};
```

A context enhancer can enhance the handlers context object. Each context enhancer gets loaded once on service startup.

If an enhancer throws an error when run the service will abort startup.

## Helper functions

> A sample helper function (this can be anything)

```javascript
// add-values.js - will be available as "context.helpers.addValues(a, b)"

const add = (a, b) => {
  return a + b;
};

module.exports = add;

```

A helper function can have any form you would like. File names will be camel cased. This means that the file name `foo-bar.js` will result in the key `fooBar`.

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
  ... contextEnhancers, // the objects defined via context enhancers
  config: {
    // all available config constants from the service configuration
  },
  helpers: {
    // all helper functions loaded from the helpers folder
  }
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
  ... contextEnhancers, // the objects defined via context enhancers
  config: {
    // all available config constants from the service configuration
  },
  helpers: {
    // all helper functions loaded from the helpers folder
  }
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
  emit: async (eventType: String, payload: any) => void, // Emit an event with a payload
  invoke: async (serviceName: String, handlerName: String, ...params: any[]) => any, // Invoke a handler of this or another service
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
  ... contextEnhancers, // the objects defined via context enhancers
  config: {
    // all available config constants from the service configuration
  },
  helpers: {
    // all helper functions loaded from the helpers folder
  }
};
```

A task handler gets called when another task asks a service to do a specific task, but it does not matter when (must not run directly, only important for one service)

## Models

> A model represents a persistable, document based storable object.
> Under the hood, mongoose is used so you can define a model like you would normally do when defining a mongoose schema.

```javascript

const modelDefinition = {
  firstName: {
    type: String,
    required: true
  },
  cool: {
    type: Boolean,
    required: true,
    default: true
  }
};

module.exports = modelDefinition;

```

Define the models you need as separate files inside the models directory.
They become available in all handlers (invocations, events and tasks) as their camel cased name under the context.

Example:

The model definition file `models/user-account.js` will become available as `context.userAccount`;

### Model methods

> A model contains the default CRUD methods plus count and distinct.

```javascript

// some handler, we will use the model defined above

const handler = async (context) => {

// count all matching query
const accountCount = await context.userAccount.count({cool: true});

// distinct fields matching query
const distinctFirstNames = await context.userAccount.distinct('firstName', {cool: false});
// returns an array of strings of all account first names where cool is false.

// Create one model.
const newAccount = await context.userAccount.createOne({firstName: 'John'});
/*
const newAccount = {
    id: 'uuid',
    firstName: 'John',
    createdAt: 'ISO Timestamp',
    updatedAt: 'ISO Timestamp'
  };
*/

// Create one model.
const newAccounts = await context.userAccount.createMany([{firstName: 'Peter'}]);
/*
const newAccounts = [
    {
      id: 'uuid',
      firstName: 'Peter',
      createdAt: 'ISO Timestamp',
      updatedAt: 'ISO Timestamp'
    }
  ]
*/

// find one
const someAccount = await context.userAccount.findOne({firstName: 'Hans'});
// or find by id
const anotherAccount = await context.userAccount.findOneById('some-uuid');
// or find many
const moreAccounts = await context.userAccount.findMany({cool: false});

// update one account
const updatedAccount = await context.userAccount.updateOne({firstName: 'Hans'}, {cool: true});
// or update one account by id
const anotherUpdatedAccount = await context.userAccount.updateOneById('some-uuid', {firstName: 'Anna'});
// or update many
const moreUpdatedAccounts = await context.userAccount.updateMany({cool: false}, {cool: true});

// remove one account
const removedAccount = await context.userAccount.removeOne({firstName: 'Hans'});
// or remove one account by id
const anotherRemovedAccount = await context.userAccount.removeOneById('some-uuid');
// or remove many
const moreRemovedAccounts = await context.userAccount.removeMany({cool: true});

};

module.exports = handler;

```

# Testing

Testing was designed to be super simple and independent of other services. As a test framework jest is supposed to be used.

> Lets imagine we have an invocation handler file `create-account.js`.

```javascript
// create-account.js

/**
 * This handler will create a new user account.
 *
 * @param context <> - The default invocation context
 * @param name <String> - The name of the new account
 *
 * @returns the new account
 */
const createAccount = async (context, firstName) => {
  let newAccount;

  try {
    newAccount = await context.foo.createOne({ name });
  } catch (error) {
    context.log.warn('Failed to create account', error.message);
    throw Error('Failed to create account');
  }

  await context.emit('account-created', newAccount);

  return newAccount;
};

module.exports = createAccount;
```

> To test it we will create a new file next to it called `create-account.test.js`

```javascript
// create-account.test.js
const { handler, contextBuilder, db } = requireHandler(__dirname, './create-account');

describe('create-account', () => {

  it('account creation should return account object with correct properties', async () => {
    // Arrange
    const context = await contextBuilder();
    const firstName = 'Peter';

    // Act
    const account = await handler(context, firstName);

    // Assert
    expect(account).not.toBeNull();
    expect(account.firstName).toBe(firstName);
    expect(account.cool).toBe(true);
  });

  it('account creation should emit account-created event', async () => {
    // Arrange
    const context = await contextBuilder();
    const firstName = 'Peter';

    // Act
    const account = await handler(context, firstName);

    // Assert
    // This uses mocks from jest. They are accessible through context.mocks.emit('event'), context.mocks.invoke('service', 'handler') and context.mocks.task.create('service', 'task-handler')
    expect(context.mocks.emit('account-created')).toHaveBeenCalledWith(account);
  });

  it('Lets imagine that our service calls another service and relies on a value', async () => {
    // Arrange
    const context = await contextBuilder();
    const firstName = 'Peter';

    // the following line would mock the return value of the in-handler invoke call to the handler "some-handler" of "some-service" to return {id: '123', allGood: true}
    // check https://jestjs.io/docs/en/mock-function-api.html to get a full list of values
    context.mocks.invoke('some-service', 'some-handler').mockReturnValue({id: '123', allGood: true});

    // Act
    await handler(context, firstName);

    // Assert
    // This uses mocks from jest. They are accessible through context.mocks.emit('event'), context.mocks.invoke('service', 'handler') and context.mocks.task.create('service', 'task-handler')
    expect(context.mocks.invoke('some-service', 'some-handler')).toHaveBeenCalledTimes(1);
    // or simply
    expect(context.mocks.invoke('some-service', 'some-handler')).toHaveBeenCalled();
  });

});

```

For this to work we will need to install the flare rest plugin and add a jest plugin definition.

`yarn add -D jest-plugins jest-plugin-flare`

You will also need to create a file called `jest-plugins.js` in the root of your service.

```javascript
/* eslint-disable import/no-extraneous-dependencies */
// jest-plugins.js
require('jest-plugins')([
  'jest-plugin-flare',
]);
```

> As well as modifing the `package.json` to include the following block

```cson
{
  # ... rest of package.json
  "jest": {
    "setupTestFrameworkScriptFile": "./jest-plugins.js"
  }
}
```

# Restrictions

Some restrictions apply.

### The service name

A service name may not start with the prefix `sys-` as it is reserved for system services with possibly escalated privileges.

### Context enhancers

Context enhancers may not have the following names: `cid`, `helpers`, `config`, `invocation`, `task`, `event`, `invoke`, `emit`, `self`, `log`, `cache` or any registered model name or context enhancer name.
