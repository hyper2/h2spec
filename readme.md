
# h2spec (WIP)

## Abstract

A specification for proper `h()` calls.

There is some compatibility problems among many UI libraries like [`hyperapp`](https://github.com/hyperapp), [`hyperscript`](https://github.com/hyperhype/hyperscript), [`choo`](https://github.com/yoshuawuyts/choo), [`snabbdom`](https://github.com/snabbdom/snabbdom), etc. So this introduces a consistent version of `h()` functions called "h2" or "hyper2".
These projects make the usage consistent no matter what view you're working with: browser dom, vdom, server rendering, terminal, canvas, etc.

Once the spec is more mature, several of these libraries will be forked and aligned with the spec. Patches will be sent upstream where it can be merged/rejected by the authors.  But, there will also be several "official" implementations before this as a proof-of-concept.  Ones that get rejected may be maintained on their own.

## Specification

### `h(tag, data?, children?) -> node`

1. `tag` must be a string and can't contain extra data like classes or ids (e.g. `div.foo` or `div#bar`)
2. If the 2nd parameter is defined but the 3rd parameter is not
    1. And the 2nd is a default javascript object, then it becomes `data`
    2. And the 2nd is an array or _primitive value_, then it becomes `children`
3. `data` must be a default javascript object, unless `node` is too. Otherwise it can only be `null` or `undefined`
4. `children` must be a `node`, _primitive value_, or an array of either

- _primitive value_ is a Boolean, Number, String, `null`, or `undefined` (specified [by ECMAScript](https://www.ecma-international.org/ecma-262/5.1/#sec-4.3.2))

## Projects

 - [`h2spec`](https://github.com/hyper2/h2spec) (this)
 - [`h2dom`](https://github.com/hyper2/h2dom) creates DOM nodes
 - [`h2ml`](https://github.com/hyper2/h2ml) creates HTML strings
 - [`h2json`](https://github.com/hyper2/h2json) creates array trees resembling `h()` calls
 
## FAQ

### Should the return types of `h`-libraries (DOM, VDOMs, strings, etc) cross into each other?

This spec isolates `h` functions' children to their own return type for a reason.
Do not mix return types outside of something experimental.
However, you can change your entire project tree to DOM, VDOM, and strings, because the elements created are compatible.

Elements are symbolized through combining `{ tag, data, children }`.
The `h(tag, data?, children?)` function is what ties the production of a DOM node, VDOM object, string, or anything, together with consistent usage under that simple combination.

While some of these return types do mix well (i.e. VDOMs + strings = SSR) most do not. Handling several content types in each creates needless complexity. This is exactly why `h(...)` should be very well defined in itself, so it is easy to swap out whole trees, because the libraries rely on the same `h` function usage.

### VDOM objects and the `data` parameter

If we take a code scenario like this one:

```js
h('div', h('span', 'hello world'))
```

If `h()` returns a default javascript object we have to make some compromises to distinguish it from the `data` parameter the call sequence could expect. We must specify `null` or use an array:

```js
h('div', null, h('span', 'hello world'))
h('div', [h('span', 'hello world')])
```

This could be seen as a downside, but on the upside it's a consistent behavior no matter what `h()` returns.

### How would a VDOM or JSON abstraction of this look?

Since the elements are only tied together through `tag`, `data`, and `children`, you could just plop those into objects or arrays:

```js
{
  tag: 'div',
  data: { class: 'foo' },
  children: [
    { tag: 'div', data: null, children: 'foo' },
    { tag: 'span', data: null, children: 'bar' },
    // ...
  ]
}
```

Or more compact

```js
['div', { class: 'foo' }, [
  ['div', 'foo'],
  ['span', 'bar']
]]
```

Which could map directly to `h()` calls:

```js
h('div', { class: 'foo' }, [
  h('div', 'foo'),
  h('span', 'bar')
])
```

This is possible because the spec limits trees to a single node type, so you can guarantee the `h(...)` function will be the same for every node.

### Example of possible uses?

```js
h('div', { ...data }, [ ...children|empty ])
h('div', { ...data }, child|empty)
h('div', null, children)
h('div', children)
h('div', { ...data })
h('div', null)
h('div')
```

### How strict is the spec?

Some conditions are critical, such as the parameter's precedence, but others are simple assertions that can be "reasonably ignored" for now to create smaller implementation sizes. For example, checking primitive types and array contents every time can be costly, so they can be assumed. But, an implementation should never support something outside of what the spec allows.

We may create fuzztesters to ensure compatbility as the spec gets more mature.
