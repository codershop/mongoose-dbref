mongoose-dbref - Plugin support for DBRef in Mongoose 
==============

### Overview

Mongoose-DBRef is an extension for Mongoose that exposes the DBRef type from the the `node-mongodb-native`
driver as a top level type in the Mongoose ORM and provides some utilities, plugins and patches that allow 
DBRef instances to be dereferenced from the models.

#### Extension contents

The extension provides the following types:

- `DBRef` : this is the schema type that can be used for database references

The extension provides the following plugins:

- `resolveDBRefs` : used to create getter/setter methods that resolve DBRefs
- `dbrefHooks` : adds hooks to schema and model for DBRefs

The extension includes the following monkey-patches:

- `dbref.fetch` : used to resolve the dbref against a supplied connection

The extension provides the following utilities:

- `fetch` : fetches the object referenced by a DBRef value


### Installation
	npm install mongoose-dbref

### Setup
To install all of the types, plugins, patches and utilities provided by the extension into a Mongoose 
instance:

	var mongoose = require("mongoose");
	   
	// Create a connection to your database
	var db = mongoose.createConnection("mongodb://localhost/sampledb");
	
	// Access the mongoose-dbref module and install everything
	var dbref = require("mongoose-dbref");
	var utils = dbref.utils
	
	// Install the types, plugins and monkey patches
	var loaded = dbref.install(mongoose);

The `loaded` value returned contains 2 properties:

- `loaded.types` : the join types that were loaded
- `loaded.plugins` : the extension plugins that were loaded

To just install the types provided by the extension (either all types or a list of named types):

	var mongoose = require("mongoose");
   
	// Create a connection to your database
	var db = mongoose.createConnection("mongodb://localhost/sampledb");

	// Access the mongoose-dbref module
	var dbref = require("mongoose-dbref");
	var utils = dbref.utils
	
	// Install the plugins
	var loaded = dbref.loadTypes(mongoose);

The `loaded` value returned contains the types that were loaded, keyed by the name of each type 
loaded.

To just install the plugins provided by the extension (either all plugins or list of named plugins):

	var mongoose = require("mongoose");
	   
	// Create a connection to your database
	var db = mongoose.createConnection("mongodb://localhost/sampledb");
	
	// Access the mongoose-dbref module
	var dbref = require("mongoose-dbref");
	var utils = dbref.utils
	
	// Install the plugins
	var loaded = dbref.installPlugins(mongoose);

The `loaded` value returned contains the plugins that were loaded, keyed by the name of each plugin 
loaded.

To just install the patches provided by the extension (either all patches or list of named patches):

	var mongoose = require("mongoose");
	   
	// Create a connection to your database
	var db = mongoose.createConnection("mongodb://localhost/sampledb");
	
	// Access the mongoose-dbref module and the utilities
	var dbref = require("mongoose-dbref");
	var utils = dbref.utils;
	
	// Install the monkey patches
	dbref.installPatches(mongoose);

### Using the types
Once you have loaded the types, or installed the whole extension, you can begin to use them.

#### Type: `DBRef`
The `DBRef` type is a top-level schema type that can be used to identfied a field as holding
a MongoDb database reference.  You use the type as you would any other standard type.

	var DBRef = mongoose.SchemaTypes.DBRef;
	
	var LineItemSchema = new Schema({
		order: DBRef,
	 	description: String,
		cost: Number
	});
	
	var OrderSchema = new Schema({
		poNumber: String,
		lineItems: [DBRef]
	});

This will create two schema - `OrderSchema` that has a field (`lineItems`) which can hold 
multiple references, and `LineItemSchema` that has a field `order` with a single reference.

All of the *standard* options (`required`, `index` etc) can be applied to these fields.

### Using the plugins
Once you have installed the plugins, or installed the whole extension, you can begin to use them.

#### Plugin: `resolveDBRefs`
The `resolveDBRefs` plugin can be used to be used to automatically install *getter* and *setter*
methods for fields where a `resolve` option has been set.

These methods will map DBRef values to/from objects via the database connection of the owning 
model.

For example:

	var LineItemSchema = new Schema({
		order: {type: DBRef, resolve: true},
	 	description: String,
		cost: Number
	});

This will create two methods:

- `getOrder(callback)` - this will asynchronously resolve the `DBRef` value in the `order` field

- `setOrder(value)` - this will cast the `value` (a model) to a `DBRef` value that is set in the `order` field

In addition if the `cache` option is set, then the object resolved from teh `DBRef` value will be 
cached in a cache property (`$order` for the `order` field) and the getter method signature
will be changed to `getOrder(callback, force)`.  The additional, optional, parameter `force`
can be used to bypass any cached value.

This plugin can be installed on the mongoose instance or on individual schema, but the "owning"
mongoose instance for the schema must always be specified during the installation.

#### Plugin: `dbrefHooks`
The `dbrefHooks` plugin can be used to create *hooks* that on the `model` and `schema` instances that
map to/from `DBRef` values.

To do this a `_dbref` virtual and a `fetch` static are installed on each `schema` created such that 
the `_dbref` virtual will return the `DBRef` for the `model` instance, and the `fetch` static will
resolve a supplied `DBRef` and invoke a callback function appropriately.

### Using the patches
Once you have installed the patches, or installed the whole extension, you can begin to use them.

#### Patch: `dbref.fetch`
This `fetch` monkey patch to the `DBRef` class can be used to resolve a `DBRef` value when supplied with 
a mongoos database connection and a callback function.

	var mongoose = require("mongoose");
	var db = mongoose.createConnection("mongodb://localhost/sampledb");
	
	var utils = require("mongoose-dbref").utils;
	
	var LineItem = db.model('LineItem');
	
	LineItem.findById("4dee1f473abd4fbc61000001",
		function(err, doc) {
			doc.order.fetch(db, 
				function(err, doc) {
				    if (err) throw err;
					console.log("Order = " + doc);
				});
		});

In this example, the `order` of a specific `LineItem` is fetched using the database connection that
was used to find the `LineItem` instance.

### Using the utilities
Once you have installed the utilities, or installed the whole extension, you can begin to use them.

#### Utility: `fetch`
This `fetch` utility function can be used to resolve a given `DBRef` value when supplied with a mongoose
database connection and a callback function.

	var mongoose = require("mongoose");
	var db = mongoose.createConnection("mongodb://localhost/sampledb");
	
	var utils = require("mongoose-dbref").utils;
	
	var LineItem = db.model('LineItem');
	
	LineItem.findById("4dee1f473abd4fbc61000001",
		function(err, doc) {
			utils.fetch(db, doc.order,
				function(err, doc) {
				    if (err) throw err;
					console.log("Order = " + doc);
				});
		});

In this example, the `order` of a specific `LineItem` is fetched using the database connection that
was used to find the `LineItem` instance.

### Contributors
- [Stuart Hudson](https://github.com/goulash1971)

### License
MIT License

### Acknowledgements
- [Brian Noguchi](https://github.com/bnoguchi) for the 'mongoose-types' extension that was used as a template for this extension

---
### Author
Stuart Hudson		 
