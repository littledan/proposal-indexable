# `Indexable`: optimizable overriding of array indexing

## Motivation

ES2015 gave JavaScript Proxy, which allows the implementation of objects in JavaScript that have custom behavior for property access, including the "square bracket operator" `obj[index]`. This is potentially a great boon to library authors; however, the following issues have been faced in practice:
- Proxy is slow in current implementations, and it's not clear if it's feasible to optimize in general due to the richness of its interface.
- The Proxy interface is large and difficult to learn for many developers.

This document proposes a simple parent class that can be used for classes which want to override the square bracket operator for numerical indexes only in a simple way. The hope is that this will be more usable in practice, addressing the above issues.

## Example code

```js
import { Indexable, get, set } from "@std/indexable";

// Defines an Array-like object which reports to have
// keys for all indices between 0 and the position of the last
// value, but actually uses sparse underlying storage.
class SecretlySparseArray extends Indexable {
  #length = 0;
  #map = new Map();
  get length() { return this.#length; }
  [set](index, value) {  // index will be a Number
    this.#map.set(index, value);
    if (index >= this.#length) { this.#length = index + 1; }
  }
  [get](index) {
    return this.#map.get(index);
  }
}
```

## Mechanism

Indexable objects are somewhat like TypedArrays in their object model, in that:
- Integer-indexed properties are always writable, enumerable and non-configurable
- The array is always "dense" in integer-indexed properties, from 0 to the length-1

The super constructor for Indexable returns a new exotic object which treats integer indices specially, by calling into the `set` and `get` symbol-named properties, as well as the `length` property, for their handling. The subclass of Indexable does further initialization on top of the returned Proxy.

## Open questions
- Would it be feasible to optimize Proxy in general instead?
- Is this mechanism optimizable in practice?
- This proposal leaves Indexable things which are subclasses of other things for future work (perhaps making Indexable work as a mixin, but this may be even harder to optimize for). Is this acceptable?
