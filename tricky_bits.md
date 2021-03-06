### Tricky Parts

* [Scope and Context](#scope-and-context)
* [Prototype](#prototype)
* [Constructor](#constructor)
* [Regex](#regex)
* [Primitive-vs-reference](#primitive-vs-reference)
* [Arguments pass by value](#arguments-pass-by-value)
* [This keyword in Arrow function](#this-keyword-in-arrow-function)
* [isNaN vs Number](#isnan-vs-number)
* [Type-casting n coercion](#type-casting-n-coercion)
* [Call toString on an object](#call-tostring-on-an-object)
* [Trim spaces inside string](#trim-spaces-inside-string)
* [Array.prototype.sort secret](#array-sort-secret)

### Scope and Context

- A lexical scope in Javascript means that a variable defined outside a function can be accessible inside another function defined after the variable declaration. But the opposite is not true.
- Context(this) in javascript works in a way that's very similar to dynamic scope in programming languages that support it.
- However, context in arrow function works the same way as lexical scope.

### Prototype
Every JS func (except Fucntion.bind) has a prototype property - an empty object by default. You attach properties and methods on this prototype property in order to implement inheritance.

If an object is created with an object literal `var newObj = {}`, it inherits properties from `Object.prototype` and we say its prototype object (or prototype attribute) is Object.prototype. If an object is created from a constructor function such as `new Object (), new Fruit () or new Array () or new Anything ()`, it inherits from that constructor `Object (), Fruit (), Array (), or Anything ()`. For example, with a function such as Fruit (), each time we create a new instance of Fruit `var aFruit = new Fruit ()`, the new instance’s prototype is set to the prototype from the Fruit constructor, which is `Fruit.prototype`.

```js
var obj = {};
obj.protoype // undefined why is that?
```
There is an internal property called `[[proto]]` which is not accessible to programmers. That property points to object prototype. However, most engines make this property accessible as `__proto__`. When you create an object like
`var o = {a: false, b: 'something', ...}` then `o.__proto__` is `Object.prototype`.

![js-prototype-chain](./js-prototype-chain.png)
![js-prototype-chain-2](./js-prototype-chain-2.png)

### Constructor
The critical feature of constructor invocation (when use new) is that the prototype property of the constructor is used as the prototype of the new object.

Two objects are instances of the same class only if they inherit from the same prototype object.

```js
r instanceof Range // return true if r inherits from Range.prototype
```

### Regex
* Use `g` will make Regex stateful - it remembers the last index of match and will continue the search starting at that index.

```js
var arr = ['defabc', 'abc'];

// will only match 'defabc' but not 'abc' since the last index after first match is 6 - the next index search will start at
var reg = /abc/g;

arr.forEach(ele => reg.test(ele)); // solution is set reg.lastIndex = 0 after each match
```

* `new RegExp(regexp|string)` will create a new regex object which has methods like `test` and `exec`.

* `\\?` represent the question mark in `regex string`.

### primitive-vs-reference
Primitive types are compared by value while Reference type are compared by both reference and value.

```js
3 === 3 // true
[1] === [1] // false
```

### arguments-pass-by-value
In Javascript, we can only access or modify reference-typed value by reference.
  * Primitive values are stored in stack. When variables holding the values are passed into the function, copy of values are passed. Thus, any modifications to arguments' values will not affect variables sitting outside of the function.
  * Reference values are stored in heap. The reference itself, think of it as the pointer, is stored in stack. Same as primitive values, copy of value is passed. Since value is the pointer, changes made on argument will also reflect on outside variables.

```js
var o = {
  a: function () {},
  b: [],
  c: 5
};

function calc(r) {
  r.a = 10;
  r.b = 4;
  r.c = [];
  r = {};
}

calc(o); // {a:10, b:4, c:[]}
```

### this-keyword-in-arrow-function
A few things to keep in mind
  * No `arguments` object
  * No own `this` - always look upward to find `this` context. This differs from traditional functions that have `dynamic this` - its value is determined by how they are called. Arrow functions have `lexical this` - its value is determined by the surrounding scope.
  * Cannot use `apply, call, bind` to change its context - it will be the same value as when they are defined.

### isNaN-vs-Number
`isNaN()` and `Number()` both convert `falsy values excluding undefined but including ' '` such as `'', null, '0', false` into zero. As a result, `isNaN(' ')` or `isNaN('')` returns false. Use `parseInt` will fix this problem as `parseInt` will fail to convert empty string into number.

```js
isNaN(null) // false
isNaN(parseInt('')) // true
```

### type-casting-n-coercion

#### type-casting
   * Explicit type conversion is called `type-casting`. Type conversion happens in the background is called `coercion`.
   ```js
   Number('12'); // 12 type-casting
   1 + '1'; // '11' coercion
   ```
   * All primitives except `null` and `undefined` have object wrappers around their native values.
   ```js
   // use their constructors to cast type
   String(123); // '123'
   Boolean(123); // true
   Number('123'); // 123
   Number(true); // 1
   Number(false); // 0
   ```
   But watch out for this case:
   ```js
   const bool = new Boolean(false);
   if (bool) console.log(1); // it will print 1 since bool is an object - object wrapper is created around its native value
   ```
#### coercion
   * `+` results in a string
   ```js
   '2' + 2 // 22
   15 + '' // '15'
   ```
   * Other mathematical operators such as `-`, `/` and `*` always cast to numbers
   ```js
   new Date('04-02-2018') - '1' // 1522619999999
   '12' / '6' // 2
   12 -'1' / 11
   ```

### call-tostring-on-an-object
Objects have internal value called `[[Class]]` which is a tag that represents object type. `Object.prototype.toString` returns a string
`[object ${tag}]`. Tag can be either built-in tags `Array`, `String` or `Date` or something that is set explicitly.
```js
const dogName = 'Fluffy';

dogName.toString(); // 'Fluffy' (String.prototype.toString called here)
Object.prototype.toString.call(dogName); // '[object String]'
Object.prototype.toString.call([12]); // '[object Array]'
```
Set tag by ES6 `Symbol`
```js
const Dog = function(name) {
  this.name = name;
}
Dog.prototype[Symbol.toStringTag] = 'Dog';

const dog = new Dog('Fluffy');
dog.toString(); // '[object Dog]'
```
Call `toString` on array
```js
const arr = [{}, 1, 2];
arr.toString() // '[object Object],2,3'
```

### trim-spaces-inside-string
`trim` and its siblings `trimStart, trimEnd` ONLY trims whitespaces from the beginning/end of a string. It DOES NOT remove whitespaces in between string. To remove whitespaces in between, use `'Hello World'.replace(/\s/g, '')`

```js
let a = ' Hello World ';

a.trim(); // 'Hello World'
a.trimStart(); // 'Hello World '
a.replace(/\s/g, ''); // 'HelloWorld'
```

### array-sort-secret
`Array.prototype.sort` does not create a copy of original array and sort the copy. Instead, it sorts the original one.

```js
const arr = [{
 value: 12
}, {
 value: 10
}];

const res = arr.sort((a, b) => a.value > b.value ? 1 : -1);

res === arr // true!!!
```

The default sort order is built upon converting the elements into strings, then comparing their sequences of UTF-16 code units values.

```js
apple 0061 0070 0070 006C 0065
apbble 0061 0070 0062 0062 006C 0065
accle 0061 0063 0063 006C 0065 000A
box 0062 006F 0078

['apple', 'apbble', 'box','accle'].sort()
// gives you  accle, apbble, apple, box
```

[Demystifying-the-mysteries-of-sort-in-javascript](https://medium.com/@PurpleGreenLemon/demystifying-the-mysteries-of-sort-in-javascript-515ea5b48c7d)






