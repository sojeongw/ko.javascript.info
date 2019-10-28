# Generators
# 제너레이터

Regular functions return only one, single value (or nothing).
일반적인 함수는 단 하나의 단일 값을 반환하거나 아무것도 반환하지 않습니다.

Generators can return ("yield") multiple values, one after another, on-demand. They work great with [iterables](info:iterable), allowing to create data streams with ease.
제너레이터는 여러 값을 차례대로 원할 때 언제든 반환할 수 있습니다(이것을 '생성(yield)'한다고 말합니다). 데이터 스트림을 쉽게 만들도록 해주는 [반복 가능한(iterable, 이터러블) 객체](info:iterable)를 함께 사용합니다.

## Generator functions
## 제너레이터 함수

To create a generator, we need a special syntax construct: `function*`, so-called "generator function".
제너레이터를 만들기 위해서는 특수한 구문이 필요합니다. 이른바 '제너레이터 함수'라고 불리는 `function*`입니다.

It looks like this:
이렇게 생겼답니다.

```js
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}
```

Generator functions behave differently from regular ones. When such function is called, it doesn't run its code. Instead it returns a special object, called "generator object", to manage the execution.
제너레이터 함수는 일반적인 함수와는 다른 방식으로 작동합니다. 함수가 호출될 때 자신의 코드를 실행하지 않습니다. 대신 실행을 관리하기 위해 '제너레이터 객체'라는 특수한 객체를 반환합니다. 

Here, take a look:
자, 한번 보시죠.

```js run
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

// "generator function" creates "generator object"
let generator = generateSequence();
*!*
alert(generator); // [object Generator]
*/!*
```
```js run
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

// '제너레이터 함수'가 '제너레이터 객체'를 생성합니다.
let generator = generateSequence();
*!*
alert(generator); // [object Generator]
*/!*
```

The function code execution hasn't started yet:
위 함수 코드는 아직 실행되지 않았습니다.

![](generateSequence-1.svg)

