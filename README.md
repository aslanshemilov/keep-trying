# keep-trying [![Build Status](https://travis-ci.org/machuga/keep-trying.svg?branch=master)](https://travis-ci.org/machuga/keep-trying.svg?branch=master)

A minimal function to allow retrying promises up to a given limit.

This library was inspired by some callback retry code I used to use,
https://github.com/poetic/retryable-promise, and
https://github.com/valeriangalliat/promise-retryable.

It is useful when you have some action you need to call that may fail
the first time.

The API is intended to be very simple, and leverages the native Promises
available in ES2015. May be used with Node 6+, any modern browser, or polyfill
that binds to `window.Promise`.

The `keepTrying` function returns a promise that will always yield the result
of the promise passed to it.  If your promise resolves with `"Yay!"`, then
`keepTrying` will resolve that result.  The same applies to rejecting.

See the tests directory for extra examples of usage.

## Installation

```
npm install --save keep-trying
```

## Usage

```js
import keepTrying from 'keep-trying';
```

`keepTrying` is defined as:

```
keepTrying = (fn: PromiseFunction<any>, options : Options = {}): Promise<any> => {

interface Options {
  baseTime?: number;
  maxAttempts?: number;
  maxTime?: number;
  backoffStrategy?: BackoffType;
  jitterStrategy?:  JitterType;
  logger?(status: RetryStatus): any;
}
```

that is,

- First Argument: A function with no parameters that returns a `Promise` to be
  retried on rejection.
- Second Argument: An `Options` object that accepts:
  - `baseTime`: An integer of milliseconds to be used as a base delay time for
    any strategy. With the `exact` strategy, the delay between attempts will always
    be equal to `baseTime`; `linear` will be `baseTime * attempt`, etc.
  - `maxAttempts`: An integer of how many times to attempt a failed promise. `0` will
    will cause no retries, `1` will be one retry, etc.
  - `maxTime`: An integer of milliseconds to act as an upper bound for how long to
  - `backoffStrategy`: `exact`, `linear`, or `exponential` stategies can be
    selected, or a custom backoff strategy of type `BackoffStrategy` may be provided.
  - `jitterStrategy`: `none`, `full`, or `equal` stategies can be
    selected, or a custom jitter strategy of type `JitterStrategy` may be provided.
  - `logger`: A function that accepts a `RetryStatus` object as an argument. May be used
    to log information on failure/try rates or log errors attempts to the console or to
    a service such as [Bugsnag](http://bugsnag.com) or [Sentry](http://sentry.io).

## Examples

```js
import keepTrying from 'keep-trying';
// For Node: const keepTrying = require('keep-trying');

// Create a function that wraps a promise and accepts 0 arguments
const promiseReturningFn = function() {
  return new Promise(function(resolve, reject) {
    const somethingWentWrong = true;
    if (somethingWentWrong) {
      reject(new Error("Something went wrong"));
    } else {
      resolve("Yay!");
    }
  });
};

// Try to resolve the promise from our function up to 3 times
// Uses a backoff of 10ms, multipled by the number of the attempt.
// 150ms is the default, but this example is used in the tests so
// keeping it fast here is good.
keepTrying(promiseReturningFn, { maxAttempts: 3, baseTime: 10 }).then(function(msg) {
  console.log("The promise succeeded!", msg);
}).catch(function(err) {
  console.debug('The promise failed after all 3 attempts :(');
  console.error(err);

  // Rethrow the error if you'd like
  throw err;
});
```

If you want your attempts logged to `console.error` or a service such
as Bugsnag or Sentry, pass a function that accepts a `RetryStatus`
object as an argument to the options object with the key `logger`.

```js
keepTrying(promiseReturningFn, {
  maxAttempts: 3,
  baseTime: 500,
  logger(status) { Bugsnag.notifyException(status.error) }
});

// or

keepTrying(promiseReturningFn, {
  maxAttempts: 3,
  baseTime: 500,
  logger(status) {
    // You can make any transformations on the error message here
    // err.message is in the following format:
    // "Failed on attempt 1 of 3: actual_error_message"
    console.debug("promiseReturningFn failed", status.error);
  }
});
```

## Development

- `git clone git@github.com:machuga/keep-trying`
- `cd keep-trying`
- `npm install`
- `npm run build` - Generates `dist` folder
- `npm test` - Run mocha test suite

## License

[MIT](LICENSE)
