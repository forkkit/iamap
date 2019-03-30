# IAMap

An **I**mmutable **A**synchronous **Map**.

An interface to a content store that is not easily organised, such as a content addressed storage system like [IPFS](https://ipfs.io/). IAMap is not intended to store data itself, but rather provide a data structure that can efficiently organise and reference data, and itself be stored and referenced.

## API

### Contents

 * [`async IAMap.create(store, options)`](#IAMap__create)
 * [`async IAMap.load(store, id)`](#IAMap__load)
 * [`IAMap.registerHasher(codec, hasherBytes, hasher)`](#IAMap__registerHasher)
 * [`class IAMap`](#IAMap)
 * [`async IAMap#set(key, value)`](#IAMap_set)
 * [`async IAMap#get(key)`](#IAMap_get)
 * [`async IAMap#has(key)`](#IAMap_has)
 * [`async IAMap#delete(key)`](#IAMap_delete)
 * [`async IAMap#size()`](#IAMap_size)
 * [`async IAMap#keys()`](#IAMap_keys)
 * [`async IAMap#values()`](#IAMap_values)
 * [`async IAMap#ids()`](#IAMap_ids)
 * [`IAMap#toSerializable()`](#IAMap_toSerializable)
 * [`IAMap#entryCount()`](#IAMap_entryCount)
 * [`IAMap#nodeCount()`](#IAMap_nodeCount)
 * [`async IAMap#isInvariant()`](#IAMap_isInvariant)

<a name="IAMap__create"></a>
### `async IAMap.create(store, options)`

```js
let map = await IAMap.create(store, options)
```

Create a new IAMap instance with a backing store. This operation is asynchronous and returns a `Promise` that
resolves to a `IAMap` instance.

**Parameters:**

* **`store`** _(`Object`)_: A backing store for this Map. The store should be able to save and load a serialized
  form of a single node of a IAMap which is provided as a plain object representation. `store.save(node)` takes
  a serializable node and should return a content address / ID for the node. `store.load(id)` serves the inverse
  purpose, taking a content address / ID as provided by a `save()` operation and returning the serialized form
  of a node which can be instantiated by IAMap. In addition, a `store.isEqual(id1, id2)` method is required to
  check the equality of the two content addresses / IDs (which may be custom for that data type).
  The `store` object should take the following form: `{ save(node):id, load(id):node, isEqual(id,id):boolean }`
* **`options`** _(`Object`)_: Options for this IAMap
  * **`options.codec`** _(`string`)_: A [multicodec](https://github.com/multiformats/multicodec/blob/master/table.csv)
    hash function identifier, e.g. `'murmur3-32'`. Hash functions must be registered with [`IAMap.registerHasher`](#IAMap__registerHasher).
  * **`options.bitWidth`** _(`number`)_: The number of bits to extract from the hash to form an element index at
    each level of the Map, e.g. a bitWidth of 5 will extract 5 bits to be used as the element index, since 2^5=32,
    each node will store up to 32 elements (child nodes and/or entry buckets). The maximum depth of the Map is
    determined by `floor((hashBytes * 8) / bitWidth)` where `hashBytes` is the number of bytes the hash function
    produces, e.g. `hashBytes=32` and `bitWidth=5` yields a maximum depth of 51 nodes.
  * **`options.bucketSize`** _(`number`)_: The maximum number of collisions acceptable at each level of the Map. A
    collision in the `bitWidth` index at a given depth will result in entries stored in a bucket (array). Once the
    bucket exceeds `bucketSize`, a new child node is created for that index and all entries in the bucket are
    pushed

<a name="IAMap__load"></a>
### `async IAMap.load(store, id)`

```js
let map = await IAMap.load(store, id)
```

Create a IAMap instance loaded from a serialized form in a backing store. See [`IAMap.create`](#IAMap__create).

**Parameters:**

* **`store`** _(`Object`)_: A backing store for this Map. [`IAMap.create`](#IAMap__create).
* **`id`**: An content address / ID understood by the backing `store`.

<a name="IAMap__registerHasher"></a>
### `IAMap.registerHasher(codec, hasherBytes, hasher)`

```js
IAMap.registerHasher(codec, hashBytes, hasher)
```

Register a new hash function. IAMap has no hash functions by default, at least one is required to create a new
IAMap.

**Parameters:**

* **`codec`** _(`string`)_: A [multicodec](https://github.com/multiformats/multicodec/blob/master/table.csv) hash
  function identifier, e.g. `'murmur3-32'`.
* **`hasherBytes`** _(`number`)_: The number of bytes to use from the result of the `hasher()` function (e.g. `32`)
* **`hasher`** _(`function`)_: A hash function that takes a `Buffer` derived from the `key` values used for this
  Map and returns a `Buffer` (or a `Bufferlike, such that each element of the array contains a single byte value).

<a name="IAMap"></a>
### `class IAMap`

The `IAMap` constructor should not be used directly. Use `IAMap.create()` or `IAMap.load()` to instantiate.

<a name="IAMap_set"></a>
### `async IAMap#set(key, value)`

Asynchronously create a new `IAMap` instance identical to this one but with `key` set to `value`.

**Parameters:**

* **`key`** _(`string|array|Buffer|ArrayBuffer`)_: A key for the `value` being set whereby that same `value` may
  be retrieved with a `get()` operation with the same `key`. The type of the `key` object should either be a
  `Buffer` or be convertable to a `Buffer` via [`Buffer.from()`](https://nodejs.org/api/buffer.html).
* **`value`** _(`any`)_: Any value that can be stored in the backing store. A value could be a serializable object
  or an address or content address or other kind of link to the actual value.

**Return value**  _(`Promise.<IAMap>`)_: A `Promise` containing a new `IAMap` that contains the new key/value pair.

<a name="IAMap_get"></a>
### `async IAMap#get(key)`

Asynchronously find and return a value for the given `key` if it exists within this `IAMap`.

**Parameters:**

* **`key`** _(`string|array|Buffer|ArrayBuffer`)_: A key for the value being sought. See [`IAMap#set`](#IAMap_set) for
  details about acceptable `key` types.

**Return value**  _(`Promise`)_: A `Promise` that resolves to the value being sought if that value exists within this `IAMap`. If the
  key is not found in this `IAMap`, the `Promise` will resolve to `null`.

<a name="IAMap_has"></a>
### `async IAMap#has(key)`

Asynchronously find and return a boolean indicating whether the given `key` exists within this `IAMap`

**Parameters:**

* **`key`** _(`string|array|Buffer|ArrayBuffer`)_: A key to check for existence within this `IAMap`. See
  [`IAMap#set`](#IAMap_set) for details about acceptable `key` types.

**Return value**  _(`Promise.<boolean>`)_: A `Promise` that resolves to either `true` or `false` depending on whether the `key` exists
  within this `IAMap`.

<a name="IAMap_delete"></a>
### `async IAMap#delete(key)`

Asynchronously create a new `IAMap` instance identical to this one but with `key` and its associated
value removed. If the `key` does not exist within this `IAMap`, this instance of `IAMap` is returned.

**Parameters:**

* **`key`** _(`string|array|Buffer|ArrayBuffer`)_: A key to remove. See [`IAMap#set`](#IAMap_set) for details about
  acceptable `key` types.

**Return value**  _(`Promise.<IAMap>`)_: A `Promise` that resolves to a new `IAMap` instance without the given `key` or the same `IAMap`
  instance if `key` does not exist within it.

<a name="IAMap_size"></a>
### `async IAMap#size()`

Asynchronously count the number of key/value pairs contained within this `IAMap`, including its children.

**Return value**  _(`Promise.<number>`)_: A `Promise` with a `number` indicating the number of key/value pairs within this `IAMap` instance.

<a name="IAMap_keys"></a>
### `async IAMap#keys()`

Asynchronously emit all keys that exist within this `IAMap`, including its children. This will cause a full
traversal of all nodes.

**Return value**  _(`AsyncIterator`)_: An async iterator that yields keys. All keys will be in `Buffer` format regardless of which
  format they were inserted via `set()`.

<a name="IAMap_values"></a>
### `async IAMap#values()`

Asynchronously emit all values that exist within this `IAMap`, including its children. This will cause a full
traversal of all nodes.

**Return value**  _(`AsyncIterator`)_: An async iterator that yields values.

<a name="IAMap_ids"></a>
### `async IAMap#ids()`

Asynchronously emit the IDs of this `IAMap` and all of its children.

**Return value**  _(`AsyncIterator`)_: An async iterator that yields the ID of this `IAMap` and all of its children. The type of ID is
  determined by the backing store which is responsible for generating IDs upon `save()` operations.

<a name="IAMap_toSerializable"></a>
### `IAMap#toSerializable()`

Returns a serializable form of this `IAMap` node. The internal representation of this local node is copied into a plain
JavaScript `Object` including a representation of its elements array that the key/value pairs it contains as well as
the identifiers of child nodes.
The backing store can use this representation to create a suitable serialized form. When loading from the backing store,
`IAMap` expects to receive an object with the same layout from which it can instantiate a full `IAMap` object.
For content addressable backing stores, it is expected that the same data in this serializable form will always produce
the same identifier.

```
{
  codec: Buffer
  bitWidth: number
  bucketSize: number
  depth: number
  dataMap: number
  nodeMap: number
  elements: Array
}
```

Where `elements` is an array of a mix of either buckets or links:

* A bucket is an array of two elements, the first being a `key` of type `Buffer` and the second a `value`
  or whatever type has been provided in `set()` operations for this `IAMap`.
* A link is an Object with a single key `'link'` whose value is of the type provided by the backing store in
  `save()` operations.

**Return value**  _(`Object`)_: An object representing the internal state of this local `IAMap` node, including its links to child nodes
  if any.

<a name="IAMap_entryCount"></a>
### `IAMap#entryCount()`

Calculate the number of entries locally stored by this node. Performs a scan of local buckets and adds up
their size.

**Return value**  _(`number`)_: A number representing the number of local entries.

<a name="IAMap_nodeCount"></a>
### `IAMap#nodeCount()`

Calculate the number of child nodes linked by this node. Performs a scan of the local entries and tallies up the
ones containing links.

**Return value**  _(`number`)_: A number representing the number of direct child nodes

<a name="IAMap_isInvariant"></a>
### `async IAMap#isInvariant()`

Asynchronously perform a check on this node and its children that it is in canonical format for the current data.
As this uses `size()` to calculate the total number of entries in this node and its children, it performs a full
scan of nodes and therefore incurs a load and deserialization cost for each child node.
A `false` result from this method suggests a flaw in the implemetation.

**Return value**  _(`Promise.<boolean>`)_: A Promise with a boolean value indicating whether this IAMap is correctly formatted.

## License and Copyright

Licensed under Apache 2.0

Copyright © Rod Vagg