The main method of a generator is `next()`. When called, it runs the execution till the nearest `yield <value>` statement (`value` can be omitted, then it's `undefined`). Then the function execution pauses, and the yielded `value` is returned to the outer code.
제너레이터의 메인 메서드는 `next()`입니다. 이 메서드가 호출되면 가장 가까운 `yield <value>`문이 나올 때까지 실행됩니다.(`value`는 생략할 수 있으며 그때는 `undefined`가 됩니다.) 그러면 함수 실행을 일시 정지하고 생성된 `value`가 외부 코드로 반환됩니다.

The result of `next()` is always an object with two properties:
- `value`: the yielded value.
- `done`: `true` if the function code has finished, otherwise `false`.
`next()`의 결과는 항상 두 프로퍼티를 가진 객체입니다.
- `value`: 생성된 값
- `done`: 함수 코드가 마무리되면 `true`, 아니면 `false`

For instance, here we create the generator and get its first yielded value:
예를 들어 제너레이터를 만들고 처음으로 생성된 값을 살펴봅시다.

```js run
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

let generator = generateSequence();

*!*
let one = generator.next();
*/!*

alert(JSON.stringify(one)); // {value: 1, done: false}
```

As of now, we got the first value only, and the function execution is on the second line:
지금 현재 첫번째 값만 받았고 함수 실행은 두번째 줄에 있습니다.

![](generateSequence-2.svg)

Let's call `generator.next()` again. It resumes the code execution and returns the next `yield`:
`generator.next()`를 다시 호출해봅시다. 코드 실행을 다시 진행하고 다음 `yield`를 반환합니다.

```js
let two = generator.next();

alert(JSON.stringify(two)); // {value: 2, done: false}
```

![](generateSequence-3.svg)

And, if we call it the third time, then the execution reaches `return` statement that finishes the function:
그리고 세번째로 호출한다면 실행은 함수를 끝마치는 `return`문에 도달합니다.

```js
let three = generator.next();

alert(JSON.stringify(three)); // {value: 3, *!*done: true*/!*}
```

![](generateSequence-4.svg)

Now the generator is done. We should see it from `done:true` and process `value:3` as the final result.
이제 제너레이터가 끝났습니다. `done:true`부터 마지막 결과로서의 `value:3`

New calls `generator.next()` don't make sense any more. If we do them, they return the same object: `{done: true}`.
`generator.next()`를 새로 호출하는 건 더 이상 의미가 없습니다. 그렇게 한다면 똑같은 객체 `{done: true}`를 반환할테니까요.

```smart header="`function* f(…)` or `function *f(…)`?"
Both syntaxes are correct.
두 구문 모두 맞습니다.

But usually the first syntax is preferred, as the star `*` denotes that it's a generator function, it describes the kind, not the name, so it should stick with the `function` keyword.
하지만 보통 첫번째 구문을 선호합니다. `*`이 제너레이터 함수라는, 이름이 아닌 종류를 의미하기 때문에 `function` 키워드와 붙여써야 합니다.
```

## Generators are iterable
## 제너레이터와 이터러블 객체

As you probably already guessed looking at the `next()` method, generators are [iterable](info:iterable).
`next()` 메서드를 보고 이미 추측했겠지만 제너레이터는 [이터러블 객체](info:iterable)입니다.

We can get loop over values by `for..of`:
`for..of`를 이용 값을 순회할 수 있습니다.

```js run
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

let generator = generateSequence();

for(let value of generator) {
  alert(value); // 1, then 2
}
```

Looks a lot nicer than calling `.next().value`, right?
`.next().value`를 호출하는 것보다 훨씬 더 깔끔해보이네요, 그렇죠?

...But please note: the example above shows `1`, then `2`, and that's all. It doesn't show `3`!
하지만 알아둘 것이 있습니다. 위의 예제는 `1`, 그리고 `2`를 보여주는 게 끝입니다. `3`을 보여주지 않아요!

It's because `for..of` iteration ignores the last `value`, when `done: true`. So, if we want all results to be shown by `for..of`, we must return them with `yield`:
`for..of` 반복은 `done: true`일 때 마지막 `value`를 무시하기 때문입니다. 그래서 `for..of`로 모든 결과값을 보고 싶다면 `yield`로 반환해야 합니다.

```js run
function* generateSequence() {
  yield 1;
  yield 2;
*!*
  yield 3;
*/!*
}

let generator = generateSequence();

for(let value of generator) {
  alert(value); // 1, then 2, then 3
}
```

As generators are iterable, we can call all related functionality, e.g. the spread operator `...`:
제너레이터는 반복 가능하기 때문에 전개 연산자 `...`와 같이 관련된 모든 기능을 호출할 수 있습니다.

```js run
function* generateSequence() {
  yield 1;
  yield 2;
  yield 3;
}

let sequence = [0, ...generateSequence()];

alert(sequence); // 0, 1, 2, 3
```

In the code above, `...generateSequence()` turns the iterable generator object into array of items (read more about the spread operator in the chapter [](info:rest-parameters-spread-operator#spread-operator))
위의 코드에서 `...generateSequence()`는 반복 가능한 제너레이터 객체를 항목의 배열로 바꿔줍니다.(전개 연산자를 더 알고 싶다면 챕터 [](info:rest-parameters-spread-operator#spread-operator)를 참고하세요.)

## Using generators for iterables
## 반복 가능한 객체를 위해 제너레이터 사용하기

Some time ago, in the chapter [](info:iterable) we created an iterable `range` object that returns values `from..to`.
좀 전에 [](info:iterable) 챕터에서 `from..to`의 값을 반환하는 이터러블 객체 `range`를 만들었습니다.

Here, let's remember the code:
이 코드를 기억해주세요.

```js run
let range = {
  from: 1,
  to: 5,

  // for..of range calls this method once in the very beginning
  // for..of range는 맨 처음에 이 메서드를 한 번 호출합니다.
  [Symbol.iterator]() {
    // ...it returns the iterator object:
    // ...반복 가능한 객체를 반환합니다.
    // onward, for..of works only with that object, asking it for next values
    // 계속해서 for..of가 다음 값을 요청하면서 이 객체의 작업을 수행합니다.
    return {
      current: this.from,
      last: this.to,

      // next() is called on each iteration by the for..of loop
      // for..of 반복문으로 next()가 반복적으로 호출됩니다.
      next() {
        // it should return the value as an object {done:.., value :...}
        // {done:.., value :...} 객체 형태로 값을 반환합니다.
        if (this.current <= this.last) {
          return { done: false, value: this.current++ };
        } else {
          return { done: true };
        }
      }
    };
  }
};

// iteration over range returns numbers from range.from to range.to
// range에 대한 반복문이 range.from에서 range.to까지 숫자를 반환합니다.
alert([...range]); // 1,2,3,4,5
```

We can use a generator function for iteration by providing it as `Symbol.iterator`.
`Symbol.iterator`로 제너레이터 함수

Here's the same `range`, but much more compact:

```js run
let range = {
  from: 1,
  to: 5,

  *[Symbol.iterator]() { // a shorthand for [Symbol.iterator]: function*()
    for(let value = this.from; value <= this.to; value++) {
      yield value;
    }
  }
};

alert( [...range] ); // 1,2,3,4,5
```

That works, because `range[Symbol.iterator]()` now returns a generator, and generator methods are exactly what `for..of` expects:
- it has `.next()` method
- that returns values in the form `{value: ..., done: true/false}`

That's not a coincidence, of course. Generators were added to JavaScript language with iterators in mind, to implement them easier.

The variant with a generator is much more concise than the original iterable code of `range`, and keeps the same functionality.

```smart header="Generators may generate values forever"
In the examples above we generated finite sequences, but we can also make a generator that yields values forever. For instance, an unending sequence of pseudo-random numbers.

That surely would require a `break` (or `return`) in `for..of` over such generator, otherwise the loop would repeat forever and hang.
```

## Generator composition

Generator composition is a special feature of generators that allows to transparently "embed" generators in each other.

For instance, we have a function that generates a sequence of numbers:

```js
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}
```

Now we'd like to reuse it for generation of a more complex sequence:
- first, digits `0..9` (with character codes 48..57),
- followed by uppercase alphabet letters `A..Z` (character codes 65..90)
- followed by lowercase alphabet letters `a..z` (character codes 97..122)

We can use this sequence e.g. to create passwords by selecting characters from it (could add syntax characters as well), but let's generate it first.

In a regular function, to combine results from multiple other functions, we call them, store the results, and then join at the end.

For generators, there's a special `yield*` syntax to "embed" (compose) one generator into another.

The composed generator:

```js run
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}

function* generatePasswordCodes() {

*!*
  // 0..9
  yield* generateSequence(48, 57);

  // A..Z
  yield* generateSequence(65, 90);

  // a..z
  yield* generateSequence(97, 122);
*/!*

}

let str = '';

for(let code of generatePasswordCodes()) {
  str += String.fromCharCode(code);
}

alert(str); // 0..9A..Za..z
```

The `yield*` directive *delegates* the execution to another generator. This term means that `yield* gen` iterates over the generator `gen` and transparently forwards its yields outside. As if the values were yielded by the outer generator.

The result is the same as if we inlined the code from nested generators:

```js run
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}

function* generateAlphaNum() {

*!*
  // yield* generateSequence(48, 57);
  for (let i = 48; i <= 57; i++) yield i;

  // yield* generateSequence(65, 90);
  for (let i = 65; i <= 90; i++) yield i;

  // yield* generateSequence(97, 122);
  for (let i = 97; i <= 122; i++) yield i;
*/!*

}

let str = '';

for(let code of generateAlphaNum()) {
  str += String.fromCharCode(code);
}

alert(str); // 0..9A..Za..z
```

A generator composition is a natural way to insert a flow of one generator into another. It doesn't use extra memory to store intermediate results.

## "yield" is a two-way road

Till this moment, generators were similar to iterable objects, with a special syntax to generate values. But in fact they are much more powerful and flexible.

That's because `yield` is a two-way road: it not only returns the result outside, but also can pass the value inside the generator.

To do so, we should call `generator.next(arg)`, with an argument. That argument becomes the result of `yield`.

Let's see an example:

```js run
function* gen() {
*!*
  // Pass a question to the outer code and wait for an answer
  let result = yield "2 + 2 = ?"; // (*)
*/!*

  alert(result);
}

let generator = gen();

let question = generator.next().value; // <-- yield returns the value

generator.next(4); // --> pass the result into the generator  
```

![](genYield2.svg)

1. The first call `generator.next()` is always without an argument. It starts the execution and returns the result of the first `yield "2+2=?"`. At this point the generator pauses the execution (still on that line).
2. Then, as shown at the picture above, the result of `yield` gets into the `question` variable in the calling code.
3. On `generator.next(4)`, the generator resumes, and `4` gets in as the result: `let result = 4`.

Please note, the outer code does not have to immediately call`next(4)`. It may take time. That's not a problem: the generator will wait.

For instance:

```js
// resume the generator after some time
setTimeout(() => generator.next(4), 1000);
```

As we can see, unlike regular functions, a generator and the calling code can exchange results by passing values in `next/yield`.

To make things more obvious, here's another example, with more calls:

```js run
function* gen() {
  let ask1 = yield "2 + 2 = ?";

  alert(ask1); // 4

  let ask2 = yield "3 * 3 = ?"

  alert(ask2); // 9
}

let generator = gen();

alert( generator.next().value ); // "2 + 2 = ?"

alert( generator.next(4).value ); // "3 * 3 = ?"

alert( generator.next(9).done ); // true
```

The execution picture:

![](genYield2-2.svg)

1. The first `.next()` starts the execution... It reaches the first `yield`.
2. The result is returned to the outer code.
3. The second `.next(4)` passes `4` back to the generator as the result of the first `yield`, and resumes the execution.
4. ...It reaches the second `yield`, that becomes the result of the generator call.
5. The third `next(9)` passes `9` into the generator as the result of the second `yield` and resumes the execution that reaches the end of the function, so `done: true`.

It's like a "ping-pong" game. Each `next(value)` (excluding the first one) passes a value into the generator, that becomes the result of the current `yield`, and then gets back the result of the next `yield`.

## generator.throw

As we observed in the examples above, the outer code may pass a value into the generator, as the result of `yield`.

...But it can also initiate (throw) an error there. That's natural, as an error is a kind of result.

To pass an error into a `yield`, we should call `generator.throw(err)`. In that case, the `err` is thrown in the line with that `yield`.

For instance, here the yield of `"2 + 2 = ?"` leads to an error:

```js run
function* gen() {
  try {
    let result = yield "2 + 2 = ?"; // (1)

    alert("The execution does not reach here, because the exception is thrown above");
  } catch(e) {
    alert(e); // shows the error
  }
}

let generator = gen();

let question = generator.next().value;

*!*
generator.throw(new Error("The answer is not found in my database")); // (2)
*/!*
```

The error, thrown into the generator at the line `(2)` leads to an exception in the line `(1)` with `yield`. In the example above, `try..catch` catches it and shows.

If we don't catch it, then just like any exception, it "falls out" the generator into the calling code.

The current line of the calling code is the line with `generator.throw`, labelled as `(2)`. So we can catch it here, like this:

```js run
function* generate() {
  let result = yield "2 + 2 = ?"; // Error in this line
}

let generator = generate();

let question = generator.next().value;

*!*
try {
  generator.throw(new Error("The answer is not found in my database"));
} catch(e) {
  alert(e); // shows the error
}
*/!*
```

If we don't catch the error there, then, as usual, it falls through to the outer calling code (if any) and, if uncaught, kills the script.

## Summary

- Generators are created by generator functions `function* f(…) {…}`.
- Inside generators (only) there exists a `yield` operator.
- The outer code and the generator may exchange results via `next/yield` calls.

In modern JavaScript, generators are rarely used. But sometimes they come in handy, because the ability of a function to exchange data with the calling code during the execution is quite unique. And, surely, they are great for making iterable objects.

Also, in the next chapter we'll learn async generators, which are used to read streams of asynchronously generated data (e.g paginated fetches over a network) in `for await ... of` loop.

In web-programming we often work with streamed data, so that's another very important use case.
