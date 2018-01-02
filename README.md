# Object.freeze and Object.seal syntax

## Rationale

Object.freeze and Object.seal are both useful functions, but they aren't particularly ergonomic to use, especially when dealing with deeply nested objects or trying to do complex things like create a superset of a frozen object.

In addition, it would be useful to have these syntaxes in other places. It'd be useful to seal a destructuring expression, or freeze function arguments to create immutable bindings.

## Sketches

### Basic freezing of an object

```js
const foo = {#
  a: {#
    b: {#
      c: {#
        d: {#
          e: [# "some string!" #]
        #}
      #}
    #}
  #}
#}
```

<details><summary>Click here to see the desugared version</summary>

```js
const freeze = obj => Object.freeze(Object.create(null), Object.assign(o, obj));

const foo = freeze({
  a: freeze({
    b: freeze({
      c: freeze({
        d: freeze({
          e: Object.freeze([ "some string!" ])
        })
      })
    })
  })
})
```

</details>

### Basic sealing of an object

```js
const foo = {|
  a: {|
    b: {|
      c: {|
        d: {|
          e: [| "some string!" |]
        |}
      |}
    |}
  |}
|}
```

<details><summary>Click here to see the desugared version</summary>

```js
const seal = obj => Object.seal(Object.create(null), Object.assign(o, obj));
const foo = seal({
  a: seal({
    b: seal({
      c: seal({
        d: seal({
          e: Object.seal(["some string!"])
        })
      })
    })
  })
})
```

</details>


### Sealing a functions destructured options bag

```js
function ajax({| url, headers, onSuccess |}) {
  fetch(url, { headers }).then(onSuccess)
}
ajax({ url: 'http://example.com', onsuccess: console.log })
// throws TypeError('cannot define property `onsuccess`. Object is not extensible')
```

<details><summary>Click here to see the desugared version</summary>

```js
function ajax(_ref1) {
  const _ref2 = Object.seal({ url: undefined, headers: undefined, onSuccess: undefined })
  Object.assign(_ref2, _ref1)
  let url = _ref2.url
  let headers = _ref2.headers
  let onSuccess = _ref2.onSuccess

  fetch(url, { headers }).then(onSuccess)
}
ajax({ url: 'http://example.com', onsuccess: console.log })
// throws TypeError('cannot define property `onsuccess`. Object is not extensible')
```

</details>

### Freezing a functions destructured options bag

```js
function ajax({# url, headers, onSuccess #}) {
  url = new URL(url) // throws TypeError('cannot assign to const `url`')
  fetch(url, { headers }).then(onSuccess)
}
ajax({ url: 'http://example.com', onSuccess: console.log })
```

<details><summary>Click here to see the desugared version</summary>

```js
function ajax(_ref1) {
  const _ref2 = Object.seal({ url: undefined, headers: undefined, onSuccess: undefined }) // seal now, const later
  Object.assign(_ref2, _ref1)
  const url = _ref2.url
  const headers = _ref2.headers
  const onSuccess = _ref2.onSuccess

  url = new URL(url) // throws TypeError('cannot assign to const `url`')
  fetch(url, { headers }).then(onSuccess)
}
ajax({ url: 'http://example.com', onSuccess: console.log })
```

</details>


## Potential extensions - sketches

### Sealed function param bindings

```js
function add(| a, b |) {
  return a + b
}
add(2, 2, 2) === 6
// throws TypeError('invalid third parameter, expected 2`)
```

<details><summary>Click here to see the desugared version</summary>

```js
function add(_1, _2) {
  if (arguments.length > 2) {
    throws TypeError('invalid third parameter, expected 2')
  }
  let a = arguments[0]
  let b = arguments[1]

  return a + b
}
add(2, 2, 2) === 6
// throws TypeError('invalid third parameter, expected 2`)
```

</details>

### Frozen function param bindings

```js
function add1(# a #) {
  a += 1 // throws TypeError `invalid assignment...`
  return a
}
add1(1) === 2
```

<details><summary>Click here to see the desugared version</summary>

```js
function add1(_1) {
  if (arguments.length > 1) {
    throws TypeError('invalid second parameter, expected 1')
  }
  const a = arguments[0]

  a += 1 // throws TypeError `invalid assignment...`
  return a
}
add1(1) === 2
```

</details>

### Extending syntax to general destructing for better type safety

```js
const foo = { a: 1, b: 2 }
const {| a, b, c |} = foo
// Throws TypeError 'invalid assignment to unknown property c'
```

<details><summary>Click here to see the desugared version</summary>

```js
const foo = { a: 1, b: 2 }
if (!('a' in foo)) throw new TypeError('invalid assignment to unknown property c')
const a = foo.a
if (!('b' in foo)) throw new TypeError('invalid assignment to unknown property c')
const b = foo.b
if (!('c' in foo)) throw new TypeError('invalid assignment to unknown property c')
const c = foo.c
// Throws TypeError 'invalid assignment to unknown property c'
```

</details>
