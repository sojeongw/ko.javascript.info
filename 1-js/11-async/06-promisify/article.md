# Promisification

Promisification -- is a long word for a simple transform. It's conversion of a function that accepts a callback into a function returning a promise.
Promisification은 간단한 형태 변환을 뜻하는 단어입니다. 콜백을 받는 함수를 프라미스를 반환하는 함수로 바꾸는 것을 의미하죠.

Such transforms are often needed in real-life, as many functions and libraries are callback-based. But promises are more convenient. So it makes sense to promisify those.
이러한 변환은 많은 함수와 라이브러리가 콜백 기반이기 때문에 실제로 필요할 때가 많습니다. 하지만 프라미스가 더 편리하기 때문에 프라미스화 하는 것이 합리적입니다.

For instance, we have `loadScript(src, callback)` from the chapter <info:callbacks>.
예를 들어 챕터 <info:callbacks>의 `loadScript(src, callback)`를 살펴보도록 하죠.

```js run
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;

  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`Script load error for ${src}`));

  document.head.append(script);
}

// usage:
// loadScript('path/script.js', (err, script) => {...})
```

Let's promisify it. The new `loadScriptPromise(src)` function will do the same, but accept only `src` (no `callback`) and return a promise.
이걸 프라미스화 해봅시다. 새로운 `loadScriptPromise(src)` 함수가 같은 일을 수행할 겁니다. 하지만 `callback`이 아닌 `src`만 받고 프라미스를 반환할 거예요.

```js
let loadScriptPromise = function(src) {
  return new Promise((resolve, reject) => {
    loadScript(src, (err, script) => {
      if (err) reject(err)
      else resolve(script);
    });
  })
}

// usage:
// loadScriptPromise('path/script.js').then(...)
```

Now `loadScriptPromise` fits well in promise-based code.
이제 `loadScriptPromise`가 프라이스 기반의 코드에 잘 들어맞네요.

As we can see, it delegates all the work to the original `loadScript`, providing its own callback that translates to promise `resolve/reject`.
지금까지 본 것처럼 이 코드는 모든 작업을 원래의 `loadScript`에 넘겨버립니다. `resolve/reject` 프라미스로 전환하는 자신의 콜백을 전달하면서요.

In practice we'll probably need to promisify many functions, it makes sense to use a helper.
실전에서 많은 함수를 프라미스화 해야할 때가 있을텐데요, helper(헬퍼)를 사용하면 좋습니다.

We'll call it `promisify(f)`: it accepts a to-promisify function `f` and returns a wrapper function.
이 helper를 `promisify(f)`라고 부릅니다. 프라미스화 하기위한 함수 `f`를 받아서 래퍼 함수로 반환합니다.

That wrapper does the same as in the code above: returns a promise and passes the call to the original `f`, tracking the result in a custom callback:
래퍼 함수는 위의 코드와 같은 역할을 합니다. 커스텀 콜백의 결과를 추적하면서 프라미스를 리턴하고 원래의 함수 `f`를 호출하도록 넘깁니다.

```js
function promisify(f) {
  return function (...args) { // return a wrapper-function
    return new Promise((resolve, reject) => {
      function callback(err, result) { // our custom callback for f
        if (err) {
          return reject(err);
        } else {
          resolve(result);
        }
      }

      args.push(callback); // append our custom callback to the end of f arguments

      f.call(this, ...args); // call the original function
    });
  };
};

// usage:
let loadScriptPromise = promisify(loadScript);
loadScriptPromise(...).then(...);
```
```js
function promisify(f) {
  return function (...args) { // 래퍼 함수 반환
    return new Promise((resolve, reject) => {
      function callback(err, result) { // f에 대한 커스텀 콜백
        if (err) {
          return reject(err);
        } else {
          resolve(result);
        }
      }

      args.push(callback); // f의 인수 마지막에 커스텀 콜백을 덧붙임

      f.call(this, ...args); // 원래의 함수를 호출함
    });
  };
};

// usage:
let loadScriptPromise = promisify(loadScript);
loadScriptPromise(...).then(...);
```

