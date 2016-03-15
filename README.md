This is very work-in-progress but, in general, a central point to keep track
of Loopback's various idiosyncrasies and define some best practices when
working with Loopback.

### Model Definitions
Keep custom model methods at the top of the file and refer to named functions
defined later, rather than anonymous functions.

*Why?:* It keeps all model definitions together in a neat list

*Why?:* It makes it easy to see what methods can be called on a model/instance
and what observers/remote hooks have been added to the model.

*Why?:* It also encourages function re-use when defining multiple similar remote hooks.

```javascript
/* Avoid */
Model.beforeRemote('find', function() {
  // Add some mandatory filters
});

Model.prototype.createClient = function() {
  // Custom instance method
};

Model.listUserClients = function() {
  // Custom static method
}
```

```javascript
/* Recommended */
Model.beforeRemote('find', addFilters);
Model.prototype.createClient = createClient;
Model.listUserClients = listUserClients;

function addFilters() {

}

function createClient() {

}

function listUserClients() {

}
```


### Observers and Remote Hooks
Prefer `Model.beforeRemote` over `Model.observe` unless you are sure that you
always want the functionality to trigger.

*Why?*: `beforeRemote` will only trigger if a method is used from an API endpoint.

*Why?*: Methods are often called from within server-side model definitions and
API specific triggers often want to be avoided.

```javascript
/* Avoid */
Model.observe('after save', addClientId);

function addClientId() {
  // Trying to look for an access token here will break server-side
}
```

```javascript
/* Recommended */
Model.beforeRemote('create', addClientId);
Model.beforeRemote('updateAttributes', addClientId);

function addClientId() {
  // We can look for an access token because this only triggers for REST
}
```


### Debugging

Prefer the `debug` module over `console.log`

*Why?*: It's easy to forget to remove `console.log`s and commit them to a project

*Why?*: It allows for filtering of specific debug information that you want to see

*Why?*: Excessive `console.logs` can flood the app output and make debugging difficult

```javascript
/* Avoid */
module.exports = function(MyModel) {
  console.log('Model has been loaded')
}
```

```javascript
/* Recommended */
var debug = require('debug')('MyModel');

module.exports = function(MyModel) {
  debug('Model has been loaded');
}
// from terminal:
// $ DEBUG=MyModel node .
```

Use modular debug names separated with a colon

*Why?*: It allows broad stroke debugging using *

```javascript
/* Recommended */

// ModelA.js
var debug = require('debug')('ModelA');

// ModelB.js
var debug = require('debug')('ModelA');

// from terminal:
// $ DEBUG=ModelA,ModelB node .
```

```javascript
/* Recommended */

// ModelA.js
var debug = require('debug')('models:custom:ModelA');

// ModelB.js
var debug = require('debug')('models:custom:ModelB');

// from terminal:
// $ DEBUG=models:custom:* node .
```

### Remote methods

Be careful with remote hook observers on certain properties. Instance methods
may require `prototype.{method}` in the observer:

```javascript
/* Will NOT work */
Model.beforeRemote('updateAttributes', interceptUpdate);
```

```javascript
/* WILL work */
Model.beforeRemote('prototype.updateAttributes', interceptUpdate);
```
