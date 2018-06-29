# ES6+ Functional
### by dullus@gmail.com

33 metaprograming, composition
44 sql table like data

2. **[Variable visibility](#h2)**  
  2.1 [Hoisting](#h2-1)  
  2.2 [Shadowing](#h2-2)  
  2.3 [Closures](#h2-3)  
  2.4 [Anonymous function scope - IIFE](#h2-4)  
3. **[Functional concepts](#h3)**  
  3.1 [Currying](#h3-1)  
  3.2 [Higher order functions](#h3-2)  
  3.3 [Curry as Higher order functions](#h3-3)  

# 2. <a name="h2"></a>Variable visibility

## 2.1 <a name="h2-1"></a>Hoisting

All `var` declarations in a function body are implicitly moved to the top
of the function in which they occur. This rearrange variable declaration is
called _hoisting_.

```javascript
function demo(n) {
    for (var i = 0; i < n; i++) {
        var j = n * 10;
    };
    return [i, j];
}
demo(5)  // [5, 50]
```
will become
```javascript
function demo(n) {
    var i, j;
    for (i = 0; i < n; i++) {
        j = n * 10;
    };
    return [i, j];
}
demo(5)  // [5, 50]
```

## 2.2 <a name="h2-2"></a>Shadowing
_Shadowing_ happens when a variable of name _x_ is declared within certain scope and then another
variable of the same name is declared later in lower scope.

```javascript
var shadowed1 = 'a';
var shadowed2 = 'b';
var shadowed3 = 'c';
function doShadow(shadowed1) {
    var shadowed2 = 'bb';
    return `${shadowed1} ${shadowed2} ${shadowed3}`;
}
function noShadow() {
    return `${shadowed1} ${shadowed2} ${shadowed3}`;
}
noShadow();     // "a b c"
doShadow();     // "undefined bb c"
doShadow('aa'); // "aa bb c"
```

## 2.3 <a name="h2-3"></a>Closures
_Closure_ is a **function** that captures the external bindings (i.e. not own arguments) contained in
scope in which it was defined for later use (even if scope has completed). Closures are equivalent of
vampires - they capture minions and give them everlasting life until they themselves are destroyed.

```javascript
var GLOBAL = 'a';
function getLocal() {
    var LOCAL = `__${GLOBAL}`;
    return function() {
        return `GLOBAL: ${GLOBAL}; LOCAL: ${LOCAL}`;
    }
}
var gimmeLocal = getLocal();
/* LOCAL was captured */
gimmeLocal(); // "GLOBAL: a; LOCAL: __a"
/* LOCAL will not change even if we change GLOBAL */
GLOBAL = 'b';
/* see that LOCAL remained */
gimmeLocal(); // "GLOBAL: b; LOCAL: __a"
```

## 2.4 <a name="h2-4"></a>Anonymous function scope - IIFE
_IIFE_ (Immediately invoked function expression) is anonymous function scope _design pattern_. That
allows use closure to have own **private** scope not accessible from outside. Typical pattern is:

```javascript
var havePrivate = (function() {
    var _private = 'Word';
    return {
        set: function(val) {
            _private = val;
        },
        get: function() {
            return _private;
        },
        getUpper: function() {
            return _private.toUpperCase();
        },
        getLower: function() {
            return _private.toLowerCase();
        }
    }
}());
havePrivate            // Object {set: function, get: function, getUpper: function, getLower: function}
havePrivate.get()      // "Word"
havePrivate.getUpper() // "WORD"
havePrivate.getLower() // "word"
```
You can modify object, but will not able access \_private (because closure is created at time of execution).
```javascript
havePrivate.hack = function() {return _private + '!!'}
havePrivate.hack()     // Uncaught ReferenceError: _private is not defined
```
But you can modify it via our _setter_.
```javascript
havePrivate.set('NewWord');
havePrivate.getUpper();  // "NEWWORD"
```
NOTE: Javascript does not protect existing object methods. It is possible to override them and potentially
break existing functionality. So it is wise to have also private functions. For example:
```javascript
var havePrivate = (function() {
    var _private = 'Word';
    function _get() {
        return _private;
    }
    return {
        get: function() {
            return _get();
        },
        getUpper: function() {
            // uses private get()
            return _get().toUpperCase();
        },
        getLower: function() {
            // uses public get()
            return this.get().toLowerCase();
        }
    }
}());
```
Lets hack exposed get() method:
```javascript
havePrivate.get = function() {return 'HaCk'};
havePrivate.getUpper() // "WORD"
havePrivate.getLower() // "hack"
```
See that _getLower_ is hacked because uses publicly overridable _get_ method, but _getUpper_ is intact
because uses private *_get* method. To easily distinguish between private and public vars, functions,
properties or methods we usually use ** \_ ** prefix for private stuff.

# 3 <a name="h3"></a>Functional concepts
## 3.1 <a name="h3-1"></a>Currying
_Curried_ function is one that returns a new function for every logical argument that it takes. It allows
partial application of a function’s arguments. What this means is that you can:
  * a) pass a subset of those arguments and get a function back that’s waiting for the rest of the arguments
  * b) pass all of the arguments a function is expecting and get the result

Simple example:
```js
function greetCurryed(greeting) {
  return function(name) {
    console.log(`${greeting}, ${name}`);
  };
};
/* a) Ahoy will be curried implicitly in greetAhoy function */
var greetAhoy = greetCurryed('Ahoy');
greetAhoy('pirate'); // Ahoy, pirate
greetAhoy('Jack'); // Ahoy, Jack

/* b) or we can supply all params to greetCurried */
greetCurried('Hello')('world') // Hello, world
```

Funny example
```js
var o, b = function(x) {
  return function(y) {
    return {bs: '(o)(o)'};
  };
};
b(o)(o).bs
// "(o)(o)"
```

Currying of existing function **f(n, m) --> f'(n)(m)**
```js
const multiply = (a, b) => a * b
const curryedMultiply = (a) => (b) => multiply(a, b)
curryedMultiply(3)(4)
// 12
const triple = curryedMultiply(3)
triple(4)
// 12
```

Partial Apply **f(n, m) --> f'(m)**
```js
const multiply = (a, b) => a * b
const triple = (b) => multiply(3, b)
triple(4)
// 12
```

Uncurrying **f(n)(m) --> f'(n, m)**
```js
const curryedMultiply = (a) => (b) => a * b
curryedMultiply(3)(4)
// 12
const multiply = (a, b) => curryedMultiply(a)(b)
multiply(3, 4)
// 12
```

## 3.2 <a name="h3-2"></a>Higher order functions
Functions that operate on other functions, either by taking them as arguments or by returning
them, are called _higher-order functions_. Curryied function is example of higher-order function,
because it returns function.

Example of HO function getting functions as params:
```js
const unless = (test, then) => { if (test) then(); }
const repeat = (times, body) => { for (let i = 0; i < times; i++) body(i); }

/** iterate from 0 to 9 and print even values */
repeat(10, n => {
  unless(
    (n % 2 === 0 && n !== 0),
    () => console.log(`${n} is even`)
  )
})
// 2 is even
// 4 is even
// 6 is even
// 8 is even
```

We can make it more functional:
```js
const even = (n) => (n % 2 === 0 && n !== 0)
repeat(10, n => {
  unless(even(n), () => console.log(`${n} is even`))
});
```

Or more fancy:
```js
const unless = (test, then) => { if (test === true) then(); }
const repeat = (times, body) => { for (let i = 0; i < times; i++) body(i); }
const even = (n) => (n % 2 === 0 && n !== 0)
const printMsgAndNumber = (msg, n) => console.log(`${n} ${msg}`)
const printIsEven = (n) => printMsgAndNumber('is even', n);
//@TODO
const isEven = (n) => {unless(even(n), () => printIsEven(n))}
repeat(10, n => {isEven(n)})
```

**NOTE** many array operations functions as _map_, _filter_, _reduce_ .. are in fact higher order
functions because they require iteration callback function as parameter.

## 3.3 <a name="h3-3"></a>Curry as Higher order functions
We can define curry, uncurry and papply as higher order functions as follows.
```js
const curry = f => a => b => f(a, b)
const uncurry = f => (a, b) => f(a)(b)
const papply = (f, a) => b => f(a, b)

const multiply = (a, b) => a * b

var x = curry(multiply)
x(3)(5)
// 15
var y = uncurry(x)
y(2, 4)
// 8
```
