
## Promises and Deferred Actions

The [Q](https://github.com/kriskowal/q) library is helpful in working with Promises.

```bash
bower install q --save
```


Promises (hereafter represented by 'Q') uses deferred objects to control the order of execution in our asynchronous code. This is advantageous over callback functions, which have to be executed immediately once data is returned from an asynchronous (e.g. Ajax) operation. With deferred objects, we can let any other code that is listening to the operation know when it either succeeded or failed. The steps for using Promises:

##### 1. Define a function that will execute the XHR, and make a variable to hold a deferred object.

```js
function getSongs() {
  var deferred = Q.defer();
}
```

##### 2. This function will then return a Promise.

```js
function getSongs() {
  var deferred = Q.defer();

  return deferred.promise;
}
```

##### 3. Get access to the data returned by the XHR in other places in your code by **resolving** and **rejecting** the Promise:

 - Resolve is used to broadcast that the action succeeded.
 - Reject is used to broadcast that the action failed.

```js
function getSongs() {
  var deferred = Q.defer();

  $.ajax({ url: "songs.json" })
    // XHR was successful
    .done(function(json_data) {
      // Now we can resolve the promise and send the data
      deferred.resolve(json_data);
    })

    // XHR failed for some reason
    .fail(function(xhr, status, error) {
      // Since the call failed, we have to reject the promise
      deferred.reject(error);
    });

  return deferred.promise;
}
```

##### 4. When a promise is resolved, you can specify which function will get executed in the `then()` method.

```js
//Original call site of the promise
getSongs()
  // Then gets executed when promise is resolved
  .then(function(json_data) {
    console.log("API call successful and responded with", json_data);
  });
```

##### 5. When a promise is rejected, you can specify which function will get executed in the `fail()` method.

```js
//Original call site of the promise
getSongs()
  // Then gets executed when promise is resolved
  .then(function(json_data) {
    console.log("API call successful and responded with", json_data);
  })
  // Fail gets executed when promise is rejected
  .fail(function(error) {
    console.log("API call failed with error", error);
  });
```

### Chaining Promises

Promises are especially helpful when needing to perform synchronous operations from the successive data of multiple asychronous operations.

```js
// This function does one thing, and returns a promise
var firstXHR = function() {
  var deferred = Q.defer();

  $.ajax({
    url: "https://nss-demo-instructor.firebaseio.com/songs.json"
  }).done(function(data) {
    deferred.resolve(data);
  }).fail(function(xhr, status, error) {
    deferred.reject(error);
  });

  return deferred.promise;
};

// This function does one thing, and returns a promise
var secondXHR = function(result_of_firstXHR) {
  var deferred = Q.defer();

  $.ajax({
    url: "https://nss-demo-instructor.firebaseio.com/more-songs-info.json",
    data: result_of_firstXHR
  }).done(function(data) {
    deferred.resolve(data);
  }).fail(function(xhr, status, error) {
    deferred.reject(error);
  });

  return deferred.promise;
};

// This function does one thing, and returns a promise
var thirdXHR = function(result_of_secondXHR) {
  var deferred = Q.defer();

  $.ajax({
    url: "https://nss-demo-instructor.firebaseio.com/song-details.json",
    data: result_of_secondXHR
  }).done(function(data) {
    deferred.resolve(data);
  }).fail(function(xhr, status, error) {
    deferred.reject(error);
  });

  return deferred.promise;
};

/*
  Now we use those Promises to describe the order of execution, 
  and how data flows between each one.
 */
firstXHR()
  .then(function(data1) {
    return secondXHR(data1);
  })
  .then(function(data2) {
    return thirdXHR(data2);
  })
  .done();
```

### Checking the State of a Promise

Promises also maintain their state. If you store a Promise object in a variable, you can then check the state of that Promise at any other time in your code. Think of a banking application where you use an XHR to get the current balance of an account, and all transactions in the last 30 days.

By using a promise to broadcast, and maintain, the state of that operation, you can then check it later 

```js
// This stores the Promise object
 var promiseStorage = promise();

// You can then handle success/failure of the promise
 promiseStorage.then(function(results) {
   console.log("results",results);
 }).fail(function(error) {
   console.log("error", error);
 });

 $("#clearFilter").click(function() {
   // This does not execute the XHR function again, but simply
   // checks the state of the Promise and acts accordingly
   promiseStorage.then(function(results) {
     console.log("results",results);
   }).fail(function(error) {
      console.log("error", error);
   });
 });
```

###Native Promises in ES6
ES6 features a native system for promises that is supported in most browsers as of October 2015. Here's an example borrowed from http://www.html5rocks.com/en/tutorials/es6/promises/. 
```
var promise = new Promise(function(resolve, reject) {
  // do a thing, possibly async, then…
  if (/* everything turned out fine */) {
    resolve("Stuff worked!");
  }
  else {
    reject(Error("It broke"));
  }
});
```
The Promise is stored in the promise variable.
```
promise.then(function(result) {
  console.log(result); // "Stuff worked!"
}, function(err) {
  console.log(err); // Error: "It broke"
});
```
###Links
MDN Promises: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise

Q API Reference: https://github.com/kriskowal/q/wiki/API-Reference
