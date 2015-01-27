# mongoose-gen

## Gist

**mongoose-gen** generates mongoose.Schema instances from plain json documents.

## History and Rationale

This module started life as a way to persist user defined database schemas.

Backend as a Service (BaaS) products allow users to define their data entities entities through a web-based UI.

Basically users edit&save a configuration object which is then translated by the backend services into mongoose.js schemas.

This module is intended to translate a configuration object into mongoose.Schema objects.

## Status

[![NPM](https://nodei.co/npm/mongoose-gen.png?downloads=true&stars=true)](https://nodei.co/npm/mongoose-gen/)

[![NPM](https://nodei.co/npm-dl/mongoose-gen.png?months=12)](https://nodei.co/npm-dl/mongoose-gen/)

| Indicator              |                                                                          |
|:-----------------------|:-------------------------------------------------------------------------|
| continuous integration | [![Build Status](https://travis-ci.org/topliceanu/mongoose-gen.svg?branch=master)](https://travis-ci.org/topliceanu/mongoose-gen) |
| dependency management  | [![Dependency Status](https://david-dm.org/topliceanu/mongoose-gen.svg?style=flat)](https://david-dm.org/topliceanu/mongoose-gen) [![devDependency Status](https://david-dm.org/topliceanu/mongoose-gen/dev-status.svg?style=flat)](https://david-dm.org/topliceanu/mongoose-gen#info=devDependencies) |
| code coverage          | [![Coverage Status](https://coveralls.io/repos/topliceanu/mongoose-gen/badge.svg?branch=master)](https://coveralls.io/r/topliceanu/mongoose-gen?branch=master) |
| change log             | [CHANGELOG](https://github.com/topliceanu/mongoose-gen/blob/master/CHANGELOG.md) |

## Installation

```shell
npm install mongoose-gen --save
```

## Example

In this example we will generate a `book` mongo collection, mongoose.Schema and Model from a simple json document `book.json`.

_book.json_

```json
{
    "title": {"type": "String", "trim": true, "index": true, "required": true},
    "year": {"type": "Number", "max": 2012, "validate": "validateBookYear"},
    "author": {"type": "ObjectId", "ref": "Author", "index": true, "required": true}
}
```

_index.js_

```javascript
var fs = require('fs');

var mongoose = require('mongoose');
var generator = require('mongoose-gen');

// load json
var data = fs.readFileSync('./book.json', {encoding: 'utf8'});
var bookJson = JSON.parse(data);

// In the above _book.json_ file there is a `validateBookYear` token.
// mongoose-gen uses this token to lookup an actual validator function which
// should be registered beforehand. This is how to register validators.
generator.setValidator('validateBookYear', function (value) {
    return (value <= 2015);
});

// Generate the Schema object.
var BookSchema = generator.getSchema(bookJson);

// Connect to mongodb and bind the book model.
mongoose.connect('mongodb://localhost:27017/test-mongoose-gen');
var BookModel = mongoose.model('Book', BookSchema);
```

## Supported Schema Types and Options

**mongoose-gen** aims for the optimal transformation of json documents into mongoose schemas.

However some features cannot be represented in json very well (particulary function validators, setters, getters or defaults) so this utility is using strings that identify pre-registered functions.

Types are expected as strings in the json document and will be converted acordingly (case insensitive). Supported types (and their options) are:


* **String**
    - lowercase: Boolean
    - uppercase: Boolean
    - trim: Boolean
    - match: String - expects a string that will be passed to new Regexp() constructor
    - enum: [String]

* **Number**
    - min: Number
    - max: Number

* **Boolean**

* **Date**

* **Buffer**

* **ObjectId**
    - ref: String - the name of the Model to reference [Required]

* **Mixed**

* **Additional Type Options**
    - type: String - a type from one of the above [Required]
    - _default_: String - identifier of a previously registered default
    - required: Boolean
    - select: Boolean
    - _get_: String - identifier of a previously registered getter
    - _set_: String - identifier of a previously registered setter
    - index: Boolean
    - unique: Boolean
    - sparse: Boolean
    - _validate_: String - identifier of a previously registered getter


**NOTE** Only the types and options defined above are permitted, unrecognized values are whitelisted or generate and exception!

## API

    generator.setValidator(validator: Function): undefined

    generator.setDefault(default: Function): undefined

    generator.setSetter(getter: Function): undefined

    generator.setGetter(setter: Function): undefined

    generator.getSchema(json: Object): mongoose.Schema


## Setters, Getters, Defaults and Validators

Setters, Getters, Defaults and Validators must be pre-registered as such with the `generator` instance.

**mongoose-gen** uses the name under which these were registered to look them up and add them to the mongoose.Schema

The registered name are global to all generated schemas so you can reuse them.


    var generator = require('mongoose-gen');

    generator.addSetter( mySetter, function (value) { .. }); // return a new value
    generator.addGetter( myGetter, function (value) { .. }); // return a new value
    generator.addDefault( myDefault, function (value) { .. }); // return a new value
    generator.addValidator( myValidator, function (value) { .. }); // return Boolean

## Alternatives

* [mongoose-from-json-schema](https://github.com/work-in-progress/mongoose-from-json-schema)
* [json-mongoose-schemadef](https://github.com/adityab/json-mongoose-schemadef)
* [json-schema-converter](https://github.com/Clever/json-schema-converter)
 - the best i've found, it can translate JSONSchema objects into mongoose.Schema valid parameter object.

## Licence

(The MIT License)

Copyright (c) 2012 Alexandru Topliceanu (alexandru.topliceanu@gmail.com)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
