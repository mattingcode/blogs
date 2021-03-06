# es6中的生成器
在本文中，我们将了解 ECMAScript 6 中引入的生成器（Generator）。先看一看它究竟是什么，然后用几个示例来说明它的用法。
## 什么是 JavaScript 生成器？
生成器是一种可以用来控制迭代器（iterator）的函数，它可以随时暂停，并可以在任意时候恢复。
上面的描述没法说明什么，让我们来看一些例子，解释什么是生成器，以及生成器与 for 循环之类的迭代器有什么区别。
下面是一个 for 循环的例子，它会在执行后立刻返回一些值。这段代码其实就是简单地生成了 0-5 这些数字。
```js
for (let i = 0; i < 5; i += 1) {
  console.log(i);
}
// 它将会立刻返回 0 -> 1 -> 2 -> 3 -> 4

```

现在看看生成器函数。
```js
function * generatorForLoop(num) {
  for (let i = 0; i < num; i += 1) {
    yield console.log(i);
  }
}

const genForLoop = generatorForLoop(5);

genForLoop.next(); // 首先 console.log - 0
genForLoop.next(); // 1
genForLoop.next(); // 2
genForLoop.next(); // 3
genForLoop.next(); // 4
```
上面的代码只是把for循环包裹在一个带`*`号的函数中，但产生了很大的变化——只有在需要的时候它才会进行下一步。

生成器函数和普通函数没有什么区别，最大的不同是，普通函数是一旦被调用，就会一口气执行完。比如下面的代码：
```js
setTimeout(function(){
    console.log("Hello World");
},1);

function foo() {
    // NOTE: don't ever do crazy long-running loops like this
    for (var i=0; i<=1E10; i++) {
        console.log(i);
    }
}

foo();
// 0..1E10
// "Hello World"
```
上面的代码中，for循环阻塞了，但是代码无法暂停，如果代码可以像遥控器控制DVD一样，可以随时被暂停和继播该多好。
es6的生成器函数提供了这种能力，一个`function`可以在执行中被暂停一次，甚至多次，之后可以被继续执行，在此期间我们可以运行其他代码。

在生成器函数体内，我们可以使用`yield`关键字，使用`yield`关键字可以在函数内部暂停代码的执行，这是使代码暂停和重启的唯一办法。

一个生成器函数可以暂停、重启、暂停、重启无数次，其实你可以让生成器函数无限循环，只需配合`while(true){……}` 就可以。
```js
function *fn() {
  while (true) {
      yield console.log('again')
  }
}

let f = fn()
f.next()  // again
f.next()  // again
```


## 生成器语法
生成器语法注意2点，一个是`*`号，另一个是`yield`关键字.

### `*`
函数名前面的`*`定义一个生成器函数
```js
function * generator () {}
function* generator () {}
function *generator () {}

let generator = function * () {}
let generator = function* () {}
let generator = function *() {}

let generator = *() => {} // SyntaxError
let generator = ()* => {} // SyntaxError
let generator = (*) => {} // SyntaxError
```

> 我们并不能使用箭头函数来创建一个生成器。

生成器函数同样可以作为class的成员和对象的成员。
```js
class MyClass {
  *generator() {}
  * generator() {}
}

const obj = {
  *generator() {}
  * generator() {}
}
```

### `yield`关键字
现在让我们一起看看新的关键词 yield。它有些类似 return，但又不完全相同。return 会在完成函数调用后简单地将值返回，在 return 语句之后你无法进行任何操作。
```js
function withReturn(a) {
  let b = 5;
  return a + b;
  b = 6; // 不可能重新定义 b 了
  return a * b; // 这儿新的值没可能返回了
}

withReturn(6); // 11
withReturn(6); // 11
```

而 yield 的工作方式却不同。

```js
function * withYield(a) {
  let b = 5;
  yield a + b;
  b = 6; // 在第一次调用后仍可以重新定义变量
  yield a * b;
}

const calcSix = withYield(6);

calcSix.next().value; // 11
calcSix.next().value; // 36
```

用 yield 返回的值只会返回一次，当你再次调用同一个函数的时候，它会执行至下一个 yield 语句处。

```js
    function *test () {
      console.log(1)
      yield console.log('yield 1')
      console.log(2)
	  yield console.log('yield 2')
	  yield console.log('yield 3')
	  console.log(3)
	  yield console.log('yield 4')
	  console.log(4)
    }
    t1 = test()
    // 只有实例的next()调用，函数里面的代码才会执行。如果只是函数调用，而实例的next方法不调用，则函数的内的代码不会执行
    // 当实例的第一个next() 执行时，函数内的代码执行到第一个yield, 并且执行完yield代码之后暂停。
    t1.next()
    console.log('开始第二个next')

    // 当第二个next()被调用时，会直接忽略掉上次已经执行过的代码，直接从第一个yield的下一行开始执行
    t1.next()

    t1.next()
    t1.next()
    t1.next()

    console.log('此时代码里所有的yield已经执行完了')
    // 这里并不会报错
    t1.next()
```


