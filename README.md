connection-string
=================

Advanced URL Connection String Parser, with fully optional syntax.

[![Build Status](https://travis-ci.org/vitaly-t/connection-string.svg?branch=master)](https://travis-ci.org/vitaly-t/connection-string)
[![Coverage Status](https://coveralls.io/repos/vitaly-t/connection-string/badge.svg?branch=master)](https://coveralls.io/r/vitaly-t/connection-string?branch=master)

Takes a URL connection string (with every element being optional):

```
protocol://user:password@host1:123,[abcd::]:456/seg1/seg2?p1=val1&p2=val2
```

and converts it into an object that contains only what's specified:

```js
{
    protocol: 'protocol',
    user: 'user',
    password: 'password',
    hosts: [
            {name: 'host1', port: 123, isIPv6: false},
            {name: 'abcd::', port: 456, isIPv6: true}
            ],
    segments: ['seg1', 'seg2'],
    params: {
        p1: 'val1',
        p2: 'val2'
    }
}
```

## Why use it?

Unlike the standard URL parser, this one supports the following:

* Multiple hosts - for connecting to multiple servers
* Fully optional syntax for every element in the connection string
* Configuration of defaults for any missing parameter
* Construction of a connection string from all parameters
* Friendlier access to the URL's segments and parameters
* TypeScript declarations are deployed with the library

**Short-syntax examples:**

* `localhost` => `{hosts: [{name: 'localhost', isIPv6: false}]`
* `localhost:12345` => `{hosts: [{name: 'localhost', port: 12345, isIPv6: false}]`
* `1.2.3.4:123` => `{hosts: [name: '1.2.3.4', port: 123, isIPv6: false}]`
* `[12ab:34cd]:123` => `{hosts: [{name: '12ab:34cd', port: 123, isIPv6: true}]`
* `abc:///seg1?p1=val` => `{protocol: 'abc', segments: ['seg1'], params: {p1: 'val'}}`
* `:12345` => `{hosts: [{port: 12345}]`
* `username@` => `{user: 'username'}`
* `:pass123@` => `{password: 'pass123'}`
* `/seg1/seg2` => `{segments: ['seg1', 'seg2']}`
* `?p1=1&p2=2` => `{params: {p1: '1', p2: '2'}}`

For more short-syntax examples see [Optional Format].

All browsers are supported, plus Node.js v4.0 and later.

## Installing

```
$ npm install connection-string
```

## Usage

* **Node.js**

```js
const parse = require('connection-string');

const obj1 = parse('my-server:12345');

// with a default value:
parse('my-server:12345', {user: 'admin'});
//=> {user: 'admin', hosts: [{name: 'my-server', port: 12345, isIPv6: false}]}
```

or as a class:

```js
const ConnectionString = require('connection-string');

const obj1 = new ConnectionString('my-server:12345');

// with a default value:
const obj2 = new ConnectionString('my-server:12345', {user: 'admin'});
//=> {user: 'admin', hosts: [{name: 'my-server', port: 12345, isIPv6: false}]}
```

* **Browsers**

```html
<script src="./connection-string/src"></script>

<script>
    var obj = new ConnectionString('my-server:12345');
</script>
```

* **TypeScript**

```ts
import {ConnectionString} from 'connection-string'

const a = new ConnectionString('my-server:12345');
```

For details and examples see the [WiKi Pages].

## API

Both the root function and class `ConnectionString` take a second optional parameter `defaults`.
If it is specified, the parser will call method `setDefaults` automatically (see below).

The object returned by the parser contains all the properties as specified in the connection string,
plus two methods: `setDefaults` and `toString` (see below).

### `setDefaults(defaults) => ConnectionString`

The method takes an object with default values, and sets those for all the properties that were not
specified within the connection string, and returns the same object (itself).

You can make use of this method either explicitly, after constructing the class, or implicitly, by
passing the `defaults` object into the parser/constructor.

Please note that while missing defaults for `hosts` and `params` are merged with the existing ones,
`segments` are not, since their order is usually important, so the defaults for `segments` are only
used when no segment exists.

### `toString() => string`

For the root `ConnectionString` object, the method constructs and returns a connection string from
all the current properties.

**Example:**

```js
const a = new ConnectionString('abc://localhost');
a.setDefaults({user: 'guest'});
a.toString(); //=> 'abc://guest@localhost'
```

You can also call `toString()` on both `hosts` property of the object, and individual host
objects, if you want to generate a complete host name from the current properties.

**Example:**

```js
const a = new ConnectionString('abc://my-host:123,[abcd::]:456');
a.hosts.toString(); //=> 'my-host:123,[abcd::]:456'
a.hosts[0].toString(); //=> 'my-host:123'
a.hosts[1].toString(); //=> '[abcd::]:456'
```

### `static parseHost(host) => {name,port,isIPv6} | null`

If you use an external list of default hosts, you may need to parse them separately,
using this method, so they can be passed in as valid objects.

```js
const h = ConnectionString.parseHost('[abcd::]:111');
//=> {name: 'abcd::', port: 111, isIPv6: true}

const a = new ConnectionString('test://localhost:222/dbname', {hosts: [h]});
a.toString();
//=> test://localhost:222,[abcd::]:111/dbname
```

If no valid host information found, the method returns `null`.

[WiKi Pages]:https://github.com/vitaly-t/connection-string/wiki
[Optional Format]:https://github.com/vitaly-t/connection-string/wiki#optional-format
