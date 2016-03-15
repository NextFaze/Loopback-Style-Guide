This is very work-in-progress but, in general, a central point to keep track
of Loopback's various idiosyncrasies and define some best practices when
working with Loopback.

### Project Structure

Prefer more, smaller files.

*Why?:* Mostly common sense.

*Why?:* It's far easier to navigate and maintain and results in less git conflicts when
compared to large, monolithic files.

```javascript
/* Avoid */
// roles.js
module.exports = function(app) {
  function roleResolverA() {
    // Role resolver definition for Role A
  }

  function roleResolverB() {
    // Role resolver definition for Role B
  }

  function roleResolverC() {
    // Role resolver definition for Role C
  }

  // etc...
}
```

```javascript
/* Prefer */
// roles.js
module.exports = function(app) {
  require('roles/roleA')(app);
  require('roles/roleB')(app);
  require('roles/roleC')(app);
}

// roles/roleA.js
module.exports = function(app) {
  // Role resolver definition for Role A
}

// roles/roleB.js
// etc...
```

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


### Promises

Prefer promises over callbacks

*Why?*: Promises generally lead to neater, more readable code

*Why?*: It's easier to separate out components in a promise chain into separate functions

**NOTE** Not all built in Loopback methods are promisified, and some have broken promises: (https://github.com/strongloop/loopback/issues/418#issue-38984704)[https://github.com/strongloop/loopback/issues/418#issue-38984704].
In cases where promises are not supported, prefer to add custom promises over using callbacks for the reasons stated above

```javascript
/* Avoid */
modelInstance.save(function(err, result) {
  if(err) {
    throw err;
  }
  cb(null, result);
})
```

```javascript
/* Avoid */
modelInstance.save()
.then(function(result) {
  cb(null, result);
})
.catch(function(err) {
  throw err;
})
```

Prefer named functions over long anonymous functions in promise chains

*Why?*: It's much easier to read and allows other developers to quickly scan
and get an overall sense of what is happening in the promise chain

```javascript
/* Avoid */
modelInstance.save()
.then(function(result) {
  // Put left foot in
  return somePromise()
})
.then(function(result) {
  // Put left foot out
  return somePromise()
})
.then(function(result) {
  // Put left foot in
  return somePromise()
})
.then(function(result) {
  // Shake it all about
  return somePromise()
})
.catch(function(err) {
  throw err;
})
```

```javascript
/* Prefer */
modelInstance.save()
.then(putLeftFootIn)
.then(putLeftFootOut)
.then(putLeftFootIn)
.then(shakeItAllAbout)
.catch(handleError);

function putLeftFootIn() {
  // Put left foot in
  return somePromise;
}
// ... etc
```
