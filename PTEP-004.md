# PTEP-004 - Core Schema

The core schema defines the essence of how data will be represented 
and grow, it defines what kind of things are core objects, the kinds
of actions which will be performed on them and methods of retrieval.

The goal of this document is to describe standard data formats, access
methods and query operations which can be implemented in a backend-neutral
way and *NOT* to implement another DBMS.

This should also be considered pre-draft quality as partial implementations
of core services will need to be implemented first.


## Semantics

We must first explain the sorts of complexity involved which need
to be right from the get-go and the semantic principles behind it.

### Products, Versions and Variations

One of the downfalls prevelent in the 'torrent industry' is the notion
of Versions and Variations of Products because the sites do not strive
to act like a Shop for their Customers, it becomes a basic inventory
keeping problem which has been successfully tackled for the good part
of a century just by looking at it from a different perspective.

For legal and semantic reasons the this terminology is being used, and
in the long-run it will keep design on-track.

 * Product - something which a Customer desires.

 * Version - chronological or technical difference.

 * Variation - e.g. different bitrate or paint colour.

There may be many versions and variations of versions of a product,
but there will only be one product.

Example:

```
  Product - BurnMaster
  Version - 9000
  Variation - Black, Extra Large
```

Additional Example:

```
  Product - Red Tails
  Version - BRRip 2012
  Variation - x264 - 800MB - YIFY
```

To give an ideal use case, a request can be made for a Product or 
variation of one which does not yet exist in the store, allowing 
the store to pre-fill information based on the users description.


#### Specialisation

This approach is being promoted because any further specialisation
starts to make the database structure much more brittle. While this
is suitable for a specific type of Shop, e.g. a Music Shop, it makes
it extremely hard if they want to start selling Food but are required
by their inventory system to attribute an Album or Artist to everything.

### Links

Objects can be linked to one another by specifying a uniquely identifying
query, e.g. the Type and Unique ID of the target object, and can be tagged
with names for easy manipulation.

Links are intended to be as flexible as possible and turn the otherwise
fairly normal key-value store into a graph structure.

Suitable use cases include:

 * Attaching a torrent to a Forum-Post.
 * Relationships between Products, Versions and Variations.
 * Last Favorited torrent of a user.
 

### Metadata

Metadata can be associated with any component, with more specific
components overriding metadata from the previous.

Example:

```
  Product[GrillMaster].Manufacturer = Derp Industries
  Product[GrillMaster].Country = America
  Product[GrillMaster].Version[9000].Manufacturer = Darp Inc.
  Product[GrillMaster].Version[9000].Country = China
```

Meaning that all GrillMasters are made by Derp Industries, apart from 
the Model 9000, which is made (illicitly) by Darp Inc. in Nanjing. 


### Purchases and Downloads

Customers are able to purchase anything in the Store, their receipt in
our case would be a .torrent file. Converting between terminology a
Snatch would be an item on the customers Receipt etc.

A Purchase entails the Store authorising the customer, keeping receipt
records and handing out the appropriate .torrent files.


## Core Objects

All objects have the following fields:
```
       _id: str, Globally unique ID
     links: dic, objects which this one is linked to
  metadata: dic, descriptive keys/values (e.g. synopsis)
```

It is the job of the database layer to map this structure into whichever logical
storage method is used as long as the standard database operations are supported.

### Example Objects:

```javascript
[{ '_id':  'a1',
  'type': 'product',  
  'metadata': {
    'name': 'GrillMaster',
    'manufacturer': 'Derp Industries',
    'country': 'America',
  }
},

{ '_id':  'b2',
  'type': 'version',  
  'links': {
    0:{'type':'product', '_id':'a1'},
  },
  'metadata': {
    'name': '9000',
    'manufacturer': 'Darp Inc.',
    'country': 'China',
  }
},

{ '_id':  'c3',
  'type': 'variation',  
  'links': {
    0:{'type':'version', '_id':'b2'},
    1:{'type':'product', '_id':'a1'},
    2:{'type':'torrent': '_id':'D0FAA0E413695B39B12D4A4A28E1917CE9450C7B'},
  },
  'metadata': {
    'name': 'Black, Extra Large',
    'comments': 'The only Extra Large model in existance',
  }
}]
```


## Query Operations

Because the schema is flexible and 'standardised' regardless of the type of object the
data can be structured in a way which a small number of database operations can perform
any query necessary for an end-user.

 * Get Object
 * Delete Object
 * Put Object
 * Find by Link
 * Find by Metadata Search
 * Set Link
 * Set Metadata

All example requests are for demonstration purposes only and represented using HTTP
roughly following REST principals. Implementations using other transports will
implement the same database operations but may be slightly different so that programming
feels natural, e.g. using JSON over ØMQ.

### Get Object

Retrieve a single object given the `_id` field.

_Example Request_
```
GET /db/D0FAA0E413695B39B12D4A4A28E1917CE9450C7B HTTP/1.0
Content-Type: application/json
```


### Delete Object

Remove an object given the `_id` 

_Example Request_
```
DELETE /db/D0FAA0E413695B39B12D4A4A28E1917CE9450C7B HTTP/1.0
```


### Put Object

Store a single object in the database.

_Example Request_
```
PUT /db HTTP/1.0
Content-Type: application/json


{ '_id':  'D0FAA0E413695B39B12D4A4A28E1917CE9450C7B',
  'type': 'torrent',  
  'metadata': {
    'name': 'Derp',
    'size': 1002438656,
    'pieces': 957
  }
}
```

Once an object has been `PUT` into the database the `type` field
cannot be changed without deleting and re-inserting into the database.


### Find by Link

Find all object IDs which have the given link.
TODO: implementation confusion!!!


### Find by Metadata Search

Perform a full text search for all objects which match the criteria.
TODO: implementation confusion!!!


### Set Link

Link a single object to another, remove the link if the target is NULL.

_Example Request_
```
POST /db/D0FAA0E413695B39B12D4A4A28E1917CE9450C7B/links
Content-Type: application/json


{'owner':{'type':'user','_id':'a2937c8086cefe9555d2c8aee0cce2f9bd6ad9d4'}}
```
This will create a link on the object to a user, the link will be called `owner`.
Multiple links can be specified at the same time.


### Set Metadata

Set metadata keys, remove if the value is NULL.

_Example Request_
```
POST /db/D0FAA0E413695B39B12D4A4A28E1917CE9450C7B/metadata
Content-Type: application/json


{'imdb':'http://www.imdb.com/title/tt1910272/',
 'wikipedia':'http://en.wikipedia.org/wiki/John_Titor'}
```
Will set the `imdb` and `wikipedia` keys of the objects metadata.


## Implementations

Please note that the core schema may change slightly after feedback from the initial
implementations. We aim to provide reference implementations which support all query
operations on all major types of database systems:

 * SQLite
 * Riak
 * MongoDB

It is noted that any query operations involving full text-metadata search may involve
significant engineering to get it working as fast as expected.

Ideas on how to search this type of schema using Solr has explained by Frederick Giasson at:
http://fgiasson.com/blog/index.php/2009/04/29/rdf-aggregates-and-full-text-search-on-steroids-with-solr/