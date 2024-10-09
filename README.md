# async-queue

Extended Promise with resolve(), reject(), abort() methods, timeout, and much more

* Simple to use
* resolve(), reject() methods
* abort() method with suport AbortSignal and AbortController
* isFinished, error and result Properties
* timeout property, setTimeout(), clearTimeout() methods
* ref(), unref() methods, isRef property
* onFinish Callback
* sleep(ms), applyTimeout(promise, ms)

## Install
```
npm i hkey-extended-promise
```
then

```js
const ExtendedPromise = require('hkey-extended-promise'); 
```
OR
```js
const {ExtendedPromise} = require('hkey-extended-promise'); 
```

OR
```js
const { 
	ExtendedPromise, 
	sleep, SleepPromise,
	applyTimeout, ApplyTimeoutPromise,	
	TimeoutError, DontFinishError,
	PseudoAbortError, isAbortError, 
} = require('hkey-extended-promise'); 
```

## Usage
```js
const promise = new ExtendedPromise();
setTimeout(()=>promise.resolve(), 100);
await promise;
```

### sleep(ms)
the code above is equivalent to `await sleep(100);` or `await (new SleepPromise(100))`

```
const {..., sleep, SleepPromise, ... } = require('hkey-extended-promise');   
...
await sleep(100);
...
```


### applyTimeout(innerPromise, ms, opts={})
If the embedded promise (innerPromise) does not complete within the specified time, ApplyTimeoutPromise will throw TimeoutError
```js
const {..., applyTimeout, ApplyTimeoutPromise, ... } = require('hkey-extended-promise');   
...
const innerPromise = ... // new Promise(...)
...
try{
	await applyTimeout(innerPromise, 100);
} catch(e){
	if(e instance of TimeoutError){
		...
	}
}
```

### timeout
```js
const promise = new ExtendedPromise({timeout: 100});
```

OR

```js
const promise = new ExtendedPromise();
promise.timeout = 100; 
// OR promise.setTimeout(100);
```


#### TimeoutError
```js
const {..., TimeoutError, ... } = require('hkey-extended-promise');   
...
try{
	await new ExtendedPromise({...,  timeout: 1000});
} catch (e){
	if(e instance of TimeoutError){
		...
	}
}
```

#### clearTimeout
```js
promise.timeout = false; 
```

OR
```js
promise.clearTimeout();
```

#### Get ms till timeout fires
Just use `promise.timeout`

```js
const promise = new ExtendedPromise({timeout: 100});
console.log(promise.timeout); // 100;

await sleep(20);
console.log(promise.timeout); // 80;

promise.timeout = false;
console.log(promise.timeout); // Infinity;

```
	
#### timeout in sleep() and applyTimeout()
You can use timeout on SleepPromise(`sleep()`) and ApplyTimeoutPromise(`applyTimeout()`) just as in ExtendedPromise 

```js
const promise = sleep(100); 
console.log(promise.timeout); // 100;

promise.timeout = 1000; // OR promise.setTimeout(1000);
console.log(promise.timeout); // 1000;
```

In SleepPromise(`sleep()`) timeout cases promise resolve. On other cases on timeout promise will be rejected (throw TimeoutError);

### promise.isFinished, promise.result, promise.error

if promise.isFinished=true then promise is finised and promise.result or promise.error filled.
```js
...
const promise = new ExtendedPromise(...);
...
if(promise.isFinished){
	if(promise.error){
		console.log('promise finised with error', promise.error);
	} else {
		console.log('promise resolved', promise.result);
	}
} else {
	console.log('promise is pending');
}
```

### onFinish(promise, result, error)
```js
new ExtendedPromise({onFinish: function(promise, result, error){
	...
}})
```

## Advances Usage

### abort()
Extended promises have `promise.abort()` method. By defaut it equal to  `promise.reject(new PseudoAbortError())`

Unfortunately there is no access to the AbortError in the Node.js. You need to check it using `isAbortError(error)` function;

```js
const promise = new ExtendedPromise();
setTimeout(()=>promise.abort(), 100); 
		
try{
	await promise;	
} catch(error){
	if(isAbortError(error)){
		...
	}
}
```
You can use abortSignal or abortController or true or false as abort option

```js
new ExtendedPromise({abort: ...});
```

#### abort=false (defaut)

`new ExtendedPromise({abort: false})` or `new ExtendedPromise({})` or `new ExtendedPromise()`

`promise.abort()` equal to `promise.reject(new PseudoAbortError())`
	
#### abort=true 

```js
const promise =  new ExtendedPromise({abort: true});
```

Properties promise.abortController and promise.abortSignal will be generated

On `promise.abort()` or on timeout promise.abortController will be aborted	

`promise.abort()` equal to `promise.abortController.abort()`

On `promise.abortController.abort()` promise will throw AbortError (check with isAbortError())

#### abort=abortController
```js
const abortController = new AbortController();
const promise = new ExtendedPromise({abort: abortController});
```

On `promise.abort()` or on timeout abortController will be aborted	

`promise.abort()` equal to `abortController.abort()`

if you `abortController.abort()` promise will throw AbortError (check with isAbortError())
 
#### abort=abortSignal
```js
const abortController = new AbortController();
const abortSignal = abortController.signal;
const promise = new ExtendedPromise({abort: abortSignal});
```

On `promise.abort()` or on timeout promise just throw error but abortController will NOT be aborted	

If abortSignal will be aborted (`abortController.abort()`) promise will be rejected

`promise.abort()` equal to `promise.reject(new PseudoAbortError())`

### promise.ref(), promise.unref() and promise.isRef
As Net.Socket, ExtendedPromise has ref() and unref() methods;

`promise.ref()` or `promise.isRef = true` or `new ExtendedPromise({ref: true})` 
will not let the program exit before promise is finished
	
	
`promise.unref()` or `promise.isRef = false` or `new ExtendedPromise({ref: false})` or by defaut `new ExtendedPromise()` 
will let the program exit before promise is finished
	
### How to change promise result

```js
const {DontFinishError,ExtendedPromise} = require('hkey-extended-promise'); 

new ExtendedPromise({
	onFinish: function(promise, result, error){
		if(...){ //resolve 
			promise.result = 123;
			promise.error  = undefined;
		} 
		if(...){ //reject
			promise.result = undefined;
			promise.error  = new Error(...);
			//OR just throw new Error(...);
		} 
		if(...){ //dont finish (continue pending)
			throw new DontFinishError();
		}
	}
});
```

#### example 
code below equal to `await sleep(100)`

```js
await new ExtendedPromise({
	timeout: 100,
	onFinish: function(promise, result, error){
		if(error instanceof TimeoutError){
			promise.result = true;
			promise.error  = undefined;
		} 
	}
});
```