`yield`关键字会暂停代码的执行，同时把右侧的值返回出去。
返回出去的是一个对象，这个对象有两个属性：`value` 与 `done`。如你所想，`value`为返回值，`done`则会显示生成器是否完成了它的工作。

```js
function * generator() {
  yield 5;
}

const gen = generator();

gen.next(); // {value: 5, done: false}
gen.next(); // {value: undefined, done: true}
gen.next(); // {value: undefined, done: true} - 之后的任何调用都会返回相同的结果

​````

### `yield`表达式
`yield`被称为关键字，而`yield 'abc'`被称为`yield表达式`。
需要注意的地方是，这个表达式的值并不是由`yield` 右侧的代码决定，而是我们下一次重启生成器传入的值，也就是`next()`传入的参数。
如下面的代码：
​```js
function *fn(){
    let x = 1 + (yield 'foo')
    console.log(x)
}
var f = fn() 
console.log(f.next())   
console.log(f.next(2))  
```
上面的代码输出的结果是：
```js
{value: 'foo', done: false}
3
{value: undefined, done:true}
```
第一个`f.next()`执行，生成器运行到`yield 'foo'`暂停，`yield 'foo'`会首先把`'foo'`抛出，然后暂停代码执行。
故第六行`console.log(f.next()) `会输出`{value: 'foo', done: false}`。
此时`(yield 'foo')`表达式，像一个嗷嗷待哺的孩子等待下一个`next()`传入的值，因为`yield表达式`的值，由`next()`传入的参数决定。

第七行调用`f.next(2)`时，`2`被传入生成器，`yield 'foo'`表达式的值由我们传入的值决定，`yield 'foo'`表达式的值为2，`x`值为`3`输出。

看到双向之间的沟通了吗？`yield`把`foo`传出来，暂停代码，接着之后某个时刻，我把`2`传入，`yield`表达式计算出值。

```js
function foo(x) {
    console.log(x);
}
function *bar() {
    yield; // just pause
    foo( yield ); // pause waiting for a parameter to pass into `foo(..)`
}

let b = bar()
console.log(b.next())
console.log(b.next(1))
console.log(b.next(2))
```
上面代码输出的结果依次是：
```js
{value: undefined, done:false} 
{value: undefined, done:false}
2
{value: undefined, done:true}
```
第一个`b.next()`执行到第五行的`yield`, `yield`抛出undefined, 暂停代码执行。
第二个`b.next(1)`执行到`foo( yield )`，会抛出右侧的值，因为没有，则为undefined, 并暂停代码执行。注意！！`foo( yield )`表达式此时不接受第二个`b.next(1)`传入的`1`, 
而是接受下一个`next(2)`传入的值。
第三个`b.next(2)`开始执行第二个`yield`之后的代码，`foo( yield )`接受传入的参数, 这里参数为`2`, 计算出表达式的值为`2`, `foo(2)`被调用，且函数体中之后再没有`yield`关键字了，抛出`done:true`


注意：在生成器函数中，return 返回的对象和yield返回的一致

为了确保你真的理解了，我们再来看一段代码：
```js
function * fn (x) {
  var y = 2 * (yield (x + 1))
  var z = yield (y / 3)
  return (x + y + z)
}

var it = fn(5)

console.log(it.next())
console.log(it.next(12))
console.log(it.next(13))
```
上面代码的执行结果是
```js
{ value: 6, done: false}
{ value: 8, done: false }
{ value: 42, done: true }
```
第一个`next()`执行，`yield (x + 1)` 返回`6`，代码暂停执行，`yield (x + 1)`表达式暂时未定, 由下一个next()传入的参数决定
第二个`next()`执行，传入12，`yield (x + 1)`表达式的值为12, 代码暂停。
第三个`next()`执行，`return`返回的值类似于`yield`返回的，同时没有剩下的`yield`的，`done`为`false`



## yield 委托迭代
带星号的 yield 可以将它的工作委托给另一个生成器。通过这种方式，你就能将多个生成器连接在一起。

```js
function * anotherGenerator(i) {
  yield i + 1;
  yield i + 2;
  yield i + 3;
}

function * generator(i) {
  yield * anotherGenerator(i);
}

var gen = generator(1);

gen.next().value; // 2
gen.next().value; // 3
gen.next().value; // 4
```

### 初始化与方法
生成器是可以被复用的，但是你需要对它们进行初始化。还好初始化的方法十分简单。
```js

function * generator(arg = 'Nothing') {
  yield arg;
}

const gen0 = generator(); // OK
const gen1 = generator('Hello'); // OK
const gen2 = new generator(); // 不 OK
generator().next(); // 可以运行，但每次都会从头开始运行
```

如上所示，gen0 与 gen1 不会互相影响，gen2 完全不会运行（会报错）。因此初始化对于保证程序流程的状态是十分重要的。
下面让我们一起看看生成器给我们提供的方法。
### next() 方法
```js
function * generator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = generator();

