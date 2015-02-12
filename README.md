# `confine` finely-tuned configuration

[![Build Status](https://img.shields.io/travis/brandonhorst/confine.svg?style=flat)](https://travis-ci.org/brandonhorst/confine)
[![Coverage Status](https://img.shields.io/coveralls/brandonhorst/confine.svg?style=flat)](https://coveralls.io/r/brandonhorst/confine)
[![npm](http://img.shields.io/npm/v/confine.svg?style=flat)]()

We live in a post-Sublime Text age. The cool way to store persistant state is in raw JSON. `confine` makes it easy to make some simple assertions about what is in that JSON before passing it to your application logic.

Clearly specifying configuration also gives us power when it comes to GUI auto-generation. For example, see [`react-confine`](https://github.com/brandonhorst/react-confine).

`confine` works similarly to the [Config system](https://atom.io/docs/api/v0.179.0/Config) in the Atom editor, but with no type coercion. It is also modular and extendable, with support for custom types.

## Installation

```sh
npm install confine
```

## Usage

```js
var Confine = require('confine')
var confine = new Confine()
var schema = {
  type: 'object',
  properties: {
    name: {type: 'string'},
    age: {type: 'integer', min: 0, max: 120},
    income: {type: 'number', min: 0},
    universe: {type: 'string', enum: ['Marvel', 'DC']},
    living: {type: 'boolean', default: true},
    alterEgos: {type: 'array', items: {type: 'string'}},
    location: {
      type: 'object',
      properties: {
        city: {type: 'string'},
        state: {type: 'string', regex: /[A-Z]{2}/}
      }
    }
  }
}

var object = {
  name: 'Peter Parker',
  age: 17,
  income: 38123.52,
  alterEgos: ['Spider-Man'],
  universe: 'Marvel',
  location: {city: 'New York', state: 'NY'},
  girlfriend: 'Mary Jane'
}

console.log(confine.validateSchema(schema)) // true
console.log(confine.validate(object, schema)) // true
console.log(confine.normalize(object, schema)) /* {
  name: 'Peter Parker',
  age: 17,
  income: 38123.52,
  living: true,
  alterEgos: ['Spider-Man'],
  universe: 'Marvel',
  location: {city: 'New York', state: 'NY'}
} */
```

## Methods

#### `confine.validateSchema(schema)`

- Returns a boolean indicating if `schema` is valid.

#### `confine.validate(obj, schema)`

- Returns a boolean indicating if `obj` is valid according to `schema`

#### `confine.normalize(obj, schema)`

- Returns an adjusted representation of `obj`, filling in defaults and removing unnecessary fields.
- Throws an error if `validateSchema` returns `false`

## Custom Types

You can add custom types by setting properties on `confine.types`. By default, it understands `integer`, `number`, `string`, `boolean`, `array`, and `object`.

```js
confine.types['typeName'] = {
  validateSchema: function (schema, confine) {...},
  validate: function (obj, schema, confine) {...},
  normalize: function (obj, schema, confine) {...} // optional
}

```

#### `validateSchema(schema, confine)`

- Should return a boolean indicating if `schema` is valid.
- `confine` is provided for subschema parsing (see `lib/array`).

#### `validate(obj, schema, confine)`

- should return a boolean indiciating if the `obj` fits `schema`.
- `confine` is provided for subschema parsing (see `lib/array`).

#### `normalize(obj, schema, confine) // optional`
- should return a version of obj adjusted to fit the schema
- if not provided, `normalize` will return `default` if it exists, or `undefined`
- do not use this to coerce values - this should only be used for adjusting subschema parsing
- see `lib/object` for an example