Here we assume that the original function expects a callback with two arguments `(err, result)`. That's what we encounter most often. Then our custom callback is in exactly the right format, and `promisify` works great for such a case.
여기서 원래의 함수는 `(err, result)` 두 인수를 갖고 있는 콜백을 필요로 한다고 가정합니다. 제일 자주 맞닥뜨리는 일이죠. 그러면 커스텀 콜백은 완전히 적합한 포맷이 되고 `promisify`는 이런 경우에 아주 잘 동작합니다.

But what if the original `f` expects a callback with more arguments `callback(err, res1, res2, ...)`?
하지만 원래의 함수 `f`가 `callback(err, res1, res2, ...)`처럼 더 많은 인수를 필요로 한다면 어떻게 할까요?

Here's a more advanced version of `promisify`: if called as `promisify(f, true)`, the promise result will be an array of callback results `[res1, res2, ...]`:
여기 `promisify`의 더 고급 버전이 있습니다. `promisify(f, true)`로 호출되면 프라미스의 결괏값은 `[res1, res2, ...]`처럼 콜백 결괏값의 배열이 될 것입니다.

```js
// promisify(f, true) to get array of results
function promisify(f, manyArgs = false) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      function *!*callback(err, ...results*/!*) { // our custom callback for f
        if (err) {
          return reject(err);
        } else {
          // resolve with all callback results if manyArgs is specified
          *!*resolve(manyArgs ? results : results[0]);*/!*
        }
      }

      args.push(callback);

      f.call(this, ...args);
    });
  };
};

// usage:
f = promisify(f, true);
f(...).then(arrayOfResults => ..., err => ...)
```
```js
// 결괏값의 배열을 받기 위한 promisify(f, true)
function promisify(f, manyArgs = false) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      function *!*callback(err, ...results*/!*) { // f에 대한 커스텀 콜백
        if (err) {
          return reject(err);
        } else {
          // manyArgs가 지정되면 모든 콜백 결괏값이 처리됨
          *!*resolve(manyArgs ? results : results[0]);*/!*
        }
      }

      args.push(callback);

      f.call(this, ...args);
    });
  };
};

// usage:
f = promisify(f, true);
f(...).then(arrayOfResults => ..., err => ...)
```

For more exotic callback formats, like those without `err` at all: `callback(result)`, we can promisify such functions without using the helper, manually.
`err`가 전혀 없는 `callback(result)`처럼 특이한 콜백 포맷을 처리하고 싶다면 helper를 쓰지 않고 수동으로 프라미스화 하면 됩니다.

There are also modules with a bit more flexible promisification functions, e.g. [es6-promisify](https://github.com/digitaldesignlabs/es6-promisify). In Node.js, there's a built-in `util.promisify` function for that.
[es6-promisify](https://github.com/digitaldesignlabs/es6-promisify)와 같이 좀 더 융통성 있는 함수 promisification을 사용하는 모듈도 있습니다. Node.js에는 이를 위한 함수 `util.promisify`가 내장되어 있습니다.

```smart
Promisification is a great approach, especially when you use `async/await` (see the next chapter), but not a total replacement for callbacks.
Promisification은 강력한 접근 방식입니다. 특히 다음 챕터에서 다룰 `async/await`를 사용한다면요. 하지만 콜백의 모든 걸 대체하지는 않습니다.

Remember, a promise may have only one result, but a callback may technically be called many times.
프라미스는 단지 하나의 결과만 갖지만 콜백은 기술적으로 여러 번 호출할 수 있다는 것을 기억해두세요.

So promisification is only meant for functions that call the callback once. Further calls will be ignored.
promisification은 콜백을 한 번만 호출하기 위해 만들어진 함수입니다. 추가적인 호출은 무시합니다.
```