gen.next(); // {value: 1, done: false}
gen.next(); // {value: 2, done: false}
gen.next(); // {value: 3, done: false}
gen.next(); // {value: undefined, done: true} 之后所有的 next 调用都会返回同样的输出
```

这是最常用的方法。它每次被调用时都会返回下一个对象。在生成器工作结束时，next() 会将 done 属性设为 true，value 属性设为 undefined。

那么生成器什么时候会结束呢？
如果生成器有n个`yield`关键字，则`.next()`执行过n次后，仍会返回`{done:false}`, 需要执行`n+1`次才会返回`{done:true}`


## for..of
我们不仅可以用 next() 来迭代生成器，还可以用 for of 循环来一次得到生成器所有的值（而不是对象）。
```js
function * generator(arr) {
  for (const el in arr)
    yield el;
}

const gen = generator([0, 1, 2]);

for (const g of gen) {
  console.log(g); // 0 -> 1 -> 2
}

gen.next(); // {value: undefined, done: true}
```
但它不适用于 for in 循环，并且不能直接用数字下标来访问属性：generator[0] = undefined。
### return() 方法

```js
function * generator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = generator();

gen.return(); // {value: undefined, done: true}
gen.return('Heeyyaa'); // {value: "Heeyyaa", done: true}

gen.next(); // {value: undefined, done: true} - 在 return() 之后的所有 next() 调用都会返回相同的输出

```

return() 将会忽略生成器中的任何代码。它会根据传值设定 value，并将 done 设为 true。任何在 return() 之后进行的 next() 调用都会返回 done 属性为 true 的对象。
### throw() 方法
```js
function * generator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = generator();

gen.throw('Something bad'); // 会报错 Error Uncaught Something bad
gen.next(); // {value: undefined, done: true}

```
throw() 做的事非常简单 —— 就是抛出错误。我们可以用 try-catch 来处理。

### 自定义方法的实现
由于我们无法直接访问 Generator 的 constructor，因此如何增加新的方法需要另外说明。下面是我的方法，你也可以用不同的方式实现：
```js
function * generator() {
  yield 1;
}

generator.prototype.__proto__; // Generator {constructor: GeneratorFunction, next: ƒ, return: ƒ, throw: ƒ, Symbol(Symbol.toStringTag): "Generator"}

// 由于 Generator 不是一个全局变量，因此我们只能这么写：
generator.prototype.__proto__.math = function(e = 0) {
  return e * Math.PI;
}

generator.prototype.__proto__; // Generator {math: ƒ, constructor: GeneratorFunction, next: ƒ, return: ƒ, throw: ƒ, …}

const gen = generator();
gen.math(1); // 3.141592653589793

```

### 生成器的用途
在前面，我们用了已知迭代次数的生成器。但如果我们不知道要迭代多少次会怎么样呢？为了解决这个问题，需要在生成器函数中创建一个无限循环。下面以一个会返回随机数的函数为例进行演示：
```js
function * randomFrom(...arr) {
  while (true)
    yield arr[Math.floor(Math.random() * arr.length)];
}

const getRandom = randomFrom(1, 2, 5, 9, 4);

getRandom.next().value; // 返回随机的一个数
```

这是个简单的例子。下面来举一些更复杂的函数为例，我们要写一个节流（throttle）函数。

```js
function * throttle(func, time) {
  let timerID = null;
  
  function throttled(arg) {
    clearTimeout(timerID);
    timerID = setTimeout(func.bind(window, arg), time);
  }
  
  while (true)
    throttled(yield);
}

const thr = throttle(console.log, 1000);

thr.next(); // {value: undefined, done: false}
thr.next('hello'); // 返回 {value: undefined, done: false} ，然后 1 秒后输出 'hello'
```

还有没有更好的利用生成器的例子呢？如果你了解递归，那你肯定听过斐波那契数列。通常我们是用递归来解决这个问题的，但有了生成器后，可以这样写：
```js

function * fibonacci(seed1, seed2) {
  while (true) {
    yield (() => {
      seed2 = seed2 + seed1;
      seed1 = seed2 - seed1;
      return seed2;
    })();
  }
}

const fib = fibonacci(0, 1);
fib.next(); // {value: 1, done: false}
fib.next(); // {value: 2, done: false}
fib.next(); // {value: 3, done: false}
fib.next(); // {value: 5, done: false}
fib.next(); // {value: 8, done: false}
```
不再需要递归了！我们可以在需要的时候获得数列中的下一个数字。
将生成器用在 HTML 上
既然是讨论 JavaScript，那显然要用生成器来操作下 HTML。
假设有一些 HTML 块需要处理，可以使用生成器来轻松实现。（当然除了生成器之外还有很多方法可以做到）
我们只需要少许代码就能完成此需求。
```js
const strings = document.querySelectorAll('.string');
const btn = document.querySelector('#btn');
const className = 'darker';

function * addClassToEach(elements, className) {
  for (const el of Array.from(elements))
    yield el.classList.add(className);
}

const addClassToStrings = addClassToEach(strings, className);

btn.addEventListener('click', (el) => {
  if (addClassToStrings.next().done)
    el.target.classList.add(className);
});
```

仅有 5 行逻辑代码。
## 总结
还有更多使用生成器的方法。例如，在进行异步操作或者按需循环时生成器也非常有用。
我希望这篇文章能帮你更好地理解 JavaScript 生成器。
